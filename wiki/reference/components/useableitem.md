---
title: "useableitem"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "为已装备物品接入一次开始／停止使用，明确 inuse、回调返回与收尾责任。"
aliases: ["已装备物品右键", "物品主动使用", "开始停止使用", "使用状态", "useableitem"]
tags: ["component", "equipment", "callback", "lifecycle"]
dst_build: "steam-buildid:26230496"
sources: ["actions.lua", "componentactions.lua", "components/useableitem.lua", "prefabs/armor_snurtleshell.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# useableitem

## 快速判断

- 职责与适用：已装备物品的 USEITEM 开始／停止状态；不是任意背包道具的通用右键开关。[components/useableitem.lua:33](../../../raw/current/scripts/components/useableitem.lua#L33 [componentactions.lua:3054](../../../raw/current/scripts/componentactions.lua#L3054
- 优先使用：SetOnUseFn／SetOnStopUseFn，并设计实际结束事件／卸装清理。[components/useableitem.lua:25](../../../raw/current/scripts/components/useableitem.lua#L25
- 重要副作用：onuse 只要不明确返回 false 就维持 inuse，后续交互被阻止；onuse 参数仅 inst。[components/useableitem.lua:37](../../../raw/current/scripts/components/useableitem.lua#L37
- 修改边界：默认动作检查装备、cannotuse 与 inventory 开启条件；组件不会自动安排持续任务或监听通用停止事件。[actions.lua:3306](../../../raw/current/scripts/actions.lua#L3306

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `CanInteract(doer)` | boolean | 只检查 not inuse 与 replica.equippable:IsEquipped，组件内未使用 doer。[components/useableitem.lua:33](../../../raw/current/scripts/components/useableitem.lua#L33 |
| `SetOnUseFn(fn)` | `(inst) -> false 或其他值` | false 取消使用状态；nil 会保持 inuse，不能当自动结束。[components/useableitem.lua:37](../../../raw/current/scripts/components/useableitem.lua#L37 |
| `SetOnStopUseFn(fn)`／`StopUsingItem()` | `(inst)`／结束状态 | Stop 每次都清 inuse 并调用回调，不保证只调用一次；清理应可重复。[components/useableitem.lua:49](../../../raw/current/scripts/components/useableitem.lua#L49 |
| `stopuseevents` | 可选 `(inst)` 函数 | StartUsingItem 在 onuse 后调用此函数，名字不是事件名数组，也不会自动监听。[components/useableitem.lua:43](../../../raw/current/scripts/components/useableitem.lua#L43 |

## 典型接入

### 缩壳甲的开始与结束协作

原版 armorsnurtleshell 添加 useableitem 并注册两个回调；onuse 从 inventoryitem.owner 找人物、进入 shell_enter 状态并启动一次任务；onstopuse 取消任务。装备期间监听人物 newstate，由 ProtectionLevels 在离开 shell 状态时 StopUsingItem；卸装另取消任务／监听。[prefabs/armor_snurtleshell.lua:39](../../../raw/current/scripts/prefabs/armor_snurtleshell.lua#L39 [prefabs/armor_snurtleshell.lua:10](../../../raw/current/scripts/prefabs/armor_snurtleshell.lua#L10 [prefabs/armor_snurtleshell.lua:71](../../../raw/current/scripts/prefabs/armor_snurtleshell.lua#L71 [prefabs/armor_snurtleshell.lua:116](../../../raw/current/scripts/prefabs/armor_snurtleshell.lua#L116

对自定义模式切换：若按一次完成、无持续状态，可让 onuse 完成操作后明确返回 false；若需要持续 inuse，必须把停止入口、任务、监听、角色状态和卸下／删除边界一起实现。不要复制缩壳 SG 状态给无对应状态的角色。

### 一次切换与持续使用

本地片段参照（未随库分发，未测游戏／外部 Hook；[版本与文件指纹](../../../docs/reports/w05/local-mod-references-20261017.md)）：MCW 的 mcw_lollipopstaff.lua:53～78、Sora 的 sora2sword.lua:53～66、DSTKisaki 的 kisaki_tool.lua:17～28 均在模式切换后返回 false，使组件不保持 inuse。这适合切动作／外观／名称的一次操作；缩壳等持续行为则保持状态并由匹配结束入口 StopUsingItem。回调的 false 控制 inuse，不代表模式变化必须回滚。[components/useableitem.lua:37](../../../raw/current/scripts/components/useableitem.lua#L37

只需名称变化时链接 [[reference/components/named]]，动作能力变化链接 [[reference/components/tool#多功能工具的模式切换]]；场景代码按这几个组件分工，不需要复制整个模组工厂。

## 常见陷阱与限制

- **回调拿第二个参数当使用者**：StartUsingItem 不向 onuse 传 doer，即使 ACTION 调用带了额外参数；从明确归属取 owner 并判空。[components/useableitem.lua:37](../../../raw/current/scripts/components/useableitem.lua#L37 [actions.lua:3319](../../../raw/current/scripts/actions.lua#L3319
- **只写 onuse 后永远点不动**：默认 nil 保持 inuse；一次性操作返回 false 或设计匹配 StopUsingItem。[components/useableitem.lua:40](../../../raw/current/scripts/components/useableitem.lua#L40
- **移组件触发 onstopuse**：OnRemoveFromEntity 只移 inuse 标签，不调业务清理；须明确停止并取消自己的任务／监听。[components/useableitem.lua:21](../../../raw/current/scripts/components/useableitem.lua#L21

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 中的漂移文件 actions.lua（R-01）与 componentactions.lua（R-04）逐 hunk 对照，锚点重定位 3 处（EQUIPPED 候选 2993→3042、USEITEM 动作门禁 3293→3296、StartUsingItem 调用行 3306→3309），结论修订 0 处——USEITEM.useableitem 动作分支、CanInteract／onuse 返回契约与候选均未变；未覆盖：运行验证。

- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：装备交互门禁、onuse 返回 false 与 inuse、停止／移组件责任及缩壳甲协作，依据见上文对应锚点；未交付新主动技能工程或任意角色缩壳 SG 适配。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；本页为机制与原版路径说明，未附新主动技能工程。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

相关：[[reference/components/equippable]]；[[reference/components/inventoryitem#生命周期]]；[[reference/components/spellcaster]]。
