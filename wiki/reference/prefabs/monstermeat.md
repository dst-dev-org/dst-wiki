---
title: "怪物肉三态（monstermeat、cookedmonstermeat、monstermeat_dried）"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "怪物肉三态的负值三维与次要食性、猪王拒收的 goldvalue、生肉独有 selfstacker 自动并堆，以及它作为作祟转换产物时的继承与不可逆边界。"
aliases: ["monstermeat", "cookedmonstermeat", "monstermeat_dried", "怪物肉", "熟怪物肉", "怪物肉干", "怪物肉链"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/meats.lua", "tuning.lua", "constants.lua", "components/edible.lua", "components/perishable.lua", "components/stackable.lua", "components/dryable.lua", "components/cookable.lua", "components/tradable.lua", "components/selfstacker.lua", "components/bait.lua", "components/hauntable.lua", "standardcomponents.lua", "cooking.lua", "preparedfoods.lua", "preparedfoods_warly.lua", "prefabs/pigking.lua", "prefabs/antlion.lua", "prefabs/meatrack2.lua", "containers.lua", "screens/redux/scrapbookdata.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 怪物肉三态（monstermeat、cookedmonstermeat、monstermeat_dried）

## 快速判断

- 职责与适用：本页覆盖 `prefabs/meats.lua` 的 `monstermeat`／`cookedmonstermeat`／`monstermeat_dried`。共同初始化走 `common()`，工厂逐步说明见 [[reference/prefabs/meat#共用工厂与初始化顺序]]；本页只写三态自身的差异，以及它们**作为作祟产物**时的继承规则。[注册](../../../raw/current/scripts/prefabs/meats.lua#L902-L904)
- 优先使用：要改三维或食性看[数值与食物类型](#数值与食物类型)；要改自动并堆看[selfstacker](#selfstacker)；要改作祟转换的发起方看 [[reference/prefabs/meat#作祟转换]]，本页只是目标形态。
- 重要副作用：三维生命与理智为负（必带 `badfood`）；`tradable.goldvalue` 被置 0，猪王拒收，蚁狮另有 `rocktribute` 门禁（三态从未登记）；熟态鲜度反而比熟大肉更长（`PERISH_SLOW`）；只有生肉能自动并堆。
- 修改边界：转换的发起与概率在大肉页；晾肉架、`spoiled_food`、锅各有归属页待建；`quagmire` 分支不在本快照内，不覆盖。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `monstermeat` | `monster()` → `common("monstermeat","meat_monster","idle",{"monstermeat","selfstacker"},{product="monstermeat_dried",time=DRY_FAST},{product="cookedmonstermeat"})` | bank 是 `monstermeat`、build 是 `meat_monster`（两者不同名）；播 `idle`；主模拟段设次要食性、负三维、`goldvalue = 0`、`floater` 偏移 0.05，并 `AddComponent("selfstacker")`。[构造](../../../raw/current/scripts/prefabs/meats.lua#L282-L303) |
| `cookedmonstermeat` | `cookedmonster()` → `common("monstermeat","meat_monster","cooked",{"monstermeat"})`：无 `dryable`、无 `cookable` | 播 `cooked`；`PERISH_SLOW`（比熟大肉长）；`goldvalue = 0`；没有 `selfstacker`。[构造](../../../raw/current/scripts/prefabs/meats.lua#L306-L325) |
| `monstermeat_dried` | `driedmonster()` → `common("meat_rack_food","meat_rack_food","idle_dried_monster",{"monstermeat"},{isdried = true})` | bank／build 换成 `meat_rack_food`；第 5 参只含 `isdried`，故无 `dryable`／`cookable` 组件与标签，但仍带 `lureplant_bait`；`PERISH_PRESERVED`；无 `selfstacker`。[构造](../../../raw/current/scripts/prefabs/meats.lua#L327-L341) |

### 名称核验

`NAMES.MONSTERMEAT`＝**怪物肉**、`NAMES.COOKEDMONSTERMEAT`＝**熟怪物肉**、`NAMES.MONSTERMEAT_DRIED`＝**怪物肉干**（英文 msgid 依次 Monster Meat／Cooked Monster Meat／Monster Jerky）。三键各唯一命中，均无 `NAME_DETAIL_EXTENTION`。[MONSTERMEAT](../../../raw/current/scripts/languages/chinese_s.po#L345590-L345592)、[COOKEDMONSTERMEAT](../../../raw/current/scripts/languages/chinese_s.po#L341640-L341642)、[MONSTERMEAT_DRIED](../../../raw/current/scripts/languages/chinese_s.po#L345595-L345597)

## 数值与食物类型

时间基准：`total_day_time`＝480 秒、`perish_warp`＝1。[基准](../../../raw/current/scripts/tuning.lua#L19-L20)、[warp](../../../raw/current/scripts/tuning.lua#L47-L47)

| 注册 | 饥饿／生命／理智 | 鲜度 | 堆叠 | 猪王 | 自并堆 |
|---|---|---|---|---|---|
| `monstermeat` | 18.75（`CALORIES_MEDSMALL`）／−20（`−HEALING_MED`）／−15（`−SANITY_MED`） | `PERISH_FAST`＝6 天＝2880 秒 | 20 | 拒收（`goldvalue` 0） | 有 |
| `cookedmonstermeat` | 18.75／−3（`−HEALING_SMALL`）／−10（`−SANITY_SMALL`） | `PERISH_SLOW`＝15 天＝7200 秒 | 20 | 拒收（`goldvalue` 0） | 无 |
| `monstermeat_dried` | 18.75／−3／−5（`−SANITY_TINY`） | `PERISH_PRESERVED`＝20 天＝9600 秒 | 20 | 拒收（`goldvalue` 0） | 无 |

图鉴数值与源码逐项一致（`perishable` 2880／7200／9600，堆叠均为 20）：[monstermeat](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L922-L922)、[cookedmonstermeat](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L346-L346)、[monstermeat_dried](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L923-L923)

- **次要食性是本链特有**：三态都设 `edible.secondaryfoodtype = FOODTYPE.MONSTER`；`edible` 的属性 setter 会同时维护主类型标签 `edible_MEAT` 与次要标签 `edible_MONSTER`，并断言两者不能相同。大肉三态不设次要类型。[setter](../../../raw/current/scripts/components/edible.lua#L9-L17)、[赋值](../../../raw/current/scripts/prefabs/meats.lua#L290-L290)、[常量](../../../raw/current/scripts/constants.lua#L2026-L2047)
- **负值让三态都带 `badfood`**：`healthvalue < 0` 或 `sanityvalue < 0` 就挂该标签，本链两值皆负，所以熟态与干制品也不例外（大肉只有生肉带）。[判定](../../../raw/current/scripts/components/edible.lua#L1-L7)
- **干制品反而最轻**：三态饥饿都是 18.75；怪物肉干生命 −3（与熟态相同）、理智只 −5（熟态 −10、生肉 −15），代价是它没有 `cookable`，不能再加工。

## 消费端

- **猪王拒收是 `goldvalue` 被置 0；蚁狮是另一套门禁**：`common()` 给的是 `GOLD_VALUES.MEAT`＝1，三个工厂随后都改成 0，而猪王的接受判定要求 `goldvalue > 0`（另有鱼人交付者门禁），所以三态换不到金块。蚁狮不看 `goldvalue`：它的 `AcceptTest` 要求 `tradable.rocktribute ~= nil` 且 `> 0`（且当前离线／无待发奖励），而 `tradable` 组件默认 `rocktribute = nil`、`meats.lua` 从不设它，三态连贡品都递不上去；`goldvalue` 只在贡品已被接受后决定谢礼（>0 才给 `townportaltalisman`）。[置 0](../../../raw/current/scripts/prefabs/meats.lua#L296-L296)、[猪王](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)、[蚁狮门禁](../../../raw/current/scripts/prefabs/antlion.lua#L115-L119)、[谢礼](../../../raw/current/scripts/prefabs/antlion.lua#L134-L142)、[默认值](../../../raw/current/scripts/components/tradable.lua#L1-L7)
- **烹饪锅按"肉＋怪物"计价**：`monstermeat` 登记为 `meat=1, monster=1`，并自动派生 `monstermeat_cooked`（`precook=1`）与 `monstermeat_dried`（`dried=1`）；`cookedmonstermeat` 靠别名表映射成 `monstermeat_cooked` 才算食材。[登记](../../../raw/current/scripts/cooking.lua#L106-L109)、[别名](../../../raw/current/scripts/cooking.lua#L178-L187)
- **先过硬条件，再比同锅优先级**：怪物值至少 2 且没有 inedible 时才通过 `monsterlasagna` 的配方判定（`tags.monster >= 2 and not tags.inedible`，优先级 10），成品三维为生命 −20／饥饿 37.5／理智 −20，并带 `monstermeat` 配方标签与 `FOODTYPE.MONSTER` 次要类型；注意后半个条件：加一根树枝（`twigs` 登记 `inedible=1`）就会让整个判定不成立。同时满足时还会被更高优先级压过去：沃利（Warly）便携锅 `portablecookpot` 的 `monstertartare` 用**同一个 test**、优先级 30（饥饿 62.5／生命 −20／理智 −20），`koalefig_trunk` 用「象鼻（`trunk_summer`／`trunk_cooked`／`trunk_winter` 任一）＋无花果（`fig`／`fig_cooked`）」、优先级 40。所以在**这些示例里**出菜顺序是 40 象鼻 > 30 鞑靼 > 10 千层饼。普通锅也有反例：两块怪物肉加两块生鱼肉 `fishmeat`（登记 `meat=1, fish=1`）得到 `meat=4, fish=2`，同时满足 `surfnturf`（`tags.meat >= 2.5 and tags.fish >= 1.5 and not tags.frozen`，优先级 30），实际出海鲜牛排。要指定千层饼得避开这些更高优先级的配方，或改动优先级。[千层饼](../../../raw/current/scripts/preparedfoods.lua#L308-L321)、[鞑靼](../../../raw/current/scripts/preparedfoods_warly.lua#L163-L177)、[象鼻](../../../raw/current/scripts/preparedfoods.lua#L904-L917)、[海鲜牛排](../../../raw/current/scripts/preparedfoods.lua#L661-L674)、[树枝](../../../raw/current/scripts/cooking.lua#L134-L134)、[鱼肉登记](../../../raw/current/scripts/cooking.lua#L117-L117)
- **晾肉架**：只有生怪物肉的 `dryable` 生效（product=`monstermeat_dried`，time=`DRY_FAST`＝480 秒）；干制品无 `dryable` 标签，不能再上架。[接线](../../../raw/current/scripts/prefabs/meats.lua#L284-L284)、[筛选](../../../raw/current/scripts/containers.lua#L234-L244)

## 作祟转换的目标形态

- **发起方不在本页**：`AddMonsterMeatChange` 只接线在大肉三态（`meat`→`monstermeat`、`cookedmeat`→`cookedmonstermeat`、`meat_dried`→`monstermeat_dried`）；怪物肉三态内部没有该调用，被作祟时只走 `MakeHauntableLaunchAndPerish` 的抛飞。[cooked 接线](../../../raw/current/scripts/prefabs/meats.lua#L357-L357)、[dried 接线](../../../raw/current/scripts/prefabs/meats.lua#L378-L378)、[raw 接线](../../../raw/current/scripts/prefabs/meats.lua#L398-L398)
- **继承什么**：转换在 25% 概率下命中后，新件同坐标生成，继承旧件的**件数**（双方都有 `stackable` 且旧件是堆时）、**湿度／湿润**与**鲜度百分比**，随后被 `spawnedfromhaunt` 回调击飞；旧件 `persists = false` 并在下一帧移除。逐条与证据见 [[reference/prefabs/meat#作祟转换]]。[函数体](../../../raw/current/scripts/prefabs/meats.lua#L104-L132)
- **没有回头路**：`monstermeat` 自带 `cookable`，能烤成 `cookedmonstermeat`，但熟态与干制品都没有 `cookable`；生肉虽带 `dryable`，产物仍是怪物肉干，熟态与干制品则连 `dryable` 都没有。检索范围内也没有任何"怪物肉→普通肉"的生成点。

## selfstacker

- **只有生怪物肉**：`selfstacker` 先写在 `common()` 的 tags 数组里（pristine 之前），再在 master sim 段 `AddComponent("selfstacker")`；熟态与干制品的 tags 里没有它，也没有组件。[标签](../../../raw/current/scripts/prefabs/meats.lua#L284-L284)、[组件](../../../raw/current/scripts/prefabs/meats.lua#L300-L300)
- **何时触发**：`OnEntityWake`（实体从睡眠被唤醒，通常是玩家靠近）时若 `CanSelfStack()` 成立，就在 0–0.1 秒随机延迟后尝试并堆。成立条件：`bait` 空闲（没被陷阱占着）、没有在燃烧、`stackable` 未满、未被任何库存持有、速度平方 < 1、且当前没有 `stackpartner`。[条件](../../../raw/current/scripts/components/selfstacker.lua#L18-L26)、[唤醒](../../../raw/current/scripts/components/selfstacker.lua#L64-L73)
- **怎么找目标**：以自身为中心半径 20，必须带 `selfstacker` 标签、排除 `outofreach`，并用 `stackable:CanStackWith` 校验（同 prefab、同皮肤）；找到后只搬自己空位数量的那一份（`RoomLeft()` → 对方 `Get(num)` → `Put`）。[查找](../../../raw/current/scripts/components/selfstacker.lua#L36-L62)
- **效果**：地上的散落怪物肉会自己并到 20 上限，熟态与干制品只能手动堆（但可手动堆满 20）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：edible | 负三维；`secondaryfoodtype = MONSTER` 追加 `edible_MONSTER` 标签 | 三注册都有；仅主模拟 | [[reference/components/edible#关键接口]]；[L290-L293](../../../raw/current/scripts/prefabs/meats.lua#L290-L293) |
| 组件：selfstacker | 生肉自动并堆；`OnRemoveEntity` 时摘掉 `selfstacker` 标签 | 仅 `monstermeat` | [[reference/components/selfstacker#关键接口]]；[L300-L300](../../../raw/current/scripts/prefabs/meats.lua#L300-L300) |
| 组件：tradable | `goldvalue` 被三态都改成 0（猪王因此拒收）；`rocktribute` 从未设置，蚁狮另按它门禁 | 三注册都有 | [[reference/components/tradable#关键接口]]；[L296](../../../raw/current/scripts/prefabs/meats.lua#L296-L296)、[L313](../../../raw/current/scripts/prefabs/meats.lua#L313-L313)、[默认值](../../../raw/current/scripts/components/tradable.lua#L1-L7) |
| 组件：perishable | `SetPerishTime`＋`StartPerishing`，腐坏换 `spoiled_food` | 2880／7200／9600 秒 | [[reference/components/perishable#关键接口]]；[L294-L294](../../../raw/current/scripts/prefabs/meats.lua#L294-L294) |
| 组件：cookable | 仅 `monstermeat` 挂，product=`cookedmonstermeat` | 熟态与干制品无 | [[reference/components/cookable#关键接口]]；[L209-L212](../../../raw/current/scripts/prefabs/meats.lua#L209-L212) |
| 组件：dryable | 仅 `monstermeat` 挂，product=`monstermeat_dried`、time=`DRY_FAST` | 干制品无该标签，不能再上架 | [[reference/components/dryable#关键接口]]；[L201-L207](../../../raw/current/scripts/prefabs/meats.lua#L201-L207) |
| 组件：hauntable | 只有 `MakeHauntableLaunchAndPerish`，无自定义转换反应 | 三注册都有 | [[reference/components/hauntable#关键接口]]；[L218-L219](../../../raw/current/scripts/prefabs/meats.lua#L218-L219) |
| 数据：`edible.secondaryfoodtype`／`edible_MONSTER` | 只决定实体食性标签：`edible` 的 setter 维护主类型 `edible_MEAT` 与次要标签 `edible_MONSTER`，吃者侧用 `<TYPE>_eater` 配对判断能否吃。**只改次要食性不会改变锅里的怪物值** | 只影响食用判定，与配方无关 | [[reference/components/edible#关键接口]]；[setter](../../../raw/current/scripts/components/edible.lua#L9-L17)、[吃者配对](../../../raw/current/scripts/componentactions.lua#L1185-L1185)、[常量](../../../raw/current/scripts/constants.lua#L2043-L2043) |
| 数据：`cooking.lua` 的 `tags.monster` | 烹饪锅按**食材名与别名**查 `ingredients` 表累加标签，`monstermeat` 登记 `meat=1, monster=1`；与 `FOODTYPE` 无关 | 只影响配方 `test` 的 `tags.monster` 统计 | [登记](../../../raw/current/scripts/cooking.lua#L107-L107)、[取值](../../../raw/current/scripts/cooking.lua#L217-L231)、[配方 test](../../../raw/current/scripts/preparedfoods.lua#L310-L310) |
| 关联 Prefab：`meat`／`cookedmeat`／`meat_dried` | 作祟转换的来源；本页三态是它们的替换结果 | 25% 概率，仅主模拟 | [[reference/prefabs/meat]]；[接线](../../../raw/current/scripts/prefabs/meats.lua#L398-L398) |
| 关联 Prefab：`meatrack` | 生怪物肉的干制宿主，只收 `dryable` 标签且不接堆叠 | 3 槽；另有 `meatrack_hermit`（1 槽）与 `meatrack_hermit_multi`（9 槽） | Wiki 待建（晾肉架）；[注册](../../../raw/current/scripts/prefabs/meatrack2.lua#L584-L592) |
| 资源：`anim/meat_monster.zip` | 生／熟用 `monstermeat` bank ＋ `meat_monster` build；干制改用 `meat_rack_food` | 原版资源，不随页分发 | [assets](../../../raw/current/scripts/prefabs/meats.lua#L4-L4) |

## 典型接入

### 让猪王重新收怪物肉

- 前提与选择：服务端 `modmain.lua` 的 postinit。三态的 `goldvalue` 是工厂在主模拟里改成 0 的，所以要在构造之后改回来；不要改 `GOLD_VALUES.MEAT`，那会连普通肉、鱼、猪皮等一起变。
- 生效与撤销：只影响之后构造的实体（含读档重建），已有实例保持旧值直到重建；不改任何产出与掉落。改它**只影响猪王的换金判定**——`AcceptTest` 先拒鱼人交付者（`giver:HasTag("merm")` 且非 `pig_token` 时直接返回 `nil`），再看 `goldvalue > 0`／万圣糖果／`pig_token`，所以改金值对鱼人无效。蚁狮的接受门禁是 `tradable.rocktribute > 0`，与 `goldvalue` 无关；要让蚁狮收下怪物肉，得另设 `rocktribute`，而且只有同时 `goldvalue > 0` 时它才会给 `townportaltalisman`。[猪王门禁](../../../raw/current/scripts/prefabs/pigking.lua#L598-L606)、[蚁狮门禁](../../../raw/current/scripts/prefabs/antlion.lua#L115-L119)、[谢礼](../../../raw/current/scripts/prefabs/antlion.lua#L134-L142)

```lua
-- modmain.lua：让怪物肉三态在猪王那里值 1 块金块
local NAMES = { "monstermeat", "cookedmonstermeat", "monstermeat_dried" }

for _, name in ipairs(NAMES) do
    AddPrefabPostInit(name, function(inst)
        if inst.components.tradable ~= nil then
            inst.components.tradable.goldvalue = GLOBAL.TUNING.GOLD_VALUES.MEAT
        end
    end)
end
```

- 调整与验证：`GOLD_VALUES.MEAT`＝1；想换更多就写 2 或 3（猪王按该值循环发金块）。验证时给猪王一件怪物肉，看它是否进入 `cointoss` 状态并掉出对应数量的 `goldnugget`。

## 常见陷阱与限制

- **边界：库存陷阱是"整叠"的。** 作祟命中时件数被完整继承，一堆 20 个 `meat_dried` 会一次变成 20 个 `monstermeat_dried`；结果是既不能烤（无 `cookable`）、也不能上架（无 `dryable`）、猪王还不收（`goldvalue` 0）。别把肉干堆常年摆在野外架子旁。
- **误区：熟怪物肉比熟大肉更容易坏。** 相反：`cookedmonstermeat` 用 `PERISH_SLOW`（7200 秒），`cookedmeat` 用 `PERISH_MED`（4800 秒）；干制品两者都是 `PERISH_PRESERVED`。
- **误区：怪物肉也能被作祟换成别的。** 转换只登记在大肉三态，怪物肉三态被作祟只有抛飞；也没有"怪物肉→普通肉"的反向路径。
- **边界：`selfstacker` 不是"随时并堆"。** 它要求实体被唤醒、速度接近静止、没被持有、`bait` 空闲且未满；被陷阱当诱饵占用时不会并堆，手里拿着的也不会。
- **边界：三态都带 `badfood`。** 判定只看生命／理智是否为负，熟态与干制品一样带；仅按"烤过就安全"推断会出错。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。通读 `prefabs/meats.lua` 的三个怪物肉工厂、`common()` 与 `AddMonsterMeatChange`，核对 `tuning.lua` 的 `HEALING_*`／`CALORIES_MEDSMALL`／`SANITY_*`／`PERISH_*`／`GOLD_VALUES`／`STACK_SIZE_MEDITEM`，`edible`／`perishable`／`stackable`／`selfstacker`／`bait`／`tradable`／`cookable`／`dryable`／`hauntable` 的相关分支，`cooking.lua` 的食材与别名，`preparedfoods.lua` 的 `monsterlasagna`，`pigking` 的 `goldvalue` 接受与发金块、`antlion` 的 `AcceptTest`（`rocktribute` 门禁）与 `goldvalue` 谢礼分支，`tradable` 的默认字段，`containers` 的架子筛选。
  名称核验：`chinese_s.po` 的 `NAMES.MONSTERMEAT`＝怪物肉、`NAMES.COOKEDMONSTERMEAT`＝熟怪物肉、`NAMES.MONSTERMEAT_DRIED`＝怪物肉干；三键各唯一命中。
  未覆盖：`quagmire` 模式；怪物肉作为食材的完整配方清单（只核了 `monsterlasagna`、沃利 `monstertartare`、`koalefig_trunk`、`surfnturf` 与 `shroombait` 的相关判定）；模组注入。
- 游戏运行：未运行。秒数、堆叠与金块数为 tuning 与工厂推导（`total_day_time`＝480 秒）。
- 结构检查：见交付回报的 `validate_page.py` 结果。

## 相关链接

- 共用工厂与同批叶页：[[reference/prefabs/meat]]、[[reference/prefabs/smallmeat]]、[[reference/prefabs/fishmeat|鱼肉两链]]
- 可食与鲜度：[[reference/components/edible]]、[[reference/components/perishable]]、[[reference/components/stackable]]
- 自动并堆：[[reference/components/selfstacker]]、[[reference/components/bait]]
- 兑换与作祟：[[reference/components/tradable]]、[[reference/components/hauntable]]

返回 [[reference/prefabs/index|Prefab 索引]]。
