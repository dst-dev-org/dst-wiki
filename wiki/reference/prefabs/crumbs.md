---
title: "饼干屑（crumbs）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "冬宴姜饼追猎的踪迹物品：单注册 crumbs，饥饿 1、生命 0、GOODIES，鲜度基准 1 天（480 秒）后原地消失而不替换成腐烂食物，40 堆叠、可当陷阱诱饵，另有姜饼猪、姜饼屋与姜饼座狼三个来源。"
aliases: ["crumbs", "饼干屑", "姜饼屑", "crumbs 踪迹"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/crumbs.lua", "components/gingerbreadhunter.lua", "prefabs/gingerbreadpig.lua", "prefabs/gingerbreadhouse.lua", "stategraphs/SGgingerbreadpig.lua", "components/lootdropper.lua", "prefabs/warg.lua", "prefabs/forest.lua", "components/edible.lua", "components/perishable.lua", "components/bait.lua", "components/trap.lua", "components/tradable.lua", "componentactions.lua", "actions.lua", "standardcomponents.lua", "tuning.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
size_class: simple
reliability: verified
validation: source-reviewed
status: active
---

# 饼干屑（crumbs）

## 快速判断

- 职责与适用：单一注册 `crumbs`，物品态食物，原版里是冬宴姜饼追猎的踪迹；既由系统 `SpawnPrefab` 撒点，也从姜饼猪、姜饼屋、姜饼座狼三处产出。简中名 **饼干屑**（`NAMES.CRUMBS`，英文 "Cookie Crumbles"）。[注册](../../../raw/current/scripts/prefabs/crumbs.lua#L58)、[名称](../../../raw/current/scripts/languages/chinese_s.po#L341935-L341937)
- 优先使用：改三维与鲜度看[数值与腐坏](#数值与腐坏)；只想知道谁生成它看[来源](#来源谁生成谁掉落)；挂诱饵看[关联机制导航](#关联机制导航)的 `bait` 行。
- 重要副作用：`perishable` 到点调 `SetOnPerishFn(inst.Remove)`，**整块饼屑直接删除、没有替换物**（不是 `spoiled_food`）；`MakeHauntableLaunchAndPerish` 同样只剩 Launch。[到点删除](../../../raw/current/scripts/prefabs/crumbs.lua#L50-L53)、[Perish 流程](../../../raw/current/scripts/components/perishable.lua#L277-L319)、[仅 Launch](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337)
- 修改边界：本页只负责 `crumbs` 自身；姜饼追猎的刷新预算归 [[reference/components/gingerbreadhunter]]，姜饼猪／屋／座狼本体行为各归其页（Wiki 待建）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `crumbs` | `fn()`：`CreateEntity` → bank／build `cookie_crumbs`、播 `idle` → 加标签 → `MakeInventoryFloatable` → `SetPristine`；**非主模拟在这里就返回**，玩法组件只在主模拟挂 | 物品态：`inventoryitem`、`stackable`（`STACK_SIZE_SMALLITEM`）、`edible`（GOODIES／0／1）、`bait`、`tradable`、`inspectable`，可燃＋传热＋可作祟，最后 `perishable`；标签 `catfood`／`crumbs` 在 `SetPristine` **之前**加（两端都有），`burnable`／`propagator` 只在主模拟挂。[工厂](../../../raw/current/scripts/prefabs/crumbs.lua#L8-L56)、[主模拟分界](../../../raw/current/scripts/prefabs/crumbs.lua#L27-L32)、[标签](../../../raw/current/scripts/prefabs/crumbs.lua#L22-L23)、[可燃与传热](../../../raw/current/scripts/prefabs/crumbs.lua#L46-L48) |

## 数值与腐坏

时间基准：`seg_time`＝30 秒、`total_day_time`＝480 秒、`perish_warp`＝1，所以 1 天＝480 秒。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[`perish_warp`](../../../raw/current/scripts/tuning.lua#L47-L47)

| 项 | 值 | 证据 |
|---|---|---|
| 饥饿／生命／理智 | 1／0／**理智未赋值**（`edible` 构造时默认 0） | [赋值](../../../raw/current/scripts/prefabs/crumbs.lua#L36-L39)、[默认值](../../../raw/current/scripts/components/edible.lua#L19-L25) |
| 食物类型 | `FOODTYPE.GOODIES`，赋值时同步加 `edible_GOODIES` 标签 | [赋值](../../../raw/current/scripts/prefabs/crumbs.lua#L37-L37)、[标签同步](../../../raw/current/scripts/components/edible.lua#L9-L16) |
| 堆叠 | `STACK_SIZE_SMALLITEM`＝40 | [赋值](../../../raw/current/scripts/prefabs/crumbs.lua#L33-L34)、[常量](../../../raw/current/scripts/tuning.lua#L83-L83) |
| 交易值 | 未赋值 → `tradable` 构造默认 0（金块换不到东西） | [挂件](../../../raw/current/scripts/prefabs/crumbs.lua#L43-L43)、[默认值](../../../raw/current/scripts/components/tradable.lua#L1-L3) |
| 鲜度 | `PERISH_ONE_DAY`＝1 天＝480 秒，到点删除、无替换物 | [赋值](../../../raw/current/scripts/prefabs/crumbs.lua#L50-L53)、[常量](../../../raw/current/scripts/tuning.lua#L2043-L2043) |

- **计时起点是构造完成那一刻**：`SetPerishTime` 同时写 `perishtime` 与 `perishremainingtime`，紧接的 `StartPerishing()` 起周期任务，所以一生成就走表、不需玩家先捡；该任务 `DoPeriodicTask(10 + math.random()*FRAMES*8, …)` 约每 10 秒结算一档，不是逐帧扣。[设时长](../../../raw/current/scripts/components/perishable.lua#L188-L194)、[起表](../../../raw/current/scripts/components/perishable.lua#L262-L271)
- **480 秒只是基准**：`perishable:Update` 用 `dt × modifier` 扣剩余时间，其中地面（无 owner）乘 `PERISH_GROUND_MULT`＝1.5、湿身再乘 `PERISH_WET_MULT`＝1.3、低温乘 `PERISH_WINTER_MULT`＝0.75、全局再乘 `PERISH_GLOBAL_MULT`＝1；倍率在同一组件里连乘，落地摆放因此比拿在手里烂得快。[扣时](../../../raw/current/scripts/components/perishable.lua#L60-L137)、[倍率](../../../raw/current/scripts/tuning.lua#L2079-L2087)

## 来源：谁生成、谁掉落

| 来源 | 接法 | 条件与证据 |
|---|---|---|
| 姜饼猪 `gingerbreadpig` | `SpawnPrefab("crumbs")`；退场时另有 `ReplacePrefab(inst, "crumbs")` | `DropCrumb` 由 `StartDroppingCrumbs` 以 `CRUMB_SPAWN_PERIOD`＝3 秒周期驱动；猪带 `idle`／`sleeping` 状态标签时跳过，5 格内已有 `crumbs` 标签实体也跳过；生成点在其上方 +2 并给随机抛物线初速。被玩家追到放弃且猎手排点成功时，**整只猪被替换成一块饼屑**（生成新实体、删旧实体，不走掉落表）。[周期与过滤](../../../raw/current/scripts/prefabs/gingerbreadpig.lua#L21-L53)、[替换](../../../raw/current/scripts/prefabs/gingerbreadpig.lua#L61-L71)、[固定掉落表](../../../raw/current/scripts/prefabs/gingerbreadpig.lua#L13-L19) |
| 姜饼屋 `gingerbreadhouse` | `SetSharedLootTable` 掉落表 | 表里 5 条 `{'crumbs', 1.00}`。锤毁时 `onhammered` **先** `Extinguish()` 再判 `burnt`：燃烧中的屋子先被灭火、随后按原表 `DropLoot()` 掉满 5 份；只有已 `burnt` 的屋子会跳过 DropLoot 直接 `Remove()`，故燃烧改写分支在这里走不到。[掉落表](../../../raw/current/scripts/prefabs/gingerbreadhouse.lua#L18-L26)、[锤毁顺序](../../../raw/current/scripts/prefabs/gingerbreadhouse.lua#L51-L68)、[燃烧改写](../../../raw/current/scripts/components/lootdropper.lua#L415-L442) |
| 姜饼座狼 `gingerbreadwarg` | `SetSharedLootTable` 掉落表 | 表里 `{'crumbs', 1.00}` 两条、`{'crumbs', 0.50}` 两条；`MakeWarg` 工厂把 `name` 交给 `lootdropper:SetChanceLootTable(name)`，座狼变体的 `name` 正是 `gingerbreadwarg`。[掉落表](../../../raw/current/scripts/prefabs/warg.lua#L124-L141)、[接线](../../../raw/current/scripts/prefabs/warg.lua#L951-L955)、[变体](../../../raw/current/scripts/prefabs/warg.lua#L1097-L1103) |
| 冬宴猎手 `GingerbreadHunter` | `SpawnPrefab("crumbs")` | `SpawnCrumbTrail` 遍历 `crumb_pts`：**除末点外**每点各 `SpawnPrefab("crumbs")` 一块；末点（`i == #self.crumb_pts`）按 `hunt_count` 与运气改成生成姜饼猪、姜饼屋或姜饼座狼，本身不撒饼屑。[撒点](../../../raw/current/scripts/components/gingerbreadhunter.lua#L184-L217) |
| `forest.lua` | 仅依赖声明 | `local prefabs`（L221 起）里列了 `crumbs`，这是预加载声明，不代表森林会生成它。[声明](../../../raw/current/scripts/prefabs/forest.lua#L392-L395) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`edible` | 写 GOODIES／生命 0／饥饿 1；食物类型变更会同步 `edible_GOODIES` 标签 | 仅主模拟 | [[reference/components/edible]]；[接线](../../../raw/current/scripts/prefabs/crumbs.lua#L36-L39) |
| 组件：`perishable` | 鲜度基准 480 秒（1 天）后 `Remove`，无替换物 | 仅主模拟；两端读鲜度靠 Replica | [[reference/components/perishable]]；[接线](../../../raw/current/scripts/prefabs/crumbs.lua#L50-L53) |
| 组件：`stackable` | `maxsize`＝40 | 两端由 Replica 同步 | [[reference/components/stackable]]；[接线](../../../raw/current/scripts/prefabs/crumbs.lua#L33-L34) |
| 组件：`bait` | 组件本身不判定生物，只登记"当前占用它的陷阱"；被吃／被偷／被拿起时通知该陷阱 | 主模拟；`bait.trap` 由陷阱写入 | [[reference/components/bait]]；[挂载](../../../raw/current/scripts/prefabs/crumbs.lua#L41-L41)、[通知](../../../raw/current/scripts/components/bait.lua#L1-L29) |
| 陷阱侧的两条判定 | ① 玩家 `BAIT` 放饵：目标带 `canbait` 才出现动作；② 陷阱合拢时若没放饵，在 2 格内找**带 `bait` 组件**、标签属 `BAIT_TAGS`（`molebait` 或任一 `edible_<FOODTYPE>`）、且被捕目标 `eater:CanEat` 能吃的物品并消耗它 | 主模拟；`crumbs` 走 `edible_GOODIES` 这条路，`molebait` 分支不适用 | [[reference/components/trap]]；[候选](../../../raw/current/scripts/componentactions.lua#L1046-L1050)、[动作](../../../raw/current/scripts/actions.lua#L1600-L1605)、[标签集](../../../raw/current/scripts/components/trap.lua#L264-L267)、[判定](../../../raw/current/scripts/components/trap.lua#L313-L337) |
| 组件：`tradable` | 挂上但没写 `goldvalue`，保持默认 0 | 主模拟 | [[reference/components/tradable]]；[接线](../../../raw/current/scripts/prefabs/crumbs.lua#L43-L43) |
| 标签：`catfood` | 浣猫收下玩家递来的饼屑；行为树也把它当"可玩食物" | 两端都在 pristine 前加 | [接收](../../../raw/current/scripts/prefabs/catcoon.lua#L273-L279)、[行为树](../../../raw/current/scripts/brains/catcoonbrain.lua#L24-L45) |
| 标签：`crumbs` | 唯一消费者是姜饼猪的去重：5 格内已有饼屑就不再生成 | 两端都有 | [去重](../../../raw/current/scripts/prefabs/gingerbreadpig.lua#L24-L30) |
| 资源：`cookie_crumbs.zip` | bank／build 都是 `cookie_crumbs`，动画 `idle` | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/crumbs.lua#L3-L6) |

## 典型接入

### 延长饼屑的保留时间

服务端 `modmain.lua` 的 postinit 改实例的 `perishable`（别改 `TUNING.PERISH_ONE_DAY`，会连带同档食物）。`SetPerishTime` 同时把剩余时间设为新总时长并在计时中重启任务，新生成的饼屑立刻生效；卸模组后新实例回到 1 天。原版接线见[赋值](../../../raw/current/scripts/prefabs/crumbs.lua#L50-L53)。

```lua
-- modmain.lua：把饼干屑从 1 天改成 6 天（480 → 2880 秒）
AddPrefabPostInit("crumbs", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local perishable = inst.components.perishable
    if perishable ~= nil then
        perishable:SetPerishTime(GLOBAL.TUNING.PERISH_FAST)
    end
end)
```

`PERISH_FAST`＝6 天＝2880 秒[档位](../../../raw/current/scripts/tuning.lua#L2046-L2046)；验证看新生成饼屑的鲜度条与读档后的剩余比例，不要只看组件字段。

## 常见陷阱与限制

- **误区：饼屑烂掉会变成腐烂食物。** 它只设了 `SetOnPerishFn(inst.Remove)`，没有 `onperishreplacement`，`Perish()` 执行完回调后实体就不在了。[到点删除](../../../raw/current/scripts/prefabs/crumbs.lua#L53-L53)、[无替换物](../../../raw/current/scripts/components/perishable.lua#L293-L319)
- **边界：客户端没有这些组件。** `fn` 在 `SetPristine` 后先判断 `TheWorld.ismastersim` 再挂 `inventoryitem`／`edible`／`perishable`，客户端读 `inst.components.perishable` 会是 nil。[分界](../../../raw/current/scripts/prefabs/crumbs.lua#L27-L32)
- **边界：`MakeHauntableLaunchAndPerish` 名不副实。** 该 helper 的 `ReducePercent` 分支整段被注释，本 build 只会 `Launch`。[helper](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337)
- **边界：掉落表 ≠ 必掉，落地物还可能被燃烧改写。** 猎手撒点是 `SpawnPrefab` 直出；猪（`lootdropper:SetLoot` 固定表，需 `chased` 且 `chased_by_player` 或 30 格内有玩家）与座狼（机会表）走 `DropLoot`，若死亡时正在**普通燃烧**或已 `burnt`，表项按 `BURNED_LOOT_OVERRIDES`→`_cooked`→`cooked`→`ash` 改写，`crumbs` 无 cooked 变体，因此已进入 `DropLoot` 的饼干屑表项会转成灰；**受控燃烧**（`burnable:GetControlledBurn()`）保留原产物。屋的 5 份只在「锤毁且未 `burnt`」时成立；座狼另有 2 条 0.50 可能不掉。[燃烧改写](../../../raw/current/scripts/components/lootdropper.lua#L415-L442)、[屋表](../../../raw/current/scripts/prefabs/gingerbreadhouse.lua#L18-L26)、[猪死亡](../../../raw/current/scripts/stategraphs/SGgingerbreadpig.lua#L76-L78)、[座狼表](../../../raw/current/scripts/prefabs/warg.lua#L133-L136)
- **未核验**：未运行游戏。"多久消失"按 `PERISH_ONE_DAY` 与倍率推导（基准 480 秒，落地 1.5 倍约 320 秒），未实测；本页检索范围内 `cooking.lua` 未把 `crumbs` 注册为锅食材。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/crumbs.lua` 全文（58 行）；核对 `edible` 默认值与标签同步、`perishable` 的 `SetPerishTime`／`StartPerishing`／`Perish`／`Update` 倍率、`tradable` 默认 `goldvalue`、`bait` 与 `trap` 取饵判定；来源侧核对 `SpawnCrumbTrail`、`DropCrumb`／`ReplacePrefab`、两处掉落表及 `SetChanceLootTable` 接线、`forest.lua` 依赖列表；`tuning.lua` 的时间／堆叠／鲜度／倍率常量与 `scrapbookdata.lua`（`hungervalue=1`、`healthvalue=0`、`sanityvalue=0`、`stacksize=40`、`foodtype="GOODIES"`）一致；名称核验 `languages/chinese_s.po` 的 `NAMES.CRUMBS`＝饼干屑。
- 未覆盖：姜饼追猎的刷新概率与 `GINGERWARG_CHANCE` 分支、`ReplacePrefab` 替换后的动画表现、浣猫对 `catfood` 的后续礼物逻辑。
- 游戏运行：未运行；天数与秒数均为源码推导（`total_day_time`＝480 秒）。
- 修订：2026-11-09，初检 P1/P3（撒点方向、结算粒度）与二轮外部复检 P2（猪死亡掉落、燃烧改写、鲜度基准）——已按源代码更正，见摘要、「来源」「数值与腐坏」。
- 修订：2026-11-09，外部复检 P2（必给清单拆开、补燃烧改写；屋表口径保持）——已改。
- 结构检查：见交付回执。

## 相关链接

- 食物与鲜度：[[reference/components/edible]]、[[reference/components/perishable]]、[[reference/components/stackable]]
- 诱饵与陷阱：[[reference/components/bait]]、[[reference/components/trap]]
- 追猎组件：[[reference/components/gingerbreadhunter]]
- 同批小件：[[reference/prefabs/bird-egg]]

返回 [[reference/prefabs/special-food-index|节庆、角色与特殊食品索引]]。
