---
title: "watersource"
created: "2026-10-18"
updated: "2026-10-28"
type: component
summary: "配置实体水源的可用标签、使用回调与补充份量，区分可耗材料、池塘和接收方行为。"
aliases: ["一块冰补两次水", "水源消耗", "水源可用开关", "watersource"]
tags: [component, item, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/watersource.lua", "components/fillable.lua", "prefabs/inv_rocks_ice.lua", "prefabs/wateringcan.lua", "prefabs/pond.lua", "prefabs/snowball_item.lua", "tuning.lua", "actions.lua", "componentactions.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# watersource

## 快速判断

- 职责与适用：实体水源侧的可用标签与使用委托；不内建水量、补水数值运算或自动扣一件。[完整组件](../../../raw/current/scripts/components/watersource.lua#L1-L39)
- 优先使用：切换动作资格改 `available`，消费逻辑用 onusefn，份量由接收方支持的 override_fill_uses 表达；不要手动只删 watersource 标签。[属性](../../../raw/current/scripts/components/watersource.lua#L1-L25) [原壶消费者](../../../raw/current/scripts/prefabs/wateringcan.lua#L26-L37)
- 重要副作用：Use 只调用 onusefn(source)，不检查 available、不返回回调结果；是否消费和消费多少取决于该回调。[Use](../../../raw/current/scripts/components/watersource.lua#L33-L37)
- 修改边界：本页服务端；正常实体 FILL 检查 watersource 标签，直接 Fill(source)／Use() 不含相同门禁。海水地点分支不创建 watersource 实体。[动作](../../../raw/current/scripts/actions.lua#L2150-L2164) [直接填充](../../../raw/current/scripts/components/fillable.lua#L28-L35)

## 关键接口

| 接口／字段 | 参数或默认 | 作用、条件与证据 |
|---|---|---|
| `available` | 默认 true，属性回调 | 真值加 watersource 标签，假值移除；不是内部蓄水量或 Use 的拒绝开关。[实现](../../../raw/current/scripts/components/watersource.lua#L1-L25) |
| `onusefn`／`Use()` | `(source)`；Use 无显式返回 | 无回调就不做消费；回调没有 doer／目标参数。不能靠 return false 阻止 Fill 后续执行。[实现](../../../raw/current/scripts/components/watersource.lua#L33-L37) [调用方](../../../raw/current/scripts/components/fillable.lua#L28-L35) |
| `override_fill_uses` | 构造未赋值；原壶读取 number | 原壶有此值则给 finiteuses 加该量并限制到 total，无该值才补满；不是通用水量，也不影响 filledprefab 换物分支。[组件注释](../../../raw/current/scripts/components/watersource.lua#L13-L25) [实际消费者](../../../raw/current/scripts/prefabs/wateringcan.lua#L26-L37) |
| `OnRemoveFromEntity()` | 清 watersource 标签 | 组件无自有 OnSave／OnLoad；动态 availability 的存读／重建须由宿主安排，不能假定字段自动持久化。[移除](../../../raw/current/scripts/components/watersource.lua#L27-L31) |

原可耗来源：ice 用回调从 stackable 取一件删除，份量 1；snowball_item 同样删一份，配置份量来自 TUNING，本快照为 0.3，不能假定只允许整数。池塘不配置这类物品消耗，结冰／解冻时改 available。[冰消费](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L45-L51) [冰配置](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L115-L117) [雪团消费](../../../raw/current/scripts/prefabs/snowball_item.lua#L115-L121) [雪团配置](../../../raw/current/scripts/prefabs/snowball_item.lua#L227-L229) [份量](../../../raw/current/scripts/tuning.lua#L8560) [池塘](../../../raw/current/scripts/prefabs/pond.lua#L145-L182)

## 典型接入

### 一块冰给原版洒水壶补 2 次水，仍只消耗一块

放目标模组 `modmain.lua`，无需新资源。只改 `ice` 的份量，不包裹／重复调用原 onusefn；原冰工厂注册名为 ice，服务端已有 watersource 和一份消费回调。[工厂](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L107-L117) [注册](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L131) [PostInit](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("ice", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local source = inst.components.watersource
    if source ~= nil then
        source.override_fill_uses = 2
    end
end)
```

适用于读取该字段的普通／高级原版洒水壶，最多补到总容量，剩余空间不足 2 时仍先消费一块冰；本例不提供退款／按容量折算。字段不改变冰的食用／修复效果，也不让蚊子血囊换出两个水球。停用并重新初始化恢复原份量，已补入的次数不回减。[原壶共享工厂](../../../raw/current/scripts/prefabs/wateringcan.lua#L140-L149) [两个注册](../../../raw/current/scripts/prefabs/wateringcan.lua#L174-L175) [先消费](../../../raw/current/scripts/components/fillable.lua#L28-L44)

实体作为水源与被填充物可在动作中互换 invobject／target，最终仍由接收者 Fill(source) 发起 Use。新水源若需要客户端动作候选，还要沿宿主工厂核验 pristine 标签与服务端组件，不把只有服务端字段改动当完整新水源骨架。[动作分派](../../../raw/current/scripts/actions.lua#L2126-L2148) [候选](../../../raw/current/scripts/componentactions.lua#L1257-L1261) [反向候选](../../../raw/current/scripts/componentactions.lua#L1925-L1929) [冰双端边界](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L66-L77)

## 常见陷阱与限制

- **available=false 后直接 Use 期待拒绝**：方法不检查它；走正常动作标签门禁，或在自定义调用方先验证前提。不要以 Use 的 nil 返回推断成功／失败。[Use](../../../raw/current/scripts/components/watersource.lua#L33-L37)
- **在 Fill 回调里再次扣水源**：Fill 已先调用 Use，会双扣；来源缺 onusefn 时反而不会自动消费，按物品／池塘场景分别配置。[先消费](../../../raw/current/scripts/components/fillable.lua#L28-L35) [冰回调](../../../raw/current/scripts/prefabs/inv_rocks_ice.lua#L45-L51)
- **以为 override_fill_uses 对所有容器都生效**：该字段靠接收方解释，默认 filledprefab 模式不读取它；具体优先级与失败边界见 [[reference/components/fillable]]。[原壶解释](../../../raw/current/scripts/prefabs/wateringcan.lua#L26-L37) [默认替换](../../../raw/current/scripts/components/fillable.lua#L33-L44)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的变更文件（actions.lua、componentactions.lua、tuning.lua）对照更新报告 R-01/R-04 与 hunk 地图逐条核验，FILL 分派、双向候选与 SNOWBALL_WATERSOURCE_FILL_USES 均无语义变化，锚点重定位 4 处、结论修订 0 处；未覆盖：运行验证。

- 源码审阅：2026-10-18，Steam buildid 25643519；覆盖可用标签／移除、Use 的门禁与返回、Fill 调用顺序、冰／雪团消费与份量、池塘冻结开关、原壶按容量补充及动作方向。未全面审校池塘宿主存读、所有水源或其他模组接收方。
- 结构检查：格式、来源、链接／行号、大小与登记通过；完整示例通过已有 Full Moon Lua 5.1 syntax-only，不证明实际消费或满容量行为。
- 游戏运行：未运行；单块／堆叠／最后一份、满壶／余量不足、池塘冻结、雪团小数份量、存读与其他模组消费者待实测。
