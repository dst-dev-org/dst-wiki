---
title: "sizetweener"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "对宿主 Transform 做等比缩放渐变补间，结束时归位目标缩放并派发事件与回调。"
aliases: ["缩放渐变", "尺寸补间", "sizetweener"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/sizetweener.lua", "prefabs/mossling.lua", "stategraphs/SGmossling.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# sizetweener

## 快速判断

- 职责与适用：把宿主从当前缩放匀速补间到目标缩放（x/y/z 等比），适合生物变身、旋转放大等演出；原版唯一消费者是 mossling（[安装](../../../raw/current/scripts/prefabs/mossling.lua#L154)）。
- 优先使用：stategraph 动作里直接 `StartTween(目标缩放, 秒, 回调)`，如 mossling 的 spin_loop（[调用](../../../raw/current/scripts/stategraphs/SGmossling.lua#L329)）。
- 重要副作用：启动派发 `sizetweener_start`、结束派发 `sizetweener_end` 事件（[StartTween](../../../raw/current/scripts/components/sizetweener.lua#L44)、[EndTween](../../../raw/current/scripts/components/sizetweener.lua#L29)）。
- 修改边界：仅改 Transform 缩放，不写存档；宿主移除或组件重装即丢失补间状态，无恢复机制。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `StartTween(size, time, callback)` | `size: number；time: number；callback: fn(inst) 或 nil` | 以当前 `Transform:GetScale()` 为起点；`time > 0` 时启动组件更新，`time <= 0` 立即 EndTween。[实现](../../../raw/current/scripts/components/sizetweener.lua#L37-L51) |
| `EndTween()` | 无参数 | 停止更新、直接设为目标缩放、清状态，先派发 `sizetweener_end` 再调 callback。[实现](../../../raw/current/scripts/components/sizetweener.lua#L13-L35) |
| `OnUpdate(dt)` | 组件更新循环 | `Lerp(i_size, t_size, timepassed/time)` 后 `SetScale(s, s, s)` 等比应用；累计到 time 时收尾。[实现](../../../raw/current/scripts/components/sizetweener.lua#L53-L61) |

## 典型接入

### 生物演出中平滑放大／还原

- 前提与选择：宿主需有 Transform（实体默认有）；在 master_postinit 或 stategraph onenter 调用，与 mossling 相同入口。
- 生效与撤销：StartTween 即开始逐帧插值；到达目标或再次 StartTween 前持续生效，EndTween 后缩放停在目标值，不会自动还原。
- 原版依据：mossling 在 spin_loop 进入时 `StartTween(1.55, 2)`（[SGmossling](../../../raw/current/scripts/stategraphs/SGmossling.lua#L329)）。

```lua
-- prefab 的 master_postinit 内
inst:AddComponent("sizetweener")

-- stategraph 状态 onenter 内：2 秒内放大到 1.55，结束后还原为 1
inst.components.sizetweener:StartTween(1.55, 2, function(inst)
    inst.components.sizetweener:StartTween(1, 0.5)
end)
```

- 调整与验证：size 为最终等比缩放值，time 为秒；callback 在 `sizetweener_end` 事件之后调用。嵌套补间可只靠回调串联，无需手动 StopUpdating。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖全部接口与 mossling 消费链；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
