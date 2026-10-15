---
title: "foodaffinity"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "按食物 prefab、类型或实体标签配置饥饿倍率，辨别 prefab 偏好对负生命与精神基础值的豁免。"
aliases: ["喜爱食物", "食物偏好", "最爱食物", "偏好饥饿倍率", "foodaffinity"]
tags: [component, food, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/foodaffinity.lua", "components/edible.lua", "components/eater.lua", "components/wisecracker.lua", "prefabs/player_common.lua", "prefabs/wilson.lua", "prefabs/mermking.lua", "spicedfoods.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# foodaffinity

## 快速判断

- 职责与适用：挂在**食用者**身上，按食物实体选出一个饥饿收益倍率；不改食物自身基础值。普通人物公共构造添加组件，威尔逊再登记培根煎蛋偏好。[player_common.lua:2797](../../../raw/current/scripts/prefabs/player_common.lua#L2797)、[wilson.lua:150](../../../raw/current/scripts/prefabs/wilson.lua#L150
- 优先使用：指定料理用 `AddPrefabAffinity`，主食物类型用 `AddFoodtypeAffinity`，食物实体标签用 `AddTagAffinity`；多项命中取最大数字，不累加或连乘。[foodaffinity.lua:69](../../../raw/current/scripts/components/foodaffinity.lua#L69
- 重要副作用：**prefab 偏好**还令 eater 跳过基础 `healthvalue`／`sanityvalue` 为负的计算分支；类型和标签偏好没有这项效果。倍率为 1 的 prefab 条目仍可提供该豁免。[eater.lua:169](../../../raw/current/scripts/components/eater.lua#L169)、[:212](../../../raw/current/scripts/components/eater.lua#L212)、[:234](../../../raw/current/scripts/components/eater.lua#L234)、[mermking.lua:572](../../../raw/current/scripts/prefabs/mermking.lua#L572
- 修改边界：服务端食用结算；组件不改变饮食资格、动作或客户端显示。`CanEat`／`PrefersToEat` 仍走 eater 饮食门禁，添加偏好不能让角色吃原本拒绝的食物。[eater.lua:304](../../../raw/current/scripts/components/eater.lua#L304)、[:320](../../../raw/current/scripts/components/eater.lua#L320)

## 关键接口

| 接口／字段 | 参数与返回约定 | 条件、消费者与副作用 |
|---|---|---|
| `AddPrefabAffinity(prefab, bonus)` | prefab 字符串；bonus 用数值倍率 | 覆盖该键旧值；同时参与饥饿倍率和 `HasPrefabAffinity`。`1.5` 表示乘 1.5，不是增加 1.5 点。[foodaffinity.lua:20](../../../raw/current/scripts/components/foodaffinity.lua#L20 |
| `AddFoodtypeAffinity(foodtype, bonus)` | 如 `GLOBAL.FOODTYPE.VEGGIE`；数值倍率 | 只查 `food.components.edible.foodtype`，不查 `secondaryfoodtype`，也不是烹饪配方里的食材值。[foodaffinity.lua:24](../../../raw/current/scripts/components/foodaffinity.lua#L24)、[:82](../../../raw/current/scripts/components/foodaffinity.lua#L82) |
| `AddTagAffinity(tag, bonus)` | 食物实体标签；数值倍率 | 用 `food:HasTag(tag)` 匹配，不查食用者标签。[foodaffinity.lua:16](../../../raw/current/scripts/components/foodaffinity.lua#L16)、[:86](../../../raw/current/scripts/components/foodaffinity.lua#L86) |
| `RemovePrefabAffinity`／`RemoveFoodtypeAffinity`／`RemoveTagAffinity` | 对应键 | 置 nil，撤销该键；没有来源键、叠加管理或恢复旧值。[foodaffinity.lua:28](../../../raw/current/scripts/components/foodaffinity.lua#L28 |
| `GetAffinity(food)` | 食物实体；最大匹配数值，无命中 nil | 汇集精确 prefab、调味前基础 prefab、主 foodtype 和所有标签的匹配值；消费者 `edible:GetHunger(eater)` 才应用。[foodaffinity.lua:69](../../../raw/current/scripts/components/foodaffinity.lua#L69)、[edible.lua:96](../../../raw/current/scripts/components/edible.lua#L96 |
| `HasAffinity(food)`／`HasPrefabAffinity(food)` | 前者命中 true，否则 nil；后者布尔值 | 前者判断任意一类，后者仅精确／基础 prefab；二者不能替换使用。[foodaffinity.lua:40](../../../raw/current/scripts/components/foodaffinity.lua#L40)、[:61](../../../raw/current/scripts/components/foodaffinity.lua#L61) |
| `GetFoodBasePrefab(food)` | prefab 字符串 | 只借 `spicedfoods[food.prefab].basename` 还原已登记调味食品；其他 prefab 返回自身，不按名称后缀猜测。[foodaffinity.lua:56](../../../raw/current/scripts/components/foodaffinity.lua#L56)、[spicedfoods.lua:40](../../../raw/current/scripts/spicedfoods.lua#L40 |

### 结算与生命周期

`edible:GetHunger` 先处理腐败／可处理腐败物条件，再将倍率乘上 `GetAffinity`；结果进入 eater 后还受 foodmemory、hungerabsorption、自定义三维函数、整堆倍率和 hunger 上限影响。偏好并不保证固定增加 15 点饥饿；原版威尔逊的调参值是 `1.2`。[edible.lua:96](../../../raw/current/scripts/components/edible.lua#L96)、[eater.lua:203](../../../raw/current/scripts/components/eater.lua#L203)、[tuning.lua:2072](../../../raw/current/scripts/tuning.lua#L2072

三个映射默认空表；`prefab_affinites` 是保留给模组的拼写兼容别名，与 `prefab_affinities` 初始指向同一表。优先用方法，不重新赋表使别名分离。组件全文没有更新任务、事件注册、`OnSave`／`OnLoad`／移除钩子；配置由宿主构造重新登记，不自动存档运行时改动。临时偏好要自行管理触发与撤销，覆盖已有键时先保存旧值。[foodaffinity.lua:4](../../../raw/current/scripts/components/foodaffinity.lua#L4)、[wilson.lua:150](../../../raw/current/scripts/prefabs/wilson.lua#L150

## 典型接入

### 将威尔逊吃培根煎蛋的偏好倍率改为 1.5

完整 `modmain.lua` 入口，普通生存模式；使用原版威尔逊和 `baconeggs`，无需新增资源或改料理 prefab。公共人物构造已有组件，角色的原版条目由该 postinit 覆盖。模组入口注册方式与端门禁同 [[reference/components/eater#典型接入]]。

```lua
AddPrefabPostInit("wilson", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local affinity = inst.components.foodaffinity
    if affinity ~= nil then
        affinity:AddPrefabAffinity("baconeggs", 1.5)
    end
end)
```

1.5 为无单位倍率，每次成功食用时计算；基础料理及 `spicedfoods` 已登记的对应调味料理都可命中。新鲜基础饥饿为 H 时，这一步贡献 `H × 1.5`，最终实际恢复仍受上述结算链限制。与更高倍率标签等同时命中时，可能改为使用更大的值。

停用模组并重建人物实例后恢复原版配置，已经恢复的饥饿不回滚。运行期直接 `RemovePrefabAffinity("baconeggs")` 会删除整个键，**不会恢复原版 1.2**；可撤销功能应保存并恢复旧值，且要约定其他模组写同键时的所有权。这里没有临时任务或外部监听需要清理；示例未运行游戏。

### 原版数值为 1 的偏好有什么用途

鱼人王把 VEGGIE 类型倍率设为 1，又把海带、榴莲等 prefab 倍率设为 1，并注释用于防止负收益。类型条目覆盖饥饿查询，prefab 条目另外命中 `DoFoodEffects` 豁免；这两类配置不能合并成“所有蔬菜免负效果”。[mermking.lua:572](../../../raw/current/scripts/prefabs/mermking.lua#L572

## 常见陷阱与限制

- **当成全部三维增益**：倍率仅由 `GetHunger` 消费。prefab 豁免检查的是基础 `healthvalue`／`sanityvalue` 的符号，不能保证腐败、动态 getter、自定义三维函数或食物 `OnEaten` 没有负效果。食物回调与消耗仍在之后执行，顺序见 [[reference/components/eater#进食顺序与恢复]]。
- **用类型偏好代替 prefab 偏好**：类型／标签命中不能触发 `HasPrefabAffinity`。后者还被 wisecracker 用于抑制相应痛苦台词判定；偏好可能同时影响结算与台词。[wisecracker.lua:15](../../../raw/current/scripts/components/wisecracker.lua#L15
- **用负数表达讨厌，用 0 删除**：setter 不校验或限制数字，0 在 Lua 中仍为真，仍保留偏好身份；负数可变成负饥饿倍率，多条目仍取数值最大值。撤销请用 Remove；若要改变饮食资格，使用 [[reference/components/eater#关键接口]]。
- **直接调用排序辅助方法处理倍率列表**：`SortAffinitiesByBonus` 排的是含 `hunger_bonus` 字段的表项；`GetAffinity` 自己对数字排序，不调用该方法。主查询不用构造该辅助格式。[foodaffinity.lua:12](../../../raw/current/scripts/components/foodaffinity.lua#L12)、[:95](../../../raw/current/scripts/components/foodaffinity.lua#L95)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；逐文件核验漂移依据：eater.lua（C-12：DoFoodEffects 的 HasPrefabAffinity 豁免、healthvalue/sanityvalue 分支、Eat 结算链均无语义变化，仅行号平移）、wisecracker.lua（C-44：仅 grue 台词分支与新增 shrouden/dreadstone 提示，痛苦台词判定原样移至 :15）、player_common.lua（P-83：仅删虫胃 netvar，foodaffinity 注册移至 :2795）、wilson.lua（P-137：火把投掷改标签判定，与偏好无关，培根煎蛋登记移至 :150）、tuning.lua（无食物／好感键变化，AFFINITY_15 块整体位移 2063→2072）；本页结论全部成立，重定位锚点 8 处；未覆盖：运行验证。

- 源码审阅：2026-10-21，Steam buildid 25643519；读组件全文，核对 edible 饥饿消费、eater 负生命／精神门禁及饮食门禁、wisecracker 消费、公共人物构造、威尔逊与鱼人王配置、调味映射和实际倍率。进食消耗顺序复用同版本 eater 页。
- 结构／语法：本页元数据、大小与链接检查通过，源码锚点 strict 检查无错误／警告；完整 `modmain.lua` 经已有 Full Moon 解析器的 Lua 5.1 `--syntax-only` 检查通过。
- 游戏运行：未运行；未覆盖所有角色／食品、多人显示、存读及多模组同键覆盖实测，不能把源码审阅或语法通过视为运行通过。

相关：[[reference/components/edible]]；[[reference/components/eater]]。
