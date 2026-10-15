---
title: 网络与分片组件索引
created: 2026-10-28
updated: 2026-10-29
type: index
tags: [overview]
---

# 网络与分片

跨分片同步、世界会话、迁移与服务器管理类组件；netvar 基础协议归 API，本主题承载组件级封装。只读命中分组；适用版本与验证范围见具体页，已知名称可直读正文。

## 跨分片同步

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/shard_worldreset]] | 跨分片重置倒计时桥：net_byte 转 secondary_worldresetupdate | 跨服重置同步 / active |
| [[reference/components/shard_wagbossinfo]] | Wagstaff 击败状态跨分片 net_bool 广播 | 跨服 Wagstaff 信息 / active |
| [[reference/components/shard_autosaver]] | 主机分片快照号跨分片广播，转 secondary_autosaverupdate | 跨服自动存档 / active |
| [[reference/components/shardtransactionsteps]] | 跨分片三步事务（INITIATE/ACCEPTED/FINALIZED）带重放防护 | 跨分片事务 / active |
| [[reference/components/shard_mermkingwatcher]] | 鱼人王与三贡品跨分片 net 桥，主分片聚合转推 *_anywhere | 鱼人王跨分片同步 / active |
| [[reference/components/shardstate]] | world_net 上同步主分片会话 ID 供客户端缓存 | 主分片会话 ID / active |
| [[reference/components/shard_seasons]] | 跨分片季节 net 桥：主分片写入、次分片转推 secondary_seasonsupdate | 跨分片季节同步 / active |
| [[reference/components/shard_clock]] | 主→次分片昼夜时间网络变量同步桥，无公开方法 | 分片时钟同步 / active |
| [[reference/components/shard_players]] | 跨分片玩家/幽灵计数广播与 ms_playercounts 聚合 | 跨分片人数 / active |
| [[reference/components/shard_worldvoter]] | 跨分片投票状态镜像（64 槽网络变量转播） | 跨分片投票同步 / active |


## 世界会话、出生与存档

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/worldreset]] | 世界重置倒计时：全员皆鬼触发，net_byte 同步，到点删档重开 | 世界重置 / active |
| [[reference/components/playerspawner]] | 玩家出生调度：fixed/scatter 选点、读档落地、出生保护、迁移 | 玩家出生点 / active |
| [[reference/components/worldcharacterselectlobby]] | 熔炉/活动场出生大厅网络倒计时与就绪汇集 | 角色选择大厅 / active |
| [[reference/components/autosaver]] | 世界网络自动存档调度，主机写档/次分片快照同步回滚 | 自动存档 / active |

## 投票与管理

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/worldvoter]] | 用户指令投票会话：倒计时同步计票禁言与 UI 事件 | 投票踢人 / active |
| [[reference/components/playervoter]] | 玩家投票端组件：桥接 classified 票选/禁言变量，SubmitVote＋乐观刷新 | 玩家投票 / active |

返回 [[reference/components/index|组件主题导航]]。
