---
title: "compostingbin"
created: "2026-10-22"
updated: "2026-10-28"
type: component
summary: "配置绿料与棕料堆肥周期，连接投料判定、计时和宿主产物回调。"
aliases: ["堆肥桶", "堆肥速度", "绿料棕料", "compostingbin"]
tags: [component, building, food, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/compostingbin.lua", "prefabs/compostingbin.lua", "actions.lua", "componentactions.lua", "tuning.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# compostingbin

## 快速判断

- 职责与适用：宿主上累计 greens／browns，材料至少 2 时用名为 `composting` 的 timer 周期处理；组件本身不生成 compost。[刷新与计时](../../../raw/current/scripts/components/compostingbin.lua#L99-L130)、[宿主产物回调](../../../raw/current/scripts/prefabs/compostingbin.lua#L264-L272)
- 优先使用：改原版 `compostingbin` 的时间字段，保留材料判定、计时、动画和产物逻辑；原厂服务端先添加 timer，再添加本组件并注入回调。[宿主组装](../../../raw/current/scripts/prefabs/compostingbin.lua#L376-L399)
- 重要副作用：成功投料先回调、消耗一件物品，再加材料并刷新；周期结束固定扣两份材料，刷新后才调用 finishcyclefn。[投料](../../../raw/current/scripts/components/compostingbin.lua#L132-L164)、[周期结束](../../../raw/current/scripts/components/compostingbin.lua#L13-L37)
- 修改边界：材料容量、食物分类与产物由宿主协作；`accepts_items` 只影响动作标签，不是 AddMaterials／AddCompostable 的统一拒绝开关。[标签](../../../raw/current/scripts/components/compostingbin.lua#L1-L11)、[原厂判料](../../../raw/current/scripts/prefabs/compostingbin.lua#L191-L213)

## 关键接口

| 接口／字段 | 约定 | 条件、作用与证据 |
|---|---|---|
| `AddCompostable(item)` | boolean | 缺 calcmaterialvalue 或其返回假值则 false；成功需返回 `{greens=数值, browns=数值}`，随后消耗一件。拆取规则见 [[reference/components/stackable#关键接口]]。[实现](../../../raw/current/scripts/components/compostingbin.lua#L132-L154) |
| `AddMaterials(greens?, browns?)` | 默认分别 0 | 直接加计数并 Refresh；不检查容量、accepts_items、负数或整数。有效域由调用方保证。[实现](../../../raw/current/scripts/components/compostingbin.lua#L156-L164) |
| `GetMaterialTotal()`／`IsFull()` | 总数／boolean | max_materials 默认 6；IsFull 只比较总量，不阻止直接添加；原版判料函数自行拒绝满桶。[接口](../../../raw/current/scripts/components/compostingbin.lua#L91-L97)、[容量判定](../../../raw/current/scripts/prefabs/compostingbin.lua#L191-L194) |
| `composting_time_min/max` | 秒数，组件默认 6／20 | 等比例材料取 min，纯单色取 max，中间线性插值再乘 calcdurationmultfn；原版用 tuning 覆盖，且按总量 6／4／更少分别乘 0.7／0.85／1。[刷新公式](../../../raw/current/scripts/components/compostingbin.lua#L99-L119)、[宿主倍率](../../../raw/current/scripts/prefabs/compostingbin.lua#L274-L279)、[原厂时间](../../../raw/current/scripts/tuning.lua#L5846-L5847) |
| `Refresh(cycle_completed?)` | 重算比例与计时 | 少于 2 停止；已有计时按“旧剩余／旧总时长”的**平方**乘新总时长更新，不能描述为保持一次进度比例。[实现](../../../raw/current/scripts/components/compostingbin.lua#L99-L130) |
| 回调字段 | `(inst)`；onrefreshfn 为 `(inst, cycle_completed)`；onaddcompostable 为 `(inst, item)` | calcdurationmultfn 返回倍率；低于 2 时直接调用 onrefreshfn，独立宿主必须提供它及 timer。原版还配置产物、启停和投料回调。[回调调用](../../../raw/current/scripts/components/compostingbin.lua#L99-L152)、[组装](../../../raw/current/scripts/prefabs/compostingbin.lua#L383-L393) |

原版食物投料排除 ELEMENTAL／GEARS／INSECT／BURNT；forcecompostable 优先决定绿／棕，否则 ROUGHAGE／WOOD 为棕，其余允许食物为绿。动作候选来自 edible／forcecompostable，服务端 `ADDCOMPOSTABLE` 转交 AddCompostable。[分类](../../../raw/current/scripts/prefabs/compostingbin.lua#L16-L22)、[判料](../../../raw/current/scripts/prefabs/compostingbin.lua#L191-L213)、[候选](../../../raw/current/scripts/componentactions.lua#L1210-L1217)、[强制材料候选](../../../raw/current/scripts/componentactions.lua#L1273-L1277)、[动作](../../../raw/current/scripts/actions.lua#L5540-L5544)

保存仅记录 greens、browns、current_composting_time；OnLoad 恢复计数／比例和总时长，不自行启动计时。剩余时长和取消任务复用 [[reference/components/timer#生命周期]]；原版宿主另保存可收获数量并恢复显示／声音。移除本组件只注销 timerdone，不停止仍在 timer 上的 `composting`，主动拆组件需先 StopTimer。[组件保存与清理](../../../raw/current/scripts/components/compostingbin.lua#L170-L185)、[清理监听](../../../raw/current/scripts/components/compostingbin.lua#L87-L89)、[宿主存取](../../../raw/current/scripts/prefabs/compostingbin.lua#L313-L348)

## 典型接入

### 将原版堆肥桶的新周期基础时间减半

目标模组 `modmain.lua`；复用原有资源及 `compostingbin` 注册名，保留原材料、产物、动画、负载倍率和烧毁逻辑。[注册](../../../raw/current/scripts/prefabs/compostingbin.lua#L430-L431)、[原配置](../../../raw/current/scripts/prefabs/compostingbin.lua#L383-L399)、[postinit 入口](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("compostingbin", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local bin = inst.components.compostingbin
    if bin ~= nil then
        bin.composting_time_min = bin.composting_time_min * 0.5
        bin.composting_time_max = bin.composting_time_max * 0.5
    end
end)
```

0.5 为无单位倍率，时间字段单位为游戏秒；下次启动／Refresh 用新配置，不在 postinit 主动刷新。已有存档的剩余 timer 与 current_composting_time 仍按保存值恢复；运行中投料触发 Refresh 时适用上表的平方公式。停用后工厂恢复原时间，已消耗材料、已产出堆肥和已保存的当前周期不自动回滚。

## 常见陷阱与限制

- **改 materials_per_fertilizer 就改变每次耗料**：虽默认赋值 2，但周期处理直接减绿棕各 1，或单色减 2；此字段没有参与该计算。[字段](../../../raw/current/scripts/components/compostingbin.lua#L54-L64)、[实际扣料](../../../raw/current/scripts/components/compostingbin.lua#L19-L33)
- **裸加组件就能堆肥／产物**：缺 timer 会访问失败，材料少于 2 时缺 onrefreshfn 会直接调用 nil；finishcyclefn 没提供则不产物。优先扩展完整原版宿主。[刷新依赖](../../../raw/current/scripts/components/compostingbin.lua#L99-L130)、[产物协作](../../../raw/current/scripts/prefabs/compostingbin.lua#L264-L272)
- **关闭 accepts_items 就停止运行且禁止所有调用**：只撤动作标签；原版烧毁还显式停止 timer、清零材料和关闭采收。自定义禁用同样定义计时／材料／产物边界。[标签开关](../../../raw/current/scripts/components/compostingbin.lua#L1-L11)、[烧毁处理](../../../raw/current/scripts/prefabs/compostingbin.lua#L113-L129)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页漂移文件 actions.lua（R-01）、componentactions.lua（R-04）、tuning.lua 对照 build-26148036 更新报告与 actions／componentactions hunk 地图逐条核验；ADDCOMPOSTABLE 动作、edible／forcecompostable 两处动作候选与 COMPOSTING 调参均未在变更 hunk 内，结论不变，锚点重定位 2 处、结论修订 0 处；未覆盖：运行验证。

- 源码审阅：2026-10-22，Steam buildid 25643519；覆盖材料消费／容量归属、周期耗料、刷新公式、回调、标签、保存与移组件、原版注册／产物／烧毁和减半示例；timer 与 stackable 通用行为复用同版本页面。未审校所有投料物或离线追赶的产出次数。
- 结构检查：定向页面校验通过；源码锚点 strict 校验无错误／警告；完整 modmain 示例通过 Full Moon Lua 5.1 语法检查，未执行游戏逻辑。
- 游戏运行：未运行；追加投料、读档剩余时间、烧毁及多人采收待实测。
