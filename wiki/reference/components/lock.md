---
title: "lock"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "旧式锁的状态、卡住门禁、钥匙实体暂存与 GUID 恢复；区别类型匹配、动作成功和实际解锁。"
aliases: ["服务端安全解锁探测杖底座", "锁内保留钥匙", "旧式锁", "lock"]
tags: [component, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/lock.lua", "components/key.lua", "prefabs/diviningrodbase.lua", "prefabs/diviningrod.lua", "componentactions.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# lock

## 快速判断

- 职责与适用：旧式目标锁记录 islocked／isstuck／locktype，留存钥匙实体并存 GUID；原底座用 MAXWELL 且起初卡住，ready 后才允许插入。[组件](../../../raw/current/scripts/components/lock.lua#L31-L46) [原配置](../../../raw/current/scripts/prefabs/diviningrodbase.lua#L65-L70) [就绪](../../../raw/current/scripts/prefabs/diviningrodbase.lua#L22-L29)
- 优先使用：先 CompatableKey(keytype)（源码拼写如此）检查类型／卡住，再按实际场景 Unlock；需要消费型目标回调时比较 [[reference/components/klaussacklock]]，普通栅栏开关优先看 [[reference/components/activatable]]。[方法](../../../raw/current/scripts/components/lock.lua#L68-L94)
- 重要副作用：Unlock 先改状态并调目标回调，再通知钥匙、拆一件／移出主人并 AddChild／RemoveFromScene；不销毁该钥匙。[解锁](../../../raw/current/scripts/components/lock.lua#L80-L94) [暂存](../../../raw/current/scripts/components/lock.lua#L114-L123)
- 修改边界：服务端使用；Unlock 不检类型、不返回成功，UNLOCK 动作即使没改状态也可返回 true。原动作的重新上锁分支被注释，不能据组件方法假定玩家可点击取回。[动作](../../../raw/current/scripts/actions.lua#L3228-L3236)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `locktype`／`isstuck`／`IsStuck()` | 默认 DOOR／false；卡住时移除类型锁标签，复原时补标签。运行时修改类型须考虑旧标签与卡住状态，不能靠隐藏候选代替执行校验。[标签](../../../raw/current/scripts/components/lock.lua#L1-L28) [初始化](../../../raw/current/scripts/components/lock.lua#L31-L46) |
| `CompatableKey(keytype)`／`IsLocked()` | 前者只判断非卡住且类型相等，不检查当前是否已锁；后者直接读 islocked。[方法](../../../raw/current/scripts/components/lock.lua#L68-L77) |
| `Unlock(key?,doer?)` | 非卡住且已锁才执行；有 key 时假定 key／inventoryitem 等组件可用；目标回调 fn(target,key,doer)，钥匙 OnUsed 却未传 doer。无显式返回。[方法](../../../raw/current/scripts/components/lock.lua#L80-L94) |
| `Lock(doer)` | 非卡住且未锁时回调、通知／归还已有 key，再 SetKey(nil)；有钥匙时会直接访问 doer.components，不能省略 doer。[方法](../../../raw/current/scripts/components/lock.lua#L98-L110) |
| `SetKey(key?)` | 旧钥匙 RemoveChild／ReturnToScene，新钥匙 AddChild／RemoveFromScene；自身不从库存移出、不自动发放或销毁。[方法](../../../raw/current/scripts/components/lock.lua#L114-L123) |
| `SetLocked(bool)` | 只在变化时调用目标回调，**回调时 islocked 仍为旧值**，随后赋值；不处理钥匙、不卡 isstuck，不能当 Unlock／Lock 的同义接口。[方法](../../../raw/current/scripts/components/lock.lua#L135-L144) |
| `TestForUnlock(key)`／`unlocktest` | 卡住返回 false；有自定义测试时由 fn(key,target) 自行执行，否则 Unlock(key)；不做类型检查，不透传成功结果。[方法](../../../raw/current/scripts/components/lock.lua#L126-L132) |
| `OnSave`／`OnLoad`／`LoadPostPass` | 保存锁／卡住状态和钥匙 GUID／引用列表；恢复先重放 SetLocked，再设卡住状态，postpass 重连实体。不是只保存钥匙 Prefab 名。[存读](../../../raw/current/scripts/components/lock.lua#L148-L170) |

`unlockable` 标签条件实际是 key 非 nil、islocked 为 true 且未卡住；与“已插入钥匙后自动可取回”的直觉不同。场景候选和物品候选分开读，不能用标签解释为通用门锁权限。[条件](../../../raw/current/scripts/components/lock.lua#L12-L28) [场景候选](../../../raw/current/scripts/componentactions.lua#L520-L523)

## 典型接入

### 服务端逻辑用经过校验的探测杖尝试解锁原底座

目标模组 `modmain.lua`；无新资源，给原底座增加私有服务端事件入口。调用者持有合法 doer 与钥匙实体；不是客户端 RPC，也不跳过 ready。原 OnUnlock 会设卡住、改动画并尝试通知 teleportato，故保留原回调。[宿主](../../../raw/current/scripts/prefabs/diviningrodbase.lua#L6-L29) [原配置](../../../raw/current/scripts/prefabs/diviningrodbase.lua#L65-L70)

```lua
AddPrefabPostInit("diviningrodbase", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:ListenForEvent("my_mod_insert_rod", function(target, data)
        local lock = target.components.lock
        local item = data ~= nil and data.key or nil
        local doer = data ~= nil and data.doer or nil
        if lock == nil or item == nil or not item:IsValid()
            or doer == nil or not doer:IsValid()
            or not doer:HasTag("player") or doer:HasTag("playerghost") then return end
        local key = item.components.key
        local inventoryitem = item.components.inventoryitem
        if item.prefab ~= "diviningrod" or key == nil or inventoryitem == nil
            or inventoryitem:GetGrandOwner() ~= doer
            or not lock:IsLocked() or not lock:CompatableKey(key.keytype) then return end
        lock:Unlock(item, doer)
    end)
end)
```

服务端已有逻辑发 `base:PushEvent("my_mod_insert_rod", { key = rod, doer = player })`；末端持有者规则复用 [[reference/components/inventoryitem#关键接口]]。这里只接受原探测杖，既不把非法组件组合送入 Unlock，也不把 UI 类型标签当执行授权；事件不做距离校验，适用于已授权服务端流程，面向玩家的新交互仍需动作／距离／权限链。

解锁将钥匙留在原底座；停用／重载不回滚已保存状态或取回钥匙。原底座解锁后 isstuck=true，直接 Lock 也不会归还；本例不提供重新启用旧装置的完整方案。[效果](../../../raw/current/scripts/prefabs/diviningrodbase.lua#L6-L18) [锁住条件](../../../raw/current/scripts/components/lock.lua#L98-L110)

## 常见陷阱与限制

- **Unlock 返回 true／动作 true 就一定成功**：方法无成功返回，动作只看组件存在便可 true；先查状态和前提，不用返回值计算奖励。[方法](../../../raw/current/scripts/components/lock.lua#L80-L94) [动作](../../../raw/current/scripts/actions.lua#L3228-L3236)
- **SetLocked(false) 等同插入钥匙**：不处理实体，且回调时旧状态尚未改变；OnLoad 也经过此回调。不能在回调里无条件重复奖励。[设置与加载](../../../raw/current/scripts/components/lock.lua#L135-L162)
- **移除组件会自动归还钥匙**：OnRemoveFromEntity 只清标签；自定义目标删除／卸载需另外核验保留实体的去向，本页不声称自动发放。[清理](../../../raw/current/scripts/components/lock.lua#L49-L53)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：actions.lua（R-01）、componentactions.lua（R-04）对照 build-26148036 更新报告逐条核验，锚点重定位 2 处、结论修订 0 处：UNLOCK 执行（actions.lua:3218-3226，重新上锁分支仍为注释）与场景 unlockable 候选（componentactions.lua:520-523）逐行无变化；hunk 地图 R-07（containers.lua 容器 widget 重构）不在本页 sources，无命中；未覆盖：运行验证。

- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整 lock、key 通知、动作／候选、原底座初始化／ready／回调和探测杖类型；持有者接口复用同版本 Wiki。未全面审校实体父子销毁、跨分片或整个旧传送装置。
- 结构检查：本批格式、来源、链接／锚点、大小与登记检查；完整事件例经已有 Full Moon Lua 5.1 syntax-only，检查不证明授权流程或旧装置可用。
- 游戏运行：未运行；卡住／就绪、重复事件、钥匙归属转移、存读和宿主联动待实测。
