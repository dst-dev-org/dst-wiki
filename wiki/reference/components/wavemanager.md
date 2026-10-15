---
title: "wavemanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "客户端水面波纹装饰管理器：围绕本地玩家按地皮类型每秒生成浪花/波光特效，支持注册遮挡物抑制生成。"
aliases: ["波浪管理器", "wavemanager", "水面波纹", "浪花生成"]
tags: [component, callback]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/wavemanager.lua", "prefabs/forest.lua", "prefabs/oceanwhirlbigportal.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# wavemanager

## 快速判断

- 职责与适用：森林世界专用、**仅非专用服务器（即客户端）安装**（`if not TheNet:IsDedicated()` 分支）的世界组件；构造即 `StartUpdatingComponent`，OnUpdate 只在 `ThePlayer` 存在时围绕其 30～75（随相机距离）半径撒波纹特效。[安装](../../../raw/current/scripts/prefabs/forest.lua#L627-L637) [构造](../../../raw/current/scripts/components/wavemanager.lua#L106-L128)
- 优先使用：抑制某区域波纹用 `RegisterBlocker(inst, dist)`／`UnregisterBlocker(inst)`——原版大漩涡传送门用它清空周围浪花。[注册](../../../raw/current/scripts/components/wavemanager.lua#L156-L162) [消费者](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L92-L100)
- 重要副作用：按地皮表 `shimmer` 每类地皮每秒约 80 次尝试，乘全局 `shimmer_per_sec_mod` 与相机距离倍率（1x～1.5x）；生成物是 wave_shimmer／wave_shore／wave_shimmer_med／wave_shimmer_deep 纯特效实体。[生成选择](../../../raw/current/scripts/components/wavemanager.lua#L78-L104)
- 修改边界：生成物落点必须 `canSpawn` 通过（地皮匹配＋不在任何 blocker 距离内）；`shimmer_per_sec_mod <= 0` 时组件自停，再置正值需重新 StartUpdating。[门禁](../../../raw/current/scripts/components/wavemanager.lua#L142-L154) [自停](../../../raw/current/scripts/components/wavemanager.lua#L193-L195)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `RegisterBlocker(inst, dist)`／`UnregisterBlocker(inst)` | `dist: number（半径）` | blockers 表按实体记距离，落点进入任一 blocker 半径即拒绝；注册方自管生命周期（原版漩涡在 onremove 也注销）。[实现](../../../raw/current/scripts/components/wavemanager.lua#L156-L162) [注销](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L140) |
| `shimmer` | 地皮表 | 按 `WORLD_TILES.OCEAN_*`（COASTAL_SHORE/COASTAL/BRINEPOOL/WATERLOG 走岸浪分支，SWELL/ROUGH/HAZARDOUS 走中/深水波光），字段 `per_sec`/`spawn_rate`/`tryspawn`。[表](../../../raw/current/scripts/components/wavemanager.lua#L109-L118) |
| `shimmer_per_sec_mod` | `number，默认 1.0` | 全局生成速率乘数；≤0 触发自停。[使用](../../../raw/current/scripts/components/wavemanager.lua#L174-L175) |
| `ripple_per_sec`／`ripple_idle_time` | `10`／`5` | 构造赋值后在本文件内无任何读取，属未接线字段。[定义](../../../raw/current/scripts/components/wavemanager.lua#L120-L121) |

## 常见陷阱与限制

- **不是服务端系统：** 专用服务器没有该组件；世界生成、鱼群等任何服务端逻辑都读不到它，也别指望它产生可交互实体（生成物均为纯视觉特效）。[安装分支](../../../raw/current/scripts/prefabs/forest.lua#L627-L637)
- **岸浪朝向来自地块邻居：** `GetWaveBearing` 检查周围两圈地块，近岸返回朝向角度，深海返回 true（用 shimmer），无地则 false；它操作的是 tile 坐标而非实体位置。[求向](../../../raw/current/scripts/components/wavemanager.lua#L1-L76)
- **OnUpdate 无 ThePlayer 直接返回：** 主视角缺失（过场/专用逻辑）时静默不生成，不报错也不重试。[门禁](../../../raw/current/scripts/components/wavemanager.lua#L164-L170)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（202 行），核验 forest.lua 安装分支与 oceanwhirlbigportal 的 blocker 消费。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 本批 16 页通过。
- 游戏运行：未运行。
