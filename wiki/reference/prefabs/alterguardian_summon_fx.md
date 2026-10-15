---
title: "天体英雄召唤光柱（alterguardian_summon_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "天体英雄三个阶段的召唤警示光柱：联网主光柱显式循环＋逐帧灯光调制，非专服本地背景层；只有生成者推 endloop 事件才进入收尾并自删。"
aliases: ["alterguardian_summon_fx", "天体英雄召唤光柱", "召唤警示光柱", "天体英雄召唤特效", "gestalt summon fx"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/alterguardian_summon_fx.lua", "prefabs/alterguardian_phase1.lua", "stategraphs/SGalterguardian_phase2.lua", "stategraphs/SGalterguardian_phase3.lua", "languages/chinese_s.po"]
size_class: simple
reliability: verified
validation: source-reviewed
status: active
---

# 天体英雄召唤光柱（alterguardian_summon_fx）

## 快速判断

- 职责与适用：天体英雄（alterguardian_phase1／2／3，官方译名见[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L337939-L337942)）召唤弹幕前的蓝光警示柱：主光柱（FinalOffset 1）＋非专服本地背景层（-1），主模拟逐帧调制灯光。纯表现＋警示，无玩法组件。[工厂](../../../raw/current/scripts/prefabs/alterguardian_summon_fx.lua#L153-L193)、[灯光调制](../../../raw/current/scripts/prefabs/alterguardian_summon_fx.lua#L95-L105)
- 优先使用：查"召唤范围警示／蓝光光柱"选本页；三个生成者见关联机制导航。
- 重要副作用：主光柱动画显式循环，只有生成者推 `endloop` 事件才播 `summon_pst` 并自删；背景层经 `_end_loop` 网络事件同步收尾。[收尾](../../../raw/current/scripts/prefabs/alterguardian_summon_fx.lua#L6-L13)、[endloop 监听](../../../raw/current/scripts/prefabs/alterguardian_summon_fx.lua#L185)
- 修改边界：亮度任务只在主模拟，背景层只在非专服；工厂不设缩放，由生成者传 1.2／1.3。[任务挂载](../../../raw/current/scripts/prefabs/alterguardian_summon_fx.lua#L187-L188)、[背景层入口](../../../raw/current/scripts/prefabs/alterguardian_summon_fx.lua#L174-L176)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `alterguardian_summon_fx` | fn：Transform＋AnimState＋Light＋Network → 灯光初始 0、蓝色 (0,0.35,1) → bank/build `alterguardian_summon_fx` 播 `summon_pre` 后 Push `summon_loop`（**显式循环**）、FinalOffset 1、FX 标签 → `_end_loop` net_event → 非专服 0 帧后建本地背景层 → SetPristine → 客户端早退 → 主模拟监听 `endloop`、pre 时长后置 `_pre_finished`、每帧灯光调制 → `persists=false` | 单注册；缩放、位置完全由生成者决定。[工厂](../../../raw/current/scripts/prefabs/alterguardian_summon_fx.lua#L153-L193)、[注册](../../../raw/current/scripts/prefabs/alterguardian_summon_fx.lua#L195) |

灯光调制按当前动画帧查三张表：pre 段 24 帧（升到 0.99）、loop 段 16 帧波动（0.85-0.99）、pst 段 16 帧（1.0 降到 0.0625），按值缩放 Light 的 intensity(0.6×val²)／radius(8×val)／falloff(4×val) 与 LightOverride。[表与取值](../../../raw/current/scripts/prefabs/alterguardian_summon_fx.lua#L27-L93)

本 FX 无 NAMES 中文条目；"天体英雄"是宿主 Boss 的官方译名，标题"召唤光柱"为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：phase1 DoGestaltSummon | 生成于 Boss 自身位置（scale 1.2）作警示，随后按人数环布 `gestalt_alterguardian_projectile`，最后一名+1 秒后推 `endloop` | 进入护盾或 summon_cooldown 计时到点且仍在护盾时触发 | [召唤](../../../raw/current/scripts/prefabs/alterguardian_phase1.lua#L248-L290)、[endloop](../../../raw/current/scripts/prefabs/alterguardian_phase1.lua#L285-L287)、[两个触发点](../../../raw/current/scripts/prefabs/alterguardian_phase1.lua#L292-L299) |
| 生成者：SGphase2 atk_summon | 一轮召唤链首次进入状态时生成（scale 1.3）存 `sg.mem.summon_fx`；状态 onexit 在链中断或结束时推 `endloop` | 仅首帧生成；22 帧地面击点配镜头震动 | [状态](../../../raw/current/scripts/stategraphs/SGalterguardian_phase2.lua#L582-L648)、[生成](../../../raw/current/scripts/stategraphs/SGalterguardian_phase2.lua#L596-L600)、[onexit 收尾](../../../raw/current/scripts/stategraphs/SGalterguardian_phase2.lua#L642-L648) |
| 生成者：SGphase3 召唤圈状态组 | atk_summon_pre 经 start_summon_circle 生成（scale 1.3，已有则复用），pst／退出经 stop_summon_circle 推 `endloop` | 与 `alterguardian_phase3circle` 地面圈并存 | [start／stop](../../../raw/current/scripts/stategraphs/SGalterguardian_phase3.lua#L222-L245)、[pre 态调用](../../../raw/current/scripts/stategraphs/SGalterguardian_phase3.lua#L445-L455) |
| net_event：`_end_loop` | 服务端播 pst 时推给客户端，触发背景层播 `summon_back_pst` 并 animover 自删 | 背景层为非联网本地实体、`SetCanSleep(false)`、FinalOffset -1、跳 1 帧起步 | [net_event 声明](../../../raw/current/scripts/prefabs/alterguardian_summon_fx.lua#L172)、[背景层工厂](../../../raw/current/scripts/prefabs/alterguardian_summon_fx.lua#L107-L132)、[客户端收尾](../../../raw/current/scripts/prefabs/alterguardian_summon_fx.lua#L134-L151) |
| 资源：alterguardian_summon_fx.zip | 主层 `summon_pre/loop/pst`、背景层 `summon_back_pre/loop/pst` 两组动画 | loop 段均显式传循环 true；实际帧数未核验 | [主层动画](../../../raw/current/scripts/prefabs/alterguardian_summon_fx.lua#L164-L168)、[背景层动画](../../../raw/current/scripts/prefabs/alterguardian_summon_fx.lua#L122-L123) |

## 典型接入

复用为自定义召唤警示：生成后设置位置与缩放，结束时 `fx:PushEvent("endloop")` 即走原版收尾（pst＋animover 自删＋背景层同步退场）。[收尾函数](../../../raw/current/scripts/prefabs/alterguardian_summon_fx.lua#L6-L13)

```lua
local fx = SpawnPrefab("alterguardian_summon_fx")
fx.Transform:SetPosition(x, y, z)
fx.Transform:SetScale(1.3, 1.3, 1.3)
-- 召唤物就位后：
fx:PushEvent("endloop")
```

- 调整与验证：灯光节奏由三张帧表决定（[表](../../../raw/current/scripts/prefabs/alterguardian_summon_fx.lua#L27-L93)），改动画时需同步表长；无伤害、无存档（`persists=false`）。

## 常见陷阱与限制

- **误区：生成后不管会自动消失**：`summon_loop` 显式循环且工厂不自设寿命，唯一收尾是推 `endloop` 事件；直接 Spawn 而不推事件的 FX 会永远循环（`persists=false` 只保证不入存档）。[监听](../../../raw/current/scripts/prefabs/alterguardian_summon_fx.lua#L185)
- **边界：灯光调制只在主模拟**：客户端 Light 参数靠引擎复制；调制任务与 `endloop` 监听都在 SetPristine 之后注册。[分界](../../../raw/current/scripts/prefabs/alterguardian_summon_fx.lua#L178-L188)
- **边界：背景层专服不存在**：`TheNet:IsDedicated()` 为真时跳过背景层，专服只有主光柱；背景层位置不跟随主层（同帧生成时对齐）。[入口](../../../raw/current/scripts/prefabs/alterguardian_summon_fx.lua#L174-L176)
- **边界：三个生成者缩放不同**：phase1 传 1.2，phase2／3 传 1.3；工厂不设默认缩放，直接复用需自行设定。[缩放证据](../../../raw/current/scripts/prefabs/alterguardian_phase1.lua#L251-L253)

## 验证记录

- 源码审阅：2026-10-30，build 26230496；196 行工厂全文（灯光三表、双工厂、endloop 链）、phase1／SGphase2／SGphase3 生成点与收尾、译名已核验；宿主 Boss 完整战斗链、动画实际帧数、晚加入表现未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、链接 --strict、label-check 通过；本批依据见[[updates/build-26335394/index#wiki-沿用记录]]（批次-2）。

返回 [[reference/prefabs/fx-creature-index|生物、Boss 与遭遇演出]]、[[reference/prefabs/index|Prefab 索引]]。
