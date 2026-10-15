---
title: "大霜鲨攻击冰面特效族（sharkboi_swipe_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "大霜鲨（Frostjaw）攻击的五件套冰面表现：挥击地面动画、鱼雷冰犁、冰面砸痕、入水/出水冲击与冰洞；全族无玩法组件，伤害由宿主 SG 的 AOE 结算。"
aliases: ["sharkboi_swipe_fx", "sharkboi_iceplow_fx", "sharkboi_iceimpact_fx", "sharkboi_icetrail_fx", "sharkboi_icehole_fx", "大霜鲨特效", "冰犁特效", "冰面裂纹特效", "sharkboi fx"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/sharkboi_fx.lua", "stategraphs/SGsharkboi.lua", "prefabs/sharkboi.lua", "prefabs/sharkboi_icespike.lua", "prefabs/crabking_icewall.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 大霜鲨攻击冰面特效族（sharkboi_swipe_fx）

## 快速判断

- 职责与适用：大霜鲨（官方译名，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L348680-L348682)）战斗的五件套冰面表现：挥击地面动画、鱼雷冲刺两侧冰犁、冰面裂纹拖尾、入水／出水冰花冲击与冰洞。全族只有 FX／NOCLICK 标签，无玩法组件，伤害由宿主 SG 的 AOE 结算。[注册](../../../raw/current/scripts/prefabs/sharkboi_fx.lua#L204-L208)、[宿主结算](../../../raw/current/scripts/stategraphs/SGsharkboi.lua#L856)
- 优先使用：查"冰面滑行痕迹／冰犁／冰洞动画"选本页；宿主攻击节奏与 AOE 见 SGsharkboi。
- 重要副作用：仅表现——不挂 combat／不查实体；冰犁与拖尾带随机变体、镜像与缩放。
- 修改边界：swipe／iceimpact／icehole 播单次动画后 animover 自删；iceplow／icetrail 由主模拟定时（1.35-1.65 秒／2 秒）切 pst 再自删。动画均未显式传循环参数（PushAnimation false），`_idle` 段是否重复未核验。[iceplow 生命周期](../../../raw/current/scripts/prefabs/sharkboi_fx.lua#L83-L94)、[icetrail 生命周期](../../../raw/current/scripts/prefabs/sharkboi_fx.lua#L156-L168)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `sharkboi_swipe_fx` | swipe_fn：Transform＋AnimState＋Network → FX／NOCLICK、四面向 → bank/build `sharkboi_swipe_fx` 播 `atk1`、OnGround 背景层 SortOrder 3 → SetPristine → 主模拟 Reverse（播 atk2）、persists=false、animover Remove | 宿主反手调 Reverse。[工厂](../../../raw/current/scripts/prefabs/sharkboi_fx.lua#L22-L53)、[Reverse](../../../raw/current/scripts/prefabs/sharkboi_fx.lua#L18-L20) |
| `sharkboi_iceplow_fx` | iceplow_fn：Transform＋AnimState＋Network → 同标签 → bank/build `sharkboi_iceplow_fx` 播 `iceplow{1-2}_pre` 后 Push `iceplow{n}_idle`（不循环）、随机缩放 0.6-1.0 与 50% 镜像 → SetPristine → 主模拟 1.35+rand×0.3 秒后 KillFX 播 `iceplow{n}_pst`，animover Remove | 变体／缩放／镜像在 master 段决定。[工厂](../../../raw/current/scripts/prefabs/sharkboi_fx.lua#L62-L95) |
| `sharkboi_iceimpact_fx` | iceimpact_fn：同框架，播 `ice_impact` 单次动画（FinalOffset 1），animover Remove | 与 iceplow 共用同一动画 zip。[工厂](../../../raw/current/scripts/prefabs/sharkboi_fx.lua#L99-L124) |
| `sharkboi_icetrail_fx` | icetrail_fn：同框架，bank/build `sharkboi_trail` 播 `crack{1-3}_pre` 后 Push `crack{n}_idle`（不循环）、50% 镜像、OnGround 背景层；2 秒后 KillFX 播 `crack{n}_pst` | 变体 3 选 1；朝向由宿主旋转设置。[工厂](../../../raw/current/scripts/prefabs/sharkboi_fx.lua#L133-L169) |
| `sharkboi_icehole_fx` | icehole_fn：同框架，播 `icehole_pst` 单次动画，背景层但**非 ground oriented**（源码注释），animover Remove | 与 icetrail 共用 zip。[工厂](../../../raw/current/scripts/prefabs/sharkboi_fx.lua#L173-L200) |

五注册均无 NAMES 中文条目；标题为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| swipe 生成：attack1／2／3 三连 | SpawnSwipeFX 父挂宿主、前向偏移 2，attack2 传 reverse；注释：提前 3 帧生成防网络丢帧 | 同态 FrameEvent(16) DoArcAttack(SWIPE_OFFSET 2, RADIUS 3.5, ARC 240°) 用宿主 combat 结算（默认伤害 75） | [常量](../../../raw/current/scripts/stategraphs/SGsharkboi.lua#L383-L385)、[attack1](../../../raw/current/scripts/stategraphs/SGsharkboi.lua#L849-L863)、[attack2 反手](../../../raw/current/scripts/stategraphs/SGsharkboi.lua#L914-L924)、[attack3](../../../raw/current/scripts/stategraphs/SGsharkboi.lua#L1050-L1058)、[伤害值](../../../raw/current/scripts/tuning.lua#L6980)、[combat 设值](../../../raw/current/scripts/prefabs/sharkboi.lua#L761) |
| iceplow 生成：torpedo 鱼雷 | torpedo onupdate 每 3 帧在身体两侧 ±2 落两只冰犁，另每 2 帧落拖尾 | 冲刺期间持续 AOE 攻击＋掘地（DoAOEAttackAndDig） | [鱼雷 onupdate](../../../raw/current/scripts/stategraphs/SGsharkboi.lua#L1306-L1324)、[AOE](../../../raw/current/scripts/stategraphs/SGsharkboi.lua#L1306) |
| icetrail 生成 | torpedo 每 2-3 帧（traildelay 计数）落一枚，旋转复制宿主朝向 | 鱼雷路径视觉拖尾 | [SpawnIceTrailFX](../../../raw/current/scripts/stategraphs/SGsharkboi.lua#L428-L433)、[计数](../../../raw/current/scripts/stategraphs/SGsharkboi.lua#L1319-L1324) |
| iceimpact 生成 | dive_jump_pre 出水第 3 帧落点冲击；dive_dig_pre 入水时再落一次 | 配合 DoJumpOutShake／DoImpactShake 屏震 | [出水](../../../raw/current/scripts/stategraphs/SGsharkboi.lua#L1655-L1667)、[入水](../../../raw/current/scripts/stategraphs/SGsharkboi.lua#L1763-L1769) |
| icehole 生成 | dive_jump 非 fastdig 落冰洞；dive_dig_pst 出土时落冰洞并先 Pause、第 10 帧 Resume 播放 | fastdig 标记路径不出洞 | [dive_jump](../../../raw/current/scripts/stategraphs/SGsharkboi.lua#L1702-L1706)、[出土暂停/续播](../../../raw/current/scripts/stategraphs/SGsharkboi.lua#L1856-L1881) |
| 依赖声明 | sharkboi prefab 依赖表声明五注册名 | 声明保证资源加载，不代表即时生成 | [依赖表](../../../raw/current/scripts/prefabs/sharkboi.lua#L13-L19) |
| 资源复用 | `sharkboi_iceplow_fx` 的 build／`iceplow{n}_pre` 动画被鲨鱼冰刺（sharkboi_icespike）与蟹王冰墙（crabking_icewall）复用 | 只借动画资源，不生成本页实体 | [冰刺](../../../raw/current/scripts/prefabs/sharkboi_icespike.lua#L188-L190)、[冰墙](../../../raw/current/scripts/prefabs/crabking_icewall.lua#L41-L45) |

## 典型接入

仿宿主组合可给冲刺型生物做"路径冰面表现"：FX 提供 pre→idle→pst 三段与随机变体，SG 侧用延迟计数按帧落点并在结束时切 pst。iceplow／icetrail 的 pst 收尾由主模拟 DoTaskInTime 触发（[iceplow](../../../raw/current/scripts/prefabs/sharkboi_fx.lua#L57-L60)、[icetrail](../../../raw/current/scripts/prefabs/sharkboi_fx.lua#L128-L131)），animover 才自删。

## 常见陷阱与限制

- **误区：冰犁／拖尾有减速或伤害**：全族无玩法组件；减速与伤害都在宿主 SG 的 AOE 里，FX 只是地面动画。[AOE](../../../raw/current/scripts/stategraphs/SGsharkboi.lua#L1306)
- **边界：变体在 master 段随机**：iceplow／icetrail 的编号、缩放、镜像在主模拟创建时才决定，客户端壳看到的是复制后的动画命令。[随机段](../../../raw/current/scripts/prefabs/sharkboi_fx.lua#L83-L92)
- **边界：icehole 不贴地**：源码注释明确 NOT ground oriented（保留立体感），复用时别照抄背景层参数加 OnGround。[注释](../../../raw/current/scripts/prefabs/sharkboi_fx.lua#L188)
- **边界：pooled 无、定时收尾**：iceplow／icetrail 靠 DoTaskInTime 收尾，宿主中断攻击不会提前回收已落地的冰面动画。[定时](../../../raw/current/scripts/prefabs/sharkboi_fx.lua#L92-L94)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；209 行工厂全文（五工厂全生命周期）、SGsharkboi 三连击／鱼雷／入水出土全生成点、宿主依赖表与 combat 设值、冰刺与冰墙资源复用、tuning 数值、译名检索已核验；动画 `_idle` 段实际是否重复、冰洞 Pause 的客户端表现未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-creature-index|生物、Boss 与遭遇演出]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
