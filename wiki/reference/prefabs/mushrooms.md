---
title: "蘑菇族八注册：红／绿／蓝／月亮蘑菇的生熟两态（red_cap、green_cap、blue_cap、moon_cap 与各自 _cooked）"
created: "2026-11-08"
updated: "2026-11-09"
type: prefab
summary: "mushrooms.lua 的三项数据表经 mushfn／capfn／cookedfn 产出红／绿／蓝蘑菇三态，moon_mushroom.lua 另产月亮蘑菇两态；记录八注册的三维／鲜度／堆叠、可烤制与熟帽燃料、世界蘑菇／农场／蘑菇树／地精／蝠丘／蟾蜍等获得路径，以及 badfood 与月亮蘑菇昏睡接线。"
aliases: ["red_cap", "red_cap_cooked", "green_cap", "green_cap_cooked", "blue_cap", "blue_cap_cooked", "moon_cap", "moon_cap_cooked", "采摘的红蘑菇", "熟红蘑菇", "采摘的绿蘑菇", "熟绿蘑菇", "采摘的蓝蘑菇", "熟蓝蘑菇", "月亮蘑菇", "熟月亮蘑菇", "蘑菇", "蘑菇族"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/mushrooms.lua", "prefabs/moon_mushroom.lua", "prefabs/mushroom_farm.lua", "prefabs/mushtree.lua", "prefabs/mushtree_moon.lua", "prefabs/mushgnome.lua", "prefabs/molebathill.lua", "prefabs/toadstool.lua", "prefabs/skilltree_wormwood.lua", "components/edible.lua", "components/perishable.lua", "components/stackable.lua", "components/cookable.lua", "components/cooker.lua", "actions.lua", "components/tradable.lua", "components/pickable.lua", "components/worldstate.lua", "components/snowmandecoratable.lua", "components/regrowthmanager.lua", "standardcomponents.lua", "tuning.lua", "constants.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 蘑菇族八注册：红／绿／蓝／月亮蘑菇的生熟两态（red_cap、green_cap、blue_cap、moon_cap 与各自 _cooked）

## 快速判断

- 职责与适用：`prefabs/mushrooms.lua` 用一张 3 项数据表驱动 `mushfn`／`capfn`／`cookedfn` 三个工厂，每种颜色产出「世界蘑菇 + 生帽 + 熟帽」；`prefabs/moon_mushroom.lua` 另用 `capfn`／`cookedfn` 产出月亮蘑菇两态。本页覆盖 8 个物品注册，世界蘑菇 `red_mushroom`／`green_mushroom`／`blue_mushroom` 只作获得路径。[数据表](../../../raw/current/scripts/prefabs/mushrooms.lua#L447-L488)、[三工厂](../../../raw/current/scripts/prefabs/mushrooms.lua#L417-L445)、[注册循环](../../../raw/current/scripts/prefabs/mushrooms.lua#L490-L497)、[月亮蘑菇](../../../raw/current/scripts/prefabs/moon_mushroom.lua#L170-L171)
- 优先使用：调三维／保质／堆叠先看[生熟数值](#生熟数值与鲜度)；烤制产物看[烤制链](#烤制链)；获得看[获得路径](#世界蘑菇与获得路径)；月亮蘑菇的昏睡与醒神接线看[月亮蘑菇的食用回调](#月亮蘑菇的食用回调)。
- 重要副作用：生帽与熟帽的负生命或负理智赋值会立即给实体打 `badfood`；月亮蘑菇在进食者未被冰冻／钉住／石化时施加昏睡或困倦（有坐骑会同时推坐骑昏睡，技能另有分支；3 秒只是时长参数），熟月亮蘑菇扣饥饿与理智但清除眩晕。[badfood](../../../raw/current/scripts/components/edible.lua#L1-L7)、[睡眠](../../../raw/current/scripts/prefabs/moon_mushroom.lua#L25-L49)、[醒神](../../../raw/current/scripts/prefabs/moon_mushroom.lua#L112-L116)
- 修改边界：只讲八个物品形态的构造、数值与获得；世界蘑菇的相位、再生与分布见[世界蘑菇与获得路径](#世界蘑菇与获得路径)，蘑菇树／农场／地精／蝠丘／蟾蜍只列本页用到的产出关系；蘑菇帽、孢子、菜谱与 quagmire 数据不在本页。

## 注册与构造

八个注册分属两套工厂：`MakeMushroom(data)` 每色返回「世界蘑菇、生帽、熟帽」，月亮蘑菇文件直接返回「生帽、熟帽」；三色共 9 个注册，世界蘑菇不计入本页八注册。[返回顺序](../../../raw/current/scripts/prefabs/mushrooms.lua#L442-L444)、[月亮返回](../../../raw/current/scripts/prefabs/moon_mushroom.lua#L170-L171)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `red_cap` | `capfn` → `capcommonfn`：bank／build＝`mushrooms`、`PlayAnimation("red_cap")`、`pickupsound="vegetation_firm"`、公共端 tag `cookable`／`mushroom`、浮水 `{"small",0.1,0.88}` → `SetPristine` → 主模拟挂组件 | `cookable.product="red_cap_cooked"`；主模拟末挂 `snowmandecor`。堆叠 40。[公共函数](../../../raw/current/scripts/prefabs/mushrooms.lua#L302-L362)、[红帽数据](../../../raw/current/scripts/prefabs/mushrooms.lua#L450-L460) |
| `red_cap_cooked` | `cookedfn` → `cookedcommonfn`：同名 bank／build、`PlayAnimation("red_cap_cooked")`、浮水 `{"small",0.05,0.9}` → `SetPristine` → 主模拟 | 组件为 stackable／tradable／inspectable／fuel／burnable／propagator／inventoryitem／edible／perishable／hauntable；**无** `cookable`、**无** `mushroom` tag、无 `snowmandecor`。[熟帽工厂](../../../raw/current/scripts/prefabs/mushrooms.lua#L364-L415) |
| `green_cap` | 同 `capfn`：`PlayAnimation("green_cap")` | 数值与 `red_cap` 不同（见[生熟数值](#生熟数值与鲜度)）；堆叠 40；同样可烤制、可作雪人装饰。[绿帽数据](../../../raw/current/scripts/prefabs/mushrooms.lua#L463-L473) |
| `green_cap_cooked` | 同 `cookedfn`：`PlayAnimation("green_cap_cooked")` | 熟态只有 `HEALING_TINY` 的负生命与 `SANITY_MED` 的理智；除燃料外无专属接线。 |
| `blue_cap` | 同 `capfn`：`PlayAnimation("blue_cap")` | 生帽是四色里唯一给 `HEALING_MED` 正生命的一件；理智 `−SANITY_MED`。[蓝帽数据](../../../raw/current/scripts/prefabs/mushrooms.lua#L476-L486) |
| `blue_cap_cooked` | 同 `cookedfn`：`PlayAnimation("blue_cap_cooked")` | 熟态生命转为 `−HEALING_SMALL`、理智 `+SANITY_SMALL`。 |
| `moon_cap` | `capfn`（moon_mushroom.lua）：bank／build＝`moon_cap`、`PlayAnimation("moon_cap")`、tag `cookable`／`moonmushroom`／`mushroom`、浮水 `{"med",0.0,0.7}` → `SetPristine` → 主模拟 | 多一条 `edible:SetOnEatenFn(mooncap_oneaten)`；`cookable.product="moon_cap_cooked"`；**无** `snowmandecor`、无燃料。[月亮生帽](../../../raw/current/scripts/prefabs/moon_mushroom.lua#L51-L109) |
| `moon_cap_cooked` | `cookedfn`：同名 bank／build、`PlayAnimation("moon_cap_cooked")`、浮水 `{"small",0.05,1.0}` → `SetPristine` → 主模拟 | 燃料 `TINY_FUEL`、食用回调 `mooncap_cooked_oneaten`；无 `mushroom`／`moonmushroom` tag，不可再烤、不可进蘑菇农场。[月亮熟帽](../../../raw/current/scripts/prefabs/moon_mushroom.lua#L118-L168) |

数据表每项 11 个字段：`name`、`animname`、`pickloot`（生帽名，也拼出熟帽名）、`open_time`、生三维 `sanity`／`health`／`hunger`、熟三维 `cookedsanity`／`cookedhealth`／`cookedhunger` 与 `transform_prefab`；后者未被消费，对应的 `transformer` 接线整段被注释。[字段与三行数据](../../../raw/current/scripts/prefabs/mushrooms.lua#L447-L488)、[注释段](../../../raw/current/scripts/prefabs/mushrooms.lua#L241-L245)

- `capcommonfn` 在 `SetPristine` 前把 `cookable`／`mushroom` 写进公共端供客户端识别，组件只在主模拟添加。[tag](../../../raw/current/scripts/prefabs/mushrooms.lua#L318-L320)、[组件](../../../raw/current/scripts/prefabs/mushrooms.lua#L356-L357)
- 月亮蘑菇的 `moonmushroom` tag 只用于蘑菇农场的准入判断，不参与烤制。[tag](../../../raw/current/scripts/prefabs/moon_mushroom.lua#L67-L73)、[农场判定](../../../raw/current/scripts/prefabs/mushroom_farm.lua#L245-L265)

## 生熟数值与鲜度

时间基准 `seg_time`＝30、`total_day_time`＝480 秒，`perish_warp`＝1；饥饿基准 `calories_per_day`＝75。[时间基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[perish_warp](../../../raw/current/scripts/tuning.lua#L47-L47)、[热量基准](../../../raw/current/scripts/tuning.lua#L41-L41)

八注册的食物类型统一为 `FOODTYPE.VEGGIE`，鲜度统一 `SetPerishTime(TUNING.PERISH_MED)`＝10 天（4800 秒），腐坏替换 `spoiled_food`；堆叠统一 `TUNING.STACK_SIZE_SMALLITEM`＝40。[食性](../../../raw/current/scripts/constants.lua#L2026-L2030)、[生帽鲜度](../../../raw/current/scripts/prefabs/mushrooms.lua#L347-L350)、[熟帽鲜度](../../../raw/current/scripts/prefabs/mushrooms.lua#L409-L412)、[月亮鲜度](../../../raw/current/scripts/prefabs/moon_mushroom.lua#L98-L101)、[堆叠常量](../../../raw/current/scripts/tuning.lua#L81-L83)

| 注册 | 饥饿／生命／理智 | badfood | 备注与证据 |
|---|---|---|---|
| `red_cap` | 12.5／**−20**（`−HEALING_MED`）／0 | 有 | 负生命使生吃成为净损失。[红帽行](../../../raw/current/scripts/prefabs/mushrooms.lua#L455-L457) |
| `red_cap_cooked` | 0／+1（`HEALING_TINY`）／**−10**（`−SANITY_SMALL`） | 有 | 熟后只补 1 生命并扣 10 理智。[红熟行](../../../raw/current/scripts/prefabs/mushrooms.lua#L457-L459) |
| `green_cap` | 12.5／0／**−50**（`−SANITY_HUGE`） | 有 | 三色生帽里理智惩罚最大。[绿帽行](../../../raw/current/scripts/prefabs/mushrooms.lua#L467-L471) |
| `green_cap_cooked` | 0／**−1**（`−HEALING_TINY`）／+15（`SANITY_MED`） | 有 | 熟后理智回正，但生命仍为负。[绿熟行](../../../raw/current/scripts/prefabs/mushrooms.lua#L470-L472) |
| `blue_cap` | 12.5／**+20**（`HEALING_MED`）／**−15**（`−SANITY_MED`） | 有 | 唯一生吃给大额生命的一件。[蓝帽行](../../../raw/current/scripts/prefabs/mushrooms.lua#L480-L484) |
| `blue_cap_cooked` | 0／**−3**（`−HEALING_SMALL`）／+10（`SANITY_SMALL`） | 有 | 熟后生命转负、理智回正。[蓝熟行](../../../raw/current/scripts/prefabs/mushrooms.lua#L483-L485) |
| `moon_cap` | 12.5／0／+10（`SANITY_SMALL`） | 无 | 食用回调附加昏睡，见[月亮蘑菇的食用回调](#月亮蘑菇的食用回调)。[月亮生帽行](../../../raw/current/scripts/prefabs/moon_mushroom.lua#L92-L96) |
| `moon_cap_cooked` | **−12.5**（`−CALORIES_SMALL`）／0／**−10**（`−SANITY_SMALL`） | 有 | 熟后扣饥饿与理智，但清除眩晕。[月亮熟帽行](../../../raw/current/scripts/prefabs/moon_mushroom.lua#L156-L160) |

`badfood` 只由 `edible` 的 `healthvalue < 0` 或 `sanityvalue < 0` 触发，**不看饥饿**：因此 `moon_cap_cooked` 即便触发了负饥饿，也是靠 −10 理智进入 `badfood`；`moon_cap` 的正理智不吃惩罚。[判定](../../../raw/current/scripts/components/edible.lua#L1-L7)

### 烤制链

- 四个生帽（三色 + 月亮）都在主模拟挂 `cookable`，`product` 分别指向同名 `_cooked`；熟帽没有 `cookable`，不能二次烤制。[三色](../../../raw/current/scripts/prefabs/mushrooms.lua#L356-L357)、[月亮](../../../raw/current/scripts/prefabs/moon_mushroom.lua#L105-L106)
- 服务端 `ACTIONS.COOK` 从持有者物品栏取出待烤食材，交给 `Cooker:CookItem` → `Cookable:Cook`：新物品生成后，原料被 `Remove`，产物进入原持有者物品栏。[动作](../../../raw/current/scripts/actions.lua#L2013-L2039)、[锅](../../../raw/current/scripts/components/cooker.lua#L25-L50)
- `Cookable:Cook` 会把「已损失的新鲜度打五折」传给产物：`new_percent = 1 - (1 - raw_percent) * 0.5`。生帽与熟帽都是 10 天，满鲜生帽烤完仍满鲜，陈旧生帽烤后会回到两态之间的一半损失。[结算](../../../raw/current/scripts/components/cookable.lua#L28-L35)
- 熟帽额外带 `fuel`（`TINY_FUEL`＝7.5）且可燃烧 3 秒；生帽没有 `fuel`。八件的燃烧、熟帽的燃料都只在主模拟。[燃料](../../../raw/current/scripts/prefabs/mushrooms.lua#L392-L394)、[燃烧](../../../raw/current/scripts/prefabs/mushrooms.lua#L336-L337)、[常量](../../../raw/current/scripts/tuning.lua#L1673-L1681)
- 可烤目标在客户端靠 `cookable` tag 识别，服务端才检查 `components.cookable`；只给 tag 不给组件不会真的产出熟帽。[cookable 组件](../../../raw/current/scripts/components/cookable.lua#L18-L38)

### 月亮蘑菇的食用回调

- `mooncap_oneaten` 先排除被冰冻、被钉住、被石化的进食者，然后取 `MOON_MUSHROOM_SLEEPTIME`＝3 秒。**分支不是互斥的**：有坐骑时先对坐骑推 `ridersleep`（`sleepiness=4`），**不 return**，随后仍会走进食者自己的分支——Wormwood 技能 `wormwood_moon_cap_eating` 激活时在进食者位置生成 `sleepcloud_lunar` 并 `SetOwner`，否则按 `sleeper`／`grogginess` 施加 `AddSleepiness(4, 3)` 或 `AddGrogginess(2, 3)`，两者都没有才推 `knockedout`；3 秒是传给这些调用的时长参数，是否生效取决于目标身上的组件。[回调](../../../raw/current/scripts/prefabs/moon_mushroom.lua#L25-L49)、[技能](../../../raw/current/scripts/prefabs/skilltree_wormwood.lua#L139-L147)
- `mooncap_cooked_oneaten` 在进食者仍有效且有 `grogginess` 时调用 `ResetGrogginess()`，因此熟月亮蘑菇是「以饥饿和理智换醒神」的解毒类食物。[熟回调](../../../raw/current/scripts/prefabs/moon_mushroom.lua#L112-L116)

## 世界蘑菇与获得路径

三种世界蘑菇共用 `mushcommonfn`：`pickable:SetUp(data.pickloot, nil)` 只给产物、不给再生秒数，再生改由 `inst.rain` 计数驱动。[公共函数](../../../raw/current/scripts/prefabs/mushrooms.lua#L172-L271)、[pickable](../../../raw/current/scripts/prefabs/mushrooms.lua#L218-L224)

- **开放相位**：红在 `day`、绿在 `dusk`、蓝在 `night` 打开；`open_time` 与 `TheWorld.state.cavephase` 比较，主模拟用 `WatchWorldState("iscave"..open_time)` 跟随开关。未到相位时动画为 `inground` 且 `pickable.caninteractwith=false`，不能摘。[相位](../../../raw/current/scripts/prefabs/mushrooms.lua#L255-L265)、[开关](../../../raw/current/scripts/prefabs/mushrooms.lua#L74-L95)、[cavephase](../../../raw/current/scripts/components/worldstate.lua#L63-L70)
- **采摘与再生**：摘下后 `rain = 10 + math.random(10)`，播放 `picked`；之后每 `SEG_TIME` 检查一次，**仅在下雨**时把 `rain` 减 1，减到 0 才 `pickable:Regen()`。没有普通再生计时器；`rain` 实际落在 11～20，且只在雨中递减。周期任务在构造时就登记，首次检查延迟 `SEG_TIME + random(SEG_TIME)`＝30～60 秒；采摘只把 `rain` 重置为 11～20、不重启任务，所以再摘时间取决于剩余检查间隔，约 11～20 个雨间检查周期后再 `Regen()`。[摘取](../../../raw/current/scripts/prefabs/mushrooms.lua#L28-L35)、[再生](../../../raw/current/scripts/prefabs/mushrooms.lua#L41-L48)
- **挖掘**：`workable` 用 `ACTIONS.DIG`、1 次工作完成；完成回调只看 `pickable:CanBePicked()`（只读 `canbepicked`，**与相位和 `caninteractwith` 无关**），为真先掉 1 个帽，再无条件下掉 1 个，然后移除植株。所以**当前未采空的植株挖出 2 个帽**（闭合相位也能挖；采过但已再生的同样算未采空），已采空、尚未再生的才只给 1 个。[挖掘](../../../raw/current/scripts/prefabs/mushrooms.lua#L228-L239)
- **移除后的世界再生**：植株挂 `AddToRegrowthManager`，移除时向世界推 `beginregrowth`；为三色各登记 `MUSHROOM_REGROWTH_TIME`＝20 天，并在 `isday`／`isdusk`／`isnight` 匹配时按 `MUSHROOM_REGROWTH_TIME_FAST_MULT`＝2 加速到 10 天（地表分片的 `OnPhaseChanged` 直接调用 `OnCavePhaseChanged`，`cavephase` 跟地表相位走；洞穴分片由自己的时钟驱动）。[同步](../../../raw/current/scripts/components/worldstate.lua#L72-L80)[登记](../../../raw/current/scripts/components/regrowthmanager.lua#L198-L212)、[常量](../../../raw/current/scripts/tuning.lua#L3183-L3185)、[挂接](../../../raw/current/scripts/standardcomponents.lua#L1843-L1847)
- **作祟换色**：对世界蘑菇作祟有 `HAUNT_CHANCE_OCCASIONAL`＝25% 概率换成同类型另一颜色，保留可摘／入土状态；否则开放时关回土里。[换色表](../../../raw/current/scripts/prefabs/mushrooms.lua#L101-L115)、[回调](../../../raw/current/scripts/prefabs/mushrooms.lua#L121-L170)
- **分布**：世界设置里的 `mushroom` 一项同时管地表与洞穴；`forest_map` 把它翻译成三色摆放名，地块白名单是 ROAD／WOODFLOOR／SCALE／CARPET／CHECKER／ROCKY／MARSH／DIRT／DESERT_DIRT／MONKEY_DOCK 十类。地表恶地房间三色各 0.3、`chess` 与 `blockers` 各 0.02；洞穴 `cave/generic` 三色各 0.1、`cave/fungusnoise` 各 0.5、`cave/bluemush` 只放蓝（0.5／2.5）。[设置](../../../raw/current/scripts/map/customize.lua#L359-L359)、[翻译](../../../raw/current/scripts/map/forest_map.lua#L198-L198)、[白名单](../../../raw/current/scripts/map/terrain.lua#L52-L54)、[恶地](../../../raw/current/scripts/map/rooms/forest/DLCrooms.lua#L138-L140)、[洞穴](../../../raw/current/scripts/map/rooms/cave/generic.lua#L34-L36)

| 获得路径 | 产出 | 条件与证据 |
|---|---|---|
| 世界蘑菇采摘 | `red_cap`／`green_cap`／`blue_cap` | 必须对应该颜色的开放相位（`caninteractwith`）；见上。 |
| 世界蘑菇挖掘 | 同上 | `ACTIONS.DIG` 不看相位，只按 `CanBePicked()` 决定 2 个还是 1 个；见上。 |
| `mushroom_farm`（蘑菇农场） | 投入的帽或孢子对应的帽 | 生帽种同名；孢子按 `spore_tall→blue_cap`、`spore_medium→red_cap`、`spore_small→green_cap`（孢子物品与喷发见 [[reference/prefabs/mushtree-spores]]）。孢子或 Wormwood 升级时产量 6，否则 4；`MUSHROOMFARM_FULL_GROW_TIME`＝3.75 天是基础满产时长，Wormwood 技能 `wormwood_mushroomplanter_ratebonus1`／`2` 再乘 0.9／0.8；种下时 `harvestable:Grow()` 立即长出第一份，此后每份产物的间隔＝`grow_time / 产量`；收 4 次后需 `livinglog`；雪覆盖转 `spoiled_food`。[映射](../../../raw/current/scripts/prefabs/mushroom_farm.lua#L33-L38)、[生长](../../../raw/current/scripts/prefabs/mushroom_farm.lua#L46-L76)、[准入](../../../raw/current/scripts/prefabs/mushroom_farm.lua#L245-L275)、[常量](../../../raw/current/scripts/tuning.lua#L1661-L1663) |
| `mushtree_small`／`mushtree_medium`／`mushtree_tall` | 绿／红／蓝帽 | 砍倒后 `DropLoot` 给固定表：小树 `log`＋`green_cap`、中树 `log`＋`red_cap`、高树 2 `log`＋`blue_cap`；树桩另留。[树数据](../../../raw/current/scripts/prefabs/mushtree.lua#L309-L351)、[掉落](../../../raw/current/scripts/prefabs/mushtree.lua#L626-L633) |
| `mushtree_moon`（月亮蘑菇树） | `moon_cap` | 砍倒固定掉 2 `log`＋1 `moon_cap`。[数据](../../../raw/current/scripts/prefabs/mushtree_moon.lua#L74-L92)、[掉落](../../../raw/current/scripts/prefabs/mushtree_moon.lua#L222-L230) |
| `mushgnome`（蘑菇地精） | `moon_cap`×2 | 共享掉落表两条 `moon_cap` 均为 1.0 概率。[掉落](../../../raw/current/scripts/prefabs/mushgnome.lua#L18-L29) |
| `molebathill`（裸鼹蝠丘） | `moon_cap` | 随机挖掘表权重 0.5，每次抽 2 件（同表有胡萝卜种子 0.10、草 5）。[表](../../../raw/current/scripts/prefabs/molebathill.lua#L12-L21) |
| `toadstool`／`toadstool_dark`（毒菌蟾蜍） | 三色帽各 1.0＋0.33＋0.33 | 两份蟾蜍掉落表相同三色条目。[普通](../../../raw/current/scripts/prefabs/toadstool.lua#L100-L110)、[暗影](../../../raw/current/scripts/prefabs/toadstool.lua#L126-L136) |
| 蘑菇农场种月亮蘑菇 | `moon_cap` | 准入要求 `moonmushroom` tag 且种植者已点 Wormwood 技能 `wormwood_moon_cap_eating`，否则拒绝并提示 `MUSHROOMFARM_NOMOONALLOWED`；月亮蘑菇没有孢子映射。[准入](../../../raw/current/scripts/prefabs/mushroom_farm.lua#L245-L265) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | 写三维与 `FOODTYPE.VEGGIE`；月亮两态另设 `SetOnEatenFn` | 主模拟；`badfood` 在赋值时判定 | [[reference/components/edible#关键接口]]；[三色生](../../../raw/current/scripts/prefabs/mushrooms.lua#L341-L345)、[月亮](../../../raw/current/scripts/prefabs/moon_mushroom.lua#L91-L96) |
| 组件：perishable | `SetPerishTime(PERISH_MED)`＋`StartPerishing`；腐坏替换 `spoiled_food` | 八件同值 10 天；主模拟 | [[reference/components/perishable#关键接口]]；[生](../../../raw/current/scripts/prefabs/mushrooms.lua#L347-L350)、[熟](../../../raw/current/scripts/prefabs/mushrooms.lua#L409-L412) |
| 组件：stackable | 构造后显式把 `maxsize` 设为 40 | 生熟一致；组件默认是 20 | [[reference/components/stackable#关键接口]]；[设置](../../../raw/current/scripts/prefabs/mushrooms.lua#L330-L331)、[默认](../../../raw/current/scripts/components/stackable.lua#L23-L28) |
| 组件：cookable | `product` 指向同名熟帽；熟帽无此组件 | 仅生帽；客户端靠 tag 识别 | [[reference/components/cookable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/mushrooms.lua#L356-L357)、[Cook](../../../raw/current/scripts/components/cookable.lua#L18-L38) |
| 组件：fuel／burnable／propagator | 熟帽给 `TINY_FUEL`＝7.5；八件都 `MakeSmallBurnable`（3 秒）＋`MakeSmallPropagator` | 生帽无 `fuel`、熟帽无 `cookable` | [[reference/components/fuel#关键接口]]、[[reference/components/burnable#关键接口]]；[燃料](../../../raw/current/scripts/prefabs/mushrooms.lua#L392-L393)、[生](../../../raw/current/scripts/prefabs/mushrooms.lua#L336-L337)、[熟](../../../raw/current/scripts/prefabs/mushrooms.lua#L394-L395) |
| 组件：tradable | 只挂载，不设 `goldvalue`（默认 0） | 八件都有；猪王拒收 | [[reference/components/tradable#关键接口]]；[挂载](../../../raw/current/scripts/prefabs/mushrooms.lua#L333-L334)、[默认](../../../raw/current/scripts/components/tradable.lua#L1-L7) |
| 组件：hauntable | 帽：`MakeHauntableLaunchAndPerish`＋10% 换色；植物：25% 换色 | 帽换色保留堆叠／湿度／新鲜度；熟帽同样换色 | [[reference/components/hauntable#关键接口]]；[帽](../../../raw/current/scripts/prefabs/mushrooms.lua#L352-L354)、[回调](../../../raw/current/scripts/prefabs/mushrooms.lua#L273-L300) |
| 组件：pickable | 世界蘑菇只设产物与回调，不设再生秒数 | 仅三色世界蘑菇；相位与雨计数控制 | [[reference/components/pickable#关键接口]]；[SetUp](../../../raw/current/scripts/prefabs/mushrooms.lua#L218-L224)、[Regen](../../../raw/current/scripts/components/pickable.lua#L386-L396) |
| 组件：workable | 世界蘑菇可 `DIG`，1 次完成并移除 | 1～2 个帽；植株消失后交世界再生 | [[reference/components/workable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/mushrooms.lua#L228-L239) |
| 组件：snowmandecor | 生帽挂空标记组件，雪人装饰表按注册名取动画 | 仅 `red_cap`／`green_cap`／`blue_cap`；月亮生帽与所有熟帽没有 | [[reference/components/snowmandecor]]；[挂载](../../../raw/current/scripts/prefabs/mushrooms.lua#L359-L359)、[表](../../../raw/current/scripts/components/snowmandecoratable.lua#L11-L38) |
| 组件：floater／inventoryitem | `MakeInventoryFloatable` 控制浮水；`pickupsound` 只在生帽公共端设 | 生帽三色小号、月亮中号；熟帽小号 | [[reference/components/floater#关键接口]]；[生](../../../raw/current/scripts/prefabs/mushrooms.lua#L322-L322)、[熟](../../../raw/current/scripts/prefabs/mushrooms.lua#L378-L378) |
| 关联 Prefab：`mushroom_farm` | 用 `mushroom`／`spore`／`moonmushroom` tag 准入并种植 | 设施侧组件归其页面（Wiki 待建） | [准入](../../../raw/current/scripts/prefabs/mushroom_farm.lua#L245-L265) |
| 资源：`anim/mushrooms.zip`／`anim/moon_cap.zip` | 三色与月亮各自一个 bank／build；生熟共用同一包 | 生帽动画名＝颜色帽／`moon_cap`，熟帽＝同名 + `_cooked` | [三色资产](../../../raw/current/scripts/prefabs/mushrooms.lua#L1-L14)、[月亮资产](../../../raw/current/scripts/prefabs/moon_mushroom.lua#L1-L11) |

## 典型接入

### 统一八件蘑菇的堆叠与保质

- 前提与选择：服务端 `modmain.lua`；八件独立注册，须逐件 postinit，不能改本地数据表回灌。工厂在 postinit 之前就挂好 `perishable` 并 `StartPerishing()`（此刻满鲜），用 `SetNewMaxPerishTime` 按当前百分比换算、不会把已入包的食物回鲜。
- 生效与撤销：postinit 每次构造运行；停用模组恢复工厂原值。读档时 `Perishable:OnLoad` 照抄存档剩余秒数，比例按新上限重算。[SetNewMaxPerishTime](../../../raw/current/scripts/components/perishable.lua#L204-L208)
- 原版依据：八件的原始值见[生熟数值与鲜度](#生熟数值与鲜度)；工厂在挂完组件后立即 `StartPerishing`，故 postinit 改上限时仍满鲜。[Perishable:OnLoad](../../../raw/current/scripts/components/perishable.lua#L337-L349)

```lua
-- modmain.lua：八件蘑菇统一堆叠 20、保质 10 天
local function retune(prefab)
    AddPrefabPostInit(prefab, function(inst)
        if not GLOBAL.TheWorld.ismastersim then
            return
        end
        if inst.components.stackable ~= nil then
            inst.components.stackable.maxsize = 20
        end
        if inst.components.perishable ~= nil then
            inst.components.perishable:SetNewMaxPerishTime(GLOBAL.TUNING.PERISH_MED)
        end
    end)
end

for _, prefab in ipairs({
    "red_cap", "red_cap_cooked",
    "green_cap", "green_cap_cooked",
    "blue_cap", "blue_cap_cooked",
    "moon_cap", "moon_cap_cooked",
}) do
    retune(prefab)
end
```

- 调整与验证：单位：个与秒；`maxsize` 影响新构造实例，`SetNewMaxPerishTime` 对已有实例按比例保留。

## 常见陷阱与限制

- **误区：熟帽是生帽的升级**：`red_cap_cooked` 只回 1 生命却扣 10 理智，`green_cap_cooked` 仍 −1 生命，`blue_cap_cooked` 把 +20 变成 −3；只有月亮熟帽的「醒神」是独特收益。[红熟](../../../raw/current/scripts/prefabs/mushrooms.lua#L457-L459)、[绿熟](../../../raw/current/scripts/prefabs/mushrooms.lua#L470-L472)、[蓝熟](../../../raw/current/scripts/prefabs/mushrooms.lua#L483-L485)
- **边界：`badfood` 不看饥饿**：判定只看负生命或负理智；月亮熟帽的负饥饿本身不触发，靠 −10 理智触发。[判定](../../../raw/current/scripts/components/edible.lua#L1-L7)
- **边界：只有生帽带 `mushroom` tag**：熟帽没有该 tag，不能塞进蘑菇农场；`moon_cap` 还另带 `moonmushroom`，农场只在种植者点了 Wormwood 技能时接受。[生 tag](../../../raw/current/scripts/prefabs/mushrooms.lua#L318-L320)、[准入](../../../raw/current/scripts/prefabs/mushroom_farm.lua#L245-L265)
- **边界：生帽无 `fuel`、熟帽无 `cookable`**：把熟帽当燃料可以，把熟帽再烤不行；反过来生帽可烤但作燃料无效。[生](../../../raw/current/scripts/prefabs/mushrooms.lua#L336-L350)、[熟](../../../raw/current/scripts/prefabs/mushrooms.lua#L386-L396)
- **边界：世界蘑菇没有普通再生计时器**：`SetUp(product, nil)` 不启动 `regentime`，只有 `inst.rain` 在雨中递减到 0 才 `Regen`；不下雨就不恢复可摘。[SetUp](../../../raw/current/scripts/components/pickable.lua#L222-L228)、[雨计数](../../../raw/current/scripts/prefabs/mushrooms.lua#L41-L48)
- **边界：作祟换色先弹飞再换**：帽的 `MakeHauntableLaunchAndPerish` 是主回调，`AddHauntableCustomReaction(..., true, false, true)` 作为副回调后执行；因此 10% 换色与 100% 弹飞叠加，新帽继承堆叠、湿度和新鲜度。[副回调](../../../raw/current/scripts/standardcomponents.lua#L1528-L1552)、[换色](../../../raw/current/scripts/prefabs/mushrooms.lua#L281-L289)
- **未核验**：农场产量／雪覆盖、月亮蘑菇昏睡与坐骑分支、quagmire 钩子均未运行验证；`event_server_data` 的 quagmire 数据不在本快照 `scripts` 根内。

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26335394。核对 `mushrooms.lua` 全文（数据表、三工厂、世界蘑菇／帽／熟帽公共函数、换色与作祟、注册循环）、`moon_mushroom.lua` 全文（两工厂、食用回调、资源）、`mushroom_farm.lua` 的准入／生长／收获／雪覆盖分支、`mushtree.lua` 与 `mushtree_moon.lua` 的掉落表与砍倒回调、`mushgnome`／`molebathill`／`toadstool` 掉落、Wormwood 技能表，以及 `edible`／`perishable`／`stackable`／`cookable`／`cooker`／`tradable`／`pickable`／`worldstate`／`regrowthmanager` 的相关分支；`standardcomponents` 的燃烧、作祟、浮水与再生 helper，`tuning` 与 `constants` 的相关常量。
  名称核验：`NAMES.RED_CAP`＝采摘的红蘑菇、`RED_CAP_COOKED`＝熟红蘑菇、`GREEN_CAP`＝采摘的绿蘑菇、`GREEN_CAP_COOKED`＝熟绿蘑菇、`BLUE_CAP`＝采摘的蓝蘑菇、`BLUE_CAP_COOKED`＝熟蓝蘑菇、`MOON_CAP`＝月亮蘑菇、`MOON_CAP_COOKED`＝熟月亮蘑菇；世界蘑菇为红蘑菇／绿蘑菇／蓝蘑菇。[RED_CAP](../../../raw/current/scripts/languages/chinese_s.po#L348545-L348547)、[RED_CAP_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L348550-L348552)、[GREEN_CAP](../../../raw/current/scripts/languages/chinese_s.po#L343450-L343452)、[GREEN_CAP_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L343455-L343457)、[BLUE_CAP](../../../raw/current/scripts/languages/chinese_s.po#L339565-L339567)、[BLUE_CAP_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L339570-L339572)、[MOON_CAP](../../../raw/current/scripts/languages/chinese_s.po#L345820-L345822)、[MOON_CAP_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L345825-L345827)
  未覆盖：没有穷举全部动态生成点（模组注入、活动脚本、`leafpile` 等次级掉落）；蘑菇帽与菜谱未展开；`quagmire` 事件数据缺失。
- 游戏运行：未运行。三维、天数与秒数均为 tuning 推导（`HEALING_MED`＝20、`PERISH_MED`＝10×480 秒、`MOON_MUSHROOM_SLEEPTIME`＝3），非运行测量。
- 结构检查：`validate_page.py` 通过；`validate_source_links.py --strict` 无 error／warning；页内 Lua 示例通过 `parser_probe.exe --syntax-only`。
- 复核：2026-11-08，Steam buildid 26474873。核对本页引用源在 26335394→26474873 间的差异（`tuning.lua`、`plantables.lua` 各插入 1 行，`SGwilson.lua` 仅换声效），锚点按新树重定位、结论未变；上一条 26335394 审阅记录保留作历史。

## 相关链接

- 同类数据表驱动的生熟食物：[[reference/prefabs/farm-crops-staple]]、[[reference/prefabs/farm-crops-fruit]]、[[reference/prefabs/wild-berries]]
- 可食与鲜度：[[reference/components/edible]]、[[reference/components/perishable]]
- 采摘与世界再生：[[reference/components/pickable]]、[[reference/components/regrowthmanager]]

返回 [[reference/prefabs/index|Prefab 索引]]。
