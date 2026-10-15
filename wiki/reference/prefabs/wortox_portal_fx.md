---
title: "灵魂跳跃出入场特效（wortox_portal_jumpin_fx／wortox_portal_jumpout_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "沃拓克斯灵魂跳跃（Soul Hop）在起跳点与落点生成的一次性门形动画：bank wilson／build wortox_portal，入场与出场两注册，跳跃位移与无敌由宿主状态图承担。"
aliases: ["wortox_portal_fx", "wortox_portal_jumpin_fx", "wortox_portal_jumpout_fx", "灵魂跳跃特效", "灵魂回声", "Soul Hop", "soulhop"]
tags: [prefab, fx, network-sync]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/wortox_portal_fx.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "actions.lua", "prefabs/wortox.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 灵魂跳跃出入场特效（wortox_portal_jumpin_fx／wortox_portal_jumpout_fx）

## 快速判断

- 职责与适用：沃拓克斯（官方译名"沃拓克斯"，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L351850-L351852)）灵魂跳跃动作链在**起跳点**与**落点**各生成一个一次性动画：bank `wilson`／build `wortox_portal`，FinalOffset 3；无组件、无玩法作用，位移、无敌与消耗都由宿主状态承担。[工厂](../../../raw/current/scripts/prefabs/wortox_portal_fx.lua#L10-L21)
- 优先使用：查"灵魂跳跃／Soul Hop 特效"选本页；动作链入口是灵魂物品驱动的 `ACTIONS.BLINK`（无物品、`soulstealer` 标签分支），见下文导航表。
- 重要副作用：FX 自身无副作用；生成它的宿主状态在跳跃中段给玩家无敌、关 DynamicShadow、加 `noattack`。[状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21018-L21030)
- 修改边界： allegiance 技能染色由状态图在生成后对 FX 实例设置，不在工厂内；动画播完后延迟 2 帧自删。[退场](../../../raw/current/scripts/prefabs/wortox_portal_fx.lua#L6-L8)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wortox_portal_jumpin_fx` | MakeFX(name, anim)：Transform＋AnimState＋Network → 播传入动画 → SetPristine → 客户端早退 → 主模拟挂 `animover → DoTaskInTime(2*FRAMES, Remove)`、`persists=false` | anim `wortox_portal_jumpin`；未显式传循环参数。[注册](../../../raw/current/scripts/prefabs/wortox_portal_fx.lua#L38-L39) |
| `wortox_portal_jumpout_fx` | 同一 MakeFX 工厂，仅动画不同 | anim `wortox_portal_jumpout`。[注册](../../../raw/current/scripts/prefabs/wortox_portal_fx.lua#L38-L39) |

清单里的 `wortox_portal_fx` 是族名（沿用文件名），**不是注册名**；实际注册只有上表两个。标题中文采用动作 `ACTIONS.BLINK.SOUL` 官方译名"灵魂跳跃"（[译名](../../../raw/current/scripts/languages/chinese_s.po#L290-L292)；地图分支 `BLINK_MAP.SOUL`"灵魂跳跃({souls})" [译名](../../../raw/current/scripts/languages/chinese_s.po#L295-L297)），FX 注册名本身无 NAMES 条目。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 动作链：ACTIONS.BLINK | 无 `invobject` 且 doer 有 `soulstealer` 标签时走 soulhop：`TryToSoulhop` 校验后 `GoToState("portal_jumpin")` | 有物品时是传送道具（blinkstaff），不生成本 FX | [路由](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1345-L1352)、[BLINK.fn](../../../raw/current/scripts/actions.lua#L3574-L3595) |
| 地图跳跃：ACTIONS.BLINK_MAP | 无物品分支经 `TryToSoulhop(…, true)` 传 consumeall=true 强制结算当前跳跃账单，按 distancecount／累计 _soulhop_cost 扣除，并非清空背包所有灵魂，`from_map=true` | 理论无限距离，仅 soulhop | [BLINK_MAP.fn](../../../raw/current/scripts/actions.lua#L3608-L3620) |
| 宿主状态：portal_jumpin | 在玩家原地生成 jumpin fx；11 帧超时转 portal_jumpout；4 帧起无敌＋关阴影；skill `wortox_souldecoy_1` 首跳生成诱饵 | 主模拟状态 | [生成与染色](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20973-L20985)、[无敌](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21018-L21030)、[超时转场](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21033-L21036) |
| 宿主状态：portal_jumpout | 传送后在落点生成 jumpout fx；14 帧超时回 idle；onenter 推 `soulhop` 事件 | 传送、物理开关都由状态完成 | [落点生成](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21072-L21084)、[无敌与事件](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21085-L21089) |
| 消耗结算：wortox.lua | `CanSoulhop` 要求背包有灵魂且不骑乘；`TryToPortalHop` 记账并在跳完经 `FinishPortalHop` 扣灵魂；`soulhop` 事件接 `OnSoulHop` 检查动画后结算 | 消耗与特效解耦；失败时灵魂返还由 `CheckSoulsRemovedAfterAnim` 处理 | [CanSoulhop](../../../raw/current/scripts/prefabs/wortox.lua#L512-L520)、[TryToPortalHop](../../../raw/current/scripts/prefabs/wortox.lua#L614-L651)、[监听](../../../raw/current/scripts/prefabs/wortox.lua#L789)、[动画后结算](../../../raw/current/scripts/prefabs/wortox.lua#L434-L440) |
| allegiance 染色 | 状态图按 `wortox_allegiance_shadow`／`wortox_allegiance_lunar` 对 FX 设 MultColour 0.6 或 AddColour 0.1 | 仅技能激活时；玩家本体另有 DoWortoxPortalTint 渐变 | [jumpin](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20977-L20985)、[jumpout](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21076-L21084)、[常量](../../../raw/current/scripts/stategraphs/SGwilson.lua#L18-L19) |
| 声音 | FX 无 SoundEmitter；起跳音（infection_post＋soul/spawn）、落点音（soul/hop_out、bodyfall）由状态时间线播 | 主模拟 3D 音效联网复制 | [jumpin 声音](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21010-L21017)、[jumpout 声音](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21101-L21110) |
| 客户端预测 | `SGwilson_client` 有同名 `portal_jumpin_pre` 预测状态（server_states 对齐），fx 生成仍只在主模拟 | 客户端只播预备动画 | [预测状态](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L5932-L5941) |
| 动画资源 | bank `wilson`＋build `wortox_portal` 两个 portal 动画 | 资源包内容未解包 | [设置](../../../raw/current/scripts/prefabs/wortox_portal_fx.lua#L18-L21) |

## 典型接入

### 复用为任意传送的出入场特效

- 前提与选择：主模拟侧脚本生成后 `SetPosition` 即可；FX 自带 animover 延迟删除，无需手动清理。
- 生效与撤销：染色彩参考状态图做法，生成后直接对实例设 AnimState。

```lua
local fx = SpawnPrefab("wortox_portal_jumpout_fx")
fx.Transform:SetPosition(x, y, z)
fx.AnimState:SetMultColour(0.6, 0.6, 0.6, 1) -- 模拟 shadow allegiance
```

- 调整与验证：`SetMultColour`／`SetAddColour` 的数值取自 allegiance 分支；若把 FX 挂给非沃拓克斯实体，宿主消耗／无敌逻辑都不会自动跟随。[染色依据](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21076-L21084)

## 常见陷阱与限制

- **误区：`wortox_portal_fx` 是注册名**：实际注册 `wortox_portal_jumpin_fx`／`wortox_portal_jumpout_fx`，`SpawnPrefab("wortox_portal_fx")` 无对应注册。[注册](../../../raw/current/scripts/prefabs/wortox_portal_fx.lua#L38-L39)
- **误区：FX 负责跳跃位移或无敌**：传送、物理开关、无敌、阴影开关全在宿主状态；FX 只在两个位置播动画。[portal_jumpout](../../../raw/current/scripts/stategraphs/SGwilson.lua#L21072-L21089)
- **边界：骑乘不能 soulhop**：`CanSoulhop` 在骑乘时返回 false，因此不存在骑乘变体的生成路径；fx 也无 MakeMounted 类接口。[条件](../../../raw/current/scripts/prefabs/wortox.lua#L512-L520)
- **边界：FX 与玩家跳跃时长不同步**：fx 按 `animover`＋2 帧退场，状态按 11／14 帧超时转场，两者独立计时；动画实际时长未核验。[超时](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20986)
- **边界：晚加入与重放时序未核验**：跳跃中晚加入的客户端看到的 fx 表现未核验。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；39 行工厂全文、两个注册名、BLINK／BLINK_MAP 动作链、SGwilson portal_jumpin(_pre)／portal_jumpout 生成点与染色、wortox.lua 消耗结算、客户端预测状态、译名已核验；动画资源内容与实际时长、晚加入表现未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
