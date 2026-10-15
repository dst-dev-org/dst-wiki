---
title: "quagmire_hangriness"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "暴食活动 Gnaw 饥饿值：主机侧运动学推进与隆隆提醒，网络变量同步给 HUD 渴望状态条与前后端事件。"
aliases: ["暴食饥饿值", "Gnaw饥饿度", "quagmire_hangriness"]
tags: [component, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/quagmire_hangriness.lua", "prefabs/quagmire_network.lua", "widgets/statusdisplays_quagmire_cravings.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# quagmire_hangriness

## 快速判断

- 职责与适用：挂在暴食活动世界网络实体 `TheWorld.net`（[安装](../../../raw/current/scripts/prefabs/quagmire_network.lua#L56)，注释要求在 worldcharacterselectlobby 之前加），承载 Gnaw 饥饿值 0–6000 的推进、渴望对错反馈与隆隆节奏；对象是活动 HUD 与音效。
- 优先使用：读值用 `GetCurrent`／`GetPercent`／`GetLevel`／`GetTimeRemaining`（[L169-L197](../../../raw/current/scripts/components/quagmire_hangriness.lua#L169-L197)）；HUD 状态条即按此轮询（[消费](../../../raw/current/scripts/widgets/statusdisplays_quagmire_cravings.lua#L5-L17)）。
- 重要副作用：在 TheWorld 推 `quagmirehangrinessrumbled {major}` 与 `quagmirehangrinessmatched {matched}` 事件，并在 `TheFocalPoint` 播 Gnaw 音效（[L62-L70](../../../raw/current/scripts/components/quagmire_hangriness.lua#L62-L70)）。
- 修改边界：主推逻辑在 `quagmire_event_server` 侧（快照外，见下）；客户端只收网络变量，改值无意义；活动外 `levelstart=0` 不更新。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GetPercent()` / `GetCurrent()` | → 0..1 / 0..6000 | 饥饿值与百分比；`MAX_HANGRY = 6000`（[L11](../../../raw/current/scripts/components/quagmire_hangriness.lua#L11)、[L169-L175](../../../raw/current/scripts/components/quagmire_hangriness.lua#L169-L175)）。 |
| `GetLevel()` | → 1..3 | `current<=0 或 speed>=8` 为 3，`speed>=4` 为 2，否则 1；决定隆隆强度与音效音量（[L177-L182](../../../raw/current/scripts/components/quagmire_hangriness.lua#L177-L182)）。 |
| `GetTimeRemaining()` | → 秒 | 按 16 段加速度阈值表对当前值做运动学求和（[L184-L197](../../../raw/current/scripts/components/quagmire_hangriness.lua#L184-L197)）。 |
| `Start(levelstart)` / `Stop()` | 仅 mastersim | 经 `_serverdata` 写网络变量后走 `OnLevelStartDirty` 启停组件更新（[L199-L211](../../../raw/current/scripts/components/quagmire_hangriness.lua#L199-L211)）。 |
| 网络变量 5 个 | net_float ×3＋net_bool ×2 | `current`／`speed`／`levelstart`（脏事件 `levelstartdirty`）＋`rumbled`／`matched`（[L37-L44](../../../raw/current/scripts/components/quagmire_hangriness.lua#L37-L44)）。 |
| `OnUpdate(dt)` | 组件更新 | 阈值表推进 speed 与 current，间隔 ≤10s 强制同步；隆隆倒计时按等级 ×1/×1.5 加速（[L217-L262](../../../raw/current/scripts/components/quagmire_hangriness.lua#L217-L262)）。 |

## 关键流程

1. 阈值表：i=1..15 时 `{threshold = 6000-25i, accel = .018+.002i}`，末段 `{0, .05}`；越接近 0 越快（[L13-L17](../../../raw/current/scripts/components/quagmire_hangriness.lua#L13-L17)）。
2. 等级启停：客户端收 `levelstartdirty`，`levelstart>0` 启动更新且主机注册 craving 匹配回调；归 0 停止（[L121-L137](../../../raw/current/scripts/components/quagmire_hangriness.lua#L121-L137)）。
3. 渴望对错：主机收 craving 事件经 `_serverdata` 改值，写 `matched` 并触发 `quagmirehangrinessmatched`；对/错分别增减饥饿值（[L91-L119](../../../raw/current/scripts/components/quagmire_hangriness.lua#L91-L119)）。
4. 隆隆：等级 >1 时按 `_rumbledelay` 周期播声并写 `rumbled`，客户端收 `rumbleddirty` 转推事件（[L241-L252](../../../raw/current/scripts/components/quagmire_hangriness.lua#L241-L252)）。

## 常见陷阱与限制

- **服务器半边不在脚本快照内**：`event_server_data("quagmire", "components/quagmire_hangriness")`（[L156-L157](../../../raw/current/scripts/components/quagmire_hangriness.lua#L156-L157)，[加载机制](../../../raw/current/scripts/entityscript.lua#L77-L86)）要求 `quagmire_event_server/components/quagmire_hangriness`，该目录随游戏数据包分发、不在 `raw/current/scripts`；`master_postinit`／`OnStart`／`OnDoDelta`／`DoRumble` 等的确切数值未核验，不得据此改写主机数值。
- **HUD 依赖组件存在**：状态条直接读 `TheWorld.net.components.quagmire_hangriness`，非暴食世界该实体不存在，消费端自行判空（[消费](../../../raw/current/scripts/widgets/statusdisplays_quagmire_cravings.lua#L5)）。
- **同步是粗粒度**：`current`／`speed` 平时 `set_local`，仅渴望对错与周期 10s 时才网络同步；客户端短时数值可能滞后（[L94-L97](../../../raw/current/scripts/components/quagmire_hangriness.lua#L94-L97)、[L254-L261](../../../raw/current/scripts/components/quagmire_hangriness.lua#L254-L261)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、quagmire_network 装配段、statusdisplays 消费行与 event_server_data 加载机制（entityscript.lua）；未覆盖 quagmire_event_server 内主机逻辑（快照外）与活动全流程。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/quagmire_music]]、[[reference/components/quagmire_recipebook]]（同挂 quagmire_network 的活动组件）。
