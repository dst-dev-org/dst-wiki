---
title: "knowndynamiclocations"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "在实体上按名字记忆坐标点（DynamicPosition）：点落在可走平台上时随平台漂移，平台被移除后回退世界坐标，可存档。"
aliases: ["记忆位置", "动态位置记忆", "knowndynamiclocations"]
tags: [component, lifecycle]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/knowndynamiclocations.lua", "util.lua", "prefabs/wagdrone_common.lua", "brains/wagdrone_rollingbrain.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# knowndynamiclocations

## 快速判断

- 职责与适用：给实体（机器人无人机等 AI）维护一组命名记忆点，点是 `DynamicPosition`——落在可走平台（船）上时跟随平台移动。[构造](../../../raw/current/scripts/components/knowndynamiclocations.lua#L1-L4)
- 优先使用：写 `RememberLocation(name, pt, dont_overwrite)`，删 `ForgetLocation(name)`，读 `GetLocation(name)`（世界坐标）或 `GetDynamicLocation(name)`（原始 DynamicPosition）。[Remember](../../../raw/current/scripts/components/knowndynamiclocations.lua#L22-L44)
- 重要副作用：为平台上的记忆点挂平台 `onremove` 监听，平台移除时把坐标固化回世界系并清 `walkable_platform`。[Watch](../../../raw/current/scripts/components/knowndynamiclocations.lua#L54-L64)
- 修改边界：坏坐标直接 error；`dont_overwrite` 为真时已有同名条目不覆盖；无任何事件广播，纯数据组件。[坏值处理](../../../raw/current/scripts/components/knowndynamiclocations.lua#L40-L42)

## 关键接口

| 接口／字段 | 已核验类型／单位 | 条件、作用与证据 |
|---|---|---|
| `RememberLocation(name, pt, dont_overwrite)` | 无返回 | `pt` 需含有限 x/y/z；`POPULATING` 期间只记 `local_pt` 并标 `loading`，平台挂接推迟到 `LoadPostPass`。[L22-L44](../../../raw/current/scripts/components/knowndynamiclocations.lua#L22-L44) |
| `GetLocation(name)` | Vector3 或 nil | 经 `DynamicPosition:GetPosition()` 解析当前世界坐标（平台活着则换算，平台没了返回剩余 local_pt）。[L12-L15](../../../raw/current/scripts/components/knowndynamiclocations.lua#L12-L15) |
| `GetDynamicLocation(name)` | DynamicPosition 或 nil | brain 用它拿原始对象再自行换算方向。[L17-L20](../../../raw/current/scripts/components/knowndynamiclocations.lua#L17-L20) |
| `ForgetLocation(name)` | 无返回 | 解绑平台监听并删除条目。[L46-L52](../../../raw/current/scripts/components/knowndynamiclocations.lua#L46-L52) |
| `DynamicPosition`（util） | Class | 构造时按 `GetPlatformAtPoint` 绑定平台并把点转 local 系；`GetPosition()` 平台无效时自清并回退。[util.lua](../../../raw/current/scripts/util.lua#L1121-L1131)、[GetPosition](../../../raw/current/scripts/util.lua#L1144-L1157) |

## 存读流程

- OnSave：所有点导出为 `{locations = {{name, x, y, z}}}`（y 为 0 时省略），坐标取的是解析后的世界坐标；空表不存。[OnSave](../../../raw/current/scripts/components/knowndynamiclocations.lua#L85-L100)
- OnLoad：逐条 `RememberLocation` 恢复；此时若点在平台上会重新绑定平台。[OnLoad](../../../raw/current/scripts/components/knowndynamiclocations.lua#L102-L108)
- LoadPostPass：把 `POPULATING` 期间挂起的 `loading` 条目重建为完整 DynamicPosition 并补平台监听——这就是"读档晚于平台生成"的接续点。[LoadPostPass](../../../raw/current/scripts/components/knowndynamiclocations.lua#L110-L118)
- 组件移除时解绑全部平台监听，避免悬挂回调。[OnRemoveFromEntity](../../../raw/current/scripts/components/knowndynamiclocations.lua#L6-L10)

## 典型接入

### 原版宿主：Wagstaff 无人机的部署点

- 唯一宿主：wagdrone_common 在无人机激活形态 `AddComponent("knowndynamiclocations")`，耗尽拆解时 `RemoveComponent`；`RememberDeployPoint`／`ForgetDeployPoint` 维护 `"deploypoint"` 条目。[挂载](../../../raw/current/scripts/prefabs/wagdrone_common.lua#L242)、[写入](../../../raw/current/scripts/prefabs/wagdrone_common.lua#L173-L179)
- 消费者：wagdrone_rollingbrain 的 `GetDeployPoint` 用 `GetDynamicLocation("deploypoint")` 决定巡逻目标；点随船漂移，无人机能跟回甲板上的部署点。[brain](../../../raw/current/scripts/brains/wagdrone_rollingbrain.lua#L45-L47)
- 全库核查：`grep knowndynamiclocations` 仅命中上述 wagdrone 三文件与组件本身，无其他原版宿主或读者。

### 模组：记住生成/返回点

- 前提与选择：服务端实体 + 需要跨船跟随的坐标记忆；纯静态点用普通存档字段更简单。
- 生效与撤销：`RememberLocation` 默认覆盖同名；`dont_overwrite = true` 实现"只在第一次记录"。

```lua
-- 服务端实体上
inst:AddComponent("knowndynamiclocations")
inst.components.knowndynamiclocations:RememberLocation("home", inst:GetPosition())
local pt = inst.components.knowndynamiclocations:GetLocation("home") -- Vector3 或 nil
```

- 调整与验证：示例仅语法检查未运行；`name` 为任意字符串 key，坐标单位为世界单位。

## 常见陷阱与限制

- 生成期（worldgen）写点必须走组件的 POPULATING 分支约定，直接在 `POPULATING` 为真时传世界坐标是安全的，但平台绑定要等 `LoadPostPass`；自己绕开组件伪造 entry 会跳过该修复。[L29-L37](../../../raw/current/scripts/components/knowndynamiclocations.lua#L29-L37)
- 平台被移除后点退化为固定世界坐标且不再绑定新平台；`GetPosition()` 返回剩余 local_pt（此时等于世界坐标），语义变化需消费方自行接受。[util GetPosition](../../../raw/current/scripts/util.lua#L1144-L1157)
- OnSave 固化的是"当时"的世界坐标，读档后重新判定平台绑定；船在两次会话间移动过则记忆点不跟随旧船，而是按读档时所在位置重新解析。[OnSave](../../../raw/current/scripts/components/knowndynamiclocations.lua#L91-L97)

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文（含 POPULATING 分支、平台监听、存读三钩）与 util.lua 的 DynamicPosition 构造/GetPosition；核对 wagdrone 挂载、写入/擦除与 brain 消费；未覆盖世界生成期 `LoadPostPass` 的实际调用时序（引擎侧）。
- 游戏运行：未运行；本页结论均来自源码静态核验。
- 结构检查：`validate_page.py`、`validate_source_links.py --strict` 与示例 Lua 语法检查通过（见批记录）。
