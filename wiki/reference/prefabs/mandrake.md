---
title: "曼德拉草与熟曼德拉草（mandrake、cookedmandrake）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "prefabs/mandrake_inactive.lua 注册的两件物品：生件可烤成熟件，两者吃下都会在 0.5 秒后范围催眠（生 15、熟 25，世界坐标单位），生件烹饪时以 `chef or cooker` 为中心发出催眠判定。"
aliases: ["mandrake", "cookedmandrake", "曼德拉草", "熟曼德拉草", "烤曼德拉草", "曼德拉草催眠"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/mandrake_inactive.lua", "prefabs/mandrake_active.lua", "prefabs/mandrake_planted.lua", "map/rooms/forest/mandrake.lua", "tuning.lua", "components/edible.lua", "components/cookable.lua", "components/stackable.lua", "components/tradable.lua", "standardcomponents.lua", "cooking.lua", "preparedfoods.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 曼德拉草与熟曼德拉草（mandrake、cookedmandrake）

## 快速判断

- 职责与适用：文件叫 `prefabs/mandrake_inactive.lua`，但 `Prefab(...)` 的注册名是 `mandrake`（生）与 `cookedmandrake`（熟），两件共用 `commonfn(anim, cookable)`；生件多一套 `cookable` 与 `SetOnPutInInventoryFn`。[注册](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L149-L150)、[共用工厂](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L69-L117)
- 优先使用：改"吃下催眠"看[吃下与烹饪的催眠](#吃下与烹饪的催眠)；改三维与堆叠看[数值与部件](#数值与部件)；改烤制产物看 `cookable` 那一行。
- 重要副作用：**烹饪生曼德拉草也会催眠**——`cookable:SetOnCookedFn(oncooked)` 以 `chef or cooker` 为中心按 `MANDRAKE_SLEEP_RANGE_COOKED`＝25（世界坐标单位）催眠；吃生件 15、吃熟件 25。三者都用同一个 `MANDRAKE_SLEEP_TIME`＝10 秒（每目标再加 0–1 秒随机）。[吃生](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L47-L52)、[烹饪](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L54-L60)、[吃熟](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L62-L67)、[常量](../../../raw/current/scripts/tuning.lua#L1935-L1937)
- 修改边界：`mandrake_active`／`mandrake_planted` 是另外两个 Prefab（地上的活体与植株），只在本页[形态链](#形态链谁生成谁)写关系与行号，行为归各自源码；曼德拉草汤归 `cooking.lua`／`preparedfoods.lua`。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `mandrake` | `rawfn()` → `commonfn("object", true)` → 主模拟生件段 | 生件：pristine 就带 `cookable` 标签，主模拟挂 `cookable`（`product = "cookedmandrake"`、`SetOnCookedFn(oncooked)`）；`healthvalue = HEALING_HUGE`＝60、`hungervalue = CALORIES_HUGE`＝75；吃回调 `oneaten_raw`；入包回调重播 `object` 动画。[rawfn](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L119-L133) |
| `cookedmandrake` | `cookedfn()` → `commonfn("cooked")`（**不传** `cookable`） | 熟件：没有 `cookable` 标签也没有同名组件，不能再烤；`healthvalue = HEALING_SUPERHUGE`＝100、`hungervalue = CALORIES_SUPERHUGE`＝150；吃回调 `oneaten_cooked`。[cookedfn](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L135-L147) |

共用 pristine（两端）：bank／build 都是 `mandrake`，`PlayAnimation(anim)` 并写 `inst.scrapbook_anim = anim`；只在 `cookable` 为真时加 `cookable` 标签；`MakeInventoryFloatable(inst, "med", 0.2, 0.8)`；`SetPristine`。[pristine 段](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L78-L90)

共用主模拟（仅服务端）：`inspectable` → `inventoryitem` → `stackable`（40）→ `edible`（`foodtype = FOODTYPE.VEGGIE`）→ `tradable` → 可选 `cookable` → `MakeSmallBurnable`（不传时长，取默认 10 秒）+ `MakeSmallPropagator`。**没有** `perishable`、**没有** `fuel`、**没有** `deployable`，也没有 `getstatus`。[主模拟段](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L96-L116)、[燃烧默认](../../../raw/current/scripts/standardcomponents.lua#L164-L174)、[传播者](../../../raw/current/scripts/standardcomponents.lua#L200-L213)

### 名称核验

`NAMES.MANDRAKE`＝**曼德拉草**、`NAMES.COOKEDMANDRAKE`＝**熟曼德拉草**，注册名与名称键同名；`NAMES.MANDRAKE_ACTIVE` 与 `NAMES.MANDRAKE_PLANTED` 的译文也是"曼德拉草"，那是另外两个实体的名字，不是本页两件的别名。[生](../../../raw/current/scripts/languages/chinese_s.po#L344984-L344987)、[熟](../../../raw/current/scripts/languages/chinese_s.po#L341629-L341632)、[活体](../../../raw/current/scripts/languages/chinese_s.po#L344994-L344997)

## 吃下与烹饪的催眠

| 触发 | 中心 | 半径 | 时长 | 证据 |
|---|---|---|---|---|
| 吃 `mandrake` | 吃的人 `eater` | `MANDRAKE_SLEEP_RANGE`＝15 | `MANDRAKE_SLEEP_TIME`＝10 + 随机 | [oneaten_raw](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L47-L52) |
| 吃 `cookedmandrake` | 吃的人 `eater` | `MANDRAKE_SLEEP_RANGE_COOKED`＝25 | 10 + 随机 | [oneaten_cooked](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L62-L67) |
| 烤 `mandrake`（在火／炉上） | `chef or cooker` | 25 | 10 + 随机 | [oncooked](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L54-L60) |

三者都先播 `dontstarve/creatures/mandrake/death`，再 `DoTaskInTime(0.5, ...)` 才结算范围——0.5 秒是**发出催眠判定的延迟**，不是睡眠时长；10 秒是传给目标的参数（每目标再加 `math.random()` 得 10–11 秒），玩家实际躺多久还要过 `yawn` 的状态门禁。

- 选目标：`FindEntities(x, y, z, range, nil, SLEEPTARGETS_CANT_TAGS, SLEEPTARGETS_ONEOF_TAGS)`，排除 `playerghost`／`FX`／`DECOR`／`INLIMBO`，且目标必须带 `sleeper` 或 `player` 标签之一。[查找](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L18-L23)
- 逐目标门禁：`(v == inst or canpvp or not v:HasTag("player"))` 且目标未 `freezable:IsFrozen()`、未 `pinnable:IsStuck()`、未化石化。`canpvp = not inst:HasTag("player") or TheNet:GetPVPEnabled()`，而这里 `inst` 就是被传进来的 `eater`／`chef`：**自己是玩家时，对其他玩家只在 PVP 服生效；自己不是玩家时对所有人生效**。[门禁](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L24-L29)
- 结算分支：有坐骑先给坐骑 `ridersleep`（`sleepiness = 7`）；玩家推 `yawn`（`grogginess = 4`、`knockoutduration = time + math.random()`）——但 `yawn` 在玩家端还要过门禁：已死、正在睡、被冻、被钉住时**整条跳过**，不会进 `knockout`；有 `sleeper` 的 `AddSleepiness(7, time + math.random())`；有 `grogginess` 的 `AddGrogginess(4, ...)`；其余推 `knockedout`。[分支](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L30-L43)、[yawn 门禁](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2331-L2344)
- 推导：`time + math.random()` 使**传给目标**的 `knockoutduration`／`sleeptime` 落在 10–11 秒，这是参数值而不是实测睡眠长度；半径 15／25 是以吃的人为圆心的**世界坐标单位**，不带视线判定（源码里没有 `CanEntitySeeTarget` 之类的过滤）。
- `mandrake_active` 烧死时也会生成 `cookedmandrake` 并催眠，但它调的是**生件**的 `MANDRAKE_SLEEP_RANGE`（15），与上表第三行不同源，不能互相套用。[烧死](../../../raw/current/scripts/prefabs/mandrake_active.lua#L94-L104)

## 数值与部件

| 注册 | 饥饿／生命／理智 | 堆叠 | 食物类型 | 其它 |
|---|---|---|---|---|
| `mandrake` | 75／60／0 | 40（`STACK_SIZE_SMALLITEM`） | `VEGGIE` | `cookable` → `cookedmandrake`；`tradable`（`goldvalue` 默认 0） |
| `cookedmandrake` | 150／100／0 | 40 | `VEGGIE` | 无 `cookable`；仍是 `tradable` |

数值来源：[生件赋值](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L126-L128)、[熟件赋值](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L142-L144)、[治疗档](../../../raw/current/scripts/tuning.lua#L2028-L2030)、[热量档](../../../raw/current/scripts/tuning.lua#L2063-L2065)、[堆叠](../../../raw/current/scripts/tuning.lua#L83-L83)。采集册同项写作 `hungervalue=75/150`、`healthvalue=60/100`、`stacksize=40`、`foodtype=VEGGIE`，与源码一致：生件条目在 L823、熟件条目在 L344（L824 是另一个实体 `mandrake_active`）。[生件条](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L823)、[熟件条](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L344)

- 燃料与腐烂：两件都只有 `burnable`＋`propagator`（可被点燃、火会蔓延），**没有 `fuel` 组件**，不能当燃料塞进火堆；也**没有 `perishable`**，不会腐坏成腐烂物。[主模拟段](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L113-L114)
- 生件的入包动画：`inventoryitem:SetOnPutInInventoryFn(onputininventory)` 在进背包／手上时重播 `object`，所以地上被烧过、被吃过之后回到包里仍回到"草根"外观。[回调](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L14-L16)、[接线](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L130-L130)
- 食材标签：`cooking.lua` 只给 `mandrake` 登记 `{veggie=1, magic=1}`，第三参 `cancook` 派生的是 `mandrake_cooked` 这个并不存在的名字；实际熟件叫 `cookedmandrake`，既不在 `ingredients` 表里也没有别名，所以 **`IsCookingIngredient` 判假——烹饪锅的容器过滤直接拒收熟件，它根本放不进锅**。曼德拉草汤的 `test` 也只认 `names.mandrake`。[登记](../../../raw/current/scripts/cooking.lua#L46-L52)、[食材值](../../../raw/current/scripts/cooking.lua#L127-L127)、[查表](../../../raw/current/scripts/cooking.lua#L185-L187)、[锅过滤](../../../raw/current/scripts/containers.lua#L373-L375)、[汤](../../../raw/current/scripts/preparedfoods.lua#L159-L161)

## 形态链：谁生成谁

| 起点 | 事件 | 结果 | 证据 |
|---|---|---|---|
| `mandrake_planted`（世界生成的植株） | 被摘 `pickable.onpickedfn` | `SpawnPrefab("mandrake_active")` 到同位置并 `active:onpicked(picker)`，原植株 `remove_when_picked = true` 自删 | [采摘](../../../raw/current/scripts/prefabs/mandrake_planted.lua#L26-L33)、[配置](../../../raw/current/scripts/prefabs/mandrake_planted.lua#L58-L62) |
| `mandrake_active`（活体，20 血） | 天亮 `startday` | `SpawnPrefab("mandrake_planted")` + `planted:replant(inst)`，活体 `Remove()`；不能种时每 1 秒重试，累计超过 5 次仍未成功就改按白天判定处理 | [重植](../../../raw/current/scripts/prefabs/mandrake_active.lua#L62-L82)、[监听](../../../raw/current/scripts/prefabs/mandrake_active.lua#L188-L188) |
| `mandrake_active` | 白天被摘后 26 帧 `CheckDay` 判定为白天 → `health:Kill()` | `ondeath` 生成 `mandrake`（生件）并播完 `death` 尾帧，活体自删 | [死亡](../../../raw/current/scripts/prefabs/mandrake_active.lua#L23-L30)、[ondeath](../../../raw/current/scripts/prefabs/mandrake_active.lua#L84-L92)、[摘后判定](../../../raw/current/scripts/prefabs/mandrake_active.lua#L126-L134) |
| `mandrake_active` | 带燃烧死亡 → `oncooked` | 进 `death` 时若仍在烧，先 `Extinguish` 并设 `21 * FRAMES` 超时，超时才走 `oncooked`（否则死亡动画结束时走 `ondeath` 掉生件）；`oncooked` 生成 `cookedmandrake` 抛向自己，再等 0.5 秒按**生件**范围催眠 | [death 状态](../../../raw/current/scripts/stategraphs/SGmandrake.lua#L47-L69)、[oncooked](../../../raw/current/scripts/prefabs/mandrake_active.lua#L94-L104) |

三条链全部是 `SpawnPrefab` **新建实体**再删旧实体，没有就地改字段的形态切换；要在模组里缓存实体引用会失效。植株的生成入口是房间定义 `MandrakeHome` 的 `countprefabs = { mandrake_planted = 1 }`；`mandrake_planted` 还登记了 regrowth，被烧后可以重新长回。[房间](../../../raw/current/scripts/map/rooms/forest/mandrake.lua#L2-L17)、[regrowth](../../../raw/current/scripts/prefabs/mandrake_planted.lua#L66-L71)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`edible` | `foodtype = VEGGIE`、三维、`SetOnEatenFn`；两注册各写一组 | 仅主模拟 | [[reference/components/edible]]；[生](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L102-L103) |
| 组件：`cookable` | `product = "cookedmandrake"`、`SetOnCookedFn(oncooked)` | 仅生件；标签在 pristine、组件在主模拟 | [[reference/components/cookable]]；[接线](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L107-L111) |
| 组件：`stackable`／`inventoryitem`／`tradable` | 40 一叠、可拾取、`goldvalue` 默认 0 | 仅主模拟 | [[reference/components/stackable]]、[[reference/components/tradable]]；[挂件](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L97-L105) |
| 组件：`burnable`／`propagator` | 可点燃、火会蔓延；无 `fuel` | 主模拟；`MakeSmallBurnable` 默认 10 秒 | [[reference/components/burnable]]；[挂载](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L113-L114) |
| 组件：`sleeper`／`grogginess`／`rider` | 催眠的三个目标分派：`AddSleepiness`／`AddGrogginess`／`ridersleep` | 服务端结算，按目标已有组件分流 | [[reference/components/sleeper]]、[[reference/components/grogginess]]、[[reference/components/rider]]；[分支](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L30-L43) |
| 门禁：`freezable`／`pinnable`／`fossilizable` | 被冻、被钉住、被石化者跳过催眠 | 三个 `IsFrozen`／`IsStuck`／`IsFossilized` 判定 | [[reference/components/freezable]]、[[reference/components/pinnable]]；fossilizable 见源码（Wiki 待建）[跳过](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L27-L29) |
| 关联 Prefab：`mandrake_active` | 活体形态；被摘后由植株生成 | `mandrake_inactive` 的 `prefabs` 表只声明依赖，不生成 | [声明](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L9-L12)、[生成](../../../raw/current/scripts/prefabs/mandrake_planted.lua#L30-L32) |
| 关联 Prefab：`mandrake_planted` | 植株形态；世界生成与 regrowth 的落点 | 房间 `MandrakeHome` 每间 1 株 | [房间](../../../raw/current/scripts/map/rooms/forest/mandrake.lua#L15-L17) |
| SG／Brain：`SGMandrake`／`mandrakebrain` | 活体的摘取、跟随、重植状态 | 只挂在 `mandrake_active` | [安装](../../../raw/current/scripts/prefabs/mandrake_active.lua#L180-L183) |
| 资源：`anim/mandrake.zip` | 两件物品与活体、植株共用同一 bank／build | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L4-L7) |

## 典型接入

### 缩小生件的催眠半径

- 前提与选择：服务端 `modmain.lua` 直接改 `TUNING` 常量。`doareasleep` 在**被调用时**才读 `TUNING.MANDRAKE_SLEEP_RANGE`（闭包里没有提前缓存），所以改表即刻生效，不必给两个 prefab 各写一份 postinit。
- 生效与撤销：影响共用 15 格的那**两条**路径：吃生件、以及 `mandrake_active` 烧死掉落时的催眠；吃熟件与烹饪走 `MANDRAKE_SLEEP_RANGE_COOKED`＝25，不受影响。
- 原版依据：见[吃下与烹饪的催眠](#吃下与烹饪的催眠)。

```lua
-- modmain.lua：生曼德拉草的催眠半径 15 → 8，其余不变
GLOBAL.TUNING.MANDRAKE_SLEEP_RANGE = 8
```

- 调整与验证：`MANDRAKE_SLEEP_TIME` 仍是 10 秒。15／25 是**世界坐标单位**，量距离要读坐标而不是目测格数；观测点要分开记：**0.5 秒时是否发出判定**（看目标有没有收到 `yawn`／`AddSleepiness`）、**再之后是否真倒地**（`yawn` 进 `knockout` 才睡）。样本三组：①PVP 开启时，8 单位内／外的第二名玩家这两个观测点分别如何；②带 `sleeper` 的普通目标在 8 单位内外是否被 `AddSleepiness`；③一个被冻住或被钉住的目标，确认它连判定都不进（外层 `FindEntities` 之后还有逐目标门禁）。

## 常见陷阱与限制

- **误区：文件名叫 `mandrake_inactive`，注册名就叫 mandrake_inactive。** 注册名是 `mandrake` 与 `cookedmandrake`；`mandrake_active`／`mandrake_planted` 是另外两个 prefab 文件，界面名都叫"曼德拉草"，但行为完全不同。[注册](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L149-L150)
- **误区：只有吃才会催眠。** 在火／炉上烤生件时 `oncooked` 同样催眠，而且半径更大（25，世界坐标单位）。被烧死的 `mandrake_active` 生成的熟件也会催眠，但用的是 15（同单位）。[烹饪](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L54-L60)、[烧死](../../../raw/current/scripts/prefabs/mandrake_active.lua#L94-L104)
- **边界：熟件不能再烤，也放不进烹饪锅。** `cookedfn` 不传 `cookable`；`cooking.lua` 的 `cancook` 派生名是 `mandrake_cooked`，与实际 `cookedmandrake` 不匹配，`IsCookingIngredient` 查表落空，锅的容器过滤直接拒收。[熟件](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L135-L147)、[派生](../../../raw/current/scripts/cooking.lua#L46-L52)、[查表](../../../raw/current/scripts/cooking.lua#L185-L187)、[锅过滤](../../../raw/current/scripts/containers.lua#L373-L375)
- **边界：其他玩家只在 PVP 服被波及。** `canpvp` 由"被传进来的中心实体是不是玩家"决定；玩家吃、非 PVP 服时，其他玩家被跳过；**自己要先通过外层筛选才会收到 `yawn`**，是否倒地再看状态机门禁（已死、正在睡、被冻、被钉住时停在原地）；冻结／被钉／石化的目标连外层筛选都过不了，根本不会收到事件。[查找与门禁](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L18-L29)
- **边界：被冻／被钉／被石化的目标免疫。** 三个判定任一为真就整条跳过，不会"解冻再睡"。[跳过](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L27-L29)
- **边界：不能当燃料、不会腐烂。** 只有 `burnable`，没有 `fuel` 与 `perishable`；放进火堆不会变成燃料，放到天亮也不会变腐烂物。[主模拟段](../../../raw/current/scripts/prefabs/mandrake_inactive.lua#L96-L116)
- **未核验**：未运行游戏。`time + math.random()` 的实现、以及"26 帧后才判白天"的时序均为源码推导（26 帧按 30 fps 约 0.87 秒）。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/mandrake_inactive.lua` 全文（两注册、共用 `commonfn`、三条催眠入口、`doareasleep`）；`prefabs/mandrake_active.lua` 与 `prefabs/mandrake_planted.lua` 全文（形态链四个转换点）；`map/rooms/forest/mandrake.lua` 的生成入口；`tuning.lua` 的 `MANDRAKE_SLEEP_*`／`HEALING_*`／`CALORIES_*`／`STACK_SIZE_SMALLITEM`；`standardcomponents` 的 `MakeSmallBurnable`／`MakeSmallPropagator`／`MakeInventoryFloatable`；`cooking.lua` 的 `AddIngredientValues` 与曼德拉草行、`preparedfoods.lua` 的曼德拉草汤；`chinese_s.po` 名称；`scrapbookdata` 交叉核对。
- 修订记录：2026-11-09 第 1 轮独立检视回源复核后按点名条目修正——P2-1 熟件改写为“放不进锅”（L67、L114）、P2-2 采集册锚点拆为 L823／L344（L63）、P2-3“三条路径”改为两条（L100）。外部复检轮再改：P2 明确 0.5 秒是发出判定的延迟、10–11 秒是传给目标的参数（L48、L53）、补 `yawn` 的玩家端门禁并把“自己一定中招”收窄（L52、L115）、补 `mandrake_active` 燃烧死亡经 `death` 状态 `21 * FRAMES` 才转熟品（L76）、示例改为世界坐标单位与两组样本（L108）。D 组第二轮再改：摘要“连厨师一起睡”改为“以厨师为中心发出催眠判定”（L6）、示例拆“0.5 秒是否发出判定／之后是否倒地”两个观测点并补被冻被钉样本（L108）、陷阱句改为“事件必发但倒地看门禁”（L115）。第三轮再改：摘要、快速判断与陷阱的 15／25 统一标注“世界坐标单位”（L6、L22、L113），L115 把外层排除的锚点扩到 L18-L29 并点明“连判定都不进”。第四轮再改：L115 消除“事件必发”与“外层已筛掉”的矛盾，改为“先通过外层筛选才会收到 `yawn`，倒地再看状态机门禁”；其余段落未动。
- 未覆盖：`SGMandrake` 的状态细节（只确认 `SetStateGraph` 安装）、`mandrakebrain` 的跟随决策、以及 `klaussackloot` 里出现 `mandrake` 的那条战利品配置。
- 游戏运行：未运行。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 见交付回报。

## 相关链接

- 物品组件：[[reference/components/edible]]、[[reference/components/cookable]]、[[reference/components/stackable]]、[[reference/components/tradable]]
- 催眠目标：[[reference/components/sleeper]]、[[reference/components/grogginess]]、[[reference/components/rider]]
- 形态：[[reference/components/pickable]]、[[reference/components/freezable]]、[[reference/components/pinnable]]
- 燃烧：[[reference/components/burnable]]
- 同批可食物品：[[reference/prefabs/bird-egg]]

返回 [[reference/prefabs/special-food-index|节庆、角色与特殊食品索引]]。
