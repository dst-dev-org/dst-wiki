---
title: "月熠（moonstorm_spark）"
created: "2026-11-09"
updated: "2026-11-10"
type: prefab
summary: "月风暴期间由 moonstormmanager 投放的飞行电火花：SGspore／sporebrain 的地面实体、NET 捕捉拾取、地面与手持两套腐坏计时、周期电击与 moonsparkchargeable 充能，以及月之装置与星象护目镜两条消费链。"
aliases: ["moonstorm_spark", "月熠", "Moongleam", "月风暴火花"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/moonstorm_spark.lua", "components/moonstormmanager.lua", "components/moonsparkchargeable.lua", "stategraphs/SGspore.lua", "brains/sporebrain.lua", "prefabs/hats.lua", "prefabs/monkeyisland_portal.lua", "prefabs/moonstorm_static.lua", "prefabs/bugnet.lua", "prefabs/wortox_nabbag.lua", "recipes.lua", "components/electricattacks.lua", "components/perishable.lua", "components/stackable.lua", "components/inventoryitem.lua", "components/playercontroller.lua", "components/combat.lua", "stategraphs/commonstates.lua", "actions.lua", "constants.lua", "standardcomponents.lua", "fx.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 月熠（moonstorm_spark）

## 快速判断

- 职责与适用：月风暴里会放电的飞行火花，既是可捕捉物品（inventoryitem＋workable），也是会自己找玩家／怪物跟随的地面实体（SGspore＋sporebrain）；原版用途是月之装置建材与星象护目镜的耐久补充。[注册 L154-L245](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L154-L245)、[SG 与 brain L235-L236](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L235-L236)
- 优先使用：要改刷新节奏读 [[reference/components/moonstormmanager|moonstormmanager]] 的 `DoTestForSparks`；要做"被火花回充"的装备读 [[reference/components/moonsparkchargeable]]；月风暴范围与节点在 [[reference/components/moonstorms]]。
- 重要副作用：落地即周期电击半径 4 内目标——伤害结算后隔 5–15 秒再排下一轮的前置 FX，结算前还有 5 帧前摇；手持时腐坏速率降到 0.0125（等效 60 天）；耗尽走 death 态＋3 秒兜底移除且不存档。
- 修改边界：本页只覆盖月熠自身、生成入口与直接消费者；瓦斯塔夫剧情与风暴节点不在本页，月之装置只写到"消耗几只"（成品本体 Wiki 待建，支柱／破碎等外围见 [[reference/prefabs/moon_device_pillar]]）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `moonstorm_spark` | `fn` 单工厂、无变体：网络段（Transform／AnimState／SoundEmitter／DynamicShadow／Light／Network；`MakeFlyingCharacterPhysics(1,.5)`；bank 与 build 均 `charged_particle`、播 `idle_flight_loop`；灯色 (111,111,227)/255、强度 0.75、衰减 0.5、半径 1.5；加 `show_spoilage`、`moonstorm_spark` 标签）→ `SetPristine` → 客户端返回 → master 段（inspectable／knownlocations／tradable／locomotor／inventoryitem／workable／perishable／stackable／electricattacks／hauntable；`SetStateGraph("SGspore")`＋`SetBrain(sporebrain)`；排第一轮 sparktask） | 无变体、无形态切换。`inventoryitem.canbepickedup = false`，拾取只能走 `workable` 的 NET；五个 `scrapbook_*` 赋值只影响图鉴展示。[构造 L154-L243](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L154-L243)、[图鉴字段 L184-L188](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L184-L188)、[拾取件 L208-L215](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L208-L215) |

- 标签两级：`show_spoilage`（新鲜度图标）与 `moonstorm_spark`（管理器检索条件，猴子岛传送门也用它判断贡品）。[标签 L181-L182](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L181-L182)、[传送门判断](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L288-L295)
- 名称键 `STRINGS.NAMES.MOONSTORM_SPARK`＝月熠（英文 Moongleam）。[po L345715-L345717](../../../raw/current/scripts/languages/chinese_s.po#L345715-L345717)
- 出厂即开灯、拾取关灯、落地与读档再开灯；休眠时取消 sparktask，唤醒后重排。[开灯 L171-L177](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L171-L177)、[拾取与落地 L72-L110](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L72-L110)、[休眠与唤醒 L131-L146](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L131-L146)、[读档 L112-L116](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L112-L116)

## 生成：DoTestForSparks

- 触发器：`StartMoonstorm` 成功后挂 `DoPeriodicTask(30, DoTestForSparks)`，风暴停止时与另两个剧情任务一起取消。[挂载 L383-L385](../../../raw/current/scripts/components/moonstormmanager.lua#L383-L385)、[取消 L396-L399](../../../raw/current/scripts/components/moonstormmanager.lua#L396-L399)
- 条件链（对每个 `_activeplayers` 玩家各算一次）：在风暴内 → 现查半径 30 内带 `moonstorm_spark` 且非 `INLIMBO` 的实体 → 数量小于 `SPARKLIMIT`＝3 才在距玩家 5–25（`5 + math.random()*20`）的可行走点生成一只。这里的 3 不是全场上限，而是**这一次检查对该玩家放行的阈值**：多名玩家范围重叠时，后检查的人会把前面刚刷出的月熠数进自己的重数；实体移动后同一处的计数也会变。[主体 L849-L866](../../../raw/current/scripts/components/moonstormmanager.lua#L849-L866)、[阈值 L16](../../../raw/current/scripts/components/moonstormmanager.lua#L16)、[标签表 L846-L847](../../../raw/current/scripts/components/moonstormmanager.lua#L846-L847)
- 落点还要过 `customcheckfn`：4 半径内无 `birdblocker`，且该点仍在风暴内。[选点校验 L89-L94](../../../raw/current/scripts/components/moonstormmanager.lua#L89-L94)
- 拾取会先补检但不腾名额：`onworked` 在 `GiveItem` **之前**先调一次 `DoTestForSparks()`，此时被捡的这只仍在场景里、仍计入半径 30 的 3 只上限（入库后的 `RemoveFromScene` 才把它移出场景），所以这次拾取不会让计数降到 3 以下；只有原本就不足 3 只时才会补。[L118-L129](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L118-L129)、[计数与生成 L855-L861](../../../raw/current/scripts/components/moonstormmanager.lua#L855-L861)、[入库后才出场景 L348-L360](../../../raw/current/scripts/components/inventoryitem.lua#L348-L360)

## 地面形态与两套计时

- `SGspore`：初始态 `takeoff`（出厂播 `cough_out`，第 14 帧开灯开影并播 `mushtree_tall_spore_land`）；`death` 态播 `land`、第 45 帧关灯关影、`animover` 时移除。[初始态](../../../raw/current/scripts/stategraphs/SGspore.lua#L137)、[takeoff](../../../raw/current/scripts/stategraphs/SGspore.lua#L109-L132)、[death](../../../raw/current/scripts/stategraphs/SGspore.lua#L38-L60)
- `sporebrain`：优先跟随 2–10 距离内的 player／character／monster（目标丢失或离得超过 20 再重找），否则以 `knownlocations` 的 home 为锚在 20 半径内漫游。[sporebrain L14-L36](../../../raw/current/scripts/brains/sporebrain.lua#L14-L36)
- 拾取入口是 `workable`：`SetWorkAction(ACTIONS.NET)`、`SetWorkLeft(1)`、`SetOnFinishCallback(onworked)`；"捕捉"是工具动作——玩家侧走 `GetPickupAction` 的 `TOOLACTIONS` 循环：目标带 `NET_workable`、手持物带 `NET_tool` 时才返回 `ACTIONS.NET`；捕虫网与铥矿捕虫网、沃拓克斯强抢袋都用 `tool:SetAction(ACTIONS.NET)` 提供该标签，空手没有入口。[workable L212-L215](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L212-L215)、[拾取动作选择 L2005-L2019](../../../raw/current/scripts/components/playercontroller.lua#L2005-L2019)、[工具动作表](../../../raw/current/scripts/constants.lua#L2187-L2197)、[捕虫网 L61-L62](../../../raw/current/scripts/prefabs/bugnet.lua#L61-L62)、[铥矿捕虫网 L144-L145](../../../raw/current/scripts/prefabs/bugnet.lua#L144-L145)、[强抢袋 L209-L210](../../../raw/current/scripts/prefabs/wortox_nabbag.lua#L209-L210)、[NET 结算](../../../raw/current/scripts/actions.lua#L1859-L1877)
- 落地恢复：`SetLocalMultiplier(1)`、重播循环音、`SetWorkLeft(1)`、整叠逐件散落到同一点、开灯、重排 dospark。[L83-L110](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L83-L110)
- 计时定义：`SPARK_PERISH_TIME = total_day_time × 0.75`，而 `total_day_time = seg_time × 16`＝480 秒 → **360 秒**。[定义](../../../raw/current/scripts/tuning.lua#L6250)、[基准 L19-L20](../../../raw/current/scripts/tuning.lua#L19-L20)、[赋值 L217-L219](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L217-L219)
- 地面结算：`localPerishMultiplyer` 为 1，但无主时 `perishable:Update` 再乘 `PERISH_GROUND_MULT`＝1.5 → 实际约 **240 秒**；雨淋（×1.3，物品默认带 moisture）、严寒／过热等条件命中时再变。[地面分支](../../../raw/current/scripts/components/perishable.lua#L97-L105)、[tuning L2083](../../../raw/current/scripts/tuning.lua#L2083)、[公式 L127-L133](../../../raw/current/scripts/components/perishable.lua#L127-L133)、[默认开 moisture](../../../raw/current/scripts/components/inventoryitem.lua#L87-L89)
- 手持结算：`SetLocalMultiplier(SEG_TIME × 3 / PERISH_SLOW)`＝90/7200＝**0.0125** → 360/0.0125＝28800 秒＝**60 天**。[L72-L81](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L72-L81)、[PERISH_SLOW L2049](../../../raw/current/scripts/tuning.lua#L2049)
- 耗尽：`perishable:SetOnPerishFn(depleted)`。`depleted` 关 workable、推 `death`、`persists = false`，并 3 秒后兜底 `Remove`（death 动画自己会移除，3 秒是离屏兜底）；已在 Limbo 则立即移除。[depleted L60-L70](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L60-L70)、[Perish 调用点](../../../raw/current/scripts/components/perishable.lua#L277-L289)
- 堆叠：`stackable` 未设 `SetStackSize`，取构造默认 `TUNING.STACK_SIZE_MEDITEM`＝**20**。[组件 L222](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L222)、[默认值](../../../raw/current/scripts/components/stackable.lua#L23-L29)、[tuning L82](../../../raw/current/scripts/tuning.lua#L82)
- 新鲜度分档：`PERISH_FRESH`＝0.5、`PERISH_STALE`＝0.2，越过阈值切 fresh／stale／spoiled 标签；`show_spoilage` 负责图标，`displayadjectivefn` 借用 HUD 的 `STALE_POWER`（"衰减中"）／`SPOILED_POWER`（"已衰减"）作前缀。[分档](../../../raw/current/scripts/components/perishable.lua#L1-L27)、[tuning L1647-L1648](../../../raw/current/scripts/tuning.lua#L1647-L1648)、[前缀 L148-L152](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L148-L152)、[po L431629-L431648](../../../raw/current/scripts/languages/chinese_s.po#L431629-L431647)

## 电击与充能

- 节拍（两段时间别混）：`5 + math.random()*10` 秒是**上一次伤害结算到下一次前置 FX**的等待，构造时与每轮结束时都会重排；`dospark` 生成 `moonstorm_spark_shock_fx` 后还要等 5 帧（5/30 秒）前摇才结算；结算时另外排一个 0.5 秒任务把半径收回 1.5，这个任务与下一轮等待并行，不阻塞排程。[dospark L52-L58](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L52-L58)、[dospark1 L21-L50](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L21-L50)、[首轮 L238](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L238)
- 伤害判定：半径 4、must `_combat`，排除 playerghost／INLIMBO／flight／invisible／notarget／noattack／`moonstorm_static`／wall／structure；再排除绝缘背包、已死与不可攻击目标。[标签表 L13-L14](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L13-L14)、[筛选与结算 L25-L40](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L25-L40)
- 伤害数值：基础 `MOONSTORM_SPARK_DAMAGE = 10/1.5`≈6.67，再乘电击倍率——电免疫目标 ×1（≈6.67），否则 ×`ELECTRIC_DAMAGE_MULT`(1.5)＋`ELECTRIC_WET_DAMAGE_MULT`(1)×潮湿度（干燥≈10，全湿≈16.67）；命中后对带 `hauntable.panicable` 的目标额外 `Panic(2)`。[tuning L6237](../../../raw/current/scripts/tuning.lua#L6237)、[tuning L659-L660](../../../raw/current/scripts/tuning.lua#L659-L660)、[结算 L31-L38](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L31-L38)
- 电击来源：伤害是 `dospark1` 直接调 `combat:GetAttacked(inst, …, "electric")` 结算的，不走 `DoAttack` 的武器 stimuli 分支；构造时挂的 `electricattacks`（`AddSource(inst)`）供 `commonstates` 判定“攻击者带 electricattacks → 允许电击表现”。[结算 L31-L38](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L31-L38)、[组件 L224-L225](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L224-L225)、[commonstates L195-L213](../../../raw/current/scripts/stategraphs/commonstates.lua#L195-L213)
- 充能：同一轮对半径 4 内所有带 `moonsparkchargeable` 标签的实体调 `DoSpark`，每次把宿主 `fueled` 百分比 +0.1（上限 1）。[检索与调用 L42-L46](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L42-L46)、[DoSpark](../../../raw/current/scripts/components/moonsparkchargeable.lua#L18-L23)、[默认 0.1](../../../raw/current/scripts/tuning.lua#L9243)
- 原版唯一充能宿主是星象护目镜 `moonstorm_goggleshat`：`fueled` 初始 `MOONSTORM_GOGGLES_PERISHTIME`＝480 秒；每次 `DoSpark` 只提高 **10 个百分点**，10 次相当于一条满值回充量，**只有期间没有任何消耗时才能直接累计**；护目镜 `fueled` 耗尽会 `inst.Remove()`，所以不能等它到零再回充。[护目镜 L2296-L2303](../../../raw/current/scripts/prefabs/hats.lua#L2296-L2303)、[耗尽移除 L2301](../../../raw/current/scripts/prefabs/hats.lua#L2301)、[tuning L6239](../../../raw/current/scripts/tuning.lua#L6239)

## 消费端

| 用途 | 用量与条件 | 证据 |
|---|---|---|
| 月之装置一阶段 | `wagpunk_bits` 4＋月熠 5＋`transistor` 2，`TECH.LOST`，带 placer、`no_deconstruction` | [recipes L593](../../../raw/current/scripts/recipes.lua#L593) |
| 月之装置建设计划 | 同名 `moon_device_construction1` 的建设计划另要 `wagpunk_bits` 4＋月熠 10＋注能月亮碎片 10 | [recipes L1530](../../../raw/current/scripts/recipes.lua#L1530) |
| 星象护目镜续航 | 每次 `DoSpark` +10 个百分点；10 次＝一条满值（仅期间无消耗才可累计），耗尽即移除 | [hats L2298-L2303](../../../raw/current/scripts/prefabs/hats.lua#L2298-L2303) |
| 猴子岛传送门 | 只接受带 `moonstorm_spark` 标签的贡品，用来启动传送门事件 | [monkeyisland_portal L288-L295](../../../raw/current/scripts/prefabs/monkeyisland_portal.lua#L288-L295) |

装置的等级、施工完成与破碎时序见 [[reference/prefabs/moon_device_pillar]]；成品 `moon_device` 本体页 Wiki 待建。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`inventoryitem` | 持有与入包回调；`canbepickedup = false` 关掉直接拾取 | 服务端；`SetOnPutInInventoryFn` 在这里被"赋值"而非调用（见陷阱） | [[reference/components/inventoryitem]]；[L208-L210](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L208-L210) |
| 组件：`workable` | NET 动作捕捉，`SetWorkLeft(1)`，落地重置 | 服务端；玩家侧需手持 `NET_tool` 工具 | [[reference/components/workable]]；[L212-L215](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L212-L215) |
| 组件：`perishable` | 360 秒定义值；落地与手持两套 `SetLocalMultiplier` | 服务端；`SetOnPerishFn(depleted)` | [[reference/components/perishable]]；[L217-L220](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L217-L220) |
| 组件：`stackable` | 默认 20，落地拆散 | 服务端；未显式设 maxsize | [[reference/components/stackable]]；[L222](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L222) |
| 组件：`electricattacks` | 让直接 `GetAttacked(…, "electric")` 的那次伤害具备“电击”判定 | 服务端；`commonstates` 读攻击者组件决定电击表现 | [[reference/components/electricattacks]]；[L224-L225](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L224-L225)、[判定](../../../raw/current/scripts/stategraphs/commonstates.lua#L195-L213) |
| 组件：`moonsparkchargeable`（在目标上） | 被本 prefab 半径 4 检索并 `DoSpark` | 宿主需有 `fueled` | [[reference/components/moonsparkchargeable]]；[L42-L46](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L42-L46) |
| SG／Brain：`SGspore`／`sporebrain` | 飞行、跟随、死亡动画 | 服务端；初始态 `takeoff` | [SG](../../../raw/current/scripts/stategraphs/SGspore.lua#L137)、[brain](../../../raw/current/scripts/brains/sporebrain.lua#L24-L36) |
| API／数据：`moonstormmanager:DoTestForSparks` | 唯一生成入口，30 秒周期 | 主端；风暴存在且玩家在风暴内 | [[reference/components/moonstormmanager]]；[L849-L866](../../../raw/current/scripts/components/moonstormmanager.lua#L849-L866) |
| 关联 Prefab：`moonstorm_spark_shock_fx` | 每轮电击前生成；bank/build `shock_fx`、anim `weremoose_shock`、音效 `spark_attack`、八方向 | 通用 FX 表注册 | [fx L2436-L2443](../../../raw/current/scripts/fx.lua#L2436-L2443) |
| 关联 Prefab：`moonstorm_static`／`moonstorm_lightning` | 风暴侧兄弟对象：前者才是 `MOONSTORM_SPARK_HEALTH` 的消费者，后者与火花同批随机落点 | 两者 Wiki 待建 | [静态装置 L100](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L100)、[闪电](../../../raw/current/scripts/components/moonstormmanager.lua#L868-L889) |
| 资源：`anim/charged_particle.zip` | bank 与 build 同名，idle 动画 `idle_flight_loop` | 资源声明在 assets | [L1-L4](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L1-L4)、[L167-L169](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L167-L169) |

## 常见陷阱与限制

- **误区：地面就是 360 秒**：360 秒只是 `SPARK_PERISH_TIME` 定义值；落地时 `perishable` 还会乘 `PERISH_GROUND_MULT`＝1.5，实际约 240 秒，潮湿与温度再叠加修正。[定义](../../../raw/current/scripts/tuning.lua#L6250)、[结算](../../../raw/current/scripts/components/perishable.lua#L127-L133)
- **误区：`canbepickedup = false` 表示捡不起来**：它只关掉直接拾取，入口换成 NET 工具动作——要手持带 `NET_tool` 标签的捕虫网类工具，服务端结算本身再查一次动作与工作次数。[L209-L215](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L209-L215)、[拾取动作选择](../../../raw/current/scripts/components/playercontroller.lua#L2005-L2019)、[NET 结算](../../../raw/current/scripts/actions.lua#L1859-L1877)
- **边界：赋值不触发回调，真正入口是 `onputininventory` 事件（可重复触发）**：`inst.components.inventoryitem.SetOnPutInInventoryFn = onpickup` 只是把实例上的方法名替换成 `onpickup`，并没有调用 setter，组件字段 `onputininventoryfn` 仍是 nil；实际生效的是 `onputininventory` 事件监听，每次 `PutInInventory`（含转移与堆叠合并）都会再触发一次。[L210](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L210)、[L229](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L229)、[setter L306-L307](../../../raw/current/scripts/components/inventoryitem.lua#L306-L307)、[事件推送 L357-L360](../../../raw/current/scripts/components/inventoryitem.lua#L357-L360)
- **边界：`MOONSTORM_SPARK_HEALTH` 不是月熠的血量**：全库唯一消费者是 `moonstorm_static`；月熠没有 `health` 组件，耗尽由 `perishable` 而非受伤触发。[消费端](../../../raw/current/scripts/prefabs/moonstorm_static.lua#L100)、[master 段组件清单](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L198-L233)
- **边界：作祟没有效果**：`MakeHauntablePerish(inst, .5)` 的回调在 `standardcomponents` 里整体被注释、恒返回 false，只剩冷却配置。[helper](../../../raw/current/scripts/standardcomponents.lua#L1294-L1309)
- **边界：3 是每次检查的逐玩家阈值，不是全场上限**：每 30 秒对每名玩家各重数一次半径 30 内的月熠，重叠范围的玩家会把别人刚刷出的计入；捕捉在 `GiveItem` 之前补检，这一次捕捉不释放名额（见「生成」节）。[L855-L861](../../../raw/current/scripts/components/moonstormmanager.lua#L855-L861)
- **边界：`SGspore` 的 `land`／`land_idle` 两态在本 prefab 没有触发入口**：全库检索没有指向本 SG 的 `GoToState("land")` 调用。[land 态](../../../raw/current/scripts/stategraphs/SGspore.lua#L83-L107)
- **持久化**：耗尽后 `persists = false`，不随存档保留；读档只补开灯光与阴影，不重置新鲜度。[onload L112-L116](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L112-L116)

## 验证记录

- 源码审阅：2026-11-09，steam-buildid:26474873（与 `raw/current/BUILD.txt` 一致）。覆盖 `prefabs/moonstorm_spark.lua` 全文、`components/moonstormmanager.lua`（生成任务、`DoTestForSparks`、`customcheckfn`、闪电分支）、`components/moonsparkchargeable.lua` 全文、`SGspore.lua` 与 `sporebrain.lua` 全文、`hats.lua` 护目镜段、`recipes.lua`（L593、L1530）、`monkeyisland_portal.lua` 接受判断、`electricattacks.lua`、`combat.lua` 与 `commonstates.lua` 的电击分支、`perishable.lua` 结算、`stackable.lua`／`inventoryitem.lua` 默认值、`actions.lua` 的 NET、`standardcomponents.lua` 的作祟 helper、`fx.lua` 表项、`components/playercontroller.lua` 的 `GetPickupAction`、`constants.lua` 的 `TOOLACTIONS`、`prefabs/bugnet.lua` 与 `prefabs/wortox_nabbag.lua` 的 `tool:SetAction`、`tuning.lua` 数值、`chinese_s.po` 名称键。
- 游戏运行：未运行；全部结论为源码审阅，数值为定义值加结算公式（推导，非实测）。
- 未覆盖：月熠在风暴中的实际生成数量与节奏、电击与作祟的实际观感、`moonstorm_static` 与本页之外的风暴对象细节。
- 修订（2026-11-09，独立检视 P2/P3 逐条）：①「拾取即补检会补一只」改为「补检发生在 GiveItem 之前，被捡这只仍计数，不腾名额」；②「`onpickup` 只跑一次」标题改「赋值不触发回调，真正入口是 `onputininventory` 事件（可重复触发）」；③NET 入口锚点由 `playercontroller` 的捉虫捷径（要求 `_health`）换成 `GetPickupAction` 的 `TOOLACTIONS` 循环，并补捕虫网／强抢袋的 `tool:SetAction` 证据；④图鉴字段由三个改为五个并给 L184-L188，灯色改写为 `(111,111,227)/255`；⑤`Panic` 补 `hauntable.panicable` 条件；⑥电击改为直接 `GetAttacked(…, "electric")` 结算，`electricattacks` 只供电击表现判定，删掉 `combat:DoAttack` 的 stimuli 说法；⑦sources 补 `components/playercontroller.lua`、`constants.lua`、`prefabs/bugnet.lua`、`prefabs/wortox_nabbag.lua`。均按点名条目改，未顺手动其它段落。P3 补修（同日）：铥矿捕虫网补 `bugnet.lua` L144-L146 锚点；验证记录覆盖清单补 `playercontroller.lua`、`constants.lua`、`bugnet.lua`、`wortox_nabbag.lua`。
- 外部检视 R14/R15（2026-11-10）：计时口径统一为「360 秒定义／地面约 240 秒／手持约 60 天」；生成阈值改为「每次检查逐玩家重数 30 半径、<3 才放行」，并在陷阱节补「不是全场上限、捕捉不释放名额」；电击节拍区分 5–15 秒等待与 5 帧前摇、0.5 秒复位任务并行；「满值需 10 只火花」改为「每次 +10 个百分点，10 次相当于一条满值回充量，仅期间无消耗才可直接累计；护目镜耗尽会移除，不能等它到零再回充」。

相关：[[reference/components/moonstormmanager]]、[[reference/components/moonsparkchargeable]]、[[reference/components/moonstorms]]。

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
