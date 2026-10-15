---
title: "blowinwind"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "随风飘行：给带 locomotor 的实体注入风向速度并持续 WalkForward，风滚草等无脑飘移体的驱动器。"
aliases: ["随风移动", "blowinwind", "风滚草驱动"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/blowinwind.lua", "prefabs/tumbleweed.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# blowinwind

## 快速判断

- 职责与适用：把"风"建模为单位向量，逐帧积分出速度并映射到 locomotor 的 walkspeed，再 `WalkForward` 实现飘移；原版唯一消费者是风滚草（[消费](../../../raw/current/scripts/prefabs/tumbleweed.lua#L460)）。
- 优先使用：需要顺风乱飘、不需要寻路的实体（滚动/飘动类）。
- 重要副作用：直接改写 `locomotor.walkspeed` 并覆盖朝向（[驱动](../../../raw/current/scripts/components/blowinwind.lua#L147-L152)）；实体休眠自动停、唤醒自动续（[休眠](../../../raw/current/scripts/components/blowinwind.lua#L33-L39)）。
- 修改边界：需 [[reference/components/locomotor]]；无网络变量，位置同步交给实体自身。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `Start(ang, vel)` | 弧度角、可省倍率 | 设风向/初速、转身并启动更新与音效循环（[实现](../../../raw/current/scripts/components/blowinwind.lua#L56-L67)）。 |
| `Stop()` | 无参数 | 停音效与更新（[实现](../../../raw/current/scripts/components/blowinwind.lua#L69-L72)）。 |
| `ChangeDirection(ang, vel)` | 弧度角 | 换风向后不立即转身，速度自然偏转（[实现](../../../raw/current/scripts/components/blowinwind.lua#L74-L80)）。 |
| `SetMaxSpeedMult(spd)` / `SetMinSpeedMult(spd)` / `SetAverageSpeed(spd)` | 数值 | 默认 1.5 / 0.5 /（跑速+走速）/2（[默认](../../../raw/current/scripts/components/blowinwind.lua#L10-L12)）。 |
| `GetSpeed()` / `GetVelocity()` | → `number` / `Vector3` | 读当前速度标量与方向向量（[实现](../../../raw/current/scripts/components/blowinwind.lua#L94-L100)）。 |
| `soundPath` / `soundName` / `soundParameter` | 字符串字段 | 循环音效及速度参数，`Start`/`Stop` 时启停（[音效](../../../raw/current/scripts/components/blowinwind.lua#L41-L54)）。 |
| `OnUpdate(dt)` | 自动周期 | 速度积分、正弦速度起伏与驱动 locomotor（[更新](../../../raw/current/scripts/components/blowinwind.lua#L106-L153)）。 |

## 典型接入

### 让自定义实体随风飘

- 前提与选择：服务器侧注册；实体需有 locomotor，且不装斥力类移动控制。
- 生效与撤销：`Start` 后持续飘移；休眠/`Stop` 即停，位置由存档里的角度与速度恢复（[读档](../../../raw/current/scripts/prefabs/tumbleweed.lua#L252-L258)）。
- 原版依据：风滚草装配音效字段、监听世界 `windchange` 并以世界风角启动（[装配](../../../raw/current/scripts/prefabs/tumbleweed.lua#L460-L472)）。

```lua
inst:AddComponent("locomotor")
inst:AddComponent("blowinwind")
inst.components.blowinwind:SetAverageSpeed(4)
inst.components.blowinwind.soundPath = "dontstarve_DLC001/common/tumbleweed_roll"
inst.components.blowinwind.soundName = "tumbleweed_roll"
inst.components.blowinwind.soundParameter = "speed"
inst.components.blowinwind:Start(math.random() * 2 * math.pi) -- 起始风向（弧度）
```

- 调整与验证：速度在 0 与 averageSpeed 间随向量长度映射，再加 5±2 秒周期的正弦起伏（[起伏](../../../raw/current/scripts/components/blowinwind.lua#L126-L135)）。

## 常见陷阱与限制

- 与 [[reference/components/locomotor]] 的其他寻路/控制写入同一 `walkspeed` 字段，装了 brains 或寻路逻辑会互相覆盖。
- `windAngle` 只在 `Start` 时应用到朝向，之后由速度向量决定转身（[积分](../../../raw/current/scripts/components/blowinwind.lua#L113-L115)）；只调 `ChangeDirection` 不会立即掉头。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文与 tumbleweed 消费链；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
