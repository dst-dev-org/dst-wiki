---
title: "月坛连接光效（moon_altar_link_fx）"
created: "2026-11-01"
updated: "2026-11-04"
type: prefab
summary: "月坛连接移动生成器与月脉冲在地面生成的随机动画光效。"
aliases: ["moon_altar_link_fx", "moon_altar_link_fx_spawner", "月坛连接光效", "月脉冲光点"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/fx.lua", "prefabs/moon_altar.lua", "prefabs/moonpulse.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# 月坛连接光效（moon_altar_link_fx）

## 快速判断

- 单一动画 FX，月坛连接线和月脉冲光点共同复用；连接与屏幕后处理由生成者负责。此页为单注册简单档；工厂端、播放和清理见 [[reference/mechanisms/generic-fx-factory#关键流程|通用 FX 工厂流程]]。

## 注册与构造

| 注册名 | 工厂与差异 |
|---|---|
| `moon_altar_link_fx` | 通用表代理与本地动画实体；bank、build 均为 `moon_altar_link_fx`，初始动画 `fx1`，无 sound；fn 以同一个随机数在小于 0.33 时改 `fx2`，否则小于 0.67 时改 `fx3`，其余保持 `fx1`。[条目](../../../raw/current/scripts/fx.lua#L2300-L2311) |

简中 `.po` 定向检索未命中该注册名显示名键；中文为效果描述，非官方译名。

## 关联机制导航

| 生成者 | 触发与证据 |
|---|---|
| 月坛连接生成器 | 主模拟找到有效另一月坛时创建 `moon_altar_link_fx_spawner` 并设目标；生成器每 0.2 秒朝目标前进 3 单位，在新位置 y=0 生成 FX；距目标不大于 2 时移除生成器。FX 为落点式，不父挂。[触发](../../../raw/current/scripts/prefabs/moon_altar.lua#L392-L397)、[步进](../../../raw/current/scripts/prefabs/moon_altar.lua#L674-L696)、[端分界](../../../raw/current/scripts/prefabs/moon_altar.lua#L700-L709) |
| 月脉冲 | 非专用服本地后处理链的 `SpawnWisp` 在有效本地玩家周围以随机角度和 4～36 距离落点（y=0）；小波设 8 次，大波设 17 次，后续任务间隔为 0.04 加随机 0～0.07 秒。玩家不存在或失效时停止本次续排。[生成](../../../raw/current/scripts/prefabs/moonpulse.lua#L19-L35)、[波次](../../../raw/current/scripts/prefabs/moonpulse.lua#L44-L75)、[端分界](../../../raw/current/scripts/prefabs/moonpulse.lua#L191-L212) |

## 常见陷阱与限制

- 月坛连接生成器是独立实体，不能将其到达后的移除误写成每个 FX 的移除；后者遵循 [[reference/mechanisms/generic-fx-factory#关键流程|通用工厂流程]]。[生成器](../../../raw/current/scripts/prefabs/moon_altar.lua#L677-L696)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394；核对注册字段、工厂与上述消费者；scripts 下按完整注册名及 `moon_altar_link` 检索，依赖声明不当生成点。未核验动画资源与模组动态生成。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-table-world-index|通用表 FX 索引]]。
