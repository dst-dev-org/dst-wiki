---
title: "crop"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "管理旧式作物的连续成长进度、成熟／枯萎与收获，区分旧农场及现代 farm_plant 阶段系统。"
aliases: ["旧作物生长时间减半", "旧农场收获", "连续成长进度", "crop"]
tags: [component, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/crop.lua", "components/grower.lua", "prefabs/plant_normal.lua", "prefabs/veggies.lua", "actions.lua", "componentactions.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# crop

## 快速判断

- 职责与适用：旧 plant_normal 系列上的单株连续进度／收获组件；grower 管农场批次，crop 管每株。现代 farm_plant 的阶段、压力与土壤不是这条路径。[宿主](../../../raw/current/scripts/prefabs/plant_normal.lua#L237-L241) [旧农场启动](../../../raw/current/scripts/components/grower.lua#L128-L135) 现代成长见 [[reference/components/growable]]。
- 优先使用：完整启动用 `StartGrowing(product,grow_time,grower?,percent?)`，收获走 Harvest；别只改成熟标签或直接 Mature 就当完成成长。[启动](../../../raw/current/scripts/components/crop.lua#L191-L203) [成熟](../../../raw/current/scripts/components/crop.lua#L251-L258)
- 重要副作用：自然成长受温度、雨水／春季和夜间光照影响；黑暗过久可强制枯萎。Harvest 会发放／掉落产物并移除作物或通知旧农场清理，不是只读取产物。[成长](../../../raw/current/scripts/components/crop.lua#L114-L168) [收获](../../../raw/current/scripts/components/crop.lua#L205-L249)
- 修改边界：本页服务端旧机制；原蔬菜种子的 plant_normal_ground 部署代码已在注释块内，当前 OnDeploy 生成 farmplantable 指定的现代植物，不能把注释当现行调用。[现行与注释分支](../../../raw/current/scripts/prefabs/veggies.lua#L187-L202)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `StartGrowing(prod,grow_time,grower?,percent?)` | 产物 Prefab、正秒数、农场实体、0～1 初始比例 | rate=1/grow_time，比例夹到 0～1；设置动画／归属并重建每 2 秒任务。不自动清已有 matured／withered，不能当任意状态重置器。[实现](../../../raw/current/scripts/components/crop.lua#L191-L203) |
| `DoGrow(dt,nowither?)` | 进入成长分支通常 true；被枯萎／已满阻止则 false | nowither 只使 shouldgrow 通过夜晚光照判断，不绕过开头 withered 或温度倍率；进度增量 dt*rate*世界倍率，达到 1 才调用成熟并取消任务。[实现](../../../raw/current/scripts/components/crop.lua#L126-L168) |
| `GetWorldGrowthRateMultiplier()` | 温度过低 0，否则按雨水／春季／默认返回 | 雨水先于春季，且要求无 rainimmunity；是原 crop 的倍率，不是现代作物季节表。[实现](../../../raw/current/scripts/components/crop.lua#L114-L125) |
| `Fertilize(item,doer?)` | 非冬季低温阻止时 true，否则无显式返回 | 先 StopSmoldering；有 fertilizer 时进度增加 fertilizervalue*rate，不读取三养分，也不负责消费物品。[实现](../../../raw/current/scripts/components/crop.lua#L84-L110) |
| `Mature()`／`IsReadyForHarvest()` | 设置成熟与回调／读取 matured | Mature 要求有效产物且未成熟／未枯萎，不把进度自动设为 1；只改 matured 也不能代替完整外观回调。[实现](../../../raw/current/scripts/components/crop.lua#L251-L258) [查询](../../../raw/current/scripts/components/crop.lua#L282-L284) |
| `MakeWithered()` | 无显式返回 | 取消主任务，清成熟／进度，产物变 cutgrass、rate=0，播 picked 并处理燃烧组件／回调；本方法不自行加 withered 标签，标签通常由 witherable 管理。[实现](../../../raw/current/scripts/components/crop.lua#L260-L280) |
| `SetOnMatureFn`／`SetOnWitheredFn`／`SetOnHarvestFn` | `(crop_entity)`／`(crop_entity)`／`(crop_entity,product,harvester)` | 宿主负责外观及附加逻辑；Harvest 回调在产物发放和作物清理之前。[setter](../../../raw/current/scripts/components/crop.lua#L35-L45) [调用](../../../raw/current/scripts/components/crop.lua#L218-L244) |
| `Harvest(harvester?)` | 可收获时 true,product；否则无显式返回 | 接受 matured **或** withered 标签；燃烧路径尝试生成熟产物，没有 cookable 则 seeds_cooked。harvester 非 nil 时要求其 inventory，nil 则地面掉落。[实现](../../../raw/current/scripts/components/crop.lua#L205-L249) |

正常 HARVEST 候选允许 readyforharvest／withered 且执行者有 inventory replica，动作再调用组件；因此 IsReadyForHarvest=false 不等于枯萎作物不能清理。收获成功后重置内部产物／进度，再由旧 [[reference/components/grower]] RemoveCrop 或移除自身。[候选](../../../raw/current/scripts/componentactions.lua#L343-L347) [动作](../../../raw/current/scripts/actions.lua#L2651-L2656) [清理](../../../raw/current/scripts/components/crop.lua#L236-L247)

存读保存产物、进度、rate 和 matured，不保存 grower 引用；旧农场读档另恢复归属并 Resume，独立作物在 LoadPostPass 才自行 Resume。LongUpdate 直接 DoGrow(dt)。移除组件的钩子只清标签，没有显式取消 task；动态卸载不能据此假定成长任务已停。[存读](../../../raw/current/scripts/components/crop.lua#L47-L82) [农场恢复](../../../raw/current/scripts/components/grower.lua#L87-L108) [移除](../../../raw/current/scripts/components/crop.lua#L30-L33) [长更新](../../../raw/current/scripts/components/crop.lua#L286-L288)

## 典型接入

### 旧 plant_normal 作物以后启动时，基准生长时间减半

放目标模组 `modmain.lua`，适用于已有旧农场调用链，不新增资源／配方。只包装 plant_normal 的 StartGrowing，保留产物、grower、初始比例和原调度；不修改现代 farm_plant 或 ground 变体。[原工厂注册](../../../raw/current/scripts/prefabs/plant_normal.lua#L321-L322) [旧农场调用](../../../raw/current/scripts/components/grower.lua#L128-L135) [PostInit](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("plant_normal", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local crop = inst.components.crop
    if crop == nil then return end
    local original = crop.StartGrowing
    crop.StartGrowing = function(self, product, grow_time, grower, percent)
        return original(self, product, grow_time * 0.5, grower, percent)
    end
end)
```

只影响后续实际调用 StartGrowing 时传入的正时长；温度／夜晚等门禁仍生效，不保证现实等待固定减半。存档中的 rate 由 OnLoad 直接恢复，不自动为已种作物追溯重算；停用后重新初始化不再包裹，已存 rate 也不回滚。与 grower／plantable 的时长修改同时安装会叠乘，按需求只选一个主要入口。

## 常见陷阱与限制

- **直接 Mature 或写 matured=true 等于完整长熟**：不会统一补齐进度、动画与任务停止；正常完成走 DoGrow 的满进度分支及原宿主回调。[成熟](../../../raw/current/scripts/components/crop.lua#L251-L258) [完成分支](../../../raw/current/scripts/components/crop.lua#L153-L162)
- **MakeWithered 自己就提供 withered 状态**：它改产物／动画而不加标签；自然枯萎须复用 [[reference/components/witherable]] 的协作，不能仅靠此方法宣称状态完整。[方法](../../../raw/current/scripts/components/crop.lua#L260-L280)
- **施肥返回 true 就代表消耗且增加进度**：本方法不扣物品，特定输入可无增量仍 true；正常 FERTILIZE 的来源检查／消费见 [[reference/components/fertilizer]]，不要手动双扣。[施肥实现](../../../raw/current/scripts/components/crop.lua#L84-L110)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对本页 sources 变更文件 actions.lua、componentactions.lua（R-01/R-04）：HARVEST 候选（componentactions.lua:343 未移位）与动作未被本次 hunk 触及；新增 CORRUPTNIGHTMARE 的 nightmarecorruptable 目标面为 boss／棋子等生物 prefab，不涉及作物页，结论无修订；锚点重定位 1 处（HARVEST.fn +3 行）；未覆盖：运行验证。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖连续进度／自然门禁、施肥、成熟／枯萎、收获／回调／归属清理、存读与 plant_normal 宿主，以及新启动时长减半例。未审校旧农场恢复模组、全部可作产物、加载／存档迁移或现代农作全链。
- 结构检查：格式、来源、链接／行号、大小与登记通过；完整示例通过已有 Full Moon Lua 5.1 syntax-only，不证明游戏成长、收获或旧对象可制作。
- 游戏运行：未运行；黑暗／低温、枯萎／燃烧收获、存读恢复、旧农场取得与其他模组倍率叠加待实测。
