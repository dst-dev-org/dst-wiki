---
title: "墨荒弹幕投射物（shadowthrall_projectile_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "墨荒（暗影憎恶召唤的影魔翼怪）的抛物线弹幕：complexprojectile 落地后 AOE 结算 25＋30 位面伤害、共享 targets 防重复、留灼痕贴片，同批弹幕共享一次命中音。"
aliases: ["shadowthrall_projectile_fx", "墨荒弹幕", "影魔弹幕", "暗影弹投射物", "shadow thrall projectile"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/shadowthrall_projectile_fx.lua", "stategraphs/SGshadowthrall_wings.lua", "prefabs/shadowthrall_wings.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 墨荒弹幕投射物（shadowthrall_projectile_fx）

## 快速判断

- 职责与适用：Prefab 页语义下的"会结算伤害的 FX"：影魔翼怪 shadowthrall_wings（官方译名"墨荒"，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L348605-L348607)）施法抛出的弹幕，complexprojectile 抛物线飞行，落地 AOE 直接 `GetAttacked`；**不是纯视觉**。[OnHit 结算](../../../raw/current/scripts/prefabs/shadowthrall_projectile_fx.lua#L18-L56)
- 优先使用：查"抛物线 AOE 弹幕／落地范围伤害"选本页；两个生成状态见关联机制导航。
- 重要副作用：粗查半径 1＋3，实际命中距离严格小于 1＋目标物理半径（padding 3 不计入判伤），伤害 25＋位面 30（shrouden 召唤路径额外 +5）；同批弹幕共享 `targets` 表防多段结算、共享 `sfx` 表只播一次命中音；落点留 `fused_shadeling_bomb_scorch` 灼痕（缩放 0.9）。[伤害数值](../../../raw/current/scripts/tuning.lua#L7710-L7711)
- 修改边界：组件（complexprojectile／planardamage）仅主模拟挂载；客户端只有动画与影子；实体 `persists=false` 不入存档。[构造分界](../../../raw/current/scripts/prefabs/shadowthrall_projectile_fx.lua#L96-L117)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `shadowthrall_projectile_fx` | fn：Transform＋AnimState＋SoundEmitter＋DynamicShadow＋Network → 影子 0.8、Physics（质量 1、无摩擦阻尼、CHARACTERS 组、只碰地面、胶囊 0.2）→ 标签 FX／NOCLICK／shadow_aligned／projectile／complexprojectile、六面向、bank/build `shadow_thrall_projectile_fx` 播 `projectile_pre`、LightOverride 1 → SetPristine → 客户端早退 → 主模拟 Push `projectile_loop`、`idle_loop`（**均未显式传循环参数**）→ complexprojectile（水平速 15、重力 -35、发射偏移 (.25,3,0)、OnLaunch 记 owner、OnHit）→ planardamage 基础 30 → `persists=false` | 单注册；`shadow_aligned` 标签同时是弹幕 AOE 的排除标签（弹幕互不误伤）。[工厂](../../../raw/current/scripts/prefabs/shadowthrall_projectile_fx.lua#L62-L120)、[标签](../../../raw/current/scripts/prefabs/shadowthrall_projectile_fx.lua#L81-L94) |

伤害归属：`OnLaunch(inst, attacker)` 把发射者记到 `inst.owner`；OnHit 用 `attacker = inst.owner 有效 ? owner : 弹幕自身` 做 `GetAttacked(attacker, 25, nil, nil, spdmg)`，spdmg 由 `SpDamageUtil.CollectSpDamage` 从弹幕 planardamage 收集（30，召唤路径 +5 bonus）。[归属](../../../raw/current/scripts/prefabs/shadowthrall_projectile_fx.lua#L43-L45)、[OnLaunch](../../../raw/current/scripts/prefabs/shadowthrall_projectile_fx.lua#L58-L60)

本 FX 无 NAMES 中文条目；标题"墨荒"是发射者官方译名，"弹幕投射物"为已核验效果描述。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：SGshadowthrall_wings cast | 23 帧施法点生成 6 发：1 发直射目标当前位置＋5 发环绕落点（半径 2 均分圆周）；全部共享同一 `targets`／`sfx` 表后 `complexprojectile:Launch(pos, inst)` | 6 发同帧齐射，落点分散；目标丢失时朝面向方向 8 距离处兜底 | [cast 状态](../../../raw/current/scripts/stategraphs/SGshadowthrall_wings.lua#L275-L357)、[齐射](../../../raw/current/scripts/stategraphs/SGshadowthrall_wings.lua#L326-L344) |
| 生成者：SGshadowthrall_wings shrouden_flyby | 被暗影憎恶召唤的墨荒飞行缠斗时发射：偏移 (1.5,4.3,0) 高位起跳；若宿主 shrouden 存在，监听 resetboss 自删并给 planardamage 加 +5 位面 bonus（来源 shrouden） | 仅召唤体路径加 bonus；野生墨荒无 | [发射](../../../raw/current/scripts/stategraphs/SGshadowthrall_wings.lua#L458-L465)、[偏移常量](../../../raw/current/scripts/stategraphs/SGshadowthrall_wings.lua#L91)、[召唤建立](../../../raw/current/scripts/prefabs/shadowthrall_wings.lua#L131-L152) |
| [[reference/components/complexprojectile#关键接口]] | 飞行由组件按水平速度＋重力解算，落点触发 OnHit；命中后组件被移除、播放 impact 动画等待 animover 删除 | 仅主模拟；客户端只见网络复制动画 | [组件挂载](../../../raw/current/scripts/prefabs/shadowthrall_projectile_fx.lua#L105-L110)、[OnHit 移除](../../../raw/current/scripts/prefabs/shadowthrall_projectile_fx.lua#L19-L22) |
| [[reference/components/planardamage#关键接口]] | 弹幕自身位面伤害源，OnHit 时 CollectSpDamage 打包进 GetAttacked | shrouden 路径 AddBonus(5)；数值 [tuning](../../../raw/current/scripts/tuning.lua#L7710-L7711)、[bonus](../../../raw/current/scripts/tuning.lua#L9727) | [挂载](../../../raw/current/scripts/prefabs/shadowthrall_projectile_fx.lua#L112-L113) |
| 关联 Prefab：fused_shadeling_bomb_scorch | 命中点生成的灼痕贴片（同文件 prefabs 依赖声明），缩放 0.9 | 无伤害逻辑；生成于 (x,0,z) 贴地 | [生成](../../../raw/current/scripts/prefabs/shadowthrall_projectile_fx.lua#L53-L55)、[依赖](../../../raw/current/scripts/prefabs/shadowthrall_projectile_fx.lua#L8-L11) |
| 宿主：shadowthrall_wings（墨荒） | 独立影魔或暗影憎恶（shrouden）召唤的飞行召唤体；召唤体不可点击、无敌、随宿主 resetboss 移除 | OnShroudenSummon 把 wings 置为召唤形态后进 shrouden_flyby | [召唤回调](../../../raw/current/scripts/prefabs/shadowthrall_wings.lua#L131-L152) |

## 典型接入

复用"共享去重的弹幕齐射"：每轮攻击建一个共享 `targets`／`sfx` 表赋给每发弹幕，再逐发 Teleport＋Launch；不共享表则多发弹幕会对同一目标叠加多轮判定（见陷阱）。[齐射原版写法](../../../raw/current/scripts/stategraphs/SGshadowthrall_wings.lua#L326-L344)

```lua
local targets, sfx = {}, {}
local proj = SpawnPrefab("shadowthrall_projectile_fx")
proj.Physics:Teleport(x, y, z)
proj.targets, proj.sfx = targets, sfx
proj.components.complexprojectile:Launch(Vector3(tx, 0, tz), attacker)
```

- 调整与验证：伤害／位面走 tuning 键（[25／30](../../../raw/current/scripts/tuning.lua#L7710-L7711)）；AOE 排除表在文件头（[L13-L16](../../../raw/current/scripts/prefabs/shadowthrall_projectile_fx.lua#L13-L16)），复制此工厂时注意 `shadow_aligned` 既是自身标签也是排除项。

## 常见陷阱与限制

- **误区：名字带 fx 是纯视觉**：OnHit 直接 `GetAttacked`，是真实伤害源；击杀归属 owner（墨荒）。[结算](../../../raw/current/scripts/prefabs/shadowthrall_projectile_fx.lua#L35-L51)
- **误区：每发弹幕独立打一套 AOE 就等于原版**：原版同批 6 发共享 `targets`，每目标整轮只结算一次；独立表会叠加伤害与位面。[共享表](../../../raw/current/scripts/stategraphs/SGshadowthrall_wings.lua#L326-L327)
- **边界：AOE 不含玩家豁免**：排除表只排 INLIMBO／flight／invisible／notarget／noattack／shadow_aligned——玩家在圈内会被打；`shadow_aligned` 生物（含弹幕自身）免疫。[排除表](../../../raw/current/scripts/prefabs/shadowthrall_projectile_fx.lua#L15-L16)
- **边界：循环参数未显式传入**：`projectile_loop`／`idle_loop` 均未显式传循环参数，实际是否重复未核验；命中即切 `projectile_impact` 并移除组件。[主模拟动画](../../../raw/current/scripts/prefabs/shadowthrall_projectile_fx.lua#L102-L103)
- **边界：命中音去重依赖 sfx 表**：弹幕自带 `sfx.played` 防重，只有同批共享同一表才生效；单发生成时永远播音。[去重](../../../raw/current/scripts/prefabs/shadowthrall_projectile_fx.lua#L24-L33)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；123 行工厂全文（构造、AOE 结算、归属、灼痕）、SGshadowthrall_wings cast／shrouden_flyby 生成点、OnShroudenSummon 召唤链、tuning 数值、译名检索已核验；fused_shadeling_bomb_scorch 内部实现、shrouden 完整 Boss 流程（SGcharlie_boss 侧）未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
