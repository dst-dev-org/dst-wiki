---
title: "transparentonsanity"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "客户端专用组件：按本地玩家理智度把宿主渐隐为半透明，并同步压低其音量。"
aliases: ["理智透明", "随理智渐隐", "transparentonsanity"]
tags: [component, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/transparentonsanity.lua", "prefabs/gestalt.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# transparentonsanity

## 快速判断

- 职责与适用：影子系生物/投影类 FX 随观察者（`ThePlayer`）理智或自定义函数渐隐；文件头明确"client side component, no server logic"（[头注](../../../raw/current/scripts/components/transparentonsanity.lua#L1-L2)），原版都在 `not TheNet:IsDedicated()` 分支安装（[示例](../../../raw/current/scripts/prefabs/gestalt.lua#L157-L164)）。
- 优先使用：想让生物对低理智玩家若隐若现，装组件后覆盖 `most_alpha`/`osc_*` 并给 `calc_percent_fn` 即可。
- 重要副作用：`PushAlpha` 同时 `OverrideMultColour`、`SoundEmitter:OverrideVolumeMultiplier` 并回调 `onalphachangedfn`（[实现](../../../raw/current/scripts/components/transparentonsanity.lua#L4-L12)）。
- 修改边界：实体睡眠停更新、唤醒 `ForceUpdate`（[睡眠](../../../raw/current/scripts/components/transparentonsanity.lua#L28-L35)）；构造即强制 `UsePointFiltering(true)` 并把 alpha 压到 0（[构造](../../../raw/current/scripts/components/transparentonsanity.lua#L23-L25)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `most_alpha` | `number`，默认 `0.4` | 透明度上限（相对不透明的最大 alpha），各类影怪在 0.2~1.0 间覆盖。 |
| `osc_amp` / `osc_speed` / `offset` | `number`，默认 `0.25` / `0.25+rand*2` / `math.random()` | 正弦摆动幅度、频率与相位（[构造](../../../raw/current/scripts/components/transparentonsanity.lua#L14-L21)）。 |
| `calc_percent_fn` | `function(inst, observer) -> number or nil` | 自定义透明度占比；返回 nil 时回退到"理智模式且 `1 - 理智%`"（[回退](../../../raw/current/scripts/components/transparentonsanity.lua#L61-L66)）。 |
| `onalphachangedfn` | `function(inst, alpha, most_alpha)` | 每次实际改 alpha 后回调，常用于同步子实体头部件。 |
| `ForceUpdate()` | 无参数 | 以 dt=0 立即重算并应用 alpha，安装后必须调用（[实现](../../../raw/current/scripts/components/transparentonsanity.lua#L41-L43)）。 |
| `DoUpdate(dt, force)` | 每帧更新 | alpha 以 1/秒 线性逼近目标值（[实现](../../../raw/current/scripts/components/transparentonsanity.lua#L76-L89)）。 |

目标透明度 = `pct * most_alpha * (1 + osc_amp * (sin(offset*osc_speed) - 1))`；宿主 replica.combat 的目标恰为 ThePlayer 时直接取 `most_alpha`（[计算](../../../raw/current/scripts/components/transparentonsanity.lua#L45-L74)）。

## 典型接入

### 影怪随玩家理智渐隐（客户端）

- 前提与选择：宿主需有 `AnimState`，可选 `SoundEmitter`；在 prefab 客户端分支（非 dedicated）内安装。
- 生效与撤销：组件随实体生命周期持续渐隐，无存档与网络变量；移除组件即停止。
- 原版依据：gestalt 用 `Client_CalcSanityForTransparency`（月狂模式曲线，[函数](../../../raw/current/scripts/prefabs/gestalt.lua#L27-L33)）+ `SetHeadAlpha` 回调（[安装](../../../raw/current/scripts/prefabs/gestalt.lua#L158-L164)）。

```lua
-- prefab 的 common_postinit 内（仅客户端需要，可在 not TheNet:IsDedicated() 分支）
inst:AddComponent("transparentonsanity")
inst.components.transparentonsanity.most_alpha = .4
inst.components.transparentonsanity.calc_percent_fn = Client_CalcSanityForTransparency
inst.components.transparentonsanity:ForceUpdate()
```

- 调整与验证：`calc_percent_fn` 返回 0~1 占比；不提供且玩家非理智模式时目标恒为 0（[守卫](../../../raw/current/scripts/components/transparentonsanity.lua#L62-L73)）。

## 常见陷阱与限制

- 内部方法名拼写为 `CalcaulteTargetAlpha`（源码如此，[定义](../../../raw/current/scripts/components/transparentonsanity.lua#L45)）；仅组件内部调用，但覆写/反射时注意原拼写。
- `PushAlpha` 的音量乘数是 `alpha / most_alpha`：`most_alpha` 覆盖过小会让音量比例异常，改动后需实测听觉表现。
- 组件只驱动表现层，服务器战斗、AI 不受 alpha 影响；不要在服务器端读取其状态。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文与 gestalt/gestalt_guard/fused_shadeling/ruins_shadeling 等 11 个消费 prefab 的安装参数；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。

相关：[sanity](sanity.md)
