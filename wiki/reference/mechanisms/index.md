---
title: 内部机制索引
created: 2026-10-24
updated: 2026-10-30
type: index
tags: [overview]
---

# 内部机制

跨组件内部流程、行为与数据消费链及版本迁移影响面；单组件自身接口见 [[reference/components/index|组件]]，模组接入约定见 [[reference/api/index|API]]。

| 页面入口 | 需求摘要 | 必要别名 / 状态 |
|---|---|---|
| [[reference/mechanisms/vault-virtualroom-migration]] | build 26148036 金库（vault）判定重构为通用虚拟房间体系的影响面：map 新查询入口、暗影手/指南针/守卫消费者接入、vault_teleport 泛化、存档 5.25 自动迁移与模组迁移判断 | 金库虚拟房间迁移、vaultroommanager 替换 / active |
| [[reference/mechanisms/rain-fx-gating]] | build 26148036 雨 FX（rain／caverain）粒子与溅落逐点无雨区门控：三处判定链、weather／caveweather 强度驱动与雨棚补偿滴 | 无雨区门控、雨滴FX门控 / active |
| [[reference/mechanisms/spike-launcher]] | 四族地面尖刺（化石×2／沙晶／冰刺）共用 SpikeLaunch 发射模式与 build 26148036 发射传送回退：TryTeleportToLaunchPos 语义、SetLanded 空中标记、与 lootdropper 回退对照 | 尖刺发射、发射传送回退 / active |
| [[reference/mechanisms/generic-fx-factory]] | fx.lua 通用注册表 504 特效的公共工厂协议：网络代理与本地表现分工、父挂与一次性定位、音效与暂停时钟、fn／fntime 回调及三类回收 | 通用FX工厂、fx.lua 通用工厂、MakeFx、fx_spawned / active |

返回 [[reference/index|机制与接口参考]]。
