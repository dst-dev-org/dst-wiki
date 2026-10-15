---
title: "terraformerremoveable"
created: "2026-10-22"
updated: "2026-10-28"
type: component
summary: "给目标提供草叉移除入口，由回调负责返还物品、删除实体和宿主清理。"
aliases: ["terraformerremoveable", "草叉移除目标", "拆除高尔夫球场", "可铲除实体"]
tags: [component, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/terraformerremoveable.lua", "componentactions.lua", "actions.lua", "prefabs/carnivalgame_golfgame.lua", "prefabs/carnivalgame_common.lua", "prefabs/pitchfork.lua", "components/finiteuses.lua", "bufferedaction.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# terraformerremoveable

## 快速判断

- 职责与适用：挂在被草叉移除的**目标**上，构造添加同名标签、接受移除回调；不负责地图铲地。`removeable` 是原版拼写，组件名、标签、API 都保持此写法。[完整组件](../../../raw/current/scripts/components/terraformerremoveable.lua#L1-L24)。
- 优先使用：有明确拆卸与返还逻辑的目标用 `SetOnRemovedFn`；工具仍需 [[reference/components/terraformer]]。原版高尔夫球场用它拆卸并返还套件。[动作执行](../../../raw/current/scripts/actions.lua#L7105-L7111)、[球场移除回调](../../../raw/current/scripts/prefabs/carnivalgame_golfgame.lua#L619-L654)。
- 重要副作用：`TryToRemove` 只调用可选回调并始终返回 true；删除实体、掉落和清理都由回调／宿主完成。[调用实现](../../../raw/current/scripts/components/terraformerremoveable.lua#L16-L22)。
- 修改边界：原版目标在 pristine 之前预加标签，服务端再添加组件；运行中移除组件会清除标签。不要只改客户端标签来模拟服务端资格。[球场构造](../../../raw/current/scripts/prefabs/carnivalgame_golfgame.lua#L782-L829)、[组件移除钩子](../../../raw/current/scripts/components/terraformerremoveable.lua#L8-L10)。

## 关键接口

| 接口／字段 | 签名或已核验约定 | 条件、作用与证据 |
|---|---|---|
| `SetOnRemovedFn(fn)` | `fn(inst, doer)`；nil 可清空 | 直接赋值 `onremovedfn`；回调返回值忽略。[实现](../../../raw/current/scripts/components/terraformerremoveable.lua#L12-L22) |
| `TryToRemove(doer)` | 返回 true | 有回调才调用；没有回调、回调返回 false 均不会使动作失败；不会自行 `inst:Remove()`。[实现](../../../raw/current/scripts/components/terraformerremoveable.lua#L16-L22) |
| `OnRemoveFromEntity()` | 宿主移除组件钩子 | 仅移除 `terraformerremoveable` 标签，不执行移除回调；`RemoveComponent` 会分派此钩子并注销组件动作。[组件](../../../raw/current/scripts/components/terraformerremoveable.lua#L8-L10)、[宿主分派](../../../raw/current/scripts/entityscript.lua#L670-L684) |

### 从工具右键到目标回调

装备工具的 `terraformer` 动作收集器在右键、目标有 `terraformerremoveable` 标签时加入 `TERRAFORM_REMOVE`；服务端动作还要求工具有 terraformer 且其 `plow` 字段不为真、目标确实有本组件，才调用目标 `TryToRemove(doer)`。[动作收集](../../../raw/current/scripts/componentactions.lua#L2500-L2504)、[动作执行](../../../raw/current/scripts/actions.lua#L7105-L7111)。候选提示使用标签，最终执行使用组件，不能把“显示按钮”当成必定成功。

动作的自定义到达判定在目标提供 `IsInGolfArea` 时检查执行者是否进入球场区域；没有此方法时返回 true。它是此动作专有逻辑，不是组件内的一般距离检查。[到达判定](../../../raw/current/scripts/actions.lua#L132-L139)、[动作定义](../../../raw/current/scripts/actions.lua#L740)。

成功的 BufferedAction 才调用工具 `OnUsedAsItem`；草叉将 TERRAFORM 配置为 `.125`，finiteuses 同步映射 TERRAFORM_REMOVE，所以“移除目标”也可能耗耐久。直接 `TryToRemove` 不经过这个动作消费入口。[成功动作](../../../raw/current/scripts/bufferedaction.lua#L22-L37)、[草叉配置](../../../raw/current/scripts/prefabs/pitchfork.lua#L63-L79)、[映射](../../../raw/current/scripts/components/finiteuses.lua#L27-L34)。

## 典型接入

### 原版高尔夫球场：空闲可拆，比赛中禁拆

这是已核验的原版接入说明，适合复用球场生命周期；球场依赖课程、套件、附属物和小游戏状态，不把单独添加组件称作完整独立 demo。

1. 注册名 `carnivalgame_golfgame` 对应 `fn`；构造在 pristine 前加入目标标签，服务端绑定 `AddTerraformerRemoveable`／`RemoveTerraformerRemoveable` 并立即添加组件。添加函数每次取得或新建组件并设置原版 `OnTerraformerRemoved`。[注册](../../../raw/current/scripts/prefabs/carnivalgame_golfgame.lua#L1025-L1031)、[pristine 标签](../../../raw/current/scripts/prefabs/carnivalgame_golfgame.lua#L782-L802)、[服务端入口](../../../raw/current/scripts/prefabs/carnivalgame_golfgame.lua#L810-L831)、[辅助函数](../../../raw/current/scripts/prefabs/carnivalgame_golfgame.lua#L651-L658)。
2. `OnStartPlaying` 移除组件，`OnStopPlaying` 再添加并重装回调。这样禁用阶段同时失去标签和服务端执行入口；恢复阶段重新接回原版拆卸行为。[开始／停止](../../../raw/current/scripts/prefabs/carnivalgame_golfgame.lua#L302-L325)。扩展球场时保留这两个生命周期入口，不用返回 false 的移除回调代替禁拆。
3. 回调用 `GetKitPrefab` 选择 DIY 或难度套件，生成并处理课程候选数据后 `Launch2`，最后明确 `inst:Remove()`；不是组件自动返还套件。[套件选择与回调](../../../raw/current/scripts/prefabs/carnivalgame_golfgame.lua#L610-L649)。
4. 删除由宿主继续清理：公共 `OnRemoveEntity` 执行 ShutdownGame 后调用球场 `OnRemoveGame`，后者删除 courseparts；`onremove` 监听另外清理独立视觉物与球场物理体。[公共钩子](../../../raw/current/scripts/prefabs/carnivalgame_common.lua#L204-L207)、[钩子安装](../../../raw/current/scripts/prefabs/carnivalgame_common.lua#L222-L233)、[课程清理](../../../raw/current/scripts/prefabs/carnivalgame_golfgame.lua#L364-L369)、[视觉物监听](../../../raw/current/scripts/prefabs/carnivalgame_golfgame.lua#L881-L886)、[物理体清理及监听](../../../raw/current/scripts/prefabs/carnivalgame_golfgame.lua#L723-L739)、[监听安装](../../../raw/current/scripts/prefabs/carnivalgame_golfgame.lua#L804-L808)。

组件没有 OnSave／OnLoad；回调随服务端工厂重新安装。球场另存课程字符串、customizable 与 difficulty，并在 OnLoadPostPass 恢复；不要把回调函数本身写入存档，也不要据此推断组件保存了比赛中禁拆状态。[完整组件](../../../raw/current/scripts/components/terraformerremoveable.lua#L1-L24)、[球场存档](../../../raw/current/scripts/prefabs/carnivalgame_golfgame.lua#L742-L763)。

## 常见陷阱与限制

- **加标签就能拆**：只满足动作提示，服务端还要求目标组件与非 plow 的工具组件；为实际目标同时准备端边界、回调和资源。[动作两阶段](../../../raw/current/scripts/componentactions.lua#L2500-L2504)、[服务端检查](../../../raw/current/scripts/actions.lua#L7105-L7111)。
- **回调返回 false 阻止拆卸／扣耐久**：返回值被忽略，TryToRemove 恒 true；无回调也会报动作成功。需在不允许拆卸的阶段移除组件，并在允许阶段恢复正确回调。[组件实现](../../../raw/current/scripts/components/terraformerremoveable.lua#L12-L22)、[原版阶段切换](../../../raw/current/scripts/prefabs/carnivalgame_golfgame.lua#L302-L325)。
- **RemoveComponent 等于删除目标**：它只执行组件移除清理，不调用 `onremovedfn`；删除实体要由移除回调明确负责，附属物清理另交宿主。[宿主移除组件](../../../raw/current/scripts/entityscript.lua#L670-L684)、[原版删除](../../../raw/current/scripts/prefabs/carnivalgame_golfgame.lua#L647-L658)。
- **把它用于铲掉地图地皮**：本动作只委派目标回调，未调用 Terraform 或 SetTile。地图地块与土堆坍塌的说明见 [[reference/components/terraformer]]；本组件也没有通用战利品或权限判断。

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对本页 sources 变更文件 actions.lua／componentactions.lua（R-01/R-04）、entityscript.lua（R-08）：TERRAFORM_REMOVE 定义仍在 actions.lua:740（本次仅删其 fn 前 1 空行），候选与 RemoveComponent 分派未变，结论无修订；锚点重定位 5 处（fn 后移至 7095、候选后移至 2488）；未覆盖：运行验证。

- 源码审阅：2026-10-22，Steam buildid 25643519；覆盖完整组件、标签／组件移除钩子、EQUIPPED 动作与到达判定、草叉耗次映射、球场注册／构造／阶段切换／套件返还／附属清理与保存分工。
- 结构检查：本次两页定向页面校验与源码锚点严格校验通过。本页采用原版接入说明，没有新增完整 Lua 示例。
- 游戏运行：未运行；未验证联机提示、球场多阶段切换、课程返还与重载的实际效果，不扩大为球场全机制审校。

相关：[[reference/components/terraformer]]；[[reference/components/finiteuses]]。
