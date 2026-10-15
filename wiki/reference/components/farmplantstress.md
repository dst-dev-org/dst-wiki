---
title: "farmplantstress"
created: "2026-10-18"
updated: "2026-10-28"
type: component
summary: "管理作物阶段压力、累计结算与最终等级，区分查看检测、成长结算和实际巨型／产出消费者。"
aliases: ["作物压力", "免季节压力", "巨型作物条件", "farmplantstress"]
tags: [component, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/farmplantstress.lua", "prefabs/farm_plants.lua", "prefabs/farm_plant_defs.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# farmplantstress

## 快速判断

- 职责与适用：植物侧记录各压力项、阶段累计点数和最终等级；检测／成长触发及掉落由植物宿主安排。[组件](../../../raw/current/scripts/components/farmplantstress.lua#L3-L84) [阶段结算](../../../raw/current/scripts/prefabs/farm_plants.lua#L223-L258)
- 优先使用：已有项通过 `SetStressed(name,boolean,doer)` 改状态，阶段结算用 `MakeCheckpoint()`，最终评级用 `CalcFinalStressState()`；不要把查看描述当结算。[接口](../../../raw/current/scripts/components/farmplantstress.lua#L37-L84)
- 重要副作用：每次 checkpoint 累加当前真值项并把假项重新设真；默认新项也为真。原作物阶段会重新检测多数项，提前清一次未必保留到结算。[累计](../../../raw/current/scripts/components/farmplantstress.lua#L17-L22) [checkpoint](../../../raw/current/scripts/components/farmplantstress.lua#L55-L69) [宿主](../../../raw/current/scripts/prefabs/farm_plants.lua#L223-L253)
- 修改边界：本页服务端；最终 NONE 仍需结合巨型门禁，原宿主还查 force_oversized／no_oversized。普通种子替换与魔法成长会设置 no_oversized。[成熟](../../../raw/current/scripts/prefabs/farm_plants.lua#L514-L519) [替换](../../../raw/current/scripts/prefabs/farm_plants.lua#L151-L161) [魔法](../../../raw/current/scripts/prefabs/farm_plants.lua#L595-L604)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `AddStressCategory(name,testfn,onchangefn?)` | 新项默认为 true，num_stressors 加 1 | 不去重：重复同名会覆盖函数／状态却继续加计数；原工厂注册 nutrients／moisture／killjoys／family／overcrowding／season／happiness 七项。[接口](../../../raw/current/scripts/components/farmplantstress.lua#L17-L22) [注册](../../../raw/current/scripts/prefabs/farm_plants.lua#L854-L861) |
| `SetStressed(name,stressed,doer?)` | 仅已注册项，无显式返回 | 存储为 stressed==true，变化时调 onchangefn(plant,stressed,doer)；比较／回调使用原参数，传严格 boolean，未知项不自动创建。[实现](../../../raw/current/scripts/components/farmplantstress.lua#L37-L45) |
| `MakeCheckpoint()` | 累加 stress_points，写 checkpoint_stress_points | 每个真值项记 1；假项重设 true，不走 SetStressed 的变化回调；max_stress_points 是调试累计，不是最终评级分母。[实现](../../../raw/current/scripts/components/farmplantstress.lua#L47-L70) |
| `CalcFinalStressState()`／`GetFinalStressState()` | 前者计算并返回枚举，后者只读缓存 | 按累计点数 ≤1 NONE、≤6 LOW、≤11 MODERATE、其余 HIGH；不是按 num_stressors 自动缩放。未算前缓存 nil。[实现](../../../raw/current/scripts/components/farmplantstress.lua#L72-L84) |
| `GetStressDescription(viewer)` | 当前查看描述 | 调各 testfn(plant,currentstress,false)，结合可见／冒烟／角色与装备条件选描述；不调用 MakeCheckpoint。[实现](../../../raw/current/scripts/components/farmplantstress.lua#L90-L118) |
| `Reset()`／`CopyFrom(rhs)`／存读 | 重置或迁移状态 | Reset 将现有项设真、清累计与最终等级，不清调试字段；存读保留最终等级／点数／已注册项状态，不保存注册函数，Load 要求匹配 data.stressors。[重置](../../../raw/current/scripts/components/farmplantstress.lua#L24-L35) [存读](../../../raw/current/scripts/components/farmplantstress.lua#L120-L136) |

原植物按阶段直接调用本地检测函数写入六项；happiness 由照料／点燃等回调更新。随后取饮水比例并重置计时、结算养分、MakeCheckpoint，成熟前再算最终等级。只替换 stressors_testfns 不会替换这些本地成长检测函数，尤其养分检测 apply=true 会实际操作养分。[阶段流程](../../../raw/current/scripts/prefabs/farm_plants.lua#L223-L258) [养分测试](../../../raw/current/scripts/prefabs/farm_plants.lua#L211-L217) [快乐变化](../../../raw/current/scripts/prefabs/farm_plants.lua#L93-L119)

## 典型接入

### 种植胡萝卜结算时豁免季节压力，保留其他压力

放目标模组 `modmain.lua`，无需新资源。包裹该实例的 MakeCheckpoint，在原宿主完成各项检测之后、实际计数之前清 season，避免被本地 SeasonStressTest 覆盖。名称与原植物注册已核验。[季节检测](../../../raw/current/scripts/prefabs/farm_plants.lua#L199-L201) [结算顺序](../../../raw/current/scripts/prefabs/farm_plants.lua#L235-L253) [命名](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L254-L258) [注册](../../../raw/current/scripts/prefabs/farm_plants.lua#L926-L937) [PostInit](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("farm_plant_carrot", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local stress = inst.components.farmplantstress
    if stress == nil then return end
    local original = stress.MakeCheckpoint
    stress.MakeCheckpoint = function(self, ...)
        self:SetStressed("season", false)
        return original(self, ...)
    end
end)
```

此例只豁免后续每次 checkpoint 的 season 点数，不回减已累计点数，不修改 plant_def.good_seasons、季节生长速度或检查描述；查看时仍可能提示季节压力，因为描述会重新运行原检测。停用并重新初始化后恢复原方法，保存的累计／最终等级不回滚。[生长时间](../../../raw/current/scripts/prefabs/farm_plants.lua#L364-L383) [查看测试](../../../raw/current/scripts/components/farmplantstress.lua#L99-L110)

最终等级参与普通作物产物／种子数量，巨型则由额外标志及成熟逻辑判定；不把豁免一个压力项称为保巨型。[掉落消费者](../../../raw/current/scripts/prefabs/farm_plants.lua#L344-L360) [巨型条件](../../../raw/current/scripts/prefabs/farm_plants.lua#L514-L519) 水分统计见 [[reference/components/farmsoildrinker]]，照料入口见 [[reference/components/farmplanttendable]]，阶段触发见 [[reference/components/growable]]。

## 常见陷阱与限制

- **注册检测函数就会自动定时检测**：组件没有内建周期检测；原宿主直接调用本地函数结算，testfn 列表主要供描述／调试。新增类别需明确在哪个阶段写状态，否则默认为压力。[注册](../../../raw/current/scripts/components/farmplantstress.lua#L17-L22) [宿主结算](../../../raw/current/scripts/prefabs/farm_plants.lua#L223-L253)
- **提前 SetStressed(false) 永久免压**：宿主可能覆盖且 checkpoint 后会重设真；结合真实阶段顺序修改，不每帧反复清零累计。[checkpoint](../../../raw/current/scripts/components/farmplantstress.lua#L55-L69)
- **反复 CalcFinalStressState 期待重新调查农田**：它只按已累计点数算缓存，不检测地块或重跑成熟阶段；外观与后续掉落如何使用缓存须分别核验，不能只改枚举便声称整株重算。[评级](../../../raw/current/scripts/components/farmplantstress.lua#L72-L84) [成熟消费](../../../raw/current/scripts/prefabs/farm_plants.lua#L514-L529) [掉落消费](../../../raw/current/scripts/prefabs/farm_plants.lua#L344-L360)

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 源码审阅：2026-10-18，Steam buildid 25643519；覆盖注册／状态回调、checkpoint 累加与重置、固定阈值、查看与成长检测分工、饮水／养分协作、存读／替换、季节豁免及产出／巨型消费者。未审校全部阶段、植物参数和研究／交互系统。
- 结构检查：格式、来源、链接／行号、大小与登记通过；完整 modmain 示例通过已有 Full Moon Lua 5.1 syntax-only，不证明实际阶段和包装顺序。
- 游戏运行：未运行；跨季节、魔法成长、普通种子替换、存读／腐烂重置、掉落与巨型、其他模组方法包装兼容待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。