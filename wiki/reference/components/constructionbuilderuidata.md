---
title: "constructionbuilderuidata"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "用两个 net_entity 把玩家当前的施工容器与施工站点从服务端桥接到客户端 UI。"
aliases: ["施工 UI 数据", "客户端施工状态", "施工容器桥接", "constructionbuilderuidata"]
tags: [component, data, ui, server-client, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/constructionbuilderuidata.lua", "prefabs/player_common.lua", "components/constructionbuilder.lua", "entityreplica.lua", "netvars.lua", "actions.lua", "componentactions.lua", "containers.lua", "components/container.lua", "components/inventory.lua", "widgets/containerwidget.lua", "widgets/invslot.lua", "prefabs/container_classified.lua", "prefabs/inventory_classified.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# constructionbuilderuidata

## 快速判断

- 职责与适用：玩家实体上的客户端桥接组件，只保存两个实体引用——当前施工容器与当前施工站点——让客户端在缺少 `constructionbuilder_replica` 时仍能画施工槽位、判断“材料该放进哪个容器”。[构造](../../../raw/current/scripts/components/constructionbuilderuidata.lua#L5-L10) [存在理由](../../../raw/current/scripts/components/constructionbuilder.lua#L1-L2)
- 优先使用：客户端读 `GetContainer`／`GetTarget`／`GetConstructionSite`／`GetIngredientForSlot`／`GetSlotForIngredient`；需要写入时改 [[reference/components/constructionbuilder]] 的 `constructioninst`／`constructionsite` 字段，不要直接调 Set。
- 重要副作用：两个 net_entity 由服务端写入并复制给客户端；组件没有存档逻辑，读档后只恢复容器引用，站点引用为空。
- 修改边界：唯一宿主是 `player_common` 的公共构造段（两端都有实例）；服务端唯一写入者是 constructionbuilder；不承担材料计数、完成判定或持久化。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `_containerinst`／`_targetinst` | `net_entity` | 构造时各自声明一次；两端必须用相同顺序与名字声明，否则实体反序列化失败。[L8-L9](../../../raw/current/scripts/components/constructionbuilderuidata.lua#L8-L9) [netvar 约束](../../../raw/current/scripts/netvars.lua#L29-L32) |
| `SetContainer(inst)`／`GetContainer()` | 实体／实体或 nil | 只写／读 net_entity，不判空、不触发容器打开。[L12-L18](../../../raw/current/scripts/components/constructionbuilderuidata.lua#L12-L18) |
| `SetTarget(inst)`／`GetTarget()` | 实体／实体或 nil | 同上；目标是站点实体本身。[L20-L26](../../../raw/current/scripts/components/constructionbuilderuidata.lua#L20-L26) |
| `GetConstructionSite()` | 站点 replica 或 nil | 目标非空时返回 `target.replica.constructionsite`；目标没有该 replica 时恒为 nil。[L28-L30](../../../raw/current/scripts/components/constructionbuilderuidata.lua#L28-L30) |
| `GetIngredientForSlot(slot)` | 材料 prefab 名或 nil | 返回 `CONSTRUCTION_PLANS[目标prefab][slot].type`；目标为空、plan 缺失或槽越界都返回 nil，不报错。[L32-L34](../../../raw/current/scripts/components/constructionbuilderuidata.lua#L32-L34) |
| `GetSlotForIngredient(prefab)` | 1-based 槽号或 nil | 在目标 prefab 的 plan 中线性查找 `type`，找不到返回 nil。[L36-L44](../../../raw/current/scripts/components/constructionbuilderuidata.lua#L36-L44) |
| `EMPTY_TABLE` | 空表常量 | `GetIngredientForSlot` 在目标为空或槽缺失时用它兜底，避免索引 nil。[L3](../../../raw/current/scripts/components/constructionbuilderuidata.lua#L3) |

### 谁写、谁读

- 写：只有 constructionbuilder 的属性访问器——`constructioninst`／`constructionsite` 赋值时同步两个 net_entity；开始、停止、载入都会触发。[setter](../../../raw/current/scripts/components/constructionbuilder.lua#L4-L10) [属性表](../../../raw/current/scripts/components/constructionbuilder.lua#L24-L25)
- 读（动作与服务端容器）：STORE 的目标槽选择、BUNDLESTORE 的 `CONSTRUCT` 文案、施工容器的 `itemtestfn`、容器/背包转移时的槽位判断。[动作](../../../raw/current/scripts/actions.lua#L2535-L2550) [文案](../../../raw/current/scripts/actions.lua#L2601-L2609) [槽校验](../../../raw/current/scripts/containers.lua#L523-L529) [转移](../../../raw/current/scripts/components/container.lua#L1348-L1355)
- 读（客户端 UI）：容器窗口决定施工槽背景与按钮、背包槽把材料优先投进施工容器、classified 容器与背包同步槽位。[窗口](../../../raw/current/scripts/widgets/containerwidget.lua#L131-L132) [背包槽](../../../raw/current/scripts/widgets/invslot.lua#L207-L215) [classified 容器](../../../raw/current/scripts/prefabs/container_classified.lua#L758-L759) [classified 背包](../../../raw/current/scripts/prefabs/inventory_classified.lua#L1123-L1124)
- 没有 replica：`REPLICATABLE_COMPONENTS` 中没有 constructionbuilder，本组件正是它的替代品。[复制表](../../../raw/current/scripts/entityreplica.lua#L5-L26)

## 典型接入

### 原版唯一接入：player_common 的构造顺序

- 文件与端：`prefabs/player_common.lua` 中 `MakePlayerCharacter` 返回的 `fn()`；`AddComponent("constructionbuilderuidata")` 在 `SetPristine` 之前、服务端提前返回之前，所以每个角色预制的服务端与客户端都有实例。[添加](../../../raw/current/scripts/prefabs/player_common.lua#L2512) [pristine 与分支](../../../raw/current/scripts/prefabs/player_common.lua#L2611-L2613)
- 为什么在公共段：两个 net_entity 必须在两端按相同位置声明，否则实体反序列化失败；客户端要读服务端写入的值，因此不能只在 master 段添加。[netvar 约束](../../../raw/current/scripts/netvars.lua#L29-L32)
- 服务端写入者：`constructionbuilder` 的构造器里 `self.constructioninst = nil` 就会触发属性 setter，setter 直接索引 `inst.components.constructionbuilderuidata`；所以 uidata 必须先添加（原版放在更早的公共段），顺序反了在建组件的构造期就会 nil 索引。[builder](../../../raw/current/scripts/prefabs/player_common.lua#L2721) [setter](../../../raw/current/scripts/components/constructionbuilder.lua#L4-L10)
- 生命周期：玩家 despawn 前 `constructionbuilder:StopConstruction` 把两个字段置空并同步；读档时 `constructionbuilder:OnLoad` 只恢复容器（`constructioninst`），`constructionsite` 被显式置 nil，所以客户端在读档后 `GetTarget`／`GetConstructionSite` 为空，直到再次开始施工。[despawn](../../../raw/current/scripts/prefabs/player_common.lua#L1437) [载入](../../../raw/current/scripts/components/constructionbuilder.lua#L158-L176)
- 单位：两个字段都是实体引用，不涉及数量或时长；槽号是 1-based，与容器的 `GetItemInSlot`／`CanTakeItemInSlot` 槽号一致。

## 常见陷阱与限制

- **客户端没有 constructionbuilder**：任何客户端代码都应经 uidata；`inst.components.constructionbuilder` 在客户端不存在。这也是本组件存在的原因。[注释](../../../raw/current/scripts/components/constructionbuilder.lua#L1-L2)
- **不做存档**：组件没有 `OnSave`／`OnLoad`，值只存在于实体网络状态；不要把它当权威进度，读档或网络滞后时所有 getter 都可能返回 nil。
- **`GetConstructionSite` 依赖 replica**：目标是普通实体但没有 constructionsite replica（服务端未添加组件或未走分类复制）时恒为 nil；取到后仍要判空。[L28-L30](../../../raw/current/scripts/components/constructionbuilderuidata.lua#L28-L30)
- **plan 缺失静默返回 nil**：两个查表接口都以 `CONSTRUCTION_PLANS` 为准，站点 prefab 没有 plan 时返回 nil，UI 表现为没有可放槽位；这与 [[reference/components/constructionsite]] 的空 plan 行为一致，不是 uidata 的错。[L32-L44](../../../raw/current/scripts/components/constructionbuilderuidata.lua#L32-L44)
- **服务端也读它**：`containers.construction_container.itemtestfn` 通过容器父实体找 doer，再读 uidata；施工容器必须被 parent 到玩家（constructionbuilder 打开后 `SetParent`），否则槽校验一律失败。[itemtestfn](../../../raw/current/scripts/containers.lua#L523-L529) [parent](../../../raw/current/scripts/components/constructionbuilder.lua#L60-L65)
- **每玩家一份**：不要挂在容器或站点实体上；多人同时施工时各自的 uidata 指向各自的容器。
- **直接调 Set 会绕过流程**：`SetContainer`／`SetTarget` 只写 netvar，不打开容器、不绑定站点、不改状态机；这些副作用都在 constructionbuilder 里。

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 漂移文件 prefabs/player_common.lua、components/constructionbuilder.lua、actions.lua、componentactions.lua、containers.lua 对照 build-26148036 更新报告（R-01、R-04、constructionbuilder 内部重构条目 detail-components.md:19）逐条核验，锚点重定位 5 处、结论修订 0 处（player_common 漂移仅删 `_wormdigestionsound` 本地音效接线、constructionbuilder 仅 stopconstruction 改同步派发、componentactions 新处理器与 containers 新按钮均不触及本页读写链）；未覆盖：运行验证。

- 源码审阅：2026-10-21，Steam buildid 25643519；通读组件（46 行），核验 48 处引用、12 个文件中的读写方，唯一宿主 `player_common`（L2522）与唯一写入者 `constructionbuilder`（L4-L10）；确认 `REPLICATABLE_COMPONENTS` 中没有 constructionbuilder。
- 游戏运行：未运行；读档后的 UI 槽位表现与多人同时施工的客户端刷新均未实测。
- 结构检查：`validate_page.py` 通过（0 issue；本页 68 行／9539 字节）；`validate_source_links.py --strict` 共 33 条源码引用、0 error／0 warning；本页无 Lua 示例，未生成临时 `.lua`。
