---
title: "位面命中与抵抗表现（planar_hit_fx）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "通用表两注册的位面战斗表现：无位面防御目标被位面伤害命中时挂身爆点，带位面实体的防御方承受普通攻击时环带偏转；触发条件在 combat 与 planarentity。"
aliases: ["planar_hit_fx", "planar_resist_fx", "位面命中表现", "位面抵抗表现", "位面偏转", "位面爆点", "位面防御特效"]
tags: [prefab, fx, network-sync]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "components/planarentity.lua", "components/combat.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 位面命中与抵抗表现（planar_hit_fx）

## 快速判断

- 职责与适用：位面特殊伤害（spdamage.planar）攻防两侧的伴随表现，共 2 注册；触发条件分别在 combat 与 planarentity 组件，本页只展开表现接线和必要边界，机制见 [[reference/components/planarentity|planarentity]]。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L3128-L3148)
- 优先使用：查"位面命中特效／抵抗偏转"选本页；伤害变换曲线、防御装备与版本增量选 [[reference/components/planarentity|planarentity]]。
- 重要副作用：无；`planar_hit_fx` 另有 `FinalOffset 7` 高层显示。[条目](../../../raw/current/scripts/fx.lua#L3139-L3148)
- 修改边界：`planar_hit_fx` 是**父挂**用例（挂到受击目标）；`planar_resist_fx` 是环带落点；动画是否循环未核验。[挂接](../../../raw/current/scripts/components/planarentity.lua#L38-L40)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `planar_resist_fx` | 通用表条目 → 加载器代理＋一帧后本地实体，fn 内联 | bank／build=`planar_resist_fx`，anim=`deflect`，音效 `rifts/fx/planar_resist_fx`；fn 随机 0.8～1.2 倍缩放并 50% 水平翻转。[条目](../../../raw/current/scripts/fx.lua#L3128-L3137) |
| `planar_hit_fx` | 同上协议，无音效 | bank／build=`planar_damage_fx`，anim=`damage2`；fn 随机 1.2～1.4 倍缩放、50% 翻转、`SetFinalOffset(7)`。[条目](../../../raw/current/scripts/fx.lua#L3139-L3148) |

定向检索简中 .po 未发现这 2 个注册名的 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目，fn 内联随机缩放／翻转 | 两名同协议 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L3128-L3148) |
| 生成者：位面抵抗 | 防御方（挂 planarentity 的实体）受到**不含** `spdamage.planar` 的攻击时，`OnResistNonPlanarAttack` 在其周围环带落点：半径＝物理半径＋0.2＋随机 0.5，朝攻击者方向 ±1 弧度抖动，y 随机 0～1；`spawn_effect_on` 可改参照（蜈蚣预留） | 服务端；调用链在 combat 伤害吸收处（"Planar entities dampen regular damage"） | [抵抗生成](../../../raw/current/scripts/components/planarentity.lua#L15-L36)、[吸收分支](../../../raw/current/scripts/components/combat.lua#L658-L661) |
| 生成者：位面命中 | 攻击携带 `spdamage.planar`、攻击者有 planarentity、且防御方 inventory **无**位面特别防御（`EquipHasSpDefenseForType("planar")` 为假）时，`OnPlanarAttackUndefended` 生成后 **`SetParent` 挂到受击目标** | 服务端；该分支要求目标持有 `components.inventory` 且未设 `ignorecombat`（兔人等带 inventory 的生物同样命中） | [命中生成](../../../raw/current/scripts/components/planarentity.lua#L38-L40)、[combat 分支](../../../raw/current/scripts/components/combat.lua#L624-L627) |
| 机制页 | 触发条件、伤害变换与装备交互 | 版本增量（26148036 增加 planar 门槛）已在机制页标注 | [[reference/components/planarentity#关键接口]] |

## 典型接入

### 借用位面攻防表现

- 前提与选择：模组武器想复用命中表现，需让目标走 `combat:GetAttacked` 且攻击数据带 `spdamage.planar`、攻击者挂 planarentity——表现不可单独 `SpawnPrefab` 伪造"命中了位面伤"之外的语义；抵抗表现在防御方带 planarentity 时自动出现。
- 生效与撤销：命中表现随目标（父挂）、抵抗表现为一次性落点；animover 自删。
- 原版依据：[命中挂接](../../../raw/current/scripts/components/planarentity.lua#L38-L40)。

```lua
-- 伴随伤害路径自动触发；手动复刻抵抗落点的几何
local radius = target:GetPhysicsRadius(0) + 0.2 + math.random() * 0.5
local fx = SpawnPrefab("planar_resist_fx")
fx.Transform:SetPosition(x + radius * math.cos(theta), math.random(), z - radius * math.sin(theta))
```

- 调整与验证：theta 朝攻击者并加 ±1 弧度抖动，无攻击者时全随机（[几何](../../../raw/current/scripts/components/planarentity.lua#L19-L30)）；缩放与翻转由数据 fn 决定，运行时不再改。

## 常见陷阱与限制

- **边界：命中表现要求持有 inventory 且未设 ignorecombat**：combat 的触发分支位于 `self.inst.components.inventory` 且 `not ignorecombat` 的判断块内；没有 inventory 的生物不显示 `planar_hit_fx`，但兔人等自带 inventory 的生物同样会显示，不能按玩家／生物划分。[分支](../../../raw/current/scripts/components/combat.lua#L624-L627)
- **边界：抵抗与命中条件互补不重叠**：抵抗看攻击**不含** planar 特殊伤害（[ AbsorbDamage 判断](../../../raw/current/scripts/components/planarentity.lua#L9-L11)）；命中看攻击**含** planar 且目标无对应防御。别把一方条件套给另一方。
- **边界：表现不改伤害**：`planar_resist_fx` 的偏转观感伴随的是 `AbsorbDamage` 的非线性减伤曲线（[曲线](../../../raw/current/scripts/components/planarentity.lua#L7-L13)）；`planar_hit_fx` 在护甲结算前生成本身不减伤。
- **边界：落点差异**：命中表现挂目标随其移动；抵抗表现固定在世界落点（y 随机），生成后不跟随。[挂接](../../../raw/current/scripts/components/planarentity.lua#L38-L40)
- **边界：动画是否循环未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。两条目配置、加载器协议、combat 两个触发分支（含 inventory 限定与 planar 门槛）、planarentity 两个生成函数与落点几何逐条核验；.po 名称键定向核验无结果。未覆盖：`spawn_effect_on` 的实际使用者、动画资源内容、位面防御装备清单。
- 游戏运行：未运行；爆点挂接与偏转表现未实测。

返回 [[reference/prefabs/fx-status-index|增益、受击、护盾与状态反馈]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
