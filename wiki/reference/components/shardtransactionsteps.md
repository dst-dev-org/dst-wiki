---
title: "shardtransactionsteps"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "跨分片可靠事务：按 INITIATE/ACCEPTED/FINALIZED 三步确认并带重放防护，原版用于迁移物品跨分片搬运与存档一致。"
aliases: ["跨分片事务", "shardtransactionsteps", "分片数据传输"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/shardtransactionsteps.lua", "prefabs/world.lua", "shardnetworking.lua", "networkclientrpc.lua", "components/worldmigrator.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# shardtransactionsteps

## 快速判断

- 职责与适用：mastersim 专用的 TheWorld 组件（[断言](../../../raw/current/scripts/components/shardtransactionsteps.lua#L14)、[挂载](../../../raw/current/scripts/prefabs/world.lua#L574)），把"跨分片改数据"包成可确认、可重试、可去重的事务；事务 id 按 `shardid` 命名空间自增（[注释](../../../raw/current/scripts/components/shardtransactionsteps.lua#L1-L11)）。
- 优先使用：传物品直接用 `shardnetworking.lua` 的包装 `Shard_CreateTransaction_TransferInventoryItem`（[包装](../../../raw/current/scripts/shardnetworking.lua#L294-L303)）；新事务类型才在组件内扩展。
- 重要副作用：载荷经 `DataDumper` 序列化走 `SHARD_RPC.ShardTransactionSteps`，接收端在安全沙箱还原执行（[RPC](../../../raw/current/scripts/networkclientrpc.lua#L1541-L1556)）。
- 修改边界：`CreateTransaction` 断言目标分片不能是自己（[断言](../../../raw/current/scripts/components/shardtransactionsteps.lua#L190)）；目标世界不可用时事务停在 INITIATE，等 `OnShardConnected` 重发（[重连](../../../raw/current/scripts/components/shardtransactionsteps.lua#L111-L124)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `CreateTransaction(shardid, transactiontype, data)` | shardid: string | 生成 uniqueid、登记载荷、执行 `HandleTransactionInitialization` 后置 INITIATE 发送（[实现](../../../raw/current/scripts/components/shardtransactionsteps.lua#L188-L212)）。 |
| `OnShardTransactionSteps(shardpayload)` | shardpayload: table | 双向状态机：发送方转发/收尾，接收方落载、回 ACCEPTED 并 FINALIZE；完成按序推进 `finalizedid` 并发 Prune（[实现](../../../raw/current/scripts/components/shardtransactionsteps.lua#L25-L98)）。 |
| `HandleTransactionFinalization(shardpayload)` | → `bool` | 按迁移数据找传送门，交给 `itemstore` 或直接 `SpawnPrefab`+`SetPersistData` 落地；失败可重排（[实现](../../../raw/current/scripts/components/shardtransactionsteps.lua#L138-L165)）。 |
| `OnShardConnected(shardid)` | shardid: string | 重连后重发本分片发起的 INITIATE 事务（[实现](../../../raw/current/scripts/components/shardtransactionsteps.lua#L111-L124)）。 |
| `OnPruneShardTransactionSteps(shardid, newfinalizedid)` | 调试/RPC 入口 | 清理已确认区间（[实现](../../../raw/current/scripts/components/shardtransactionsteps.lua#L100-L109)）。 |
| `OnSave()` / `OnLoad(data)` | 组件存档钩子 | 整个 transactions 表随世界存档（[实现](../../../raw/current/scripts/components/shardtransactionsteps.lua#L214-L230)）。 |

## 典型接入

### 跨分片转移物品

- 前提与选择：多分片世界、mastersim；物品先在本地序列化移除，接收端按迁移数据放回传送门（原版只有 `TRANSFERINVENTORYITEM` 一种类型，[类型处理](../../../raw/current/scripts/components/shardtransactionsteps.lua#L126-L136)）。
- 生效与撤销：`worldmigrator` 在玩家迁移时调用包装函数把随身物品转为事务（[调用](../../../raw/current/scripts/components/worldmigrator.lua#L227)）；接收端应用失败时 `DoTaskInTime(0)` 重排（[重排](../../../raw/current/scripts/components/shardtransactionsteps.lua#L51-L57)）。
- 原版依据：完成链路见上文 RPC 处理器；重放防护靠 `finalizedid < uniqueid` 判定，重复收到直接回 ACCEPTED 让发送方收尾（[重放](../../../raw/current/scripts/components/shardtransactionsteps.lua#L67-L71)）。

```lua
-- 服务端 TheWorld 上（mastersim），示意自定义事务登记（原版仅内置物品类型）
TheWorld.components.shardtransactionsteps:CreateTransaction(shardid, SHARDTRANSACTIONTYPES.TRANSFERINVENTORYITEM,
    { item = item, migrationdata = migrationdata })
```

- 调整与验证：自定义 transactiontype 需在 `HandleTransactionInitialization`/`HandleTransactionFinalization` 各加分支并保持 `ClearFields` 同步（[ClearFields](../../../raw/current/scripts/components/shardtransactionsteps.lua#L179-L186)），否则字段泄漏到存档。

## 常见陷阱与限制

- 事务状态含调试序号比较（`status >= SHARDTRANSACTIONSTEPS.FINALIZED`），自定义状态值必须按枚举序追加（[比较](../../../raw/current/scripts/components/shardtransactionsteps.lua#L82)）。
- uniqueid 只在接收端单调推进：发送端重新载入存档后 id 从存档 transactions 恢复，不要手工重置（[OnLoad](../../../raw/current/scripts/components/shardtransactionsteps.lua#L224-L230)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文与 world 挂载、shardnetworking 包装、worldmigrator 调用、networkclientrpc RPC 处理器；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
