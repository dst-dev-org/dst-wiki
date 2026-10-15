# 装备 Wiki 第一轮检视

源码快照 `26335394`，知识仓基准 `97afb4e`。以 [工具与野外实用索引](../../../wiki/reference/prefabs/tools-field-index.md) 的「采集、制作与维修工具」为范围，逐页对照工厂、动作与装备门禁、耐久、清理路径及 Lua 示例。本轮覆盖 10 页、20 个主要注册名，装备全批累计 10/69 页。以下 Wiki 行号按本轮工作树记录。

## P1

### R01 暗影收割者先完成收割，再扣动作耐久

`voidcloth_scythe.md:71–72` 把 `OnUsedAsItem` 写在 `ACTIONS.SCYTHE.fn` 之前。`BufferedAction:Do` 先执行动作函数，成功后才对仍然有效的 `invobject` 执行 `OnUsedAsItem`。镰刀的动作函数先调用 `DoScythe`，最后一次使用也先执行收割逻辑，再进入耐久耗尽的破损分支。动作成功与实际采到作物应分开表述，`SCYTHE.fn` 在调用 `DoScythe` 后直接返回 `true`。 [Wiki](../../../wiki/reference/prefabs/voidcloth_scythe.md#L71-L72) · [动作执行顺序](../../../raw/current/scripts/bufferedaction.lua#L22-L36) · [SCYTHE 动作](../../../raw/current/scripts/actions.lua#L4545-L4554) · [收割实现](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L224-L247)

### R02 暗影收割者示例的 45 点伤害会被覆盖

`voidcloth_scythe.md:132–149` 在 `AddPrefabPostInit` 中直接设 `weapon:SetDamage(45)`，随后承诺套装加成以 45 为基数。穿脱套装触发的更新从 `TUNING.VOIDCLOTH_SCYTHE_DAMAGE` 重算，修复重建 `weapon` 也从该值重新初始化。若示例意图改变整个预制物的伤害，应在生成前设专用 tuning。示例还只改了最大耐久，若意图让新物品以 250 点满耐久生成，需要连同初始使用数一起处理，原厂的专用 tuning 同时控制两者。 [Wiki](../../../wiki/reference/prefabs/voidcloth_scythe.md#L132-L149) · [套装更新](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L54-L69) · [武器重建](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L295-L307) · [修复路径](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L360-L368) · [耐久初始化](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L452-L455) · [SetMaxUses](../../../raw/current/scripts/components/finiteuses.lua#L54-L66)

### R03 气孔工具示例漏接破损和修复组件

`fumarole-tools.md:131–169` 的破损回调仅卸掉 `tool` 和 `weapon`，而 `MakeFumaroleTool` 只负责在破损时卸下已装备物品。示例仍保留 `equippable`，会留下重新装备破损物品的入口。修复回调又只添回空的 `weapon`，没有初始化伤害。照原厂在破损时移除 `equippable`、`tool` 和 `weapon`，在修复时恢复装备回调、动作参数和武器伤害。示例末尾的四次破损上限也应归到变体工厂的 `finiteuses` 配置和耗尽回调。 [Wiki](../../../wiki/reference/prefabs/fumarole-tools.md#L131-L169) · [通用破损处理](../../../raw/current/scripts/standardcomponents.lua#L2079-L2106) · [气孔斧回调](../../../raw/current/scripts/prefabs/axe.lua#L232-L251)

## P2

### R04 区分影契范围攻击门禁和灵魂结算

`voidcloth_scythe.md:23,46,76,157` 的几处“消耗 1 灵魂”会读成每次触发当场扣一个。实际要求影契技能、灵魂跳跃窗口及 `TryToPortalHop(1, false)` 成功。该调用可能累积成本并延长窗口，也可能立即收口，实际扣除在 `FinishPortalHop` 按累积成本结算。 [Wiki](../../../wiki/reference/prefabs/voidcloth_scythe.md#L76) · [镰刀影契门禁](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L286-L292) · [跳跃窗口](../../../raw/current/scripts/prefabs/wortox.lua#L614-L650) · [扣除灵魂](../../../raw/current/scripts/prefabs/wortox.lua#L580-L591)

### R05 补全暗影收割者伤害加成调用签名

`voidcloth_scythe.md:114` 写成 `AddBonus("lunar_aligned", 1.1)`，第二参实际是来源对象，倍率是第三参。写成 `AddBonus("lunar_aligned", inst, ...)` 或明确这里只列效果。 [Wiki](../../../wiki/reference/prefabs/voidcloth_scythe.md#L114) · [原厂调用](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L460-L461) · [组件签名](../../../raw/current/scripts/components/damagetypebonus.lua#L10-L16)

### R06 恢复气孔工具攻击磨损需处理忽略标志

`fumarole-tools.md:129` 只提恢复动作 `SetConsumption` 和 `attackwear`。`MakeFumaroleTool` 会设置 `finiteuses:SetIgnoreCombatDurabilityLoss(true)`，武器先检查该标志，通过才读取 `attackwear`。若示例要恢复攻击扣耐久，需在 helper 后清除标志，并在破损后重建武器时恢复攻击磨损配置。 [Wiki](../../../wiki/reference/prefabs/fumarole-tools.md#L129) · [设置忽略标志](../../../raw/current/scripts/standardcomponents.lua#L2074-L2077) · [武器扣耐久](../../../raw/current/scripts/components/weapon.lua#L105-L120)

### R07 分开气孔工具修复阈值、显示档位和读档

`fumarole-tools.md:104–112,174,176` 应分开说明。热度恰好 60 已满足 `>= 60` 修复，按严格 `> 60` 比较仍为第 3 档。未持有时，修复回调还可单独排入 `idle_4`，动画不能反推档位。读档恢复已破损状态时传 `loading=true`，原厂跳过 `finiteuses:Use(1)`，应把“每次破损扣一次”限定为实际发生的新破损。 [Wiki](../../../wiki/reference/prefabs/fumarole-tools.md#L104-L112) · [阈值与档位](../../../raw/current/scripts/components/fumaroletool.lua#L5-L19) · [热度更新](../../../raw/current/scripts/components/fumaroletool.lua#L133-L142) · [读档恢复](../../../raw/current/scripts/components/fumaroletool.lua#L160-L168) · [破损扣除和修复动画](../../../raw/current/scripts/standardcomponents.lua#L2102-L2119)

### R08 更正露西替换事件与触发那一斧的耐久顺序

`lucy.md:76` 将伍迪自身监听写成 `worked`。伍迪在普通形态且非幽灵时监听自身的 `working`，而 `worked` 发给被工作的目标。替换在 `working` 期间保存旧斧当前耐久并删除旧斧，之后 `BufferedAction` 因旧物品已失效跳过耗用。同步 `axe.md:22,73,125` 的速查、替换流程和陷阱。`lucy.md:79` 的退回说明也应限定特效路径，普通容器分支直接交还旧斧，读档时另可恢复已保存的计时。 [露西 Wiki](../../../wiki/reference/prefabs/lucy.md#L76-L79) · [斧头 Wiki](../../../wiki/reference/prefabs/axe.md#L73) · [伍迪监听](../../../raw/current/scripts/prefabs/woodie.lua#L873-L878) · [工作事件派发](../../../raw/current/scripts/components/workable.lua#L145-L152) · [替换流程](../../../raw/current/scripts/prefabs/woodie.lua#L681-L702) · [动作耗用](../../../raw/current/scripts/bufferedaction.lua#L26-L31) · [退回和读档](../../../raw/current/scripts/components/possessedaxe.lua#L196-L224)

### R09 区分当前已生成物品和存档重建

`pickaxe.md:74` 写成现有存档不应用示例设置，`shovel.md:62` 将现有物品与读档对象合写，`hammer.md:71` 承诺停用模组即恢复原值。读档重建先走 `SpawnPrefab` 及其 PostInit，再加载持久数据。`finiteuses` 恢复的是存档当前使用数，消耗参数和上限仍来自本次初始化。停用模组后配置可回到默认，已存的使用数可能高于默认上限。 [鹤嘴锄 Wiki](../../../wiki/reference/prefabs/pickaxe.md#L74) · [铲子 Wiki](../../../wiki/reference/prefabs/shovel.md#L62) · [锤子 Wiki](../../../wiki/reference/prefabs/hammer.md#L71) · [PostInit](../../../raw/current/scripts/mainfunctions.lua#L394-L407) · [存档生成](../../../raw/current/scripts/mainfunctions.lua#L464-L474) · [读持久数据](../../../raw/current/scripts/mainfunctions.lua#L498-L504) · [耐久保存和恢复](../../../raw/current/scripts/components/finiteuses.lua#L48-L66)

### R10 铲子的 inherentactions 有 Lua 消费者

`shovel.md:49,85` 将 `inherentactions` 写成 Lua 未消费、仅供引擎处理。`EntityScript:CanDoAction` 先检查该表，再回落到动作的 `_tool` 标签，气孔工具的 `OnUsedAsItem` 也调用此方法。可保留 DIG 动作候选使用工具标签的说明，并区分两个入口。 [Wiki](../../../wiki/reference/prefabs/shovel.md#L49) · [CanDoAction](../../../raw/current/scripts/entityscript.lua#L1819-L1835) · [气孔工具调用](../../../raw/current/scripts/components/fumaroletool.lua#L149-L152) · [DIG 候选](../../../raw/current/scripts/componentactions.lua#L2507-L2520)

### R11 多用斧镐的 toughwork 要先匹配工作动作

`multitool_axe_pickaxe.md:46,81` 将绝望石支柱作为不会反弹的实例。支柱要求 `HAMMER`，而多用斧镐只配置 `CHOP` 和 `MINE`，通用动作先核对目标动作。示例应换成动作匹配且已核验的对象，或把结论限定在默认 `CanBeWorked` 门禁，保留目标自定义回调接管的情况。 [Wiki](../../../wiki/reference/prefabs/multitool_axe_pickaxe.md#L46) · [多用斧镐动作](../../../raw/current/scripts/prefabs/axe_pickaxe.lua#L65-L75) · [支柱动作](../../../raw/current/scripts/prefabs/support_pillar.lua#L598-L603) · [工作入口](../../../raw/current/scripts/actions.lua#L1693-L1697) · [CanBeWorked](../../../raw/current/scripts/components/workable.lua#L189-L205)

### R12 鹤嘴锄皮肤注册可以在脚本中核验

`pickaxe.md:47,79,91` 将缺口写成皮肤数据不在 scripts 中。`prefabs/skinprefabs.lua` 已有黄金鹤嘴锄皮肤条目和 `base_prefab`、`init_fn` 等字段。保留贴图、动画资源或完整资产清单未核验的范围，区分脚本注册与资源内容。若正文加上皮肤注册证据，同步声明来源。 [Wiki](../../../wiki/reference/prefabs/pickaxe.md#L47) · [皮肤注册](../../../raw/current/scripts/prefabs/skinprefabs.lua#L6102-L6122)

### R13 露西与暗影收割者的正文引用未纳入 sources

`lucy.md:40,143` 引用 `screens/redux/scrapbookdata.lua`、`prefabs/evergreens.lua`、`prefabs/deciduoustrees.lua`、`components/spooked.lua`，四项未声明。`voidcloth_scythe.md:132` 引用 `mods.lua`，也未声明。补齐仍保留的证据文件，来源基线随修订处理。 [露西 Wiki](../../../wiki/reference/prefabs/lucy.md#L40) · [暗影收割者 Wiki](../../../wiki/reference/prefabs/voidcloth_scythe.md#L132)

## P3

### R14 亮茄工具保留了已建页面的待建提示

`lunarplant-tools.md:34,56` 的关联说明仍把相关亮茄剑及防具 Wiki 标成待建。可直链已存在的亮茄剑和战斗头盔族页。 [亮茄工具 Wiki](../../../wiki/reference/prefabs/lunarplant-tools.md#L34) · [亮茄剑](../../../wiki/reference/prefabs/sword_lunarplant.md) · [战斗头盔](../../../wiki/reference/prefabs/hats-armor.md)

## 覆盖结果

| 页面 | 检视结果 |
| --- | --- |
| `axe` | R08 |
| `pickaxe` | R09、R12 |
| `shovel` | R09、R10 |
| `hammer` | R09 |
| `multitool_axe_pickaxe` | R11 |
| `farm-hoe` | 本轮未发现需修订项 |
| `fumarole-tools` | R03、R06、R07 |
| `lunarplant-tools` | R14 |
| `lucy` | R08、R13 |
| `voidcloth_scythe` | R01、R02、R04、R05、R13 |

本轮共 3 项 P1、10 项 P2、1 项 P3。名称只做与本轮证据相关的定向核对，存量名称专项另行处理。依据为当前快照的静态源码，本轮未运行游戏或校验脚本。上述 9 页已按下列处置修复，装备范围另有 59 页尚待检视。

## 处置结果（2026-11-07）

R01～R14 逐条对照当前源码后全部成立，已修复 9 页：

- R01：`voidcloth_scythe` 执行链改为 `SCYTHE.fn` 先 `DoScythe`，动作成功后再由 `BufferedAction:Do` 调 `OnUsedAsItem` 扣耐久。
- R02：示例改为生成前改 `TUNING.VOIDCLOTH_SCYTHE_DAMAGE`／`_USES`，不再在 postinit 直接 `SetDamage(45)`；说明已有实例需 `SetMaxUses`＋`SetUses`。
- R03：`fumarole-tools` 示例补齐破损移除 `equippable`、修复恢复装备回调／动作／武器伤害，并在 `master_postinit` 配 `finiteuses` 4 次上限与耗尽移除。
- R04：影契描述改为至少 1 灵魂、成本累计、`FinishPortalHop` 结算，不再写成每次当场扣 1。
- R05：改为 `AddBonus("lunar_aligned", inst, 1.1)`，补第二参来源对象。
- R06：恢复攻击磨损补 `SetIgnoreCombatDurabilityLoss(false)`，并注明破损重建武器后要重设。
- R07：区分修复判定 `>=60`、档位显示 `>60` 与读档恢复不扣次。
- R08：露西替换改为伍迪监听自身 `working`；还原仅地面／装备栏分支播 `revert_fx`，普通容器直接归还；补读档计时；同步 `axe` 页。
- R09：镐／铲／锤的读档语义改为“读档重建先走 postinit，当前值按存档恢复”。
- R10：铲页改为 `EntityScript:CanDoAction` 确有 Lua 消费，区分 `inherentactions` 与 `DIG_tool` 标签两个入口。
- R11：多用斧镐删除 HAMMER 支撑柱示例，限定为 CHOP/MINE 动作匹配目标。
- R12：镐页补 `prefabs/skinprefabs.lua` 的皮肤注册证据，区分脚本注册与资源清单。
- R13：`lucy` 补 4 个来源、`voidcloth_scythe` 补 `mods.lua`；3 页 record 后 72 组 unchanged。
- R14：亮茄工具页改为直链 `sword_lunarplant` 与 `hats-armor`。

校验：9 页 `validate_page.py` 全部通过；`validate_source_links.py --strict` 691 引用 0 错 0 警；两条修改 Lua 示例 `parser_probe --syntax-only` 通过；全库 `validate_page --check-indexes` 通过（1,228 页、51 索引、1,177 知识页、0 问题）。未运行游戏。

## 首轮复检处置（2026-11-07）

首轮 14 项均已采纳，无驳回；复检 5 项 P2 是对修复的补充，逐条核对源码后全部采纳：

1. `fumarole-tools`：示例新增 `setup_components`，首次生成与修复共用 equippable／tool／weapon；`heatonuse` 改用 `FUMAROLEPICKAXE_HEAT_ON_USE`；注明基础工厂已加 inspectable／inventoryitem／floater。
2. `lucy`：补“触发替换的一斧不扣旧耐久”；`Inventory:FindItem` 改为物品栏／鼠标持有物／溢出容器（不扫装备栏）；sources 增 `components/workable.lua`、`components/inventory.lua`。
3. 读档：镐／铲／锤改为“仅存档写有 `uses` 时覆盖当前耐久；满耐久通常不写 `uses`，读档保留构造/PostInit 初始值”；暗影收割者停用说明区分默认配置与已存剩余耐久。
4. `multitool_axe_pickaxe`：被囚者石柱最低档 `AlwaysRecoil` 仍反弹，从“不反弹”列表分出。
5. `pickaxe`：普通镐皮肤改为 7 种，注明 `pickaxe_scythe` 与 `pickaxe_scythe_alt` 共享动画 build 但独立注册。

复检后校验：7 页 `validate_page.py` 全过；`validate_source_links.py --strict` 559 引用 0 错 0 警；修改后的 fumarole Lua 示例 `parser_probe` 通过；`lucy` source baseline record 后 29 组 unchanged；全库 `validate_page --check-indexes` 1,228 页／51 索引／1,177 知识页／0 问题。未运行游戏。

## 二次复检补充（2026-11-07）

- `multitool_axe_pickaxe`：tough work 豁免限定到默认 `SetRequiresToughWork(true)` 门禁；自定义 `SetShouldRecoilFn` 优先，返回 nil 才回落默认判定，返回 true 时按其返回的剩余工作量结算（不保证为 0）。只有反弹、工作量为 0 且持有者 SG 有 `recoilstate` 时推 `tooltooweak`。
- `lucy`「快速判断」改为：仅地面／装备栏分支尝试播还原特效，普通容器槽直接交还原斧，与正文一致。
- `fumarole-tools` 修正基础工厂归属：inspectable／inventoryitem／floater 由基础工厂提供，`equippable` 由示例 `setup_components` 添加。

上述 3 页 `validate_page.py` 全过（multitool 最终措辞复核后再次通过）；`validate_source_links.py --strict` 通过；全库 `validate_page --check-indexes` 1,228 页／51 索引／1,177 知识页／0 问题；`git diff --check` 干净。未运行游戏。

## 收口确认（2026-11-07）

本轮 10 页已完成独立检视和修复复核，未留待修项。装备范围累计 10/69 页，另有 59 页待检视。最后同步多用斧镐正文和工具叶索引，区分默认 tough 门禁、月裂水晶认可 tough 能力的自定义判定，以及被囚者石柱的反弹和振动反馈。

本次定向校验覆盖多用斧镐页和工具叶索引，结构通过，页与登记入口互链已核对。多用斧镐 85 条源码引用通过 strict 检查，0 错 0 警。未运行游戏，全量复验留待装备批次收口。
