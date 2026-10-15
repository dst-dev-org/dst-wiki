---
title: "探具与仪式工具族（thurible 等六注册）"
created: 2026-11-07
updated: 2026-11-07
type: prefab
summary: "五件 HANDS 槽探具与仪式工具的原版组装：暗影香炉的影怪引诱与暗影标签链、指南针的镜头指针 HUD 与地图显影、探测杖测距环、木手杖移动磨损加速、女巫扫帚清扫与附魔骑乘动画。"
aliases: ["thurible", "thuriblebody", "compass", "diviningrod", "walking_stick", "witchbroom", "暗影香炉", "指南针", "罗盘", "探测杖", "木手杖", "行走手杖", "女巫扫帚", "附魔女巫扫帚", "骑士香炉", "血色香炉", "Shadow Thurible", "Compass", "Divining Rod", "Wooden Walking Stick", "Witch's Broom", "Enchanted Witch's Broom", "探具与仪式工具"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/thurible.lua", "prefabs/thurible_smoke.lua", "prefabs/compass.lua", "prefabs/diviningrod.lua", "prefabs/walking_stick.lua", "prefabs/witchbroom.lua", "prefabs/cane.lua", "prefabs/stalker.lua", "prefabs/livingtree_root.lua", "prefabs/leafpile.lua", "prefabs/nightmarefuel.lua", "prefabs/skinprefabs.lua", "prefabs/globalmapicon.lua", "prefabs/gestalt_guard_evolved.lua", "prefabs/wx78_possessedbody.lua", "prefabs/skilltree_woodie.lua", "brains/stalkerbrain.lua", "stategraphs/SGwilson.lua", "widgets/hudcompass.lua", "screens/mapscreen.lua", "components/equippable.lua", "components/fueled.lua", "components/shadowlevel.lua", "components/maprevealable.lua", "components/sweepable.lua", "components/sweeper.lua", "components/playercontroller.lua", "components/combat_replica.lua", "actions.lua", "stringutil.lua", "ocean_util.lua", "containers.lua", "constants.lua", "tuning.lua", "recipes.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 探具与仪式工具族（thurible 等六注册）

## 快速判断

- 职责与适用：五个源文件注册的六名 HANDS 槽装备（`witchbroom_fx` 归专页）：暗影香炉 `thurible`（燃料型影怪引诱）、指南针 `compass`（镜头指针 HUD）、探测杖 `diviningrod`（测距）、木手杖 `walking_stick`（移动磨损加速）、女巫扫帚 `witchbroom`（清扫＋附魔骑乘动画）；适合查耐久、标签消费与跟随 FX 的参照。
- 优先使用：耐久与燃料查 [[reference/components/fueled]]；装备槽与 walkspeedmult 查 [[reference/components/equippable]]；与 `cane`（步行手杖）辨析见 walking_stick 小节。
- 重要副作用：香炉点燃挂 shadowlure／shadow_item 并生成两个跟随 FX，改变 stalker 系 NPC 行为与月裔守卫敌意；罗盘装备者成为地图可见图标；木手杖仅移动耗耐久且耗尽消失；女巫扫帚清扫扣次、附魔后移动磨损。
- 修改边界：五件全在主模拟分支配组件，公共端只做动画、标签与漂浮（锚点见注册行）；不展开 thurible_smoke 粒子与探测杖距离环 FX（后者已有专页）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `thurible` | fn：公共端物理／动画＋shadow_item、shadowlure、nopunch、shadowlevel 标签 → 主模拟 inspectable、inventoryitem、equippable、fueled(NIGHTMARE)、shadowlevel | 唯一有皮肤的注册，且 `SetSinks(true)` 沉水；装备生成 thuriblebody、点燃生成 thurible_smoke 跟随 FX。[工厂](../../../raw/current/scripts/prefabs/thurible.lua#L232-L300)、[标签](../../../raw/current/scripts/prefabs/thurible.lua#L247-L252)、[注册](../../../raw/current/scripts/prefabs/thurible.lua#L329-L330) |
| `thuriblebody` | thuriblebodyfn：纯视觉 FX 体（bank `thurible`），`persists=false` 不存档 | 装备时按 thurible 皮肤名生成并挂 owner，卸装／移除清理。[工厂](../../../raw/current/scripts/prefabs/thurible.lua#L302-L327) |
| `compass` | fn：公共端 compass、weapon 标签＋漂浮 → 主模拟 inventoryitem、equippable、fueled(1200s)、weapon(10) | fueled.accepting 保持默认 false 不可补燃料，耗尽直接 Remove。[工厂](../../../raw/current/scripts/prefabs/compass.lua#L98-L161)、[耗尽](../../../raw/current/scripts/prefabs/compass.lua#L79-L90) |
| `diviningrod` | fn：公共端 irreplaceable、nonpotatable、diviningrod、nopunch 标签＋小地图图标 → 主模拟 inspectable(getstatus)、key(MAXWELL)、equippable | 无耐久／燃料，装备后循环测距播报。[工厂](../../../raw/current/scripts/prefabs/diviningrod.lua#L149-L201)、[图标](../../../raw/current/scripts/prefabs/diviningrod.lua#L160) |
| `walking_stick` | fn：公共端动画＋漂浮 → 主模拟 fueled(USAGE)、inspectable、inventoryitem、equippable(1.15)、fuel | 无 weapon 组件与 weapon 标签（无伤害加成），仅移动中耗耐久。[工厂](../../../raw/current/scripts/prefabs/walking_stick.lua#L50-L110)、[移动监听](../../../raw/current/scripts/prefabs/walking_stick.lua#L94-L102) |
| `witchbroom` | fn：公共端 weapon、propweapon、propweapon_nopvp 标签＋displaynamefn → 主模拟 inspectable、equippable、finiteuses(125)、weapon(1)、fuel、sweeper | 运行时形态切换：被树根使用后 `MakeLiving` 转附魔态。[工厂](../../../raw/current/scripts/prefabs/witchbroom.lua#L138-L205)、[转附魔](../../../raw/current/scripts/prefabs/witchbroom.lua#L94-L114) |

- equipslot 均为默认 `EQUIPSLOTS.HANDS`（[组件默认值](../../../raw/current/scripts/components/equippable.lua#L29)，五文件均未覆写）。
- 同文件附带注册：`witchbroom_fx`（附魔态隐形联网载体＋拖尾，主页归 [[reference/prefabs/witchbroom_fx|witchbroom_fx]]）、`thurible_smoke`（独立文件，[注册](../../../raw/current/scripts/prefabs/thurible_smoke.lua#L217)）。
- 官方名（.po）：THURIBLE 暗影香炉（[L350390](../../../raw/current/scripts/languages/chinese_s.po#L350390-L350392)）、COMPASS 指南针（[L341605](../../../raw/current/scripts/languages/chinese_s.po#L341605-L341607)）、DIVININGROD 探测杖（[L342140](../../../raw/current/scripts/languages/chinese_s.po#L342140-L342142)）、WALKING_STICK 木手杖（[L351615](../../../raw/current/scripts/languages/chinese_s.po#L351615-L351617)）、WITCHBROOM 女巫扫帚／WITCHBROOM_LIVING 附魔女巫扫帚（[L352320-L352325](../../../raw/current/scripts/languages/chinese_s.po#L352320-L352325)）。

## 关联机制导航

### thurible（暗影香炉）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| fueled | fueltype=NIGHTMARE、满油 `(night_time+dusk_time)×3=540s`、accepting=true 可补噩梦燃料（fuelvalue=LARGE_FUEL=180s）；点燃瞬间先扣 1/60（满油）或 0.5（非满），此后每秒 1 点 | 主模拟；nofuel 摘 shadow_item 并经 itemranout 公告 | [[reference/components/fueled]]；[配置](../../../raw/current/scripts/prefabs/thurible.lua#L279-L285)、[nofuel](../../../raw/current/scripts/prefabs/thurible.lua#L175-L189)、[首扣](../../../raw/current/scripts/components/fueled.lua#L269-L286)、[每秒](../../../raw/current/scripts/components/fueled.lua#L325-L328)、[时长](../../../raw/current/scripts/tuning.lua#L19-L29)、[满油](../../../raw/current/scripts/tuning.lua#L3416-L3417)、[燃料物](../../../raw/current/scripts/prefabs/nightmarefuel.lua#L82-L83) |
| shadowlevel | SetDefaultLevel(4)；LevelFn 有油 4、空油 0 | 主模拟；消费端：shadowmagic 角色（Maxwell）检视追加 ANNOUNCE_SHADOWLEVEL_ITEM | [[reference/components/shadowlevel]]；[配置](../../../raw/current/scripts/prefabs/thurible.lua#L287-L289)、[LevelFn](../../../raw/current/scripts/prefabs/thurible.lua#L228-L230)、[检视消费](../../../raw/current/scripts/stringutil.lua#L338-L345)、[shadowmagic](../../../raw/current/scripts/constants.lua#L1570) |
| shadowlure 标签 | 生成预挂；turnoff 摘、turnon（有油）挂回 | 主模拟；`IsNearShadowLure` 以玩家为中心、6 格 `THURIBLE_AOE_RANGE` 现查：复活的骨架（`stalker`）选敌排除光环内玩家，保留目标时仅当对方不是近期攻击者才放弃；复活的骨架与远古织影者的缠绕筛选都跳过光环内玩家 | [增删](../../../raw/current/scripts/prefabs/thurible.lua#L37-L72)、[6 格判定](../../../raw/current/scripts/prefabs/stalker.lua#L184-L187)、[选敌](../../../raw/current/scripts/prefabs/stalker.lua#L199)、[保留](../../../raw/current/scripts/prefabs/stalker.lua#L306-L312)、[缠绕](../../../raw/current/scripts/prefabs/stalker.lua#L529-L541) |
| 消费端：stalker_npc | Brain 的 FindAndSetLeader 让 20 格内持炉玩家成为跟随 leader；被攻击时攻击者手持 shadowlure 物品则直接 `health:Kill()` | 服务端；SEE_LURE_DIST=20、SAFE_LURE_DIST=5 | [跟随](../../../raw/current/scripts/brains/stalkerbrain.lua#L252-L270)、[面向掉落香炉](../../../raw/current/scripts/brains/stalkerbrain.lua#L511-L514)、[击杀](../../../raw/current/scripts/prefabs/stalker.lua#L1735-L1742)、[注册](../../../raw/current/scripts/prefabs/stalker.lua#L1791)、[距离](../../../raw/current/scripts/brains/stalkerbrain.lua#L10-L11) |
| 消费端：stalker_atrium | 未交战分支 FindClosest(shadowlure) 后面向香炉 | 服务端 Brain | [Brain](../../../raw/current/scripts/brains/stalkerbrain.lua#L617) |
| 获取 | 非衰亡击杀远古织影者时 AddChanceLoot("thurible",1)；无合成配方，仅拆解返料 | stalker_atrium 掉落 | [掉落](../../../raw/current/scripts/prefabs/stalker.lua#L1010-L1019)、[挂接](../../../raw/current/scripts/prefabs/stalker.lua#L1629)、[拆解](../../../raw/current/scripts/recipes.lua#L1677) |
| shadow_item 标签 | ontakefuel 挂、nofuel 摘 | 月裔守卫对装备者常驻最高敌意；WX-78 附身躯壳对此武器有伤害系数分支 | [挂摘](../../../raw/current/scripts/prefabs/thurible.lua#L214-L220)、[守卫](../../../raw/current/scripts/prefabs/gestalt_guard_evolved.lua#L110-L125)、[躯壳](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L286-L306) |
| 关联 Prefab：thuriblebody／thurible_smoke | onequip 生成 body 挂 owner 的 swap_object；turnon 在装备或掉落地面时生成烟体 Follower，入包移除；两侧监听 onremove 清理 | 主模拟生成；皮肤名与 skin_id 传入 body | [onequip](../../../raw/current/scripts/prefabs/thurible.lua#L122-L149)、[turnon](../../../raw/current/scripts/prefabs/thurible.lua#L37-L62) |
| 皮肤 | thurible_chess 骑士香炉、thurible_shadow 血色香炉；持握符号走 OverrideItemSkinSymbol | CreatePrefabSkin base_prefab="thurible" | [定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L12423-L12441)、[名称](../../../raw/current/scripts/languages/chinese_s.po#L407560-L407567) |

### compass（指南针）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| fueled | `COMPASS_FUEL=day_time×4=1200s`；accepting 默认 false，耗尽 announce＋自毁 | 主模拟 | [配置](../../../raw/current/scripts/prefabs/compass.lua#L137-L140)、[耗尽](../../../raw/current/scripts/prefabs/compass.lua#L79-L90)、[常量](../../../raw/current/scripts/tuning.lua#L1719-L1720)、[accepting](../../../raw/current/scripts/components/fueled.lua#L93) |
| weapon | 伤害 UNARMED_DAMAGE=10；SetOnAttack 每次攻击 `DoDelta(maxfuel×-0.3)` 扣 30% 燃料 | 主模拟 | [武器](../../../raw/current/scripts/prefabs/compass.lua#L142-L144)、[攻击扣油](../../../raw/current/scripts/prefabs/compass.lua#L92-L96)、[常量](../../../raw/current/scripts/tuning.lua#L346) |
| maprevealable | onequip 给 owner 挂 AddRevealSource(inst,"compassbearer")，卸装／装模型移除 | 主模拟；有 reveal source 时生成 globalmapicon 并 TrackEntity，restriction 传引擎 MiniMapEntity | [[reference/components/maprevealable]]；[装备挂源](../../../raw/current/scripts/prefabs/compass.lua#L45-L56)、[显影](../../../raw/current/scripts/components/maprevealable.lua#L133-L148)、[图标](../../../raw/current/scripts/components/maprevealable.lua#L7)、[restriction](../../../raw/current/scripts/prefabs/globalmapicon.lua#L14) |
| HUD 指针 | 地图屏创建 HudCompass，按 HANDS 槽装备 compass 标签开关表盘；指针角度＝`TheCamera:GetHeading()-45` 弹簧动画，低理智与满月叠加 sin 摆动（至 720），常驻 ±5 抖动 | 纯客户端 widget | [创建](../../../raw/current/scripts/screens/mapscreen.lua#L41-L42)、[开关](../../../raw/current/scripts/widgets/hudcompass.lua#L4-L16)、[朝向](../../../raw/current/scripts/widgets/hudcompass.lua#L230-L232)、[扰动](../../../raw/current/scripts/widgets/hudcompass.lua#L268-L278) |
| 配方与消费 | 金块1＋燧石1、TECH.NONE；制图桌／哨戒 ward 等以其为材料 | 全角色可制作 | [配方](../../../raw/current/scripts/recipes.lua#L407)、[下游](../../../raw/current/scripts/recipes.lua#L386) |

### diviningrod（探测杖）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| equippable 测距循环 | onequip 起 1s 后 CheckTargetPiece：首次从 `Ents` 建 `tracking_parts` 缓存（`teleportato`／`teleportato_part` 标签与 `adventure_portal` 实体），之后只在该表内筛有效且不在 limbo 的对象；强度＝max(0, 1−d²/300²)，300 外为 0；四档 hot≤50／warmer≤100／warm≤200／cold≤400 决定 ping 间隔（1/2/4/8s）与距离环颜色 | 主模拟；档位和 FX 在距离判断前写入——最近目标超过 400 仍显示 cold 环、ping 用默认 8s；卸装取消任务并移除 FX | [测距](../../../raw/current/scripts/prefabs/diviningrod.lua#L53-L94)、[目标缓存](../../../raw/current/scripts/prefabs/diviningrod.lua#L24-L51)、[距离表](../../../raw/current/scripts/tuning.lua#L2248-L2256) |
| inspectable | getstatus=describe：装备中返回档位大写（COLD…HOT） | 主模拟 | [describe](../../../raw/current/scripts/prefabs/diviningrod.lua#L130-L137)、[挂接](../../../raw/current/scripts/prefabs/diviningrod.lua#L186-L187) |
| key | keytype=LOCKTYPE.MAXWELL | 本版 scripts 全目录 grep 无 lock 端消费者（单机冒险模式遗产） | [key](../../../raw/current/scripts/prefabs/diviningrod.lua#L183-L184) |
| 标签消费 | irreplaceable：沉海回岸／回传送门重生、禁打包；nonpotatable：禁入烹饪锅 | 服务端通用消费 | [沉海重生](../../../raw/current/scripts/ocean_util.lua#L236-L246)、[打包](../../../raw/current/scripts/containers.lua#L475)、[锅](../../../raw/current/scripts/containers.lua#L1134) |
| 生成缺口 | 无配方；检索范围（prefabs、map 布局、debugcommands）无常规生成链；调试上架清单（同列 cane、compass）收录 | 仅 debug 可刷 | [清单](../../../raw/current/scripts/debugcommands.lua#L1840-L1842) |
| 关联 FX | dr_hot_loop／dr_warmer_loop／dr_warm_loop_2／dr_warm_loop_1 四色距离环 | 专页维护 | [[reference/prefabs/dr_warm_loop_1|dr_warm_loop_1]]；[声明](../../../raw/current/scripts/prefabs/diviningrod.lua#L8-L14) |

### walking_stick（木手杖）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| fueled＋locomote | USAGE 型、`no_sewing=true`、`4×total_day_time=1920s`；监听 owner 的 locomote，`WantsToMoveForward` 才 StartConsuming；耗尽 Remove 消失 | 主模拟；不移动不磨损 | [耐久](../../../raw/current/scripts/prefabs/walking_stick.lua#L76-L80)、[移动监听](../../../raw/current/scripts/prefabs/walking_stick.lua#L94-L102)、[穿脱挂监听](../../../raw/current/scripts/prefabs/walking_stick.lua#L6-L23)、[常量](../../../raw/current/scripts/tuning.lua#L7823) |
| equippable | walkspeedmult=WALKING_STICK_SPEED_MULT=1.15 | 主模拟 | [[reference/components/equippable]]；[配置](../../../raw/current/scripts/prefabs/walking_stick.lua#L84-L89)、[常量](../../../raw/current/scripts/tuning.lua#L414-L415) |
| fuel／burnable | fuelvalue=LARGE_FUEL(180s) 可作火源燃料；MakeSmallBurnable＋MakeSmallPropagator | 主模拟 | [fuel](../../../raw/current/scripts/prefabs/walking_stick.lua#L91-L92)、[燃烧](../../../raw/current/scripts/prefabs/walking_stick.lua#L104-L105)、[常量](../../../raw/current/scripts/tuning.lua#L1676-L1677) |
| 配方 | log×3＋charcoal×1、TECH.NONE、`builder_skill="woodie_human_lucy_3"`（Lucy 雕刻 3 级），sg_state="carvewood" | 限定 Woodie 技能树 | [配方](../../../raw/current/scripts/recipes.lua#L175)、[技能](../../../raw/current/scripts/prefabs/skilltree_woodie.lua#L382-L387) |

**与 cane（步行手杖）的辨析**：两件是不同 prefab、注册名与动画资源。`cane` 无耐久、伤害 34×系数×0.5、加速 1.25、七款皮肤；`walking_stick` 无 weapon、加速 1.15、1920 秒移动磨损耗尽即消失、无皮肤。详见 [[reference/prefabs/cane]]。

### witchbroom（女巫扫帚）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| finiteuses＋sweeper | 125 次，`SetConsumption(ACTIONS.SWEEP,1)` 每次清扫扣 1，耗尽 Remove；对"sweeper 工具＋sweepable 目标"给 SWEEP 动作，`sweepable:Sweep` 播扫帚音效＋走目标回调 | 主模拟；目标示例：落叶堆 sweep 后掉落战利品并碎裂移除 | [[reference/components/finiteuses]]、[[reference/components/sweeper]]；[耐久](../../../raw/current/scripts/prefabs/witchbroom.lua#L179-L183)、[SWEEP](../../../raw/current/scripts/actions.lua#L7126-L7133)、[可用性](../../../raw/current/scripts/components/playercontroller.lua#L2063)、[sweepable](../../../raw/current/scripts/components/sweepable.lua#L24-L32)、[落叶堆](../../../raw/current/scripts/prefabs/leafpile.lua#L433-L437) |
| weapon／propweapon | SetDamage(1)（源码注释：SG 接管伤害，此值基本无效）；攻击走 `attack_prop_pre` 状态；propweapon_nopvp 参与 PVP 判定 | 主模拟＋客户端判定 | [weapon](../../../raw/current/scripts/prefabs/witchbroom.lua#L185-L186)、[SG 分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1413)、[PVP](../../../raw/current/scripts/components/combat_replica.lua#L307)、[标签](../../../raw/current/scripts/prefabs/witchbroom.lua#L153-L155) |
| propsmashed | 监听 propsmashed：扣 5 次＋播放清扫音效 | 主模拟 | [回调](../../../raw/current/scripts/prefabs/witchbroom.lua#L64-L71)、[监听](../../../raw/current/scripts/prefabs/witchbroom.lua#L198)、[常量](../../../raw/current/scripts/tuning.lua#L9800-L9803) |
| 附魔 MakeLiving | livingtree_root（完全正常的树根）用于非附魔扫帚后：置 living、加 `flyingbroom`、换 living_build 与图标、`walkspeedmult=1.1`、补满耐久＋生成 FX；此后移动每秒 `Use(125/(10×total_day_time))≈0.026 次`（约 10 总日耗尽） | 主模拟；附魔态随存档恢复 | [使用入口](../../../raw/current/scripts/prefabs/livingtree_root.lua#L79-L92)、[SetLiving](../../../raw/current/scripts/prefabs/witchbroom.lua#L74-L92)、[移动磨损](../../../raw/current/scripts/prefabs/witchbroom.lua#L64-L66)、[名称](../../../raw/current/scripts/languages/chinese_s.po#L344765-L344767) |
| flyingbroom 消费端 | SG 普通奔跑分支记录 normalflyingbroom，切入 run_flyingbroom_start／run_flyingbroom（骑乘奔跑动画）；卸装、小心行走、风暴／瘴气无护目回退普通 run | 玩家 SG 共用；纯观感，不改变位移 | [分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L500-L507)、[触发](../../../raw/current/scripts/stategraphs/SGwilson.lua#L11305-L11306)、[状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L28832-L28844) |
| 配方 | 树枝2＋绳索1＋草3、`TECH.HALLOWED_NIGHTS`（万圣夜活动期以 `{SCIENCE=10}` 临时解锁） | 需世界开启 Hallowed Nights | [配方](../../../raw/current/scripts/recipes.lua#L1415)、[门禁](../../../raw/current/scripts/constants.lua#L1311) |
| 关联 FX | 附魔态装备期生成 witchbroom_fx，非专服每 5 帧从本地轨迹池投放三变体拖尾 | 专页维护 | [[reference/prefabs/witchbroom_fx|witchbroom_fx]]；[fx_DoTrail](../../../raw/current/scripts/prefabs/witchbroom.lua#L261-L279)、[载体](../../../raw/current/scripts/prefabs/witchbroom.lua#L281-L303) |

## 典型接入

### 让自定义手持物接入香炉标签链

- 前提与选择：让自定义手持装备复用 shadowlure／shadow_item 消费端；消费者全在服务端，主模拟加标签即生效。
- modmain 环境注意：模组环境由 mods.lua 的 CreateEnvironment 提供、无 `__index=_G`——`AddPrefabPostInit` 等注入接口直用，`TheWorld`／`SpawnPrefab` 等游戏全局取 `GLOBAL.`，`TUNING` 直用。

```lua
-- modmain.lua
AddPrefabPostInit("my_tool", function(inst)
    inst:DoTaskInTime(0, function()
        if GLOBAL.TheWorld ~= nil and GLOBAL.TheWorld.ismastersim then
            inst:AddTag("shadowlure")   -- stalker_npc 将跟随持物者；持物攻击可直接击杀 stalker_npc
            -- inst:AddTag("shadow_item") -- 需触发月裔守卫敌意等暗影装备链时再加
        end
    end)
end)
```

- 生效与撤销：标签随实体网络同步，撤销即 RemoveTag；原版对照即 thurible turnon/turnoff 的增删（thurible 小节锚点）。
- 调整与验证：`DoTaskInTime(0)` 避开构造期；本例消费端均服务端，无需客户端标签；未做游戏运行验证。

### 借用"移动才磨损"与加速模式

- 前提与选择：新装备要"移动时耗耐久"时复制 walking_stick 模式：穿脱回调监听 owner 的 `locomote`，按 `WantsToMoveForward` 启停 fueled 消耗；纯加速直接设 `equippable.walkspeedmult`。
- 生效与撤销：监听随穿脱挂／摘；女巫扫帚附魔态同款任务入口见 SetLiving。
- 原版依据见上行锚点；完整造物工程见 [[cookbook/equipment/hand]]，字段见 [[reference/components/equippable]]。

## 常见陷阱与限制

- **误区：暗影香炉“驱影／驱怪”**。实际是引诱：stalker 系 NPC 面向或跟随 shadowlure，持炉攻击可直接击杀 stalker_npc；`THURIBLE_AOE_RANGE=6` 是战斗侧光环——复活的骨架选敌排除、保留时仅对非近期攻击者放弃，复活的骨架与远古织影者的缠绕筛选都跳过光环内玩家，不是伤害半径。[判定](../../../raw/current/scripts/prefabs/stalker.lua#L184-L187)、[缠绕](../../../raw/current/scripts/prefabs/stalker.lua#L529-L541)
- **误区：指南针指示世界方向或北方**。指针角度是摄像机朝向减 45 度的弹簧动画，叠加理智低下与满月的大幅摆动，且只在地图屏出现；是镜头方位仪＋氛围装置，不是寻路工具。[朝向](../../../raw/current/scripts/widgets/hudcompass.lua#L230-L232)
- **误区：木手杖是步行手杖（cane）的变体或皮肤**。两者独立注册，数值耐久均不同（1.15＋磨损 vs 1.25＋无耐久）。
- **边界：女巫扫帚"附魔"没有真实飞行或骑乘位移**。flyingbroom 只切换玩家 SG 的骑乘奔跑动画并给 1.1 倍移速，碰撞与移动仍走普通 locomotor。
- **边界：香炉会沉水**。thurible `SetSinks(true)` 且无 irreplaceable，掉入海洋即消失；对照探测杖（漂浮＋irreplaceable 回岸）与罗盘（漂浮但耗尽自毁）。[沉没](../../../raw/current/scripts/prefabs/thurible.lua#L272)
- **边界：探测杖在本版无常规获取途径**，key(MAXWELL) 无 lock 消费者；复用其测距逻辑时注意目标标签写死为 teleportato 系。

## 验证记录

- 源码审阅：2026-11-07，Steam buildid 26335394。实读五个 prefab 与 cane.lua 全文；核验六注册名、工厂分界、组件／动作链、燃料与耐久换算、四条配方、官方名与皮肤名（.po）、消费端 grep（shadowlure、shadow_item、shadowlevel、flyingbroom、irreplaceable、sweepable、HALLOWED_NIGHTS、LOCKTYPE.MAXWELL、THURIBLE_AOE_RANGE）；LOCKTYPE.MAXWELL 消费端零命中。
- 游戏运行：未运行；540s／1200s／1920s／0.026 次·秒⁻¹ 均为源码推导值；皮肤观感与 thurible_smoke 粒子未展开。
- 结构检查：validate_page 实测 144 行／24490 B，standard 档（240 行／24 KiB）内通过。

返回 [[reference/prefabs/tools-field-index|工具与野外实用]]、[[reference/prefabs/index|Prefab 索引]]。
