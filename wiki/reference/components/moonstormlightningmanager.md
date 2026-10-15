---
title: "moonstormlightningmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "月风暴地面闪电的客户端表现管理器：按相机距离调节落雷密度，在世界节点内可视半径随机落 moonstorm_ground_lightning_fx。"
aliases: ["moonstormlightningmanager", "月风暴闪电管理器", "月风暴落雷"]
tags: [component, ui]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/moonstormlightningmanager.lua", "prefabs/forest.lua", "components/moonstorms.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# moonstormlightningmanager

## 快速判断

- 职责与适用：森林世界的客户端组件（`forest.lua:626-634` 仅在 `not TheNet:IsDedicated()` 时挂载）。监听 `moonstorm_nodes_dirty_relay`：有活跃月风暴节点时开启更新，每帧在玩家周围可视半径内随机取点，过地面与节点双重校验后落雷特效。[挂载](../../../raw/current/scripts/prefabs/forest.lua#L626-L634) [事件开关](../../../raw/current/scripts/components/moonstormlightningmanager.lua#L29-L35)
- 优先使用：模组无需手动接入——组件挂在 forest 世界 prefab 上随月风暴状态自启停；要改变表现只覆盖其字段（如 `spark.checkfn`/`spawnfn`），不要重复 AddComponent。[字段注入点](../../../raw/current/scripts/components/moonstormlightningmanager.lua#L22)
- 重要副作用：落雷频率随相机距离缩放（30–100 距离映射半径 30–75、倍率 1–1.5），越拉近镜头落雷越密；仅表现层，不造成伤害（伤害在 `moonstorm_lightning` 等实体）。[距离映射](../../../raw/current/scripts/components/moonstormlightningmanager.lua#L38-L48)
- 修改边界：依赖 `ThePlayer`（OnUpdate 开头判空返回）与 `TheWorld.net.components.moonstorms` 的节点净值；洞穴等无 moonstorms 网络组件的世界读该路径会出错。[净值读取](../../../raw/current/scripts/components/moonstormlightningmanager.lua#L10)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `spark` | `{per_sec=5, spawn_rate=0, checkfn, spawnfn}` | 核心配置：每秒基准 5 次、累计器、落点校验函数与生成函数，均可替换注入。[构造默认](../../../raw/current/scripts/components/moonstormlightningmanager.lua#L22) |
| `checkground(inst, map, x, y, z)` | `-> boolean` | `IsVisualGroundAtPoint` 且节点索引在 `moonstorms._moonstorm_nodes` 净值数组内才允许落雷。[实现](../../../raw/current/scripts/components/moonstormlightningmanager.lua#L7-L17) |
| `SpawnLightning(inst, x, y, z)` | `-> nil` | 生成 `moonstorm_ground_lightning_fx` 并随机朝向；即 `spark.spawnfn` 默认值。[实现](../../../raw/current/scripts/components/moonstormlightningmanager.lua#L1-L5) |
| `sparks_per_sec_mod` | number（默认 1.0） | 全局频率倍率；归零会在下一帧停止更新。当前脚本库无外部写入者。[使用点](../../../raw/current/scripts/components/moonstormlightningmanager.lua#L63-L75) |
| `sparks_per_sec` / `sparks_idle_time` | number（默认 1 / 5） | 已声明但未接入 OnUpdate 逻辑的预留字段。[声明](../../../raw/current/scripts/components/moonstormlightningmanager.lua#L24-L25) |
| `OnUpdate(dt)` | 更新回调 | 累加 `spawn_rate += per_sec * mod * 相机倍率 * dt`，每满 1 次掷点并按 checkfn 决定是否 spawnfn。[主循环](../../../raw/current/scripts/components/moonstormlightningmanager.lua#L50-L76) |

## 典型接入

### 月风暴表现随净值启停（原版参照）

- 前提与选择：节点集合的权威在服务端 moonstorms 世界网络组件（`components/moonstorms.lua`）：其 L23-25 把净值 dirty 事件转发为世界事件 `moonstorm_nodes_dirty_relay`，本组件据此启停更新。[净值转发](../../../raw/current/scripts/components/moonstorms.lua#L23-L25)
- 生效与撤销：进入/离开月风暴（节点数组空/非空）即停/启；`sparks_per_sec_mod <= 0` 也会停止，需重新 >0 且再收 relay 事件才会恢复。

```lua
-- 客户端：换掉落雷表现的注入示例（在 forest 世界组件上操作）
local mgr = TheWorld.components.moonstormlightningmanager
if mgr then
    mgr.spark.spawnfn = function(inst, x, y, z)
        local fx = SpawnPrefab("moonstorm_ground_lightning_fx")
        fx.Transform:SetPosition(x, y, z)
    end
end
```

- 调整与验证：`spark.per_sec`（基准 5）与 `sparks_per_sec_mod` 相乘后仍受相机倍率（1–1.5）缩放；改频率优先调这两个字段。

## 常见陷阱与限制

- **非森林世界复用报错：** `checkground` 硬读 `TheWorld.net.components.moonstorms._moonstorm_nodes`，目标世界缺 moonstorms 网络组件时直接崩；移植必须连 `spark.checkfn` 一起换。[硬依赖](../../../raw/current/scripts/components/moonstormlightningmanager.lua#L10)
- **落点不保证避开玩家：** 校验只有"可视地面 + 节点内"，半径内可能掷到玩家脚下；组件纯表现层，无伤害逻辑，别把它当战斗机制。
- **字段命名误导：** `sparks_per_sec`/`sparks_idle_time` 是未接线的预留字段，调它们没有效果；实际频率由 `spark.per_sec` 与 `sparks_per_sec_mod` 控制。[未使用字段](../../../raw/current/scripts/components/moonstormlightningmanager.lua#L24-L25)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（82 行），核对 forest.lua:633 挂载条件、moonstorms 净值转发链、`moonstorm_ground_lightning_fx` prefab 存在（`prefabs/moonstorm_ground_lightning_fx.lua`）；全库 grep 确认组件无其他消费者、预留字段无写入者。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。
