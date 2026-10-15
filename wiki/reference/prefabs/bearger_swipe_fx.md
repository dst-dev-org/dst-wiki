---
title: "熊獾挥击地面特效（bearger_swipe_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "熊獾与突变熊獾攻击的地面抓击动画：同一工厂两注册，仅饱和度与发光差异；无玩法组件，伤害由宿主 SG 的 AOE 结算。"
aliases: ["bearger_swipe_fx", "mutatedbearger_swipe_fx", "bearger_fx", "熊獾挥击特效", "熊獾攻击特效", "突变熊獾挥击特效", "bearger swipe"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/bearger_fx.lua", "prefabs/bearger.lua", "stategraphs/SGbearger.lua", "languages/chinese_s.po"]
size_class: simple
reliability: verified
validation: source-reviewed
status: active
---

# 熊獾挥击地面特效（bearger_swipe_fx）

## 快速判断

- 职责与适用：熊獾（官方译名，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L338620-L338622)）与突变熊獾攻击时的地面抓击动画：联网 FX 壳＋主模拟 Reverse 接口，无任何玩法组件，伤害由宿主攻击状态用 combat 单独结算。[注册](../../../raw/current/scripts/prefabs/bearger_fx.lua#L49-L50)、[宿主结算](../../../raw/current/scripts/stategraphs/SGbearger.lua#L814)
- 优先使用：查"熊獾攻击地面动画／挥击特效换皮"选本页；宿主侧生成与伤害结算见 SGbearger。
- 重要副作用：仅表现——不挂 combat／不查实体；`Reverse()` 换播 atk2 实现反手挥击。
- 修改边界：动画 `atk1` 未显式传循环参数，播完 animover 即 Remove；突变变体与普通版共用同一动画资源，只有饱和度／发光参数不同。[工厂参数](../../../raw/current/scripts/prefabs/bearger_fx.lua#L10-L30)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `bearger_swipe_fx` | MakeFX(name, saturation, lightoverride) → fn：Transform＋AnimState＋Network → FX／NOCLICK、八面向 → bank/build `bearger_mutated_actions_fx` 播 `atk1`、OnGround 背景层 SortOrder 3、饱和度 0 → SetPristine → 客户端壳早退 → 主模拟 Reverse、persists=false、animover Remove | 普通熊獾用；去色显示。[工厂](../../../raw/current/scripts/prefabs/bearger_fx.lua#L10-L47)、[注册](../../../raw/current/scripts/prefabs/bearger_fx.lua#L49) |
| `mutatedbearger_swipe_fx` | 同工厂同初始化，饱和度 1、LightOverride 0.1 | 突变熊獾用；全彩＋轻微自发光。[注册](../../../raw/current/scripts/prefabs/bearger_fx.lua#L50) |

两个注册名共用同一动画 zip（`anim/bearger_mutated_actions_fx.zip`，普通熊獾也用它）；均无 NAMES 中文条目，"熊獾"为宿主官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：SGbearger SpawnSwipeFX | `SpawnPrefab(inst.swipefx)` 后父挂宿主、前向偏移；reverse 时调 `Reverse()` 播 atk2；源码注释：提前 3 帧生成（带 3 空白前导帧）防网络时序丢帧 | 主模拟按宿主字段 `inst.swipefx` 选注册名：普通 bearger.lua L565、突变 L812 | [生成函数](../../../raw/current/scripts/stategraphs/SGbearger.lua#L350-L360)、[普通版字段](../../../raw/current/scripts/prefabs/bearger.lua#L564-L565)、[突变版字段](../../../raw/current/scripts/prefabs/bearger.lua#L810-L812) |
| 生成时机：攻击态 | attack 第 29 帧（offset 1）、attack_combo1／attack_combo2（reverse）／attack_combo1a 第 29 帧 | 同态 FrameEvent(32) 的 DoArcAttack／DoComboArcAttack 才是伤害结算，FX 不参与 | [attack](../../../raw/current/scripts/stategraphs/SGbearger.lua#L809-L814)、[combo1](../../../raw/current/scripts/stategraphs/SGbearger.lua#L881-L886) |
| 回收 | animover 自删；各攻击态退出时 KillSwipeFX 兜底 Remove | 主模拟；`persists=false` 不入存档 | [自删](../../../raw/current/scripts/prefabs/bearger_fx.lua#L41)、[兜底](../../../raw/current/scripts/stategraphs/SGbearger.lua#L362-L367) |
| 依赖声明 | beargar／mutatedbearger 的 prefab 依赖表声明两注册名 | 声明保证资源先行加载，不代表即时生成 | [依赖表](../../../raw/current/scripts/prefabs/bearger.lua#L23-L38) |

## 典型接入

宿主生物可仿照 `inst.swipefx = "bearger_swipe_fx"` 字段＋SG 侧 SpawnSwipeFX 的组合换用/新增地面挥击动画；换皮只需换 bank/build 与注册名，Reverse 双动画约定（atk1/atk2）需一并保留。[字段](../../../raw/current/scripts/prefabs/bearger.lua#L564-L565)、[SG 生成](../../../raw/current/scripts/stategraphs/SGbearger.lua#L350-L360)

## 常见陷阱与限制

- **误区：突变变体有独立动画资源**：两注册共用 `bearger_mutated_actions_fx` 一个 zip，差异只有 SetSaturation／SetLightOverride 两个参数。[工厂入参](../../../raw/current/scripts/prefabs/bearger_fx.lua#L10)、[参数落地](../../../raw/current/scripts/prefabs/bearger_fx.lua#L29-L30)、[注册](../../../raw/current/scripts/prefabs/bearger_fx.lua#L49-L50)
- **边界：Reverse 必须成对提供**：SG 侧 reverse 参数会直接调用 FX 的 Reverse；缺少该方法的实体传入 reverse 会报错。[调用](../../../raw/current/scripts/stategraphs/SGbearger.lua#L356-L358)
- **边界：提前 3 帧生成**：动画超短靠网络时机容易丢帧，官方在生成注释中写明带 3 空白前导帧；复用该模式时不要删前导帧。[注释](../../../raw/current/scripts/stategraphs/SGbearger.lua#L351-L352)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；51 行工厂全文、SGbearger 生成函数与四个攻击态生成点、宿主 swipefx 字段与依赖表、译名检索已核验；动画实际帧数内容、突变熊獾 combo 全流程伤害细节未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
