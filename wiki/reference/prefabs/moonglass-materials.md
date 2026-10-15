---
title: "月亮碎片与注能月亮碎片（moonglass、moonglass_charged）"
created: "2026-11-08"
updated: "2026-11-08"
type: prefab
summary: "prefabs/moonglass.lua 双注册的天体材料：两态共用 createglass 工厂（可堆叠 40、LUNAR_SHARDS 可食、可交易、入水沉没、形态随机三选一并入存档）；注能态在 preinit/postinit 追加光效、infusedshard 标签与 1.5 天新鲜度，耗尽替换为普通月亮碎片。"
aliases: ["moonglass", "moonglass_charged", "月亮碎片", "注能月亮碎片", "Moon Shard", "Infused Moon Shard", "月光玻璃"]
tags: [prefab, item]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/moonglass.lua", "components/lunarhailmanager.lua", "components/lunarhailbuildup.lua", "standardcomponents.lua", "components/inventoryitem.lua", "prefabs/moonstorm_glass.lua", "prefabs/moonstorm_lightning.lua", "prefabs/alterguardian_phase1.lua", "prefabs/gestalt_guard_evolved.lua", "prefabs/hats.lua", "prefabs/birdcage.lua", "prefabs/merm.lua", "prefabs/moondial.lua", "prefabs/birds_mutant_rift.lua", "prefabs/buzzard.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 月亮碎片与注能月亮碎片（moonglass、moonglass_charged）

## 快速判断

- 职责与适用：`prefabs/moonglass.lua` 用同一工厂 `createglass(name, preinit, postinit)` 注册 2 个矿物材料物品：`moonglass`（官方简中"月亮碎片"）与 `moonglass_charged`（"注能月亮碎片"）。都是 item：可堆叠 40、LUNAR_SHARDS 食物类型、可交易、入水沉没、地面形态 f1/f2/f3 随机并存档；注能态是带光效与新鲜度的强化版。**注意与 `moonglassaxe`（月光玻璃斧，见 [[reference/prefabs/axe|斧头族]]）是不同注册**，本页不收斧本体。[工厂与注册](../../../raw/current/scripts/prefabs/moonglass.lua#L21-L151)
- 优先使用：查获得路径（月雹／月暴／天体英雄）、耗尽替换链、消费配方读本页；月暴玻璃岩本体与爆裂 fx 见 [[reference/prefabs/moonstorm_glass_fx|月之装置破碎与月暴玻璃爆裂]]，配方消耗见 [recipes.lua](../../../raw/current/scripts/recipes.lua)。
- 重要副作用：`moonglass_charged` 挂 `perishable`（1.5 天），耗尽替换为 `moonglass`（[moonglass.lua L143](../../../raw/current/scripts/prefabs/moonglass.lua#L143)），新鲜度显示走 `displayadjectivefn`（Waning／Decayed）；两态 `SetSinks(true)` 掉水即沉没（[L73](../../../raw/current/scripts/prefabs/moonglass.lua#L73)、[inventoryitem.lua L541-L546](../../../raw/current/scripts/components/inventoryitem.lua#L541-L546)）。
- 修改边界：两态差异集中在 `infused_preinit`（`SetPristine` 前，客户端也生效）与 `infused_postinit`（master 段末）；工厂 fn 内不区分注册名（仅 assets 分支，[L28-L33](../../../raw/current/scripts/prefabs/moonglass.lua#L28-L33)）；本页未覆盖月暴／月雹事件的开启条件与月之装置机制。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `moonglass` | `createglass("moonglass", nil, regular_postinit)`：共用 fn（网络段：Transform／AnimState／SoundEmitter／Network、`MakeInventoryPhysics`、`SetRayTestOnBB`、bank/build `moonglass` 播 `f1`、`moonglass_piece`／`lunarhaildebris`／`quakedebris` 标签）→ preinit（无）→ `SetPristine` → 客户端返回 → master 段（tradable／stackable 40／inspectable／inventoryitem＋SetSinks／edible LUNAR_SHARDS／bait／hauntable、随机形态、OnSave/OnLoad）→ postinit 加 `snowmandecor` | postinit 仅一行 `AddComponent("snowmandecor")`（[moonglass.lua L109-L112](../../../raw/current/scripts/prefabs/moonglass.lua#L109-L112)），冬季盛宴装饰雪人用 |
| `moonglass_charged` | `createglass("moonglass_charged", infused_preinit, infused_postinit)`：preinit 在 `SetPristine` **前**执行——加 Light（半径 1、蓝紫色）、bloom、bank/build 换 `moonglass_charged`、`show_spoilage`＋`infusedshard` 标签、`displayadjectivefn`（客户端可用）；master 段末 postinit 加 perishable 与灯光开关事件 | 光照随持有状态切换：`onputininventory` 灭灯、`ondropped` 开灯（[L101-L107](../../../raw/current/scripts/prefabs/moonglass.lua#L101-L107)、[L145-L146](../../../raw/current/scripts/prefabs/moonglass.lua#L145-L146)）；perishable 1.5 天（[tuning.lua L6248](../../../raw/current/scripts/tuning.lua#L6248)）耗尽替换 `moonglass`（[L140-L143](../../../raw/current/scripts/prefabs/moonglass.lua#L140-L143)） |

两态共用：`glassname` 随机 f1/f2/f3 并入存档（[L5-L19](../../../raw/current/scripts/prefabs/moonglass.lua#L5-L19)、[L84-L89](../../../raw/current/scripts/prefabs/moonglass.lua#L84-L89)）；edible `FOODTYPE.LUNAR_SHARDS`、`hungervalue = CALORIES_SMALL`（12.5）、`healthvalue = HEALING_TINY`（1）（[L75-L78](../../../raw/current/scripts/prefabs/moonglass.lua#L75-L78)、[tuning.lua L2059](../../../raw/current/scripts/tuning.lua#L2059)）；`tradable`、`stackable` 40、`bait`、`MakeHauntableLaunchAndSmash`（[L65-L82](../../../raw/current/scripts/prefabs/moonglass.lua#L65-L82)）。

## 获得路径

- **月雹（lunar hail）落屑**：`lunarhailmanager` 落屑权重 moonglass 2.5 : moonglass_charged 1（[lunarhailmanager.lua L33-L34](../../../raw/current/scripts/components/lunarhailmanager.lua#L33-L34)）；带月雹积垢的实体（`MakeLunarHailBuildup`，小/中/大 1/2/3 枚，[standardcomponents.lua L888-L918](../../../raw/current/scripts/standardcomponents.lua#L888-L918)、[tuning.lua L7911-L7915](../../../raw/current/scripts/tuning.lua#L7911-L7915)）被清理时 `DropRewards` 按幸运值 roll 出 charged 或 plain（[lunarhailbuildup.lua L151-L161](../../../raw/current/scripts/components/lunarhailbuildup.lua#L151-L161)）。
- **月暴（moonstorm）**：闪电劈中 4 距离内的 `moonstorm_glass` 玻璃岩 → 从岩体弹出 `moonglass_charged`，无岩则生成新岩（[moonstorm_lightning.lua L58-L71](../../../raw/current/scripts/prefabs/moonstorm_lightning.lua#L58-L71)）；玻璃岩本身挖掘掉 2 枚必得＋0.25 概率第 3 枚 charged（[moonstorm_glass.lua L16-L20](../../../raw/current/scripts/prefabs/moonstorm_glass.lua#L16-L20)、[L146](../../../raw/current/scripts/prefabs/moonstorm_glass.lua#L146)）。
- **天体守卫战利品**：天体英雄一阶段掉 plain 与 charged（[alterguardian_phase1.lua L57-L67](../../../raw/current/scripts/prefabs/alterguardian_phase1.lua#L57-L67)）；天选守卫（gestalt_guard_evolved）消散按剩余血量比例返还 charged（[gestalt_guard_evolved.lua L65-L75](../../../raw/current/scripts/prefabs/gestalt_guard_evolved.lua#L65-L75)）。
- **其他**：流浪商贩用灰×4 换 1 枚 plain（[recipes.lua L1008](../../../raw/current/scripts/recipes.lua#L1008)）；月相仪（moondial）解除 glassed 态分两条退晶路径——初始化或睡眠的即时分支取半径 4 内搜索结果的**第一个** `moonglass_piece` 候选（两态都带该标签，可以是注能态），该堆未满则数量 +1；第一堆已满或无候选时直接弹出普通碎片，后面的堆即使有空位也不继续找（[moondial.lua L57-L67](../../../raw/current/scripts/prefabs/moondial.lua#L57-L67)）；清醒分支走 `glassed_pst` 状态第 10 帧直接弹出普通碎片（[SGmoondial.lua L146-L159](../../../raw/current/scripts/stategraphs/SGmoondial.lua#L146-L159)）。未检：世界生成 `moonglass_rock`（[forest.lua L377](../../../raw/current/scripts/prefabs/forest.lua#L377) 命中，未核验挖掘产出）。

## 主要消费

- **plain 配方**（量按快照 [recipes.lua](../../../raw/current/scripts/recipes.lua)）：玻璃切割器 5（[L839](../../../raw/current/scripts/recipes.lua#L839)，见 [[reference/prefabs/glasscutter|玻璃切割器]]）、月光玻璃斧 3（[L838](../../../raw/current/scripts/recipes.lua#L838)）、弹弓碎裂弹 1→20（[L227](../../../raw/current/scripts/recipes.lua#L227)）、月暴护目镜 2（[L540](../../../raw/current/scripts/recipes.lua#L540)）、电围栏 6（[L604](../../../raw/current/scripts/recipes.lua#L604)）、蜘蛛月变异器 2（[L333](../../../raw/current/scripts/recipes.lua#L333)）、寄居蟹shellweaver 5（[L973](../../../raw/current/scripts/recipes.lua#L973)）、月之锻造套件 5（[L840](../../../raw/current/scripts/recipes.lua#L840)）、木工月刀 6（[L841](../../../raw/current/scripts/recipes.lua#L841)）、陨石地皮 2（[L842](../../../raw/current/scripts/recipes.lua#L842)）、守卫套件 8（[L858](../../../raw/current/scripts/recipes.lua#L858)）、静电捕集器 2（[L1020](../../../raw/current/scripts/recipes.lua#L1020)）、空瓶 3（[L980](../../../raw/current/scripts/recipes.lua#L980)）。
- **charged 配方**：WX-78 容量模块 1（[L773](../../../raw/current/scripts/recipes.lua#L773)）、月植炸弹 1（[L850](../../../raw/current/scripts/recipes.lua#L850)）、纯粹光辉转化 3（[L1499](../../../raw/current/scripts/recipes.lua#L1499)）、月之装置建造 10／20（[L1530-L1531](../../../raw/current/scripts/recipes.lua#L1530-L1531)）。
- **天选者王冠种植**：戴帽玩家附近需有 ≥3 枚 charged（按堆叠计，[tuning.lua L8799](../../../raw/current/scripts/tuning.lua#L8799)），种植时从堆叠扣除并召唤天选守卫宠物（[hats.lua L3180-L3186](../../../raw/current/scripts/prefabs/hats.lua#L3180-L3186)、[L3240-L3248](../../../raw/current/scripts/prefabs/hats.lua#L3240-L3248)）。
- **鸟笼转化**：裂隙突变鸟只收 LUNAR_SHARDS（[birdcage.lua L173-L176](../../../raw/current/scripts/prefabs/birdcage.lua#L173-L176)）；累计吃满 `RIFT_BIRD_EAT_COUNT_FOR_BRILLIANCE` 枚 charged 触发冷却并置位，消化后产出 `purebrilliance`（[L113-L117](../../../raw/current/scripts/prefabs/birdcage.lua#L113-L117)、[L206-L210](../../../raw/current/scripts/prefabs/birdcage.lua#L206-L210)）。
- **鱼人月化**：沃特点亮月阵营技能后，给 follower 鱼人递 `moonglass_piece` 物品会将其月化（`DoLunarMutation`，[merm.lua L416-L419](../../../raw/current/scripts/prefabs/merm.lua#L416-L419)、[L870-L874](../../../raw/current/scripts/prefabs/merm.lua#L870-L874)）。
- **进食者**：裂隙突变鸟 DIET（[birds_mutant_rift.lua L123](../../../raw/current/scripts/prefabs/birds_mutant_rift.lua#L123)）、变异秃鹫 MUTATED_DIET（[buzzard.lua L394](../../../raw/current/scripts/prefabs/buzzard.lua#L394)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：inventoryitem | `SetSinks(true)`：落水沉没 | 两态同值；灯光实体 dropped 时才亮 | [moonglass.lua L72-L73](../../../raw/current/scripts/prefabs/moonglass.lua#L72-L73)；[[reference/components/inventoryitem]] |
| 组件：edible／bait／tradable | LUNAR_SHARDS 食物（12.5 饱食＋1 血）、饵料、交易门槛 | 两态同值；吃者限裂隙突变鸟与变异秃鹫 | [moonglass.lua L65-L82](../../../raw/current/scripts/prefabs/moonglass.lua#L65-L82)；[[reference/components/edible]] |
| 组件：perishable（仅 charged） | 1.5 天、`onperishreplacement = "moonglass"`、`StartPerishing` | 服务端；`show_spoilage` 驱动新鲜度显示 | [moonglass.lua L140-L143](../../../raw/current/scripts/prefabs/moonglass.lua#L140-L143)；[[reference/components/perishable]] |
| 组件：snowmandecor（仅 plain） | 冬季盛宴装饰雪人的合格材料 | 服务端；DECORATE 动作链 | [moonglass.lua L109-L112](../../../raw/current/scripts/prefabs/moonglass.lua#L109-L112) |
| API／数据：`moonglass_piece` 标签 | 月化鱼人递礼判定、月相仪找碎片、王冠种植初筛 | 两态都有；hats 初筛后按 `prefab == "moonglass_charged"` 过滤 | [moonglass.lua L51](../../../raw/current/scripts/prefabs/moonglass.lua#L51)；[merm.lua L870-L874](../../../raw/current/scripts/prefabs/merm.lua#L870-L874)、[moondial.lua L51](../../../raw/current/scripts/prefabs/moondial.lua#L51)、[hats.lua L7-L8](../../../raw/current/scripts/prefabs/hats.lua#L7-L8) |
| API／数据：`lunarhaildebris` 标签 | 月雹落屑命中时被弹开而非受击 | 两态同值；仅月雹事件期间 | [moonglass.lua L52](../../../raw/current/scripts/prefabs/moonglass.lua#L52)、[lunarhailmanager.lua L169-L170](../../../raw/current/scripts/components/lunarhailmanager.lua#L169-L170) |
| API／数据：`infusedshard` 标签 | 快照内仅挂载，未发现原版消费者（预留标签） | 仅 charged | [moonglass.lua L126](../../../raw/current/scripts/prefabs/moonglass.lua#L126)；全域检索仅此一处 |
| API／数据：`displayadjectivefn` | stale→"Waning"、spoiled→"Decayed" 显示前缀 | 仅 charged | [moonglass.lua L114-L118](../../../raw/current/scripts/prefabs/moonglass.lua#L114-L118) |
| 关联 Prefab：`moonstorm_glass` | 月暴玻璃岩：被劈产出 charged、被挖掉 charged | 月暴事件期间 | [[reference/prefabs/moonstorm_glass_fx]]；[moonstorm_glass.lua L146](../../../raw/current/scripts/prefabs/moonstorm_glass.lua#L146) |
| 关联 Prefab：`gestalt_guard_evolved` | 消散返还 charged；种植消耗 charged | 王冠宠物 | [gestalt_guard_evolved.lua L65-L75](../../../raw/current/scripts/prefabs/gestalt_guard_evolved.lua#L65-L75) |
| 关联 Prefab：`birds_mutant_rift` | 鸟笼吃 shards 产 purebrilliance | 裂隙鸟 | [birdcage.lua L113-L117](../../../raw/current/scripts/prefabs/birdcage.lua#L113-L117) |
| 资源 | `anim/moonglass.zip`（plain）、`anim/moonglass_charged.zip`（charged） | 工厂 assets 按注册名分支 | [moonglass.lua L23-L33](../../../raw/current/scripts/prefabs/moonglass.lua#L23-L33) |

## 常见陷阱与限制

- **边界：注能态会退化**：`moonglass_charged` 1.5 天后静默替换为 `moonglass`，光环与 infusedshard 标签随之消失——批量囤积要按新鲜度规划；替换沿 `Perish` 常规链继承原堆叠：两态都有 stackable，新实体继承原数量（`SetStackSize`，超上限临时 ignore），地面替换继承旧坐标，持有替换按原槽尝试 `GiveItem`——原槽可能保留（[替换链](../../../raw/current/scripts/components/perishable.lua#L293-L318)）。
- **边界：灯光只在地面上亮**：charged 的 Light 默认开，但入栏即灭（`onputininventory`），只有被丢出（`ondropped`）后重新亮；存档读回后灯光状态由这两事件重新驱动，不在存档字段里。
- **误区：`moonglass_piece` 标签能代表注能态**：标签两态都有，鱼人月化、月相仪找碎片都命中 plain 与 charged；需要区分时必须按 `prefab` 名或 `infusedshard` 标签判断（hats.lua 种植即先按标签找、再按 prefab 过滤）。
- **边界：形态随机不是皮肤**：f1/f2/f3 是同一 Prefab 的 `PlayAnimation` 随机形态并存档（`glassname`），无独立注册名，`AddPrefabPostInit` 拿不到形态区分。
- **边界：入水即沉**：`SetSinks(true)` 与多数矿物相同，海面掉落物不回收；月雹落屑期间靠 `lunarhaildebris` 标签被弹开，但这是位移不是拾取。

## 验证记录

- 源码审阅：2026-11-08，steam-buildid:26335394。`prefabs/moonglass.lua` 全文（工厂、preinit/postinit、形态存取、替换链）；获得链 lunarhailmanager（落屑表与 debris 弹开）、lunarhailbuildup（DropRewards 幸运 roll）、standardcomponents（积垢档位）、moonstorm_glass（战利品表与注册）、moonstorm_lightning（劈岩产 charged）、alterguardian_phase1（掉落表）、gestalt_guard_evolved（消散返还）；消费链 hats（种植扣除）、birdcage（purebrilliance 转化与冷却）、merm（月化递礼）、moondial（解除产出）、birds_mutant_rift／buzzard（食性）、recipes 15 条按行核验；tuning 常量核验；`chinese_s.po` 两 `NAMES.*` 键读取。未核验：月暴／月雹事件的开启与结束条件、`moonglass_rock` 世界生成与挖掘、`infusedshard` 标签是否存在动态消费者（字面检索仅挂载处）。
- 游戏运行：未运行。
- 结构检查：validate_page 通过（0 issue；81 行、15788 字节，standard 档内）；validate_source_links 通过（72 处引用、0 error／0 warning）。
- 独立检视修订（2026-11-08）：sources 补 components/inventoryitem.lua（正文 sinks 锚点实际引用，检视 P3 回源码确认）。
- 外部复检修订（2026-11-08，R04/R05，主代理回源码核实；R05 措辞经复检二次勘正）：月相仪退晶分两条路径——即时分支取搜索结果第一个 moonglass_piece 候选（可为注能态），该堆未满才数量 +1，第一堆已满或无候选时直接弹普通碎片、不继续找后续堆（moondial.lua L57-L67），清醒分支走 glassed_pst 第 10 帧直接弹普通碎片（SGmoondial.lua L146-L159）；陷阱「替换堆叠位置不保留」改为继承原堆叠数量／地面坐标／按原槽尝试 GiveItem（perishable.lua L293-L318）。

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
