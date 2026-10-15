---
title: "health"
created: "2026-10-12"
updated: "2026-10-28"
type: component
summary: "修改生命上限、回血与来源再生，区分人物同步数值边界、生命变化和战斗受击。"
aliases: ["生命上限", "人物最大生命值", "生命值最大值", "65535", "回血", "持续回血", "装备回血", "扣血", "死亡", "health"]
tags: [component, lifecycle, callback, mechanism, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/health.lua", "components/health_replica.lua", "prefabs/player_classified.lua", "components/combat.lua", "components/equippable.lua", "prefabs/player_common.lua", "prefabs/wilson.lua", "prefabs/cane.lua", "prefabs/wormwood.lua", "modutil.lua", "mainfunctions.lua", "entityscript.lua", "class.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# health

## 快速判断

- 职责与适用：实体生命、上限／惩罚、再生和死亡；普通攻击先走 [[reference/components/combat#伤害与事件顺序]]，直接扣血不经过装备护甲链。[combat.lua:624](../../../raw/archive/build-26148036/scripts/components/combat.lua#L624
- 优先使用：上限用 `SetMaxHealth`，当前增减用 `DoDelta`；可叠加的持续回血用 `AddRegenSource`／`RemoveRegenSource`。[health.lua:503](../../../raw/archive/build-26148036/scripts/components/health.lua#L503)、[:353](../../../raw/archive/build-26148036/scripts/components/health.lua#L353)
- 重要副作用：设置上限会补满，再按惩罚封顶并刷新 HUD；`DoDelta` 可能触发死亡，返回值不保证等于实际生命差。[health.lua:503](../../../raw/archive/build-26148036/scripts/components/health.lua#L503)、[:565](../../../raw/archive/build-26148036/scripts/components/health.lua#L565)、[:613](../../../raw/archive/build-26148036/scripts/components/health.lua#L613)
- 修改边界：在服务端操作已有组件；属性赋值经 Class setter 更新 replica，不需要自行发网络值。回血不是复活接口。[health.lua:9](../../../raw/archive/build-26148036/scripts/components/health.lua#L9)、[:18](../../../raw/archive/build-26148036/scripts/components/health.lua#L18)、[class.lua:36](../../../raw/archive/build-26148036/scripts/class.lua#L36

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `SetMaxHealth(amount)` | 正数上限推荐域 | 设置 maxhealth 和 currenthealth，内部 `ForceUpdateHUD(true)`；不是只改上限。[health.lua:503](../../../raw/archive/build-26148036/scripts/components/health.lua#L503 |
| `DoDelta(amount, overtime?, cause?, ignore_invincible?, afflicter?, ignore_absorb?)` | 正回血／负扣血；返回处理后 amount 或拦截时 0 | 默认受无敌／传送、redirect、吸收、单次伤害上限等影响；事件 amount 是封顶前处理量。[health.lua:613](../../../raw/archive/build-26148036/scripts/components/health.lua#L613 |
| `SetPercent(percent, overtime?, cause?)` | 按基础 maxhealth 的比例 | 先 SetVal，再 DoDelta(0)；不按惩罚后上限换算，不是普通伤害入口。[health.lua:550](../../../raw/archive/build-26148036/scripts/components/health.lua#L550 |
| `GetPercent()`／`GetPercentWithPenalty()`／`GetMaxWithPenalty()` | 基础比例／可用比例／可用上限 | 可用上限为 `maxhealth * (1 - penalty)`；需区分比例分母。[health.lua:477](../../../raw/archive/build-26148036/scripts/components/health.lua#L477)、[:524](../../../raw/archive/build-26148036/scripts/components/health.lua#L524) |
| `SetCurrentHealth(amount)` | 直接赋当前值 | 不做 SetVal 的封顶／死亡和 DoDelta 的事件流程；常规增减优先 DoDelta。[health.lua:499](../../../raw/archive/build-26148036/scripts/components/health.lua#L499 |
| `SetInvincible(val)`／`IsInvincible()` | 开关／真值判断 | setter 发 invincibletoggle；判断还包含状态图 temp_invincible。[health.lua:145](../../../raw/archive/build-26148036/scripts/components/health.lua#L145)、[:485](../../../raw/archive/build-26148036/scripts/components/health.lua#L485) |
| `DeltaPenalty(delta)` | 惩罚比例增量 | 经 SetPenalty 限制后 ForceUpdateHUD；只 SetPenalty 不执行这次封顶刷新。[health.lua:460](../../../raw/archive/build-26148036/scripts/components/health.lua#L460 |
| `AddRegenSource(source, amount, period, key?)` | 每 period 秒回 amount；默认 key 为 "key" | 同来源／key 更新任务，独立来源可并存；source 为实体时自动监听其删除，跳过已死亡宿主。[health.lua:347](../../../raw/archive/build-26148036/scripts/components/health.lua#L347 |
| `RemoveRegenSource(source, key?)` | 省略 key 移除整来源 | 只删自己的来源／key；来源存在但指定 key 不存在时会访问 nil，保持增删配对。[health.lua:407](../../../raw/archive/build-26148036/scripts/components/health.lua#L407 |
| `StartRegen(amount, period, interruptcurrentregen?)`／`StopRegen()` | 单 regen 槽 | 默认重启已有单任务；不管理 regensources，源码建议尤其人物用 AddRegenSource。[health.lua:327](../../../raw/archive/build-26148036/scripts/components/health.lua#L327)、[:448](../../../raw/archive/build-26148036/scripts/components/health.lua#L448) |

### 人物最大生命值是否有全局上限

`SetMaxHealth(amount)` 本体直接赋 `maxhealth/currenthealth`，没有在此把参数钳到一个全局最大数；`SetVal` 对当前血量的封顶，是按本实体的惩罚后上限处理，和“最大生命值自身最多能设多少”不同。[上限赋值](../../../raw/archive/build-26148036/scripts/components/health.lua#L503-L507)、[当前值封顶](../../../raw/archive/build-26148036/scripts/components/health.lua#L565-L581)

原版人物走 `onmaxhealth → replica.health:SetMax → player_classified:SetValue`：`SetValue` 要求输入满足 **0～65535**，越界触发断言，再对合法值向上取整写网络值；`currenthealth/maxhealth` 都是 `net_ushortint`。因此人物常规同步通路不能任意设置超大生命值，也不是超过就自动裁剪。此限制属于具有该 classified 的人物通路，不应泛化为所有非人物实体的 health 上限。[属性回调](../../../raw/archive/build-26148036/scripts/components/health.lua#L9-L10)、[replica 转发](../../../raw/archive/build-26148036/scripts/components/health_replica.lua#L51-L60)、[数值断言](../../../raw/archive/build-26148036/scripts/prefabs/player_classified.lua#L11-L14)、[网络字段](../../../raw/archive/build-26148036/scripts/prefabs/player_classified.lua#L1360-L1361)

## 典型接入

### 修改威尔逊生命上限

目标模组已加载的 `modmain.lua`，普通生存模式；复用 wilson 工厂与资源。玩家工厂已在服务端建立 health 并配置默认上限，Prefab postinit 在工厂返回后执行。[player_common.lua:2754](../../../raw/archive/build-26148036/scripts/prefabs/player_common.lua#L2754)、[wilson.lua:1](../../../raw/archive/build-26148036/scripts/prefabs/wilson.lua#L1)、[mainfunctions.lua:387](../../../raw/archive/build-26148036/scripts/mainfunctions.lua#L387

```lua
AddPrefabPostInit("wilson", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.health ~= nil then
        inst.components.health:SetMaxHealth(230)
    end
end)
```

230 为基础上限；这里选择生成时初始化，不在每次受击／重载后强制补满。存档当前值随后由 OnLoad 恢复；停用后新实例按原版上限创建。运行中改上限若要保留原比例，先保存 GetPercent，再 SetMaxHealth、SetPercent；惩罚仍封顶。[health.lua:154](../../../raw/archive/build-26148036/scripts/components/health.lua#L154)、[entityscript.lua:1983](../../../raw/archive/build-26148036/scripts/entityscript.lua#L1983

### 装备手杖每十秒回复一点生命

目标模组 `modmain.lua`；保留原版手持显示回调，无新增资源。把物品作为 source，装备时注册、卸装时撤销；数量为每次回复量，周期为真实秒，不是每分钟数值。[cane.lua:7](../../../raw/archive/build-26148036/scripts/prefabs/cane.lua#L7)、[:20](../../../raw/archive/build-26148036/scripts/prefabs/cane.lua#L20)、[health.lua:353](../../../raw/archive/build-26148036/scripts/components/health.lua#L353

```lua
AddPrefabPostInit("cane", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local eq = inst.components.equippable
    if eq == nil then return end
    local oldequip, oldunequip = eq.onequipfn, eq.onunequipfn
    eq:SetOnEquip(function(item, owner, from_ground)
        if oldequip ~= nil then oldequip(item, owner, from_ground) end
        if owner.components.health ~= nil then
            owner.components.health:AddRegenSource(item, 1, 10, "dstwiki_regen")
        end
    end)
    eq:SetOnUnequip(function(item, owner)
        if owner.components.health ~= nil then
            owner.components.health:RemoveRegenSource(item, "dstwiki_regen")
        end
        if oldunequip ~= nil then oldunequip(item, owner) end
    end)
end)
```

物品删除也由来源监听取消任务；仅移出装备但物品仍存在时仍需卸装撤销。无敌／传送等可能拦截回复；不复活人物，也未提供假人或其他模组改写回调的兼容保证。原版条件再生参照虫木光合作用的成对增删。[wormwood.lua:723](../../../raw/archive/build-26148036/scripts/prefabs/wormwood.lua#L723)；装备回调顺序见 [[reference/components/equippable]]。

## 生命周期

- `OnSave` 默认保存当前生命和惩罚；只有 save_maxhealth 启用才保存动态上限，再生来源不由此存档。条件效果须从持有／装备等实际状态重新建立。[health.lua:154](../../../raw/archive/build-26148036/scripts/components/health.lua#L154
- 死亡跨过零时先向世界发 entity_death，再向实体发 death；默认还可褪去／删除。正 DoDelta 不是角色复活流程，自建生物要明确 nofadeout 与死亡收尾。[health.lua:583](../../../raw/archive/build-26148036/scripts/components/health.lua#L583
- 移除组件的宿主先停止组件更新，再调用 OnRemoveFromEntity，health 清理单再生及来源任务。[entityscript.lua:648](../../../raw/archive/build-26148036/scripts/entityscript.lua#L648)、[health.lua:116](../../../raw/archive/build-26148036/scripts/components/health.lua#L116

## 常见陷阱与限制

- **用 SetMaxHealth 当无副作用上限修改**：会补满并内置刷新；无需为“网络同步”再惯例调用 DoDelta(0)。保留比例见上文，真实伤害用 DoDelta。[health.lua:503](../../../raw/archive/build-26148036/scripts/components/health.lua#L503
- **把返回量／healthdelta.amount 当实际掉血**：SetVal 还会按上限、minhealth 封顶；需要实际差时记录调用前后的 currenthealth，事件监听也可能再次改值。[health.lua:565](../../../raw/archive/build-26148036/scripts/components/health.lua#L565)、[:642](../../../raw/archive/build-26148036/scripts/components/health.lua#L642)
- **认为每次扣血都会发 damaged／attacked**：health 的入口发 healthdelta，受击事件属于 combat；不要用不存在的统一 damaged 事件实现所有伤害触发。[health.lua:642](../../../raw/archive/build-26148036/scripts/components/health.lua#L642)、[combat.lua:715](../../../raw/archive/build-26148036/scripts/components/combat.lua#L715
- **卸下时 StopRegen 撤销自己的 buff**：可能误停他人的单再生，而且不会移除 AddRegenSource；按自己的 source／key 撤销。[health.lua:407](../../../raw/archive/build-26148036/scripts/components/health.lua#L407)、[:448](../../../raw/archive/build-26148036/scripts/components/health.lua#L448)

## 验证记录

- 局部增补：2026-10-27，Steam buildid 26148036；核验 SetMaxHealth／SetVal 与 maxhealth 属性回调、health_replica 转发及人物 classified 的 0～65535 断言／网络字段；原生命、再生及示例结论沿用上述既有记录，未重新审校完整人物同步协议，未运行游戏。
- 版本增量复核：2026-10-25，Steam buildid 26148036；相关调用与示例仍适用，本轮仅重定位证据（锚点 5 处），未运行游戏。

- 源码审阅：2026-10-12，Steam buildid 25643519；核验上述生命／惩罚、再生、死亡、保存恢复与宿主／装备接入路径；未审校完整火焰／月焰、所有动态类型和特殊角色复活链。
- 结构检查：页面／索引、链接与源码行锚点检查通过；两段 Lua 经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；上限重载、装备／卸下／删除、死亡与再生、多人和其他模组兼容仍待实测。

相关：[[reference/components/combat]]；[[reference/components/eater]]；[[reference/components/armor]]；旺达寿命与老化见 [[reference/components/oldager]]。
