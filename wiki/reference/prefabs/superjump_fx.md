---
title: "超级跳落点特效（superjump_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "熔炉武器超级跳（combat_superjump 三态跳劈）落点的表现层：玩家剪影落地动画＋落点碎屑＋破地音效，同文件注册 superjump_debris；脚本树内无生成调用，SpawnPrefab 在 lavaarena 事件数据包。"
aliases: ["superjump_fx", "superjump_debris", "超级跳", "熔炉长矛超级跳", "player_superjump", "superjump"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/superjump_fx.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "components/aoeweapon_leap.lua", "prefabs/spear_lance.lua", "prefabs/lavaarena_heavyblade.lua", "prefabs/player_common.lua", "prefabs/player_common_extensions.lua", "entityscript.lua", "prefablist.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 超级跳落点特效（superjump_fx）

## 快速判断

- 职责与适用：熔炉武器"超级跳"（玩家 SGwilson `combat_superjump` 三态跳劈）**落点的表现层 FX**：两个注册名——`superjump_fx`（bank `wilson`／build `player_superjump` 播 `superjump_land_fx` 剪影落地动画＋破地音效）与 `superjump_debris`（落点碎屑 `superjump_land_debris`）；联网 FX 壳＋主模拟 `SetTarget` 定位，无任何玩法组件。[工厂](../../../raw/current/scripts/prefabs/superjump_fx.lua#L39-L73)、[碎屑](../../../raw/current/scripts/prefabs/superjump_fx.lua#L75-L106)
- 优先使用：查"超级跳落点动画／碎屑"选本页；跳劈的输入路由与伤害结算在 SGwilson 状态与 aoeweapon_leap 组件，见关联机制导航。
- 重要副作用：`SetTarget` 按目标实体位置／朝向摆放自身；只有 debris 显式继承目标缩放（主 FX 未设缩放）；生成后 0 帧内未定位则先在生成点播 `ground_break` 声，定位调用会取消该任务并就地重播（生成点不再响声）。[SetTarget](../../../raw/current/scripts/prefabs/superjump_fx.lua#L16-L37)、[音效](../../../raw/current/scripts/prefabs/superjump_fx.lua#L11-L14)
- 修改边界：**脚本树内没有 `SpawnPrefab("superjump_fx")`**——熔炉武器的 master_postinit 走 `event_server_data("lavaarena", ...)`，实际生成与 `SetTarget` 调用未在当前脚本树定位，事件数据包本快照不含；in-tree 只有依赖声明与玩家状态机。[事件数据加载](../../../raw/current/scripts/entityscript.lua#L77-L87)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `superjump_fx` | fn：Transform＋AnimState＋SoundEmitter＋Network → SetFourFaced → bank `wilson`／build `player_superjump` 播 `superjump_land_fx`（单参，未显式传循环参数）→ bloom＋FinalOffset 2＋AddColour(.7,.5,0) → FX/NOCLICK → SetPristine → 主模拟 animover Remove＋persists=false＋暴露 SetTarget＋0 帧任务播破地声 | [工厂](../../../raw/current/scripts/prefabs/superjump_fx.lua#L39-L73)、[注册](../../../raw/current/scripts/prefabs/superjump_fx.lua#L108-L109) |
| `superjump_debris` | debrisfn：同初始化，anim `superjump_land_debris`、FinalOffset 1；主模拟 animover Remove＋persists=false | 仅由 superjump_fx 的 SetTarget 生成，按目标实体缩放。[碎屑工厂](../../../raw/current/scripts/prefabs/superjump_fx.lua#L75-L106) |

两注册名均无 NAMES 条目；标题中文为已核验效果描述，非官方译名。prefablist 的 "superjump_fx"（[L1249](../../../raw/current/scripts/prefablist.lua#L1249)）是脚本文件名条目；`superjump_debris` 不在其列。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 消费者：SGwilson combat_superjump 三态 | 持 `superjump`＋`aoeweapon_leap` 双标签武器的动作进 `combat_superjump_start`（播 `superjump_pre`，超时续播 `superjump_lag` 后 PerformBufferedAction）→ 收 `combat_superjump` 事件进主跳态（1 秒渐隐、无敌、`dropallaggro`、武器收 `superjumpstarted`）→ `combat_superjump_pst` 传送到目标点、4 帧相机震动并调 `aoeweapon_leap:DoLeap` 结算 | 三态全在服务端 SG；客户端预测态只预演 `combat_superjump_start` | [派发](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1325)、[start 态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L17732-L17778)、[主跳态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L17780-L17850)、[落点态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L17890-L17994)、[客户端预测](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L4741-L4768) |
| 伤害结算：aoeweapon_leap | 落点半径 `aoeradius` 4（另加目标物理半径）逐目标 OnHit，低空库存物品按距离投掷；结算是武器组件职责，与本 FX 无关 | `onpreleapfn`／`onleaptfn` 钩子由事件数据安装 | [[reference/components/aoeweapon-leap]]；[DoLeap](../../../raw/current/scripts/components/aoeweapon_leap.lua#L30-L94) |
| 生成者：事件数据（未核验） | 武器 master_postinit 与 `superjumpstarted`／`superjumpcancelled` 事件监听都在 `lavaarena_event_server` 包内，快照不可读；"何时 SpawnPrefab(superjump_fx) 并调 SetTarget"无法从脚本树确认 | `event_server_data("lavaarena", path)` → requireeventfile 加载事件包 | [spear_lance 接线](../../../raw/current/scripts/prefabs/spear_lance.lua#L76)、[heavyblade 接线](../../../raw/current/scripts/prefabs/lavaarena_heavyblade.lua#L89)、[加载实现](../../../raw/current/scripts/entityscript.lua#L77-L87) |
| in-tree 武器 | spear_lance（熔炉长矛）是脚本树内唯一带 `superjump` 标签的武器：aoetargeting 射程 16，依赖声明 superjump_fx | heavyblade（熔炉重剑）依赖 superjump_fx 但 pristine 无 `superjump` 标签，in-tree 路由走 `combat_leap_start`；事件数据是否补加标签未核验 | [spear_lance 依赖](../../../raw/current/scripts/prefabs/spear_lance.lua#L15)、[superjump 标签](../../../raw/current/scripts/prefabs/spear_lance.lua#L48)、[heavyblade 依赖](../../../raw/current/scripts/prefabs/lavaarena_heavyblade.lua#L11) |
| 资源 | anim/player_superjump.zip 同时供本 FX 与玩家／精英猪人跳跃动画（override build）使用；player_common 依赖本注册名 | [声明](../../../raw/current/scripts/prefabs/superjump_fx.lua#L1-L4)、[玩家 override](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L1323)、[依赖](../../../raw/current/scripts/prefabs/player_common.lua#L2217) |

## 典型接入

### 复用"落点剪影＋碎屑"表现

- 前提与选择：主模拟生成 `superjump_fx` 后调用 `fx:SetTarget(target实体, distance可选)`；SetTarget 读取目标的 Transform 位置／朝向／缩放，`distance` 传值时沿朝向前移。
- 生效与撤销：SetTarget 内即刻生成 `superjump_debris` 于同一位置；若 0 帧任务未跑过则取消并就地播破地声。动画播完 animover 自删，无需清理。
- 原版依据：in-tree 无调用方（生成在事件数据包），以上按 SetTarget 实现与 debris 生成路径核验，实际调用时机未核验。

```lua
local fx = SpawnPrefab("superjump_fx")
fx.Transform:SetPosition(x, y, z)
fx:SetTarget(target_ent)  -- 摆到目标位置/朝向并生成同朝向碎屑
```

- 调整与验证：目标需有 Transform；若延迟超过一帧才 SetTarget，破地声已在生成点播过（inittask 已清空），不会在目标点补响。[SetTarget](../../../raw/current/scripts/prefabs/superjump_fx.lua#L16-L37)

## 常见陷阱与限制

- **边界：脚本树内查不到生成调用**：当前脚本树未定位到 `SpawnPrefab("superjump_fx")` 与 `SetTarget` 的实际调用；接线依赖的事件数据包不在快照内，具体调用方保持未核验。
- **误区：月台跳／普通跳跃也走本 FX**：in-tree 动作路由只认 `superjump`＋`aoeweapon_leap` 双标签武器的动作；heavyblade 有依赖无标签，走 `combat_leap_start`。[派发](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1325)
- **边界："combat_superjump" 事件脚本树内无人 Push**：SG 只收不发，推送方在事件数据侧（如武器监听 `superjumpstarted` 后的逻辑）；不要在脚本树里找推送点。
- **边界：动画是否重复未核验**：`PlayAnimation("superjump_land_fx")` 单参播放；设计预期是播完 animover 自删，别依赖循环。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；110 行工厂全文（两工厂、SetTarget、debris）、SGwilson 三态与动作派发、客户端预测态、aoeweapon_leap.DoLeap、spear_lance／heavyblade 接线与标签、player_common 依赖、event_server_data 加载机制、NAMES 检索（无命中）已核验；事件数据包内容（实际 SpawnPrefab 与 SetTarget 调用点、参数）、heavyblade 事件数据是否补 `superjump` 标签、晚加入时序未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-transition-index|变身、召唤、传送与退场]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
