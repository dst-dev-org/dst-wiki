---
title: "麋鹿鹅幼崽旋转电光（mossling_spin_fx）"
created: "2026-10-29"
updated: "2026-11-08"
type: prefab
summary: "麋鹿鹅幼崽旋转攻击时挂在身上的电光动画与电音效：无玩法组件，闪电打击是宿主 SG 状态的行为。"
aliases: ["mossling_spin_fx", "麋鹿鹅幼崽旋转电光", "mossling 旋转特效", "spin electric fx"]
tags: [prefab, fx]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/mossling_spin_fx.lua", "stategraphs/SGmossling.lua", "languages/chinese_s.po"]
size_class: simple
reliability: verified
validation: source-reviewed
status: active
---

# 麋鹿鹅幼崽旋转电光（mossling_spin_fx）

## 快速判断

- 职责与适用：Mossling（官方译名"麋鹿鹅幼崽"，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L345329-L345332)）旋转攻击期间套在身上的电光动画，bank/build `mossling_spin_fx` 播 `spin_loop`；不是会攻击的实体。[工厂](../../../raw/current/scripts/prefabs/mossling_spin_fx.lua#L16-L44)
- 优先使用：想要"跟随宿主的自旋光效＋重复音效"参照选本页；生成与闪电行为见下文宿主状态。
- 重要副作用：FX 自身无伤害组件；旋转期间打落的闪电是宿主 SG `spin_loop` 状态的 `LightningStrike`（每 5 帧 10% 概率），不在 FX 上。[闪电](../../../raw/current/scripts/stategraphs/SGmossling.lua#L358-L362)
- 修改边界：联网实体，客户端早退；声音任务与 animover 自删只在主模拟。[构造](../../../raw/current/scripts/prefabs/mossling_spin_fx.lua#L33-L43)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `mossling_spin_fx` | fn：Transform＋AnimState＋SoundEmitter＋Network → bank/build `mossling_spin_fx` 播 `spin_loop`、bloom → FX 标签 → SetPristine → 客户端早退 → 主模拟音效任务、`persists=false`、animover 自删 | 单注册；`spin_loop` 未显式传循环参数，实际是否重复未核验（主模拟靠 animover 删除）。[工厂](../../../raw/current/scripts/prefabs/mossling_spin_fx.lua#L16-L44)、[注册](../../../raw/current/scripts/prefabs/mossling_spin_fx.lua#L46) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：SGmossling "spin_loop" | 生成后 `SetParent` 到幼崽身上并抬到 y=0.1，随宿主移动 | 每次进入 `spin_loop` 状态都新生成一个；状态退出不回收，靠 animover 删除 | [生成](../../../raw/current/scripts/stategraphs/SGmossling.lua#L337-L339)、[状态](../../../raw/current/scripts/stategraphs/SGmossling.lua#L325-L343) |
| 外部行为：LightningStrike | 与 FX 无读写关系：宿主在旋转中以 10%／5 帧概率落雷 | 仅宿主状态；FX 只是同期视觉 | [落雷](../../../raw/current/scripts/stategraphs/SGmossling.lua#L358-L362) |
| 音效：spin_electric | 主模拟 DoPeriodicTask 24 帧播放，计数到 2 后取消（共 3 次，含首次） | 仅主模拟有 SoundEmitter 任务；客户端没有对应 Lua 音效任务，实际听感未运行核验 | [任务](../../../raw/current/scripts/prefabs/mossling_spin_fx.lua#L6-L14)、[挂载](../../../raw/current/scripts/prefabs/mossling_spin_fx.lua#L37-L38) |
| 资源：mossling_spin_fx.zip | bank／build 同名 | [assets](../../../raw/current/scripts/prefabs/mossling_spin_fx.lua#L1-L4) | 内容未解包 |

## 常见陷阱与限制

- **误区：电光会电人或落雷**：FX 无 combat／lightning 组件；落雷由宿主状态按概率触发，命中与 FX 位置无关。[落雷](../../../raw/current/scripts/stategraphs/SGmossling.lua#L358-L362)
- **边界：状态多次进入生成多个 FX**：`spin_loop` 每次进入都 SpawnPrefab，旧 FX 未播完不会复用或取消；短时间反复进出状态会短暂叠加。[生成](../../../raw/current/scripts/stategraphs/SGmossling.lua#L337-L339)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；46 行工厂全文、单注册、SGmossling 生成点与落雷行为、译名检索已核验；动画资源内容、`spin_loop` 是否内部循环、客户端移除时序未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-creature-index|生物、Boss 与遭遇演出]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 6 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-11）。
