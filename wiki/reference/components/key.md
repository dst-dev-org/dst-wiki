---
title: "key"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "旧式钥匙类型标签与使用／取回通知；匹配、留在锁内及归还由 lock 和动作处理。"
aliases: ["探测杖插入底座后通知", "旧式钥匙类型", "key"]
tags: [component, item, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/key.lua", "components/lock.lua", "prefabs/diviningrod.lua", "prefabs/diviningrodbase.lua", "componentactions.lua", "actions.lua", "constants.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# key

## 快速判断

- 职责与适用：旧式钥匙侧保存 keytype，并转发使用／取回回调；原探测杖 diviningrod 配 MAXWELL，配对 diviningrodbase，不是克劳斯鹿角协议。[组件](../../../raw/current/scripts/components/key.lua#L10-L43) [宿主](../../../raw/current/scripts/prefabs/diviningrod.lua#L183-L186) [目标](../../../raw/current/scripts/prefabs/diviningrodbase.lua#L65-L70)
- 优先使用：设置 keytype 匹配既有 lock；接入通知用 SetOnUsedFn／SetOnRemovedFn。新造克劳斯式消耗钥匙应先查 [[reference/components/klaussackkey]] 和 [[reference/components/klaussacklock]]。[类型与通知](../../../raw/current/scripts/components/key.lua#L1-L43)
- 重要副作用：类型赋值替换 `<type>_key` 标签；OnUsed／OnRemoved 不负责消费或返回成功。lock 使用通知之后才拆出一件并藏入目标。[标签](../../../raw/current/scripts/components/key.lua#L1-L7) [使用顺序](../../../raw/current/scripts/components/lock.lua#L80-L94)
- 修改边界：原宿主服务端挂载；候选遍历 LOCKTYPE 并比较双方标签，不能只给随意字符串就假定原动作可发现。直接 Unlock 不再次做类型匹配。[候选](../../../raw/current/scripts/componentactions.lua#L1459-L1467) [常量](../../../raw/current/scripts/constants.lua#L1989-L1993) [执行](../../../raw/current/scripts/actions.lua#L3228-L3236)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `keytype` | 默认 LOCKTYPE.DOOR；原常量含 DOOR／MAXWELL。修改时移除旧标签并添加新标签，移除组件时清当前标签。[实现](../../../raw/current/scripts/components/key.lua#L1-L25) [常量](../../../raw/current/scripts/constants.lua#L1989-L1993) |
| `SetOnUsedFn(fn)`／`OnUsed(lock,doer)` | fn(key,lock,doer)，无显式结果；**当前 lock:Unlock 只传 lock，不传 doer**，所以此链中的 doer 为 nil。[组件](../../../raw/current/scripts/components/key.lua#L27-L38) [调用点](../../../raw/current/scripts/components/lock.lua#L80-L94) |
| `SetOnRemovedFn(fn)`／`OnRemoved(lock,doer)` | fn(key,lock,doer)，由 lock:Lock 在归还前调用。这里 Removed 指从锁中取出，不是实体 onremove／组件卸载钩子。[方法](../../../raw/current/scripts/components/key.lua#L31-L43) [调用点](../../../raw/current/scripts/components/lock.lua#L98-L110) |

## 典型接入

### 探测杖插入原底座时给目标发送模组通知

目标模组 `modmain.lua`，沿用原探测杖／底座资源；用于其他服务端模组逻辑监听插入行为，不新增 UI 或传送系统。原底座还要先通过 ready 解除 isstuck；普通世界是否生成这些对象不由本例保证。[钥匙安装](../../../raw/current/scripts/prefabs/diviningrod.lua#L175-L193) [底座](../../../raw/current/scripts/prefabs/diviningrodbase.lua#L6-L29)

```lua
AddPrefabPostInit("diviningrod", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local key = inst.components.key
    if key == nil then return end
    local old = key.onused
    key:SetOnUsedFn(function(item, target, doer)
        if old ~= nil then old(item, target, doer) end
        if target ~= nil and target:IsValid() then
            target:PushEvent("my_mod_key_inserted", { key = item })
        end
    end)
end)
```

服务端目标监听 `my_mod_key_inserted`，从 data.key 取得**使用时原实体**；回调发生在拆堆之前，通用堆叠钥匙不能把该引用认定为最终留在锁中的一件，也不能在回调内 Remove 钥匙。没有可靠 doer，通知不伪造玩家身份。原目标解锁回调先执行，之后才有本通知与 SetKey。[顺序](../../../raw/current/scripts/components/lock.lua#L80-L94)

本例不改变类型、原目标效果或钥匙存放；停用并重载后不再安装包装，不需新存档字段。新增通知不证明已经进入场景隐藏状态，想知道锁最终持有什么应在原调用结束后读取 lock.key，见 [[reference/components/lock#关键接口]]。

## 常见陷阱与限制

- **回调返回 false 阻止插入**：OnUsed 不返回结果，lock 也不判断它；资格应在执行链的实际入口校验。[回调](../../../raw/current/scripts/components/key.lua#L35-L38) [消费者](../../../raw/current/scripts/components/lock.lua#L80-L94)
- **SetOnRemovedFn 是销毁清理**：仅通知锁归还流程；移除组件只清标签，不执行 onremoved。[移除](../../../raw/current/scripts/components/key.lua#L21-L25) [取回](../../../raw/current/scripts/components/lock.lua#L98-L110)
- **给 keytype 任意字符串就有新动作**：现候选只遍历 LOCKTYPE；原 keytype 在服务端组件中设置，自定义网络对象仍需核验 pristine 标签与动作可见性，不能只复制这一行。[候选](../../../raw/current/scripts/componentactions.lua#L1459-L1467) [原工厂](../../../raw/current/scripts/prefabs/diviningrod.lua#L175-L186)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：actions.lua（R-01）、componentactions.lua（R-04）、constants.lua（R-06）对照 build-26148036 更新报告逐条核验，锚点重定位 3 处、结论修订 0 处：key／klaussackkey 候选（componentactions.lua:1459-1476）、UNLOCK 执行与 LOCKTYPE 常量均逐行无变化；hunk 地图 R-07（containers.lua 容器 widget 重构）不在本页 sources，无命中；未覆盖：运行验证。

- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖 key 全实现、lock 使用／归还、两种 key 候选入口、UNLOCK 执行、LOCKTYPE 与探测杖／底座宿主。未审校探测 FX 全链、旧传送装置整体可用性或自定义客户端动作复制。
- 结构检查：本批元数据、来源、链接／源码锚点、大小与登记检查；完整通知示例经已有 Full Moon Lua 5.1 syntax-only。结果见本轮工程检视。
- 游戏运行：未运行；ready、插入／取回、通知先后、断线读档及其他模组包装冲突待实测。
