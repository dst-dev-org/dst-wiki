---
title: "肥料包（compostwrap）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "prefabs/compostwrap.lua 双注册的沃姆木专属肥料：肥料包（compostwrap）是 plantkin 合成的高效肥料——可施肥作物／农田、可部署到农场土格、可作燃料（冒毒云）、可闷熄阴燃物、落地生苍蝇；沃姆木自我施肥时按堆肥养分触发关联注册 compostheal_buff 的 2 血／2 秒持续回血（32 点约 32 秒）。"
aliases: ["compostwrap", "compostheal_buff", "肥料包", "堆肥包", "compostheal", "沃姆木堆肥"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/compostwrap.lua", "prefabs/fertilizer_nutrient_defs.lua", "components/fertilizer.lua", "components/fertilizerresearchable.lua", "components/smotherer.lua", "components/debuff.lua", "components/debuffable.lua", "components/burnable.lua", "standardcomponents.lua", "prefabs/wormwood.lua", "componentactions.lua", "actions.lua", "stategraphs/SGwilson.lua", "prefabs/flies.lua", "tuning.lua", "recipes.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 肥料包（compostwrap）

## 快速判断

- 职责与适用：`prefabs/compostwrap.lua` 注册 2 个实体：肥料 `compostwrap`（[fn L71-L134](../../../raw/current/scripts/prefabs/compostwrap.lua#L71-L134)）与服务端专用 debuff `compostheal_buff`（[buff_fn L179-L205](../../../raw/current/scripts/prefabs/compostwrap.lua#L179-L205)）。肥料包四条用法：施肥（FERTILIZE 动作）、部署农场土格（DEPLOY）、当高能燃料（毒云副作用）、闷熄阴燃物（SMOTHER）；只有沃姆木能自我施肥触发 `compostheal_buff`。
- 优先使用：查养分与回血数值读本页"施肥通道与 compostheal_buff"；做自定义肥料对照 [[reference/prefabs/fertilizer-materials]]。
- 重要副作用：作燃料被火收走时在火位上方生成 `poopcloud`（[compostwrap.lua L27-L35](../../../raw/current/scripts/prefabs/compostwrap.lua#L27-L35)）；落地时生成苍蝇子实体、入包移除（[L38-L60](../../../raw/current/scripts/prefabs/compostwrap.lua#L38-L60)）。
- 修改边界：buff 是 CLASSIFIED 非联网实体（[L193-L196](../../../raw/current/scripts/prefabs/compostwrap.lua#L193-L196)），仅服务端；养分表经 `FERTILIZER_DEFS` 间接读 TUNING。未覆盖：农田系统（farming_manager）内部、作物侧 `crop`／`grower`／`pickable` 的施肥细节（归组件页）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `compostwrap`（肥料包） | `fn()`：网络段（bank/build `healing_cream`、`fertilizer`＋`slowfertilize`＋`fertilizerresearchable` 标签、`GetFertilizerKey`、浮水、`MakeDeployableFertilizerPristine` 的 deployable/tile_deploy 标签与 `gridplacer_farmablesoil` 放置器）→ `SetPristine` → master 段（stackable SMALLITEM 40、inventoryitem 落地／入包回调、fertilizerresearchable、fertilizer、fuel＋`SetOnTakenFn`、smotherer、小燃烧＋`OnBurn`、`MakeDeployableFertilizer`、hauntable 点燃版、延迟 0 秒生成苍蝇） | 动画复用 `healing_cream` 资产（[L80-L82](../../../raw/current/scripts/prefabs/compostwrap.lua#L80-L82)）；养分 `FERTILIZER_DEFS.compostwrap`＝`TUNING.COMPOSTWRAP_NUTRIENTS`＝`{24, 32, 24}`（[fertilizer_nutrient_defs.lua L16](../../../raw/current/scripts/prefabs/fertilizer_nutrient_defs.lua#L16)、[tuning.lua L5872](../../../raw/current/scripts/tuning.lua#L5872)） |
| `compostheal_buff`（沃姆木堆肥回血） | `buff_fn()`：客户端自删 → 服务端隐藏实体＋CLASSIFIED＋debuff（`keepondespawn=true`）＋timer | 与主注册同文件返回（[L210-L211](../../../raw/current/scripts/prefabs/compostwrap.lua#L210-L211)）；`keepondespawn` 使玩家 despawn 不清（[compostwrap.lua L198-L202](../../../raw/current/scripts/prefabs/compostwrap.lua#L198-L202)、[debuffable.lua L39-L47](../../../raw/current/scripts/components/debuffable.lua#L39-L47)） |

## 施肥通道与 compostheal_buff

### 肥料数值（fertilizer 组件）

- `fertilizervalue = TUNING.COMPOSTWRAP_FERTILIZE`＝6 个昼秒（`day_time*6`，[compostwrap.lua L115](../../../raw/current/scripts/prefabs/compostwrap.lua#L115)、[tuning.lua L4572](../../../raw/current/scripts/tuning.lua#L4572)）；`soil_cycles=20`、`withered_cycles=2`（[L116-L117](../../../raw/current/scripts/prefabs/compostwrap.lua#L116-L117)、[tuning.lua L4570-L4571](../../../raw/current/scripts/tuning.lua#L4570-L4571)）；养分 `{24, 32, 24}`＝formula／compost／manure 三通道（[tuning.lua L5854-L5856](../../../raw/current/scripts/tuning.lua#L5854-L5856)）。
- 消费入口三处：① FERTILIZE 动作对目标（作物 `notreadyforharvest`／农田 `fertile`/`infertile`／可采物 `barren` 等，[componentactions.lua L1241-L1253](../../../raw/current/scripts/componentactions.lua#L1241-L1253)，结算 [actions.lua L1812-L1831](../../../raw/current/scripts/actions.lua#L1812-L1831)）；② 自我施肥（沃姆木，见下）；③ 部署到农场土格：`fertilizer_ondeploy` 把三养分加到土格（`farming_manager:AddTileNutrients`）后 `OnApplied` 按堆叠扣 1（[standardcomponents.lua L1797-L1802](../../../raw/current/scripts/standardcomponents.lua#L1797-L1802)、消耗 [fertilizer.lua L41-L45](../../../raw/current/scripts/components/fertilizer.lua#L41-L45)）。

### 沃姆木自我施肥与 compostheal_buff

- 候选：物品带 `fertilizer` 标签＋沃姆木带 `self_fertilizable` 标签（[wormwood.lua L755](../../../raw/current/scripts/prefabs/wormwood.lua#L755)）＋可回血 → FERTILIZE（[componentactions.lua L2712-L2718](../../../raw/current/scripts/componentactions.lua#L2712-L2718)）；`slowfertilize` 标签让 SG 走完整 `fertilize` 状态而非快速版（[SGwilson.lua L1018-L1022](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1018-L1022)）。
- 结算：`fertilizable:Fertilize` → 沃姆木 `OnFertilized` 按三养分分流（[wormwood.lua L525-L536](../../../raw/current/scripts/prefabs/wormwood.lua#L525-L536)）；堆肥通道 `OnFertilizedWithCompost` 取 `HEAL_VALUES[ceil(值/8)]`——肥料包 compost 养分 32 → 第 4 档 32 点（[wormwood.lua L511-L514](../../../raw/current/scripts/prefabs/wormwood.lua#L511-L514)、[tuning.lua L5882-L5884](../../../raw/current/scripts/tuning.lua#L5882-L5884)）。
- **buff 语义**：`AddDebuff` 挂 `compostheal_buff`，`duration = 32 × (2秒/2血) = 32` 秒；`OnAttached` 起每 2 秒 `OnTick` 回 2 血（目标存活、有理智、非幽灵才回，否则停），`regenover` 计时到点或目标死亡即 `Stop`（[compostwrap.lua L150-L154](../../../raw/current/scripts/prefabs/compostwrap.lua#L150-L154)、[L139-L146](../../../raw/current/scripts/prefabs/compostwrap.lua#L139-L146)、[L160-L162](../../../raw/current/scripts/prefabs/compostwrap.lua#L160-L162)）。再次施肥走 `Extend`：只把 `regenover` 延长到不小于新 duration，**不叠加**（[L166-L175](../../../raw/current/scripts/prefabs/compostwrap.lua#L166-L175)、[debuffable.lua L104-L105](../../../raw/current/scripts/components/debuffable.lua#L104-L105)）。
- 同一次施肥还会走 formula 通道（24 点催开花进度）与 manure 通道（24 点即时回血 `MANURE_HEAL_VALUES[3]`＝8）——这两条归沃姆木开花机制，本页不展开（[wormwood.lua L525-L536](../../../raw/current/scripts/prefabs/wormwood.lua#L525-L536)、[tuning.lua L5880](../../../raw/current/scripts/tuning.lua#L5880)）。

### 附加行为

- **燃料毒云**：`LARGE_FUEL`（180 秒，[tuning.lua L1677](../../../raw/current/scripts/tuning.lua#L1677)）；被火收走时在火 FX 位置上方生成 `poopcloud`（[compostwrap.lua L27-L35](../../../raw/current/scripts/prefabs/compostwrap.lua#L27-L35)）。
- **闷熄**：`smotherer` 组件让物品可对阴燃（`smolder` 标签）目标出 SMOTHER，按堆叠扣 1（[componentactions.lua L1736-L1741](../../../raw/current/scripts/componentactions.lua#L1736-L1741)、[actions.lua L1845-L1850](../../../raw/current/scripts/actions.lua#L1845-L1850)、[burnable.lua L417-L428](../../../raw/current/scripts/components/burnable.lua#L417-L428)）。
- **苍蝇子实体**：落地（`OnDropped`）或初始化 0 秒后生成 `flies` 子实体，入包或点燃时移除；延迟生成是 V2C 备注的"避免刚合成进包就见苍蝇"（[compostwrap.lua L16-L23](../../../raw/current/scripts/prefabs/compostwrap.lua#L16-L23)、[L38-L60](../../../raw/current/scripts/prefabs/compostwrap.lua#L38-L60)、[L133-L134](../../../raw/current/scripts/prefabs/compostwrap.lua#L133-L134)）；`flies` 本体是蜂群声效 FX（[flies.lua L11-L16](../../../raw/current/scripts/prefabs/flies.lua#L11-L16)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：fertilizer | 四数值载体＋`SetNutrients`；`OnApplied` 消耗 | 服务端；FERTILIZE／DEPLOY／自我施肥三入口 | [[reference/components/fertilizer]]；[compostwrap.lua L114-L118](../../../raw/current/scripts/prefabs/compostwrap.lua#L114-L118) |
| 组件：fertilizerresearchable | `GetFertilizerKey`＝注册名，供植物图鉴研究 | 研究动作 `right` | [[reference/components/fertilizerresearchable]]；[compostwrap.lua L63-L68](../../../raw/current/scripts/prefabs/compostwrap.lua#L63-L68)、[L111-L112](../../../raw/current/scripts/prefabs/compostwrap.lua#L111-L112) |
| 组件：deployable＋tile_deploy | `DEPLOYMODE.CUSTOM`＋`gridplacer_farmablesoil`，部署加土格养分、物品保留逻辑由 `keep_in_inventory_on_deploy` 处理 | 服务端；`ondeploy` 消耗 | [standardcomponents.lua L1816-L1820](../../../raw/current/scripts/standardcomponents.lua#L1816-L1820)、[L1823-L1828](../../../raw/current/scripts/standardcomponents.lua#L1823-L1828)、[compostwrap.lua L130](../../../raw/current/scripts/prefabs/compostwrap.lua#L130) |
| 组件：fuel／burnable／smotherer | 180 秒燃料＋毒云、`MED_BURNTIME` 燃烧、可闷熄 | 服务端；点燃移除苍蝇 | [compostwrap.lua L120-L128](../../../raw/current/scripts/prefabs/compostwrap.lua#L120-L128) |
| 组件：debuff＋timer（仅 buff） | 2 秒周期回血＋`regenover` 总时长 | 仅服务端；`keepondespawn=true` | [compostwrap.lua L198-L205](../../../raw/current/scripts/prefabs/compostwrap.lua#L198-L205) |
| SG：`fertilize`（玩家） | `slowfertilize` 标签分支选完整施肥动作 | SGwilson 服务端＋客户端镜像 | [SGwilson.lua L1018-L1022](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1018-L1022) |
| 关联 Prefab：`flies`／`poopcloud`／`gridplacer_farmablesoil` | 子实体 FX／毒云 FX／部署放置器 | 落地与燃烧触发 | [compostwrap.lua L7-L12](../../../raw/current/scripts/prefabs/compostwrap.lua#L7-L12)；flies 页 Wiki 待建 |
| 数据：FERTILIZER_DEFS | 养分表共享来源（`require("prefabs/fertilizer_nutrient_defs")`） | 与 [[reference/prefabs/fertilizer-materials]] 同表 | [compostwrap.lua L14](../../../raw/current/scripts/prefabs/compostwrap.lua#L14) |

## 获得路径

- **合成**：粪便×5＋变质食物×2＋硝石×1，`TECH.NONE`，`builder_tag="plantkin"`（沃姆木专属，[recipes.lua L341](../../../raw/current/scripts/recipes.lua#L341)）。
- **消耗**：沃姆木技能配方 `wormwood_lureplant` 需 2 个（[recipes.lua L348](../../../raw/current/scripts/recipes.lua#L348)）。
- 无掉落／生成点：快照内除配方与图鉴声明外无 `SpawnPrefab("compostwrap")`。

## 常见陷阱与限制

- **边界：compostheal_buff 只有沃姆木能触发。** 自我施肥入口要求 `self_fertilizable` 标签＋`fertilizable` 组件，快照内仅沃姆木接线（[wormwood.lua L755](../../../raw/current/scripts/prefabs/wormwood.lua#L755)、[L845-L846](../../../raw/current/scripts/prefabs/wormwood.lua#L845-L846)）；给别的角色喂肥料包走不了这条链。
- **边界：回血与回血时长等值换算。** 总回复（血）＝时长（秒），由 `duration = healing × (TICK/HEALTH) = healing × 1` 保证；改 TUNING 两个常量的比例会破坏这个关系。
- **边界：再次施肥不叠加只续时。** `_tick` 周期任务每 2 秒固定 +2，`Extend` 只延长 `regenover`；想堆叠次数需要改 OnExtended 语义。
- **边界：幽灵／无理智目标直接停 buff。** OnTick 的门禁把 `playerghost` 与无 `sanity` 组件的目标排除（[compostwrap.lua L139-L146](../../../raw/current/scripts/prefabs/compostwrap.lua#L139-L146)）。

## 验证记录

- 源码审阅：2026-11-10，steam-buildid:26474873。`prefabs/compostwrap.lua` 全文（双注册、四用法、buff 全回调）；`fertilizer`／`fertilizerresearchable`／`smotherer`／`debuff`／`debuffable` 相关段；`standardcomponents` 的 fertilizer_ondeploy／MakeDeployableFertilizer(Pristine)；wormwood 的 OnFertilized 分流与自我施肥接线；FERTILIZE 动作 fn 与 componentactions 两处候选；SGwilson slowfertilize 分支；burnable:SmotherSmolder；flies FX 段；tuning 全部引用常量；配方两条；`chinese_s.po` `NAMES.COMPOSTWRAP` 实显（肥料包）。
- 未覆盖：farming_manager 土格养分结算、crop／grower／pickable 各自 Fertilize 的内部、其他角色与肥料包的交互、图鉴研究流程。
- 游戏运行：未运行。

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
