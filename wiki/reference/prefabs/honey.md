---
title: "蜂蜜与蜂王浆（honey、royal_jelly）"
created: "2026-11-09"
updated: "2026-11-10"
type: prefab
summary: "蜂箱、蜂窝、杀人蜂蜂窝、蜜蜂、蜂王、蜂王冠拆解与克劳斯袋多条产出口，加上蜂蜜／蜂王浆的数值、鲜度（40 天与 10 天）、堆叠 40 与 honeyed 标签；蜂箱的积累靠蜜蜂花粉回巢或休眠计时，烧毁的蜂窝按 hive 分支只出灰烬。"
aliases: ["honey", "royal_jelly", "蜂蜜", "蜂王浆", "honeyed", "蜜"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/honey.lua", "prefabs/royal_jelly.lua", "prefabs/beebox.lua", "prefabs/beehive.lua", "prefabs/wasphive.lua", "prefabs/bee.lua", "prefabs/beequeen.lua", "prefabs/beequeenhive.lua", "prefabs/hats.lua", "prefabs/hermitcrab.lua", "prefabs/staff_common.lua", "components/harvestable.lua", "components/pollinator.lua", "components/workable.lua", "components/lootdropper.lua", "components/edible.lua", "components/perishable.lua", "components/stackable.lua", "componentactions.lua", "actions.lua", "cooking.lua", "recipes.lua", "preparedfoods.lua", "prefabs/bearger.lua", "brains/beargerbrain.lua", "components/klaussackloot.lua", "components/container.lua", "prefabs/spoiledfood.lua", "standardcomponents.lua", "prefablist.lua", "tuning.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 蜂蜜与蜂王浆（honey、royal_jelly）

## 快速判断

- 职责与适用：`honey`／`royal_jelly` 是两件可拾取、可食用的物品（简中实显 **蜂蜜**／**蜂王浆**），各自独占一个 `prefabs/*.lua` 且只导出一个注册；两者差异只在初始化赋值、`MakeHauntableLaunch(AndPerish)` 的选型与组件挂载顺序。[honey 构造](../../../raw/current/scripts/prefabs/honey.lua#L11-L55)、[royal_jelly 构造](../../../raw/current/scripts/prefabs/royal_jelly.lua#L11-L55)
- 优先使用：数值／鲜度／堆叠看[两件对照](#两件对照)；从哪里来看[获得途径](#获得途径)，能下什么锅、能造什么看[用途与消耗](#用途与消耗)；蜂箱为什么长时间不长蜜看[蜂箱的积累路径](#蜂箱的积累路径)。
- 重要副作用：蜂蜜不是定时产物——蜂箱醒着时只有蜜蜂采满花粉回巢才 +1，实体睡着时才走 300 秒计时器；两件都带 `honeyed`，熊獾会优先吃地上的蜂蜜，并砸开放着蜂蜜的冰箱／箱子／背包。
- 修改边界：只覆盖这两件物品与其产出入口。蜜脾 `honeycomb` 归另一分派（B11），蜂箱／蜂窝／蜂王本体行为归后续批次，本页只保留"谁产出什么"。

## 注册与构造

两件分别在 `prefablist.lua` 的 `PREFABFILES` 里按文件名登记，各自 `return Prefab(...)` 导出 1 个注册，依赖表都只声明 `spoiled_food`。[honey 登记](../../../raw/current/scripts/prefablist.lua#L564)、[royal_jelly 登记](../../../raw/current/scripts/prefablist.lua#L1068)、[honey 导出](../../../raw/current/scripts/prefabs/honey.lua#L57)、[royal_jelly 导出](../../../raw/current/scripts/prefabs/royal_jelly.lua#L57)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `honey` | `fn()`：SetBank／SetBuild `honey`、播 `idle`；pristine **之前**加 `honeyed` 与 `MakeInventoryFloatable("med", nil, 0.8)`；`SetPristine()` 后进主模拟才挂组件 | 生命 `HEALING_SMALL`、饥饿 `CALORIES_TINY`（不写理智），鲜度 `PERISH_SUPERSLOW`，收尾 `MakeHauntableLaunchAndPerish`。[构造](../../../raw/current/scripts/prefabs/honey.lua#L11-L55) |
| `royal_jelly` | 同上换 bank／build `royal_jelly`；浮空参数是 `("small", nil, 0.8)` | 生命 `HEALING_LARGE`、饥饿 `CALORIES_SMALL`、理智 `SANITY_MED`（本页唯一显式给正理智的一件），鲜度 `PERISH_MED`，收尾只 `MakeHauntableLaunch`。[构造](../../../raw/current/scripts/prefabs/royal_jelly.lua#L11-L55) |

- 两件都在 `SetPristine()` 之前加 `honeyed`，客户端也能读到该标签；`stackable` 显式写 `TUNING.STACK_SIZE_SMALLITEM`，不落组件默认的 20。[标签](../../../raw/current/scripts/prefabs/honey.lua#L24)、[堆叠](../../../raw/current/scripts/prefabs/royal_jelly.lua#L39-L40)、[组件默认](../../../raw/current/scripts/components/stackable.lua#L23-L28)
- 挂载顺序不同（同值不同序）：`honey` 是 `edible` → `stackable` → `tradable` → `inspectable` → `perishable` → `inventoryitem`；`royal_jelly` 是 `edible` → `stackable` → `perishable` → `tradable` → `inventoryitem` → `inspectable`，即 `perishable` 提前、`inspectable` 靠后。[honey 组件段](../../../raw/current/scripts/prefabs/honey.lua#L34-L50)、[royal_jelly 组件段](../../../raw/current/scripts/prefabs/royal_jelly.lua#L34-L50)

### 名称核验

`NAMES.HONEY`＝蜂蜜、`NAMES.ROYAL_JELLY`＝蜂王浆，均为简中实显；两者没有同族歧义名，注册名即检索名。[HONEY](../../../raw/current/scripts/languages/chinese_s.po#L343939-L343942)、[ROYAL_JELLY](../../../raw/current/scripts/languages/chinese_s.po#L348804-L348807)

## 两件对照

1 天 = 16 × 30 = 480 秒。[时间基准](../../../raw/current/scripts/tuning.lua#L19-L20)

| 注册 | 生命 | 饥饿 | 理智 | 食性 | 鲜度 | 腐坏产物 | 堆叠 | 漂浮 |
|---|---:|---:|---:|---|---|---|---:|---|
| `honey` | +3（`HEALING_SMALL`） | +9.375（`CALORIES_TINY`） | 0（未写，组件默认） | GENERIC（未写） | 40 天（`PERISH_SUPERSLOW`） | `spoiled_food` | 40 | `med`，缩放 0.8 |
| `royal_jelly` | +40（`HEALING_LARGE`） | +12.5（`CALORIES_SMALL`） | +15（`SANITY_MED`） | GENERIC（未写） | 10 天（`PERISH_MED`） | `spoiled_food` | 40 | `small`，缩放 0.8 |

- 常量：`HEALING_SMALL`＝3、`HEALING_LARGE`＝40、`CALORIES_TINY`＝9.375、`CALORIES_SMALL`＝12.5、`SANITY_MED`＝15、`PERISH_MED`＝10 天、`PERISH_SUPERSLOW`＝40 天、`STACK_SIZE_SMALLITEM`＝40。[数值常量](../../../raw/current/scripts/tuning.lua#L2021-L2061)、[堆叠常量](../../../raw/current/scripts/tuning.lua#L82-L83)
- 图鉴（scrapbook）侧记录与上表一致（`hungervalue`／`healthvalue`／`sanityvalue`／`perishable`＝19200／4800 秒），可作交叉核对，不是运行时来源。[honey](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L710)、[royal_jelly](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1261)
- 两件都**没有**显式写 `foodtype`，落 `edible` 构造默认 `FOODTYPE.GENERIC`；`edible` 只在 `healthvalue < 0` 或 `sanityvalue < 0` 时打 `badfood`，本页两件都不带。[默认食性](../../../raw/current/scripts/components/edible.lua#L19-L24)、[badfood 判定](../../../raw/current/scripts/components/edible.lua#L1-L7)
- 腐坏时 `perishable:Perish()` 生成 `onperishreplacement`，按原叠数（超上限时放开）同步，并把原槽位传给 `holder:GiveItem(goop, slot)`：优先放回原持有者的原格，容器按 `itemtestfn` 拒收时改走 `GiveItem` 的落地分支、掉在原地。[替换](../../../raw/current/scripts/components/perishable.lua#L301-L312)、[入槽校验](../../../raw/current/scripts/components/container.lua#L295)、[拒收落地](../../../raw/current/scripts/components/container.lua#L560-L566)

## 获得途径

| 来源 | 产出 | 条件 | 证据 |
|---|---|---|---|
| 蜂箱 `beebox` | 每次收获给**当前积累量**的 `honey`（上限 6） | `harvestable:SetUp("honey", 6, nil, ...)`；实体带 `harvestable` 标签时才出 `HARVEST` 动作 | [挂载](../../../raw/current/scripts/prefabs/beebox.lua#L281)、[动作可见性](../../../raw/current/scripts/componentactions.lua#L404-L406)、[动作实现](../../../raw/current/scripts/actions.lua#L2651-L2678) |
| 蜂箱（锤掉） | 先把箱内蜂蜜 harvest 出来，再掉建造材料 | `workable` 为 `HAMMER` 4 下；`onhammered` 先 `harvestable:Harvest()` 再 `DropLoot()`，`droprecipeloot` 默认开 | [onhammered](../../../raw/current/scripts/prefabs/beebox.lua#L77-L90)、[配方掉落](../../../raw/current/scripts/components/lootdropper.lua#L242-L250) |
| 蜂窝 `beehive` | 3 `honey` + 1 `honeycomb`（固定表） | 死亡时 `DropLoot`；`SetLoot` 会清空随机表 | [战利品](../../../raw/current/scripts/prefabs/beehive.lua#L204-L205)、[死亡](../../../raw/current/scripts/prefabs/beehive.lua#L77-L86)、[SetLoot 语义](../../../raw/current/scripts/components/lootdropper.lua#L30-L35) |
| 杀人蜂蜂窝 `wasphive` | 3 `honey` + 1 `honeycomb`（固定表） | 同上 | [战利品](../../../raw/current/scripts/prefabs/wasphive.lua#L149-L150)、[死亡](../../../raw/current/scripts/prefabs/wasphive.lua#L22-L30) |
| 蜜蜂 `bee` | 1/6 概率 1 个 `honey` | `AddRandomLoot("honey", 1)`＋`("stinger", 5)`＋`numrandomloot = 1`，按权重抽一件 | [掉落表](../../../raw/current/scripts/prefabs/bee.lua#L194-L197)、[加权抽取](../../../raw/current/scripts/components/lootdropper.lua#L99-L112) |
| 蜂王 `beequeen` | `royal_jelly` 6 条 1.00＋1 条 0.50；`honey` 3 条 1.00＋1 条 0.50；另 `hivehat`／`honeycomb` | `SetSharedLootTable` 的每条独立判定 | [战利品表](../../../raw/current/scripts/prefabs/beequeen.lua#L24-L43)、[共享表](../../../raw/current/scripts/components/lootdropper.lua#L22-L24) |
| 蜂王巢 `beequeenhivegrown` | 敲一下按概率给 `honey`（`rnd < .5`）或 `honeycomb`（`rnd < .01`） | `lootscale`＝**本击扣减后**的剩余工作量／`BEEQUEEN_SPAWN_MAX_WORK`（记 workleft_after；蜜脾概率＝0.01×workleft_after/16，工作量从 16 起、本击扣 1 时首锤（16→15）＝0.9375%），`rnd`＝`random()/lootscale`，剩余越少越难出、归零不给；整段只在 `worker` 非空、有效、存活、带 `player` 且非 `playerghost` 时执行，且本次没在 `BEEQUEEN_SPAWN_WORK_THRESHOLD` 的召蜂王判定里提前 return | [敲巢掉落](../../../raw/current/scripts/prefabs/beequeenhive.lua#L188-L213)、[扣减顺序](../../../raw/current/scripts/components/workable.lua#L130-L153) |
| 蜂王冠 `hivehat` 拆解 | 拆解表原料 4 `honeycomb` + 3 `honey` + 2 `royal_jelly` + 4 `bee` | `DeconstructRecipe`；各材料按护甲耐久比例**独立**向上取整（80% 耐久蜜脾仍是 4 个、75% 起为 3 个），冠仍存在时每项至少 1 个 | [拆解表](../../../raw/current/scripts/recipes.lua#L1673)、[折算](../../../raw/current/scripts/prefabs/staff_common.lua#L637-L649)、[生成](../../../raw/current/scripts/prefabs/staff_common.lua#L663-L674) |
| 珍珠（隐士蟹） | 完成"种花"友好任务后礼物表加入 `honey` | `friendlytasks[TASKS.PLANT_FLOWERS].complete` | [礼物](../../../raw/current/scripts/prefabs/hermitcrab.lua#L864-L868) |
| 克劳斯袋 `klaussack` | 从 9 项 `giant_loot3` 里抽 2 个**不同**项，`royal_jelly` 命中约 2/9 ≈ 22% | 开袋战利品生成；`i2` 取 `random(#-1)` 后跳过 `i1` | [战利品表](../../../raw/current/scripts/components/klaussackloot.lua#L23-L34)、[抽取](../../../raw/current/scripts/components/klaussackloot.lua#L115-L124) |

- 蜂箱有两个注册：玩家可造的 `beebox`，与珍珠的 `beebox_hermit`。后者共用同一 `harvestable` 配置，但没有 `workable`／`lootdropper`（`master_postinit` 传了 `nil`）。[两个注册](../../../raw/current/scripts/prefabs/beebox.lua#L367-L369)、[master 段](../../../raw/current/scripts/prefabs/beebox.lua#L339-L350)
- 蜂王冠本体是 `MakeHat("hive")`，护甲 945／吸收 0.7，所以拆解收益随耐久下降。[注册](../../../raw/current/scripts/prefabs/hats.lua#L7887)、[构造](../../../raw/current/scripts/prefabs/hats.lua#L2070-L2090)

## 蜂箱的积累路径

`harvestable` 的 `SetUp(product, max, time, onharvest, ongrow)` 里 `time` 传了 `nil`，所以蜂箱刚造出来**没有**生长计时器；`StartGrowing()` 在 `growtime` 为空时不创建计时任务。[SetUp](../../../raw/current/scripts/components/harvestable.lua#L39-L45)、[StartGrowing](../../../raw/current/scripts/components/harvestable.lua#L146-L154)

| 状态 | 每次 +1 的触发 | 条件 | 证据 |
|---|---|---|---|
| 醒着（实体未休眠） | 蜜蜂带着花粉回巢：`childgoinghome` → `pollinator:HasCollectedEnough()` → `harvestable:Grow()` | 花粉计数 `#flowers > collectcount`（5），即第 6 朵起算"采够" | [回巢](../../../raw/current/scripts/prefabs/beebox.lua#L144-L152)、[判定](../../../raw/current/scripts/components/pollinator.lua#L28-L30) |
| 睡着（实体休眠） | 计时长度 300 秒（`BEEBOX_HONEY_TIME`＝`day_time`）到点 `Grow()` 一次 | 进休眠时 `TryStartSleepGrowing` 写入该 `growtime`；唤醒时 `StopSleepGrowing` 清回 `nil` 并暂停 | [休眠](../../../raw/current/scripts/prefabs/beebox.lua#L198-L206)、[计时写入](../../../raw/current/scripts/prefabs/beebox.lua#L160-L174)、[常量](../../../raw/current/scripts/tuning.lua#L1156-L1158) |

- **休眠计时**那条受全部门禁：没烧毁、非冬季、有蜜蜂（`childspawner:NumChildren()` 是**箱内＋箱外合计**，`> 0` 即可，不必箱内留蜂）、30 格内至少有一朵带 `flower` 标签的实体；`CanStartGrowing` 只被 `Start()` 与 `TryStartSleepGrowing` 调用。[门禁](../../../raw/current/scripts/prefabs/beebox.lua#L30-L37)、[亮处调用](../../../raw/current/scripts/prefabs/beebox.lua#L48-L55)、[休眠调用](../../../raw/current/scripts/prefabs/beebox.lua#L160-L167)
- **回巢 +1** 那条只判三件事：未烧毁、`pollinator:HasCollectedEnough()`、有 `harvestable`——不查季节、不查花距，随后直接 `Grow()`。[回巢](../../../raw/current/scripts/prefabs/beebox.lua#L144-L152)
- 收获会清空计数并停掉计时器（`onharvest` 里 `SetGrowTime(nil)`＋`StopGrowing`），下一次 +1 从零开始；满 6 时 `Grow()` 因达到 `maxproduce` 不再累加。[onharvest](../../../raw/current/scripts/prefabs/beebox.lua#L122-L142)、[上限](../../../raw/current/scripts/components/harvestable.lua#L128-L144)
- 每 `Grow()` 会回调 `updatelevel` 换外观：6／3／1／0 四档对应 `honey3`／`honey2`／`honey1`／`bees_loop`。[档位](../../../raw/current/scripts/prefabs/beebox.lua#L22-L28)

## 用途与消耗

两件的去向集中在炖锅与角色专属制造；下表的数量都是**单次制造的投入**。

| 去向 | 消耗与产物 | 条件 | 证据 |
|---|---|---|---|
| 炖锅甜味剂 | `sweetener` 1（蜂蜜、蜜脾）／3（蜂王浆），按投入份数累加 | 加载期登记 | [登记](../../../raw/current/scripts/cooking.lua#L97-L98)、[累加](../../../raw/current/scripts/cooking.lua#L217-L231) |
| 太妃糖／南瓜饼／冰淇淋 | 分别要求甜度 ≥3／≥2／≥1 | 冰淇淋另需 `dairy`＋`frozen` | [太妃糖](../../../raw/current/scripts/preparedfoods.lua#L33-L46)、[南瓜饼](../../../raw/current/scripts/preparedfoods.lua#L48-L60)、[冰淇淋](../../../raw/current/scripts/preparedfoods.lua#L379-L387) |
| 糖 `spice_sugar` | 3 蜜 → 2 糖 | 厨师 `professionalchef` | [配方](../../../raw/current/scripts/recipes.lua#L102) |
| 治疗药膏／养蜂书 | 2 蜜／4 蜜 | 科学二本／书匠 | [药膏](../../../raw/current/scripts/recipes.lua#L419)、[书](../../../raw/current/scripts/recipes.lua#L277) |
| 弹弓蜂蜜弹／蜂胶手柄 | 1 蜜 → 20 发／1 蜂王浆 | 沃尔特弹药／手柄技能 | [弹药](../../../raw/current/scripts/recipes.lua#L218)、[手柄](../../../raw/current/scripts/recipes.lua#L241) |
| 幽灵药水·加速／蜘蛛治疗物／突变体治疗物／催吐糖浆 | 各 1–2 蜜 | 温蒂药水师／韦伯／蜘蛛工艺／沃姆伍德 | [加速](../../../raw/current/scripts/recipes.lua#L156)、[蜘蛛治疗](../../../raw/current/scripts/recipes.lua#L328)、[突变体](../../../raw/current/scripts/recipes.lua#L334)、[催吐](../../../raw/current/scripts/recipes.lua#L342) |
| WX 蜜蜂模块 `wx78module_bee` | 1 蜂王浆＋8 扫描数据＋1 理智模块 | 机器人模块工艺 | [配方](../../../raw/current/scripts/recipes.lua#L759) |

- 甜味剂是**按份累加**的普通食材标签：`GetIngredientValues` 对每个投入格把该食材的标签值相加，所以 2 份蜂蜜就是 `sweetener = 2`。[累加](../../../raw/current/scripts/cooking.lua#L217-L231)
- 两件都没有 `cookable`，所以制造配方是它们唯一的"加工"去向；剩下的去向只有生吃与腐坏。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`edible` | 三维与食性；默认 `GENERIC` | 主模拟；两件都未写 `sanityvalue`（honey）或写了正值（royal_jelly） | [[reference/components/edible]]；[默认值](../../../raw/current/scripts/components/edible.lua#L19-L24) |
| 组件：`perishable` | 40 天／10 天，腐坏成 `spoiled_food` | 主模拟；`StartPerishing()` 已开 | [[reference/components/perishable]]；[替换](../../../raw/current/scripts/components/perishable.lua#L277-L319) |
| 组件：`stackable` | 显式 40 | 主模拟 | [[reference/components/stackable]]；[赋值](../../../raw/current/scripts/prefabs/honey.lua#L38-L39) |
| 组件：`harvestable`（宿主侧） | 蜂箱用它持有 `honey` 产量与外观档位 | 只挂在蜂箱／蜂箱 hermit 变体上，不挂在这两件物品 | [[reference/components/harvestable]]；[挂载](../../../raw/current/scripts/prefabs/beebox.lua#L281) |
| 组件：`pollinator`（宿主侧） | 蜜蜂的采粉计数决定蜂箱能否 +1 | 蜂箱只在 `childgoinghome` 里读它 | [[reference/components/pollinator]]；[判定](../../../raw/current/scripts/components/pollinator.lua#L28-L30) |
| 组件：`hauntable` | honey 用 `MakeHauntableLaunchAndPerish`、royal_jelly 用 `MakeHauntableLaunch` | 两者实际都只抛飞：前者的腐坏分支在标准库里被注释掉 | [[reference/components/hauntable]]；[注释段](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337) |
| 标签：`honeyed` | 熊獾的取食优先级依据 | 运行期读取；蜂蜜、蜂王浆、蜜脾与若干炖锅菜带此标签 | [熊獾取食](../../../raw/current/scripts/brains/beargerbrain.lua#L51-L64)、[判定函数](../../../raw/current/scripts/prefabs/bearger.lua#L107-L109) |
| 数据：`cooking.lua` | 炖锅食材：`sweetener` 1（蜂蜜、蜜脾）与 3（蜂王浆） | 加载期注册；两件登记时第三参为 true，会派生 `honey_cooked`／`royal_jelly_cooked` 食材条目，但物品本身没有 `cookable` | [食材](../../../raw/current/scripts/cooking.lua#L97-L98) |
| 关联 Prefab：`spoiled_food` | 两件的腐坏产物 | 主模拟腐坏时生成 | Wiki 待建；[注册](../../../raw/current/scripts/prefabs/spoiledfood.lua#L263-L265) |
| 关联 Prefab：`honeycomb` | 由另一分派（B11）覆盖；本页只把它当蜂王冠拆解与蜂窝掉落里的关联物 | 不在本页展开机制 | 见该分派页面；[掉落](../../../raw/current/scripts/prefabs/beehive.lua#L205) |
| 资源：`anim/honey.zip`／`anim/royal_jelly.zip` | 物品动画包 | 只核验声明，未验证资源内容 | [honey assets](../../../raw/current/scripts/prefabs/honey.lua#L1-L4)、[royal_jelly assets](../../../raw/current/scripts/prefabs/royal_jelly.lua#L1-L4) |

### 共享规则接入（S1–S7）

- **S1 烤制门禁**：目标须带 `cooker`、未 `fueldepleted`，`dangerouscooker` 还要 `doer:HasTag("expertchef")`，骑乘时目标须归自己所有；物品侧还须带 `cookable`。本页两件都没有 `cookable`，整条门禁不适用，只有炖锅侧的 `sweetener`。[门禁](../../../raw/current/scripts/componentactions.lua#L1096-L1105)
- **S2 燃烧掉落**：接入宿主是蜂窝与杀人蜂蜂窝——掉落者已 `burnt`，或正在燃烧且（`fueled == nil` 或 `burnable.ignorefuel`）时，先查 `BURNED_LOOT_OVERRIDES`、再试 `<x>_cooked`／`cooked<x>`，受控燃烧则早退保留原掉落，最后由 `hive`／结构分支换成 `ash`。[改写链](../../../raw/current/scripts/components/lootdropper.lua#L415-L442)
- **S3 晾晒**：不适用——两件都没有 `dryable` 组件与标签，`dryable.product` 分支无从进入。
- **S4 烹饪登记**：`honey`（与蜜脾同行）`sweetener = 1`、`royal_jelly` `sweetener = 3`；登记第三参为 true 只派生 `*_cooked` 食材条目，物品本身没有 `cookable`，条目没有对应实物。[登记](../../../raw/current/scripts/cooking.lua#L97-L98)、[派生](../../../raw/current/scripts/cooking.lua#L46-L72)
- **S5 堆叠与标签**：两件都显式 `TUNING.STACK_SIZE_SMALLITEM`＝40；显式标签只有 `honeyed`，`sweetener` 属炖锅侧的食材标签而非实体标签（`edible`／`perishable` 另会自动加 `edible_GENERIC` 与 `fresh`／`stale`／`spoiled`）。[标签](../../../raw/current/scripts/prefabs/honey.lua#L24)、[堆叠](../../../raw/current/scripts/prefabs/royal_jelly.lua#L39-L40)
- **S6 腐坏替换**：两件都替换成 `spoiled_food`，不涉及鱼专用腐坏体。[替换](../../../raw/current/scripts/components/perishable.lua#L277-L319)
- **S7 索引**：索引由主代理统一登记，本页不改 `index.md`。

## 常见陷阱与限制

- **误区：蜂箱会自己"定时"产蜜**。醒着时没有生长计时器，只有蜜蜂采满花粉回巢才 +1；300 秒一份只在实体进入休眠后成立。[两条路径](#蜂箱的积累路径)
- **边界：烧毁的蜂窝掉灰烬，不掉蜂蜜**。蜂窝与杀人蜂蜂窝都带 `hive` 标签，`DropLoot` 在掉落者已烧毁／正在燃烧且未接燃料组件时按覆盖链改写：本页物品没有 `_cooked` 变体、也不在 `BURNED_LOOT_OVERRIDES` 里；受控燃烧（`GetControlledBurn()`）会早退、保留原掉落并让其闷烧，否则由 `hive`／结构分支换成 `ash`。[燃烧改写](../../../raw/current/scripts/components/lootdropper.lua#L415-L442)、[hive 标签](../../../raw/current/scripts/prefabs/beehive.lua#L168-L171)、[覆盖表](../../../raw/current/scripts/tuning.lua#L4511-L4530)
- **边界：蜂蜜与蜂王浆不能上火烤**。烤制要物品自己带 `cookable`、目标带 `cooker`；两件都没有 `cookable` 组件与标签，只有炖锅侧登记的 `sweetener`。[烤制门禁](../../../raw/current/scripts/componentactions.lua#L1096-L1105)、[食材](../../../raw/current/scripts/cooking.lua#L97-L98)
- **边界：`MakeHauntableLaunchAndPerish` 名字带 Perish，实际不腐坏**。标准库里该分支被注释，作祟只抛飞；`royal_jelly` 用不带 Perish 的版本，行为一致。[注释段](../../../raw/current/scripts/standardcomponents.lua#L1311-L1337)
- **边界：1 份蜂王浆＝3 点甜度**。`sweetener` 按份累加：太妃糖要求 ≥3（1 份蜂王浆即可、蜂蜜要 3 份），南瓜饼与叶片肉舒芙蕾要求 ≥2。[食材值](../../../raw/current/scripts/cooking.lua#L97-L98)、[太妃糖](../../../raw/current/scripts/preparedfoods.lua#L33-L46)、[南瓜饼](../../../raw/current/scripts/preparedfoods.lua#L48-L60)
- **边界：蜂蜜会把熊獾引到箱子边**。它优先拾取地上的 `honeyed` 食物；冰箱／箱子／背包里只要有它能吃的 `honeyed`，就会变成砸开或偷取的目标。[拾取优先](../../../raw/current/scripts/brains/beargerbrain.lua#L51-L64)、[容器目标](../../../raw/current/scripts/brains/beargerbrain.lua#L107-L127)、[动作选择](../../../raw/current/scripts/brains/beargerbrain.lua#L151-L156)
- **未核验**：`beebox_hermit` 的放置来源与珍珠任务链细节；蜂王巢"长成后关掉 workable 并生成蜂王"的完整分支；蜜脾与蜂产品其余机制归 B11／后续页面。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。核对 `prefabs/honey.lua` 与 `prefabs/royal_jelly.lua` 全文、`prefablist.lua` 两条登记；`prefabs/beebox.lua` 的 `MakeBeebox` 全文（harvestable／childspawner／休眠计时／onhammered／两个注册）；`prefabs/beehive.lua` 的掉落与标签、`prefabs/wasphive.lua` 的掉落与标签、`prefabs/bee.lua` 的随机掉落、`prefabs/beequeen.lua` 的共享战利品表、`prefabs/beequeenhive.lua` 的敲巢掉落；`prefabs/hats.lua` 的 `hive` 帽构造与注册；`prefabs/staff_common.lua` 的拆解折算；`components/harvestable.lua`、`components/pollinator.lua`、`components/lootdropper.lua`、`components/edible.lua`、`components/perishable.lua`、`components/stackable.lua` 的相关分支；`componentactions.lua` 的 `HARVEST` 可见性；`actions.lua` 的 `HARVEST.fn`；`cooking.lua` 的食材登记；`recipes.lua` 的糖／药膏／书／弹弓／蜂王冠拆解；`preparedfoods.lua` 的甜味剂菜；`prefabs/bearger.lua` 与 `brains/beargerbrain.lua` 的 `honeyed` 目标；`tuning.lua` 的数值与 `BURNED_LOOT_OVERRIDES`；`components/klaussackloot.lua` 的 `giant_loot3` 与两件不同项抽取；`components/container.lua` 的 `GiveItem` 入槽校验与拒收落地分支；`chinese_s.po` 的两个名称键；`scrapbookdata.lua` 的两条图鉴记录。
  未覆盖：蜜脾与蜂产品其余内容、蜂箱／蜂窝／蜂王本体行为、`beebox_hermit` 的生成链、蜂王巢长成后的分支、克劳斯袋其他奖励档。
- 游戏运行：未运行。天数、秒数与概率均由 `tuning` 常量与权重表推导。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过（见交付回报）。

## 相关链接

- 采集与鲜度：[[reference/components/harvestable]]、[[reference/components/pollinator]]、[[reference/components/perishable]]
- 烹饪侧：[[reference/components/cookable]]、[[reference/components/stackable]]
- 同批页面：[[reference/prefabs/butter|黄油]]、[[reference/prefabs/butterfly-wings|蝴蝶翅膀与月蛾翅膀]]、[[reference/prefabs/butterfly|蝴蝶]]、[[reference/prefabs/milkywhites|乳白物]]

返回 [[reference/prefabs/index|Prefab 索引]]。
