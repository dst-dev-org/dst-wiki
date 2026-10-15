---
title: "oceancolor"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "海面颜色组件：客户端按昼夜相位渐变 Map 清屏色与海面纹理混合度，纯表现层。"
aliases: ["海面颜色", "海洋着色", "oceancolor"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/oceancolor.lua", "prefabs/world.lua", "gamelogic.lua", "update.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# oceancolor

## 快速判断

- 职责与适用：挂在世界实体（仅非专用端，[挂接](../../../raw/current/scripts/prefabs/world.lua#L558)），监听 `phasechanged` 并把 `TheWorld.Map` 的清屏色与海面纹理混合度在 default/dusk/night/no_ocean 四组参数间渐变（[颜色表](../../../raw/current/scripts/components/oceancolor.lua#L1-L7)）。
- 优先使用：世界生成后由 gamelogic 调 `Initialize(world.has_ocean)`（[调用](../../../raw/current/scripts/gamelogic.lua#L433-L434)）；有海才启动逐帧混合。
- 重要副作用：直接写引擎侧 `Map:SetClearColor`／`SetOceanTextureBlendAmount`（[应用](../../../raw/current/scripts/components/oceancolor.lua#L64-L66)）。
- 修改边界：纯客户端表现组件，无存读钩子；不修改水本身的物理或可通行性。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `COLORS` | `table`：default/dusk/night/no_ocean，各含 `color`(RGBA 0-1)、`blend_delay`、`blend_speed`、`ocean_texture_blend` | default 与 dusk 取自 `TUNING.OCEAN_SHADER.*`；night/no_ocean 为纯黑且延迟 6 秒（[表](../../../raw/current/scripts/components/oceancolor.lua#L1-L7)）。 |
| `Initialize(has_ocean)` | `has_ocean: bool -> 无` | 有海时启动 WallUpdate 并设 default 清屏色；无海只设 no_ocean 清屏色、不启动更新（[实现](../../../raw/current/scripts/components/oceancolor.lua#L28-L35)）。 |
| `OnPostInit()` | `-> 无` | 兼容性修正：先写 blend 1 再写当前值，强制客户端读档进白天时刷新（[实现](../../../raw/current/scripts/components/oceancolor.lua#L37-L43)）；由世界 PostInit 通用循环调用（([world.lua:342-346](../../../raw/current/scripts/prefabs/world.lua#L342-L346)）。 |
| `OnWallUpdate(dt)` | `-> 无` | 按延迟与速度在 start/end 颜色与纹理混合度间 lerp 并应用到 Map（[实现](../../../raw/current/scripts/components/oceancolor.lua#L45-L67)）。 |
| `OnPhaseChanged(src, phase)` | `phase: string -> 无` | 相位切换时重置 lerp 起点／终点与延迟速度（[实现](../../../raw/current/scripts/components/oceancolor.lua#L69-L86)）。 |

## 典型接入

### 相位渐变链路（原版参照）

- 前提与选择：组件在构造时监听宿主的 `phasechanged`（[监听](../../../raw/current/scripts/components/oceancolor.lua#L12)）；世界必须带该组件（原版在 world prefab 非 dedicated 分支挂载，[挂接](../../../raw/current/scripts/prefabs/world.lua#L558)）。
- 生效与撤销：相位变化即开始渐变；`ocean_texture_blend` 在 dusk 起切到 1（启用海面纹理混合），白天回到 0（[参数](../../../raw/current/scripts/components/oceancolor.lua#L3-L4)）。

```lua
-- 参考自用：在世界客户端 PostInit 后手动触发一次重绘（组件等价行为）
local oc = TheWorld.components.oceancolor
if oc ~= nil then
    TheWorld.Map:SetOceanTextureBlendAmount(1)
    TheWorld.Map:SetOceanTextureBlendAmount(oc.current_ocean_texture_blend)
end
```

- 调整与验证：自定义 TUNING.OCEAN_SHADER 颜色即可影响 default/dusk 档；night/no_ocean 为硬编码黑色。

## 常见陷阱与限制

- **构造时的 StartUpdatingComponent 是空转**：组件没有 `OnUpdate`，update 循环只调用存在 OnUpdate 的组件（[守卫](../../../raw/current/scripts/update.lua#L261-L262)）；真正逐帧入口是 `OnWallUpdate`，仅 `Initialize(true)` 后启动。
- **死索引写入**：`OnPhaseChanged` 写 `start_color[0]`（Lua 索引 0），无任何读取者（[写入](../../../raw/current/scripts/components/oceancolor.lua#L74)），改造时可直接忽略该行。
- **仅非专用端挂载**：专用服务器没有该组件，服务端代码不要引用（[挂接条件](../../../raw/current/scripts/prefabs/world.lua#L557-L560)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、world 挂接与 PostInit 循环、gamelogic Initialize 调用、update 循环的 OnUpdate 守卫。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
