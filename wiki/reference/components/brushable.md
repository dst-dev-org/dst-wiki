---
title: "brushable"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "可刷目标按世界天数结算奖品、事件与回调；区分标签门禁、实际条件表达式及宿主驯化收益。"
aliases: ["刷毛积攒两份牛毛", "刷毛产出间隔", "brushable"]
tags: [component, creature, event, callback, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/brushable.lua", "prefabs/beefalo.lua", "actions.lua", "stategraphs/SGBeefalo.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# brushable

## 快速判断

- 职责与适用：装在可刷目标上，按世界 cycles 与 lastbrushcycle 的差计算产物数，向刷毛者库存发放，再发 brushed 和调用回调；工具标记见 [[reference/components/brush]]。[数量](../../../raw/current/scripts/components/brushable.lua#L31-L34) [结算](../../../raw/current/scripts/components/brushable.lua#L36-L56)
- 优先使用：SetBrushable(bool, reset) 管标签／可选重置起算天，SetOnBrushed(fn) 管宿主回调；产物相关字段为 prize、max、cyclesperprize。[开关](../../../raw/current/scripts/components/brushable.lua#L20-L29)
- 重要副作用：进入产物分支即更新 lastbrushcycle，即使计算出 0；brushed 和 onbrushfn 在分支外执行。原牛只在 numprizes>0 时给驯化与服从收益。[结算顺序](../../../raw/current/scripts/components/brushable.lua#L36-L56) [牛收益](../../../raw/current/scripts/prefabs/beefalo.lua#L482-L490)
- 修改边界：直接 Brush 不检查死亡／战斗，也未严格以 brushable 为总门禁；正常动作另做这些检查。剃毛、毛发生长／外观和刷毛产出属于协作层。[动作](../../../raw/current/scripts/actions.lua#L4205-L4215) [牛剃毛](../../../raw/current/scripts/prefabs/beefalo.lua#L431-L442)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `prize`／`max`／`cyclesperprize` | 默认 nil／0／0；数量为 elapsed<=0 时 0，否则 min(floor(elapsed/cyclesperprize),max)。自定义间隔须设正数，不能把默认 0 当已配置的安全周期。[初值](../../../raw/current/scripts/components/brushable.lua#L5-L13) [公式](../../../raw/current/scripts/components/brushable.lua#L31-L34) |
| `SetBrushable(bool, reset)` | 属性联动 brushable 标签；reset==true 才把 lastbrushcycle 设为当前世界天数，不自动更改奖品／间隔。[标签](../../../raw/current/scripts/components/brushable.lua#L1-L3) [方法](../../../raw/current/scripts/components/brushable.lua#L20-L25) |
| `CalculateNumPrizes()` | 只算数量，不扣产物、不检查开关，不代表现在可交互；无默认正周期保证。[计算](../../../raw/current/scripts/components/brushable.lua#L31-L34) |
| `Brush(doer, brush)` | 条件原文为 `self.brushable and self.prize ~= nil or self.max > 0`，即 `(brushable and prize~=nil) or max>0`；doer 需要有效实体。brush 参数在本实现不消费、不传给回调。[实现](../../../raw/current/scripts/components/brushable.lua#L36-L56) |
| `SetOnBrushed(fn)` | 回调 fn(target,doer,numprizes)，在 target 的 brushed {doer,numprizes} 后；包装而非无条件覆盖已有收益。[回调](../../../raw/current/scripts/components/brushable.lua#L52-L56) |
| 存读 | 只保存 lastbrushcycle、brushable；prize／max／周期／回调由宿主构造配置，不能把这些配置当作保存数据。[保存](../../../raw/current/scripts/components/brushable.lua#L59-L70) |

## 典型接入

### 原牛按一世界天一份、最多积攒两份牛毛

目标模组 `modmain.lua`，无新资源。明确配置组件实际读取的 cyclesperprize，不写 regrowthdays；保留原产物、刷毛回调、已有起算日和剃毛后的开关。

```lua
AddPrefabPostInit("beefalo", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local brushable = inst.components.brushable
    if brushable ~= nil then
        brushable.cyclesperprize = 1
        brushable.max = 2
    end
end)
```

单位为世界 cycles 差，不是秒；若已有足够 elapsed，可立即计算已有累计量，并非安装起另等一天。每次刷产物分支都会从当天重新累计，没有分数余量结转。原牛回调只按“是否有产物”触发一次驯化收益，产两份不自动给双倍驯化。配置重载后重设，停用重载恢复宿主原字段；时间／开关沿用组件存读。

原牛剃毛关闭可刷，SG 毛长出时 SetBrushable(true,true) 重置起算天；不通过本例强行提前开刷。[关闭](../../../raw/current/scripts/prefabs/beefalo.lua#L431-L434) [恢复](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L831-L835) [原配置](../../../raw/current/scripts/prefabs/beefalo.lua#L1145-L1149)

## 常见陷阱与限制

- **SetBrushable(false) 拒绝所有脚本 Brush**：max>0 时上述表达式仍进入产物分支；动作标签隐藏和直接方法门禁不同。自定义直接调用应先明确检查组件状态／宿主条件。[表达式](../../../raw/current/scripts/components/brushable.lua#L36-L40)
- **照抄原牛 regrowthdays 就配置了一天周期**：组件读取的是 cyclesperprize；原宿主确实写 regrowthdays，但本实现未读它。这里只记录已核验的字段不对应，不推断引擎如何表现除零。[原字段](../../../raw/current/scripts/prefabs/beefalo.lua#L1145-L1149) [实际读取](../../../raw/current/scripts/components/brushable.lua#L31-L34)
- **每次 Brush 成功都掉毛／加驯化**：数量可为 0；事件仍发，动作也可返回 true。调用结果、产物和宿主收益分开判断。[事件](../../../raw/current/scripts/components/brushable.lua#L52-L56) [收益](../../../raw/current/scripts/prefabs/beefalo.lua#L482-L490)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；漂移文件 actions.lua、stategraphs/SGBeefalo.lua、prefabs/beefalo.lua 对照 R-01/R-04 hunk 地图逐条核验：BRUSH 动作执行链逐行未变、SGBeefalo 仅复活音效改动（:667，不在引用段）、beefalo 内容无差异；锚点重定位 1 处、结论无修订；未覆盖：运行验证。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖完整组件、表达式优先级、原牛字段／收益／剃毛和 SG 重新可刷片段、动作拒绝。未用执行测试证明默认除零在游戏 Lua／平台上的结果，未全面审校牛 SG。
- 结构检查：本批格式、来源、链接／行号、大小及登记通过；完整周期／上限例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；同日重复刷、跨天累计、双份产物／单次收益、剃毛后恢复、存读和直接调用门禁待实测。

- 2026-10-28 build 26230496：brushable.lua 重构为 AddOrRemoveTag（无行为变化）锚点重定位；actions.lua BRUSH 接入拟态第三参随之移位。