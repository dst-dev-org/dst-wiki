---
title: "薇洛暗焰（willow_shadow_flame）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "薇洛余烬技能暗焰的会攻击火焰实体：weapon／planardamage 等组件由 DoAttack 结算伤害并按角度链生，同文件另有 willow_throw_flame（落点火）与 willow_frenzy（狂热焚烧 buff 光环壳）两个注册。"
aliases: ["willow_shadow_flame", "willow_throw_flame", "willow_frenzy", "薇洛暗焰", "暗焰", "Shadow Fire", "火焰投掷", "狂热焚烧", "Burning Frenzy"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/willow_flame_fx.lua", "components/firefx.lua", "prefabs/willow_ember.lua", "prefabs/willow.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 薇洛暗焰（willow_shadow_flame）

## 快速判断

- 职责与适用：薇洛（官方译名"薇洛"，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L351390-L351392)）余烬技能"暗焰"（Shadow Fire）生成的**会攻击的火焰实体**：联网 FX 壳＋主模拟 weapon／planardamage／damagetypebonus／deflectable／firefx 组件，由施放者的 `combat:DoAttack` 以本焰为武器结算伤害，并按角度追踪目标链生下一跳；不是纯视觉。同文件还注册 willow_throw_flame（火焰投掷的落点火）与 willow_frenzy（狂热焚烧 buff 的旋转光环壳，纯视觉）。[注册](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L378-L380)
- 优先使用：查"暗焰／Shadow Fire／薇洛火焰特效"选本页；技能入口与余烬消耗见下文生成者条目，生成代码见 [TryShadowFire](../../../raw/current/scripts/prefabs/willow_ember.lua#L367-L388)。
- 重要副作用：暗焰命中造成 `WILLOW_LUNAR_FIRE_DAMAGE×3`（60）武器伤害＋`WILLOW_LUNAR_FIRE_PLANAR_DAMAGE×3`（90）位面伤害＋lunar_aligned 加成 1.1 倍，命中点另生成 willow_shadow_fire_explode 爆炸表现。[组件与伤害](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L221-L228)、[tuning](../../../raw/current/scripts/tuning.lua#L8113-L8114)
- 修改边界：暗焰／投掷焰在所有端有联网壳，伤害组件在主模拟；willow_frenzy 的光环子实体只存在于非专服端。[端分界](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L355-L360)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `willow_shadow_flame` | shadowfn：Transform＋AnimState＋SoundEmitter＋Network → bank/build `shadow_fire_fx` 播 `anim1`、乘法色 alpha 0.6 → 标签 FX／NOCLICK／willow_shadow_flame／pseudorangedweapon／deflectable → SetPristine → 客户端早退 → 主模拟随机换 anim2/3、挂 firefx/weapon/planardamage/damagetypebonus/deflectable、挂 animover 删除回调、暴露 settarget；firefx:SetLevel 会改用显式循环播放，不能保证自然触发 animover | 主模拟另随机 anim1-3 覆盖首播。[工厂](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L180-L201)、[组件](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L216-L234) |
| `willow_throw_flame` | throwfn：bank/build `flamethrow_fx` 播 `pre`（显式不循环）＋bloom → FX／NOCLICK → 主模拟 firefx level 1、10/30 秒后落点放 deerclops_laserscorch 焦痕、animover 仍是 pre 时自删 | 自身无伤害组件；点燃／加燃料由生成方对周围实体执行。[工厂](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L236-L278) |
| `willow_frenzy` | frenzyfn：Transform＋Network（本体无 AnimState）→ `_kill` net_event → 非专服本地挂 AddFrenzyFX 子实体 → SetPristine → 主模拟暴露 `Kill=FrenzyKill`、`persists=false` | 子实体 bank/build `frenzy_fx`：先 PlayAnimation(pre,false) 又立即 PlayAnimation(loop,true)，后者覆盖前者（不是排队 pre→loop）、OnGround 背景层 SortOrder 3、alpha 0.2、缩放 0.85、updatelooper 每秒旋转 15°。[工厂](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L341-L373)、[子实体](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L294-L321) |

三个注册的底部 `PlayAnimation` 循环参数：throw 与 frenzy 子实体显式传参（false／true），shadow／throw 的初始播放随后被 firefx:SetLevel 覆盖为 PlayAnimation(params.anim,true)，最终是显式循环，不能只按工厂首个 PlayAnimation 判断。中文技能名取官方 STRINGS：SHADOW_FIRE"暗焰（5余烬）"（[译名](../../../raw/current/scripts/languages/chinese_s.po#L354324-L354327)）、FIRE_THROW"火焰投掷（1余烬）"（[译名](../../../raw/current/scripts/languages/chinese_s.po#L354314-L354317)）、FIRE_FRENZY"狂热焚烧（2余烬）"（[译名](../../../raw/current/scripts/languages/chinese_s.po#L354309-L354312)）；FX 注册名本身无 NAMES 条目。

## 关联机制导航

以下按三个注册的职责选型。暗焰与投掷焰共享 firefx 播放边界：SetLevel 会把动画设为循环，不能依赖 animover 自动清空；复用暗焰还须覆盖每个链生实体的清理。frenzy 的生成与 Kill 协议见其小节。[循环覆盖](../../../raw/current/scripts/components/firefx.lua#L88-L107)、[链生](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L162-L171)

### 自动索敌暗焰（willow_shadow_flame）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：willow_ember TryShadowFire | 暗焰技能一次性以施放者为中心环形布 5 焰（半径 2、随机 0-0.2 秒延迟），每焰 `settarget(nil, 50, doer)`；重启 spellbook 冷却 8 秒 | 消耗 WILLOW_EMBER_SHADOW＝5 余烬（SPELLCOSTS） | [TryShadowFire](../../../raw/current/scripts/prefabs/willow_ember.lua#L367-L388)、[布火](../../../raw/current/scripts/prefabs/willow_ember.lua#L360-L365)、[消耗](../../../raw/current/scripts/prefabs/willow_ember.lua#L30-L37)、[tuning](../../../raw/current/scripts/tuning.lua#L8104-L8130) |
| 追踪与伤害：settarget | 0.1 秒后检索 20 距离内 `_health`+`_combat` 实体（排除 INLIMBO／invisible／noattack／notarget／flight），过滤友军／非敌对／玩家随从，取与焰朝向夹角最小者；近于 CLOSERANGE 时 `source.combat:DoAttack(target, 本焰)`（ignorehitrange＋ignoredamagereflect；本 build 起近距判定为手工 2D 平面距离，不受高度差影响），命中生成 willow_shadow_fire_explode（新旧版都在目标位置，无版本差异）；远则每跳把朝向转向目标（偏转角 `ReduceAngle` 归一，±30° 上限；本 build 起近距 dsq<9 时按距离放宽上限——近距离允许更大的追踪转角）并在半径 1 处链生新焰（life 递减）；无目标随机偏转 | 检索注释"mimic playercontroller attack targeting"；deflectable owner 设为施放者 | [检索与过滤](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L63-L96)、[选角](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L98-L118)、[攻击与爆炸](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L121-L141)、[转向与链生](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L143-L174) |
| 组件：weapon／planardamage／damagetypebonus | 暗焰作为武器被 DoAttack 使用：伤害 60＋位面 90＋lunar_aligned 加成 1.1 | 仅暗焰注册；投掷焰与 frenzy 无 | [组件](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L221-L228)、[[reference/components/weapon#关键接口]]、[[reference/components/planardamage#关键接口]]、[[reference/components/damagetypebonus#关键接口]] |
| 组件：deflectable | 暗焰可被偏转；settarget 时把 owner 设为施放者 | pristine 阶段先加标签优化 | [标签](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L197-L198)、[组件](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L227)、[[reference/components/deflectable#关键接口]] |

### 投掷落点火（willow_throw_flame）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：willow_ember ThrowFire_SpawnFire | 火焰投掷在目标点生成 willow_throw_flame，并对 2 距离内实体执行点燃／解冻／添燃料／驱雾等 | 与焰本体的 firefx 表现同源调用 | [生成](../../../raw/current/scripts/prefabs/willow_ember.lua#L199-L208)、[点燃逻辑](../../../raw/current/scripts/prefabs/willow_ember.lua#L210-L247) |

### 狂热光环（willow_frenzy）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：buff_firefrenzy debuff | buff 挂上时给目标加 `firefrenzy` 标签并生成 willow_frenzy 挂到目标身上；detached 时 `fx:Kill()`（推 `_kill` 事件、子实体播 post、16 帧后删除） | buff 由 TryFireFrenzy 加给施放者，持续 TUNING.WILLOW_FIREFRENZY_DURATION（seg_time×2） | [attach/detach](../../../raw/current/scripts/prefabs/willow_ember.lua#L1016-L1047)、[施放](../../../raw/current/scripts/prefabs/willow_ember.lua#L390-L398)、[Kill](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L326-L339)、[[reference/components/debuff#关键接口]] |
| buff 效果 | 与 FX 无读写关系：薇洛 CustomCombatDamage 对燃烧目标在 `firefrenzy` 标签下乘 1.25 | 仅薇洛本体；目标燃烧时才生效 | [CustomCombatDamage](../../../raw/current/scripts/prefabs/willow.lua#L152-L156)、[tuning](../../../raw/current/scripts/tuning.lua#L8109-L8110) |

### 共享火光与资源

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：firefx | 暗焰 levels 4 档（radius 2／intensity 0.8，1 档带 meta3/willow/shadowflame 声音）、投掷焰 1 档（dontstarve/common/campfire 声音） | 光照与声音由 firefx 组件消费 levels 表 | [暗焰 levels](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L24-L30)、[投掷 levels](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L32-L35)、[[reference/components/firefx#关键接口]] |
| 动画资源 | 暗焰 shadow_fire_fx.zip、投掷 flamethrow_fx.zip、frenzy frenzy_fx.zip（各自 assets 组） | 资源包内容未解包 | [assets](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L1-L16) |

## 典型接入

### 复用暗焰的"自动索敌武器"模式

- 前提与选择：主模拟侧生成后调 `fire:settarget(目标或 nil, life, source)`；source 是结算伤害的 combat 持有者。自定义武器伤害直接改组件值，不改 settarget。
- 生效与撤销：每跳 0.1 秒一决策；链生由焰自身完成（life 递减到 0 停止），焰本体虽然监听 animover 删除，但 firefx:SetLevel 将动画改为显式循环；没有独立的 Remove 定时器，life 只限制链生，不删除已有实体。直接复用须为每个链生实体补清理。

复用协议为主模拟生成后 `settarget(target, life, source)`；这不是完整寿命骨架。自有工厂须对初始焰与每个链生焰显式安排 Remove，或选择非循环播放并核验 animover；只给最初一焰加定时器无法清理它已生成的后代。

- 调整与验证：CLOSERANGE＝1（命中距离与链生半径同值）、maxdeflect＝30°（转向上限）写在文件顶部与函数内；`pseudorangedweapon` 标签供检索。[常量](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L37-L40)

- 调整与验证（本 build 行为变化）：近距判定改 2D 平面距离、近距离偏转上限随距离放宽（爆炸位置无版本差异；见 updates/build-26335394 P-11）。

## 常见陷阱与限制

- **误区：名字带 fx／标签 FX 就当纯视觉**：暗焰挂 weapon／planardamage 组件并被 DoAttack 结算，是真实伤害源；投掷焰与 frenzy 才是纯表现（投掷焰的点燃作用在生成方代码里）。[伤害组件](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L221-L228)
- **误区：暗焰点燃目标**：暗焰只做近战式 DoAttack＋爆炸表现，无 burnable／propagator；点燃是火焰投掷生成方的逻辑。[投掷点燃](../../../raw/current/scripts/prefabs/willow_ember.lua#L210-L247)
- **误区：willow_frenzy 直接加伤害**：增伤来自 buff_firefrenzy 给的 `firefrenzy` 标签（薇洛 CustomCombatDamage 读取），frenzy 实体只旋转发光。[效果](../../../raw/current/scripts/prefabs/willow.lua#L152-L156)
- **边界：frenzy 光环专服不存在**：AddFrenzyFX 仅非专服执行；`Kill` 仍在主模拟推事件并在 16 帧后删除联网壳。[端分界](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L355-L360)
- **边界：一次暗焰＝5 焰×最多 50 跳链**：TryShadowFire 每焰 life＝50，链生数量大；无目标时焰不消失（继续随机偏转），life=50 的链生最多推进 50 跳，每跳任务延迟 0.1 秒；life 只限制新焰产生；现有焰被 firefx 设为循环后不保证 animover 退场，不能承诺整链自动清空。[链生](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L162-L171)
- **边界：firefx 会覆盖首播循环参数**：shadow 与 throw 都调用 SetLevel，组件在 params.pre=nil 时执行 PlayAnimation(params.anim,true)；throw 仍叫 pre 但已显式循环，不能承诺收到 animover 后自删。工厂没有独立 Remove 定时器。[组件播放](../../../raw/current/scripts/components/firefx.lua#L88-L107)[条件自删](../../../raw/current/scripts/prefabs/willow_flame_fx.lua#L271-L275)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；384 行工厂全文与三注册、settarget 全程（检索／过滤／攻击／转向／链生）、willow_ember 三个生成者与 buff 生命周期、willow.lua buff 效果、tuning 数值、译名检索已核验；动画资源内容、anim1-3 时长、deflectable 偏转的实际消费端、晚加入表现未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
