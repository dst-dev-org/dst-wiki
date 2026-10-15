---
title: "拾荒疯猪挥击地面特效（daywalker2_swipe_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "拾荒疯猪（Scrappy Werepig）挥物攻击的地面横扫动画：单注册、Reverse 反手、提前 3 帧生成防丢帧；无玩法组件，伤害由宿主 SG 的弧形 AOE 结算。"
aliases: ["daywalker2_swipe_fx", "daywalker2_fx", "拾荒疯猪挥击特效", "疯猪挥击特效", "daywalker swipe"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/daywalker2_fx.lua", "stategraphs/SGdaywalker2.lua", "prefabs/daywalker2.lua", "tuning.lua", "languages/chinese_s.po"]
size_class: simple
reliability: verified
validation: source-reviewed
status: active
---

# 拾荒疯猪挥击地面特效（daywalker2_swipe_fx）

## 快速判断

- 职责与适用：拾荒疯猪（官方译名，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L341390-L341392)）`attack_swing` 挥物攻击的地面横扫动画：联网 FX 壳＋主模拟 Reverse 接口，无玩法组件，伤害由宿主同态弧形 AOE 用 combat 结算（本次攻击专值 `DAYWALKER2_ATTACK_SWING_DAMAGE` = 125）。[注册](../../../raw/current/scripts/prefabs/daywalker2_fx.lua#L43)、[攻击态](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L1269-L1297)
- 优先使用：查"疯猪挥击动画／地面横扫特效换皮"选本页；生成与伤害链见 SGdaywalker2。
- 重要副作用：仅表现；`Reverse()` 换播 atk2——本页唯一生成点就传了 reverse=true。
- 修改边界：动画 `atk1` 未显式传循环参数，播完 animover 即 Remove；状态退出时宿主兜底 Remove。[自删](../../../raw/current/scripts/prefabs/daywalker2_fx.lua#L37-L40)、[兜底](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L431-L437)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `daywalker2_swipe_fx` | swipe_fn：Transform＋AnimState＋Network → FX／NOCLICK、四面向 → bank/build `daywalker_swipe_fx` 播 `atk1`、OnGround 背景层 SortOrder 3 → SetPristine → 客户端壳早退 → 主模拟 Reverse（播 atk2）、persists=false、animover Remove | 单注册单形态；无音效、无资源池。[工厂](../../../raw/current/scripts/prefabs/daywalker2_fx.lua#L10-L41) |

无 NAMES 中文条目；标题"拾荒疯猪"为宿主官方译名（DAYWALKER2）。宿主 prefab 依赖表声明本注册名，[依赖](../../../raw/current/scripts/prefabs/daywalker2.lua#L19)。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：SGdaywalker2 SpawnSwipeFX | `SpawnPrefab("daywalker2_swipe_fx")` 后父挂宿主、前向偏移 1，传 reverse 调 Reverse()；源码注释：提前 3 帧生成（带 3 空白前导帧）防网络时序丢帧 | 主模拟；宿主 SG 持 statemem.fx 引用 | [生成函数](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L421-L429) |
| 生成时机：attack_swing 态 | 第 12 帧 SpawnSwipeFX(inst, 1, true)（反手）；第 16 帧 DoArcAttack(1, 4.5, 240°) 结算伤害 | onenter 把 combat 默认伤害临时设为 125，onexit 恢复 DAYWALKER_DAMAGE 并 KillSwipeFX 兜底 | [伤害设值](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L1284)、[生成与结算](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L1290-L1297)、[恢复与兜底](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L1313-L1315)、[数值](../../../raw/current/scripts/tuning.lua#L6959) |
| 回收 | animover 自删＋宿主状态退出兜底；`persists=false` 不入存档 | [自删](../../../raw/current/scripts/prefabs/daywalker2_fx.lua#L37-L40)、[兜底](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L431-L437) | 同左 |

## 典型接入

仿宿主组合可给近战生物加"地面横扫＋反手"表现：FX 提供 atk1/atk2 双动画与 Reverse 方法，SG 侧 SpawnSwipeFX 提前数帧生成并父挂。[生成](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L421-L429)、[Reverse](../../../raw/current/scripts/prefabs/daywalker2_fx.lua#L6-L8)

## 常见陷阱与限制

- **误区：FX 参与伤害**：本页仅动画；DoArcAttack 用宿主 combat 结算，FX 无 combat／无 AOE。[结算](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L1294-L1297)
- **边界：唯一生成点是反手**：attack_swing 传 reverse=true，即实际显示的是 atk2 段；复用时别只换 atk1。[生成参数](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L1290-L1293)
- **边界：伤害是临时值**：125 只在 attack_swing 态生效，onexit 立即恢复默认；不要在别处假设宿主常驻 125。[恢复](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L1313-L1314)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；44 行工厂全文、SGdaywalker2 生成函数与 attack_swing 态（设伤／生成／AOE／恢复）、宿主依赖表、tuning 数值、译名检索已核验；动画实际帧数、其他攻击态是否复用未核验（本文件内仅一处生成点）。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-creature-index|生物、Boss 与遭遇演出]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
