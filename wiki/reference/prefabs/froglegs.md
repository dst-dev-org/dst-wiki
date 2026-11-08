---
title: "蛙腿与熟蛙腿（froglegs、froglegs_cooked）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "prefabs/froglegs.lua 的共用 commonfn 生成的蛙腿与熟蛙腿：标签、三维、鲜度、堆叠与交易值，干成小肉干、烤成熟蛙腿两条转换，以及青蛙／月蛙战利品与浣猫礼物来源。"
aliases: ["froglegs", "froglegs_cooked", "蛙腿", "熟蛙腿", "青蛙腿"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/froglegs.lua", "prefabs/frog.lua", "prefabs/dragonfurnace.lua", "prefabs/catcoon.lua", "prefabs/meats.lua", "prefablist.lua", "cooking.lua", "tuning.lua", "components/edible.lua", "components/perishable.lua", "components/stackable.lua", "components/dryable.lua", "components/cookable.lua", "components/tradable.lua", "components/floater.lua", "components/bait.lua", "components/dryingrack.lua", "components/lootdropper.lua", "containers.lua", "componentactions.lua", "standardcomponents.lua", "prefabs/pigking.lua", "actions.lua", "prefabs/meatrack2.lua", "scrapbook_prefabs.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 蛙腿与熟蛙腿（froglegs、froglegs_cooked）

## 快速判断

- 职责与适用：两个注册都由 `prefabs/froglegs.lua` 的局部 `commonfn(anim, dryable, cookable)` 生成。生蛙腿带 `smallmeat`／`catfood`／`rawmeat` 标签，能上晾肉架干成 `smallmeat_dried`、能在带 `cooker` 的实体上烤成 `froglegs_cooked`；熟蛙腿把后置参数全缺省，既没有 `dryable`／`cookable` 标签，也没有同名组件。[生蛙腿](../../../raw/current/scripts/prefabs/froglegs.lua#L84-L97)、[熟蛙腿](../../../raw/current/scripts/prefabs/froglegs.lua#L99-L111)
- 优先使用：改三维、鲜度或堆叠看[数值与获得](#数值与获得)；改干制产物看[消费端](#消费端)的 `dryable` 行；改掉落看[来源](#来源)。
- 重要副作用：干制不是就地改名——到点后原物 `Remove`、另生成产物并只继承湿度（[完成回调](../../../raw/current/scripts/components/dryingrack.lua#L198-L224)）；烤制把新鲜度折成 `1-(1-p)*0.5` 给产物（[Cook](../../../raw/current/scripts/components/cookable.lua#L18-L37)）；生蛙腿理智 −10，赋值即带 `badfood`，默认不能喂给其他玩家；例外是 PVP 开启、或目标带 `possessedbody` 时**跳过食品安全检查**——两者都不再管 `badfood`，但目标仍须处于可喂状态且食性接受该食物（[门槛](../../../raw/current/scripts/actions.lua#L2397-L2410)、[possessedbody 分支](../../../raw/current/scripts/componentactions.lua#L1172-L1173)）。
- 修改边界：只覆盖这两个注册。青蛙／月蛙本体与晾肉架建筑本批未建（源码入口 `prefabs/frog.lua`、`prefabs/meatrack2.lua`），`smallmeat_dried` 三态见 [[reference/prefabs/smallmeat]]；干制成 `fishmeat_small_dried` 的旧鱼链见 [[reference/prefabs/pond-fish]]；`quagmire` 等其它模式未查。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `froglegs` | `defaultfn()` 调 `commonfn("idle", true, true)`：共用段做动画、标签、漂浮与 `SetPristine`，非主模拟在 `SetPristine` 之后直接返回，主模拟段才挂组件 | 生腿：共用段加三个标签 `smallmeat`／`catfood`／`rawmeat`，因两个后置参数为真再补 `dryable`／`cookable` 标签；主模拟挂 `edible`（`FOODTYPE.MEAT`，**未写** `ismeat`）、`perishable`、`dryable`、`cookable`、`stackable`、`bait`、`inspectable`、`inventoryitem`、`MakeHauntableLaunchAndPerish`、`tradable`（`goldvalue = 0`）。[工厂](../../../raw/current/scripts/prefabs/froglegs.lua#L11-L82) |
| `froglegs_cooked` | `cookedfn()` 调 `commonfn("cooked")`，后两参缺省 | 只保留 `smallmeat`／`catfood`／`rawmeat` 三个标签：不能再烤、不能上架；`sanityvalue` 未赋值，沿用 `edible` 默认 0。[熟态](../../../raw/current/scripts/prefabs/froglegs.lua#L99-L111)、[默认三维](../../../raw/current/scripts/components/edible.lua#L19-L44) |

`commonfn` 的固定顺序：`AddTransform`／`AddAnimState`／`AddNetwork` → `MakeInventoryPhysics` → bank／build 都设 `frog_legs` → 三个标签 → 条件标签 → `MakeInventoryFloatable(inst)`（默认 `small`、零偏移、缩放 1）→ `SetPristine` → 主模拟挂件。两个注册都不改 `floater` 参数，所以两态漂浮外观一致。[共用段](../../../raw/current/scripts/prefabs/froglegs.lua#L11-L46)、[漂浮默认](../../../raw/current/scripts/standardcomponents.lua#L1772-L1791)

`dryable`／`cookable` 的标签在 `SetPristine` **之前**加，组件只在主模拟分支里加：客户端只有标签、没有组件。动作判定本身也只查标签——肉架槽位测试看 `dryable`，烤制动作处理器看物品的 `cookable` 与目标上的 `cooker` 标签，另有三道附加门禁：目标未带 `fueldepleted`；目标是 `dangerouscooker` 时须 `doer:HasTag("expertchef")`；骑乘时只有烹饪器属于自己才可烤（本页走食材对烹饪器的 `cookable` 分支，骑乘门禁看的是目标；另一条 `cooker` 分支才检查食材归属）。[标签段](../../../raw/current/scripts/prefabs/froglegs.lua#L30-L38)、[肉架](../../../raw/current/scripts/containers.lua#L234-L244)、[`cookable` 分支](../../../raw/current/scripts/componentactions.lua#L1096-L1105)、[`cooker` 分支](../../../raw/current/scripts/componentactions.lua#L1113-L1126)

### 名称核验

`froglegs` 的简中实显是**蛙腿**（Frog Legs），`froglegs_cooked` 是**熟蛙腿**（Cooked Frog Legs）；两个键各命中一次，注册名与 `STRINGS.NAMES` 键同名。[FROGLEGS](../../../raw/current/scripts/languages/chinese_s.po#L343020-L343022)、[FROGLEGS_COOKED](../../../raw/current/scripts/languages/chinese_s.po#L343025-L343027)

## 数值与获得

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒、`perish_warp`＝1，所以鲜度按 480 秒／天换算。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[`perish_warp`](../../../raw/current/scripts/tuning.lua#L47)

| 注册 | 饥饿／生命／理智 | 鲜度 | 堆叠 | 漂浮 | 转换 |
|---|---|---|---|---|---|
| `froglegs` | 12.5／0／−10 | `PERISH_FAST`＝6 天＝2880 秒 | 40 | `small`，零偏移（两端同参） | 干制 `DRY_FAST`＝1 天＝480 秒 → `smallmeat_dried`；烤制 → `froglegs_cooked` |
| `froglegs_cooked` | 12.5／1／0 | `PERISH_MED`＝10 天＝4800 秒 | 40 | `small`，零偏移 | 无（无 `dryable`／`cookable`） |

数值来源：[生腿](../../../raw/current/scripts/prefabs/froglegs.lua#L91-L94)、[熟态](../../../raw/current/scripts/prefabs/froglegs.lua#L106-L108)、[常量](../../../raw/current/scripts/tuning.lua#L2021-L2038)、[热量](../../../raw/current/scripts/tuning.lua#L2058-L2065)、[鲜度](../../../raw/current/scripts/tuning.lua#L2043-L2051)、[干制](../../../raw/current/scripts/tuning.lua#L2053-L2056)、[堆叠](../../../raw/current/scripts/tuning.lua#L82-L83)。采集册同项写作 `stacksize=40`、`hungervalue=12.5`、`healthvalue=0/1`、`sanityvalue=-10/0`、`perishable=2880/4800`，与源码一致。[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L555-L556)

### 来源

| 注册 | 已核验来源 | 产出与证据 |
|---|---|---|
| `froglegs` | ①普通青蛙（`frog`）战利品固定一份；②月蛙（`lunarfrog`）在基表之外按 `LUNARFROG_ADDITIONAL_LOOT_CHANCE`＝0.5 追加一份；③浣猫（`catcoon`）好感礼物的 tier 5 食物池收录 | ①[掉落表](../../../raw/current/scripts/prefabs/frog.lua#L180)、②[追加](../../../raw/current/scripts/prefabs/frog.lua#L247)、[常量](../../../raw/current/scripts/tuning.lua#L7964)、③[礼物池](../../../raw/current/scripts/prefabs/catcoon.lua#L159-L168) |
| `froglegs_cooked` | ①`cookable.product` 指向：在带 `cooker` 标签且未 `fueldepleted` 的实体（营火、火坑、龙鳞火炉等；火炉注册名是 `dragonflyfurnace`，见 [`prefabs/dragonfurnace.lua`](../../../raw/current/scripts/prefabs/dragonfurnace.lua#L238)）上烤（`dangerouscooker` 与骑乘门禁见[注册与构造](#注册与构造)）；烹饪锅只有 `stewer` 标签与组件，不提供烤制；②掉落者已带 `burnt`，或正在燃烧且（无 `fueled` 组件或 `burnable.ignorefuel` 为真）时，`DropLoot` 会把 `froglegs` 换成 `froglegs_cooked` | ①[接线](../../../raw/current/scripts/prefabs/froglegs.lua#L62-L65)、[烤制门槛](../../../raw/current/scripts/componentactions.lua#L1096-L1104)、[烹饪锅](../../../raw/current/scripts/prefabs/cookpot.lua#L290-L309)；②[门槛](../../../raw/current/scripts/components/lootdropper.lua#L415-L421)、[换名规则](../../../raw/current/scripts/components/lootdropper.lua#L424-L431) |

②对本页来源是**不可达**的：快照内 `prefabs/frog.lua` 没有 `burnable`／`MakeSmallBurnable` 调用，青蛙不会燃烧，所以这条换名只在模组给掉落者加燃烧能力时生效。[青蛙构造](../../../raw/current/scripts/prefabs/frog.lua#L171-L188)

### 消费端

- 烹饪锅把 `froglegs` 记为 `meat = 0.5`，`cancook` 为真，所以同时按名字派生一份 `froglegs_cooked` 的食材条目（`meat = 0.5` 加 `precook = 1`）：熟蛙腿能进锅、只算半份肉，但不能再回炉烤。快照内 `precook` 只有这一处写入，没有任何配方读取它。[食材值](../../../raw/current/scripts/cooking.lua#L108)、[派生](../../../raw/current/scripts/cooking.lua#L46-L65)、[槽位判定](../../../raw/current/scripts/containers.lua#L373-L375)
- 用到蛙腿的锅料理有两道：`frogglebunwich` 要蛙腿（生／熟皆可）加至少 0.5 蔬菜，`frognewton` 要无花果加蛙腿（生／熟皆可）；本页只记原料关系，配方与成品数值归料理页。[三明治](../../../raw/current/scripts/preparedfoods.lua#L20)、[无花果版](../../../raw/current/scripts/preparedfoods.lua#L954)
- 晾肉架只接收带 `dryable` 标签的物品，所以生蛙腿能上架、熟蛙腿会被拒；主模拟另有两条例外（物品刚生成 `GetTimeAlive() == 0`、或从本架原位挪回），它们只影响能否入槽，缺 `dryable` 组件时也不会开始计时（[例外](../../../raw/current/scripts/containers.lua#L234-L243)、[计时条件](../../../raw/current/scripts/components/dryingrack.lua#L236-L270)）；`dryable` 没写 `build`／`dried_build`，架面显示回落默认 `meat_rack_food`。架子上两种暂停条件见[常见陷阱](#常见陷阱与限制)。[接收条件](../../../raw/current/scripts/containers.lua#L234-L244)、[默认 build](../../../raw/current/scripts/components/dryingrack.lua#L62-L76)
- 交易值写死 0，所以蛙腿换不到金块：猪王的普通换金只认 `goldvalue > 0`，另有两条旁路——万圣夜糖果按 `halloweencandyvalue` 计入事件物品、`pig_token` 走小游戏；`AcceptTest` 还会先把带 `merm` 标签的交付者挡掉（`pig_token` 除外）。[赋值](../../../raw/current/scripts/prefabs/froglegs.lua#L78-L79)、[接收判定](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)、[交付处理](../../../raw/current/scripts/prefabs/pigking.lua#L572-L581)。鸟腿 `drumstick` 不出自本文件，它是 `prefabs/meats.lua` 里 `common()` 的产物、交易值为 `GOLD_VALUES.MEAT`＝1。[鸟腿金值](../../../raw/current/scripts/prefabs/meats.lua#L194)、[赋值](../../../raw/current/scripts/prefabs/froglegs.lua#L78-L79)、[猪王](../../../raw/current/scripts/prefabs/pigking.lua#L575)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | 写 `FOODTYPE.MEAT`、饥饿 12.5、生命 0／1、理智 −10／0；理智为负时赋值即加 `badfood` | 仅主模拟；`ismeat` 未写，食性看 `foodtype` 与 `edible_MEAT` 标签 | [[reference/components/edible#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/froglegs.lua#L91-L94)、[`badfood`](../../../raw/current/scripts/components/edible.lua#L1-L7) |
| 组件：perishable | `SetPerishTime` 加 `StartPerishing`，腐坏替换成 `spoiled_food` | 生腿 2880 秒、熟态 4800 秒 | [[reference/components/perishable#关键接口]]；[生腿](../../../raw/current/scripts/prefabs/froglegs.lua#L51-L54)、[熟态](../../../raw/current/scripts/prefabs/froglegs.lua#L108) |
| 组件：dryable | `product=smallmeat_dried`、`time=DRY_FAST`；未传 build | 仅生腿；`dryable` 标签两端都有，组件只在主模拟 | [[reference/components/dryable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/froglegs.lua#L56-L60) |
| 组件：cookable | `product=froglegs_cooked`；`Cook()` 生成产物并把鲜度折半补回 | 仅生腿；产物本身无 `cookable`，链到此为止 | [[reference/components/cookable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/froglegs.lua#L62-L65) |
| 组件：stackable | 显式写 `maxsize = STACK_SIZE_SMALLITEM`＝40 | 两端由 replica 同步堆叠数 | [[reference/components/stackable#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/froglegs.lua#L67-L68)、[默认值](../../../raw/current/scripts/components/stackable.lua#L23-L34) |
| 组件：tradable | `goldvalue = 0`，组件默认值同样是 0 | 主模拟；两态都不被猪王接收 | [[reference/components/tradable#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/froglegs.lua#L78-L79)、[默认](../../../raw/current/scripts/components/tradable.lua#L1-L7) |
| 组件：bait／hauntable | `bait` 让蛙腿能当陷阱饵；`MakeHauntableLaunchAndPerish` 的作祟只抛飞 | 作祟的加速腐烂分支被注释，实际只 `Launch` | [[reference/components/bait#关键接口]]；[bait](../../../raw/current/scripts/prefabs/froglegs.lua#L70)、[作祟](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337) |
| 标签：`smallmeat`／`catfood`／`rawmeat` | `catfood` 让浣猫当猫粮接受；`rawmeat` 让带 `eatsrawmeat` 的吃者跳过负生命／负理智结算；`smallmeat` 在快照内没有 `HasTag("smallmeat")` 消费者 | 标签两端都有；浣猫的接受判定在服务端（`trader:SetAcceptTest` 挂在主模拟段） | [浣猫](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279)、[吃者](../../../raw/current/scripts/components/eater.lua#L166-L170) |
| 关联 Prefab：`smallmeat_dried` | 干制产物；`meatrack2` 的依赖表也把 `froglegs` 列进去，用于架面贴图 | 到点由 `OnDoneDrying` 生成 | [[reference/prefabs/smallmeat]]；[依赖](../../../raw/current/scripts/prefabs/meatrack2.lua#L34-L54)、[生成点](../../../raw/current/scripts/components/dryingrack.lua#L198-L211) |
| 资源：`anim/frog_legs.zip` | 生／熟共用同一 bank 与 build，只切 `idle`／`cooked` 动画 | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/froglegs.lua#L1-L4) |

## 典型接入

### 把蛙腿的干制产物换成本链之外的肉干

- 前提与选择：服务端 `modmain.lua` 的 postinit。要改的是 `dryable` 组件的产物，而不是 `TUNING.DRY_FAST`（后者会连带改所有引用它的物品）。
- 生效与撤销：只影响之后在主模拟上完成干制的蛙腿；已干完的产物不变。产物在完成回调里现取 `GetProduct()`，挂架之后再改也生效。卸模组后回到 `smallmeat_dried`。
- 原版依据：`commonfn` 用 `dryable.SetProduct`／`SetDryTime` 初始化，完成回调到点才 `SpawnPrefab(product)`。[接线](../../../raw/current/scripts/prefabs/froglegs.lua#L56-L60)、[完成回调](../../../raw/current/scripts/components/dryingrack.lua#L198-L211)

```lua
-- modmain.lua：蛙腿改干成肉干，时长改成 DRY_MED
AddPrefabPostInit("froglegs", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local dryable = inst.components.dryable
    if dryable ~= nil then
        dryable:SetProduct("meat_dried")
        dryable:SetDryTime(GLOBAL.TUNING.DRY_MED)
    end
end)
```

- 调整与验证：`DRY_FAST`＝480 秒、`DRY_MED`＝960 秒；实测看架子上到点生成的是 `meat_dried` 还是 `smallmeat_dried`，不要只看组件字段。

## 常见陷阱与限制

- **误区：熟蛙腿还能再烤或上架。** 它连 `cookable`／`dryable` 标签都没有，动作处理器与肉架槽位测试都会拒绝。[熟态工厂](../../../raw/current/scripts/prefabs/froglegs.lua#L99-L100)、[接收条件](../../../raw/current/scripts/containers.lua#L234-L244)
- **误区：生蛙腿可以喂队友。** 理智 −10 让它带上 `badfood`，`IsFoodSafeToEat` 只在 PVP 或 `possessedbody` 目标上放行。[门槛](../../../raw/current/scripts/actions.lua#L2397-L2410)
- **边界：晾晒暂停时腐坏不是 0 倍，但酸雨要单独看。** 架子给容器的 `preserver` 速率：正常晾晒且物品带 `dryable` 组件时返回 0；**酸雨**（`isinacid` 为真）先命中第一支，恒返回 `1 + rate`，与是否暂停无关；只有**普通雨**暂停（`isinacid` 为假）才返回 `nil`，`preserver` 回落到 1（按常规腐坏）。[速率函数](../../../raw/current/scripts/components/dryingrack.lua#L26-L38)、[`or 1`](../../../raw/current/scripts/components/preserver.lua#L12-L14)
- **边界：烧毁架子是立即出产物。** `OnBurnt` 对每格跑 `InstantDry`：直接生成产物并抛到架子位置，原物 `Remove`，连湿度都不继承。[InstantDry](../../../raw/current/scripts/components/dryingrack.lua#L376-L391)
- **误区：`ismeat` 决定是不是肉。** 本页两件都没写该字段；判定走 `foodtype` 与 `edible_MEAT` 标签。[工厂](../../../raw/current/scripts/prefabs/froglegs.lua#L48-L49)
- **未核验**：本页未运行游戏；干制链路只按源码推导，未实测多雨／酸雨环境下的暂停与烧架。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/froglegs.lua` 的 `commonfn` 与两个工厂，核对 `edible`／`perishable`／`stackable`／`dryable`／`cookable`／`tradable`／`floater`／`bait` 的相关分支，`dryingrack` 的速率／完成／烧毁回调，`containers.lua` 的 `meatrack.itemtestfn` 与 `cookpot.itemtestfn`，`componentactions.lua` 的 COOK 门槛，`cooking.lua` 的食材值与 `cancook` 派生，`tuning` 的鲜度／干制／热量／理智／堆叠常量，`prefabs/frog.lua` 的掉落、`prefabs/catcoon.lua` 的礼物池，`scrapbookdata` 交叉核对。
  名称核验：`languages/chinese_s.po` 的 `NAMES.FROGLEGS`＝蛙腿、`NAMES.FROGLEGS_COOKED`＝熟蛙腿。
  未覆盖：月蛙的其它掉落、浣猫礼物的抽取概率、模组注入与活动脚本。
- 游戏运行：未运行。天数、秒数与三维均为 tuning 推导（`total_day_time`＝480 秒）。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警。

## 相关链接

- 可食与鲜度：[[reference/components/edible]]、[[reference/components/perishable]]
- 两条转换：[[reference/components/dryable]]、[[reference/components/cookable]]、[[reference/components/stackable]]
- 同链产物与相邻页：[[reference/prefabs/smallmeat]]、[[reference/prefabs/drumstick]]、[[reference/prefabs/batwing]]
- 获得与贩卖：[[reference/components/lootdropper]]、[[reference/components/tradable]]

返回 [[reference/prefabs/index|Prefab 索引]]。
