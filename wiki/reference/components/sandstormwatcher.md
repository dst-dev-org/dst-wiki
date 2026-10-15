---
title: "sandstormwatcher"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家沙尘暴监听器：按沙暴等级、护目/骑乘状态调节移速惩罚并广播 sandstormlevel。"
aliases: ["沙尘暴监听", "沙暴减速", "sandstormwatcher"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/sandstormwatcher.lua", "components/stormwatcher.lua", "prefabs/player_common.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# sandstormwatcher

## 快速判断

- 职责与适用：玩家 master 组件（[安装](../../../raw/current/scripts/prefabs/player_common.lua#L2740)），在世界 `sandstorms` 组件存在时监听 `ms_stormchanged` 开关沙暴（[监听](../../../raw/current/scripts/components/sandstormwatcher.lua#L6-L15)）；依赖 stormwatcher/playervision/rider/locomotor。
- 优先使用：改沙暴移速惩罚走 `SetSandstormSpeedMultiplier`；读当前强度走 `GetSandstormLevel`。
- 重要副作用：激活期间向 locomotor 写/删键名 `"sandstorm"` 的外部移速乘数，移除组件会顺手清理（[清理](../../../raw/current/scripts/components/sandstormwatcher.lua#L29-L41)）。
- 修改边界：满级沙暴且无护目/鬼眼/骑乘才减速；`TUNING.SANDSTORM_FULL_LEVEL` 0.7、默认乘数 `SANDSTORM_SPEED_MOD` 0.4（tuning.lua:3331-3333）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `ToggleSandstorms(active)` | `boolean` | 开关沙暴响应并增删 gogglevision/ghostvision/mounted/dismounted 监听（[实现](../../../raw/current/scripts/components/sandstormwatcher.lua#L43-L58)）。 |
| `SetSandstormSpeedMultiplier(mult)` | `number`，钳 0~1 | 修改减速倍数；跨 1 边界自动增删监听（[实现](../../../raw/current/scripts/components/sandstormwatcher.lua#L60-L75)）。 |
| `UpdateSandstormLevel()` | 无参数 | 取等级并推 `sandstormlevel {level}` 事件（[实现](../../../raw/current/scripts/components/sandstormwatcher.lua#L77-L81)；由 [stormwatcher](../../../raw/current/scripts/components/stormwatcher.lua#L109-L113) 驱动）。 |
| `UpdateSandstormWalkSpeed()` | 无参数 | 重算外部乘数：等级 < 0.7、有护目/鬼眼或骑乘则恢复正常移速（[判定](../../../raw/current/scripts/components/sandstormwatcher.lua#L87-L98)）。 |
| `GetSandstormLevel()` | → `number or nil` | 转发玩家 `stormwatcher:GetStormLevel(STORM_TYPES.SANDSTORM)`（[实现](../../../raw/current/scripts/components/sandstormwatcher.lua#L100-L105)）。 |
| `enabled` / `sandstormspeedmult` | `boolean` / `number` | 当前沙暴是否激活、减速乘数（默认 `TUNING.SANDSTORM_SPEED_MOD`，[构造](../../../raw/current/scripts/components/sandstormwatcher.lua#L1-L4)）。 |

## 典型接入

### 调整沙暴移速惩罚

- 前提与选择：组件由玩家 fn 自动安装，模组在 master_postinit 后直接调用即可。
- 生效与撤销：立即对 locomotor 生效；`woodie` 在变身形态下把乘数设回 1 解除减速（[消费](../../../raw/current/scripts/prefabs/woodie.lua#L1276)）。
- 原版依据：咏唱施法时与多重减速源取最小值叠加（[消费](../../../raw/current/scripts/prefabs/player_common.lua#L310-L320)）。

```lua
-- 玩家 master_postinit 内自定义沙暴减速
inst.components.sandstormwatcher:SetSandstormSpeedMultiplier(0.6) -- 满级沙暴残存 60% 移速
```

- 调整与验证：`sandstormlevel` 事件在 scripts 内无监听者（grep 全库仅推送点），消费方为引擎/界面层；判定自己 UI 时需自行监听该事件。

## 常见陷阱与限制

- `SetSandstormSpeedMultiplier(1)` 在未激活沙暴时只改值不摘监听（[实现](../../../raw/current/scripts/components/sandstormwatcher.lua#L63-L69)），影响下次开暴行为，重置逻辑要成对处理。
- 世界缺 `sandstorms` 组件（无沙暴的世界）时组件完全不响应 `ms_stormchanged`，构造时也不补初始态（[守卫](../../../raw/current/scripts/components/sandstormwatcher.lua#L6)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、stormwatcher 驱动点与 player_common/woodie 消费链、tuning 取值；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
