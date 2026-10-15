---
title: "despawnfader"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "实体淡出移除组件：主模拟触发 FadeOut 后 1 秒内渐隐，进度经 net_tinybyte 同步客户端，结束时移除实体。"
aliases: ["despawnfader", "淡出移除", "渐隐消失", "despawn淡出"]
tags: [component, network]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/despawnfader.lua", "prefabs/daywalker.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# despawnfader

## 快速判断

- 职责与适用：给需"淡出后消失"的 NPC 用；FadeOut 后每帧 `fadeval -= dt`（1 秒渐隐），用二次缓动写 `AnimState:OverrideMultColour` 透明度。([despawnfader.lua:40-46](../../../raw/current/scripts/components/despawnfader.lua#L40-L46)
- 优先使用：主模拟直接 `FadeOut()`；客户端无需干预，进度由网络变量还原。([daywalker.lua:813-815](../../../raw/current/scripts/prefabs/daywalker.lua#L813-L815)
- 重要副作用：FadeOut 立即 `AddTag("NOCLICK")`、`persists=false`，且组件无取消接口；主模拟 fadeval 到 0 即 `inst:Remove()`。([despawnfader.lua:29-38](../../../raw/current/scripts/components/despawnfader.lua#L29-L38) ([despawnfader.lua:46-52](../../../raw/current/scripts/components/despawnfader.lua#L46-L52)
- 修改边界：非主模拟监听 netvar dirty 起更还原；OnRemoveFromEntity 只在非主模拟摘监听。([despawnfader.lua:1-9](../../../raw/current/scripts/components/despawnfader.lua#L1-L9) ([despawnfader.lua:23-27](../../../raw/current/scripts/components/despawnfader.lua#L23-L27)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `FadeOut()` | 无参（服务端调用） | fadeval=1 起更；若已在更新中则不重复加 NOCLICK／persists 标记。([despawnfader.lua:29-38](../../../raw/current/scripts/components/despawnfader.lua#L29-L38) |
| `_fade` | net_tinybyte `despawnfader._fade`，事件 `fadedirty` | 主模拟每帧 `set(_local)` 同步 `floor(7*fadeval+0.5)`（7 档）；首帧用完整 set。[despawnfader.lua:14](../../../raw/current/scripts/components/despawnfader.lua#L14 ([despawnfader.lua:53-58](../../../raw/current/scripts/components/despawnfader.lua#L53-L58) |
| `OnUpdate(dt)` | 缓动：`k = 1-(1-fadeval)^2` | alpha 从 1 降到 0；到 0 时主模拟 Remove，客户端停更等网络移除。([despawnfader.lua:40-52](../../../raw/current/scripts/components/despawnfader.lua#L40-L52) |

## 典型接入

### NPC 定时淡出退场

- 前提与选择：宿主需 AnimState；由存读计时器等外部逻辑决定时机（原版 daywalker 在 despawn 计时到点、醒着时调用）。([daywalker.lua:808-815](../../../raw/current/scripts/prefabs/daywalker.lua#L808-L815)

```lua
inst:AddComponent("despawnfader")

-- 需要退场时（服务端）：
inst.components.despawnfader:FadeOut()
```

- 生效与撤销：组件添加本身无副作用；FadeOut 不可逆，需保留实体就不要触发。

## 常见陷阱与限制

- 淡出时长固定 1 秒（fadeval 初值 1、逐帧减 dt），无时长参数；要变速需自行另写组件。[despawnfader.lua:30](../../../raw/current/scripts/components/despawnfader.lua#L30 [despawnfader.lua:42](../../../raw/current/scripts/components/despawnfader.lua#L42
- 客户端还原精度只有 8 档（value()/7），短实体上渐隐比主模拟略糙；`fadedirty` 只在淡出期间更新。[despawnfader.lua:4](../../../raw/current/scripts/components/despawnfader.lua#L4
- 已淡出实体 NOCLICK 且不入存档，但主模拟 1 秒内仍存在：期间碰撞等物理特性未变，只是不可点击。([despawnfader.lua:34-35](../../../raw/current/scripts/components/despawnfader.lua#L34-L35)

## 验证记录

- 源码审阅：2026-10-28，`steam-buildid:26230496`：审阅组件全文与 daywalker／daywalker2 的触发点、安装位置。
- 游戏运行：未运行；未核对客户端 8 档还原的视觉差异。
- 结构检查：validate_page.py 通过；validate_source_links.py --strict 通过（本批 16 页合并执行）。
