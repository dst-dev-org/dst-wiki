---
title: "undertile"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级临时地皮底层数据网格：为临时覆盖 tile 记忆被压住的原 tile，供码头、桥、造地等系统读写与还原。"
aliases: ["地皮底层", "临时地皮记忆", "undertile"]
tags: [component, worldgen]
dst_build: "steam-buildid:26230496"
sources: ["components/undertile.lua", "prefabs/world.lua", "prefabs/dock_kit.lua", "components/terraformer.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# undertile

## 快速判断

- 职责与适用：挂在 TheWorld 上的 DataGrid，坐标即世界 tile 格，存"被临时 tile 覆盖前的 tile id"；仅服务端存在（[断言](../../../raw/current/scripts/components/undertile.lua#L10)）。
- 优先使用：放置/移除临时地皮的实体负责 Set/Clear，读取方（放置校验、造地）用 GetTileUnderneath；消费方含码头（[dock_kit 写入](../../../raw/current/scripts/prefabs/dock_kit.lua#L112-L114)）与造地机（[terraformer 读回](../../../raw/current/scripts/components/terraformer.lua#L24)）。
- 重要副作用：数据网格等 `worldmapsetsize` 世界事件才初始化（[初始化](../../../raw/current/scripts/components/undertile.lua#L31-L35)）；tile id 对应 GROUND_ISTEMPTILE 临时层语义（[常量](../../../raw/current/scripts/constants.lua#L776)）。
- 修改边界：整格粒度、无事件通知；写读全部即时生效，跨档经 OnSave 压缩串与 tile id 转换表还原。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GetTileUnderneath(x, y)` | `x, y: tile 坐标` → `tile id 或 nil` | 读被覆盖的原 tile；网格未初始化前读取无保护。[实现](../../../raw/current/scripts/components/undertile.lua#L41-L43) |
| `SetTileUnderneath(x, y, tile)` | `tile: tile id` | 写入/覆盖记忆值；放在 SetTile 之后调以免被造地回调清掉（[ropebridge 顺序注释](../../../raw/current/scripts/components/ropebridgemanager.lua#L143-L145)）。[实现](../../../raw/current/scripts/components/undertile.lua#L45-L47) |
| `ClearTileUnderneath(x, y)` | 无返回 | 置 nil 撤销记忆。[实现](../../../raw/current/scripts/components/undertile.lua#L49-L51) |
| `OnSave`／`OnLoad(data)` | 存读钩子 | OnSave 用 `ZipAndEncodeSaveData` 压缩；OnLoad 解压后经 `TheWorld.tile_id_conversion_map` 把旧 id 映射到当前 id（[载入](../../../raw/current/scripts/components/undertile.lua#L65-L75)）。 |

## 典型接入

### 自定义临时地皮实体

- 前提与选择：实体在落位/回收时按自己的 tile 坐标写/清 undertile；读取方需要 tile id 判断可否放置时直接查本组件。
- 生效与撤销：Set 后该格永久带底层记忆（含存档）；Clear 或实体回收时撤销；与 dock／bridge 等系统互不感知，重叠覆盖需接入方自己避免。
- 原版依据：码头 kit 记忆底 tile（[dock_kit](../../../raw/current/scripts/prefabs/dock_kit.lua#L112-L114)）；wurt 沼泽造地机读旧值再回填（[wurt_swamp_terraformer](../../../raw/current/scripts/prefabs/wurt_swamp_terraformer.lua#L88-L96)）。

```lua
-- 临时地皮实体落位（服务端）
local tx, ty = TheWorld.Map:GetTileCoordsAtPoint(x, y)
TheWorld.components.undertile:SetTileUnderneath(tx, ty, original_tile_id)

-- 回收时
TheWorld.components.undertile:ClearTileUnderneath(tx, ty)
```

- 调整与验证：x/y 是 tile 坐标而非世界坐标，需先经 `TheWorld.Map:GetTileCoordsAtPoint` 转换；tilemanager 的放置校验注释明确以本组件判断"当前格是临时层"（[tilemanager](../../../raw/current/scripts/tilemanager.lua#L189)）。

## 常见陷阱与限制

- 网格只在 `worldmapsetsize` 事件后可用（[监听](../../../raw/current/scripts/components/undertile.lua#L35)）：世界载入极早期调用 Get/Set 会因 `_underneath_tiles` 为 nil 报错，postinit 写入请包 DoTaskInTime 延迟。
- OnLoad 依赖 `TheWorld.tile_id_conversion_map` 存在；换档升级时旧 id 未在转换表内的值保持原样（`map[v] or v`，[实现](../../../raw/current/scripts/components/undertile.lua#L72-L74)），自造 tile id 需自行进表。

## 相关链接

- [[reference/components/terraformer]]：以 undertile 为兜底的造地 tile 选择。
- [[reference/components/autoterraformer]]：读 undertile 判断可造点的自动造地。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖全部接口与 world／dock_kit／terraformer／ropebridgemanager 等消费链（全库检索另见 dockmanager、oceanicemanager、vinebridgemanager、wobycourier、rift_terraformer、retrofit 两件）；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
