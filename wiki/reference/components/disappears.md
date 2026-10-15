---
title: "disappears"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "倒计时消失组件：延迟（默认 25 秒）后播放消失动画并移除实体，被拾取或事件可取消倒计时。"
aliases: ["自动消失", "消失倒计时", "disappears"]
tags: [component, item, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/disappears.lua", "prefabs/ash.lua", "prefabs/furtuft.lua", "prefabs/spoiledfood.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# disappears

## 快速判断

- 职责与适用：给会腐坏风化的实体（灰烬、毛絮、变质食物）挂"倒计时后播动画并自删"的行为；倒计时被 `PrepareDisappear` 反复重掷，被 `StopDisappear` 取消（[全文](../../../raw/current/scripts/components/disappears.lua#L68-L77)）。
- 优先使用：宿主在 master 端挂组件后直接改 `sound`／`anim` 字段定制表现（([ash.lua:65-67](../../../raw/current/scripts/prefabs/ash.lua#L65-L67)）；开始／停止倒计时走 `PrepareDisappear`／`StopDisappear`。
- 重要副作用：触发后 `persists=false`、加 `NOCLICK`、禁止拾取（[触发](../../../raw/current/scripts/components/disappears.lua#L34-L40)），不再存档。
- 修改边界：组件移除等价 `StopDisappear`（[别名](../../../raw/current/scripts/components/disappears.lua#L85)）；已触发消失后不可逆转（isdisappear 守卫，[守卫](../../../raw/current/scripts/components/disappears.lua#L13-L16)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `delay` | `number`，默认 `25` | 基础延迟秒数；实际延迟为 `delay + math.random()*10`，每次 Prepare 重掷（[实现](../../../raw/current/scripts/components/disappears.lua#L74-L75)）。 |
| `anim` / `sound` | `string` / `string\|nil` | 消失动画名（默认 `"disappear"`）与音效；为直接字段赋值（[字段](../../../raw/current/scripts/components/disappears.lua#L3-L6)）。 |
| `PrepareDisappear()` | `-> 无` | 重置并启动倒计时任务，到期自动 `Disappear`（[实现](../../../raw/current/scripts/components/disappears.lua#L68-L77)）。 |
| `StopDisappear()` | `-> 无` | 取消倒计时任务并清 `tasktotime`；已触发时为空操作（[实现](../../../raw/current/scripts/components/disappears.lua#L51-L61)）。 |
| `Disappear()` | `-> 无` | 触发消失：调 `disappearFn`、实体睡眠中直接 Remove，否则播动画并双重兜底删除（animover 监听 + 动画时长后 DoTaskInTime）（[实现](../../../raw/current/scripts/components/disappears.lua#L12-L49)）。 |

## 典型接入

### 掉落倒计时／拾取暂停（原版参照）

- 前提与选择：毛絮 furtuft 在掉落时 `PrepareDisappear`、放入背包时 `StopDisappear`，配合组件挂接（([furtuft.lua:6-12](../../../raw/current/scripts/prefabs/furtuft.lua#L6-L12)、[挂接](../../../raw/current/scripts/prefabs/furtuft.lua#L36)）。
- 生效与撤销：灰烬拾取即停表（([ash.lua:15-17](../../../raw/current/scripts/prefabs/ash.lua#L15-L17)）；变质食物按下雨状态切换 Prepare／Stop（([spoiledfood.lua:180-187](../../../raw/current/scripts/prefabs/spoiledfood.lua#L180-L187)）。

```lua
-- 服务端：会风化的掉落物
inst:AddComponent("disappears")
inst.components.disappears.sound = "dontstarve/common/dust_blowaway"
inst.components.disappears.anim = "disappear"
inst:ListenForEvent("ondropped", function(inst)
    inst.components.disappears:PrepareDisappear()
end)
inst:ListenForEvent("onputininventory", function(inst)
    inst.components.disappears:StopDisappear()
end)
```

- 调整与验证：`delay` 单位秒；动画必须存在于宿主 build 中，否则消失流程无可见表现（删除兜底仍生效）。

## 常见陷阱与限制

- **回调字段名前后不一致**：构造初始化的是 `disappearsFn`（[字段](../../../raw/current/scripts/components/disappears.lua#L4)），但 `Disappear` 实际读取的是 `disappearFn`（[调用](../../../raw/current/scripts/components/disappears.lua#L25-L27)）——要用消失前回调必须赋值 `self.disappearFn`，初始化的 `disappearsFn` 是死字段。
- **双重删除兜底**：动画可能因离屏暂停，因此 animover 监听之外还有 `动画时长+0.1` 秒的定时删除（[兜底](../../../raw/current/scripts/components/disappears.lua#L46-L48)）；自定义宿主不要在消失期间再做 `persist` 相关操作。
- **已触发的消失不可取消**：`StopDisappear` 与再次 Prepare 都被 `isdisappear` 守卫拦下（[守卫](../../../raw/current/scripts/components/disappears.lua#L51-L55)）。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文与 ash／furtuft／spoiledfood 三个宿主的挂接、参数与调用时机。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
