---
title: "petrifiable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "可石化标记：树木等实体注册进世界石化系统，被 forestpetrification 批量石化后链式波及 8 距离内同伴，支持唤醒倒计时。"
aliases: ["可石化", "petrifiable", "树木石化"]
tags: [component, lifecycle, event]
dst_build: "steam-buildid:26230496"
sources: ["components/petrifiable.lua", "components/forestpetrification.lua", "prefabs/evergreens.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# petrifiable

## 快速判断

- 职责与适用：挂在可石化的自然实体（原版非 twiggy 树，[evergreens](../../../raw/current/scripts/prefabs/evergreens.lua#L918-L919)）；构造即向 TheWorld 推 `ms_registerpetrifiable` 注册（[L16](../../../raw/current/scripts/components/petrifiable.lua#L16)），世界侧 forestpetrification 监听并管理石化周期（[监听](../../../raw/current/scripts/components/forestpetrification.lua#L200-L201)）。
- 优先使用：驱动交给世界组件——`PetrifyForest` 对选中实体调 `Petrify(false)`（[forestpetrification](../../../raw/current/scripts/components/forestpetrification.lua#L160)）；宿主只需 `SetPetrifiedFn` 定义石化表现。
- 重要副作用：石化有链式传播——一个实体石化后 0.6~1.8 秒内波及 8 距离内其他可石化实体（[常量](../../../raw/current/scripts/components/petrifiable.lua#L1-L3)、[链式](../../../raw/current/scripts/components/petrifiable.lua#L40-L51)）；`Petrify` 内部先走 `OnRemoveFromEntity`，届时会注销世界注册并摘掉 `petrifiable` 标签（[L94-L98](../../../raw/current/scripts/components/petrifiable.lua#L94-L98)）。
- 修改边界：petrified 状态与唤醒倒计时跨存档（OnSave/OnLoad）；唤醒后不会自动复原，复原表现由宿主在 `onPetrifiedFn` 体系外自行实现。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `Petrify(immediate)` | `可选 bool -> void` | 默认（nil/true）立即执行石化表现；显式传 false 时：睡着的实体挂 `entitywake` 监听并排 (2+rand)×TOTAL_DAY_TIME 唤醒倒计时，醒着的实体在 `math.random()*TUNING.SEG_TIME`（SEG_TIME=30）内随机时刻石化。[实现](../../../raw/current/scripts/components/petrifiable.lua#L94-L110) |
| `SetPetrifiedFn(fn)` | `fn(inst) -> void` | 石化时刻的表现回调（含链式传播触发的每次石化）。[实现](../../../raw/current/scripts/components/petrifiable.lua#L90-L92) |
| `IsPetrified` | `-> bool` | 当前是否已石化。[实现](../../../raw/current/scripts/components/petrifiable.lua#L86-L88) |
| `OnRemoveFromEntity` | 组件移除钩子 | 取消石化/唤醒任务；未石化时推 `ms_unregisterpetrifiable` 并摘 `petrifiable` 标签。[实现](../../../raw/current/scripts/components/petrifiable.lua#L70-L84) |
| `OnSave/OnLoad` | 存读钩子 | 已石化才落盘剩余唤醒时间；加载重走 `Petrify(false)`，有剩余时间恢复倒计时、否则链式石化。[实现](../../../raw/current/scripts/components/petrifiable.lua#L112-L128) |

## 典型接入

### 让自定义"树"参与石化事件

- 前提与选择：主机端实体挂组件并设表现回调；注册进 forestpetrification 由组件构造自动完成，无需手动推事件。
- 生效与撤销：世界石化周期命中该实体后按延迟策略石化；链式传播只找带本组件且有排队任务的实体（[CanChainPetrify](../../../raw/current/scripts/components/petrifiable.lua#L33-L37)）。

```lua
inst:AddComponent("petrifiable")
inst.components.petrifiable:SetPetrifiedFn(function(inst_)
    -- 石化表现由宿主实现（原版树在此切换石化动画并停用交互）
end)
```

- 调整与验证：链式半径与延迟是文件内常量（CHAIN_RADIUS=8、0.6~1.8 秒），无 TUNING 入口；测试可手动 `inst.components.petrifiable:Petrify()`。

## 常见陷阱与限制

- **immediate 语义反直觉**：nil 视为 true 立即石化，只有显式 `Petrify(false)` 才走延迟/唤醒分支（[L100-L101 注释](../../../raw/current/scripts/components/petrifiable.lua#L100-L101)）。
- **石化即注销**：`Petrify` 先调 `OnRemoveFromEntity`，此时 `petrified` 仍为 false，故世界注册与标签都会被移除；之后组件再被移除不会重复注销，但实体也不再被石化系统选中。
- **唤醒倒计时只挂在睡着的实体**：醒着时传 false 走的是随机短延迟而非 (2~3) 天倒计时，两个分支（[L103-L108](../../../raw/current/scripts/components/petrifiable.lua#L103-L108)）语义不同。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；组件全文、evergreens 挂接、forestpetrification 注册与驱动点。
- 游戏运行：未运行。
