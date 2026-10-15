---
title: "spawnfader"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "生成淡入/淡出组件：主模拟设置 net 渐隐状态，双端驱动 AnimState:OverrideMultColour 做 1 秒渐显/渐隐，结束时推 spawnfaderin/spawnfaderout 事件。"
aliases: ["生成淡入", "出生渐显", "spawnfader"]
tags: [component, network]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/spawnfader.lua", "prefabs/bunnyman.lua", "components/shadowparasitemanager.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# spawnfader

## 快速判断

- 职责与适用：双端组件——服务器 `FadeIn`/`FadeOut` 设 `_fade`（net_tinybyte）与 `_fadeout`（net_bool）（components/spawnfader.lua:36-68）；客户端监听 `fadedirty` 与 `death`（死亡立即 Cancel）（:23-26）。
- 优先使用：生成时淡入→`FadeIn()`；即将移除的实体淡出→`FadeOut()`。原版生成者调用方如 hounded（components/hounded.lua:451-452）、moonbeastspawner（components/moonbeastspawner.lua:166）、klaus（prefabs/klaus.lua:305）；兔人等寄生宿主由 shadowparasitemanager 触发（components/shadowparasitemanager.lua:10、:159-160）。
- 重要副作用：渐变期间 AddTag `NOCLICK`，淡入结束时移除（:48、:65-66、:106-108）；结束分别推 `"spawnfaderout"`/`"spawnfaderin"`（:104、:109）。
- 修改边界：约 1 秒完成（fadeval 7 档按 FRAMES 递减，:77-78）；`highlightchildren` 一并染色（:94-98）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `FadeIn`/`FadeOut` | `() -> nil` | 主模拟 `set` net 值；非主模拟 `set_local`（客户端实体自驱兜底）；fadeval 置 1 并启动更新。:36-68 |
| `Cancel` | `() -> nil` | 立即结束当前渐变（fadeval 清零走完 OnUpdate）；死亡时客户端自动调用（:11-13）。:70-75 |
| `OnUpdate` | `(dt) -> nil` | fadeval 线性递减，方向开关 `_fadeout` 决定二次缓动方向，作用于 `AnimState:OverrideMultColour(1,1,1,k)`。:77-112 |
| `_fade`/`_fadeout` | net_tinybyte/net_bool | 脏事件 `fadedirty`；7 档量化（:18-19、:1-9）。 |

## 典型接入

### 原版参照：寄生宿主淡入

- 前提与选择：宿主列表含 bunnyman（shadowparasitemanager.lua:10 权重表 :18），生成宿主后 FadeIn 使其渐显登场——组件挂载与触发分离的典型。
- 生效与撤销：淡出结束不自动移除实体，需监听 `spawnfaderout` 自行处理（prefabs/woby_commands_classified.lua:612-616 即此用法）。
- 无 mod 接入案例，不提供代码。

## 常见陷阱与限制

- `_fadeout` 是方向开关：渐变中途改调 FadeIn/FadeOut 只改方向不重置进度，透明度会跳变。
- 淡出完成不移除 NOCLICK（只移除 tag 的分支在淡入侧，:106-108），淡出实体保持不可点。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 114 行，核验挂载点抽样（bunnyman.lua:392、hound.lua:508、klaus.lua:525 等）与 shadowparasitemanager 触发链（:10/:18/:159-160）。
- 游戏运行：未运行。

## 相关链接

- 反向场景（渐隐后移除实体）：[[reference/components/despawnfader]]。
