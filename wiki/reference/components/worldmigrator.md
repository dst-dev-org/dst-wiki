---
title: "worldmigrator"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "跨分片传送门户组件：管理门户 id／目标分片绑定与三态状态，玩家或物品激活后委派 ms_playerdespawnandmigrate 或分片物品转移事务。"
aliases: ["跨分片传送门", "迁移门户", "worldmigrator"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/worldmigrator.lua", "shardnetworking.lua", "prefabs/oceanwhirlbigportal.lua", "componentutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# worldmigrator

## 快速判断

- 职责与适用：挂在洞穴入口、地穴出口、迁移门、海洋漩涡门户等门户实体上（[例](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L347-L350)），维护"绑定到哪个分片哪个门户"与可用／禁用／满员三态，并提供玩家／物品的迁移入口。
- 优先使用：改状态用 `ValidateAndPushEvents` 体系（`SetDestinationWorld`／`SetEnabled`／`SetDisabledWithReason`）；执行迁移只调 `Activate(doer)`（[L198-L233](../../../raw/current/scripts/components/worldmigrator.lua#L198-L233)）。
- 重要副作用：状态变化在实体上推 `migration_available`／`migration_full`／`migration_unavailable` 事件（[L104-L128](../../../raw/current/scripts/components/worldmigrator.lua#L104-L128)）；可用且未隐藏时加 `migrator` 标签供动作候选。
- 修改边界：绑定与状态由 shardnetworking 在分片连接／断开时统一改写（[L102-L123](../../../raw/current/scripts/shardnetworking.lua#L102-L123)）；组件不处理玩家落到目标世界的落点，落点由 playerspawner 按 portalid 解析（[消费](../../../raw/current/scripts/componentutil.lua#L1100-L1108)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetDestinationWorld(world, permanent)` | world 为分片 id | 绑定目标分片；`permanent=true` 时 `auto=false`，之后 shardnetworking 不再自动改绑（[L69-L76](../../../raw/current/scripts/components/worldmigrator.lua#L69-L76)、[改绑逻辑](../../../raw/current/scripts/shardnetworking.lua#L102-L108)）。 |
| `SetID(id)` / `IsDestinationForPortal(world, portal)` | id 数字或字符串 | id 即 portalid，`SetID` 同时把 `receivedPortal` 临时置为同值（注释 TEMP HACK）；配对判定两字段都相等（[L134-L147](../../../raw/current/scripts/components/worldmigrator.lua#L134-L147)）。 |
| `SetDisabledWithReason(reason)` / `ClearDisabledWithReason(reason)` | reason 字符串 | 多来源禁用修饰（SourceModifierList）；`"MISSINGSHARD"` 由 shardnetworking 在分片失联时使用（[L78-L84](../../../raw/current/scripts/components/worldmigrator.lua#L78-L84)）。 |
| `SetHideActions(hidden)` | bool | 隐藏时不加 `migrator` 标签（动作候选消失），状态事件照常；海洋漩涡大门户用它关掉右键动作（[L60-L67](../../../raw/current/scripts/components/worldmigrator.lua#L60-L67)）。 |
| `Activate(doer)` | → `bool, reason` | doer 为玩家：推 `ms_playerdespawnandmigrate`（playerspawner 消费，[监听](../../../raw/current/scripts/components/playerspawner.lua#L213)）；doer 为物品：走 `Shard_CreateTransaction_TransferInventoryItem` 转移（[L198-L233](../../../raw/current/scripts/components/worldmigrator.lua#L198-L233)）。 |
| `shard_name`（约定字段） | "Master"／"Caves" | 组件不声明，prefab 直接赋值；shardnetworking 按它过滤哪些门户跟该分片绑定（[赋值例](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L348)、[过滤](../../../raw/current/scripts/shardnetworking.lua#L104)）。 |
| `OnSave` / `OnLoad` | id／linkedWorld／receivedPortal／auto | 兼容旧拼写 `recievedPortal`（[L239-L258](../../../raw/current/scripts/components/worldmigrator.lua#L239-L258)）。 |

## 状态与绑定（关键流程）

1. 初始化：构造后 `DoTaskInTime(0)` 内若 `id == nil` 则取自增序号，否则查重；随后在 TheWorld 推 `ms_registermigrationportal` 并调 `Shard_UpdatePortalState`（[L22-L36](../../../raw/current/scripts/components/worldmigrator.lua#L22-L36)；[注册](../../../raw/current/scripts/shardnetworking.lua#L125-L140)）。
2. 自动绑定：`auto=true` 且未绑定目标时，shardnetworking 在分片连接时按 `shard_name` 匹配绑定可用分片，并清 `MISSINGSHARD`；不匹配或 `auto=false` 的门户被置 `MISSINGSHARD` 禁用（[L102-L123](../../../raw/current/scripts/shardnetworking.lua#L102-L123)）。
3. 三态推进：`enabled && 无禁用修饰` 才可能 ACTIVE；目标分片可用为 ACTIVE、满员为 FULL、否则 INACTIVE，每次转移推对应事件（[L104-L128](../../../raw/current/scripts/components/worldmigrator.lua#L104-L128)）。
4. 迁移执行：玩家激活推 `migration_activate` 后经 `ms_playerdespawnandmigrate` 带上 `portalid`／`worldid`／`fxoverride`（漩涡门户专属 FX 覆盖表）；物品激活先丢弃不可迁移物再建分片转移事务（[L198-L233](../../../raw/current/scripts/components/worldmigrator.lua#L198-L233)）。

## 常见陷阱与限制

- **可携带性规则**：`irreplaceable` 标签物品不迁移；有 [[reference/components/migrationpetowner]] 登记宠物的物品不迁移，激活时还会对牛铃类物品先调 `OnStopUsing` 解绑（[L165-L183](../../../raw/current/scripts/components/worldmigrator.lua#L165-L183)）。
- **双向绑定靠约定**：`SetReceivedPortal` 注释自述"需要双向协商"；当前实现里 `SetID` 把 `receivedPortal` 置为同值，两端门户必须使用相同 id（如漩涡门户两端都用字符串 `"oceanwhirlbigportal"`，[L347-L350](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L347-L350)、[对端](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L556-L559)）。
- **`auto=false` 要显式存**：`SetDestinationWorld(world, true)` 设置后 OnSave 保存 `auto`；但 OnLoad 里先置 `auto=true` 再读档覆盖，遗漏字段会回到自动改绑语义（[L248-L258](../../../raw/current/scripts/components/worldmigrator.lua#L248-L258)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、shardnetworking 绑定／注册／状态段、componentutil 落点解析、oceanwhirlbigportal 两端装配；未覆盖 SHARD_RPC 传输层与洞穴入口开口动画细节。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/migrationpetowner]]、[[reference/components/migrationpetsoverrider]]。
