---
title: "spellcaster"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "接入传统物品施法，区分自身、目标和地点施法及最终服务端验证。"
aliases: ["右键施法", "物品施法", "对目标施法", "地点施法", "spellcaster"]
tags: ["component", "item", "equipment", "callback", "server-client"]
dst_build: "steam-buildid:26230496"
sources: ["actions.lua", "componentactions.lua", "components/spellcaster.lua", "prefabs/staff.lua", "prefabs/staff_common.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# spellcaster

## 快速判断

- 职责与适用：传统 CASTSPELL 的能力检查和效果回调；轮盘选择另见 [[reference/components/spellbook]]。[actions.lua:3445](../../../raw/current/scripts/actions.lua#L3445
- 优先使用：SetSpellFn 配效果，按自身／目标／地点选择 canuse 标志；消耗和冷却由技能实现。[components/spellcaster.lua:118](../../../raw/current/scripts/components/spellcaster.lua#L118
- 重要副作用：标志／spell 改动维护动作标签；CastSpell 直接调用不会自查 CanCast；施法成功后物品实体还会推 `onspellcast` 事件（26148036 核验：C-40 纯新增，data 含 target/pos/doer）。[components/spellcaster.lua:1](../../../raw/current/scripts/components/spellcaster.lua#L1 [components/spellcaster.lua:134](../../../raw/current/scripts/components/spellcaster.lua#L134)、[:141](../../../raw/current/scripts/components/spellcaster.lua#L141)
- 修改边界：服务端最终调用；目标检查、无目标物品栏检查和地点检查是不同分支，SetCanCastFn 并非统一门禁；动作层另有 nomagiccast 物品门控与禁长按，见[常见陷阱](#常见陷阱与限制)。[components/spellcaster.lua:152](../../../raw/current/scripts/components/spellcaster.lua#L152

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `SetSpellFn(fn)`／`SetOnSpellCastFn(fn)` | `(item, target, pos, doer)` | 先效果再通知，回调返回值被忽略；SetOnSpellCastFn 不等于判定成功才通知。（26148036 核验：C-40 施法成功后物品实体另推事件 `onspellcast`（data 同 target/pos/doer），监听该事件与回调相互独立）[components/spellcaster.lua:134](../../../raw/current/scripts/components/spellcaster.lua#L134)、[:141](../../../raw/current/scripts/components/spellcaster.lua#L141) |
| `SetCanCastFn(fn)` | `(doer, target, pos, item)` | 与效果回调顺序不同；无目标且无 pos 的 inventory 分支不执行它。[components/spellcaster.lua:152](../../../raw/current/scripts/components/spellcaster.lua#L152 |
| `canusefrominventory` | 默认 false | 允许无目标且无位置的库存施法；实际执行仍需自查冷却／权限。[components/spellcaster.lua:159](../../../raw/current/scripts/components/spellcaster.lua#L159 |
| `canuseontargets`／`canonlyuseon*` | 默认 false | 目标能力要有指定过滤分支或自定义 predicate；内置过滤与 predicate 用 OR，不保证两者都通过。（26148036 核验：这组标志派生的 caston* 物品标签自本版起同时驱动 USEITEM 级动作筛选，见[典型接入](#不装备直接对目标施法useitem-级动作筛选)）[components/spellcaster.lua:190](../../../raw/current/scripts/components/spellcaster.lua#L190 |
| `canuseonpoint`／`canuseonpoint_water` | 默认 false | 选陆地／水域位置路径，并检查地点 predicate 和地图阻挡。[components/spellcaster.lua:170](../../../raw/current/scripts/components/spellcaster.lua#L170)、[:172](../../../raw/current/scripts/components/spellcaster.lua#L172) |

## 典型接入

### 原版法杖的施法模式

| 需求 | 原版参照 | 应复用的分工 |
|---|---|---|
| 自身／生物目标传送 | telestaff，紫色工厂 | canusefrominventory + locomotorspvp 过滤，效果函数自己处理次数和传送。（26148036 核验：P-112 六杖配置逻辑迁至 staff_common.lua，行为不变）[prefabs/staff_common.lua:471](../../../raw/current/scripts/prefabs/staff_common.lua#L471)、[:472](../../../raw/current/scripts/prefabs/staff_common.lua#L472)、[:427](../../../raw/current/scripts/prefabs/staff_common.lua#L427) |
| 对世界地点产生光源 | yellowstaff，黄色工厂 | canuseonpoint／water + reticule；实际创建光源由 createlight 完成。[prefabs/staff_common.lua:843](../../../raw/current/scripts/prefabs/staff_common.lua#L843)、[:844](../../../raw/current/scripts/prefabs/staff_common.lua#L844)、[:825](../../../raw/current/scripts/prefabs/staff_common.lua#L825)、[:782](../../../raw/current/scripts/prefabs/staff_common.lua#L782) |

名称注册见 [prefabs/staff.lua:305](../../../raw/current/scripts/prefabs/staff.lua#L305 和 [prefabs/staff.lua:308](../../../raw/current/scripts/prefabs/staff.lua#L308)。实现新技能时先核验效果需要的具体机制，只复用适用分支；不要将上述表格当完整新法杖工程。

### 不装备直接对目标施法（USEITEM 级动作筛选）

26148036 核验（R-04 新增处理器）：法杖类物品不装备、从物品栏直接右键目标时也会出现 CASTSPELL——判定完全走物品标签的 replica 级动作筛选：`castontargets`／`castonrecipes`（排除 no_deconstruction 食谱）／`castonlocomotors`（含 pvp 变体）／`castonworkable`（CHOP/DIG/HAMMER/MINE_workable）／`castoncombat`；目标带 `nomagic` 反制、物品带 `nomagiccast` 直接返回、`crushitemcast` 物品在骑乘或重物搬运时不可用。[componentactions.lua:1774](../../../raw/current/scripts/componentactions.lua#L1774)、[:1786](../../../raw/current/scripts/componentactions.lua#L1786)、[:1795](../../../raw/current/scripts/componentactions.lua#L1795)

接入方式：这些标签由组件 oncancast 按 canuse 标志自动维护（`canuseontargets` 无专用 canonlyuseon* 过滤时加 `castontargets`，其余一一对应），给物品配好组件标志即同时获得装备（EQUIPPED）与不装备（USEITEM）两条接入面；已有物品因此可能多出"未装备直接对目标施法"的入口。[components/spellcaster.lua:1](../../../raw/current/scripts/components/spellcaster.lua#L1)、[:5](../../../raw/current/scripts/components/spellcaster.lua#L5) 判断等级：此筛选只决定动作是否出现，最终仍由 CASTSPELL.fn 服务端 CanCast／CastSpell 验证。[actions.lua:3445](../../../raw/current/scripts/actions.lua#L3445

### 物品栏施法的冷却边界

无目标／无 pos 时，CanCast 直接返回 canusefrominventory；若技能要冷却，必须在最终效果函数中检查 rechargeable:IsCharged，并在成功路径 Discharge(seconds)。对目标／地点也保留同一权威门禁，以免其他入口绕过。通用辅助实现见 [[reference/components/rechargeable#为自定义施放函数提供一次-10-秒充能门禁]]。[components/spellcaster.lua:159](../../../raw/current/scripts/components/spellcaster.lua#L159 [actions.lua:3465](../../../raw/current/scripts/actions.lua#L3465

## 常见陷阱与限制

- **SetCanCastFn 冷却判断覆盖全部施法**：库存无目标分支不调用它；目标内置条件成功也可能通过 OR 跳过它。最终效果函数必须复核。[components/spellcaster.lua:159](../../../raw/current/scripts/components/spellcaster.lua#L159 [components/spellcaster.lua:190](../../../raw/current/scripts/components/spellcaster.lua#L190
- **物品带 nomagiccast 标签即全局禁施法**（26148036 核验：R-01/R-04 新增门控）：CASTSPELL 执行 fn 首查该标签直接 false（[actions.lua:3450](../../../raw/current/scripts/actions.lua#L3450）；动作筛选层 USEITEM／POINT／EQUIPPED／INVENTORY 四个 spellcaster 处理器同样拦截（[componentactions.lua:1775](../../../raw/current/scripts/componentactions.lua#L1775)、[:2193](../../../raw/current/scripts/componentactions.lua#L2193)、[:2467](../../../raw/current/scripts/componentactions.lua#L2467)、[:2954](../../../raw/current/scripts/componentactions.lua#L2954)）。当前唯一动态添加／移除方是腐化王杖 king_cane（腐化值变化时加／删，[prefabs/king_cane.lua:119](../../../raw/current/scripts/prefabs/king_cane.lua#L119)、[:127](../../../raw/current/scripts/prefabs/king_cane.lua#L127)）；模组给任意施法物品加此标签即整条入口失效，SCENE 段无 spellcaster 处理器。
- **CASTSPELL 不能长按连发**（26148036 核验：R-01 动作定义加 `invalid_hold_action=true`，[actions.lua:455](../../../raw/current/scripts/actions.lua#L455)，消费点 [components/playercontroller.lua:4626](../../../raw/current/scripts/components/playercontroller.lua#L4626）：施法类动作注册后如需同样禁止按住连发，应在动作定义处加同参数。
- **CastSpell 返回成功 boolean**：当前函数无显式返回，效果返回值也忽略；正常 ACTION 在 CanCast 通过后返回 true，不能据此证明效果产生。[components/spellcaster.lua:134](../../../raw/current/scripts/components/spellcaster.lua#L134 [actions.lua:3465](../../../raw/current/scripts/actions.lua#L3465
- **只有 canuseontargets 就能选任意目标**：还需有可满足的具体过滤或 can_cast_fn；同时有实体状态限制。[components/spellcaster.lua:179](../../../raw/current/scripts/components/spellcaster.lua#L179

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 全部变更文件（actions.lua、componentactions.lua、components/spellcaster.lua、prefabs/staff.lua 迁出 staff_common.lua）对照 build-26148036 更新报告逐条核验，条目 R-01、R-04、C-40、P-112（nomagiccast 门控、USEITEM 级 spellcaster 处理器、CASTSPELL 禁长按、onspellcast 事件）；锚点重定位 16 处、另修正旧页错位锚点 2 处（旧 :157/:160 与声明内容不符）；结论修订 5 处（见正文"26148036 核验"标注）；未覆盖：运行验证。
- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：无目标／地点／实体 CanCast 分支、CastSpell 返回约定与紫黄法杖路径，依据见上文对应锚点；未交付新技能工程或核验全部目标过滤与状态图。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；本页为机制与原版路径说明，未附新技能工程。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

相关：[[reference/components/rechargeable]]；[[reference/components/spellbook]]；[[reference/components/finiteuses]]。
