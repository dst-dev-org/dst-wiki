---
title: "bloomness"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "绽放等级状态机（Wormwood 专用）：0..max 级随计时升落，施肥延长满级或加速绽放，等级变化回调驱动外观。"
aliases: ["绽放度", "bloomness", "沃姆伍德绽放"]
tags: [component, state-machine, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/bloomness.lua", "prefabs/wormwood.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# bloomness

## 快速判断

- 职责与适用：挂在 Wormwood（[wormwood.lua](../../../raw/current/scripts/prefabs/wormwood.lua#L839-L843)）身上管理 0..3 级绽放状态：计时器归零时按当前方向升级（绽放中）或降级（凋谢），等级变化经 `onlevelchangedfn` 驱动外观与属性。
- 优先使用：施肥联动走 fertilizable（见 [[reference/components/fertilizable]]）转调 `Fertilize`（wormwood 的施肥入口 [L503-L507](../../../raw/current/scripts/prefabs/wormwood.lua#L503-L507)）；技能树直接读改等级与计时（skilltree_wormwood.lua:229）。
- 重要副作用：等级 >0 时组件进入 `StartUpdatingComponent` 每帧扣计时；满级施肥经 `calcfullbloomdurationfn` 延长满级时长（[L67-L74](../../../raw/current/scripts/components/bloomness.lua#L67-L74)）。
- 修改边界：数值全部可由宿主回调定制（`calcratefn`/`calcfullbloomdurationfn`/`onlevelchangedfn`）；`max` 默认 3，无 setter，改上限需直接赋值字段。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetLevel(level)` | `number -> void` | 超出 max 夹取；0 级复位并停更新，否则按方向记 `is_blooming`、累加计时（未满 `stage_duration`、满级 `full_bloom_duration`）并启动更新；无条件调用 `onlevelchangedfn`。[实现](../../../raw/current/scripts/components/bloomness.lua#L17-L50) |
| `Fertilize(value)` | `可选 number -> void` | 满级：经 `calcfullbloomdurationfn(inst, value, timer, full_bloom_duration)` 延时；未满：0 级先升 1 级、非绽放态拉满计时，累加 `fertilizer` 供速率回调使用。[实现](../../../raw/current/scripts/components/bloomness.lua#L67-L86) |
| `SetDurations(stage, full)` | `(number, number) -> void` | 设未满级/满级阶段时长；wormwood 用 `TUNING.WORMWOOD_BLOOM_STAGE_DURATION`（1 天）与 `WORMWOOD_BLOOM_FULL_DURATION`（3 天）。[实现](../../../raw/current/scripts/components/bloomness.lua#L52-L55) |
| `UpdateRate` | `-> void` | `rate = calcratefn(inst, level, is_blooming, fertilizer)`，缺省 1；计时按 `dt × rate` 扣减。[实现](../../../raw/current/scripts/components/bloomness.lua#L61-L65) |
| `OnUpdate/LongUpdate` | `(dt) -> void` | 计时归零时升级或降级；降级到 0 停止更新；LongUpdate 仅在计时非 0 时推进（跳时间兼容）。[实现](../../../raw/current/scripts/components/bloomness.lua#L88-L108) |
| `OnSave/OnLoad` | 存读钩子 | level>0 才落盘（level/timer/rate/is_blooming/fertilizer）；加载后重启更新并重放等级回调。[实现](../../../raw/current/scripts/components/bloomness.lua#L110-L133) |
| `GetLevel` | `-> number` | 读当前等级。[实现](../../../raw/current/scripts/components/bloomness.lua#L57-L59) |

## 典型接入

### 复用绽放状态机给自定义角色

- 前提与选择：主机端角色挂组件；`onlevelchangedfn` 必须在首次 SetLevel/Fertilize/OnLoad 前赋值，否则直接报错（[L49](../../../raw/current/scripts/components/bloomness.lua#L49)）。

```lua
local bloomness = inst:AddComponent("bloomness")
bloomness:SetDurations(TUNING.WORMWOOD_BLOOM_STAGE_DURATION, TUNING.WORMWOOD_BLOOM_FULL_DURATION)
bloomness.onlevelchangedfn = function(inst_, level)
    print(inst_.prefab, "bloom level =", level)
end
bloomness:Fertilize(1) -- 从 0 级催到 1 级并进入绽放
```

- 调整与验证：`is_blooming` 在"升到非满级"时为 true、达到满级或下降时为 false（[L34](../../../raw/current/scripts/components/bloomness.lua#L34)），速率回调可按它区分生长/凋谢方向。

## 常见陷阱与限制

- **onlevelchangedfn 未设即触发报错**：SetLevel 与 OnLoad 都直接调用该回调（[L49](../../../raw/current/scripts/components/bloomness.lua#L49)、[L130](../../../raw/current/scripts/components/bloomness.lua#L130)），组件不判空。
- **同值 SetLevel 提前返回**：等级没变时不重置 fertilizer 也不回调（[L19-L21](../../../raw/current/scripts/components/bloomness.lua#L19-L21)），想刷新计时要用 Fertilize。
- **计时方向语义**：`timer` 是"剩余时间"，绽放计时满时升级；施肥在非绽放态把 timer 直接拉到 `stage_duration`（[L79-L82](../../../raw/current/scripts/components/bloomness.lua#L79-L82)），再叠加 fertilizer 只影响速率。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；组件全文、wormwood 挂接与施肥路径、tuning.lua 数值（tuning.lua:4586-4587）、skilltree_wormwood 读点。
- 游戏运行：未运行。
