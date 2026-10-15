---
title: "colourtweener"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "对宿主 AnimState 的 SetMultColour 做 RGBA 线性补间，支持完成回调与 WallUpdate 模式。"
aliases: ["颜色补间", "渐变变色", "colourtweener"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/colourtweener.lua", "prefabs/worm_boss_util.lua", "prefabs/atrium_ritual_marking.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# colourtweener

## 快速判断

- 职责与适用：把宿主 `AnimState` 的 RGBA 从当前值线性过渡到目标值，常用于整体黑化/还原、淡入淡出；纯表现层（[补间](../../../raw/current/scripts/components/colourtweener.lua#L82-L103)）。
- 优先使用：一次性变色过渡直接 `StartTween({r,g,b,a}, 时长, 回调)`；回调常用来移除组件或切换状态。
- 重要副作用：开始/结束分别推 `colourtweener_start`/`colourtweener_end` 事件（[事件](../../../raw/current/scripts/components/colourtweener.lua#L69)、[事件](../../../raw/current/scripts/components/colourtweener.lua#L38)）。
- 修改边界：`usewallupdate` 为真时走 WallUpdate（暂停也继续），否则普通每帧更新（[分派](../../../raw/current/scripts/components/colourtweener.lua#L105-L106)）；补间结束自动停更新。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `StartTween(colour, time, callback, usewallupdate)` | `(table{r,g,b,a}, number, function(inst), boolean)` | 起点取当前 `GetMultColour`，目标缺项补 1；`time<=0` 立即结束（[实现](../../../raw/current/scripts/components/colourtweener.lua#L54-L80)）。 |
| `IsTweening()` | → `boolean` | 补间进行中判断（[实现](../../../raw/current/scripts/components/colourtweener.lua#L22-L24)）。 |
| `EndTween()` | 无参数 | 应用目标色（四个目标值全真才写）、执行回调、推 `colourtweener_end` 并停更新（[实现](../../../raw/current/scripts/components/colourtweener.lua#L27-L45)）。 |
| `colour` | 长度 ≥3 的数组，缺省项补 `1` | `UnpackColour` 对 nil 或长度 <3 返回 `1,1,1,1`（[解包](../../../raw/current/scripts/components/colourtweener.lua#L47-L52)）。 |

## 典型接入

### 实体整体渐变（以暗黑化→还原为例）

- 前提与选择：宿主需 `AnimState`；组件可常驻也可按需动态增删——蠕虫 Boss 转阶段时 AddComponent 补间、回调内 RemoveComponent（[增删](../../../raw/current/scripts/prefabs/worm_boss_util.lua#L1592-L1594)）。
- 生效与撤销：补间立即开始，覆盖期间外部 `SetMultColour` 会被下一帧插值覆盖；结束后颜色停在目标值。
- 原版依据：蠕虫 Boss 12 帧转黑、5 帧还原（[转黑](../../../raw/current/scripts/prefabs/worm_boss_util.lua#L1555-L1571)）；中庭仪式印记 2 帧转黑后延迟还原（[消费](../../../raw/current/scripts/prefabs/atrium_ritual_marking.lua#L35-L36)）；沃特比变身淡入（[消费](../../../raw/current/scripts/prefabs/walter.lua#L741)）。

```lua
-- 常驻组件写法：master_postinit 内安装，之后随时调用
inst:AddComponent("colourtweener")
inst.components.colourtweener:StartTween({0, 0, 0, 1}, 0.5, function(inst_)
    -- 补间完成回调；1 秒后还原
    inst_.components.colourtweener:StartTween({1, 1, 1, 1}, 1.0)
end)
```

- 调整与验证：time 单位秒（原版常用 `n * FRAMES`）；暂停敏感的 UI 实体传 `usewallupdate = true` 保证暂停时也走完。

## 常见陷阱与限制

- `EndTween` 只在 `t_colour_r/g/b/alpha` 四个目标值都为真时才写最终色（[守卫](../../../raw/current/scripts/components/colourtweener.lua#L30-L32)）：Lua 中 0 也为真，数值颜色无碍；但传了含 nil 的表会被 `UnpackColour` 整体替换为白色，注意别依赖"部分通道不变"。
- 起点色的读取依赖 `#i_colour > 0`：`GetMultColour` 正常返回 4 值，`AnimState` 缺失的宿主上调用会在构造/更新时报 nil 索引。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文与 worm_boss_util/atrium_ritual_marking/charliecutscene/walter 消费链；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
