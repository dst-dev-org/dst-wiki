---
title: "瘴气（miasma_cloud）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "洞穴瘴气雾块主实体：周期扫描给玩家挂瘴气源（减速＋持续掉血 debuff）、可被蜈蚣食用、遇火消退；同文件另有 4 个粒子子体与 2 个联网视觉变体，客户端用相机驱动的双半圆粒子系统渲染。"
aliases: ["miasma_cloud", "瘴气", "瘴气云", "miasma", "miasma_cloud_visual", "miasma_cloud_fx", "miasma_ember_fx", "miasma_cloud_fx_thin", "miasma_cloud_fx_border", "miasma_cloud_arenabordervisual"]
tags: [prefab, fx, server-client]
size_class: complex
dst_build: "steam-buildid:26335394"
sources: ["prefabs/miasma_cloud_fx.lua", "prefabs/miasmadebuff.lua", "components/miasmamanager.lua", "components/shadowthrallmanager.lua", "stategraphs/SGcaveventmite.lua", "prefabs/cave_vents.lua", "prefabs/willow_ember.lua", "prefabs/shadowthrall_centipede.lua", "brains/shadowthrall_centipede_brain.lua", "prefabs/charlie_boss_trial.lua", "prefabs/atrium_markers.lua", "constants.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 瘴气（miasma_cloud）

## 快速判断

- 职责与适用：洞穴瘴气场的**主模拟实体**：每 0.5 秒扫描周围，把范围内带 miasmawatcher 的实体（玩家）登记为瘴气源——进而挂减速与持续掉血 debuff（机制在 [[reference/components/miasmawatcher]]，不在本页复制）；本体带 edible，是暗影蜈蚣的食物；附近有火则转入消退并自毁。名称字符串 "Miasma" 的官方译名是"瘴气"（[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L344800-L344802)）。[周期扫描](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L437-L488)
- 优先使用：查"瘴气生成／驱散／免疫边界"选本页；世界侧网格排程（生成资格、扩散、强度）在 [[reference/components/miasmamanager]]；玩家侧减速与豁免在 [[reference/components/miasmawatcher]]。
- 重要副作用：**不是纯视觉**：范围作用半径 `MIASMA_RADIUS=3` 世界单位（由 `SQRT2×MIASMA_SPACING(1 瓦)×TILE_SCALE(4)/2` 取整，[推导](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L37-L42)、[TILE_SCALE](../../../raw/current/scripts/constants.lua#L17)）；瘴气源退出有约两个扫描周期的滞后；遇火（火/热源标签，3+1=4 单位）标记消退并安排约 2 秒自毁。[源登记](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L443-L455)、[火反应](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L459-L485)、[数值](../../../raw/current/scripts/tuning.lua#L7730-L7743)
- 修改边界：客户端另有整套相机驱动的粒子系统（本体只发 net_bool，粒子在非联网子实体上，专用服务器不装）；实体云计数经 `miasmacloudexists` 世界事件联动所有玩家的 miasmawatcher 开关。[客户端分界](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L602-L623)、[计数事件](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L649-L655)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `miasma_cloud` | fn：Transform＋Network → FX／miasma 标签 → net_bool `_diminishing`（diminishingdirty）→ 非专用机：装 AttachParticles/DetachParticles、注册相机监听（首个实体时一次性装全局监听）、进客户端追踪表 → SetPristine → 主模拟：edible（FOODTYPE.MIASMA、0 血／大饥饿／-大理智）、三张 watcher 队列表、OnEntityWake/Sleep/OnRemoveEntity、ForceKillMiasma、计数＋推 `miasmacloudexists`、TheWorld.GetMiasmaCloudCount | 主实体；persists 未关闭（入存档）。[工厂](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L591-L658)、[edible](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L631-L635) |
| `miasma_cloud_fx`／`_thin`／`_border` | fn_fx(_thin/_border)：非联网 Transform 实体，SetCanSleep(false)、persists=false，SetupParticles 装烟粒子发射器（thin 换淡红色包、border 换黑色描边包与外扩速度） | 主实体的前/后半圆粒子子体，按变体名生成（[选择](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L565)）。[fn_fx](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L662-L705)、[SetupParticles](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L309-L394) |
| `miasma_ember_fx` | fn_fx_ember：非联网 Transform 实体，SetupEmberParticles 装加色火烬粒子（bloom） | `_diminishing` 为真时挂到两个粒子子体下的火烬。[fn_fx_ember](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L730-L743)、[挂载](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L206-L217) |
| `miasma_cloud_visual` | visualfn：与主实体同构的联网壳（FX／miasma 标签＋`_diminishing`＋客户端粒子机制），`thin=true`；主模拟段只有 return，**无 edible／watchers／ForceKillMiasma**；persists 对应行被注释（入存档） | 竞技场装饰用（charlie_boss_trial）。[visualfn](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L747-L791)、[消费者](../../../raw/current/scripts/prefabs/charlie_boss_trial.lua#L185-L198) |
| `miasma_cloud_arenabordervisual` | visualborderfn：同上但 `border=true`，persists=false | 竞技场边界描边（charlie_boss_trial／atrium_markers）。[visualborderfn](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L793-L837)、[消费者](../../../raw/current/scripts/prefabs/atrium_markers.lua#L136-L143) |

七个注册名在同一 return（[L839-L845](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L839-L845)）。实体本身无可检视名称（无 inspectable，无 NAMES.MIASMA_CLOUD 条目）；标题"瘴气"取自官方名称字符串（[译名](../../../raw/current/scripts/languages/chinese_s.po#L344800-L344802)）。

## 关联机制导航

### 玩法瘴气（miasma_cloud）

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | 食物类型 FOODTYPE.MIASMA，healthvalue 0／hungervalue CALORIES_LARGE／sanityvalue -SANITY_LARGE | 主模拟；食用者是暗影蜈蚣（DIET 仅 MIASMA，[Diet](../../../raw/current/scripts/prefabs/shadowthrall_centipede.lua#L187)、[挂 eater](../../../raw/current/scripts/prefabs/shadowthrall_centipede.lua#L257-L258)） | [edible](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L631-L635) |
| 消费者：shadowthrall_centipede_brain | 蜈蚣 10 单位内找可食云，构造 `ACTIONS.EAT` 食用；每成功吃 5 次（EAT_MIASMA_MAX）长一节身体并进入进食冷却 | 主模拟；FindEntity 按 eater 可食标签过滤 | [找云](../../../raw/current/scripts/brains/shadowthrall_centipede_brain.lua#L55-L61)、[食用动作](../../../raw/current/scripts/brains/shadowthrall_centipede_brain.lua#L63-L82)、[数值](../../../raw/current/scripts/tuning.lua#L8887-L8891) |
| 玩法作用：周期源登记 | OnUpdate 每 0.5 秒（PERIOD，[定义](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L398-L399)）FindEntities(MIASMA_RADIUS=3, 排除 ghost/shadow/FX 等 [NO_TAGS](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L401))，对带 miasmawatcher 的可见实体 AddMiasmaSource；三队列表轮转让退出晚约两拍（注释：跨云过渡平滑），ClearWatcherTable 逐个 RemoveMiasmaSource | 主模拟；实体睡眠（OnEntitySleep）停扫，唤醒重启（[L515-L521](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L515-L521)） | [OnUpdate](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L437-L488)、[三队列](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L438-L454)、[清理](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L403-L410)、[[reference/components/miasmawatcher#关键接口]] |
| 玩法作用：debuff 与减速 | 玩家首次获得源时由 miasmawatcher 挂 `miasmadebuff`（每 2 秒 -2 血，miasmaimmune 装备免疫）并按 MIASMA_SPEED_MOD=0.4 挂 locomotor 外部减速 | 主模拟 debuff（客户端经 classified 读 isInMiasma）；具体在组件页 | [[reference/components/miasmawatcher]]、[debuff tick](../../../raw/current/scripts/prefabs/miasmadebuff.lua#L1-L20)、[数值](../../../raw/current/scripts/tuning.lua#L7742-L7743) |
| 火反应与消退 | OnUpdate 找 FIRE_RADIUS（MIASMA_RADIUS+1）内 fire/heatstar（排 shadow_fire/blueflame；[TestFireFn](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L412-L421) 排除非点火物的装备栏物品）：有火→`SetMiasmaDiminishingAtPoint(true)`（受管云）或 2 秒假消退自毁任务（无管理网格的云）；同时 `_diminishing:set(true)` 让客户端挂火烬。无火且非强杀则反向恢复 | 主模拟；假消退时长＝MIASMA_DIMINISH_INTERVAL_SECONDS×MIASMA_MAXSTRENGTH=2 秒 | [火反应](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L459-L485)、[ForceKillMiasma](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L423-L434)、[数值](../../../raw/current/scripts/tuning.lua#L7731-L7734) |
| API：ForceKillMiasma | 薇洛火焰投掷／爆燃命中 `miasma` 标签实体时调用（本工厂暴露），走同一消退路径并置 force_killed | 主模拟；对两个视觉变体不可用（未暴露该方法） | [调用](../../../raw/current/scripts/prefabs/willow_ember.lua#L211-L213)、[调用](../../../raw/current/scripts/prefabs/willow_ember.lua#L277-L279) |
| 计数与事件 | 主实体增减维护 `_MiasmaCloudCount`，0↔1 时推世界事件 `miasmacloudexists`；`TheWorld.GetMiasmaCloudCount` 首个实体时装上（不覆盖） | 主模拟；miasmawatcher 以此开关监听组 | [计数](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L649-L655)、[递减](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L507-L513)、[[reference/components/miasmawatcher#关键接口]] |
| 生成者：miasmamanager（受管） | CreateMiasmaAtTile 在陆地瓦片中心生成主实体并激活管理排程；气孔/阴影裂隙 portal 经其接口批量造雾 | 主模拟；云随管理网格存读 | [生成](../../../raw/current/scripts/components/miasmamanager.lua#L217-L223)、[[reference/components/miasmamanager]]、[气孔](../../../raw/current/scripts/prefabs/cave_vents.lua#L217-L231) |
| 生成者：SGcaveventmite（无管理） | 洞穴气孔螨在 2 单位内无云时直接 SpawnPrefab 一朵 | 主模拟；不在管理网格，火反应走假消退 | [CheckSpawnMiasma](../../../raw/current/scripts/stategraphs/SGcaveventmite.lua#L57-L66) |
| 生成者：shadowthrallmanager | 影裂隙喷发围绕裂隙按 8 方向、半径 4 内可站地面布云，并监听 onremove 维护计数 | 主模拟；[布云](../../../raw/current/scripts/components/shadowthrallmanager.lua#L509-L522) | [[reference/components/shadowthrallmanager]] |

### 视觉云与粒子子体

本组为纯视觉：两个联网 visual 变体没有 edible／watchers／ForceKillMiasma，不能套用玩法瘴气的驱散接口；四个粒子子体为本地实体，由客户端云表现机制挂接。[visual](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L747-L791)、[border](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L793-L837)、[粒子挂接](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L559-L587)

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 客户端粒子系统 | AttachParticles 生成前后两个半圆粒子子体（变体按 thin/border 选），按相机朝向每帧摆位；相机大更新（视锥＋`PLAYER_CAMERA_MAX_DIST` 内）挂/摘粒子，重挂时 FastForward 预热；`diminishingdirty` 挂/摘火烬子体 | 仅非专用机；粒子逐点隐藏虚空上方（IsVisualGroundAtPoint）；SpawnInstantParticles 补 10 个粒子 | [AttachParticles](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L559-L587)、[相机](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L49-L130)、[瞬时补发](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L349-L366)、[虚空隐藏](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L250-L253) |
| 生成者：竞技场视觉 | charlie_boss_trial 沿边界铺 `arenabordervisual`、光柱处铺 `visual`；atrium_markers 虚拟房间出口铺 `arenabordervisual` | 联网壳，无玩法段 | [边界](../../../raw/current/scripts/prefabs/charlie_boss_trial.lua#L161-L171)、[光柱](../../../raw/current/scripts/prefabs/charlie_boss_trial.lua#L185-L198)、[出口](../../../raw/current/scripts/prefabs/atrium_markers.lua#L136-L143) |

### 共享资源

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 资源 | fx/miasma.tex＋vfx_particle.ksh（烟）、fx/snow.tex＋vfx_particle_add.ksh（火烬），色/缩放包 `miasma_cloud_*` 首次用时注册 | 客户端；声明与使用已核验 | [常量](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L1-L14)、[烟](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L317-L329)、[烬](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L714-L728) |

## 典型接入

### 生成一朵"无管理"瘴气云并支持驱散

- 前提与选择：绕过 miasmamanager 直接 SpawnPrefab 即可（SGcaveventmite 原版先例）：玩法段（源登记、debuff、火反应）完整可用，必须确认落点网格本身无受管瘴气；实际分支按当前位置 GetMiasmaAtPoint 查询，有格则交给管理器消退，无格才走 2 秒假消退。
- 生效与撤销：实体唤醒时启动扫描，首次任务有 0~0.5 秒随机初始延迟；移除走 `ForceKillMiasma()`（约 2 秒火烬渐散后 Remove）或直接 Remove（OnRemoveEntity 自动清源并递减计数）。
- 原版依据：螨虫生成与蜈蚣食用都是无管理交互的实例；受管创建请用 [[reference/components/miasmamanager]] 的 `CreateMiasmaAtPoint`。[螨虫先例](../../../raw/current/scripts/stategraphs/SGcaveventmite.lua#L57-L66)、[强杀](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L423-L434)

```lua
-- 主模拟：在洞穴放一朵不受网格管理的瘴气云
local cloud = SpawnPrefab("miasma_cloud")
cloud.Transform:SetPosition(x, 0, z)
-- 驱散：cloud:ForceKillMiasma()（约 2 秒后自毁，客户端同步火烬）
```

- 调整与验证：作用半径 3、扫描周期 0.5、退出滞后两拍均写死在工厂（半径推导见 [L37-L42](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L37-L42)）；改 TUNING.MIASMA_* 只影响管理侧与假消退时长（[数值](../../../raw/current/scripts/tuning.lua#L7730-L7743)）。

## 常见陷阱与限制

- **误区：名字带 cloud/fx 就是纯视觉**：主实体是玩法实体——玩家进 3 单位内即开始每 2 秒 -2 血并被减速到 0.4 倍，本体还是蜈蚣的食物；四个粒子子体和两个 visual 变体是纯视觉。[debuff 链](../../../raw/current/scripts/prefabs/miasmadebuff.lua#L1-L20)、[食用](../../../raw/current/scripts/brains/shadowthrall_centipede_brain.lua#L63-L75)
- **误区：debuff 挂在云上**：miasmadebuff 挂在玩家实体（经 miasmawatcher AddDebuff），云只负责登记/注销源；直接清玩家 debuff 不清源会被下一拍状态覆盖。交互协议见 [[reference/components/miasmawatcher#常见陷阱与限制]]。[源登记](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L443-L455)
- **边界：受管云与无管理云消退路径不同**：受管云（miasmamanager 造）遇火走网格强度衰减，强度归零才由管理器按击杀半径移除；无管理云走本地 2 秒假消退自毁。世界没有 miasmamanager（如森林）时全部走假消退，功能仍完整。[火反应](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L468-L485)、[管理器移除](../../../raw/current/scripts/components/miasmamanager.lua#L75-L91)
- **边界：退出瘴气有两拍滞后**：三队列表轮转让源注销晚两个扫描周期（约 1 秒），玩家在相邻云间走动时 debuff 不闪断；这是设计而非 bug（源码注释写明）。[注释与队列](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L398-L410)
- **边界：入存档范围不一致**：主实体与 `miasma_cloud_visual` 默认 persists（入存档），`arenabordervisual` 与四个粒子子体不入档；云实体与管理网格各自存读，删档侧数据时要两边都清（见 [[reference/components/miasmamanager#常见陷阱与限制]]）。[visual 注释](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L788)、[border 关闭](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L834)
- **边界：睡眠即停扫**：实体睡眠（离所有玩家远）时 OnUpdate 停止，停扫时立即清空三张 watcher 表并注销已有源，睡眠期间不再登记源；把玩家逻辑寄生于云的扫描要考虑睡眠窗口（[L515-L521](../../../raw/current/scripts/prefabs/miasma_cloud_fx.lua#L515-L521)）。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；845 行工厂全文（七注册、扫描三队列、火反应与假消退、ForceKill、计数事件、客户端相机粒子系统、edible）、miasmadebuff 全文、shadowthrall_centipede 的 DIET 与 brain 食用动作、miasmamanager 生成与移除路径、SGcaveventmite／shadowthrallmanager／cave_vents／willow_ember／charlie_boss_trial／atrium_markers 消费点、TUNING.MIASMA_* 与 TILE_SCALE、译名检索已核验。未覆盖：粒子视觉实测、蜈蚣 EAT 动作对云实体的通用移除细节（通用动作路径）、竞技场变体的实际显示、晚加入表现。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-world-index|天气、地形与生态痕迹]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
