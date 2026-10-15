---
title: "colouraddersync"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "附加色网络同步：把 RGBA 附加色打包进 net_uint 从主机同步到客户端，双端回调统一应用，供特效类实体着色。"
aliases: ["附加颜色同步", "colouraddersync", "染色同步"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/colouraddersync.lua", "components/colouradder.lua", "prefabs/bat_boss.lua", "prefabs/hats.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# colouraddersync

## 快速判断

- 职责与适用：挂在使用附加色（AddColour）着色的实体或特效上；主机端 `SyncColour` 写网络变量并本地应用，客户端解包后经回调应用，解决 AnimState 颜色不跨端的问题（[组件全文](../../../raw/current/scripts/components/colouraddersync.lua#L1-L23)）。
- 优先使用：由 colouradder 组件驱动的实体自动同步（colouradder 调 `SyncColour`，[驱动](../../../raw/current/scripts/components/colouradder.lua#L33-L34)）；无 colouradder 的特效（蝙蝠首领阴影、帽子跟随特效）直接调 `SyncColour`（[蝙蝠](../../../raw/current/scripts/prefabs/bat_boss.lua#L650)、[帽子](../../../raw/current/scripts/prefabs/hats.lua#L7805)）。
- 重要副作用：`SetColourChangedFn` 设置回调后立即用当前值回调一次，保证晚加入客户端也应用现有颜色（[回调](../../../raw/current/scripts/components/colouraddersync.lua#L25-L28)）。
- 修改边界：打包量化到每通道 8 位（0-255），高精度颜色会损失；net 变量名 `colouraddersync.colour`，dirty 事件 `colourdirty`（[网络变量](../../../raw/current/scripts/components/colouraddersync.lua#L17)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SyncColour(r, g, b, a)` | `四个 number（0-1）-> 无；主机端调用` | 打包 `r*0x1000000 + g*0x10000 + b*0x100 + a`（各通道 +0.5 取整）写入 net 变量；主机同时 `AnimState:SetAddColour` 并回调 `colourchangedfn`。[实现](../../../raw/current/scripts/components/colouraddersync.lua#L36-L49) |
| `SetColourChangedFn(fn)` | `fn(inst, r, g, b, a)` | 存回调并立即以当前 net 值回调一次；双端都应设置。[实现](../../../raw/current/scripts/components/colouraddersync.lua#L25-L28) |
| `ForceRefresh()` | `-> 无` | 手动再触发一次解包回调，用于强制重放当前颜色。[实现](../../../raw/current/scripts/components/colouraddersync.lua#L31-L33) |
| `colour` | `net_uint` | RGBA 四通道压缩在 32 位无符号整数内；客户端监听 `colourdirty` 解包（除以 255 归一）后回调。[解包](../../../raw/current/scripts/components/colouraddersync.lua#L1-L13) |

## 典型接入

### 让特效实体的附加色跨端一致

- 前提与选择：特效实体主机端加组件；主机驱动写色、双端设回调，客户端应用由回调完成（原版帽子跟随特效即此结构，[挂接](../../../raw/current/scripts/prefabs/hats.lua#L7805)、[回调接线](../../../raw/current/scripts/prefabs/hats.lua#L7765)）。
- 生效与撤销：主机调 `SyncColour` 后客户端经 `colourdirty` 自动应用；组件移除无特殊清理，颜色残留需宿主自行复位。

```lua
-- 主机端：驱动颜色（客户端经 net_uint 自动同步）
inst.components.colouraddersync:SyncColour(0.2, 0.4, 0.6, 1)
-- 双端：接收最终颜色并应用
inst.components.colouraddersync:SetColourChangedFn(function(inst, r, g, b, a)
    inst.AnimState:SetAddColour(r, g, b, a)
end)
```

- 调整与验证：回调在主机端会收到两次通知路径（SyncColour 直调与 net 回调同一函数），幂等处理即可；`SetColourChangedFn` 的立即回调可替代手动 ForceRefresh 做初值应用。

## 常见陷阱与限制

- **8 位精度**：每通道量化到整数（[打包](../../../raw/current/scripts/components/colouraddersync.lua#L37-L42)），渐变动画会有 1/255 台阶；对精度敏感的着色需另选方案。
- **客户端不自动改 AnimState**：主机端 SyncColour 顺带 SetAddColour（[实现](../../../raw/current/scripts/components/colouraddersync.lua#L43-L45)），客户端只回调 `colourchangedfn`——客户端回调里不写应用代码就只有主机有色。
- **依赖客户端监听初始化**：客户端监听在组件构造时挂（[构造](../../../raw/current/scripts/components/colouraddersync.lua#L20-L22)），晚于 net 首包的场景靠 SetColourChangedFn 的立即回调兜底，宿主务必设置回调。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文（打包/解包/回调）、colouradder 三个驱动点、bat_boss 与 hats 两类宿主挂接。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
