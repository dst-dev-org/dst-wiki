---
title: "命中火花特效（hitsparks_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "武器／护甲命中的火花表现族：hitsparks_fx（普通命中）、hitsparks_reflect_fx（伤害反弹）、hitsparks_piercing_fx（弹道穿刺）三注册共用一个工厂，网络代理＋本地 hit_3 火花，主模拟对目标做四步白闪（可染色、可黑剪影）；无伤害组件。"
aliases: ["hitsparks_fx", "hitsparks_reflect_fx", "hitsparks_piercing_fx", "命中火花", "打击火花", "hit sparks"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/hitsparks_fx.lua", "prefabs/sword_lunarplant.lua", "prefabs/shadow_battleaxe.lua", "prefabs/voidcloth_scythe.lua", "prefabs/armor_lunarplant.lua", "prefabs/houndstooth_blowpipe.lua", "prefabs/hermitcrabtea_defs.lua", "components/colouradder.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 命中火花特效（hitsparks_fx）

## 快速判断

- 职责与适用：近战／弹道命中瞬间的火花：代理实体定位到受击点，各端本地播一次 `hit_3`；主模拟在 Setup 时对受击目标做约 4 步的加色闪烁（默认白，可用 flashcolour 染色），shadow 武器再把火花本体调成黑剪影（[Setup](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L53-L65)、[闪色](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L21-L39)）。
- 优先使用：直接 `SpawnPrefab("hitsparks_fx"):Setup(attacker, target[, projectile, flashcolour])`；反弹用 `hitsparks_reflect_fx`，弹道穿刺用 `hitsparks_piercing_fx`（多 inverted／offset_y 参数）。
- 重要副作用：对目标推 colouradder 颜色栈并在收尾 Pop；fx 自身挂 updatelooper 驱动闪色，兜底 OnRemoveEntity=PopColour（[StartFlash](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L41-L49)）。
- 修改边界：无伤害／无状态；Setup 系列仅主模拟暴露；代理 1 秒自毁、视觉 animover 自删。[主模拟暴露](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L180)、[清理](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L175-L176)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `hitsparks_fx` | `MakeFX(name, false, Setup)`：fn：Transform＋Network → 非专服 `DoTaskInTime(0, PlaySparksAnim, horizontal)` → `flip`/`black` net_bool → SetPristine → 主模拟 `persists=false`、1 秒自毁、随机 flip、暴露 Setup=setupfn | 普通命中；视觉不设 OnGround（竖立面向镜头） | [注册](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L190-L193) |
| `hitsparks_reflect_fx` | 同一 MakeFX，`horizontal=true, setupfn=SetupReflect` | 反弹：视觉 `ANIM_ORIENTATION.OnGround` 铺地 | [注册](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L192)、[horizontal 分支](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L141-L143) |
| `hitsparks_piercing_fx` | 同一 MakeFX，`horizontal=true, setupfn=SetupPiercing` | 穿刺：位置取弹体、朝向按 inverted 取反，可传 offset_y | [SetupPiercing](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L90-L115) |

三变体共同视觉：bank `hits_sparks`、build `lavaarena_hit_sparks_fx`、播 `hit_3`（未显式传循环参数，实际是否重复未核验）、bloom、FinalOffset 1、scale ±0.7（flip 决定镜像）、`black` 置位时 `SetMultColour(0,0,0,1)` 黑剪影。[本地视觉](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L119-L146)

注册名均无 NAMES 条目；标题中文为已核验效果描述，非官方译名。宿主官方名称：[亮茄剑](../../../raw/current/scripts/languages/chinese_s.po#L349570-L349572)、[暗影槌](../../../raw/current/scripts/languages/chinese_s.po#L348620-L348622)、[暗影收割者](../../../raw/current/scripts/languages/chinese_s.po#L350730-L350732)、[亮茄盔甲](../../../raw/current/scripts/languages/chinese_s.po#L338245-L338247)、[嚎弹炮](../../../raw/current/scripts/languages/chinese_s.po#L343395-L343397)。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 定位：Setup 家族 | Setup：以攻击者/弹体朝向＋180° 在目标半径边缘落点；SetupReflect：由源→目标向量按源半径重标定，y=1+rand，rot+90；SetupPiercing：落点在弹体与目标连线上（inverted 决定指向），offset_y 覆盖默认高度 | 普通／reflect 的 source 可缺省；piercing 必须有有效的 projectile 或 attacker，且三者均要求有效 target；仅主模拟 | [Setup](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L53-L65)、[Reflect](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L67-L88)、[Piercing](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L90-L115) |
| 闪色：UpdateFlash | fx 挂 [[reference/components/updatelooper|updatelooper]]，flashstep 1..3 推 `step×0.05`（step>2 反向）倍的颜色：有 flashcolour 按分量乘，否则全白；目标无 colouradder 时补挂 | 主模拟组件；颜色更新走 colouradder（colouraddersync 或 AnimState）；实际客户端闪色表现未运行核验 | [UpdateFlash](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L21-L39)、[PushColour](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L8-L13)、[[reference/components/colouradder]] |
| netvar：flip／black | flip 决定视觉镜像（主模拟随机置位）；black 由生成方置位决定黑剪影 | 双端本地视觉读取 | [netvar](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L166-L167)、[读取](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L137-L140) |
| 消费者：亮茄与暗影武器 | 亮茄剑 [OnAttack](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L135-L139)（白闪）；暗影槌 [DoAttackEffects](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L257-L260)（红 {1,0,0}＋black）、暗影收割者 [TryToSparkOn](../../../raw/current/scripts/prefabs/voidcloth_scythe.lua#L251-L255)（红＋black） | 均为武器 onattack／攻击效果入口，主模拟 | [武器组件参照](../../../raw/current/scripts/prefabs/sword_lunarplant.lua#L135-L139) |
| 消费者：护甲与弹道 | 亮茄盔甲 [OnReflectDamage](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L163-L168)（反弹时对伤害来源用 reflect 变体）；嚎弹炮 [Projectile_SpawnImpactFx](../../../raw/current/scripts/prefabs/houndstooth_blowpipe.lua#L148-L151)（弹体＋inverted＋offset_y） | reflect 的"attacker"参数是护甲持有者 | [OnReflectDamage](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L163-L168) |
| 消费者：debuff 联动 | hermitcrabtea_defs 的 moon_tree_blossom debuff 监听 `attacked`，被暗影命中时蓝色火花 {0,0,1}（[颜色](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L105-L109)、[监听](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L230-L234)） | SparkLunarOnShadow 的 attacker 参数是攻击者 | [MoonBlossom_OnAttacked](../../../raw/current/scripts/prefabs/hermitcrabtea_defs.lua#L113) |
| 资源 | anim/lavaarena_hit_sparks_fx.zip（ Forge 遗产资源） | 声明即核验范围 | [assets](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L1-L3) |

## 典型接入

### 武器命中加火花

- 前提与选择：主模拟在 onattack／攻击效果里 `SpawnPrefab("hitsparks_fx"):Setup(attacker, target, nil, colour)`；要黑剪影再 `spark.black:set(true)`（netvar，生成后立即置位即可）。
- 生效与撤销：Setup 内部完成定位＋StartFlash；闪色在 flashstep 走完后自动 Pop，目标中途死亡由 PopColour 的 IsValid 保护与 OnRemoveEntity 兜底（[保护](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L15-L19)、[兜底](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L37-L38)）。
- 原版依据：暗影槌／暗影收割者的"红黑火花"即 flashcolour＋black 组合的官方示例。

```lua
local spark = SpawnPrefab("hitsparks_fx")
spark:Setup(owner, target, nil, {1, 0, 0})  -- 红染闪色；省略则白闪
spark.black:set(true)                        -- 火花本体黑剪影
```

- 调整与验证：闪色强度 0.05/步、共 3 步推色，视觉很淡，想要强闪烁需自建工厂而非调参（[步进](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L23-L27)）。

## 常见陷阱与限制

- **误区：black 是另一套黑色火花贴图**：同一 `hit_3` 动画经 `SetMultColour(0,0,0,1)` 压成剪影，靠 netvar 广播；改 flashcolour 不会影响火花本体颜色（[实现](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L138-L140)）。
- **边界：Setup 只在主模拟有效**：Setup 在 SetPristine 之后的 master 分支暴露，客户端拿到的实体没有该函数；客户端想预测需自建本地视觉（[暴露位置](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L180)）。
- **边界：闪色与火花是两套端**：火花动画各端本地播放（netvar 只传 flip/black），对目标的目标闪白走主模拟 colouradder——不能仅凭 Lua 在主模拟执行断言其他客户端看不到目标闪色（同 [[reference/prefabs/electricchargedfx#常见陷阱与限制|electricchargedfx 的端边界]]）。
- **边界：pierce 变体的朝向**：SetupPiercing 的 rot 是"目标指向弹体"再按 inverted 翻转＋90°，inverted 传错方向火花会横躺反向（[rot 逻辑](../../../raw/current/scripts/prefabs/hitsparks_fx.lua#L95-L112)）。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；工厂全文（MakeFX 三注册、Setup 家族、闪色、netvar）、六个消费者调用点（亮茄剑／暗影槌／暗影收割者／亮茄盔甲／嚎弹炮／moon_tree_blossom debuff）、colouradder 组件已核验；hit_3 实际帧数、其余 hitsparks 消费者（如有 mod 内动态拼接）未穷举、晚加入表现未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
