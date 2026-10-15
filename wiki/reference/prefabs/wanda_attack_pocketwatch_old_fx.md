---
title: "旺达怀表攻击与治疗特效（wanda_attack_pocketwatch_old_fx）"
created: "2026-11-01"
updated: "2026-11-05"
type: prefab
summary: "fx.lua 通用表 7 注册承载旺达怀表族三段表现：攻击命中爆发按年龄档与暗影武器种类选名、不老表治疗环绕带骑乘变体、第二次机会表复活倒流地面圈；纯表现无结算。"
aliases: ["wanda_attack_pocketwatch_old_fx", "wanda_attack_pocketwatch_normal_fx", "wanda_attack_shadowweapon_old_fx", "wanda_attack_shadowweapon_normal_fx", "pocketwatch_heal_fx", "pocketwatch_heal_fx_mount", "pocketwatch_ground_fx", "旺达攻击特效", "警钟攻击特效", "怀表治疗特效", "暗影武器攻击特效", "复活倒流地面特效"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/fx.lua", "prefabs/wanda.lua", "prefabs/pocketwatch.lua", "prefabs/player_common_extensions.lua", "prefabs/pocketwatch_weapon.lua", "prefabs/pocketwatch_common.lua", "prefabs/staffcastfx.lua", "components/combat.lua", "components/pocketwatch.lua", "actions.lua", "stategraphs/SGwilson.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 旺达怀表攻击与治疗特效（wanda_attack_pocketwatch_old_fx）

## 快速判断

- 职责与适用：fx.lua 通用表 7 注册组成旺达（wanda）怀表族动画特效——攻击命中爆发 4 注册（旺达年龄档 old／normal × 警钟／其他暗影武器）、不老表治疗环绕 2 注册（站立／骑乘）、第二次机会表复活倒流地面圈 1 注册；均为纯表现实体，无伤害／治疗等玩法结算。工厂协议（网络代理＋一帧本地表现、animover 自删、专服无本地动画）见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目组](../../../raw/current/scripts/fx.lua#L2534-L2596)
- 优先使用：查"旺达攻击特效／警钟命中特效／不老表治疗特效／复活倒流地面特效"到本页；警钟的常驻环绕微粒是另一族 [[reference/prefabs/pocketwatch_weapon_fx|pocketwatch_weapon_fx]]；怀表施法基座见 [[reference/components/pocketwatch|pocketwatch 组件]]。
- 重要副作用：FX 条目自身无组件、无结算——回血与停止老伤流失由不老表 DoCastSpell 完成，伤害由 combat 结算；音效由条目 sound 字段在本地实体上延迟 0 帧播放（各模拟各自播）。[生成点](../../../raw/current/scripts/prefabs/wanda.lua#L287-L302)
- 修改边界：三个生成点都在服务端主模拟（combat 命中回调、施法动作执行、复活流程）；anim 随机函数为客户端逐实例求值；攻击与地面条目没有骑乘变体，骑乘仍用站立资源。[选名](../../../raw/current/scripts/prefabs/wanda.lua#L289-L291)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wanda_attack_pocketwatch_old_fx` | 通用表条目，fn=`FinalOffset1`（渲染偏移 1） | bank/build=`pocketwatch_weapon_fx`；anim=函数返回 `"idle_big_"..math.random(3)`（idle_big_1/2/3 三选一）；sound=`wanda2/characters/wanda/watch/weapon/shadow_hit_old`。[条目](../../../raw/current/scripts/fx.lua#L2534-L2540) |
| `wanda_attack_pocketwatch_normal_fx` | 同上协议，fn=`FinalOffset1` | 同 bank；anim=`"idle_med_"..math.random(3)`；sound=`…/nightmare_FX`。[条目](../../../raw/current/scripts/fx.lua#L2542-L2548) |
| `wanda_attack_shadowweapon_old_fx` | 条目 fn 自定义：`AnimState:Hide("white")` 隐藏白色动画层＋`SetFinalOffset(1)` | 同 bank；anim 同 old 档 idle_big；sound=`…/shadow_hit`（与警钟 old 档不同声）。[条目](../../../raw/current/scripts/fx.lua#L2550-L2559) |
| `wanda_attack_shadowweapon_normal_fx` | 同上协议，fn=`FinalOffset1` | 同 bank；anim 同 normal 档 idle_med；sound=`…/nightmare_FX`。[条目](../../../raw/current/scripts/fx.lua#L2561-L2567) |
| `pocketwatch_heal_fx` | fn=`FinalOffset1`；`bloom=true`（挂 shaders/anim.ksh 泛光） | bank=`pocketwatch_cast_fx`、build=`pocketwatch_casting_fx`（两名不一致）；anim=`pocketwatch_heal_fx`，源码注明开头 16 空帧用于音效对位；sound 字段被注释，无注册音效。[条目](../../../raw/current/scripts/fx.lua#L2570-L2577) |
| `pocketwatch_heal_fx_mount` | 同 heal 协议 | bank/build=`pocketwatch_casting_fx_mount`（两名一致）；anim 同名。[条目](../../../raw/current/scripts/fx.lua#L2579-L2586) |
| `pocketwatch_ground_fx` | fn=`GroundOrientation`（OnGround 朝向＋背景层）；`bloom=true` | bank/build 同 heal 非骑乘条目；anim=`pocketwatch_ground`（同 16 空帧注记）；无声效。[条目](../../../raw/current/scripts/fx.lua#L2589-L2596) |

7 名均无 faced 字段（走工厂默认四朝向）、无 nameoverride/description（加 FX 标签）；本地表现 animover 自删，网络代理 1 秒回收。[朝向与音效分支](../../../raw/current/scripts/prefabs/fx.lua#L138-L146)、[音效延迟](../../../raw/current/scripts/prefabs/fx.lua#L43-L50)。7 名在简中 .po 均无 `STRINGS.NAMES` 显示名键，标题与正文中文为已核验效果描述，非官方译名；正文引用的宿主表名（不老表、第二次机会表、警钟）为官方译名，锚点见关联导航。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 代理落位一帧后各非专服模拟建本地动画实体 | 消费者父挂或落点；专服只有代理 | [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]] |
| 攻击生成：ShadowWeaponFx | combat.onhitotherfn 回调内选名生成：weapon 带 `shadow_item` 且 weapon/target 均有效；age_state=old → pocketwatch_old/shadowweapon_old，=normal → 各 normal，young（及 nil）→ nil 不生成；SpawnPrefab 后落点＝目标位置沿攻击者朝向 -90° 方向偏移目标物理半径，y=0，不父挂 | 服务端（wanda master_postinit 接线）；骑乘不改选名（攻击族无 mount 注册） | [ShadowWeaponFx](../../../raw/current/scripts/prefabs/wanda.lua#L287-L302)、[接线](../../../raw/current/scripts/prefabs/wanda.lua#L432)、[master_postinit](../../../raw/current/scripts/prefabs/wanda.lua#L390) |
| 攻击触发窗口 | onhitotherfn 在目标 GetAttacked 内部、仅未 blocked 分支调用——被格挡不出特效 | 服务端（目标主模拟） | [调用条件](../../../raw/current/scripts/components/combat.lua#L715-L727) |
| 治疗生成：Heal_DoCastSpell | 不老表施法成功时 SpawnPrefab（骑乘选 `_mount`）并 `SetParent(doer.entity)` 父挂施法者；前提 doer 有 health 且未死，原版回调还无条件调用 `doer.components.oldager:StopDamageOverTime()` | 服务端；FX 跟随宿主移动；自定义角色只补 `pocketwatchcaster` 标签而无 oldager 会在生成 FX 前报错 | [Heal_DoCastSpell](../../../raw/current/scripts/prefabs/pocketwatch.lua#L29-L41)、[选名与父挂](../../../raw/current/scripts/prefabs/pocketwatch.lua#L35-L36) |
| 治疗触发链 | CAST_POCKETWATCH 动作（需使用者 `pocketwatchcaster`，旺达自带）→ SGwilson ActionHandler 默认进 `pocketwatch_cast` 状态 → 25 帧执行缓冲动作 → pocketwatch 组件 CastSpell → DoCastSpell | 服务端；无特例标签的怀表（含不老表）都走该状态 | [动作](../../../raw/current/scripts/actions.lua#L3146-L3150)、[分派](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1336-L1345)、[25 帧执行](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20496-L20500)、[组件](../../../raw/current/scripts/components/pocketwatch.lua#L25-L32) |
| 施法特效辨析 | 同状态 8 帧另生成 `pocketwatch_cast_fx`／`_mount`（staffcastfx.lua 专属工厂 MakeStaffFX 注册，带施法色）并播 watch/heal 声——与本页 heal 通用表条目是不同注册 | heal 条目开头 16 空帧即与该声对位（源码注记） | [stafffx 生成](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20479-L20487)、[专属注册](../../../raw/current/scripts/prefabs/staffcastfx.lua#L69-L70) |
| 地面生成：DoActualRez | 第二次机会表复活分支（pocketwatch_revive／_reviver）：进 `rewindtime_rebirth` 状态并在重生位置落点 SpawnPrefab，不父挂 | 服务端；同 shard 先传送回死亡位置再落点，作祟触发走淡出直落 | [复活分支](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L392-L409)、[路由](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L595-L604)、[传送落点](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L540-L566)、[状态与复活声](../../../raw/current/scripts/stategraphs/SGwilson.lua#L15436-L15457) |
| 选名标签 | `shadow_item`：警钟充能时添加、耗尽移除；暗剑／暗影战斧／虚影布甲等其余暗影装备各自声明。`pocketwatch`：common_fn 装配的怀表与警钟常挂 | 选名只认 weapon 上的标签 | [警钟标签](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L84-L91)、[怀表标签](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L49) |
| 旺达年龄档 | age_state 三档 old/normal/young 由 onhealthchange 按生命百分比切换（ghost 恒 old、复活重生重置为 nil），是攻击选名的第一层条件 | 服务端维护；oldager 组件把年龄伤害折算为生命变化从而驱动切档 | [档位函数](../../../raw/current/scripts/prefabs/wanda.lua#L69-L157)、[[reference/components/oldager|oldager 组件]] |
| 资源 | 攻击组共用 `pocketwatch_weapon_fx` build（与 [[reference/prefabs/pocketwatch_weapon_fx|警钟微粒特效]] Prefab 同名不同注册，见其"同名 bank 辨析"行）；治疗／地面组用 `pocketwatch_casting_fx` 与 `pocketwatch_casting_fx_mount` build | 动画时长与内容未解包核验 | [条目组](../../../raw/current/scripts/fx.lua#L2534-L2596) |
| 依赖声明 | wanda.lua prefabs 数组列 4 攻击名；pocketwatch.lua prefabs 数组列 heal/ground 3 名＋cast_fx 2 名 | 声明不等于生成；本页生成点均已实核 | [wanda 依赖](../../../raw/current/scripts/prefabs/wanda.lua#L29-L32)、[pocketwatch 依赖](../../../raw/current/scripts/prefabs/pocketwatch.lua#L15-L19) |

宿主表官方名（[chinese_s.po](../../../raw/current/scripts/languages/chinese_s.po#L347050-L347052)）：pocketwatch_heal＝不老表（Ageless Watch）、[pocketwatch_revive＝第二次机会表](../../../raw/current/scripts/languages/chinese_s.po#L347079-L347082)、[pocketwatch_weapon＝警钟](../../../raw/current/scripts/languages/chinese_s.po#L347094-L347097)。

## 典型接入

### 让角色按年龄档出攻击特效（实现参照）

- 前提与选择：ShadowWeaponFx 与选名逻辑都是 wanda.lua 局部函数，模组不能直接调用；给自定义角色／武器复用此表现时仿写回调并接线 `combat.onhitotherfn`（服务端 master_postinit）。
- 生效与撤销：回调在目标未格挡受击时触发一次；FX 落点后动画播完自删，调用方无需清理。
- 原版依据：[ShadowWeaponFx](../../../raw/current/scripts/prefabs/wanda.lua#L287-L302)、[接线](../../../raw/current/scripts/prefabs/wanda.lua#L432)。

```lua
-- 服务端（master_postinit）：等价仿写原版选名与落点（原版为 wanda.lua 局部函数，非公开 API）
local function AgeTierHitFx(inst, target, damage, stimuli, weapon)
    if weapon == nil or target == nil or not target:IsValid() or not weapon:IsValid()
        or not weapon:HasTag("shadow_item") then
        return
    end
    local tier = inst.age_state == "old" and "old"
        or inst.age_state == "normal" and "normal" or nil
    if tier == nil then
        return
    end
    local fx_prefab = weapon:HasTag("pocketwatch")
        and "wanda_attack_pocketwatch_" .. tier .. "_fx"
        or "wanda_attack_shadowweapon_" .. tier .. "_fx"
    local fx = SpawnPrefab(fx_prefab)
    local x, y, z = target.Transform:GetWorldPosition()
    local radius = target:GetPhysicsRadius(.5)
    local angle = (inst.Transform:GetRotation() - 90) * DEGREES
    fx.Transform:SetPosition(x + math.sin(angle) * radius, 0, z + math.cos(angle) * radius)
end

-- inst.components.combat.onhitotherfn = AgeTierHitFx
```

- 调整与验证：非旺达实体没有 age_state 字段，tier 恒 nil 即恒不出特效——想复用三档表现需自设该字段或改选名；特效名须经依赖声明或世界预载保证可生成（[[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]）。

## 常见陷阱与限制

- **误区：pocketwatch_ground_fx 是攻击的地面落点特效**：全 scripts 检索其唯一生成点是第二次机会表复活分支（rewindtime_rebirth），与攻击无关；攻击爆发落在目标身边、只抬 FinalOffset 不做地面朝向处理。[生成点](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L392-L409)
- **误区：旺达持警钟攻击必出特效**：选名要求 age_state 恰为 old 或 normal；young 档（含复活重生初始 nil）返回 nil 不生成，且 weapon 无 `shadow_item`（如未充能警钟）同样不出。[选名](../../../raw/current/scripts/prefabs/wanda.lua#L289-L291)、[充能标签](../../../raw/current/scripts/prefabs/pocketwatch_weapon.lua#L84-L91)
- **边界：格挡不出攻击特效**：onhitotherfn 只在 GetAttacked 的未 blocked 分支调用。[窗口](../../../raw/current/scripts/components/combat.lua#L715-L727)
- **边界：anim 随机各端各自求值**：`"idle_big_"..math.random(3)` 经工厂 FunctionOrValue 在本地实体创建时求值（源码注明 CLIENT SIDE FUNCTION），各端可能播不同编号，不能当服务器同步数据。[求值](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)
- **边界：pocketwatch_cast_fx 不是本页条目**：该名是 staffcastfx.lua 专属工厂注册（施法表现），与 heal/ground 条目只共享 bank 名 `pocketwatch_cast_fx`（mount 条目 bank/build 为 `pocketwatch_casting_fx_mount`）；勿把两者当一族。[注册](../../../raw/current/scripts/prefabs/staffcastfx.lua#L69-L70)
- **边界：治疗／复活的音效不在 FX 条目上**：heal 与 ground 条目的 sound 字段均被注释；施法声（watch/heal）与复活声（wanda1/wanda/rewindtime_rebirth）分别由 SGwilson 状态播放，条目只靠 16 空帧对位。[施法声](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20479-L20487)、[复活声](../../../raw/current/scripts/stategraphs/SGwilson.lua#L15453-L15457)
- **边界：mount 变体仅治疗组有**：heal 按骑乘选 `_mount`；攻击与地面条目注册族内不存在 mount 变体，骑乘攻击仍在目标处落非 mount 资源。[Heal 选名](../../../raw/current/scripts/prefabs/pocketwatch.lua#L35-L36)

待核验问题：动画资源内容（idle_big/med、heal、ground 的时长与 `white` 层视觉）未解包核验；pocketwatch_cast_fx 专属工厂的施法色链（castfxcolour → SetUp）见 [[reference/prefabs/staffcastfx#关联机制导航|施法与怀表回溯表现]]（注册与 `SetUp` 分界见该页“注册与构造”，`pocketwatch_cast` 状态见其导航表）。

## 验证记录

- 源码审阅：2026-10-31，Steam buildid 26335394。逐项核验 7 条目（bank/build/anim/sound/fn/bloom/空帧注记）与 fx.lua 顶部 FinalOffset1／GroundOrientation helper；三个生成点全链核验：ShadowWeaponFx 选名与落点＋combat onhitotherfn 未格挡窗口、Heal_DoCastSpell 父挂与 CAST_POCKETWATCH→pocketwatch_cast 25 帧执行链、DoActualRez 第二次机会表复活分支与 rewindtime_rebirth 状态；选名标签（shadow_item 充能添删、pocketwatch 常挂、age_state 三档）；chinese_s.po 确认 7 名无显示名键并核验宿主表官方名。检索范围：rg "wanda_attack_pocketwatch"／"wanda_attack_shadowweapon"／"pocketwatch_heal_fx"／"pocketwatch_ground_fx"／"wanda_attack_" 于 raw/current/scripts 全树（根级＋prefabs／components／stategraphs），除 fx.lua 注册外仅命中 wanda.lua、pocketwatch.lua、player_common_extensions.lua 三处生成点与两处依赖声明，无其他动态拼接点。未覆盖：动画资源解包内容、施法色链细节、晚加入时序、游戏运行。
- 游戏运行：未运行。
- 结构检查：validate_page 与 validate_source_links --strict 执行结果见本批交付报告。

返回 [[reference/prefabs/fx-transition-index|变身、召唤、传送与退场]]、[[reference/prefabs/index|Prefab 索引]]。
