---
title: "回旋镖（boomerang）"
created: "2026-11-06"
updated: "2026-11-08"
type: prefab
summary: "原版手持投掷武器 boomerang：武器本体即投射物，攻击脱手飞出追踪命中，命中／脱靶都飞回（仅命中时掷出者为鬼魂才落地）；CATCH 提示需弹体 catchable 与操作者 cancatch，接取后仅在背包开启时自动装备（HANDS 空闲）或入包；伤害 27.2、耐久 10、耗尽不返回原地移除。"
aliases: ["boomerang", "回旋镖", "Boomerang", "飞去来器", "投掷武器"]
tags: [prefab, item, equipment]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/boomerang.lua", "components/projectile.lua", "components/weapon.lua", "components/combat.lua", "components/catcher.lua", "components/finiteuses.lua", "components/equippable.lua", "components/inspectable.lua", "components/inventoryitem.lua", "components/inventory.lua", "components/wisecracker.lua", "components/builder.lua", "widgets/widgetutil.lua", "prefabs/player_common.lua", "prefabs/charlie_boss.lua", "prefabs/voidcloth_boomerang.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "componentactions.lua", "actions.lua", "tuning.lua", "constants.lua", "recipes.lua", "recipes_filter.lua", "prefabs/scienceprototyper.lua", "standardcomponents.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "map/boons.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 回旋镖（boomerang）

## 快速判断

- 职责与适用：`prefabs/boomerang.lua` 顶层 `fn` 直接注册的单注册可拾取手持投掷武器（物品＋装备）：weapon 伤害＋finiteuses 耐久＋equippable HANDS 槽＋projectile 飞行，特色是**武器本体即投射物**——攻击时脱手飞出，命中或脱靶后都飞回掷出者；命中时若目标是掷出者本人或掷出者带 `playerghost` 则落地（脱靶走返航分支），可被接回；同文件无 quagmire 分支。[工厂与注册](../../../raw/current/scripts/prefabs/boomerang.lua#L89-L155)
- 优先使用：要"飞出＋返回＋可接回"的投掷武器按[关联机制导航](#关联机制导航)复刻本体即投射物的接线（不需 `weapon:SetProjectile` 代理）；只改数值查下方数值表；调接回容错距离读 [[reference/components/catcher#从动作提示到真正接取]]。
- 重要副作用：伤害与磨损都在飞行命中结算（[Projectile:Hit](../../../raw/current/scripts/components/projectile.lua#L236-L266)），掷出动作本身不结算；去程脱靶也返航（[OnMiss](../../../raw/current/scripts/prefabs/boomerang.lua#L81-L87)），命中时掷出者已是鬼魂则落地不返航（[OnHit](../../../raw/current/scripts/prefabs/boomerang.lua#L65-L67)）；未接住的回程弹命中掷出者——自伤并再扣 1 耐久（[OnHit](../../../raw/current/scripts/prefabs/boomerang.lua#L65-L79)）；耐久耗尽不返回，原地播 `used` 后整个实体移除（[OnFinished](../../../raw/current/scripts/prefabs/boomerang.lua#L7-L10)）。
- 修改边界：标签与 floater 挂公共端、组件挂主模拟（[分界](../../../raw/current/scripts/prefabs/boomerang.lua#L115-L121)）；投掷／接回动作状态在玩家 SG（本页只给锚点）；不覆盖同族 voidcloth_boomerang 的代理投射接线（见[关联机制导航](#关联机制导航)）与皮肤贴图内容。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `boomerang` | 无包装工厂：`fn()` 直接构造，无 postinit | 物品＋装备；皮肤经 `boomerang_init_fn`。[注册](../../../raw/current/scripts/prefabs/boomerang.lua#L155)、[皮肤 init](../../../raw/current/scripts/prefabskin.lua#L196-L197) |

共同初始化（[boomerang.lua#L89-L115](../../../raw/current/scripts/prefabs/boomerang.lua#L89-L115)）：实体组件＋`MakeInventoryPhysics`→`RemovePhysicsColliders`（全程无物理碰撞，飞行只靠 projectile 逐帧距离判定）→bank／build `boomerang`＋`idle`＋`SetRayTestOnBB(true)`→公共端标签 `thrown`／`weapon`／`projectile`（[pristine 镜像注释](../../../raw/current/scripts/prefabs/boomerang.lua#L104-L110)）→`MakeInventoryFloatable("small", 0.18, {0.8, 0.9, 0.8}, true, -6, {sym_build = "swap_boomerang"})`→`SetPristine`（非主模拟返回）→主模拟依次 weapon→finiteuses→inspectable→projectile→inventoryitem→equippable→`MakeHauntableLaunch`。装备槽用组件默认 HANDS。[equippable 默认](../../../raw/current/scripts/components/equippable.lua#L29)

名称键经简中 .po `msgctxt` 核验：`STRINGS.NAMES.BOOMERANG`＝回旋镖（Boomerang）。[.po](../../../raw/current/scripts/languages/chinese_s.po#L339949-L339952)

数值（tuning 键＋实际值；`wilson_attack`＝34×倍率 1，[tuning.lua#L31-L38](../../../raw/current/scripts/tuning.lua#L31-L38)）：

| 项 | 键／来源 | 实际值 |
|---|---|---|
| 武器伤害 | `TUNING.BOOMERANG_DAMAGE`＝`wilson_attack*.8` | 27.2 生命值／击（源码推导，[tuning.lua#L360](../../../raw/current/scripts/tuning.lua#L360)） |
| 耐久上限 | `TUNING.BOOMERANG_USES` | 10；每次命中弹体磨损 1（`attackwear` 未设，[OnAttack](../../../raw/current/scripts/components/weapon.lua#L114-L119)） |
| 攻击射程 | `TUNING.BOOMERANG_DISTANCE`（＋2） | weapon `attackrange`=12、`hitrange`=14（[tuning.lua#L173](../../../raw/current/scripts/tuning.lua#L173)、[设置](../../../raw/current/scripts/prefabs/boomerang.lua#L123)） |
| 飞行速度 | 硬编码 `SetSpeed(10)`，非 tuning 键 | 10（[boomerang.lua#L135](../../../raw/current/scripts/prefabs/boomerang.lua#L135)） |
| 命中判定 | projectile `hitdist` 默认 1，与目标物理半径相加 | 判定半径＝`GetPhysicsRadius(0)`＋1；`homing`=true（[默认值](../../../raw/current/scripts/components/projectile.lua#L20-L21)、[判定](../../../raw/current/scripts/components/projectile.lua#L396-L416)） |

获取途径（配方与恩惠布局两条；`SpawnPrefab("boomerang")` 字面检索只覆盖直接生成，不覆盖布局抽取）：

| 途径 | 证据 |
|---|---|
| 配方：boards 1＋silk 1＋charcoal 1，`TECH.SCIENCE_TWO`（={SCIENCE=2}）；Science Machine 树 SCIENCE=1、Alchemy Engine 树 SCIENCE=2，配方经 `CanPrototypeRecipe` 比对原型机树→**需炼金引擎档** | [配方](../../../raw/current/scripts/recipes.lua#L489)、[TECH](../../../raw/current/scripts/constants.lua#L1246)、[两原型机树](../../../raw/current/scripts/tuning.lua#L1262-L1270)、[机器注册](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L299-L300)、[比对](../../../raw/current/scripts/widgets/widgetutil.lua#L16)、[门禁](../../../raw/current/scripts/components/builder.lua#L674)、[过滤器](../../../raw/current/scripts/recipes_filter.lua#L1019) |
| 世界生成：WeaponBoon 恩惠布局 `item_area` 从 4 件武器（含 boomerang）PickSome 1 件；`worldgen_main` 按森林倍率随机抽 3–8 个 Sandbox 布局、登记到对应任务区，故可随世界生成落地（落点与频率未核验） | [boons.lua#L150-L155](../../../raw/current/scripts/map/boons.lua#L150-L155)、[抽取](../../../raw/current/scripts/worldgen_main.lua#L300-L317)、[布点](../../../raw/current/scripts/worldgen_main.lua#L368-L373) |
| 特殊消费端：Charlie boss 监听弹体 `weapononattack`，对飞行的 boomerang `Deflect`（`SetKeepOnDeflect(true)` 保证实体不被移除）并朝攻击者反弹 | [charlie_boss.lua#L260-L285](../../../raw/current/scripts/prefabs/charlie_boss.lua#L260-L285) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：weapon | 仅 `SetDamage`＋`SetRange(12, 14)`；**无 `SetProjectile`**，本体即投射物 | 主模拟；命中经 `OnAttack` 推 `weapononattack` 并扣耐久 | [[reference/components/weapon#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/boomerang.lua#L121-L123)、[OnAttack](../../../raw/current/scripts/components/weapon.lua#L105-L121) |
| 组件：projectile | `SetSpeed(10)`＋`SetCanCatch(true)`＋`SetKeepOnDeflect(true)`＋四个回调（OnThrown／OnHit／OnMiss／OnCaught）；homing 默认 true | 主模拟；`Throw` 后 `StartUpdatingComponent` 逐帧追踪 | [[reference/components/projectile#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/boomerang.lua#L134-L141) |
| 组件：finiteuses | 上限＝当前＝10；耗尽 `OnFinished` 播 `used` 后移除；用途经 OnSave／OnLoad 持久化 | 主模拟 | [[reference/components/finiteuses#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/boomerang.lua#L126-L130) |
| 组件：equippable／inspectable／inventoryitem | 默认 HANDS 槽；OnEquip 覆盖 `swap_object`＋`ARM_carry`，OnUnequip 复原；`SetOnDroppedFn(OnDropped)` 落地播 `idle` 并推 `on_landed` | 主模拟；皮肤分支推 `equipskinneditem`／`unequipskinneditem`（[OnEquip](../../../raw/current/scripts/prefabs/boomerang.lua#L12-L22)） | [[reference/components/equippable#关键接口]]、[[reference/components/inspectable#关键接口]]、[[reference/components/inventoryitem#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/boomerang.lua#L132-L148) |
| 组件：catcher（玩家侧） | `Throw` 把目标登记进 catcher：去程目标若有 catcher 即 `StartWatching`，回程再把掷出者登记进来；被登记者都可接取，CATCH 提示需弹体 `catchable`＋其 `cancatch`，真正接取要求其处于 `readytocatch` 且弹体进入 `catchdistance`(2) 内 | 玩家实体挂载（[player_common.lua#L2782](../../../raw/current/scripts/prefabs/player_common.lua#L2782)）；`actiondistance`=12 控制动作提示距离；接取后仅 `inventory.isopen` 时处理，HANDS 空闲 `Equip` 否则 `GiveItem`（[OnCaught](../../../raw/current/scripts/prefabs/boomerang.lua#L47-L56)） | [[reference/components/catcher#关键接口]]；[动作生成](../../../raw/current/scripts/componentactions.lua#L687-L690)、[StartWatching](../../../raw/current/scripts/components/projectile.lua#L189-L191)、[OnUpdate 接取](../../../raw/current/scripts/components/catcher.lua#L70-L93) |
| SG／Brain：SGwilson（Wiki 待建） | 武器带 `thrown` 标签→攻击进 `throw` 状态（throw_pre＋throw，第 7 帧 `PerformBufferedAction`）；`CATCH`→`catch_pre`（`readytocatch`，超时 3 秒）→`catch` 状态播接取动画＋音效 | 玩家宿主；客户端预测镜像同判 | [动画选择](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1411)、[throw 状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10664-L10695)、[catch_pre](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10734-L10755)、[catch](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10763-L10781)、[客户端](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L684) |
| API／数据：`catchable`／`cancatch` 标签＋CATCH 动作 | projectile 有 target 且 `cancatch` 时加 `catchable` 标签；componentactions 据此对玩家给 CATCH 动作；`fn` 为空壳，真接取在 catcher:OnUpdate | 公共端标签＋主模拟动作；ActionHandler→`catch_pre` | [oncatchable](../../../raw/current/scripts/components/projectile.lua#L3-L9)、[动作生成](../../../raw/current/scripts/componentactions.lua#L687-L690)、[CATCH 定义](../../../raw/current/scripts/actions.lua#L414)、[空 fn](../../../raw/current/scripts/actions.lua#L1880-L1882)、[Handler](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1444) |
| API／数据：`projectile` 标签（武器侧） | `DoAttack` 据此走 DropItem＋Throw 投掷分支；玩家远程攻击豁免虚空断档检查 | 主模拟 | [投掷分支](../../../raw/current/scripts/components/combat.lua#L1168-L1175)、[远程豁免](../../../raw/current/scripts/components/combat.lua#L813-L824) |
| API／数据：`ANNOUNCE_BOOMERANG` 播报 | 玩家被 `attacked` 且 `data.weapon.prefab == "boomerang"`（回程弹自伤）→talker 播报 | 客户端表现；wisecracker 组件挂玩家 | [wisecracker.lua#L80-L84](../../../raw/current/scripts/components/wisecracker.lua#L80-L84)、[.po 键](../../../raw/current/scripts/languages/chinese_s.po#L5969-L5972) |
| 关联 Prefab：`impact` | OnHit 命中有效目标时生成，`AddFollower` 跟随目标 `hiteffectsymbol` 做受击特效 | 主模拟；目标无 combat 则不生成 | [[reference/prefabs/impact]]；[OnHit 内](../../../raw/current/scripts/prefabs/boomerang.lua#L71-L78) |
| 关联 Prefab：`voidcloth_boomerang`（阴郁回旋镖） | 同族另一注册：装备时 `weapon:SetProjectile("voidcloth_boomerang_proj")` 发射代理投射物，而非本体脱手 | 另册注册（[recipes.lua#L870](../../../raw/current/scripts/recipes.lua#L870)）；名称 [po](../../../raw/current/scripts/languages/chinese_s.po#L351349-L351352) | [[reference/prefabs/voidcloth_boomerang]]；[接线对照](../../../raw/current/scripts/prefabs/voidcloth_boomerang.lua#L208-L209) |
| 皮肤 | bananarang（香蕉回旋镖）／bandedwood（残月回旋镖）／tiger（老虎回旋镖）共 3 种 | 统一 `boomerang_init_fn`（basic_init_fn，build `boomerang`） | [注册](../../../raw/current/scripts/prefabs/skinprefabs.lua#L2599-L2620)、[init](../../../raw/current/scripts/prefabskin.lua#L196-L197)、[名称](../../../raw/current/scripts/languages/chinese_s.po#L394634-L394647) |
| 资源：anim／swap 动画包 | boomerang.zip（bank `boomerang`）／swap_boomerang.zip（build＋持握符号） | 静态声明 | [声明](../../../raw/current/scripts/prefabs/boomerang.lua#L1-L5) |

## 典型接入

### 修改回旋镖的伤害、耐久与飞行速度

- 前提与选择：服务端 `modmain.lua`；三个数值分属 weapon／finiteuses／projectile 三个组件，`AddPrefabPostInit` 在实例组装后统一改。
- 生效与撤销：postinit 只影响新实例；已有实例改上限即时生效、当前耐久按存档恢复；停用模组恢复原值。
- 原版依据：伤害在 weapon（[挂载](../../../raw/current/scripts/prefabs/boomerang.lua#L122)）、耐久在 finiteuses、速度在 projectile（[挂载](../../../raw/current/scripts/prefabs/boomerang.lua#L135)）；未抛出时 `SetSpeed` 只改 `self.speed`，下次 `Throw` 生效，往返共用同一速度（[Throw](../../../raw/current/scripts/components/projectile.lua#L181)）。

```lua
-- modmain.lua：回旋镖伤害 40、耐久 20、飞行速度 12
AddPrefabPostInit("boomerang", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    inst.components.weapon:SetDamage(40)
    inst.components.finiteuses:SetMaxUses(20)
    inst.components.finiteuses:SetUses(20)
    inst.components.projectile:SetSpeed(12)
end)
```

- 调整与验证：伤害单位为生命值／击，耐久为耐久点／次命中（往返最多扣 2），速度为 Physics `SetMotorVel` 标量（往返共用，改大同时缩短滞空窗口）；接回窗口不变，需更大容错改 catcher 的 `SetCatchDistance`（见 [[reference/components/catcher#典型接入]]）。

## 常见陷阱与限制

- **误区：掷出瞬间结算伤害**：掷出动作经 `DoAttack` 的 projectile 分支提前 return，本体不在此刻掉血；伤害与磨损都在弹体 `Hit` 内的 `DoAttack`／`OnAttack` 结算。[投掷分支](../../../raw/current/scripts/components/combat.lua#L1168-L1175)、[Hit 结算](../../../raw/current/scripts/components/projectile.lua#L236-L266)
- **边界：未接住的回程弹自伤并重复扣耐久**：回程 `Throw(owner, owner)` 后若未被接住，`Hit(owner)` 会对掷出者结算一次 27.2 武器伤害（可被护甲减免）并再磨损 1 耐久——一次完整"掷出＋未接住返回"最多消耗 2 耐久；接住（catcher 在 Hit 前触发 `Catch`）则两者都避免。[OnHit 判 owner==target](../../../raw/current/scripts/prefabs/boomerang.lua#L65-L70)、[Hit 内 DoAttack](../../../raw/current/scripts/components/projectile.lua#L251-L258)
- **边界：耐久耗尽不返回**：`ReturnToOwner` 判 `finiteuses:GetUses() <= 0` 直接不飞回，最后一击后弹体留在目标位置播 `used`、`animover` 后 Remove——无残骸可捡。[ReturnToOwner](../../../raw/current/scripts/prefabs/boomerang.lua#L58-L63)
- **边界：命中半径是目标物理半径＋`hitdist`(1)**：每帧先判距离是否小于 `GetPhysicsRadius(0)＋hitdist`，再判本帧位移是否越过目标点（越过也算 Hit）；都不满足时，卡滞 1 秒与离屏 2 秒各自发 Miss。[判定](../../../raw/current/scripts/components/projectile.lua#L396-L416)、[卡滞](../../../raw/current/scripts/components/projectile.lua#L315-L365)、[休眠](../../../raw/current/scripts/components/projectile.lua#L268-L289)
- **边界：目标失效后的射程兜底有条件**：`CheckTarget` 只查有效／limbo／可见／非 flight·invisible，不查死亡；目标失效或丢失后，仅当持有人 `combat` 与本体 `weapon.attackrange` 都在时，才用 `owner.combat.attackrange＋weapon.attackrange` 判射程，否则直接 Miss。[失效分支](../../../raw/current/scripts/components/projectile.lua#L428-L448)、[CheckTarget](../../../raw/current/scripts/components/projectile.lua#L293-L301)
- **边界：`SetKeepOnDeflect(true)` 只保证弹反后实体不移除**：本 prefab 未设 `SetOnDeflectFn`，弹反本身无自定义行为；实际反弹由 Charlie boss 端生成新反射投射物完成。[Deflect](../../../raw/current/scripts/components/projectile.lua#L532-L539)、[charlie_boss.lua#L276](../../../raw/current/scripts/prefabs/charlie_boss.lua#L276)

## 验证记录

- 源码审阅：2026-11-06，Steam buildid 26335394。核验 prefabs/boomerang.lua 全文（156 行）；components/projectile.lua 全文（Throw／Hit／Miss／Catch／OnUpdate／CheckStuck／CheckTarget／Deflect）；weapon.lua OnAttack／LaunchProjectile 路径；finiteuses.lua 全文；catcher.lua 全文；combat.lua DoAttack 投掷分支／GetAttackRange／CanHitTarget 远程豁免／OnAttack 调用点；inventory DropItem→RemoveItem 强制卸装链；equippable 默认 HANDS；SGwilson throw／catch_pre／catch 状态与动画选择、客户端镜像；CATCH 动作定义与 componentactions 生成；wisecracker 播报；tuning 三键＋wilson_attack；配方行、科技树与两台原型机映射、配方过滤器；名称／播报／皮肤 po 键；boons 与 charlie_boss 消费端；voidcloth_boomerang 代理接线对照；treasurechest.lua 中 boomerang 核为 scrapbook 依赖数据。2026-11-07 第四轮复检：接回承诺（脱靶返航、命中鬼魂才落地、背包开启与 HANDS 空闲）、`hitdist` 判定与三条脱靶路径、WeaponBoon 经 `worldgen_main` 抽取落地。未覆盖：皮肤贴图内容、voidcloth_boomerang 自身机制（仅核对照点）。
- 游戏运行：未运行。27.2 伤害、接取半径 2、往返最多扣 2 耐久等均为源码推导，非运行测量。
- 结构检查：validate_page.py 通过（109 行／19321 字节，standard 档内）；validate_source_links.py --strict 通过（0 error 0 warning）。

返回 [[reference/prefabs/weapons-combat-index|武器与战斗]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-06）。
