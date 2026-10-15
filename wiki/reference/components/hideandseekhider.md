---
title: "hideandseekhider"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "捉迷藏藏匿者：让生物（原版 kitcoon）向藏匿点发起躲藏、超时入点、被找出或中止，按世界天次限制每日游玩次数。"
aliases: ["捉迷藏藏匿者", "hideandseekhider"]
tags: [component, callback, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/hideandseekhider.lua", "prefabs/kitcoon.lua", "prefabs/kitcoonden.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# hideandseekhider

## 快速判断

- 职责与适用：挂在会玩的生物上（原版 [kitcoon](../../../raw/current/scripts/prefabs/kitcoon.lua#L291-L295)），由藏匿点或管理方调用 `GoHide` 发起；到时或立即把实体藏进 spot，被找出/中止后回调释放。
- 优先使用：地面藏匿点机制配合 [[reference/components/hideandseekhidingspot]]；发起方原版是 kitcoonden（[L182-L183](../../../raw/current/scripts/prefabs/kitcoonden.lua#L182-L183)）与节日活动 setup。
- 重要副作用：`GoHide` 会给藏匿点动态加 hideandseekhidingspot 组件并记录 `last_day_played`（[L38-L43](../../../raw/current/scripts/components/hideandseekhider.lua#L38-L43)）。
- 修改边界：存档经 OnSave/LoadPostPass 恢复隐藏中状态；每天一次由 `CanPlayHideAndSeek`（世界天次比较）控制（[L66-L68](../../../raw/current/scripts/components/hideandseekhider.lua#L66-L68)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GoHide(hiding_spot, timeout_time, isloading)` | `(entity, 可选 number, 可选 bool) -> bool` | 未在藏时生效；给 spot 动态加组件并 `SetHider`；`timeout_time==0` 立即藏入，否则 DoTaskInTime 后入点；成功返回 true。[实现](../../../raw/current/scripts/components/hideandseekhider.lua#L37-L64) |
| `Found(doer)` | `doer -> void` | 清 `hiding_spot`、取消跑位任务并触发 `OnFound(inst, doer)`；spot 侧由 hideandseekhidingspot 反向调用。[实现](../../../raw/current/scripts/components/hideandseekhider.lua#L70-L83) |
| `Abort()` | `-> void` | 调 spot 的 Abort 后按无主处理 `Found(nil)`。[实现](../../../raw/current/scripts/components/hideandseekhider.lua#L85-L93) |
| `IsPlaying` | 字段判断 | `hiding_spot ~= nil`；无原版消费者直接调用。[实现](../../../raw/current/scripts/components/hideandseekhider.lua#L11-L13) |
| `gohide_timeout` | number（默认 3） | `GoHide` 未传 timeout 时的默认秒数；kitcoon 设为 `TUNING.KITCOON_HIDEANDSEEK_HIDETIMEOUT`（=3，tuning.lua:5269）。[默认值](../../../raw/current/scripts/components/hideandseekhider.lua#L7) |
| `OnHide/OnFound/StartGoingToHidingSpot` | 可选回调 | 入点时 / 被找出时 / 开始跑位时触发，均由宿主赋值。[回调点](../../../raw/current/scripts/components/hideandseekhider.lua#L27-L30) |
| `OnSave/LoadPostPass` | 存读钩子 | 保存 last_day_played、spot GUID 与剩余 timeout；加载恢复或重走 GoHide（isloading=true）。[实现](../../../raw/current/scripts/components/hideandseekhider.lua#L95-L126) |

## 典型接入

### 让自定义小动物玩捉迷藏

- 前提与选择：宿主只需加组件、设回调；藏匿点实体可以是任意实体（组件会补挂 spot 组件）。发起时机由宿主 AI 或事件决定。
- 生效与撤销：`GoHide` 成功后实体在 timeout 后进入 spot；找到（`Found`）或 spot 移除触发 `OnFound` 后释放。

```lua
inst:AddComponent("hideandseekhider")
inst.components.hideandseekhider.gohide_timeout = 3
inst.components.hideandseekhider.OnFound = function(inst_, doer)
    print(inst_.prefab, "was found by", tostring(doer))
end
-- 发起（主机端，传入任意候选点实体）：
-- inst.components.hideandseekhider:GoHide(spot_ent, 3)
```

- 调整与验证：kitcoon 的完整接线见 [kitcoon.lua](../../../raw/current/scripts/prefabs/kitcoon.lua#L291-L295)；`timeout_time=0` 表示加载恢复或立即藏入，不要当普通延时使用。

## 常见陷阱与限制

- **IsHidden 引用未定义的 `inst`**：函数体用 `inst.runtohidingspot_task` 判断（[L15-L17](../../../raw/current/scripts/components/hideandseekhider.lua#L15-L17)），但该方法作用域没有 `inst`（构造参数是局部变量），且全库未发现调用者；模组不要依赖此方法，自写 `self.hiding_spot ~= nil and self.inst.runtohidingspot_task == nil` 等价判断。
- **每日一次靠天次计数**：`last_day_played` 在 GoHide 成功时写入当天 `TheWorld.state.cycles`（[L39](../../../raw/current/scripts/components/hideandseekhider.lua#L39)），同一天内重复发起会在管理方 `CanPlayHideAndSeek` 处被拒。
- **spot 组件是动态加的**：对没有 hideandseekhidingspot 的实体调 GoHide 会直接改变其实体组件构成，撤销依赖 Abort/Found 路径。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；组件全文、kitcoon 挂接、kitcoonden 发起点与 hideandseekhidingspot 反向调用。
- 游戏运行：未运行。
