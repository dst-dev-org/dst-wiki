---
title: "投掷炸弹与雪球（sleepbomb、pumpkin_bomb、bomb_lunarplant、snowball_item）"
created: "2026-11-07"
updated: "2026-11-08"
type: prefab
summary: "四注册手持投掷物：睡袋、南瓜炸弹、亮茄炸弹都走 complexprojectile 抛物线，但只有南瓜／亮茄命中瞬间动态挂 explosive 并按堆叠爆炸（50 物理＋击退／200 位面），睡袋命中只生成睡眠云；雪球走 projectile 追踪，命中唤醒、0 伤、降温灭火并扩散冷度（可冻结）；weapon 0 伤、HANDS 整叠装备，爆炸参数命中时读 TUNING。"
aliases: ["sleepbomb", "pumpkin_bomb", "bomb_lunarplant", "snowball_item", "睡袋", "南瓜炸弹", "亮茄炸弹", "雪球", "Napsack", "Pumpkin Bomb", "Brightshade Bomb", "Snowball", "投掷武器", "投掷炸弹"]
tags: [prefab, item, equipment]
size_class: standard
dst_build: "steam-buildid:26474873"
sources: ["prefabs/sleepbomb.lua", "prefabs/pumpkin_bomb.lua", "prefabs/bomb_lunarplant.lua", "prefabs/snowball_item.lua", "prefabs/sleepcloud.lua", "components/explosive.lua", "components/complexprojectile.lua", "components/projectile.lua", "components/combat.lua", "components/equippable.lua", "components/inventory.lua", "components/wateryprotection.lua", "components/snowballmelting.lua", "components/snowballmanager.lua", "components/damagetypebonus.lua", "components/workable.lua", "components/builder.lua", "recipes.lua", "recipes_filter.lua", "tuning.lua", "constants.lua", "componentutil.lua", "languages/chinese_s.po", "prefabs/toadstool.lua", "prefabs/blueprint.lua", "prefabs/pigking.lua", "prefabs/mermking.lua", "prefabs/leafpile.lua", "map/boons.lua", "prefabs/snowman.lua", "prefabs/waterprojectiles.lua", "prefabs/firesuppressor.lua", "prefabs/wateringcan.lua", "stategraphs/SGworm_boss_head.lua", "actions.lua", "fx.lua", "mods.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 投掷炸弹与雪球（sleepbomb、pumpkin_bomb、bomb_lunarplant、snowball_item）

## 快速判断

- 职责与适用：四注册手持投掷物：weapon 0 伤＋HANDS 整叠装备（`equipstack`），效果全在命中回调；三枚炸弹走 `complexprojectile` 抛物线（睡袋熔炉不挂 weapon），雪球走 `projectile` 追踪。[炸弹投掷链](../../../raw/current/scripts/components/combat.lua#L1177-L1185)、[雪球投掷链](../../../raw/current/scripts/components/combat.lua#L1170-L1175)、[HANDS 默认槽](../../../raw/current/scripts/components/equippable.lua#L29)
- 优先使用：要"落地 AoE 爆炸"复刻南瓜／亮茄接线（爆炸组件命中瞬间动态挂载；睡袋不挂 explosive，见[关联机制导航](#关联机制导航)）；要范围催眠直接生成 `sleepcloud`（机制归 [[reference/prefabs/sleepcloud]]）；"飞出可接回"的回旋镖式投掷见 [[reference/prefabs/boomerang]]，不在本页范围。
- 重要副作用：命中瞬间结算——南瓜炸弹动态挂 explosive，50 物理×堆叠＋范围 3.5 击退＋10% 运气掷点掉 `pumpkin_seeds`；亮茄炸弹物理 0＋200 位面伤害×堆叠、对 `shadow_aligned` ×1.1；睡袋不挂 explosive、0 伤害，只生成 `sleepbomb_burst`＋`sleepcloud`；雪球唤醒睡眠者、按条件推 0 伤 `attacked`、经 wateryprotection 降温／灭火／扩散冷度 0.2，并整叠移除。[南瓜配置](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L23-L42)、[亮茄配置](../../../raw/current/scripts/prefabs/bomb_lunarplant.lua#L15-L35)、[睡袋 OnHit](../../../raw/current/scripts/prefabs/sleepbomb.lua#L16-L21)、[雪球 OnHit](../../../raw/current/scripts/prefabs/snowball_item.lua#L28-L50)
- 修改边界：标签与 reticule 挂公共端、玩法组件挂主模拟（[分界](../../../raw/current/scripts/prefabs/sleepbomb.lua#L109-L128)）；南瓜／亮茄的 `explosive`／`planardamage` 是命中时动态挂载的，postinit 拿不到实例组件（睡袋没有这两个组件），爆炸数值在命中时刻读 `TUNING`（见[典型接入](#典型接入)）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `sleepbomb`（睡袋） | 单注册 return：`fn` 直构 → 公共端三标签＋reticule → 主模拟 locomotor＋complexprojectile(15/-35)＋weapon(0, 8-10)＋equippable(equipstack) | 熔炉模式改 AoE 准星＋`nopunch`、不装 weapon；命中生成 `sleepbomb_burst`＋`sleepcloud`。[return](../../../raw/current/scripts/prefabs/sleepbomb.lua#L146)、[工厂](../../../raw/current/scripts/prefabs/sleepbomb.lua#L67-L144)、[熔炉分支](../../../raw/current/scripts/prefabs/sleepbomb.lua#L95-L105) |
| `pumpkin_bomb`（南瓜炸弹） | 双注册 return：本体 `fn` 同上但无 locomotor、公共端多 `explosive` 标签；伴生 FX `pumpkin_bomb_explode_fx` | 命中 `ConfigureExplosive`：50×堆叠、建筑 4、范围 3.5、击退、不引燃、10% 掉种；`ExplodeOnChew` 供啃食引爆。[return](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L329-L330)、[工厂](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L118-L184)、[FX](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L296-L324) |
| `bomb_lunarplant`（亮茄炸弹） | 双注册 return：本体 `fn` 同睡袋（有 locomotor）＋`toughworker`/`explosive` 标签＋`spinticks` net_smallbyte；伴生 FX `bomb_lunarplant_explode_fx` | 命中：爆炸伤害 0、挂 planardamage 200＋对影 ×1.1；投掷时客户端建本地发光核心；`ExplodeOnChew` 同上。[return](../../../raw/current/scripts/prefabs/bomb_lunarplant.lua#L296-L297)、[工厂](../../../raw/current/scripts/prefabs/bomb_lunarplant.lua#L180-L251)、[FX](../../../raw/current/scripts/prefabs/bomb_lunarplant.lua#L260-L291) |
| `snowball_item`（雪球） | 单注册 return：`fn` 直构 → 公共端标签 `frozen`/`icebox_valid`/`extinguisher`/`watersource`/`weapon`/`projectile`＋snowmandecoratable → 主模拟 smotherer/pushable/perishable(2 天)/watersource(0.3)/wateryprotection/projectile(15, 30)/snowballmelting | 走 `projectile` 而非 complexprojectile；不爆炸，命中降温/灭火并扩散冷度；可推滚成雪人、可浇花壶、会融化腐败。[return](../../../raw/current/scripts/prefabs/snowball_item.lua#L275)、[工厂](../../../raw/current/scripts/prefabs/snowball_item.lua#L169-L272) |

共同投掷链（三枚炸弹；睡袋仅普通世界，熔炉不挂 weapon）：攻击 → combat 见武器带 `complexprojectile` 组件 → `DropItem` 脱手 → `Launch` 解算抛物线（速度 15、重力 −35、偏移 (.25,1,0)）→ `onthrown` 改物理＋`spin_loop` → 落地 `Hit` → 文件内 `OnHit`。[投掷分支](../../../raw/current/scripts/components/combat.lua#L1177-L1185)、[Launch](../../../raw/current/scripts/components/complexprojectile.lua#L134-L180)、[落地](../../../raw/current/scripts/components/complexprojectile.lua#L198-L211)、[onthrown 示例](../../../raw/current/scripts/prefabs/sleepbomb.lua#L34-L50)。玩家 SG 播普通 `attack`（无 `thrown` 标签）。

名称经简中 .po `msgctxt` 核验：`STRINGS.NAMES.SLEEPBOMB`＝睡袋（Napsack）、`PUMPKIN_BOMB`＝南瓜炸弹、`BOMB_LUNARPLANT`＝亮茄炸弹（Brightshade Bomb）、`SNOWBALL_ITEM`＝雪球；[睡袋](../../../raw/current/scripts/languages/chinese_s.po#L349450-L349452)、[南瓜](../../../raw/current/scripts/languages/chinese_s.po#L347340-L347342)、[亮茄](../../../raw/current/scripts/languages/chinese_s.po#L339825-L339827)、[雪球](../../../raw/current/scripts/languages/chinese_s.po#L349695-L349697)

获取途径：

| 注册名 | 途径 | 证据 |
|---|---|---|
| `sleepbomb` | 悲惨的毒菌蟾蜍（`toadstool_dark`）100% 掉 `sleepbomb_blueprint`（普通毒菌蟾蜍掉落表无此项），学习后进 `builder.recipes` 绕过科技门；配方 shroom_skin 1＋canary_poisoned 1、`TECH.LOST`、一次产 4 | [掉落](../../../raw/current/scripts/prefabs/toadstool.lua#L113-L140)、[蓝图注册](../../../raw/current/scripts/prefabs/blueprint.lua#L241-L242)、[已学习免科技](../../../raw/current/scripts/components/builder.lua#L883-L886)、[配方](../../../raw/current/scripts/recipes.lua#L484)、[过滤器·武器](../../../raw/current/scripts/recipes_filter.lua#L1038) |
| `pumpkin_bomb` | 无配方。猪王／鱼人王收糖果先按实际糖果数（含装扮加成）取 `% 8` 余数，余数>0 才掷一次 20% 运气、成功生成该数量；落叶堆 TREAT 30%；万圣夜恩惠布局战利品 | [猪王](../../../raw/current/scripts/prefabs/pigking.lua#L113-L119)、[鱼人王](../../../raw/current/scripts/prefabs/mermking.lua#L179-L185)、[落叶堆](../../../raw/current/scripts/prefabs/leafpile.lua#L82-L88)、[boons](../../../raw/current/scripts/map/boons.lua#L12-L73)、[掷点](../../../raw/current/scripts/componentutil.lua#L2237-L2249) |
| `bomb_lunarplant` | 配方 purebrilliance 4＋lunarplant_husk 4＋moonglass_charged 1、`TECH.LUNARFORGING_TWO`（月之锻造台二级，`nounlock`＋`station_tag="lunar_forge"`）、一次产 6 | [配方](../../../raw/current/scripts/recipes.lua#L850)、[TECH](../../../raw/current/scripts/constants.lua#L1331-L1332)、[过滤器·LUNAR_FORGE](../../../raw/current/scripts/recipes_filter.lua#L445) |
| `snowball_item` | 无配方。冬季 snowballmanager 在玩家周边地面刷（有密度上限）；雪人推滚还原／拆毁／读档还原成雪球 | [刷雪球](../../../raw/current/scripts/components/snowballmanager.lua#L91-L112)、[雪人还原](../../../raw/current/scripts/prefabs/snowman.lua#L80-L91)、[拆毁掉落](../../../raw/current/scripts/prefabs/snowman.lua#L515-L527)、[读档还原](../../../raw/current/scripts/prefabs/snowman.lua#L580-L592) |

## 关联机制导航

### 组件与同步

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：complexprojectile | 三枚炸弹 `SetHorizontalSpeed(15)`＋`SetGravity(-35)`＋`SetLaunchOffset(.25,1,0)`＋OnLaunch/OnHit | 主模拟；`ismeleeweapon` 默认 false 才走投掷分支 | [[reference/components/complexprojectile]]；[挂载](../../../raw/current/scripts/prefabs/sleepbomb.lua#L117-L122)、[Launch](../../../raw/current/scripts/components/complexprojectile.lua#L134-L180) |
| 组件：projectile | 雪球 `SetSpeed(15)`＋`SetHitDist(1.5)`＋`SetRange(30)`＋OnHit/OnThrown/OnMiss(移除)；homing 默认 true | 主模拟；`Hit` 先按 weapon 结算（0 伤）再 `onhit`，`Miss`→`inst.Remove` | [[reference/components/projectile]]；[挂载](../../../raw/current/scripts/prefabs/snowball_item.lua#L245-L251)、[Throw](../../../raw/current/scripts/components/projectile.lua#L160-L190)、[Hit](../../../raw/current/scripts/components/projectile.lua#L236-L266) |
| 组件：weapon | 三枚炸弹＋雪球 `SetDamage(0)`＋`SetRange(8, 10)`；睡袋仅普通世界挂 weapon、熔炉分支不挂；只作投掷触发 | 主模拟；伤害不在武器侧 | [[reference/components/weapon]]；[炸弹](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L171-L173)、[雪球](../../../raw/current/scripts/prefabs/snowball_item.lua#L241-L243) |
| 组件：explosive | 命中时 `ConfigureExplosive` 临时挂载：南瓜 50/3.5/建筑 4/击退/不引燃，亮茄 0 伤/3/不引燃；睡袋不挂；`OnBurnt` 即爆 | 主模拟、命中瞬间；`ispvp` 由 onthrown 按投掷者标签选 `SetAttacker`/`SetPvpAttacker`；玩家投掷时其他玩家跳过生命值伤害，南瓜 `forcepvp` 仍推击退、亮茄无击退数据则不结算 | [[reference/components/explosive]]；[配置](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L23-L42)、[OnBurnt](../../../raw/current/scripts/components/explosive.lua#L31-L158) |
| 组件：planardamage／damagetypebonus | 亮茄命中时挂 planardamage 200＋`AddBonus("shadow_aligned", 1.1)`；explosive 经 SpDamageUtil 收集并×堆叠 | 命中瞬间；对 `shadow_aligned` ×1.1 | [[reference/components/planardamage]]；[配置](../../../raw/current/scripts/prefabs/bomb_lunarplant.lua#L15-L22)、[GetBonus](../../../raw/current/scripts/components/damagetypebonus.lua#L42-L57)、[收集](../../../raw/current/scripts/components/spdamageutil.lua#L24) |
| 组件：equippable | HANDS＋`equipstack`：装备/投掷都按整叠，堆叠越高伤害越高 | 主模拟；OnEquip 覆盖 `swap_object`＋`ARM_carry` | [[reference/components/equippable]]；[整叠脱手](../../../raw/current/scripts/components/inventory.lua#L1326-L1332)、[equipstack](../../../raw/current/scripts/prefabs/sleepbomb.lua#L136-L139) |
| 组件：stackable | 睡袋/南瓜/亮茄默认上限 20；雪球 40 | 主模拟；爆炸伤害×堆叠见 explosive 行 | [[reference/components/stackable]]；[雪球](../../../raw/current/scripts/prefabs/snowball_item.lua#L221-L222)、[默认值](../../../raw/current/scripts/components/stackable.lua#L38-L39) |
| 组件：reticule | 三枚炸弹客户端准星：targetfn 6.5→3.5 找可落点、`ease` | 公共端；熔炉模式睡袋换 `reticuleaoe` 准星 | [[reference/components/reticule]]；[配置](../../../raw/current/scripts/prefabs/sleepbomb.lua#L88-L99) |
| 组件：wateryprotection | 雪球：灭火 −1、降温 2、防枯萎 15s、加冷度 0.2、范围 1.25、`AddIgnoreTag("player")` | 主模拟；OnHit 对目标 `SpreadProtection` | [[reference/components/wateryprotection]]；[配置](../../../raw/current/scripts/prefabs/snowball_item.lua#L231-L237)、[扩散](../../../raw/current/scripts/components/wateryprotection.lua#L43-L60) |
| 组件：watersource／snowballmelting／perishable | 雪球可浇花壶（0.3 次）；`snowballmelting` 每 30±5s 查温融雪；`perishable` 2 游戏日到期转移湿度 | 主模拟；入包停融、落地恢复融 | [[reference/components/snowballmelting]]、[[reference/components/perishable]]；[取水](../../../raw/current/scripts/prefabs/snowball_item.lua#L227-L229)、[浇花壶消费](../../../raw/current/scripts/prefabs/wateringcan.lua#L29-L31)、[融化判定](../../../raw/current/scripts/components/snowballmelting.lua#L31-L34)、[腐败回调](../../../raw/current/scripts/prefabs/snowball_item.lua#L74-L101) |
| 组件：pushable／snowmandecoratable | 雪球可推滚，开始推即换成小型雪人并转 `snowaccum` | 主模拟；雪球本体即雪人小底座 | [[reference/components/pushable]]；[替换](../../../raw/current/scripts/prefabs/snowball_item.lua#L125-L147)、[装饰尺寸](../../../raw/current/scripts/prefabs/snowball_item.lua#L200-L201) |
| 组件：smotherer／标签 `extinguisher`/`frozen` | 雪球扑灭阴燃（SMOTHER）与 MANUALEXTINGUISH 灭明火 | 主模拟动作；smotherer 组件是空壳，动作在 actions.lua 定义 | [[reference/components/smotherer]]；[SMOTHER](../../../raw/current/scripts/actions.lua#L1844-L1849)、[灭火](../../../raw/current/scripts/actions.lua#L1852-L1857) |
| 同步：net_smallbyte `spinticks` | 亮茄投掷时服务端 `set(3)`，客户端 `spinticksdirty` 后本地周期任务淡入发光核心（30 tick 到 0.6 加色） | 仅表现层；核心为本地非联网实体、跟 `bomb_lunarplant_follow` 符号 | [netvar](../../../raw/current/scripts/prefabs/bomb_lunarplant.lua#L215)、[客户端](../../../raw/current/scripts/prefabs/bomb_lunarplant.lua#L123-L136) |
| API／数据：TUNING | 爆炸参数命中时读取 | `ConfigureExplosive` 与 sleepcloud timer 实参 | [南瓜](../../../raw/current/scripts/tuning.lua#L9806-L9810)、[亮茄](../../../raw/current/scripts/tuning.lua#L394-L399)、[睡袋](../../../raw/current/scripts/tuning.lua#L1939)、[雪球](../../../raw/current/scripts/tuning.lua#L8561-L8572) |

### 生成宿主与消费者

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 关联 Prefab：`sleepcloud` | 睡袋 OnHit 在命中点生成；云自带 20s `disperse` timer 与 0.5s 周期范围催眠 | 主模拟；机制归 [[reference/prefabs/sleepcloud]] | [OnHit](../../../raw/current/scripts/prefabs/sleepbomb.lua#L16-L21)、[timer](../../../raw/current/scripts/prefabs/sleepcloud.lua#L349-L352) |
| 关联 Prefab：`sleepbomb_burst` | 睡袋爆裂 FX（fx.lua 通用注册） | 主模拟；页面归 [[reference/prefabs/glass_fx]] | [OnHit 内](../../../raw/current/scripts/prefabs/sleepbomb.lua#L19)、[fx 注册](../../../raw/current/scripts/fx.lua#L1708-L1710) |
| 关联 Prefab：`pumpkin_bomb_explode_fx`／`bomb_lunarplant_explode_fx` | 各文件伴生 FX：播 `used`＋爆炸声、`animover` 自移除；南瓜 FX 唤醒时掷 6-7 片物理碎片约 1.5s 侵蚀 | 联网壳＋本地碎片；`persists=false` 不入档 | [南瓜 FX](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L296-L324)、[碎片](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L262-L289)、[亮茄 FX](../../../raw/current/scripts/prefabs/bomb_lunarplant.lua#L260-L291) |
| 关联 Prefab：`pumpkin_seeds` | 南瓜爆炸时 10% 运气掷点经临时 `lootdropper` 掉落 | 主模拟；`TryLuckRoll` 受实体幸运修正 | [配置](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L24-L27)、[掷点](../../../raw/current/scripts/componentutil.lua#L2237-L2249) |
| 消费者：SGworm_boss_head | 南瓜/亮茄的 `ExplodeOnChew`：虫 Boss 吞食分支对 `item.ExplodeOnChew` 引爆（亮茄常驻无 explosive 组件） | 主模拟；燃烧物走 `explosive:OnBurnt` | [吞食](../../../raw/current/scripts/stategraphs/SGworm_boss_head.lua#L230-L240)、[入口](../../../raw/current/scripts/prefabs/bomb_lunarplant.lua#L37-L39) |
| 消费者：workable `toughworker` 标签 | 亮茄公共端带 `toughworker`：爆炸可对 `tough` 级 workable 结算（worker=弹体本体） | 主模拟；睡袋/南瓜无此标签 | [标签](../../../raw/current/scripts/prefabs/bomb_lunarplant.lua#L192)、[消费](../../../raw/current/scripts/components/workable.lua#L199-L204) |
| 消费者：wateringcan | 雪球 `override_fill_uses = 0.3`：浇花壶从雪球取水加 0.3 次数而非满壶 | 主模拟 | [配置](../../../raw/current/scripts/prefabs/snowball_item.lua#L229)、[消费](../../../raw/current/scripts/prefabs/wateringcan.lua#L29-L31) |
| 边界：firesuppressor 的 `snowball` | 灭火器投的是另一注册 `snowball`（waterprojectiles.lua），不是 `snowball_item` | 同名不同注册 | [生成](../../../raw/current/scripts/prefabs/firesuppressor.lua#L28)、[注册](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L401) |
| 资源：anim | `sleepbomb.zip`＋`swap_sleepbomb.zip`、`pumpkin_bomb.zip`、`bomb_lunarplant.zip`＋复用 `sleepcloud.zip` 符号、`snowball.zip` | 静态声明；亮茄 FX 的 `sleepcloud_pre` 符号覆盖仅视觉 | [睡袋](../../../raw/current/scripts/prefabs/sleepbomb.lua#L1-L5)、[亮茄](../../../raw/current/scripts/prefabs/bomb_lunarplant.lua#L1-L5)、[符号覆盖](../../../raw/current/scripts/prefabs/bomb_lunarplant.lua#L278) |

## 典型接入

### 调整爆炸与投掷数值

- 前提与选择：`modmain.lua`。南瓜／亮茄的爆炸组件是命中瞬间动态挂载的，`AddPrefabPostInit` 里 `inst.components.explosive` 为 nil；但 `ConfigureExplosive` 每次命中都从 `TUNING` 读值，改 TUNING 即改之后所有爆炸。
- 生效与撤销：TUNING 修改全局生效，停用模组需手动还原；雪球水属性在装配期写进实例组件，走 postinit。
- 原版依据：南瓜命中读 [TUNING](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L29-L31)；亮茄命中读 [TUNING](../../../raw/current/scripts/prefabs/bomb_lunarplant.lua#L18-L26)；睡眠云时长 [timer](../../../raw/current/scripts/prefabs/sleepcloud.lua#L349-L352)；mod 环境内置 TUNING 无需 GLOBAL 前缀，其余游戏全局需 `GLOBAL.`（[CreateEnvironment](../../../raw/current/scripts/mods.lua#L258-L286)）。

```lua
-- modmain.lua：环境已内置 TUNING，TheWorld 等全局需 GLOBAL. 前缀
TUNING.PUMPKIN_BOMB_DAMAGE = 100            -- 南瓜炸弹爆炸物理伤害（默认 50）
TUNING.PUMPKIN_BOMB_SEED_CHANCE = 0.3       -- 掉种子概率（默认 0.1）
TUNING.BOMB_LUNARPLANT_PLANAR_DAMAGE = 400  -- 亮茄炸弹位面伤害（默认 200）
TUNING.SLEEPBOMB_DURATION = 40              -- 睡眠云消散计时（默认 20）

AddPrefabPostInit("snowball_item", function(inst)
    if GLOBAL.TheWorld == nil or not GLOBAL.TheWorld.ismastersim then
        return -- wateryprotection 仅主模拟
    end
    inst.components.wateryprotection.temperaturereduction = 4 -- 降温 2 → 4
end)
```

- 调整与验证：伤害单位为生命值/次爆炸，位面按位面防御结算；`PUMPKIN_BOMB_RANGE` 既作爆炸查询半径（命中时现读）与击退事件 `radius`（模块加载时缓存进 `EXPLOSION_KNOCKBACK_DATA`，之后改 TUNING 只影响前者）；投掷速度/重力是文件内 hardcode，只能 postinit 改组件。

## 常见陷阱与限制

- **误区：postinit 能拿到南瓜／亮茄的 explosive 组件**：`explosive`/`planardamage` 在命中回调 `ConfigureExplosive` 里临时 `AddComponent`；要改爆炸行为只能改 TUNING 或包 `OnHit`。[动态挂载](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L23-L42)
- **边界：爆炸伤害按整叠堆叠、整叠一起消失**：`equipstack = true` 让装备与投掷都取整叠；爆炸 `totaldamage = explosivedamage × stacksize`，`OnBurnt` 结尾 `Remove()` 整叠；雪球整叠命中只结算一次降温/灭火/冷度扩散。[堆叠伤害](../../../raw/current/scripts/components/explosive.lua#L48-L49)、[移除](../../../raw/current/scripts/components/explosive.lua#L157)、[整叠脱手](../../../raw/current/scripts/components/inventory.lua#L1326-L1332)
- **边界：爆炸攻击者是弹体不是投掷者**：`GetAttacked(self.inst, …)` 有意传弹体本体，不吃攻击者加成；投掷者只用于 `SuggestTarget`。投掷者为玩家时其他玩家跳过生命值伤害（南瓜 `forcepvp` 仍进击退，亮茄无击退数据则整体不结算）。[结算注释](../../../raw/current/scripts/components/explosive.lua#L114-L118)、[仇恨](../../../raw/current/scripts/components/explosive.lua#L123-L129)
- **边界：睡袋在非 PVP 放不倒玩家**：`sleepcloud` 非 PVP 分支要求 `sleeper` 标签且排除 `player`；玩家只在 PVP 世界走 `grogginess`。催眠参数归 [[reference/prefabs/sleepcloud]]。[筛选](../../../raw/current/scripts/prefabs/sleepcloud.lua#L238-L249)
- **边界：雪球靠冷度扩散冻结，不是直接调用 freezable**：命中以目标为中心 `SpreadProtection`，范围内非 `player`（`AddIgnoreTag("player")`）且有 `freezable` 的实体收到 `AddColdness(0.2)`，累积到抗性即冻结；命中事件仍按条件推 0 伤 `attacked`、唤醒睡眠者；融化增湿走 `perishable`。[扩散](../../../raw/current/scripts/components/wateryprotection.lua#L21-L60)、[冷度](../../../raw/current/scripts/components/freezable.lua#L137-L158)、[配置](../../../raw/current/scripts/prefabs/snowball_item.lua#L231-L237)
- **边界：雪球 `splash_snow_fx` 未声明在 prefabs 数组**：OnHit 直接 `SpawnPrefab`（该 FX 在 fx.lua 通用注册），`prefabs` 数组缺失只影响预载提示。[生成](../../../raw/current/scripts/prefabs/snowball_item.lua#L46)、[注册](../../../raw/current/scripts/fx.lua#L646-L648)
- **边界：亮茄炸弹爆炸物理伤害为 0**：`explosivedamage = 0`，生命值结算全来自 200 位面伤害，引燃也关闭；建筑工作量与 `explosion` 事件仍照常，其他玩家在无 `forcepvp`、无击退数据时不进入结算。[配置](../../../raw/current/scripts/prefabs/bomb_lunarplant.lua#L23-L34)、[分流](../../../raw/current/scripts/components/explosive.lua#L58-L71)
- **边界：睡袋官方名是"睡袋"**：与 bedroll 族区分，不是睡眠家具。[名称](../../../raw/current/scripts/languages/chinese_s.po#L349450-L349452)

## 验证记录

- 源码审阅：2026-11-07，Steam buildid 26335394。实读四主文件与 sleepcloud；核验投掷/命中链、equippable/inventory、snowballmelting、配方与蓝图、tuning、TryLuckRoll、.po；南瓜/雪球生成点、worm_boss 吞食、SMOTHER 动作。第四轮复检：睡袋无 explosive、雪球冷度扩散、`toadstool_dark` 专属蓝图、击退半径缓存、玩家伤害/击退分流。皮肤核验：三处 skin 文件无四注册。未覆盖：SG throw 帧级、熔炉准星表现、FX 动画内容。
- 游戏运行：未运行。各数值（爆炸／位面伤害、睡眠 20s、2 日腐败、堆叠乘算）均为源码推导，非运行测量。
- 结构检查：validate_page.py 与 strict 114 引用 0 错 0 警。

返回 [[reference/prefabs/weapons-combat-index|武器与战斗]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 2 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-12）。
