---
title: "月矿采掘、旧冰刺与雕像转变（icespike_fx_1 族）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "三种月矿及水晶采掘、四个旧冰刺和两种雕像转变 FX 的通用表配置与生成入口。"
aliases: ["mining_moonglass_fx", "mining_charged_moonglass_fx", "mining_crystal_fx", "icespike_fx_1", "icespike_fx_2", "icespike_fx_3", "icespike_fx_4", "statue_transition", "statue_transition_2", "月矿采掘特效", "旧冰刺", "雕像转变特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/fx.lua", "stategraphs/commonstates.lua", "stategraphs/SGalterguardian_phase1.lua", "stategraphs/SGalterguardian_phase2.lua", "stategraphs/SGbird.lua", "components/lunarhailmanager.lua", "prefabs/lunarrift_crystals.lua", "prefabs/gestalt_guard_evolved.lua", "prefabs/atrium_statue.lua", "prefabs/statueruins.lua", "prefabs/minotaur.lua", "prefabs/monkey.lua", "prefabs/rabbit.lua", "prefabs/bunnyman.lua", "prefabs/clockwork_common.lua", "scenarios/magic_pumpkin.lua", "scenarios/chestfunctions.lua", "prefabs/shadowrift_portal.lua", "prefabs/sacred_chest.lua", "stategraphs/SGshadowwaxwell.lua", "components/grottowarmanager.lua", "stategraphs/SGdeerclops.lua", "prefabs/antlion_sinkhole.lua", "prefabs/eyeofterror_sinkhole.lua", "stategraphs/SGhermitcrab.lua", "stategraphs/SGmerm.lua", "stategraphs/SGwagdrone_rolling.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwobybig.lua", "stategraphs/SGwx78_common.lua", "stategraphs/SGwx78_possessedbody.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 月矿采掘、旧冰刺与雕像转变（icespike_fx_1 族）

## 快速判断

- 九个通用表注册分为采掘三名、旧冰刺四名和转变两名；动画代理的端及清理见 [[reference/mechanisms/generic-fx-factory#关键流程|通用 FX 工厂流程]]。表现之外的工作、伤害和生成由调用方负责。
- 简中 `.po` 定向查各注册名显示名键无命中；中文为效果描述而非官方译名。

## 注册与构造

各注册使用通用表工厂；未列 sound 或 fn 即该字段缺省。

| 注册名 | bank / build / anim | 差异与证据 |
|---|---|---|
| `mining_moonglass_fx` | `glass_mining_fx` / 同 bank / `anim` | 无 fn、无 sound。[条目](../../../raw/current/scripts/fx.lua#L1855-L1859) |
| `mining_charged_moonglass_fx` | `glass_mining_fx` / 同 bank / `anim` | fn 设置 LightOverride 0.1。[条目](../../../raw/current/scripts/fx.lua#L1861-L1865) |
| `mining_crystal_fx` | `mining_crystal_fx` / 同 bank / `anim` | 无 fn、无 sound。[条目](../../../raw/current/scripts/fx.lua#L3122-L3126) |
| `icespike_fx_1`、`icespike_fx_2`、`icespike_fx_3`、`icespike_fx_4` | 均为 `deerclops_icespike` / 同 bank；分别 `spike1`～`spike4` | 无 fn；sound 均为 `dontstarve/creatures/deerclops/ice_small`。[1](../../../raw/current/scripts/fx.lua#L656-L661)、[2](../../../raw/current/scripts/fx.lua#L663-L668)、[3](../../../raw/current/scripts/fx.lua#L670-L675)、[4](../../../raw/current/scripts/fx.lua#L677-L682) |
| `statue_transition` | `statue_ruins_fx` / 同 bank / `transform_nightmare` | 无 sound；alpha 0.6，fn `UsePointFiltering`。[条目](../../../raw/current/scripts/fx.lua#L496-L501)、[函数](../../../raw/current/scripts/fx.lua#L20-L22) |
| `statue_transition_2` | `die_fx` / `die` / `small` | 无 fn；黑色 tint、alpha 0.6，sound 为 `dontstarve/common/deathpoof`。[条目](../../../raw/current/scripts/fx.lua#L504-L511) |

## 关联机制导航

| 路径 | 触发、落点及依据 |
|---|---|
| 共用采矿 | `PlayMiningFX` 对有效且有 Transform 的目标按 frozen → moonglass/LunarBuildup → crystal → 默认的顺序选名，目标位置落点；声音由调用者的 SoundEmitter 播放，`nosound` 可抑制，音效判断不依赖目标有 Transform。[选名](../../../raw/current/scripts/stategraphs/commonstates.lua#L2448-L2467) |
| 共用采矿调用面 | 全树 15 处调用：蚁狮及眼球陷坑各一，隐士蟹、鱼人两处、暗影麦斯威尔、无人机两处、玩家两处、沃比两处、WX78 共通两处及附身身体一处。是否选中月矿／水晶仍由目标标签决定。[蚁狮](../../../raw/current/scripts/prefabs/antlion_sinkhole.lua#L148-L148)、[眼球](../../../raw/current/scripts/prefabs/eyeofterror_sinkhole.lua#L125-L125)、[隐士蟹](../../../raw/current/scripts/stategraphs/SGhermitcrab.lua#L1338-L1338)、[鱼人](../../../raw/current/scripts/stategraphs/SGmerm.lua#L396-L396)／[二](../../../raw/current/scripts/stategraphs/SGmerm.lua#L595-L595)、[暗影麦斯威尔](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L762-L762)、[无人机](../../../raw/current/scripts/stategraphs/SGwagdrone_rolling.lua#L217-L225)、[玩家](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5196-L5196)／[二](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5492-L5492)、[沃比](../../../raw/current/scripts/stategraphs/SGwobybig.lua#L745-L750)、[WX 共通](../../../raw/current/scripts/stategraphs/SGwx78_common.lua#L426-L426)／[二](../../../raw/current/scripts/stategraphs/SGwx78_common.lua#L459-L459)、[附身](../../../raw/current/scripts/stategraphs/SGwx78_possessedbody.lua#L1739-L1739) |
| 月冰雹碎片 | `lunarhailmanager` 按碎片是否有 Light 选择 charged 或普通月矿 FX，y=0 落点后移除碎片。[生成](../../../raw/current/scripts/components/lunarhailmanager.lua#L102-L108) |
| 其他月矿表现 | 天体英雄一阶段 `spawn_landfx`、二阶段旋转拖尾及船上刺击漏水点；鸟的 `PlayShardFx` 在目标处缩放至 0.5；大、小月裂隙水晶挖碎和进化启迪者弹体失手也落普通月矿 FX。[一阶段](../../../raw/current/scripts/stategraphs/SGalterguardian_phase1.lua#L229-L237)、[二阶段旋转](../../../raw/current/scripts/stategraphs/SGalterguardian_phase2.lua#L49-L54)、[船](../../../raw/current/scripts/stategraphs/SGalterguardian_phase2.lua#L702-L715)、[鸟](../../../raw/current/scripts/stategraphs/SGbird.lua#L17-L23)、[水晶](../../../raw/current/scripts/prefabs/lunarrift_crystals.lua#L201-L205)、[小水晶](../../../raw/current/scripts/prefabs/lunarrift_crystals.lua#L239-L243)、[弹体](../../../raw/current/scripts/prefabs/gestalt_guard_evolved.lua#L485-L490) |
| 雕像与宝箱 | 中庭雕像阶段变化、遗迹雕像阶段转换、未沉睡的远古守护者宝箱和场景箱陷阱均可同点生成两层，常按各自尺寸缩放；神圣宝箱接受时也生成两层。[中庭](../../../raw/current/scripts/prefabs/atrium_statue.lua#L46-L68)、[遗迹](../../../raw/current/scripts/prefabs/statueruins.lua#L89-L102)、[宝箱](../../../raw/current/scripts/prefabs/minotaur.lua#L642-L655)、[陷阱](../../../raw/current/scripts/scenarios/chestfunctions.lua#L79-L93)、[神圣宝箱](../../../raw/current/scripts/prefabs/sacred_chest.lua#L49-L64) |
| 其他转变 | 猴子状态变化可生成两层；强制梦魇变化另用 `_2` 和 `shadow_despawn`，兔子、兔人及发条怪的强制变化也用 `_2`；影子麦斯威尔 quickspawn 使用 `_2`。[猴子](../../../raw/current/scripts/prefabs/monkey.lua#L221-L249)、[兔](../../../raw/current/scripts/prefabs/rabbit.lua#L50-L59)、[兔人](../../../raw/current/scripts/prefabs/bunnyman.lua#L49-L58)、[发条](../../../raw/current/scripts/prefabs/clockwork_common.lua#L179-L189)、[quickspawn](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L318-L324) |
| 单层转变 | 遗迹雕像重生时 respawner 未休眠、暗影裂隙阶段 FX、南瓜场景及洞穴战争生成用 `statue_transition`；战争处的 `_2` 调用为注释。[重生](../../../raw/current/scripts/prefabs/statueruins.lua#L336-L340)、[裂隙](../../../raw/current/scripts/prefabs/shadowrift_portal.lua#L66-L73)、[南瓜](../../../raw/current/scripts/scenarios/magic_pumpkin.lua#L65-L70)、[战争](../../../raw/current/scripts/components/grottowarmanager.lua#L149-L156)、[墙](../../../raw/current/scripts/components/grottowarmanager.lua#L208-L226) |

## 常见陷阱与限制

- 四个 `icespike_fx_1`～`4` 在表中被标记 deprecated；全 scripts 检索 `icespike`、`ice.?spike` 和完整名称未发现其活跃生成点，这不证明模组不会调用。当前独眼巨鹿状态图使用独立的 `deerclops_icespike_fx`，不能把两种工厂当同一实体。[旧条目](../../../raw/current/scripts/fx.lua#L656-L661)、[现用](../../../raw/current/scripts/stategraphs/SGdeerclops.lua#L73-L75)
- 采矿动画三注册自身无 sound；`PlayMiningFX` 可另播采矿音，不能由条目推断场景无声。[选名与声音](../../../raw/current/scripts/stategraphs/commonstates.lua#L2448-L2467)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394；核对九项字段、工厂和列出的生成入口；全 scripts 检索完整名及 `icespike`、`ice.?spike`、`statue_transition`，并追踪 `PlayMiningFX` 字段选名及调用点。未覆盖动画资源内部及模组动态调用。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-table-world-index|通用表 FX 索引]]。
