---
title: "gym"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "卡墩训练器械逻辑：接收受训者、按 timer 倒计时循环训练并在结束时施加训练效果，夜晚自动休息。"
aliases: ["卡墩健身房", "gym", "训练器械"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/gym.lua", "prefabs/rat_gym.lua", "stategraphs/SGrat_gym.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# gym

## 快速判断

- 职责与适用：挂在训练器械（`rat_gym` 系列）上，管理唯一受训者（trainee）；依赖 `timer` 组件计时（[依赖](../../../raw/current/scripts/prefabs/rat_gym.lua#L321-L323)），训练效果经 `SetTrainFn` 注入。
- 优先使用：四种原版器械只是换 `trainfn`（方向/速度/耐力/反应，[分派](../../../raw/current/scripts/prefabs/rat_gym.lua#L324-L331)），自定义训练同样只写一个回调。
- 重要副作用：训练循环向自身推 `rest`/`starttraining`/`endtraining` 事件驱动状态图（[推事件](../../../raw/current/scripts/components/gym.lua#L59-L63)），并周期设置 `_musicstate` 网络变量播放训练曲。
- 修改边界：夜相位强制休息、白天自动恢复（[相位](../../../raw/current/scripts/components/gym.lua#L119-L127)）；受训者死亡或移除即终止（[监听](../../../raw/current/scripts/components/gym.lua#L32-L39)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetTrainee(inst)` | 实体 | 登记受训者并监听其 `onremove`/`death`（[实现](../../../raw/current/scripts/components/gym.lua#L32-L39)）。 |
| `SetTrainFn(fn)` | `fn(inst, trainee)` | 训练结束时的效果回调；组件先自增 `trainee.training` 计数再调用（[调用](../../../raw/current/scripts/components/gym.lua#L109-L117)）。 |
| `SetOnRemoveTraineeFn(fn)` | `fn(inst)` | 失去受训者时通知宿主（[实现](../../../raw/current/scripts/components/gym.lua#L13-L21)）。 |
| `StartTraining(inst, time)` | time 可省 | 启动 `training` 计时器，默认 `TUNING.CARRAT_GYM.TRAINING_TIME`（[启动](../../../raw/current/scripts/components/gym.lua#L52-L67)）。 |
| `StopTraining()` / `RemoveTrainee()` | 无参数 | 停止循环并取消全部周期任务／注销受训者（[实现](../../../raw/current/scripts/components/gym.lua#L69-L85)、[注销](../../../raw/current/scripts/components/gym.lua#L17-L30)）。 |
| `CheckPerish()` | 内部周期 5 秒 | 受训者耐久低于 10% 时 30% 概率临时休息（[实现](../../../raw/current/scripts/components/gym.lua#L87-L100)）。 |
| `OnSave` / `LoadPostPass` | 存读钩子 | 存剩余时间；读档后重开训练（[存读](../../../raw/current/scripts/components/gym.lua#L129-L142)）。 |

## 典型接入

### 自定义训练效果

- 前提与选择：服务器 master 段注册 `timer` + `gym`；效果写在 `trainfn`，不要在回调里自己累加计数。
- 生效与撤销：每轮 `timerdone("training")` 自动 `StopTraining` + `Train`（[轮次](../../../raw/current/scripts/components/gym.lua#L102-L107)）；`RemoveTrainee` 会连带停掉计时。
- 原版依据：四个器械的 `trainfn` 与失败回调注入见 [rat_gym](../../../raw/current/scripts/prefabs/rat_gym.lua#L323-L333)；状态图消费 `rest`/`starttraining` 事件（[状态图](../../../raw/current/scripts/stategraphs/SGrat_gym.lua#L50-L61)）。

```lua
inst:AddComponent("timer") -- gym 依赖 timer 计时
inst:AddComponent("gym")
inst.components.gym:SetTrainFn(function(inst, trainee)
    if trainee.components.health ~= nil then
        trainee.components.health:DoDelta(1) -- 每轮训练的效果
    end
end)
inst.components.gym:SetOnRemoveTraineeFn(function(inst)
    -- 受训者离开时的清理
end)
inst.components.gym:SetTrainee(trainee)
inst.components.gym:StartTraining(inst)
```

- 调整与验证：默认训练时长为 3 个时段（[调参](../../../raw/current/scripts/tuning.lua#L5204)）；`trainee.training` 计数由组件维护，回调内直接读取即可。

## 常见陷阱与限制

- 夜晚开始训练会立即转 `rest` 且不取消计时器（[守卫](../../../raw/current/scripts/components/gym.lua#L59-L63)）：计时到点照样结算 `Train`，需要夜训无效时要自己加门控。
- `PushMontage` 直接写 `inst._musicstate`（[写法](../../../raw/current/scripts/components/gym.lua#L45-L50)），非 `rat_gym` 宿主没有该网络变量会报错；复用组件时需自建同名变量或去掉音乐任务。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、rat_gym 四器械注入、SGrat_gym 事件消费与相关 tuning；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
