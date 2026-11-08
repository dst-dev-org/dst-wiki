---
title: "高脚鸟蛋（tallbirdegg、tallbirdegg_cracked、tallbirdegg_cooked）与高脚鸟巢"
created: "2026-11-09"
updated: "2026-11-09"
type: prefab
summary: "prefabs/tallbirdegg.lua 的三段蛋（高脚鸟蛋／孵化中／煎高脚鸟蛋）加 tallbirdnest：hatchable 破壳换实体、玩家靠近才出小鸟、cookable 烤成煎蛋与 egg=4 食材，以及巢的 pickable／childspawner 供蛋条件。"
aliases: ["tallbirdegg", "tallbirdegg_cracked", "tallbirdegg_cooked", "tallbirdnest", "高脚鸟蛋", "孵化中的高脚鸟蛋", "煎高脚鸟蛋", "高脚鸟巢"]
tags: [prefab, item, food]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/tallbirdegg.lua", "prefabs/tallbirdnest.lua", "prefabs/tallbird.lua", "prefabs/walter.lua", "prefabs/klaus_sack.lua", "components/klaussackloot.lua", "components/hatchable.lua", "components/pickable.lua", "components/playerprox.lua", "components/childspawner.lua", "components/lootdropper.lua", "components/stackable.lua", "cooking.lua", "preparedfoods.lua", "componentactions.lua", "prefabs/cookpot.lua", "scenarios/chest_food.lua", "scenarios/chest_random_good.lua", "tuning.lua", "languages/chinese_s.po", "screens/redux/scrapbookdata.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 高脚鸟蛋（tallbirdegg、tallbirdegg_cracked、tallbirdegg_cooked）与高脚鸟巢

## 快速判断

- 职责与适用：`prefabs/tallbirdegg.lua` 返回三个注册并共用 `commonfn`；生蛋不可堆叠、不会腐坏，靠 `hatchable` 破壳并最终出小鸟，可烤成 `tallbirdegg_cooked`；巢 `tallbirdnest` 是可采摘设施，产物就是 `tallbirdegg`。[三个注册](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L284-L286)、[巢的采摘配置](../../../raw/current/scripts/prefabs/tallbirdnest.lua#L164-L169)
- 优先使用：要改孵化节奏看[孵化与碎裂](#孵化与碎裂)；要改数值／吃法看[数值与获得](#数值与获得)；要改巢的产蛋看[高脚鸟巢](#高脚鸟巢)。
- 重要副作用：`crack` 只把蛋换成 `tallbirdegg_cracked`；`hatch` 到点时组件会立刻调 `CheckHatch`，但还要玩家在附近、蛋未被持有、落在可视地面才生成 `smallbird`——玩家已在旁边就当场出雏，否则等靠近或放下重试。**计时完成不等于必定出雏，还要上述条件成立。**[换蛋](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L104-L126)、[出雏检查](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L36-L47)
- 修改边界：巢只管到“有蛋／没蛋”和一只高脚鸟的归属；高脚鸟本体行为、`smallbird`、`lightninggoathorn` 一类关联对象不在本页；Walter 的吃蛋反应只在此处关联（见[常见陷阱](#常见陷阱与限制)）。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `tallbirdegg` | `normalfn` → `defaultfn("egg")` → `commonfn(anim, true, true)`；`SetPristine` 后非主模拟返回 | 完整蛋：`cattoy`＋`tallbirdegg`＋`cookable` 标签；主模拟挂 `inspectable`／`inventoryitem`／`edible`（`MEAT`）／`cookable`（产物 `tallbirdegg_cooked`）／`hatchable`；另外写 `hatchable` 的温度偏好、`inventoryitem` 的放下／拾起回调与 `OnLoadPostPass`（玩家靠近回调只在裂蛋）。[共用工厂](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L152-L198)、[完整蛋](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L200-L231) |
| `tallbirdegg_cracked` | `crackedfn` → `defaultfn("idle_happy")`，随后改初始状态 | 裂蛋：多 `donotautopick` 标签，`hatchable.state` 直接设为 `comfy`，另挂 `playerprox`（近 4／远 6）负责“玩家靠近才出雏”，被吃时对吃者说专属台词；它仍继承 `cookable`，可烤成煎蛋，烤后原蛋消失、不能再孵。[裂蛋](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L237-L255) |
| `tallbirdegg_cooked` | `cookedfn` → `commonfn("cooked")`（**不传** `cookable`、不给音效） | 煎蛋：无 `cookable`／`hatchable`，改挂 `stackable`（不写 `maxsize`，组件默认 20）与 `perishable`（`PERISH_FAST` → `spoiled_food`）。[煎蛋](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L257-L282) |
| `tallbirdnest` | `tallbirdnest.lua` 的 `fn`；`SetPristine` 后非主模拟返回 | 世界设施：`pickable`（产物 `tallbirdegg`）＋`childspawner`（`tallbird`，上限 1）＋可燃＋可作祟；蛋的再生由高脚鸟在场驱动。[巢](../../../raw/current/scripts/prefabs/tallbirdnest.lua#L139-L199) |

`commonfn` 的固定动作：bank `egg`／build `tallbird_egg`；`cattoy`／`tallbirdegg` 标签与（可选）`cookable` 标签在 `SetPristine` 之前加，玩法组件只在主模拟分支加（两端共有标签与 `floater`）；`edible.foodtype = FOODTYPE.MEAT`。另有一个源码事实：`commonfn` 的 `anim` 形参在函数体内**未被使用**，始终先播 `egg`，熟态动画只在服务端的 `cookedfn` 里切到 `cooked`，客户端副本不执行该行（实际两端显示未运行验证）。[动画](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L164-L166)、[熟态动画](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L263-L269)

### 名称核验

简中实显：`tallbirdegg`＝**高脚鸟蛋**、`tallbirdegg_cracked`＝**孵化中的高脚鸟蛋**、`tallbirdegg_cooked`＝**煎高脚鸟蛋**、`tallbirdnest`＝**高脚鸟巢**；四个键各命中一次，注册名与 `STRINGS.NAMES` 键同名。[高脚鸟蛋](../../../raw/current/scripts/languages/chinese_s.po#L350235-L350237)、[裂蛋](../../../raw/current/scripts/languages/chinese_s.po#L350245-L350247)、[煎蛋](../../../raw/current/scripts/languages/chinese_s.po#L350240-L350242)、[巢](../../../raw/current/scripts/languages/chinese_s.po#L350250-L350252)

## 数值与获得

| 注册 | 饥饿／生命／理智 | 鲜度 | 堆叠 | 其它 |
|---|---|---|---|---|
| `tallbirdegg` | 25／3／0（`CALORIES_MED`／`HEALING_SMALL`） | **无 `perishable`**，不会腐坏 | **无 `stackable`**，一格一件 | `cattoy`、`tallbirdegg`、`cookable`；烤制 → `tallbirdegg_cooked` |
| `tallbirdegg_cracked` | 25／3／0 | 无 | 无 | 覆盖 `edible:SetOnEatenFn`，吃后说 `EAT_FOOD.TALLBIRDEGG_CRACKED`；仍继承 `cookable`，可烤成煎蛋（烤后不能再孵） |
| `tallbirdegg_cooked` | 37.5／0／0（`CALORIES_LARGE`） | `PERISH_FAST`＝6 天＝2880 秒 → `spoiled_food` | 20（组件默认 `STACK_SIZE_MEDITEM`） | 无 `cookable`／`hatchable` |

数值来源：[完整蛋](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L207-L208)、[煎蛋](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L271-L277)、[堆叠默认](../../../raw/current/scripts/components/stackable.lua#L28-L28)、[常量](../../../raw/current/scripts/tuning.lua#L82-L83)、[鲜度与热量](../../../raw/current/scripts/tuning.lua#L2046-L2062)。采集册同项写作 `hungervalue=25／37.5`、`healthvalue=3／0`、`tallbirdegg_cooked` 的 `stacksize=20`／`perishable=2880`，且 `tallbirdegg` 没有 `stacksize`／`perishable` 字段，与源码一致。[采集册](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1493-L1496)

### 获得路径

| 来源 | 成立条件 | 说明与证据 |
|---|---|---|
| 高脚鸟巢 | 巢处于“有蛋”状态（`pickable.canbepicked`） | 摘取得 `tallbirdegg`；被摘后巢置空并开始等下一枚蛋。[采摘配置](../../../raw/current/scripts/prefabs/tallbirdnest.lua#L164-L169)、[摘取回调](../../../raw/current/scripts/prefabs/tallbirdnest.lua#L51-L63) |
| 克劳斯袋 | 用真钥匙开启克劳斯袋，从巨人战利品池抽两格 | `giant_loot3` 共 9 项、含 `tallbirdegg`；`i1`／`i2` 两次不重复抽取，故每袋某个指定项出现概率为 2/9；战利品包成礼物／包裹后投出。[池](../../../raw/current/scripts/components/klaussackloot.lua#L23-L34)、[抽取](../../../raw/current/scripts/components/klaussackloot.lua#L115-L124)、[玩家开启](../../../raw/current/scripts/prefabs/klaus_sack.lua#L52-L76) |
| 布景箱子 | 世界生成的布景箱按物品表填充 | `chest_food` 与 `chest_random_good` 各含 1 枚。[食物箱](../../../raw/current/scripts/scenarios/chest_food.lua#L37-L38)、[随机箱](../../../raw/current/scripts/scenarios/chest_random_good.lua#L32-L32) |

用途：① 烹饪锅登记 `egg = 4`，`cancook` 派生 `tallbirdegg_cooked` 条目（`egg = 4`＋`precook = 1`）[食材值](../../../raw/current/scripts/cooking.lua#L129-L129)、[派生](../../../raw/current/scripts/cooking.lua#L46-L72)；② 专属料理“苏格兰高鸟蛋”（`talleggs`）要求 `names.tallbirdegg` 且 `tags.veggie >= 1`，优先级 10[配方](../../../raw/current/scripts/preparedfoods.lua#L1020-L1032)；③ 烤制得煎蛋（S1 见下节）。

### 孵化与碎裂

三段式状态机（`hatchable` 的接口与判定细节见 [[reference/components/hatchable]]）：

1. `unhatched`：只有**夜间**且半径 `HATCH_CAMPFIRE_RADIUS`＝4 内有正热源（`HASHEATER` 实体的热值合计＞0）时才累计；白天、黄昏与夜间无火都不累计并清零（`SetHeaterPrefs(false, nil, true)` 让 `wants_heater` 仅在夜间为真）。满 `SMALLBIRD_HATCH_CRACK_TIME`＝10 秒触发 `crack`。[偏好](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L210-L216)、[阈值](../../../raw/current/scripts/tuning.lua#L1240-L1242)
2. `crack`：生成 `tallbirdegg_cracked` 并播放 `crack`／`idle_happy`，随后移除完整蛋——**“碎裂”只换来裂蛋实体，不直接换小鸟**。[回调](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L109-L115)
3. `comfy`：裂蛋按舒适累计到 `SMALLBIRD_HATCH_TIME`＝3 天触发 `hatch`；过热／过冷则累计不适，满 `SMALLBIRD_HATCH_FAIL_TIME`＝夜间时长的一半（30 秒）进入 `dead`。[计时](../../../raw/current/scripts/components/hatchable.lua#L167-L189)、[档位](../../../raw/current/scripts/tuning.lua#L1240-L1242)
4. `hatch`：组件到 `hatchtime` 时在状态回调里立刻调 `CheckHatch`；后者再要求玩家在附近（`playerprox` 近 4／远 6）、蛋未被持有、位于可视地面，才生成 `smallbird`、切它的 `hatch` 状态、发成就 `hatch_tallbirdegg` 并移除蛋。玩家已经在旁边就当场出雏；不在旁边才等 `playerprox` 的靠近回调或放下蛋时重试。[出雏](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L24-L47)、[靠近回调](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L248-L250)
5. `dead`：过热在 20 帧后掉 `cookedsmallmeat`，过冷在 30 帧后掉 `wetgoop`；同时置 `canbepickedup = false`、`persists = false`，动画结束后移除。[死亡分支](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L88-L98)、[掉落表](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L14-L22)、[清理](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L127-L143)

放入背包会 `StopUpdating`（暂停孵化），放下后 `StartUpdating` 并立即重试 `CheckHatch`；`OnLoadPostPass` 再检查一次是否被持有，避免加载顺序让背包里的蛋继续计时。[放入／放下](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L58-L78)

### 高脚鸟巢

- **产蛋归属**：巢挂 `childspawner`（`childname = "tallbird"`、离屏生成、上限 1、再生周期 `5*16*SEG_TIME`＝2400 秒＝5 天）[生成器](../../../raw/current/scripts/prefabs/tallbirdnest.lua#L175-L183)；被摘蛋或巢空时 `childspawner.noregen = true`，`onpicked` 还会让巢内高脚鸟把摘蛋者设为攻击目标[摘取](../../../raw/current/scripts/prefabs/tallbirdnest.lua#L51-L68)。
- **蛋靠鸟补**：`pickable:SetUp("tallbirdegg", nil)` 让本巢的 `baseregentime` 为 `nil`，`MakeEmpty` 不会排计时器，`Regen` 只由 `ForceLay` 调用。`DoNesting` 到点时先置 `readytolay`，**若巢实体本身已入睡就立刻 `ForceLay`**；`ForceLay` 再遍历 `childspawner.childrenoutside`，只有 16 单位内有自己的高脚鸟才补回一枚蛋；巢之后才入睡时由 `entitysleep` 事件再触发一次。判据是「巢入睡＋附近有鸟」，不是「高脚鸟入睡」[计时条件](../../../raw/current/scripts/components/pickable.lua#L426-L448)、[补蛋](../../../raw/current/scripts/prefabs/tallbirdnest.lua#L23-L49)、[入睡补蛋](../../../raw/current/scripts/prefabs/tallbirdnest.lua#L85-L89)。
- **高脚鸟建巢**：没有 `homeseeker` 且无战斗目标、脚下是岩石或泥土地皮、附近没有其它高脚鸟时，高脚鸟会生成新巢、接管归属、清空蛋并开始等下一枚；它也会守着自家巢的“偷蛋贼”目标[建巢](../../../raw/current/scripts/prefabs/tallbird.lua#L126-L144)、[护蛋目标](../../../raw/current/scripts/prefabs/tallbird.lua#L96-L104)。
- **春季小鸟**：春季每个巢至多补出一只 `smallbird`（`spawnedsmallbirdthisseason` 标记，2–8 天随机延迟），非春季重置标记[春季](../../../raw/current/scripts/prefabs/tallbirdnest.lua#L111-L137)。
- **存读**：`readytolay`、`havespawned` 与距下次筑巢剩余时间写入存档；读档没有 `timetonest` 或尚未就绪时重新 `StartNesting`[存档](../../../raw/current/scripts/prefabs/tallbirdnest.lua#L91-L109)。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：hatchable | 破壳／孵化／不适三段计时，回调负责换实体与出雏 | 仅主模拟加；完整蛋与裂蛋挂，煎蛋不挂 | [[reference/components/hatchable]]；[配置](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L210-L216)、[回调](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L104-L144) |
| 组件：playerprox | 裂蛋用近 4／远 6 的邻近回调反复触发 `CheckHatch` | 仅裂蛋挂；完整蛋没有 | [[reference/components/playerprox]]；[挂载](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L248-L250) |
| 组件：cookable | `product = "tallbirdegg_cooked"` | 仅完整蛋；标签两端、组件主模拟 | [[reference/components/cookable]]；[接线](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L192-L195) |
| 组件：pickable | 巢的产物是 `tallbirdegg`，再生参数 `nil` | 主模拟；`Regen` 只由高脚鸟触发 | [[reference/components/pickable]]；[配置](../../../raw/current/scripts/prefabs/tallbirdnest.lua#L164-L169) |
| 组件：childspawner | 巢绑定 1 只 `tallbird`，离屏生成、5 天再生 | 主模拟；注册的是 `SetSpawnedFn(onvacate)`（子代生成后回调，不是 vacate 回调），未注册 `SetVacateFn` | [[reference/components/childspawner]]；[挂载](../../../raw/current/scripts/prefabs/tallbirdnest.lua#L175-L183)、[回调语义](../../../raw/current/scripts/components/childspawner.lua#L281-L283)、[调用时机](../../../raw/current/scripts/components/childspawner.lua#L570-L572) |
| 组件：lootdropper | 只在校验失败的 `dead` 分支临时挂载，掉 `cookedsmallmeat` 或 `wetgoop` | 主模拟；正常孵化不掉落 | [[reference/components/lootdropper]]；[掉落](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L88-L98) |
| 标签：`tallbirdegg`／`cattoy` | 三种蛋都带 `tallbirdegg`（Walter 吃蛋反应按此标签判定）；`cattoy` 让浣猫当玩具 | 两端都在 pristine 前加 | [标签](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L169-L175)、[Walter](../../../raw/current/scripts/prefabs/walter.lua#L557-L561) |
| 关联 Prefab：`smallbird`／`tallbird` | 出雏产物／巢的占用者 | 由宿主生成，不是本页注册 | [出雏](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L27-L33)、[生成器](../../../raw/current/scripts/prefabs/tallbirdnest.lua#L175-L183) |

### 烤制与燃烧掉落（共享规则 S1／S2 的接入）

- **S1 烤制端**：COOK 门禁要求目标带 `cooker` 标签且未 `fueldepleted`（危险炉具另需专家厨师、骑乘时要求目标归自己所有）[门禁](../../../raw/current/scripts/componentactions.lua#L1096-L1105)；烹饪锅只有 `stewer` 标签与组件，不提供烤制[烹饪锅](../../../raw/current/scripts/prefabs/cookpot.lua#L288-L309)。**在带 `cooker` 的实体（营火、火坑、龙鳞火炉等）上烤高脚鸟蛋**得到 `tallbirdegg_cooked`。
- **S2 燃烧掉落**：改写只发生在掉落者已 `burnt`，或正在燃烧且（无 `fueled` 或 `burnable.ignorefuel`）时的 `DropLoot`[门禁与改写](../../../raw/current/scripts/components/lootdropper.lua#L415-L442)。**本页例外**：完整蛋不挂 `burnable`，也没有常规 `lootdropper`（只有死亡分支临时挂），原版没有把它当燃烧掉落项的表；克劳斯袋走包裹投放，不经 `DropLoot`。

## 典型接入

### 缩短高脚鸟蛋的孵化时间

- 前提与选择：服务端 `modmain.lua` postinit。**两段计时分别挂在两个注册上**：破壳阈值 `cracktime` 只在完整蛋 `tallbirdegg` 的 `unhatched` 阶段使用，孵化阈值 `hatchtime` 只在裂蛋 `tallbirdegg_cracked` 的 `comfy` 阶段使用——完整蛋从不进入 `comfy`，所以对它调 `SetHatchTime` 永远不会生效。
- 生效与撤销：`SetCrackTime` 只影响完整蛋的破壳累计；`SetHatchTime` 只影响裂蛋，完整蛋仍按 10 秒破壳。卸模组后新建实例恢复原值，已存进度保留到下次判定。
- 原版依据：`defaultfn` 先配三个阈值（`crack` 10 秒、`hatch` 3 天、`fail` 夜间一半）再 `StartUpdating`；`crack` 回调生成裂蛋并移除完整蛋，裂蛋 `crackedfn` 直接把状态设为 `comfy`。[配置](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L210-L216)、[换蛋](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L104-L115)、[comfy 才用 hatchtime](../../../raw/current/scripts/components/hatchable.lua#L173-L182)

```lua
-- modmain.lua：完整蛋 1 秒破壳；裂蛋 1 天孵化（两个注册各管一段）
AddPrefabPostInit("tallbirdegg", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local hatchable = inst.components.hatchable
    if hatchable ~= nil then
        hatchable:SetCrackTime(1) -- 只影响 unhatched → crack
    end
end)

AddPrefabPostInit("tallbirdegg_cracked", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local hatchable = inst.components.hatchable
    if hatchable ~= nil then
        hatchable:SetHatchTime(GLOBAL.TUNING.TOTAL_DAY_TIME) -- 只影响 comfy → hatch
    end
end)
```

- 调整与验证：`TOTAL_DAY_TIME`＝480 秒[常量](../../../raw/current/scripts/tuning.lua#L63-L63)；实测看裂蛋的 `progress` 到 480 秒进入 `hatch`，而且仍要玩家靠近才出雏。

## 常见陷阱与限制

- **误区：计时走完就出小鸟。** `hatch` 只是组件状态；到点时组件会立刻调 `CheckHatch`，但还要玩家在附近、蛋未被持有、位于可视地面。玩家已在旁边就当场出雏，不在旁边才等靠近或放下蛋重试。[检查](../../../raw/current/scripts/prefabs/tallbirdegg.lua#L36-L47)
- **误区：高脚鸟蛋会腐坏、能叠放。** 完整蛋没有 `perishable` 也没有 `stackable`（一格一件、永不腐烂）；只有 `tallbirdegg_cooked` 才 6 天腐坏并默认叠 20。（S5/S6）
- **误区：Walter 拒收高脚鸟蛋。** 原版是 `eater:SetOnEatFn(oneat)`：吃掉带 `tallbirdegg` 标签的食物（或 `glommerfuel`）后播一遍 `WALTER_GLOMMER_GUTS` 的歌，属性照常结算。[接线](../../../raw/current/scripts/prefabs/walter.lua#L1188-L1188)、[回调](../../../raw/current/scripts/prefabs/walter.lua#L550-L561)
- **边界：巢不会自己按时间补蛋。** `SetUp(..., nil)` 让 `MakeEmpty` 不排计时器，必须靠巢里的高脚鸟 `ForceLay`／`Regen`；把巢搬走或鸟不在附近，蛋就一直不回来。[计时条件](../../../raw/current/scripts/components/pickable.lua#L426-L448)
- **未核验**：本页未运行游戏；温度偏好下破壳／失败的实际用时、春季小鸟的延迟分布、以及熟蛋动画在客户端的显示都只按源码推导。

## 验证记录

- 源码审阅：2026-11-09，Steam buildid 26474873。核对 `prefabs/tallbirdegg.lua` 的 `commonfn`／`defaultfn`／`crackedfn`／`cookedfn` 与三段蛋的标签、组件与回调（含 `hatch` 到点即 `CheckHatch`）；`components/hatchable` 的状态推进与存读、`pickable` 的再生条件、`childspawner` 配置；`prefabs/tallbirdnest.lua` 的摘取／补蛋（巢入睡＋16 单位内有鸟）／春季小鸟／存档；`prefabs/tallbird.lua` 的建巢与护蛋目标；`walter.lua` 的 `oneat`；`klaussackloot`／`klaus_sack` 的战利品与开启路径；`cooking`／`preparedfoods` 的食材与专属料理；两个布景箱；`tuning` 与 `scrapbookdata` 交叉核对。
- 名称核验：`languages/chinese_s.po` 的 `NAMES.TALLBIRDEGG`＝高脚鸟蛋、`NAMES.TALLBIRDEGG_CRACKED`＝孵化中的高脚鸟蛋、`NAMES.TALLBIRDEGG_COOKED`＝煎高脚鸟蛋、`NAMES.TALLBIRDNEST`＝高脚鸟巢。
- 未覆盖：`smallbird` 的成长与跟随、高脚鸟的战斗数值与 Brain、巢被烧毁后的表现、`tallbirdegg_cracked` 的台词文本、以及克劳斯战利品在冬季盛宴下的额外包。
- 游戏运行：未运行。
- 结构检查：`validate_page.py` 通过（standard 档内）；`validate_source_links.py --strict` 0 错 0 警。

## 相关链接

- 孵化组件：[[reference/components/hatchable]]、[[reference/components/playerprox]]
- 采摘与生成：[[reference/components/pickable]]、[[reference/components/childspawner]]
- 吃与烤：[[reference/components/edible]]、[[reference/components/cookable]]、[[reference/components/stackable]]
- 同批：[[reference/prefabs/bird-egg]]、[[reference/prefabs/lightninggoat]]

返回 [[reference/prefabs/index|Prefab 索引]]。
