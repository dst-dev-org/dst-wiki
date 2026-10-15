---
title: catcher：角色接取飞行投射物
created: 2026-10-22
updated: "2026-10-28"
type: component
summary: 控制角色接取可捕获投射物的提示距离、准备状态和实际接取时机
aliases: [catcher, 接回旋镖, 接取投射物, cancatch, readytocatch]
tags: [component, state-machine, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/catcher.lua, components/projectile.lua, prefabs/player_common.lua, prefabs/boomerang.lua, componentactions.lua, actions.lua, stategraphs/SGwilson.lua, modutil.lua]
reliability: verified
validation: source-reviewed
status: active
---

# catcher：角色接取飞行投射物

## 快速判断

- 职责与适用：挂在**接取者**上，监视飞向自己的 `projectile`，由距离和 SG 的 `readytocatch` 标签决定何时接取；不是捕虫器。原版玩家在主机分支添加该组件。[宿主](../../../raw/current/scripts/prefabs/player_common.lua#L2611-L2615)、[添加组件](../../../raw/current/scripts/prefabs/player_common.lua#L2779-L2784)、[更新判断](../../../raw/current/scripts/components/catcher.lua#L64-L93)
- 优先使用：调整提示距离用 `SetActionDistance`；调整准备姿态下的接取距离用 `SetCatchDistance`；暂停能力用 `SetEnabled`，保留原版投射物与 SG 协作。[配置](../../../raw/current/scripts/components/catcher.lua#L26-L44)
- 重要副作用：维护宿主 `cancatch` 标签；接取时依次向宿主发 `catch`、向投射物发 `caught`，再调用投射物 `Catch` 并停止监视。物品最终如何交给角色由投射物的 `oncaught` 决定。[标签](../../../raw/current/scripts/components/catcher.lua#L1-L7)、[接取顺序](../../../raw/current/scripts/components/catcher.lua#L77-L86)、[投射物收尾](../../../raw/current/scripts/components/projectile.lua#L194-L203)
- 修改边界：原版在服务器运行实际逻辑，调用路径要求宿主有 SG；只加组件、标签或 `ACTIONS.CATCH` 都不足以完成接取。[监视入口](../../../raw/current/scripts/components/projectile.lua#L160-L192)、[玩家状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10721-L10747)

## 关键接口

| 接口／字段 | 签名或默认值 | 条件、作用与证据 |
|---|---|---|
| `SetEnabled(enable)` | 布尔；初始 `true` | 禁用立即令 `canact=false`，若处于 `readytocatch` 则发 `cancelcatch`；不清空监视表，也不停更新。重新启用后由更新重算。[实现](../../../raw/current/scripts/components/catcher.lua#L26-L34)、[禁用分支](../../../raw/current/scripts/components/catcher.lua#L72-L89) |
| `SetActionDistance(dist)` | 初始 `12`；世界距离 | 控制动作可用范围，不使角色自动进入准备姿态；准备状态分支使用距离平方严格 `<` 比较。[默认](../../../raw/current/scripts/components/catcher.lua#L9-L20)、[设置](../../../raw/current/scripts/components/catcher.lua#L36-L44)、[比较](../../../raw/current/scripts/components/catcher.lua#L77-L89) |
| `SetCatchDistance(dist)` | 初始 `2`；世界距离 | 仅准备状态生效；距离平方 `<= dist²` 才实际接取。源码不校验参数，使用非负数值。[设置](../../../raw/current/scripts/components/catcher.lua#L41-L44)、[比较](../../../raw/current/scripts/components/catcher.lua#L77-L86) |
| `StartWatching(projectile)` / `StopWatching(projectile)` | 投射物实体 | 以实体为键登记／删除；开始监视会启动组件更新，表清空时停止更新。不是全场景扫描器。[实现](../../../raw/current/scripts/components/catcher.lua#L46-L58) |
| `CanCatch()` | 返回布尔 | 返回监视表非空且缓存的 `canact`；不表示已经接到，也不独立重算距离。[实现](../../../raw/current/scripts/components/catcher.lua#L60-L62) |

### 从动作提示到真正接取

1. `projectile:Throw` 在 `cancatch=true` 且目标具有 `catcher` 时调用目标的 `StartWatching`。默认投射物不可接；仅改 `catcher` 距离不会改变投射物资格。[默认](../../../raw/current/scripts/components/projectile.lua#L11-L26)、[开启资格](../../../raw/current/scripts/components/projectile.lua#L140-L142)、[投掷入口](../../../raw/current/scripts/components/projectile.lua#L160-L192)
2. `OnUpdate` 删除无效、已无 `projectile` 或已结束投掷的对象；有效对象在启用状态下参与距离判断，最终更新 `canact` 与 `cancatch` 标签。[完整更新](../../../raw/current/scripts/components/catcher.lua#L64-L93)
3. 动作收集读取接取者 `cancatch`；投射物目标上的动作还要求其 `catchable`。`ACTIONS.CATCH.fn` 本身只返回 `true`，SG 才负责进入准备姿态；玩家动作处理器将它映射到 `catch_pre`。[宿主动作](../../../raw/current/scripts/componentactions.lua#L248-L252)、[目标动作](../../../raw/current/scripts/componentactions.lua#L687-L691)、[动作函数](../../../raw/current/scripts/actions.lua#L1880-L1882)、[SG 映射](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1432)
4. 玩家 `catch_pre` 带 `readytocatch`，最多等 3 秒；收到 `catch` 进入接取动画，收到 `cancelcatch` 或超时返回 idle。真正交接来自组件更新及 `projectile:Catch`，后者停止投射物和物理运动，再执行 `oncaught`。[准备状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10721-L10747)、[接取状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10750-L10771)、[Catch](../../../raw/current/scripts/components/projectile.lua#L194-L203)

## 典型接入

### 扩大原版玩家接回旋镖的容错距离

前提：已启用的世界模组，在根 `modmain.lua` 放下列完整内容；无需新 Prefab 或美术资源。使用原版玩家已有的 `catcher`，只调整两个距离，保留原有启用状态、动作、SG 和投射物回调。`AddPlayerPostInit` 的玩家筛选来自原版注册入口。[注册入口](../../../raw/current/scripts/modutil.lua#L585-L590)

```lua
AddPlayerPostInit(function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local catcher = inst.components.catcher
    if catcher ~= nil then
        catcher:SetActionDistance(16)
        catcher:SetCatchDistance(3)
    end
end)
```

- 生效：此模组加载后构造的玩家，动作提示距离由 12 改为 16，准备状态的接取距离由 2 改为 3，单位均为世界距离。仍需玩家执行原版接取动作，仍只处理被登记的投射物；没有将禁用的接取能力强制开启。[默认与设置](../../../raw/current/scripts/components/catcher.lua#L9-L44)、[距离消费](../../../raw/current/scripts/components/catcher.lua#L72-L89)
- 原版参照：注册名 `boomerang` 的工厂在主机分支添加 `projectile`，开启可接取并配置 `OnCaught`；回程通过 `Throw(owner, owner)` 将原持有者登记为接取目标。接到后只有背包存在且 `isopen` 时才尝试装备或放入背包。[注册与配置](../../../raw/current/scripts/prefabs/boomerang.lua#L134-L155)、[主机门禁](../../../raw/current/scripts/prefabs/boomerang.lua#L115-L123)、[回程与交接](../../../raw/current/scripts/prefabs/boomerang.lua#L47-L63)

  （26148036 核验：boomerang 的 projectile 新设 `SetKeepOnDeflect(true)`，被格挡偏转后弹体保留、仍可继续接取回收——[boomerang.lua:137](../../../raw/current/scripts/prefabs/boomerang.lua#L137）
- 撤销：删除示例并重新加载世界／重建玩家；当前实体若需要立即恢复，可由原修改入口恢复原先记录的两个值。不要一律强制恢复默认值覆盖其他模组配置。组件无 `OnSave`／`OnLoad`，这些自定义距离需在构造入口重设。[组件全文](../../../raw/current/scripts/components/catcher.lua#L1-L95)

## 常见陷阱与限制

- **有提示仍接不到：** `cancatch` 是动作可用提示；只有 SG 带 `readytocatch` 且进入接取半径才会执行交接。自定义角色必须提供匹配状态与事件消费，不能仅复制玩家组件。[更新条件](../../../raw/current/scripts/components/catcher.lua#L72-L89)、[玩家状态协议](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10721-L10747)
- **禁用后监视表仍在：** `SetEnabled(false)` 只取消能力和准备姿态；有效的飞行物仍保留、无效对象仍在更新中清理。需要主动解除某个登记时用 `StopWatching`。[禁用](../../../raw/current/scripts/components/catcher.lua#L26-L34)、[停止监视](../../../raw/current/scripts/components/catcher.lua#L52-L58)、[更新](../../../raw/current/scripts/components/catcher.lua#L72-L89)
- **移除组件不会自动取消准备姿态：** 自身 `OnRemoveFromEntity` 只移除 `cancatch` 标签，没有发 `cancelcatch` 或持久化监视表；有即时撤销需求时先按宿主 SG 协议处理。[移除钩子](../../../raw/current/scripts/components/catcher.lua#L22-L34)
- **接取不是通用入包：** `projectile:Catch` 只停止飞行并执行 `oncaught`；物品归属、装备和资源处理必须由相应回调完成。回旋镖是专门实现，不能据此假设所有投射物都会入包。[Catch](../../../raw/current/scripts/components/projectile.lua#L194-L203)、[回旋镖回调](../../../raw/current/scripts/prefabs/boomerang.lua#L47-L56)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件对照 build-26148036 更新报告逐条核验，锚点重定位 12 处、结论修订 1 处（boomerang 偏转保留与宿主锚点，见正文"26148036 核验"标注）；未覆盖：运行验证。

- 2026-10-22，`steam-buildid:25643519`：源码审阅覆盖组件全部实现、玩家服务端宿主、动作收集与 SG 消费、投射物监视／接取及回旋镖注册和回调；示例只修改原有距离，不替换原行为链。
- 完整 `modmain.lua` 示例通过现有解析器的 Lua 5.1 语法检查；本页通过定向页面格式／本地链接与严格源码行号检查。
- 未运行游戏；未验证联机延迟下的接取手感、自定义角色 SG 或其他模组覆盖后的行为。
