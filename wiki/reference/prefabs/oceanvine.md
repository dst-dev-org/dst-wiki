---
title: "苔藓藤条（oceanvine）与苔藓藤条补丁（oceanvine_patch）"
created: "2026-11-08"
updated: "2026-11-09"
type: prefab
summary: "海面垂挂的可采藤条：NOBLOCK／flying／high_dolongaction 三个标签的实际后果、fig 的 2.5 天再生与结果／摘空两套动画、烧掉 2 秒后自毁并掉 1 份灰、被巨树撞船时补藤；以及本构建里没有任何生成点的 oceanvine_patch（首批定时、2～3 天随机补藤、存读档续计与当前不可用的证据）。"
aliases: ["oceanvine", "oceanvine_patch", "苔藓藤条", "藤条", "苔藓藤", "Mossy Vine"]
tags: [prefab]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/oceanvine.lua", "prefabs/oceantree_pillar.lua", "prefabs/watertree_pillar.lua", "prefabs/oceanvine_deco.lua", "prefabs/oceanvine_cocoon.lua", "prefablist.lua", "components/pickable.lua", "components/map.lua", "debugtools.lua", "componentutil.lua", "standardcomponents.lua", "stategraphs/SGwilson.lua", "simutil.lua", "tuning.lua", "entityscript.lua", "main.lua", "tiledefs.lua", "prefabs/books.lua", "map/layouts.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 苔藓藤条（oceanvine）与苔藓藤条补丁（oceanvine_patch）

## 快速判断

- 职责与适用：`prefabs/oceanvine.lua` 两个注册。`oceanvine` 是挂在水中巨树旁、垂到海面的可采藤条（`pickable` 产物 `fig`）；`oceanvine_patch` 是本该「一批批补藤」的生成器。藤条本体无战斗、无 Brain、无物理碰撞，只有采摘、燃烧与两种掉落收场。[两个注册](../../../raw/current/scripts/prefabs/oceanvine.lua#L334-L335)
- 优先使用：问「多久结一次果、采完什么样」看[采摘与再生](#采摘与再生)；问「为什么船能贴着它放」看[三个标签的实际后果](#三个标签的实际后果)；问「谁生成它、补丁怎么计时」看[生成来源与补丁](#生成来源与补丁)。`fig`／`fig_cooked` 的数值与熟制见 [[reference/prefabs/tropical-gathered-fruits]]。
- 重要副作用：采摘会改播 `harvest`→`idle_nofruit` 并移除 `inspectable`（`fig` 符号由 `makeempty` 隐藏），同时惊动 16 半径内所有带 `webbed` tag 的实体（蛛茧会放蜘蛛）；点燃后 2 秒进入烧毁动画、`persists=false`，起手即在原地抛 1 份 `ash`，`animover` 时移除；巨树被砍倒时藤条会先落地再消失（地面 `ErodeAway`，水面直接移除）。[采摘](../../../raw/current/scripts/prefabs/oceanvine.lua#L205-L214)、[烧毁](../../../raw/current/scripts/prefabs/oceanvine.lua#L165-L186)、[落地](../../../raw/current/scripts/prefabs/oceanvine.lua#L243-L263)
- 修改边界：`fig`／`fig_cooked` 的三维、鲜度与配方见 [[reference/prefabs/tropical-gathered-fruits]]；`oceanvine_cocoon`（蛛茧）与 `oceanvine_deco`（纯装饰）不属本主题，只在[同族排除](#同族排除)说明。本页只写藤条与补丁。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `oceanvine` | `commonfn`：transform／anim／sound／dynamicshadow／minimap／network → 阴影 1.5×0.75 → bank／build `oceanvine`＋`idle_fruit` ＋`scrapbook_anim="idle_fruit"` → tag `hangingvine`／`flying`／`NOBLOCK`／`oceanvine`／`high_dolongaction` → `SetPristine` → 主模拟：inspectable、pickable、可燃可传播、作祟，并暴露 `placegoffgrids`／`fall`／`OnLoadPostPass` | 可采藤条；距离淡出代码在本构建**被整段注释**，实际不生效。[工厂](../../../raw/current/scripts/prefabs/oceanvine.lua#L265-L332)、[注释段](../../../raw/current/scripts/prefabs/oceanvine.lua#L288-L293) |
| `oceanvine_patch` | `patchfn`：只有 transform 与 animstate，`DoTaskInTime(0, spawnvines)` 做首批，挂 `OnSave`／`OnLoad`，把 `spawnNewVine` 暴露到实例上 | 无 tag、无物理、无组件；**全 scripts 没有 `SpawnPrefab("oceanvine_patch")`**，`prefablist.lua` 也未列入，实际无法通过正常途径出现。[工厂](../../../raw/current/scripts/prefabs/oceanvine.lua#L144-L154)、[预载清单](../../../raw/current/scripts/prefablist.lua#L878-L880) |

名称：`STRINGS.NAMES.OCEANVINE`＝苔藓藤条（Mossy Vine）；`oceanvine_patch` 没有名称键，也不进图鉴。[名称键](../../../raw/current/scripts/languages/chinese_s.po#L346580-L346582)、[图鉴条目](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1068-L1068)

### 三个标签的实际后果

| 标签 | 谁在读 | 实际后果与证据 |
|---|---|---|
| `NOBLOCK` | 部署空位检查 `Map:IsDeployPointClear` 的忽略表 | 船只与建筑的放置判定会跳过带此 tag 的实体，所以藤条不会挡住船的部署（源码注释也写明「To not block boat deployment」）。[放置忽略表](../../../raw/current/scripts/components/map.lua#L189-L192)、[船的判定](../../../raw/current/scripts/components/map.lua#L580-L603)、[注释](../../../raw/current/scripts/prefabs/oceanvine.lua#L285-L285) |
| `flying` | 船放置时把附近物品推开的逻辑（忽略表含 `flying`） | 下锚／放船不会把藤条当物品弹飞；同理它没有物理体，玩家穿过即过。[忽略表](../../../raw/current/scripts/componentutil.lua#L72-L72) |
| `high_dolongaction` | 玩家长动作 `dolongaction` 的分支 | 采摘藤条时播「高处作业」的 `construct_pre`／`construct_loop`／`construct_pst`，骑乘时退回普通动作；它不改变耗时，只改动画。[读取](../../../raw/current/scripts/stategraphs/SGwilson.lua#L8372-L8376) |

补充：`hangingvine` 与 `oceanvine` 两个 tag 是给别处按 tag 检索用的：巨树补藤时 `FindEntities` 用 `oceanvine` 统计周围藤条数量；`oceanvine` 另在 `HARVESTABLE_PLANT_TARGET_TAGS`（暗影收割者右键采收按此族筛选，见 [[reference/prefabs/voidcloth_scythe]]）与园艺学书候选表里。[统计](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L248-L262)、[采集族](../../../raw/current/scripts/componentutil.lua#L166-L172)、[书本候选](../../../raw/current/scripts/prefabs/books.lua#L22-L23)

### 采摘与再生

- 接线：`pickable:SetUp("fig", TUNING.OCEANVINE_REGROW_TIME)`、`picksound="dontstarve/wilson/harvest_berries"`、`onpickedfn`／`makeemptyfn`／`makefullfn` 三个回调；`max_cycles = nil` 且 `cycles_left = 1`，所以永不枯死。[接线](../../../raw/current/scripts/prefabs/oceanvine.lua#L305-L314)、[SetUp](../../../raw/current/scripts/components/pickable.lua#L222-L228)
- 时长：`OCEANVINE_REGROW_TIME = total_day_time * 2.5`＝2.5 天＝1200 秒；春季 `SpringGrowthMod` 再乘 0.75（900 秒）。[常量](../../../raw/current/scripts/tuning.lua#L6366-L6366)、[时间基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[春季修正](../../../raw/current/scripts/simutil.lua#L563-L565)
- 采摘表现：`onpicked` 先惊动蛛茧，再播 `harvest`（非循环）→ `idle_nofruit`（循环），并移除 `inspectable`；`Pick` 末尾按 `baseregentime` 起再生任务。[采摘](../../../raw/current/scripts/prefabs/oceanvine.lua#L205-L214)、[Pick](../../../raw/current/scripts/components/pickable.lua#L535-L583)
- 再生表现：它**没有 `onregenfn`**，恢复可见性靠 `makefullfn`——`AnimState:Show("fig")`，非 `POPULATING` 时播 `fruit_grow` → `idle_fruit`，并把 `inspectable` 加回来。[makefull](../../../raw/current/scripts/prefabs/oceanvine.lua#L225-L237)、[Regen 的顺序](../../../raw/current/scripts/components/pickable.lua#L386-L396)
- 摘空态：`makeemptyfn` 隐藏 `fig` 符号、播 `idle_nofruit`、移除 `inspectable`；读档时由 `pickable` 的 `OnLoad` 按 `data.picked` 决定调空态还是满态，`OnLoadPostPass` 再把动画帧随机化（`scrapbook_anim` 固定为 `idle_fruit`）。[空态](../../../raw/current/scripts/prefabs/oceanvine.lua#L216-L223)、[读档随机帧](../../../raw/current/scripts/prefabs/oceanvine.lua#L239-L241)、[pickable 读档](../../../raw/current/scripts/components/pickable.lua#L333-L345)
- 动作：藤条没有 `quickpick`，玩家走 `dolongaction`；配合 `high_dolongaction` 播高处动画（见上表）。[动作选择](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1133-L1138)

### 燃烧与灰烬

- 接线：`MakeSmallBurnable(inst, nil, nil, nil, "swap_fire")`（默认燃烧时长 10 秒、火焰符号换成 `swap_fire`），随后把火焰特效改成 `character_fire` 并 `followaschild`、偏移 (0,1,0)，再把燃烧时长压到 `BURN_DURATION + 5`＝7 秒，点火／烧尽回调都换成本文件的实现。[接线](../../../raw/current/scripts/prefabs/oceanvine.lua#L316-L323)、[常量](../../../raw/current/scripts/prefabs/oceanvine.lua#L14-L14)、[小燃可燃](../../../raw/current/scripts/standardcomponents.lua#L164-L174)
- 流程：`OnIgnite` 排一个 2 秒任务；期间若被扑灭，`OnExtinguishNotFinishedBurning` 取消任务、藤条保住。2 秒后 `OnStartBurnAnim`：`persists=false`、移除 `inspectable` 与 `pickable`、把扑灭回调也改成 `inst.Remove`、播 `burn` 并在 `animover` 时移除，同时**手动生成 1 份 `ash`** 并给它一个向上的初速（水平 0～2、垂直 8～12）。[点火](../../../raw/current/scripts/prefabs/oceanvine.lua#L195-L198)、[烧毁](../../../raw/current/scripts/prefabs/oceanvine.lua#L165-L186)、[扑灭](../../../raw/current/scripts/prefabs/oceanvine.lua#L188-L193)
- 边界：它不走 `DefaultBurntFn`，所以灰的数量恒为 1（不看堆叠），也没有「海里不留灰」的判断——海面上的藤条照样留灰。[对照](../../../raw/current/scripts/standardcomponents.lua#L33-L43)
- 作祟：`MakeHauntableIgnite`，点火分支在当前构建被注释，只有冷却与恒 false 的回调。[工厂](../../../raw/current/scripts/standardcomponents.lua#L1173-L1189)

### 掉落的两种收场

`fall` 是巨树砍倒时对藤条的收场函数（`fall_down_fn` 则用它播「长出来」的 `spawn` 动画）：

- `persists = false`，按落点判断：可见地面（含甲板平台）播 `fall_land` 并在 `animover` 时 `ErodeAway`；海面播 `fall_ocean` 并在 `animover` 时直接 `Remove`。[分支](../../../raw/current/scripts/prefabs/oceanvine.lua#L243-L253)
- 第 19 帧（`19*FRAMES`）时若仍 `CanBePicked()`，先 `MakeEmpty()` 再把 1 个产物 `SpawnPrefab("fig")` 放到落点（y 归零）——所以藤条掉下来还能捡到 1 个无花果，捡完再消失。[补偿](../../../raw/current/scripts/prefabs/oceanvine.lua#L254-L261)
- 巨树被移除时，先让「附近没有别的巨树」的藤条逐个 `fall`，再从**剩余**藤条里补到 3 条——补的只是现存藤条，总数本来不足 3 条时就有多少倒多少，不会新生成；范围内没有藤条时直接返回。[调用方](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L370-L410)

### 生成来源与补丁

- 藤条由水中巨树的树干生成：`oceantree_pillar` 在周围藤条不足 1 条时补 1～2 条（新藤先 `MakeEmpty` 再 `fall_down_fn` 播落下动画），`watertree_pillar` 另有一份同名 `SpawnMissingVines`，被船撞击时同样补藤，但阈值是「现存 < `OCEANTREE_VINE_DROP_MAX`(4)＋`math.random(1,2)`−1」，即每次在 4 与 5 之间随机，再严格按「现存藤数 < 阈值」判断：已有 4 条时只有抽到 5 才补，已有 5 条及以上不补；且新藤不先 `MakeEmpty`。[树柱补藤](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L234-L262)、[撞船补藤](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L217-L236)、[撞船触发](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L238-L264)、[阈值常量](../../../raw/current/scripts/tuning.lua#L6359-L6359)
- 布局与世界生成：`map/layouts.lua` 把 `oceanvine`／`oceanvine_deco`／`oceanvine_cocoon` 放进巨树布局的候选表（只是候选，不等于每次实际放置）。[布局候选](../../../raw/current/scripts/map/layouts.lua#L29-L30)、[蛛茧位](../../../raw/current/scripts/map/layouts.lua#L56-L60)
- 蛛茧联动：每次采摘都 `PushEvent("activated", {target = picker})` 给 `TUNING.OCEANVINE_COCOON_SPIDER_RADIUS`＝16 内所有 `webbed` 实体，把采摘者标成目标；蛛茧自身的行为归 `oceanvine_cocoon`（Wiki 待建）。[惊动](../../../raw/current/scripts/prefabs/oceanvine.lua#L156-L163)、[半径常量](../../../raw/current/scripts/tuning.lua#L6299-L6299)
- `oceanvine_patch` 的设定行为（源码意图）：首批 `spawnvines` 生成 8～16 条 `hanging_vine` 与 6～9 条 `grabbing_vine`，落在半径 12～14 内、避开已有 `hangingvine` 的格点，且只放在 `WORLD_TILES.DEEPRAINFOREST` 地块上（该地块键在本构建未注册，见下条）；之后每次 `spawnNewVine` 在 `TOTAL_DAY_TIME*2 + TOTAL_DAY_TIME*random()`（2～3 天）后补 1 条。[首批](../../../raw/current/scripts/prefabs/oceanvine.lua#L80-L89)、[格点筛选](../../../raw/current/scripts/prefabs/oceanvine.lua#L23-L70)、[补藤计时](../../../raw/current/scripts/prefabs/oceanvine.lua#L91-L105)
- 存读档：`OnSave` 记 `spawnedchildren` 与每个待办任务的「剩余秒数」（`TimeRemainingInTask`），`OnLoad` 用 `ResumeTask` 按剩余时间续排（读档时不足 1 秒按 1 秒），未运行期间不计时。[存档](../../../raw/current/scripts/prefabs/oceanvine.lua#L107-L121)、[读档](../../../raw/current/scripts/prefabs/oceanvine.lua#L123-L142)、[剩余秒数](../../../raw/current/scripts/entityscript.lua#L1576-L1586)
- **当前不可用**（四条，检索范围为全 `raw/current/scripts`）：①无 `SpawnPrefab("oceanvine_patch")`，`prefablist` 也无此名；②`spawnvines` 产出的 `hanging_vine`／`grabbing_vine` 在本构建不存在（只有本文件的字面量，无注册、无预载）；③`placegoffgrids` 的地块判定用 `WORLD_TILES.DEEPRAINFOREST`，该地块名在本构建没有注册（`tiledefs.lua` 的 `AddTile` 名单与全库检索都没有），判定恒假、`spawnitem` 永远返回 false；④`round` 被整段注释，筛附近 `hangingvine` 时仍调 `round(...)`，候选点 1 格内有现成藤条就会 `attempt to call a nil value`。`OnSave` 里的 `dumptable` 不算阻断：`main.lua` 无条件 `require("debugtools")`，它只会打印调试输出。[注释掉的 round](../../../raw/current/scripts/prefabs/oceanvine.lua#L16-L22)、[调用点](../../../raw/current/scripts/prefabs/oceanvine.lua#L51-L51)、[地块判定](../../../raw/current/scripts/prefabs/oceanvine.lua#L62-L69)、[地块注册表](../../../raw/current/scripts/tiledefs.lua#L96-L130)、[调试函数](../../../raw/current/scripts/debugtools.lua#L113-L113)、[加载点](../../../raw/current/scripts/main.lua#L214-L214)、[预载清单](../../../raw/current/scripts/prefablist.lua#L878-L880)
- 补丁被摧毁：它没有物理、没有 `burnable`、也没有记录已生藤条的反向引用（`plant.spawnpatch` 只写不读），所以「摧毁补丁」既不回滚已生成的藤条，也没有清理回调；`spawnedchildren` 只在存档里表示「首批已做过」。[反向引用](../../../raw/current/scripts/prefabs/oceanvine.lua#L64-L67)

### 同族排除

- `oceanvine_cocoon`：蛛茧，带 `webbed`、有生命与掉落、会放蜘蛛，和藤条只有「被采摘惊动」这一条关系；行为归其自身页面（Wiki 待建）。[注册](../../../raw/current/scripts/prefabs/oceanvine_cocoon.lua#L452-L453)
- `oceanvine_deco`：纯装饰藤，`NOBLOCK`＋`NOCLICK`＋`flying`、无 `pickable`／`inspectable`，只作巨树区域的景片；它反而启用了 15／25 的距离淡出，可与藤条本体的「注释停用」对照。[工厂](../../../raw/current/scripts/prefabs/oceanvine_deco.lua#L24-L46)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：pickable | 产物 `fig`、2.5 天再生、三个回调；`max_cycles=nil` 永不到达 `barren` | 主模拟 | [[reference/components/pickable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/oceanvine.lua#L305-L314) |
| 组件：inspectable | 摘空即移除、结果时重新加回 | 主模拟；客户端靠是否可交互判断 | [移除](../../../raw/current/scripts/prefabs/oceanvine.lua#L211-L213)、[加回](../../../raw/current/scripts/prefabs/oceanvine.lua#L234-L236) |
| 组件：burnable／propagator | 小燃可燃（改 7 秒）＋小传播；点火 2 秒后自毁并留 1 份 `ash` | 主模拟 | [[reference/components/burnable#关键接口]]、[[reference/components/propagator#关键接口]]；[接线](../../../raw/current/scripts/prefabs/oceanvine.lua#L316-L323) |
| 组件：hauntable | `MakeHauntableIgnite`：点火段被注释，只有冷却 | 主模拟 | [[reference/components/hauntable#关键接口]]；[工厂](../../../raw/current/scripts/standardcomponents.lua#L1173-L1189) |
| helper：`AddDynamicShadow`／`MakeSmallBurnable` | 阴影 1.5×0.75；燃烧符号 `swap_fire` | 阴影在公共端；可燃仅主模拟 | [阴影](../../../raw/current/scripts/prefabs/oceanvine.lua#L270-L281)、[helper](../../../raw/current/scripts/standardcomponents.lua#L164-L174) |
| API／数据：`placegoffgrids`／`fall`／`spawnNewVine` | 实例字段，供巨树与补丁调用；`spawnNewVine` 在本构建**无调用点** | 主模拟 | [暴露](../../../raw/current/scripts/prefabs/oceanvine.lua#L327-L329)、[补丁暴露](../../../raw/current/scripts/prefabs/oceanvine.lua#L152-L152) |
| API／数据：`OnLoadPostPass` | 读档后把动画帧随机化，避免整齐 | 主模拟 | [实现](../../../raw/current/scripts/prefabs/oceanvine.lua#L239-L241) |
| 关联 Prefab：`fig`／`fig_cooked` | 采摘产物与熟制产物 | 主模拟生成 | [[reference/prefabs/tropical-gathered-fruits#生熟两态数值]]；[产物](../../../raw/current/scripts/prefabs/oceanvine.lua#L312-L312) |
| 关联 Prefab：`ash` | 烧毁时的唯一掉落 | 主模拟生成；海面也生成 | [烧毁](../../../raw/current/scripts/prefabs/oceanvine.lua#L181-L185) |
| 关联 Prefab：`oceantree_pillar`／`watertree_pillar` | 唯一的常规生成者与「砍倒／撞船」调用方 | 主模拟 | [树柱](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L234-L262)、[撞船](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L238-L264) |
| 资源：anim | `anim/oceanvine.zip`（bank／build `oceanvine`：`idle_fruit`／`idle_nofruit`／`fruit_grow`／`harvest`／`spawn`／`fall_land`／`fall_ocean`／`burn`）＋小地图图 `oceanvine` | 静态声明 | [资源](../../../raw/current/scripts/prefabs/oceanvine.lua#L2-L6) |

## 典型接入

### 让藤条采完 0.5 天后结果（缩短 2.5 天再生）

- 前提与选择：服务端 `modmain.lua`，只在构造后改 `pickable.baseregentime`。`Pick` 每次都拿当前 `baseregentime` 重算 `regentime` 再起任务，所以改完字段，之后每一次采摘都按新时长再生，不需要再调 `Regen()`。[重算](../../../raw/current/scripts/components/pickable.lua#L559-L573)、[补满路径](../../../raw/current/scripts/components/pickable.lua#L386-L396)
- 生效与撤销：postinit 每次构造运行；停用模组后新构造的实例恢复 2.5 天，已排定的再生任务不会被回溯重排。春季仍会再乘 0.75。
- 构造期补不出「这一次」：`SetUp` 刚跑完的藤条本来就是 `canbepicked = true`，postinit 里 `if not pickable:CanBePicked() then pickable:Regen() end` 这种分支在构造期恒不成立；旧档的「已采空＋剩余秒数」是 `SpawnSaveRecord` 先构造 prefab、再由 `pickable:OnLoad` 按 `data.picked`／`data.time` 调 `makeemptyfn` 并续排剩余任务才恢复的，那时 postinit 早已跑完。[构造](../../../raw/current/scripts/prefabs/oceanvine.lua#L305-L314)、[读档](../../../raw/current/scripts/components/pickable.lua#L327-L366)、[读档顺序](../../../raw/current/scripts/mainfunctions.lua#L464-L503)

```lua
-- modmain.lua：之后每次采摘按 0.5 天再生（已有实例读档时沿用存档剩余秒数，下一次采摘才走新时长）
AddPrefabPostInit("oceanvine", function(inst)
    if not GLOBAL.TheWorld.ismastersim or inst.components.pickable == nil then
        return
    end
    inst.components.pickable.baseregentime = GLOBAL.TUNING.TOTAL_DAY_TIME * 0.5
end)
```

- 要「采完立刻结果」须让补满动作落在采摘之后：把再生时长压到 0（`pickable.baseregentime = 0`），`Pick` 照常排一个 0 秒的再生任务，任务到点走 `Regen` → `makefullfn`，`fig` 与 `inspectable` 一起恢复、`canbepicked` 置回 true；也可以在 `onpickedfn` 之后再排一次 `Regen`。与构造期调用 `Regen()` 的区别在时序：前者发生在 `Pick` 已把 `canbepicked` 置 false 之后，对应的才是「刚采空」这一刻；后者只在实例刚构造、本来就满的时候跑，对「这一次」没有效果。[Pick 排程](../../../raw/current/scripts/components/pickable.lua#L557-L573)、[补满](../../../raw/current/scripts/prefabs/oceanvine.lua#L225-L237)
- 调整与验证：单位是秒（`TOTAL_DAY_TIME`＝480）；春季会再乘 0.75，示例未关掉这条修正。`Regen` 不重置 `cycles_left`，但藤条的 `cycles_left` 本来就不参与扣减。未运行游戏验证。

## 常见陷阱与限制

- **误区：藤条会有一圈「远处淡出」**：`distancefade` 的 `Setup(15,25)` 在本文件里整段处于 Lua 块注释内（`--` 加双方括号包裹），实际不生效；同族的巨树、装饰藤、蛛茧仍各自启用。[注释段](../../../raw/current/scripts/prefabs/oceanvine.lua#L288-L293)、[对照](../../../raw/current/scripts/prefabs/oceanvine_deco.lua#L43-L46)
- **边界：`oceanvine_patch` 现在什么都生成不了**：没有生成点、目标 prefab 不存在、地块键未注册、`round` 被注释；把它当作「可用机制」会写出跑不通的接入（四条证据见[生成来源与补丁](#生成来源与补丁)）。
- **边界：烧掉只留 1 份灰，海里也留**：烧毁逻辑自己 `SpawnPrefab("ash")`，不走 `DefaultBurntFn` 的海面判断，也不看堆叠。[烧毁](../../../raw/current/scripts/prefabs/oceanvine.lua#L181-L185)、[默认实现](../../../raw/current/scripts/standardcomponents.lua#L33-L43)
- **边界：2 秒内扑灭能救回**：`OnIgnite` 只是排任务，`OnExtinguishNotFinishedBurning` 会取消它；超过 2 秒进入 `OnStartBurnAnim` 后 `persists=false`，救不回来。[点火](../../../raw/current/scripts/prefabs/oceanvine.lua#L195-L198)、[取消](../../../raw/current/scripts/prefabs/oceanvine.lua#L188-L193)
- **边界：再生不走 `onregenfn`**：本 prefab 只设了 `makefullfn`，改 `onregenfn` 对藤条没有任何效果；同理 `makeemptyfn` 才是摘空与读档的统一入口。[回调集](../../../raw/current/scripts/prefabs/oceanvine.lua#L307-L314)
- **未核验**：`hangingvine` tag 的其它消费者、`oceanvine_patch` 在本构建之前的版本是否可用、蛛茧被惊动后的实际数量与延迟均未运行验证。

## 验证记录

- 源码审阅：2026-11-08，Steam buildid 26474873（2026-11-09 复检补藤阈值与读档顺序下的示例）。核对 `oceanvine.lua` 全文（两个工厂、五个标签、距离淡出注释、pickable 三回调、燃烧三段、`fall` 与 `fall_down_fn`、补丁的首批／计时／存读档）、`oceantree_pillar.lua` 的补藤与 `OnRemoveEntity` 放倒、`watertree_pillar.lua` 的撞船补藤、`oceanvine_deco.lua` 与 `oceanvine_cocoon.lua` 的归属判断、`prefablist` 的预载项、`pickable` 的 `SetUp`／`Pick`／`Regen`／`OnLoad`、`map` 的放置忽略表与 `componentutil` 的推物忽略表、`SGwilson` 的高处长动作、`simutil` 的春季修正、`tuning` 的相关常量、`debugtools` 的 `dumptable`。
- 名称核验：`STRINGS.NAMES.OCEANVINE`＝苔藓藤条；`oceanvine_patch` 无名称键（无 po 条目，非玩家可见对象）。[名称键](../../../raw/current/scripts/languages/chinese_s.po#L346580-L346582)
- 未覆盖：`fig`／`fig_cooked` 的数值与去向（归 [[reference/prefabs/tropical-gathered-fruits]]）；蛛茧与装饰藤的完整机制；巨树本身的砍伐流程。
- 游戏运行：未运行。2.5 天、2～3 天与 2 秒燃烧均为源码推导。
- 复审直修（2026-11-08）：补 `WORLD_TILES.DEEPRAINFOREST` 未注册这一独立阻断、`watertree_pillar` 补藤阈值与 `MakeEmpty` 差异、`dumptable` 可由 `main.lua` 无条件加载、离线不计时、`fall` 的巨树避让、阴影／可燃端归属；未运行游戏。
- 结构检查：`validate_page.py`（`--wiki-root` 指向 `K/wiki`）通过（standard 档内）；`validate_source_links.py` 0 错 0 警。

## 相关链接

- 产物：[[reference/prefabs/tropical-gathered-fruits]]
- 采摘与再生：[[reference/components/pickable]]、[[reference/components/regrowthmanager]]
- 世界生成：[[reference/components/regrowthmanager]]、[[reference/prefabs/index|Prefab 索引]]

返回 [[reference/prefabs/index|Prefab 索引]]。
