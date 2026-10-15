---
title: "阿比盖尔反击、怨咒与暗影增益特效（abigail_retaliation 等）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "阿比盖尔 prefab 随本体返回的五个外挂子注册：蒸馏复仇触发的反击、怨咒命中反馈与承伤 debuff 两变体、温蒂暗影技能的凶杀增益；按工厂、端、触发、父挂与清理逐项核验。"
aliases: ["abigail_retaliation", "abigail_vex_hit", "abigail_vex_debuff", "abigail_vex_shadow_debuff", "abigail_murder_buff", "阿比盖尔反击特效", "阿比盖尔怨咒特效", "阿比盖尔暗影增益", "复仇反击特效"]
tags: [prefab, fx]
size_class: complex
dst_build: "steam-buildid:26335394"
sources: ["prefabs/abigail.lua", "prefabs/ghostly_elixirs.lua", "prefabs/wendy.lua", "prefabs/skilltree_wendy.lua", "prefabs/ghostcommand_defs.lua", "prefabs/abigailforcefield.lua", "prefabs/hats.lua", "stategraphs/SGabigail.lua", "components/combat.lua", "actions.lua", "tuning.lua", "entityscript.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 阿比盖尔反击、怨咒与暗影增益特效（abigail_retaliation 等）

## 快速判断

- 职责与适用：`prefabs/abigail.lua` 随幽灵伙伴 `abigail` 同文件返回的五个外挂：延迟反击的 `abigail_retaliation`、目标身上的怨咒 `abigail_vex_debuff`／`abigail_vex_shadow_debuff`、本地命中反馈 `abigail_vex_hit`、温蒂暗影技能的 `abigail_murder_buff`；除命中反馈外均有联网壳。[返回列表](../../../raw/current/scripts/prefabs/abigail.lua#L1292-L1297)
- 优先使用：延迟反击→ `abigail_retaliation` 的 `SetRetaliationTarget`；承伤标记＋命中反馈→ 两个怨咒 debuff 的 attach／detach；只借命中反馈→ `abigail_vex_hit`；暗影 build／光环切换→ `abigail_murder_buff`。
- 重要副作用：怨咒让目标承伤 ×1.1（温蒂另有 ×1.4；阿比盖尔自身伤害被 `1/1.1` 归一，仅其他来源吃满），暗影怨咒使目标每次被（非阿比盖尔、非重定向、非 `attacktriggereddebuff`）攻击时**自身额外承受** 0 物理＋10 位面，攻击者记为 debuff 本体，不是反伤给攻击者；凶杀增益给阿比盖尔 +25 位面伤害、+5 位面防御并换暗影 build；反击在 12 帧后结算 20 伤害。[倍率](../../../raw/current/scripts/prefabs/abigail.lua#L1106)、[位面](../../../raw/current/scripts/prefabs/abigail.lua#L294-L298)、[反击](../../../raw/current/scripts/prefabs/abigail.lua#L990-L997)
- 修改边界：本页只覆盖这五个注册的工厂、端、触发、父挂与清理；同文件首个返回 `abigail` 是幽灵伙伴本体（召唤、战斗与等级见 [[reference/components/ghostlybond|ghostlybond]]），召唤与升级 FX 见 [[reference/prefabs/abigailsummonfx]]。模组动态调用不在结论内。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 端、父挂与清理 |
|---|---|---|
| `abigail_retaliation` | `retaliationattack_fn`：Transform／AnimState／SoundEmitter／Network → bank／build `abigail_shield`、动画 `retaliation_fx`、`SetFinalOffset(3)` → `FX` 标签 → SetPristine；客户端返回；主模拟导出 `SetRetaliationTarget`，12 帧结算伤害、30 帧删除 | 联网；生成方指定目标后父挂该目标并按目标缩放（大型 ×1.1／其他 ×0.8），目标死亡或移除时脱挂到世界坐标；伤害走目标 `combat:GetAttacked(inst, 20)`；未设 `persists=false`。[工厂](../../../raw/current/scripts/prefabs/abigail.lua#L999-L1031)、[父挂](../../../raw/current/scripts/prefabs/abigail.lua#L971-L988) |
| `abigail_vex_debuff` | `CreateDebuff("abigail_vex_debuff")`：Transform／AnimState／Network → bank／build `abigail_debuff_fx`、`vex_debuff_pre` → `vex_debuff_loop`（循环）＋`SetFinalOffset(3)` → net_event `hitevent`（非专服监听）→ SetPristine；客户端返回；主模拟 `debuff` 组件＋`persists=false` | 联网；attach 父挂目标并按目标缩放、注册承伤 ×1.1 与 `attacked` 监听；2 秒（gestalt 的 ×3 分支为死代码，buffer 恒 nil）；detach 移除倍率、推 `vex_debuff_pst` 后 `animqueueover` 自删；换变体先移除旧。[工厂](../../../raw/current/scripts/prefabs/abigail.lua#L1143-L1183)、[attach](../../../raw/current/scripts/prefabs/abigail.lua#L1089-L1121)、[detach](../../../raw/current/scripts/prefabs/abigail.lua#L1123-L1141)、[延长](../../../raw/current/scripts/prefabs/abigail.lua#L1051-L1061) |
| `abigail_vex_shadow_debuff` | 同一工厂的 `name` 分支：attach 时额外加 `planardamage`（10）、`damagetypebonus`、`attacktriggereddebuff` 标签与 `attacked` 监听；命中反馈染黑 | 联网；由 `ApplyDebuff` 在阿比盖尔带 `super_elixir_buff` 且 prefab 为 `ghostlyelixir_shadow_buff` 时选用；监听挂在被标记目标身上，目标每次被（非阿比盖尔、非重定向、非 `attacktriggereddebuff`）攻击时自身额外承受 0 物理＋10 位面，攻击者记为 debuff 本体。[变体](../../../raw/current/scripts/prefabs/abigail.lua#L380-L403)、[附加](../../../raw/current/scripts/prefabs/abigail.lua#L1063-L1104)、[染色](../../../raw/current/scripts/prefabs/abigail.lua#L1036-L1043) |
| `abigail_vex_hit` | `abigail_vex_hit_fn`：`CLASSIFIED`、无 Network；Transform／AnimState → bank／build `abigail_debuff_fx`、动画 `vex_hit`＋`SetFinalOffset(3)` → `FX` 标签 → `persists=false` → `animover` Remove | 纯本地；由怨咒实体收到 net event 后在自身世界位置生成（非专服），无 sound；暗影变体由生成方 `SetMultColour(0,0,0,1)`。[工厂](../../../raw/current/scripts/prefabs/abigail.lua#L1189-L1209)、[生成](../../../raw/current/scripts/prefabs/abigail.lua#L1036-L1043)、[事件](../../../raw/current/scripts/prefabs/abigail.lua#L1159-L1163) |
| `abigail_murder_buff` | `abigail_murder_buff_fn`：Transform／Network → `FX` 标签 → SetPristine；客户端返回；主模拟 `persists=false`＋`debuff` 组件，无 AnimState | 联网但不父挂；attach 改宿主 build、重启 aura、生成 `shadow_puff_large_front`、+5 位面防御并刷新 `UpdateDamage`（+25 位面）；8 秒，可续到 160 秒；detach 全部还原。[工厂](../../../raw/current/scripts/prefabs/abigail.lua#L1268-L1290)、[attach](../../../raw/current/scripts/prefabs/abigail.lua#L1220-L1241)、[detach](../../../raw/current/scripts/prefabs/abigail.lua#L1243-L1266)、[常量](../../../raw/current/scripts/tuning.lua#L7318-L7320) |

五个注册名在简中 `.po` 均无 `STRINGS.NAMES.*` 键；标题与别名中文是已核验效果描述，不是官方译名。相关官方名：触发物品 `ghostlyelixir_retaliation`＝“蒸馏复仇”，玩家链的 `ghostflowerhat`＝“幽魂花冠”。[po](../../../raw/current/scripts/languages/chinese_s.po#L343204-L343227)

## 关联机制导航

### 生成点与触发

| 生成链 | 涉及注册 | 条件、端与阶段 | 证据 |
|---|---|---|---|
| 温蒂突击命令 | 两个怨咒 debuff、命中反馈 | `wendy_ghostcommand_2` 解锁的 ATTACK_AT → `do_ghost_attackat` → `DoGhostAttackAt` 设目标并进入 SG `abigail_attack_start`；非 gestalt 的 dash 在范围内每个可攻击目标上 push `onareaattackother`，`linktoplayer` 注册的 `ApplyDebuff` 随即加／换 debuff。仅主模拟。 | [技能节点](../../../raw/current/scripts/prefabs/skilltree_wendy.lua#L237-L260)、[命令](../../../raw/current/scripts/prefabs/ghostcommand_defs.lua#L210-L233)、[入口](../../../raw/current/scripts/prefabs/abigail.lua#L527-L558)、[push](../../../raw/current/scripts/stategraphs/SGabigail.lua#L76-L116)、[监听](../../../raw/current/scripts/prefabs/abigail.lua#L405-L412) |
| 温蒂 gestalt 普通攻击 | 两个怨咒 debuff、命中反馈 | `gestalt_attack` 的 `gestalt_loop_attack`／`gestalt_loop_homing_attack` 在 `DoAttack` 后直接 `inst:ApplyDebuff({target=target})`（L966／L1082），不经 `onareaattackother`。 | [SG](../../../raw/current/scripts/stategraphs/SGabigail.lua#L965-L966)、[同型](../../../raw/current/scripts/stategraphs/SGabigail.lua#L1082-L1082) |
| 怨咒命中反馈 | `abigail_vex_hit` | 目标 `attacked` 事件里攻击者带 `ghostlyfriend` 标签才 push `hitevent`；非专服收到 net event 后本地生成。温蒂在 `common_postinit` 获此标签，阿比盖尔本体不加。 | [回调](../../../raw/current/scripts/prefabs/abigail.lua#L1045-L1049)、[事件](../../../raw/current/scripts/prefabs/abigail.lua#L1159-L1163)、[标签](../../../raw/current/scripts/prefabs/wendy.lua#L152-L153) |
| 阿比盖尔受击 | `abigail_retaliation` | `OnAttacked` 先查 `forcefield`，攻击者非玩家主且带 combat、`elixir_buff` 为 `ghostlyelixir_retaliation_buff` 时生成并指向攻击者；函数末尾才 `StartForceField` 补盾。仅主模拟。 | [受击](../../../raw/current/scripts/prefabs/abigail.lua#L204-L217)、[补盾](../../../raw/current/scripts/prefabs/abigail.lua#L180-L185)、[药剂](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L253-L284) |
| 玩家受击（幽魂花冠链） | `abigail_retaliation` | 玩家戴头部 `rechargeable` 已充能的幽魂花冠并有复仇药剂 buff 时，`attacked` 回调对 5 范围内每个可攻击 `_combat` 实体**先立即造成 20**、再各生成一个反击 fx 指向它（12 帧后再 20，单目标合计可达 40）；随后帽子放电 10。仅 `playerreatliate` 药剂走反击。 | [回调](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L6-L85)、[花冠](../../../raw/current/scripts/prefabs/hats.lua#L5186-L5205)、[药剂](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L262-L283) |
| 温蒂凶杀 | `abigail_murder_buff` | 温蒂 `murdered` 事件（MURDER 动作、活物下锅、焚化炉或背包腐烂等来源，带 stackmult）→ `checkforshadowsacrifice` 需 `wendy_shadow_3`、幽灵存在且非 INLIMBO → `DoShadowBurstBuff`；无 buff 先加，再按剩余＋层数×8 秒续，上限 160 秒。仅主模拟。 | [事件](../../../raw/current/scripts/actions.lua#L3166-L3183)、[下锅](../../../raw/current/scripts/actions.lua#L2027)、[焚化炉](../../../raw/current/scripts/components/incinerator.lua#L29-L32)、[腐烂](../../../raw/current/scripts/standardcomponents.lua#L929-L937)、[门禁](../../../raw/current/scripts/prefabs/wendy.lua#L246-L252) |

### 消费与相关对象

- **伤害倍率**：怨咒 attach 给目标 `externaldamagetakenmultipliers` 设 1.1，战斗结算时相乘；温蒂的 `customdamagemultfn` 对带怨咒目标返回 1.4。[目标倍率](../../../raw/current/scripts/prefabs/abigail.lua#L1106)、[温蒂](../../../raw/current/scripts/prefabs/wendy.lua#L277-L283)、[结算](../../../raw/current/scripts/components/combat.lua#L644-L651)
- **SG／表现消费**：凶杀增益存在时，SG `startaura` 把攻击环换成 `abigail_attack_fx_shadow_build` 并覆写三个漩涡符号，`getidleanim` 也因此跳过随机 flicker；暗影爆发同时生成 `abigail_attack_shadow_fx` 与父挂 `abigail_shadow_buff_fx`（两页另述）。[光环](../../../raw/current/scripts/stategraphs/SGabigail.lua#L32-L45)、[idle](../../../raw/current/scripts/stategraphs/SGabigail.lua#L3-L20)
- **相关 Prefab**：`abigailforcefield`／`abigailforcefieldbuffed`／`abigailforcefieldretaliation` 是同一 `abigail_shield` build 的护盾环，与反击 `abigail_retaliation` 不是同一实体；后者只播 `retaliation_fx`。[护盾工厂](../../../raw/current/scripts/prefabs/abigailforcefield.lua#L33-L68)
- **资源**：`anim/abigail_shield.zip`（反击复用 `retaliation_fx`）、`anim/abigail_debuff_fx.zip`；音效与 bloom 着色器见工厂。动画帧数与循环未解包。[资产](../../../raw/current/scripts/prefabs/abigail.lua#L1293-L1296)

## 典型接入

### 复用“父挂目标＋延迟伤害”的反击

- 前提与选择：服务端调用，目标为有效且带 `combat` 的攻击者；`abigail_retaliation` 随 `abigail.lua` 注册。客户端在 SetPristine 后早退，`SetRetaliationTarget` 只在主模拟存在。
- 生效与撤销：调用即父挂并按目标缩放；12 帧后目标仍有效、非 limbo 才结算 20 伤害并脱挂；30 帧后实体删除。原版每次受击最多生成一枚，重复生成会叠加多枚、各自结算。
- 原版依据：[生成点](../../../raw/current/scripts/prefabs/abigail.lua#L204-L214)、[父挂](../../../raw/current/scripts/prefabs/abigail.lua#L971-L988)、[伤害与定时删除](../../../raw/current/scripts/prefabs/abigail.lua#L990-L997)

```lua
-- 服务端片段：target 为本次攻击者，调用方保证其有效且带 combat。
local function spawn_retaliation(target)
    if not TheWorld.ismastersim then
        return nil
    end
    local fx = SpawnPrefab("abigail_retaliation")
    if fx == nil then
        return nil
    end
    fx:SetRetaliationTarget(target)
    return fx
end
```

- 调整与验证：伤害值来自 `TUNING.GHOSTLYELIXIR_RETALIATION_DAMAGE`（20），本工厂不接收参数；示例只做语法检查，未在游戏内执行。[常量](../../../raw/current/scripts/tuning.lua#L4850-L4851)

## 常见陷阱与限制

- **误区：五个注册都是“特效”。** 两个怨咒与凶杀增益都挂 `debuff` 并改战斗数值；把注册名当纯视觉会在目标身上留下 2 秒承伤 +10%，或给阿比盖尔 +25 位面伤害、+5 位面防御。[倍率](../../../raw/current/scripts/prefabs/abigail.lua#L1106)、[位面](../../../raw/current/scripts/prefabs/abigail.lua#L294-L298)
- **边界：怨咒的种下不止突击命令。** 非 gestalt 普通攻击确实不 push `onareaattackother`，但 gestalt 形态的普通攻击经 `SGabigail` 的 `gestalt_loop_attack`／`gestalt_loop_homing_attack` 直接调用 `ApplyDebuff`（L966／L1082），同样种怨咒。[push](../../../raw/current/scripts/stategraphs/SGabigail.lua#L76-L116)
- **边界：gestalt 的 3 倍怨咒时长是死代码。** 3 倍只在 `buff_OnExtended` 的 `buffer:HasTag("gestalt")` 分支；`AddDebuff` 只接收 4 参（debuffable L95），`ApplyDebuff` 多传的 `inst` 被忽略，`buffer` 恒 nil，分支不可达。[延长](../../../raw/current/scripts/prefabs/abigail.lua#L1051-L1061)、[AddDebuff](../../../raw/current/scripts/components/debuffable.lua#L95-L100)
- **边界：命中反馈由温蒂触发，不是阿比盖尔。** 回调检查攻击者 `ghostlyfriend` 标签；脚本内只有温蒂 `common_postinit` 添加该标签，阿比盖尔本体不加。[回调](../../../raw/current/scripts/prefabs/abigail.lua#L1045-L1049)、[标签](../../../raw/current/scripts/prefabs/wendy.lua#L152-L153)
- **边界：反击判定在补盾之前。** `OnAttacked` 先检查 `forcefield`，末尾才 `StartForceField`；上药后若还没有护盾，本次受击只补盾、不生成反击。[顺序](../../../raw/current/scripts/prefabs/abigail.lua#L204-L217)
- **边界：反击未设 `persists=false`。** 实体默认持久（[entityscript](../../../raw/current/scripts/entityscript.lua#L219-L223)），清理只靠 30 帧定时器；存活期存档的恢复行为未实测。[定时删除](../../../raw/current/scripts/prefabs/abigail.lua#L1025-L1030)
- **边界：暗影变体的 debuff 本体不染色。** 工厂与 attach 都没对 debuff 换 build／染色，暗影差异是附加组件、反伤与命中反馈染黑。[工厂](../../../raw/current/scripts/prefabs/abigail.lua#L1143-L1181)

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394（`raw/current/BUILD.txt`）。逐项读五个注册的工厂、端分界、父挂、清理与数值；触发链另读 ghostcommand_defs、SGabigail、wendy、ghostly_elixirs、hats、combat、tuning。按五个完整注册名在 `raw/current/scripts` 全域 `grep -rn --include=*.lua` 反查，命中仅 abigail.lua、ghostly_elixirs.lua、wendy.lua、SGabigail.lua；`prefablist.lua` 无这些名的预载条目。未覆盖：动画包帧数与循环、客户端表现时序、存档边界、模组动态调用。
- 名称核验：`languages/chinese_s.po` 定向检索五个注册名无 `STRINGS.NAMES.*` 键；仅触发物品 `GHOSTLYELIXIR_RETALIATION` 命中共官方名“蒸馏复仇”。[po](../../../raw/current/scripts/languages/chinese_s.po#L343224-L343227)
- 游戏运行：未运行。
- 结构检查：`validate_page.py` 通过（105 行、约 16.5 KiB，complex 档 400 行／32 KiB）；`validate_source_links.py --strict` 通过（61 处引用、13 个源文件，errors 0／warnings 0）；示例经 `parser_probe --syntax-only` 通过（未执行）。
- 分档：五个注册、非 gestalt 与 gestalt 两条触发路径、反击/减益/命中反馈三类生命周期，按 `complex` 记录。

## 相关链接

- [[reference/prefabs/abigail_attack_fx|阿比盖尔攻击环特效]]（凶杀增益换 shadow build）
- [[reference/prefabs/abigail_attack_shadow_fx|温蒂、阿比盖尔与纯辉标记特效]]（暗影爆发同时生成的表现）
- [[reference/prefabs/abigailsummonfx|阿比盖尔之花召唤、收回与升级特效]]（召唤／等级 FX）
- [[reference/prefabs/ghostlyelixir_slowregen_fx#生成链与生命周期|幽灵灵药环绕与滴落特效]]（蒸馏复仇的药剂与饮用链）

返回 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]、[[reference/prefabs/index|Prefab 索引]]。
