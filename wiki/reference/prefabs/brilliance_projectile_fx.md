---
title: "亮茄魔杖辉光弹（brilliance_projectile_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "亮茄魔杖的远程弹体与命中闪爆：直线飞行、按目标偏好选择后续目标，最多命中 5／7 次（戴亮茄头盔 +2），伤害由持杖者武器结算，命中目标被 colouradder 加色闪一下。"
aliases: ["brilliance_projectile_fx", "brilliance_projectile_blast_fx", "亮茄魔杖辉光弹", "亮茄弹体", "辉光弹", "brightshade staff projectile"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/brilliance_projectile_fx.lua", "prefabs/staff_lunarplant.lua", "components/weapon.lua", "components/projectile.lua", "components/combat.lua", "components/colouradder.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 亮茄魔杖辉光弹（brilliance_projectile_fx）

## 快速判断

- 职责与适用：亮茄魔杖 staff_lunarplant（官方译名"亮茄魔杖"，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L349340-L349342)）的武器投射物：`brilliance_projectile_fx` 弹体直线飞行并可弹跳，`brilliance_projectile_blast_fx` 命中闪爆与目标加色闪光。伤害不经弹幕，弹体只是武器结算的载体。[发射链](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L92-L96)
- 优先使用：查"可弹跳的远程弹体／魔法杖投射物"选本页；弹跳目标偏好算法见典型接入。
- 重要副作用：命中生成 blast 并给目标挂 colouradder 分帧加色（10 帧内恢复）；命中额度在首发时建立（5，头戴亮茄头盔时 7，包含首击），`initial_hostile` 记住首发目标阵营影响弹跳偏好；弹跳命中不消耗魔杖耐久。[弹跳建立](../../../raw/current/scripts/prefabs/brilliance_projectile_fx.lua#L23-L30)、[耐久豁免](../../../raw/current/scripts/components/weapon.lua#L111-L119)
- 修改边界：projectile 组件仅主模拟；弹体标签 FX／NOCLICK 不被点击；`persists=false`。客户端只见动画（idle_loop 显式循环、bloom 符号在 pristine 段设置）。[构造](../../../raw/current/scripts/prefabs/brilliance_projectile_fx.lua#L135-L178)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `brilliance_projectile_fx` | fn：Transform＋AnimState＋Physics＋Network → MakeInventoryPhysics＋移除碰撞体 → 标签 FX／NOCLICK／projectile → bank/build `brilliance_projectile_fx` 播 `idle_loop`（**显式循环**）、light_bar 半透明＋bloom、LightOverride 0.5 → SetPristine → 客户端早退 → 主模拟 projectile 组件（速度 15、射程 25、OnThrown／OnHit／OnMiss）→ `persists=false` | 弹体本体；OnThrown 记 owner 并一次性建立 bounces／initial_hostile。[工厂](../../../raw/current/scripts/prefabs/brilliance_projectile_fx.lua#L135-L178)、[OnThrown](../../../raw/current/scripts/prefabs/brilliance_projectile_fx.lua#L23-L30) |
| `brilliance_projectile_blast_fx` | blastfn：Transform＋AnimState＋Network → 标签 FX／NOCLICK → 同 bank/build 播 `blast1`、bloom 全实体、LightOverride 0.5 → SetPristine → 客户端早退 → 主模拟 50% 概率改播 `blast2`、animover Remove、暴露 PushFlash | 闪爆层；PushFlash 把目标挂 colouradder 分帧加色并注册 OnRemoveEntity 清理。[工厂](../../../raw/current/scripts/prefabs/brilliance_projectile_fx.lua#L208-L240)、[PushFlash](../../../raw/current/scripts/prefabs/brilliance_projectile_fx.lua#L198-L206) |

两注册名均无 NAMES 中文条目；"亮茄"取自武器官方译名，"辉光弹／闪爆"为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：staff_lunarplant 武器 | 魔杖 `weapon:SetProjectile("brilliance_projectile_fx")`、射程 8/10、`projectiledelay` 1 帧；攻击时 `Weapon:LaunchProjectile` Spawn 弹体并 `Throw(staff, target, attacker)` | 武器伤害 0＋位面 10＋对 shadow_aligned 倍率 2（[tuning](../../../raw/current/scripts/tuning.lua#L388-L393)），由持杖者 combat 结算 | [武器装配](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L87-L97)、[发射实现](../../../raw/current/scripts/components/weapon.lua#L123-L153)、[远程分支](../../../raw/current/scripts/components/combat.lua#L1185-L1187) |
| 伤害结算：Projectile:Hit | attacker 取 Throw 的 owner（魔杖）→ 无 combat 时回退到 `inventoryitem.owner`（持杖者）→ `combat:DoAttack(target, 魔杖, 弹体)` | 武器 OnAttack 顺带唤醒沉睡目标、`SuggestTarget` 并推 damage 0 的 "attacked" 事件 | [Hit 归属](../../../raw/current/scripts/components/projectile.lua#L236-L252)、[魔杖 OnAttack](../../../raw/current/scripts/prefabs/staff_lunarplant.lua#L68-L85) |
| 弹跳：TryBounce | 命中后 bounces>1 则藏身 0.1 秒，在命中点 12 距离找新目标：须 `_combat`、非墙／玩家／同伴／飞行／隐身等，CanTarget 且非盟友；`initial_hostile` 时跳过"非敌对＋不能移动＋从未被打过"的目标 | 重投速度降为 10、`overridestartpos` 起点在命中点、`SetBounced(true)`；找不到目标即 Remove | [TryBounce](../../../raw/current/scripts/prefabs/brilliance_projectile_fx.lua#L35-L101)、[OnHit 分支](../../../raw/current/scripts/prefabs/brilliance_projectile_fx.lua#L119-L126) |
| [[reference/components/projectile#关键接口]] | 弹体飞行、命中／脱靶回调、`SetBounced` 与 `overridestartpos` 都是组件接口 | 仅主模拟；miss 播 `disappear` 自删 | [组件挂载](../../../raw/current/scripts/prefabs/brilliance_projectile_fx.lua#L168-L173)、[OnMiss](../../../raw/current/scripts/prefabs/brilliance_projectile_fx.lua#L129-L133) |
| [[reference/components/colouradder#关键接口]] | blast 的 PushFlash 对目标 Push 四档加色色（0.1→0.025），10 帧后 PopColour；提前删除走 OnRemoveEntity 兜底 | 挂在被打目标上；无 colouradder 组件时临时添加 | [PushFlash](../../../raw/current/scripts/prefabs/brilliance_projectile_fx.lua#L182-L206)、[加色更新](../../../raw/current/scripts/components/colouradder.lua#L55-L82) |
| [[reference/components/setbonus#关键接口]] | 命中额度在 OnThrown 一次性判定：头装 prefab == `lunarplanthat`（亮茄头盔，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L344245-L344247)）取 7 否则 5 | 之后换帽不影响本轮弹跳；魔杖本体另有 setbonus 组件（LUNARPLANT 套装） | [判定](../../../raw/current/scripts/prefabs/brilliance_projectile_fx.lua#L25-L29)、[次数](../../../raw/current/scripts/tuning.lua#L392-L393) |
| 资源：brilliance_projectile_fx.zip | 弹体 idle_loop／disappear，闪爆 blast1／blast2 | 弹体 light_bar／glow 符号 bloom；blast 用全实体 bloom shader | [弹体动画](../../../raw/current/scripts/prefabs/brilliance_projectile_fx.lua#L149-L157)、[闪爆动画](../../../raw/current/scripts/prefabs/brilliance_projectile_fx.lua#L218-L223) |

## 典型接入

### 弹跳目标偏好怎么选

首轮弹跳目标由 OnThrown 记录：`bounces`（5／7）与 `initial_hostile`（首发目标是否 hostile 标签）。每次命中 TryBounce 在 12 距离内收集候选后按序择优：敌对优先于非敌对；同级时优先"从未被打过"（不在 recenttargets）；未命中过的候选之间优先与上一目标同 prefab；已命中过的候选之间取 recenttargets 中索引较小者。上一目标被显式排除；更早命中过的目标仍可再次选中，recenttargets 参与优先级排序。[择优逻辑](../../../raw/current/scripts/prefabs/brilliance_projectile_fx.lua#L40-L101)、[recenttargets 维护](../../../raw/current/scripts/prefabs/brilliance_projectile_fx.lua#L87-L94)

- 前提与选择：想改弹跳行为时改 TryBounce 的候选过滤／排序，而不是外层 OnHit——bounces 递减与藏身重投都在 OnHit 侧。
- 生效与撤销：弹跳链上任一环节 attacker 无 combat 或无效即整个 Remove；目标死亡／无效走 OnMiss 播 disappear。

## 常见陷阱与限制

- **误区：伤害数值在弹体上**：弹体无 weapon／combat 组件，伤害来自魔杖（0 白字＋10 位面＋对影 2 倍）；直接 Spawn 弹体不产生任何伤害。[结算链](../../../raw/current/scripts/components/projectile.lua#L236-L252)
- **误区：弹跳是弹体自转**：弹跳由 projectile 组件的 `SetBounced`＋`overridestartpos` 重新 Throw 实现，速度从 15 降为 10；自己改 `SPEED` 不影响弹跳段。[重投](../../../raw/current/scripts/prefabs/brilliance_projectile_fx.lua#L83-L97)
- **边界：弹跳排除玩家**：BOUNCE_NO_TAGS 含 `player`，弹跳不会转向玩家；但首发直射目标可以是玩家（武器可攻击玩家时）。[排除表](../../../raw/current/scripts/prefabs/brilliance_projectile_fx.lua#L32-L33)
- **边界：耐久只扣一次**：`SetBounced` 后的命中被 Weapon:OnAttack 的 IsBounced 检查跳过，一次投掷中的后续弹跳命中不重复扣耐久。[豁免](../../../raw/current/scripts/components/weapon.lua#L111-L119)
- **边界：PushFlash 加色挂在目标而非闪爆自身**：闪爆本体只是动画；提前删除弹体/闪爆时 PopColour 由 OnRemoveEntity 兜底，跨实体清理依赖 target 仍有效。[清理](../../../raw/current/scripts/prefabs/brilliance_projectile_fx.lua#L191-L196)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；246 行工厂全文（双注册、弹跳择优、闪爆加色）、staff_lunarplant 装配与 OnAttack、weapon.lua LaunchProjectile、projectile.lua Hit 归属、tuning 数值、译名检索已核验；combat.lua CalcDamage 全链、missing 视觉细节（弹体飞行朝向）未逐项核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-combat-index|攻击、弹道与落点]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
