---
title: "哥特灯笼光屑飘落特效（lantern_gothic_fx_held）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "哥特灯笼皮肤提灯的飘落光屑特效：与纸灯笼花瓣特效工厂近同构，符号与动画前缀不同，机制细节主页为 lantern_flower_fx_held。"
aliases: ["lantern_gothic_fx_held", "lantern_gothic_fx_ground", "哥特灯笼特效", "哥特灯笼光屑"]
tags: [prefab, fx, lifecycle, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/lantern_gothic_fx.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "prefabs/mininglantern.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 哥特灯笼光屑飘落特效（lantern_gothic_fx_held）

## 快速判断

- 职责与适用：哥特灯笼（官方皮肤译名，见下）提灯的飘落光屑特效组；同工厂注册 `lantern_gothic_fx_held` 与 `lantern_gothic_fx_ground`。生成时机、花瓣式本地粒子协议、地面 pre→loop→pst 与 KillFX 语义全部与 [[reference/prefabs/lantern_flower_fx_held#注册与构造|纸灯笼花瓣特效]] 同构，本页只记自身差异，不重复展开。
- 优先使用：查"哥特灯笼特效"选本页；同构机制与陷阱直接读 flower 主页对应章节。照明仍由独立 lanternlight 实体承担，本工厂无 AddLight。[照明实体](../../../raw/current/scripts/prefabs/mininglantern.lua#L206-L230)
- 重要副作用：与 flower 完全一致——非专服才调度本地光屑；ground 靠 animover 在 glow_loop 上自循环；KillFX 只置 killed 标记等 pst。[同构结论](../../../raw/current/scripts/prefabs/lantern_gothic_fx.lua#L15-L33)
- 修改边界：held 无 AnimState／无 KillFX；ground 安装 KillFX；persists=false。资源为 lantern_gothic 动态包。[工厂](../../../raw/current/scripts/prefabs/lantern_gothic_fx.lua#L71-L104)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lantern_gothic_fx_held` | 同 flower 结构；光屑 bank lantern_gothic_fx、build lantern、符号 glowflake、动画 glowfall1..7 | 发射器无 AnimState、不装 KillFX；本地光屑实体非联网。[构造](../../../raw/current/scripts/prefabs/lantern_gothic_fx.lua#L71-L104)、[光屑](../../../raw/current/scripts/prefabs/lantern_gothic_fx.lua#L35-L58) |
| `lantern_gothic_fx_ground` | 同 flower 结构；glow_pre 起播，POPULATING 直入 glow_loop 随机帧，安装 KillFX | 联网动画实体；pre／loop／pst 未显式传循环参数，loop 靠 animover 重播。[构造](../../../raw/current/scripts/prefabs/lantern_gothic_fx.lua#L119-L151)、[注册](../../../raw/current/scripts/prefabs/lantern_gothic_fx.lua#L153-L154) |

皮肤键 `STRINGS.SKIN_NAMES.lantern_gothic` 为"哥特灯笼"（官方译名）；标题"光屑飘落特效"按 glowfall 动画名与 glowflake 符号写的已核验效果描述，非官方译名。[译名](../../../raw/current/scripts/languages/chinese_s.po#L398304-L398307)。皮肤 `lantern_gothic` 的 fx_prefab 两项即本组注册名，rarity_modifier 为 Woven。[皮肤定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L7356-L7366)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 同构协议 | 调度表、移动门控、pre→loop→pst、宿主 lantern_on／off 协议与 enterlimbo 还原与 flower 页逐条相同（diff 已核验） | 端分界同 flower：专服无本地光屑；主模拟才装 KillFX | [[reference/prefabs/lantern_flower_fx_held#关联机制导航]]；[宿主协议](../../../raw/current/scripts/prefabskin.lua#L3431-L3473) |
| 自身参数 | 覆盖符号 `glowflake`（来自皮肤 build lantern_gothic）；动画前缀 `glowfall`；bank `lantern_gothic_fx` | 花瓣级覆盖在各光屑实体上做（held 无 AnimState，宿主跳过） | [覆盖](../../../raw/current/scripts/prefabs/lantern_gothic_fx.lua#L46-L48)、[动画名](../../../raw/current/scripts/prefabs/lantern_gothic_fx.lua#L52) |
| 照明分工 | 本组 FX 无 Light；照明在 lanternlight 实体 | 独立联网实体 | [照明实体](../../../raw/current/scripts/prefabs/mininglantern.lua#L206-L230) |
| 资源 | anim/dynamic/lantern_gothic.zip＋PKGREF lantern_gothic.dyn | 声明与使用已核验，包内容未解包 | [资源声明](../../../raw/current/scripts/prefabs/lantern_gothic_fx.lua#L1-L5) |

## 典型接入

### 复用光屑流或调整密度

协议、调度常数与移动门控与 [[reference/prefabs/lantern_flower_fx_held#典型接入|flower 主页]] 完全一致，不重复展开；套用时只替换本页参数：bank `lantern_gothic_fx`、覆盖符号 `glowflake`、动画段 `glowfall1..7` 与 `glow_pre／glow_loop／glow_pst`、动态包 lantern_gothic。这些动画段在 lantern_gothic 动态包内的实际内容未解包，自备资源前先确认段名存在。[参数证据](../../../raw/current/scripts/prefabs/lantern_gothic_fx.lua#L46-L52)

## 常见陷阱与限制

- 同族通用陷阱（花瓣非联网、KillFX 非立即、held 符号覆盖跳过、enterlimbo 还原）见 [[reference/prefabs/lantern_flower_fx_held#常见陷阱与限制|flower 主页陷阱节]]，不在此重复。
- **边界：本皮肤无 alt 变体**：flower／winter 各有复用同组 FX 的 alt 皮肤，gothic 没有；换肤清理由 lantern_clear_fn 统一处理。[clear_fn](../../../raw/current/scripts/prefabskin.lua#L3501-L3515)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；完整 155 行工厂、gothic 皮肤定义与 fx_prefab 映射、与 flower 逐对 diff 同构、宿主 lantern 协议与照明实体已核验。未覆盖：动画包内容未解包、最终视觉、光屑重播时序实测。
- 游戏运行：未运行。
- 结构检查：validate_page 与 validate_source_links --strict 通过（实际结果见本批交付报告）。

返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
