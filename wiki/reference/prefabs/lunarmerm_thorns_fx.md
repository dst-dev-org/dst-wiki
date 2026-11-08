---
title: "鱼人荆棘反伤与翻土标记（lunarmerm_thorns_fx）"
created: "2026-10-29"
updated: "2026-11-08"
type: prefab
summary: "月裔鱼人受击时炸开的扩张荆棘环：主模拟逐帧 AOE 结算 40 点伤害（玩家减半）、沿用宿主敌我判定；同文件另有鱼人协助翻土的 merm_soil_marker 占位标记。"
aliases: ["lunarmerm_thorns_fx", "merm_soil_marker", "merm_fx", "鱼人荆棘", "荆棘反伤特效", "翻土标记", "lunar merm thorns"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/merm_fx.lua", "prefabs/merm.lua", "stategraphs/SGmerm.lua", "prefabs/wurt.lua", "prefabs/wurt_terraform_item.lua", "brains/braincommon.lua", "componentutil.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 鱼人荆棘反伤与翻土标记（lunarmerm_thorns_fx）

## 快速判断

- 职责与适用：带 `lunarminion` 标签的鱼人变体（`merm_lunar`／`mermguard_lunar`）受击时在其身上生成的扩张荆棘环：联网 FX＋主模拟 updatelooper 逐帧 AOE `GetAttacked`，是真实伤害源而不是纯视觉。同文件注册 `merm_soil_marker`，是鱼人协助翻土时的落点占位标记，无任何表现。[注册](../../../raw/current/scripts/prefabs/merm_fx.lua#L145-L146)、[伤害](../../../raw/current/scripts/prefabs/merm_fx.lua#L11-L64)
- 优先使用：查"受击反伤／荆棘环 AOE"选 `lunarmerm_thorns_fx`；查鱼人协助翻土选 `merm_soil_marker`；宿主的变异入口见关联机制导航。
- 重要副作用：判定半径初始 0.75，每次更新先 +0.75（首次结算 1.5），到 MAXRANGE 3 后停止伤害更新；每目标整轮只结算一次（命中即记入 `ignore`）；伤害 40，对玩家按 50% 结算。[环与限次](../../../raw/current/scripts/prefabs/merm_fx.lua#L6-L27)、[玩家系数](../../../raw/current/scripts/componentutil.lua#L1611-L1614)、[数值](../../../raw/current/scripts/tuning.lua#L8346-L8347)
- 修改边界：动画只播一次 `idle`（未显式传循环参数），animover 即 Remove；能否打玩家由宿主阵营与 PVP 决定；`merm_soil_marker` 无客户端交互。[动画与回收](../../../raw/current/scripts/prefabs/merm_fx.lua#L92-L106)、[SetFXOwner](../../../raw/current/scripts/prefabs/merm_fx.lua#L66-L75)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lunarmerm_thorns_fx` | MakeLunarMermThornsFx(name, anim, damage) → fxfn：Transform＋AnimState＋Network → FX／thorny 标签、四面向 → bank/build `merm_lunar_spike_fx` 播 `idle`、FinalOffset -1、死亡公告名挂 `bramblefx` → SetPristine → 客户端壳早退 → 主模拟 updatelooper＋animover Remove、persists=false、damage／range／ignore 初始化、暴露 SetFXOwner | 无 SG／Brain、无音效；伤害取 TUNING 键名参数。[工厂](../../../raw/current/scripts/prefabs/merm_fx.lua#L77-L120) |
| `merm_soil_marker` | soilmarkerfn：Transform＋Network → NOBLOCK／merm_soil_blocker 标签 → SetPristine → 主模拟 persists=false、5 秒后 Remove | 纯逻辑标记：无 AnimState、无渲染资源。[标记](../../../raw/current/scripts/prefabs/merm_fx.lua#L123-L142) |

两个注册名均无 NAMES 中文条目（po 中检索 LUNARMERM 无命中）；标题中文为已核验效果描述，非官方译名。宿主"鱼人"为官方译名，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L344675-L344677)。

## 关联机制导航

### 荆棘伤害（lunarmerm_thorns_fx）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 伤害结算：OnUpdateThorns | 每帧环半径 +0.75，以 `range+3` 找 `_combat` 实体，命中距离计入目标物理半径；宿主路径按 owner combat 的 CanTarget／IsAlly（含 leader 判定）过滤后 `GetAttacked(attacker, 伤害)` | 目标是宿主自己的 follower 时 attacker 记为 FX 自身，避免宿主被记为攻击者；命中后 `ignore` 永久跳过该目标 | [结算](../../../raw/current/scripts/prefabs/merm_fx.lua#L11-L64)、[宿主过滤](../../../raw/current/scripts/prefabs/merm_fx.lua#L28-L40)、[attacker 选择](../../../raw/current/scripts/prefabs/merm_fx.lua#L36-L39) |
| 无宿主路径 | owner 为 nil 时走 `CanBeAttacked` 分支（源码注释：留给荆棘陷阱类非穿戴物）；非 PVP 跳过玩家 follower，除非其正以玩家为目标 | 当前原版只有鱼人生成并传宿主；nil 路径是保留入口 | [nil 路径](../../../raw/current/scripts/prefabs/merm_fx.lua#L41-L56) |
| 生成者：SGmerm hit 态 | 受击进入 hit 态第 0 帧，`lunarminion` 才调 `inst:DoThorns()`：SpawnPrefab＋SetFXOwner(宿主)＋仙人掌音效 | 主模拟；无 `lunarminion` 的普通鱼人受击不生成 | [hit 态](../../../raw/current/scripts/stategraphs/SGmerm.lua#L1027-L1045)、[DoThorns](../../../raw/current/scripts/prefabs/merm.lua#L796-L801) |
| 宿主：月裔鱼人 | `lunarminion`／`lunar_aligned` 标签在 lunar_merm_common（mermguard 同）添加；变异由 DoLunarMutation 生成 `merm_lunar`／`mermguard_lunar` 并转移血量／物品／跟随 | 沃特月辉忠诚技能相关检查见 merm.lua L388；merm prefab 依赖表声明本 FX 与标记 | [标签](../../../raw/current/scripts/prefabs/merm.lua#L1707-L1708)、[变异](../../../raw/current/scripts/prefabs/merm.lua#L804-L838)、[依赖表](../../../raw/current/scripts/prefabs/merm.lua#L44-L56) |
| 关联 debuff：wurt_merm_planar | SetFXOwner 读宿主 `HasDebuff("wurt_merm_planar")` 决定自身是否加 bloom；该 debuff 由沃特的月耀改造物（wurt_terraform_item，中文描述非官方译名）施加给鱼人随从，附加时宿主同样加 bloom 并设 planardamage | 仅月耀改造路径；buff 持续 8*seg_time | [FX 端判断](../../../raw/current/scripts/prefabs/merm_fx.lua#L72-L74)、[上 debuff](../../../raw/current/scripts/prefabs/wurt_terraform_item.lua#L284-L288)、[附加效果](../../../raw/current/scripts/prefabs/wurt.lua#L740-L744)、[时长](../../../raw/current/scripts/tuning.lua#L8334) |
| 数值：tuning | 伤害 40、玩家系数 0.5、buff 8*seg_time；环最大 3 为文件内 MAXRANGE 常量 | 单位为每击伤害与秒；玩家系数只对玩家目标生效 | [数值](../../../raw/current/scripts/tuning.lua#L8346-L8347)、[MAXRANGE](../../../raw/current/scripts/prefabs/merm_fx.lua#L6) |

### 翻土占位标记（merm_soil_marker）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 标记消费者：翻土协助 | BrainCommon 的 TILL 协助在 FindTillPosition 选点后落 `merm_soil_marker`（存在 5 秒）；CollectTillSites 统计泥地时以 SOILMUSTNOT 排除 merm_soil_blocker／farm_debris／NOBLOCK | 鱼人 follower 协助领队 tilling 时；标记带 NOBLOCK，不做实体阻挡 | [TILL 选点](../../../raw/current/scripts/brains/braincommon.lua#L396-L404)、[排除表](../../../raw/current/scripts/brains/braincommon.lua#L205)、[选点查询](../../../raw/current/scripts/brains/braincommon.lua#L226-L242) |

## 典型接入

### 复用"受击反伤荆棘环"

- 前提与选择：主模拟生成后立即 `SetFXOwner(宿主)`；宿主决定落点、敌我过滤与玩家伤害系数，不传宿主则走无宿主分支。
- 生效与撤销：生成当帧起逐帧扩张结算，`animover` 自动 Remove，无需手工清理；`persists=false` 不入存档。
- 原版依据：SGmerm hit 态在受击第 0 帧生成即完整调用链。

```lua
local fx = SpawnPrefab("lunarmerm_thorns_fx")
fx:SetFXOwner(mycreature)  -- 定位＋敌我判定＋玩家伤害系数
```

- 调整与验证：改伤害需换 TUNING 键（工厂第三参）；`ignore` 表意味着同一只 FX 每目标只打一次，多段伤害需再次生成。

## 常见陷阱与限制

- **误区：名字带 fx 就是纯视觉**：主模拟挂 updatelooper 逐帧 `GetAttacked`，是真实伤害源，击杀归属按 attacker 记账。[结算](../../../raw/current/scripts/prefabs/merm_fx.lua#L11-L64)
- **边界：无宿主时玩家会被打**：`canhitplayers` 初始为 true；传宿主后才改为"宿主非玩家或开 PVP"判断。[初始值](../../../raw/current/scripts/prefabs/merm_fx.lua#L111)、[宿主判断](../../../raw/current/scripts/prefabs/merm_fx.lua#L69)
- **边界：每目标一次**：命中即写入 `ignore`，后续帧不再结算；环扩张阶段可覆盖新目标；达到 3 后已移除更新回调，剩余动画不继续判伤。[ignore](../../../raw/current/scripts/prefabs/merm_fx.lua#L35)
- **边界：伤害结算在主模拟**：专用服务器照常结算（无渲染不影响）；表现层无客户端专用任务，bloom 由宿主 debuff 决定。[SetFXOwner](../../../raw/current/scripts/prefabs/merm_fx.lua#L66-L75)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；146 行工厂全文（荆棘环结算、SetFXOwner、soil marker）、SGmerm hit 态、DoThorns／DoLunarMutation、wurt planar debuff 链、翻土协助与 SOILMUSTNOT、tuning 数值、译名检索已核验；动画实际帧数、客户端 bloom 显示、merm_soil_blocker 在泥地统计查询中的实际命中路径未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-creature-index|生物、Boss 与遭遇演出]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 6 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-11）。
