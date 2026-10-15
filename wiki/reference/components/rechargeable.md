---
title: "rechargeable"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "实现物品技能使用后的自动恢复，区分充能容量、时间和实际技能门禁。"
aliases: ["技能冷却", "自动充能", "充能时间", "物品CD", "rechargeable"]
tags: ["component", "item", "energy", "callback", "lifecycle"]
dst_build: "steam-buildid:26230496"
sources: ["components/rechargeable.lua", "prefabs/pocketwatch.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# rechargeable

## 快速判断

- 职责与适用：物品从未满充能逐步恢复至满，不处理技能本身。用后调用 Discharge，执行入口另行检查 IsCharged。[components/rechargeable.lua:54](../../../raw/current/scripts/components/rechargeable.lua#L54 [components/rechargeable.lua:129](../../../raw/current/scripts/components/rechargeable.lua#L129
- 优先使用：Discharge(seconds) 明确一次恢复时间；SetChargeTime 设基础时长，SetMaxCharge 设容量。[components/rechargeable.lua:60](../../../raw/current/scripts/components/rechargeable.lua#L60
- 重要副作用：rechargechange 每次量变化发送；charged／discharged 回调只在跨满充状态时触发。[components/rechargeable.lua:101](../../../raw/current/scripts/components/rechargeable.lua#L101
- 修改边界：服务端管理；SetMaxCharge(10) 不是 10 秒冷却。恢复速度受可加修饰影响。[components/rechargeable.lua:54](../../../raw/current/scripts/components/rechargeable.lua#L54

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `SetMaxCharge(val)` | 正数容量推荐域；默认 180 | 保留百分比／满充状态；不设恢复时间。[components/rechargeable.lua:60](../../../raw/current/scripts/components/rechargeable.lua#L60 |
| `SetChargeTime(seconds)`／`Discharge(seconds)` | 秒；默认基础 30 | Discharge 同设时间并把 charge 归零；必须明确传时间，非无参重置。[components/rechargeable.lua:68](../../../raw/current/scripts/components/rechargeable.lua#L68 [components/rechargeable.lua:129](../../../raw/current/scripts/components/rechargeable.lua#L129 |
| `SetCharge(value)`／`IsCharged()` | 0～total 夹值／boolean | 开关更新，并调用跨状态回调；不会自行拦截自定义能力。[components/rechargeable.lua:97](../../../raw/current/scripts/components/rechargeable.lua#L97 |
| `SetOnChargedFn(fn)`／`SetOnDischargedFn(fn)` | `(inst)` | 单回调，重复对空充能 Discharge 不重复触发 discharged。[components/rechargeable.lua:32](../../../raw/current/scripts/components/rechargeable.lua#L32 [components/rechargeable.lua:101](../../../raw/current/scripts/components/rechargeable.lua#L101 |
| `SetChargeTimeMod(source, key, mod)`／`RemoveChargeTimeMod(source, key)` | 来源、键、加算修饰 | **参数顺序 source,key,mod**；有效时间 = 基础 × (1+修饰和)，非正时可瞬间满。[components/rechargeable.lua:54](../../../raw/current/scripts/components/rechargeable.lua#L54 [components/rechargeable.lua:75](../../../raw/current/scripts/components/rechargeable.lua#L75 |
| `GetTimeToCharge()` | 剩余秒数 | (1-percent) × 有效总时间；OnSave 保存未满时的当前量／基础时间，不是离线时钟。[components/rechargeable.lua:146](../../../raw/current/scripts/components/rechargeable.lua#L146 |

## 典型接入

### 为自定义施放函数提供一次 10 秒充能门禁

这是已加载 `modmain.lua` 中供服务端技能回调调用的完整辅助函数；item 已有 rechargeable，perform_effect 必须返回成功 boolean。不注册新动作／物品，不新增资源。调用顺序依据 [components/rechargeable.lua:97](../../../raw/current/scripts/components/rechargeable.lua#L97 和 [components/rechargeable.lua:129](../../../raw/current/scripts/components/rechargeable.lua#L129)；原版怀表在能力成功路径 Discharge，见 [prefabs/pocketwatch.lua:38](../../../raw/current/scripts/prefabs/pocketwatch.lua#L38)。

```lua
local function TryUseChargedItem(item, doer, perform_effect)
    if not GLOBAL.TheWorld.ismastersim then return false end
    local rechargeable = item.components.rechargeable
    if rechargeable == nil or not rechargeable:IsCharged() then return false end
    if perform_effect(item, doer) ~= true then return false end
    if item:IsValid() and item.components.rechargeable ~= nil then
        item.components.rechargeable:Discharge(10)
    end
    return true
end
```

10 是基础秒数，时间修饰仍适用；成功后恢复过程中必须由所有技能执行入口共同检查门禁。此函数适用同步、不会重入的效果；延迟技能须预占状态并设计失败返还，不能套用此顺序。存档加载按组件已保存的量继续更新；业务动态修饰需恢复。

## 常见陷阱与限制

- **加组件就自动禁止重复施法**：执行入口仍可绕过；在服务器最终执行处核验充能，客户端灰图标不算门禁。[components/rechargeable.lua:97](../../../raw/current/scripts/components/rechargeable.lua#L97
- **在 SetOnDischargedFn 中每次启动效果**：重复 Discharge 已空物品不再跨状态；实际每次施放逻辑放能力入口，回调仅同步充能状态。[components/rechargeable.lua:101](../../../raw/current/scripts/components/rechargeable.lua#L101
- **基础 SetChargeTime 永久覆盖技能时间**：已有技能可能每次 Discharge(原值) 重设；修改实际调用点／参数，不能只改构造时值。[components/rechargeable.lua:129](../../../raw/current/scripts/components/rechargeable.lua#L129 [prefabs/pocketwatch.lua:38](../../../raw/current/scripts/prefabs/pocketwatch.lua#L38

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 9 处（删除死代码 oncurrent/ontotal 观察器后全文件 −12 行，含显示标签同步，区间文本逐段比对一致）；结论修订：无。新增 `Rechargeable.LongUpdate = OnUpdate` 别名（新 :58）——本页无 LongUpdate 相关结论；该别名为类定义期赋值，包装实例 OnUpdate 不影响 LongUpdate 路径（与报告第九轮检视条目一致），后续涉及离线追赶的表述再另行核验。
- 依据：变更清单命中 components/rechargeable.lua＋报告第九轮检视条目（LongUpdate 别名绕过包装）。

- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：容量／时长区别、跨满充回调与时间修饰、同步非重入技能的成功后 Discharge 辅助函数，依据见上文对应锚点；未交付动作注册或延迟技能预占／失败返还工程。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；辅助函数经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查，未交付新技能工程。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/spellcaster]]；[[reference/components/spellbook]]；[[reference/components/fueled]]。
