---
title: "groundcreep"
created: "2026-10-28"
updated: "2026-10-29"
type: component
summary: "TheWorld 上的地面蠕变存取垫片：仅负责玩家激活快进与存档序列化，蠕变数据与查询本体在引擎实体 inst.GroundCreep 上。"
aliases: ["地面蠕变", "groundcreep", "地表蠕变层"]
tags: [component, data]
dst_build: "steam-buildid:26230496"
sources: ["components/groundcreep.lua", "prefabs/world.lua", "components/locomotor.lua", "prefabs/player_classified.lua", "components/birdspawner.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# groundcreep

## 快速判断

- 职责与适用：挂在 TheWorld 上；组件自身没有业务接口，只为引擎蠕变对象做两件事——玩家激活时快进、存档读写（[组件全文](../../../raw/current/scripts/components/groundcreep.lua#L1-L48)）。文件头注释自述"只为让地面蠕变的序列化合理化"。
- 优先使用：查询/操作蠕变数据直接走引擎对象 `inst.GroundCreep`（由 `inst.entity:AddGroundCreep()` 创建，[实体](../../../raw/current/scripts/prefabs/world.lua#L451)、[组件挂载](../../../raw/current/scripts/prefabs/world.lua#L536)），如移动判定的 `OnCreep` 与 `GetTriggeredCreepSpawners`（[locomotor](../../../raw/current/scripts/components/locomotor.lua#L606-L611)）。
- 重要副作用：监听 `playeractivated` 调 `GroundCreep:FastForward()`（[快进](../../../raw/current/scripts/components/groundcreep.lua#L22-L30)）；玩家同步实体 `player_classified` 的 OnPlayerFadeDirty 在有 HUD、淡入时间大于 0 且处于淡入分支时也直接调用 FastForward（[淡入回调](../../../raw/current/scripts/prefabs/player_classified.lua#L1018-L1026)）。
- 修改边界：`OnSave`/`OnLoad` 仅在 mastersim 下定义（条件式方法定义，[存读](../../../raw/current/scripts/components/groundcreep.lua#L36-L42)）；蠕变渲染层在 world 构造时经 MapLayerManager 建立后 `GroundCreep:AddRenderLayer`（[渲染层](../../../raw/current/scripts/prefabs/world.lua#L508)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `OnSave()` | `mastersim 限 -> string` | 返回 `inst.GroundCreep:GetAsString()`；返回值是字符串而非表。[实现](../../../raw/current/scripts/components/groundcreep.lua#L36-L38) |
| `OnLoad(data)` | `mastersim 限；data: string` | `inst.GroundCreep:SetFromString(data)` 整体恢复蠕变。[实现](../../../raw/current/scripts/components/groundcreep.lua#L40-L42) |

## 典型接入

### 读取蠕变状态（原版参照）

- 前提与选择：组件无可调接口；需要蠕变信息的代码都直接访问 `TheWorld.GroundCreep` 引擎对象。原版消费者：locomotor 按蠕变减速并触发蠕变刷怪（[locomotor](../../../raw/current/scripts/components/locomotor.lua#L606-L611)）、鸟落点避开蠕变（[birdspawner](../../../raw/current/scripts/components/birdspawner.lua#L68)）、地皮定义判定（worldtiledefs.lua:146）。

```lua
local oncreep = TheWorld.GroundCreep ~= nil and TheWorld.GroundCreep:OnCreep(x, y, z) or false
```

- 调整与验证：`OnCreep(x, y, z)` 为引擎侧方法，Lua 侧无定义可查，签名以原版调用处为准；本页不覆盖引擎内部实现。

## 常见陷阱与限制

- **别从组件找 API**：组件除存读钩子外无方法；扩展蠕变行为只能操作引擎对象或监听 world 事件，给组件加字段不会被序列化。
- **OnSave 返回字符串**：存档钩子把蠕变整体序列化为一个字符串（[实现](../../../raw/current/scripts/components/groundcreep.lua#L37)），想拆分或转换需在引擎数据层处理，Lua 侧拿到的是黑盒字符串。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文（含条件式 OnSave/OnLoad）、world.lua 的实体创建与组件挂载、locomotor/birdspawner/player_classified/worldtiledefs 四处引擎对象消费。
- 术语补核：2026-10-29，同一 build，定向复核 player_classified 的 OnPlayerFadeDirty 淡入分支，替换“分类实体”直译并明确调用条件；未重审其余机制。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
