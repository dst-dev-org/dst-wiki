---
title: "slingshotmods"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "沃尔特弹弓的模块化改装接口：在弹弓内隐藏挂载 slingshotmodscontainer 容器装弹弓带／架／柄零件，经弹弓改装套件动作开关，按技能与归属校验并随弹弓存档。"
aliases: ["slingshotmods", "弹弓改装", "slingshot 改装容器", "MODSLINGSHOT"]
tags: [component, item, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/slingshotmods.lua", "prefabs/slingshot.lua", "components/slingshotmodder.lua", "actions.lua", "containers.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# slingshotmods

## 快速判断

- 职责与适用：挂在弹弓 prefab 上的双端组件（`SetPristine` 之前添加）；服务端在弹弓下挂一个 `slingshotmodscontainer` 隐藏容器（3 槽：带 1／架 2／柄 3），管理开关、零件迁移与存档；客户端只用 `CanBeOpenedBy` 做动作校验。[挂载](../../../raw/current/scripts/prefabs/slingshot.lua#L503 [容器创建](../../../raw/current/scripts/components/slingshotmods.lua#L91-L105)
- 优先使用：改装入口走 [[reference/components/slingshotmodder]]（改装套件上的组件）→ `Open`／`Close`；查询外观件 → `GetPartBuildAndSymbol("band"|"frame"|"handle")`。[动作链](../../../raw/current/scripts/actions.lua#L6419-L6437)
- 重要副作用：打开时把容器的 `Network:SetClassifiedTarget` 指向开启者（客户端才看得到内容）；`ondropped` 自动关闭；每帧检查开启者 SG 是否仍带 `moddingslingshot` 状态，丢失即关闭并推 `ms_slingshotmodsclosed`。[关闭与事件](../../../raw/current/scripts/components/slingshotmods.lua#L167-L188)
- 修改边界：全部服务端方法以 `self.ismastersim` 早退；零件迁移 `TransferPartsTo` 只在"对方尚无容器"时整体搬移，不做零件合并。[迁移](../../../raw/current/scripts/components/slingshotmods.lua#L111-L136)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `CanBeOpenedBy(doer)` | 返回 boolean（双端） | 需要点亮技能 `walter_slingshot_modding` 且弹弓 `replica.inventoryitem:IsGrandOwner(doer)`；改装动作与客户端动作过滤共用。([slingshotmods.lua:10-23](../../../raw/current/scripts/components/slingshotmods.lua#L10-L23) [技能注册](../../../raw/current/scripts/prefabs/skilltree_walter.lua#L36 |
| `Open(opener)` | 返回 boolean（服务端） | 容器不存在则创建；要求 `container:CanOpen()` 且弹弓 `inventoryitem:GetGrandOwner() == opener`；首次开启 `StartUpdatingComponent` 监听开启者状态。([slingshotmods.lua:138-165](../../../raw/current/scripts/components/slingshotmods.lua#L138-L165) |
| `Close(opener)` | 返回 boolean（服务端） | 容器不存在或传入非当前开启者时返回 false；关闭后 classified 目标归还容器并推 `ms_slingshotmodsclosed`。([slingshotmods.lua:167-182](../../../raw/current/scripts/components/slingshotmods.lua#L167-L182) |
| `GetPartBuildAndSymbol(slot)` | `("band"|"frame"|"handle")` → `{prefab, swap_build, swap_symbol}` 或 nil | 从容器对应槽（1/2/3）读零件的外观三元组，供客户端换装逻辑取用。([slingshotmods.lua:52-69](../../../raw/current/scripts/components/slingshotmods.lua#L52-L69) |
| `HasPartName(name)`／`HasAnyParts()` | 返回 boolean（服务端） | 经容器 `Has`／`IsEmpty` 查询装配状态；无容器一律 false。([slingshotmods.lua:35-50](../../../raw/current/scripts/components/slingshotmods.lua#L35-L50) |
| `CheckRequiredSkillsForPlayer(player)` | 返回 boolean（服务端） | 任一零件带 `REQUIRED_SKILL` 且玩家未点亮该技能即 false，供弹弓能否正常使用判定。([slingshotmods.lua:71-85](../../../raw/current/scripts/components/slingshotmods.lua#L71-L85) |
| `TransferPartsTo(other)` | other 为另一弹弓的 slingshotmods | 整体搬移容器实体并重设父子与 `install_target`，对每个零件调 `containerinstallableitem:OnInstalled`；开启状态一并移交。([slingshotmods.lua:111-136](../../../raw/current/scripts/components/slingshotmods.lua#L111-L136) |
| `OnSave`／`OnLoad` | 存档钩子 | 容器非空时持久化整个容器的 `GetPersistData`（含零件引用），载入时重建容器。([slingshotmods.lua:207-221](../../../raw/current/scripts/components/slingshotmods.lua#L207-L221) |

## 典型接入

### 为自定义弹弓复用改装体系

- 前提与选择：自定义弹弓需在双端 `SetPristine` 前加组件（客户端动作过滤读它），并复用 `slingshotmodscontainer` 的容器布局（([containers.lua:1544-1580](../../../raw/current/scripts/containers.lua#L1544-L1580) 的 `itemtestfn` 只收弹弓零件）。
- 生效与撤销：玩家持改装套件对弹弓发起 `ACTIONS.MODSLINGSHOT`（目标优先取手装备，其次背包内其他弹弓，([actions.lua:6419-6437](../../../raw/current/scripts/actions.lua#L6419-L6437)）；关闭后零件留在隐藏容器内随弹弓存档。
- 原版依据：套件侧入口 [[reference/components/slingshotmodder]]（([slingshotmodder.lua:5-13](../../../raw/current/scripts/components/slingshotmodder.lua#L5-L13)），SG 状态 `openslingshotmods` 由 `slingshotmodkit` tag 触发。

```lua
-- 自定义弹弓 prefab fn 内，SetPristine 之前（双端）：
inst:AddComponent("slingshotmods")
-- 服务端如需换装即时生效，读取外观三元组：
-- local prefab, build, symbol = inst.components.slingshotmods:GetPartBuildAndSymbol("band")
-- 若要求零件技能校验纳入可用性判定：
-- local ok = inst.components.slingshotmods:CheckRequiredSkillsForPlayer(owner)
```

- 调整与验证：`Open` 只认 `GetGrandOwner() == opener`，放进箱子/他人背包的弹弓无法被打开改装；容器槽位顺序固定为 带=1、架=2、柄=3。

## 常见陷阱与限制

- 组件加在 `SetPristine` 之前意味着客户端也持有实例，但除 `CanBeOpenedBy` 外的方法在客户端一律早退（([slingshotmods.lua:28-33](../../../raw/current/scripts/components/slingshotmods.lua#L28-L33) 等逐个 `ismastersim` 守卫）；直接在客户端读容器数据会拿到 nil。
- 开启是状态绑定：`OnUpdate` 只认 SG tag `moddingslingshot`（[L184-188](../../../raw/current/scripts/components/slingshotmods.lua#L184-L188)），被传送、打断或 SG 离开该状态会立即关闭并触发 `ms_slingshotmodsclosed`；监听关闭事件要挂在开启者身上。
- `DropAllPartsWithoutUninstalling` 用 `containerinstallableitem.ignoreuninstall` 临时跳过卸载副作用（[L190-205](../../../raw/current/scripts/components/slingshotmods.lua#L190-L205)），用 `container:DropItem` 手动丢零件时不走这个保护，零件会正常触发卸载逻辑。
- 零件迁移不合并：`TransferPartsTo` 要求 `other.containerinst == nil`，两个已装零件的弹弓之间不发生迁移（[L114-114](../../../raw/current/scripts/components/slingshotmods.lua#L114-L121)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全文；核验 slingshot.lua 挂载与 pristine 顺序、slingshotmodder 组件、MODSLINGSHOT／STOPMODSLINGSHOT 动作、containers.lua 容器参数与技能名。
- 游戏运行：未运行。
- 结构检查：未执行。

相关链接：[[reference/components/slingshotmodder]]、[[reference/components/container]]、[[reference/components/containerinstallableitem]]、[[reference/components/inventoryitem]]
