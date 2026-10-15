---
title: "saltlicker"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "管理生物寻找盐砖、带盐状态与延后耗用；暂停停止盐计时，驯化收益由宿主消费。"
aliases: ["牛舔盐消耗减半", "盐砖防驯化衰减", "saltlicker"]
tags: [component, creature, event, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/saltlicker.lua", "prefabs/saltlick.lua", "prefabs/beefalo.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# saltlicker

## 快速判断

- 职责与适用：装在舔盐生物，依赖预先存在的 timer；寻找 saltlick 实体后保存引用、启动 salt 计时并设置 salted。盐砖本体的容量由 finiteuses 管。[构造](../../../raw/current/scripts/components/saltlicker.lua#L71-L87) [寻找](../../../raw/current/scripts/components/saltlicker.lua#L7-L18) [盐砖](../../../raw/current/scripts/prefabs/saltlick.lua#L168-L171)
- 优先使用：SetUp(uses_per_lick) 启动完整监听，Stop 停止；SetSalted 只改状态和发事件，不负责搜索／计时。[启动](../../../raw/current/scripts/components/saltlicker.lua#L110-L124) [状态](../../../raw/current/scripts/components/saltlicker.lua#L149-L153)
- 重要副作用：计时结束才向仍有效的原盐砖 Use(uses_per_lick)，然后重新寻找；saltchange 发给生物自身，牛宿主用它暂停驯化衰减。[结算](../../../raw/current/scripts/components/saltlicker.lua#L50-L68) [牛消费者](../../../raw/current/scripts/prefabs/beefalo.lua#L922-L924) [监听](../../../raw/current/scripts/prefabs/beefalo.lua#L1230-L1233)
- 修改边界：不自动增加驯化度、恢复饥饿或为全部生物定义盐收益。休眠／入 limbo／冻结时会停止计时和搜索，不保留该轮剩余时间；暂停本身不立即清 salted。[暂停](../../../raw/current/scripts/components/saltlicker.lua#L89-L104)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `SetUp(uses_per_lick)` | 拼写为 SetUp；先 Stop 再设置消耗，非 nil 才注册 timerdone、睡醒／冻结／limbo／死亡事件并尝试启动。不把它当无副作用 setter。[实现](../../../raw/current/scripts/components/saltlicker.lua#L110-L124) |
| `saltedduration` | 默认 TUNING.SALTLICK_DURATION；用于新 salt 计时，搜索任务周期为其 1/8。修改字段不重设已创建 timer／周期任务。[初值](../../../raw/current/scripts/components/saltlicker.lua#L79-L82) [周期](../../../raw/current/scripts/components/saltlicker.lua#L40-L47) |
| `uses_per_lick` | 已启用组件的每轮结算耗用量；在 timerdone 时读取。不要赋 nil 作为暂停方式，否则 Stop 的保护分支可能不再执行清理。[结算](../../../raw/current/scripts/components/saltlicker.lua#L50-L58) [停止条件](../../../raw/current/scripts/components/saltlicker.lua#L126-L140) |
| `salted`／`SetSalted(bool)` | 属性联动 saltlicker_salted 标签；值改变才发 saltchange {salted=bool}。不能仅据字段推断当前仍有盐砖或有效计时。[标签](../../../raw/current/scripts/components/saltlicker.lua#L1-L3) [通知](../../../raw/current/scripts/components/saltlicker.lua#L149-L153) |
| `Stop()`／移除 | 撤销完整监听、salt 计时、搜索并取消 salted；移除再去标签。死亡也调用 Stop。[停止](../../../raw/current/scripts/components/saltlicker.lua#L126-L146) [死亡](../../../raw/current/scripts/components/saltlicker.lua#L106-L108) |
| 存读 | OnSave 只在 salted 时返回数据以触发 LoadPostPass；剩余时间由 timer 保存，LoadPostPass 根据 salt 计时重设状态，未保存原盐砖引用。[恢复](../../../raw/current/scripts/components/saltlicker.lua#L156-L166) |

盐砖建造时向附近每个 saltlicker 生物发 saltlick_placed {inst=盐砖}，不是世界广播；组件仅搜索时监听。耗尽会移除盐砖 saltlick 标签；保存在本组件的旧引用仍需等结算时检查有效性。[发送](../../../raw/current/scripts/prefabs/saltlick.lua#L21-L29) [建造](../../../raw/current/scripts/prefabs/saltlick.lua#L54-L60) [耗尽](../../../raw/current/scripts/prefabs/saltlick.lua#L63-L68)

## 典型接入

### 原牛每轮舔盐的盐砖消耗减半

目标模组 `modmain.lua`，无新资源。保留原牛 SetUp 的监听／计时；只把已有非零正耗用量减半，不用再次 SetUp 打断当前轮次。数量允许小数的消费边界见 [[reference/components/finiteuses]]。

```lua
AddPrefabPostInit("beefalo", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local saltlicker = inst.components.saltlicker
    if saltlicker ~= nil
        and GLOBAL.type(saltlicker.uses_per_lick) == "number"
        and saltlicker.uses_per_lick > 0 then
        saltlicker.uses_per_lick = saltlicker.uses_per_lick * 0.5
    end
end)
```

下一次结算读取新值，原 saltedduration、搜索距离、带盐状态与驯化暂停协议保持宿主逻辑。字段不由本组件保存，每次构造应用一次；停用重载恢复默认。原牛后续另一处 SetUp 会重设耗用量，其他模组重设也可能覆盖本例，因此不承诺所有生涯状态始终半耗。[原初始化](../../../raw/current/scripts/prefabs/beefalo.lua#L1230-L1233) [另一设置点](../../../raw/current/scripts/prefabs/beefalo.lua#L712) [结算读值](../../../raw/current/scripts/components/saltlicker.lua#L50-L55)

## 常见陷阱与限制

- **暂停会冻结计时，醒来继续原轮**：OnPause 调 StopTimer，TryUnpause 重新检查／开轮；它也未立即发 salted=false。根据宿主事件处理实际收益。[暂停与恢复](../../../raw/current/scripts/components/saltlicker.lua#L89-L104)
- **SetSalted(true) 就完成舔盐**：没有盐砖引用／计时与结算，只设置标签及事件；正常需求使用完整 SetUp 与宿主消费者。[状态方法](../../../raw/current/scripts/components/saltlicker.lua#L149-L153)
- **读档继续向原盐砖扣费**：组件不保存该实体引用；仅恢复 timer 对应状态，不能据此设计可靠的跨存档欠费／归属系统。[保存](../../../raw/current/scripts/components/saltlicker.lua#L156-L166)

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖完整组件、盐砖通知／容量／耗尽、牛的注册和驯化衰减消费者。有限次数消费复用既有 finiteuses 页；未全面审校所有舔盐生物、牛另一 SetUp 所属状态转换和 timer 内部序列化。
- 结构检查：本批格式、来源、链接／行号、大小与登记通过；完整半耗例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；结算／盐砖耗尽、休眠／冻结／limbo、存读引用缺失及其他模组重设待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。