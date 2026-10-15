---
title: "canopyshadows"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "在树冠宿主周围按 4 单位网格铺叶子遮阴贴图（ShadeRenderer），用全局引用计数让重叠树冠共享遮阴。"
aliases: ["树冠遮阴", "叶子阴影", "canopyshadows"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/canopyshadows.lua", "shadeeffects.lua", "prefabs/watertree_pillar.lua", "prefabs/oceantree_pillar.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# canopyshadows

## 快速判断

- 职责与适用：水源杉/海洋树等树冠宿主安装后，在圆形范围内随机铺 `SpawnLeafCanopy` 遮阴贴图；原版消费者仅 watertree_pillar 与 oceantree_pillar（[安装](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L439-L440)）。
- 优先使用：想复用同款树冠遮阴直接 AddComponent 并按树冠尺寸覆盖 `range`。
- 重要副作用：遮阴贴图登记在全局表 `Global_Canopyshadows`（键 `"x-z"`），重叠树冠按 `refs`/`spawnrefs` 双计数共享（[全局表](../../../raw/current/scripts/components/canopyshadows.lua#L27-L28)）。
- 修改边界：构造后 `DoTaskInTime(0)` 自动生成并铺放；组件移除双向计数回收（[移除](../../../raw/current/scripts/components/canopyshadows.lua#L21-L25)）；不写存档、无网络变量。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `range` | `number`，默认 `math.floor(TUNING.SHADE_CANOPY_RANGE/4)`（tuning.lua:6334 为 28，即 7 格） | 方形枚举半径，圆形判定后落点对齐 `floor(坐标/4)*4+2` 网格；海洋树用 `SHADE_CANOPY_RANGE_SMALL`（22→5 格）（[消费](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L553-L554)）。[构造](../../../raw/current/scripts/components/canopyshadows.lua#L13) |
| `GenerateCanopyShadowPositions()` | 无参数 | 80% 随机率、`range²` 圆内生成候选点并累计 `refs`（[实现](../../../raw/current/scripts/components/canopyshadows.lua#L30-L51)）。 |
| `SpawnShadows()` / `DespawnShadows(ignore_entity_sleep)` | 无参 / `boolean` | 按 `spawnrefs` 增删 ShadeRenderer 贴图；首个引用才真建（[铺放](../../../raw/current/scripts/components/canopyshadows.lua#L77-L90)、[回收](../../../raw/current/scripts/components/canopyshadows.lua#L92-L106)）。 |
| `RemoveCanopyShadowPositions()` | 无参数 | 逐点减 `refs`，减到 0 删全局表键（[实现](../../../raw/current/scripts/components/canopyshadows.lua#L53-L63)）。 |

## 典型接入

### 大型树木挂树冠遮阴

- 前提与选择：树 prefab 内在 `not TheNet:IsDedicated()` 分支安装；dedicated 端 `SpawnLeafCanopy` 是空函数（[空实现](../../../raw/current/scripts/shadeeffects.lua#L1-L7)），组件本身不判端。
- 生效与撤销：实体睡眠自动 `DespawnShadows`、唤醒重铺（[睡眠](../../../raw/current/scripts/components/canopyshadows.lua#L65-L75)）；海洋树在 `hascanopydirty`（树冠丢失）时移除组件（[动态移除](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L556-L559)）。
- 原版依据：watertree_pillar 保持默认 range，oceantree_pillar 覆盖小 range（[宿主](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L439-L440)）。

```lua
-- 树 prefab 的客户端分支内
inst:AddComponent("canopyshadows")
inst.components.canopyshadows.range = math.floor(TUNING.SHADE_CANOPY_RANGE / 4)
```

- 调整与验证：贴图旋转/缩放由 ShadeRenderer 的 `ShadeTypes.LeafCanopy` 配置与 `TUNING.CANOPY_*` 控制（[配置](../../../raw/current/scripts/shadeeffects.lua#L10-L26)），组件只管落点。

## 常见陷阱与限制

- `SpawnShadows` 直接索引 `Canopyshadows[x.."-"..z]` 不判空（[实现](../../../raw/current/scripts/components/canopyshadows.lua#L82-L83)）：先 `SpawnShadows` 后再 `GenerateCanopyShadowPositions` 的乱序调用会索引 nil 报错，按组件正常生命周期使用。
- 遮阴是纯视觉（shadeeffects 在 dedicated 端空实现），但全局表 `Global_Canopyshadows` 只存在于客户端世界；模组别跨端读写。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖全部接口、shadeeffects 渲染链与 watertree_pillar/oceantree_pillar 消费链、tuning 取值；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。

相关：[canopylightrays](canopylightrays.md)
