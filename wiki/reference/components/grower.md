---
title: "grower"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "管理旧农场的种植点、作物子实体与肥力轮次，明确直接种植的破坏性及与现代农田的区别。"
aliases: ["旧高级农场时长减半", "旧农场肥力轮次", "种植点与作物归属", "grower"]
tags: [component, building, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/grower.lua", "components/crop.lua", "prefabs/farmplot.lua", "prefablist.lua", "recipes.lua", "componentactions.lua", "actions.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# grower

## 快速判断

- 职责与适用：旧农场设施侧保存作物集合、相对种植点和肥力轮次，生成 plant_normal 子实体；不是现代 farming_manager 的地块水分／三养分。[PlantItem](../../../raw/current/scripts/components/grower.lua#L110-L147) 现代地块见 [[reference/components/farming-manager]]。
- 优先使用：正常旧农场种植走 ACTIONS.PLANT，由其从 inventory 取一颗，再 PlantItem；配置 croppoints、growrate、cycles_left 时核对旧宿主工厂。[动作](../../../raw/current/scripts/actions.lua#L2632-L2649) [工厂配置](../../../raw/current/scripts/prefabs/farmplot.lua#L327-L336)
- 重要副作用：PlantItem 仅先检查种子有 plantable，随后立即 Reset 清旧作物；不检查空位／肥力。growrate 是生长**时长乘数**，调小才快。[执行顺序](../../../raw/current/scripts/components/grower.lua#L110-L142)
- 修改边界：本页服务端旧结构。farmplot 脚本仍在加载清单且工厂返回 slow_farmplot／fast_farmplot；recipes 将其列作旧结构的拆解配方，不据此宣称当前正常制作菜单能建。[脚本清单](../../../raw/current/scripts/prefablist.lua#L417) [注册](../../../raw/current/scripts/prefabs/farmplot.lua#L375-L379) [旧拆解记录](../../../raw/current/scripts/recipes.lua#L1701-L1704)

## 关键接口

| 接口／字段 | 参数或默认 | 条件、作用与证据 |
|---|---|---|
| `croppoints`／`growrate` | 默认空数组／1 | 每个相对坐标生成一株 plant_normal，传入 grow_time=plantable.growtime*growrate；空点表仍可消费种子并设 isempty=false，单加组件不是完整农场。[初值](../../../raw/current/scripts/components/grower.lua#L16-L26) [生成](../../../raw/current/scripts/components/grower.lua#L124-L142) |
| `cycles_left`／`max_cycles_left`／`isempty` | 默认 1／6／true | 属性回调按空位与轮次维护 infertile／fertile／fullfertile；满种状态不保留这些肥力候选标签。[属性](../../../raw/current/scripts/components/grower.lua#L1-L33) |
| `IsEmpty()`／`IsFertile()`／`IsFullFertile()`／`GetFertilePercent()` | 当前状态／轮次比例 | 肥力比例不夹到 0～1，Fertilize 也没有上限裁剪；不要把它当现代土壤养分百分比。[查询](../../../raw/current/scripts/components/grower.lua#L41-L55) [施肥](../../../raw/current/scripts/components/grower.lua#L68-L85) |
| `PlantItem(item,doer?)` | 无 plantable 时 false，否则结尾 true | Reset 后用 product(item)／product／item.prefab 选**产物**，固定生成 plant_normal，设 persists=false 与 crop.grower 归属；删除整个传入 item，发世界 itemplanted。[实现](../../../raw/current/scripts/components/grower.lua#L110-L147) |
| `onplantfn`／`setfertility` | `(item)`／`(farm,percent)` | 前者在删种子前调用，只传 item；后者由施肥、移除最后作物、加载等路径显式调用，不因直接改字段就自动刷新外观。[种植](../../../raw/current/scripts/components/grower.lua#L139-L144) [施肥](../../../raw/current/scripts/components/grower.lua#L77-L84) [移除](../../../raw/current/scripts/components/grower.lua#L157-L166) |
| `Fertilize(item,doer?)` | 结尾 true | 有 fertilizer 时加 soil_cycles，不自行消费，且不检查 IsEmpty／IsFullFertile；正常动作另限制空农场。[实现](../../../raw/current/scripts/components/grower.lua#L68-L85) [动作](../../../raw/current/scripts/actions.lua#L1817-L1820) |
| `RemoveCrop(crop)` | 无显式返回 | 删除指定子实体；集合清空后才恢复空位／去 NOCLICK，并扣一轮肥力，最低 0。[实现](../../../raw/current/scripts/components/grower.lua#L149-L167) |
| `Reset(reason?)` | 无显式返回 | 删除全部作物并清集合，不按收获扣轮次；fire 分支另产熟食／灰／熟种子并监听掉落清理，不用于普通无损清空。[实现](../../../raw/current/scripts/components/grower.lua#L173-L202) |

种植候选依赖 fertile／fullfertile 标签；PlantItem 本身不重查这些条件。当前动作对失败种子尝试 GiveItem 还回，但不能替已经 Reset、生成报错或删除的对象做事务回滚。直接传整堆也不是正常动作拆一颗的等价调用。[候选](../../../raw/current/scripts/componentactions.lua#L1542-L1546) [动作](../../../raw/current/scripts/actions.lua#L2632-L2649)

存档保存每个作物的 SaveRecord 和剩余轮次，加载时 SpawnSaveRecord、恢复 crop.grower、设非独立持久化并 Resume；growrate／croppoints／max_cycles_left 由宿主重新配置。移除组件只清三类肥力标签，不清 grower 标签或调用 Reset，动态拆卸须自己安排子实体清理。[存读](../../../raw/current/scripts/components/grower.lua#L57-L65) [恢复](../../../raw/current/scripts/components/grower.lua#L87-L108) [移除钩子](../../../raw/current/scripts/components/grower.lua#L35-L39)

## 典型接入

### 已存在旧高级农场的后续播种，基准成长时长减半

放目标模组 `modmain.lua`，适用于已有或明确恢复旧农场对象的项目；无需新资源，但本例不恢复制作配方，也不替代旧对象取得方案。只改 fast_farmplot 实例工厂完成后的 growrate，不能在组件构造后马上赋值并期待不被宿主覆盖。[工厂赋值](../../../raw/current/scripts/prefabs/farmplot.lua#L327-L336) [注册名](../../../raw/current/scripts/prefabs/farmplot.lua#L376-L379) [PostInit](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("fast_farmplot", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local grower = inst.components.grower
    if grower ~= nil then
        grower.growrate = grower.growrate * 0.5
    end
end)
```

只影响后续 PlantItem 创建的 crop 时长；既有子作物的 rate 已写入自身及存档，不自动追溯更新。slow_farmplot、现代农田、肥力上限不变；天气／光照门禁仍生效。停用并重新初始化恢复原设施配置，不回滚已保存的子作物 rate。种子基准时长见 [[reference/components/plantable]]，单株执行见 [[reference/components/crop]]；多处减时长会叠乘，不必同时安装三个示例。

## 常见陷阱与限制

- **growrate 越大生长越快**：它乘 growtime，再由 crop 求倒数；应按时长倍率理解。[调用](../../../raw/current/scripts/components/grower.lua#L128-L132) [倒数](../../../raw/current/scripts/components/crop.lua#L191-L194)
- **PlantItem 会拒绝满种／无肥力**：方法先 Reset 再种，不检查这些门禁；自定义直接调用前确认资格，不用它试探能否种植。[实现](../../../raw/current/scripts/components/grower.lua#L110-L142)
- **删除 grower 自动清所有作物与标签**：该移除钩子只清肥力标签；宿主正常拆除先 Reset 再移除农场，动态卸载要保留等价清理。[钩子](../../../raw/current/scripts/components/grower.lua#L35-L39) [原拆除](../../../raw/current/scripts/prefabs/farmplot.lua#L38-L51)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对本页 sources 变更文件 actions.lua、componentactions.lua（R-01/R-04）、prefablist.lua、recipes.lua（detail-root，无独立条目 ID）：PLANT／FERTILIZE 动作未变，farmplot 仍在加载清单与旧拆解配方中（行号因新增 Rifts8 条目后移），结论无修订；锚点重定位 5 处；未覆盖：运行验证。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖种植点／时长／产物、标签与肥力、Reset／消费顺序、作物集合与最后一株扣轮次、存读、旧工厂／脚本清单与拆解记录，以及后续种植减时长例。未审校旧制作入口恢复、火烧重置全链或任意多点布局。
- 结构检查：格式、来源、链接／行号、大小与登记通过；完整示例通过已有 Full Moon Lua 5.1 syntax-only，不证明旧结构取得或游戏种植成功。
- 游戏运行：未运行；旧设施加载、空点表／多点、肥力上下限、满种直接调用、读档／拆除和其他模组倍率叠加待实测。
