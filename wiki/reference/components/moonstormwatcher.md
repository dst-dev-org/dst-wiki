---
title: "moonstormwatcher"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家侧月暴响应组件：监听世界月暴开关与 stormwatcher 等级，按护目镜/幽灵视觉、骑乘状态控制 locomotor 的月暴减速修正。"
aliases: ["月暴减速", "月暴步行速度", "moonstormwatcher"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/moonstormwatcher.lua", "prefabs/player_common.lua", "components/stormwatcher.lua", "components/moonstorms.lua", "prefabs/woodie.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# moonstormwatcher

## 快速判断

- 职责与适用：所有玩家挂载（prefabs/player_common.lua:2741）；世界 `moonstorms` 开/关月暴时同步 enabled（components/moonstorms.lua:126、:132 推 `ms_stormchanged`），并按月暴等级决定是否给 locomotor 挂 `moonstorm` 外部减速。
- 优先使用：调整月暴中减速强度→`SetMoonstormSpeedMultiplier(mult)`（clamp 0..1，默认 `TUNING.MOONSTORM_SPEED_MOD`）；查询等级→`GetMoonStormLevel()`。
- 重要副作用：PushEvent `"moonstormlevel"`（components/moonstormwatcher.lua:81）；enabled 且减速生效时动态挂/卸 gogglevision、ghostvision、mounted、dismounted 监听（:21-34）。
- 修改边界：服务器组件（玩家 master postinit 挂载）；依赖 stormwatcher、playervision、rider、locomotor；OnRemoveFromEntity 只在减速生效时清理（:36-40）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `ToggleMoonstorms` | `(data) -> nil` | 仅响应 `STORM_TYPES.MOONSTORM`；构造时若月暴节点已存在会立即同步一次（:9-14）。:42-59 |
| `SetMoonstormSpeedMultiplier` | `(mult) -> nil` | clamp 0..1；mult≥1 移除监听，mult 从≥1 变<1 时补挂；enabled 时立即重算。:61-76 |
| `GetMoonStormLevel` | `() -> number\|nil` | 转发 `stormwatcher:GetStormLevel(STORM_TYPES.MOONSTORM)`；stormwatcher.lua:116 在月暴等级刷新时调 `UpdateMoonstormLevel`。:101-106 |
| `UpdateMoonstormWalkSpeed_Internal` | `(level) -> nil` | level<SANDSTORM_FULL_LEVEL 或有护目镜/幽灵视觉或骑乘→`RemoveExternalSpeedMultiplier`，否则 `SetExternalSpeedMultiplier(inst,"moonstorm",mult)`。:88-99 |

## 典型接入

### 调整月暴减速强度（服务器）

- 前提与选择：组件默认倍率即 `TUNING.MOONSTORM_SPEED_MOD`；mod 在玩家 master postinit 后调 setter 即可，原版在道具施法开始/结束（player_common.lua:311、:321）与 Woodie 变形切换（prefabs/woodie.lua:1221、:1277、:1335、:1393）会重置回原值。
- 生效与撤销：setter 只影响 `moonstorm` 这一项外部速度修正；卸载 mod 后倍率残留需自行复位。

```lua
-- modmain.lua（服务器端）：把月暴减速改为原版一半
AddPrefabPostInit("wilson", function(inst)
    if not TheWorld.ismastersim then
        return
    end
    inst.components.moonstormwatcher:SetMoonstormSpeedMultiplier(
        TUNING.MOONSTORM_SPEED_MOD * 0.5)
end)
```

- 调整与验证：注意道具施法期间会被覆盖为原值，需要持续压制时监听重置时机再调。

## 常见陷阱与限制

- 非玩家实体挂载会在重算时因缺 playervision/rider 报错（:90-96 直接访问组件）。
- 等级为 nil（无 stormwatcher）时重算整体跳过（:89），减速保持上一次状态。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 108 行，核验挂载点（player_common.lua:2741）、世界事件源（moonstorms.lua:126/132）、等级联动（stormwatcher.lua:116）与 Woodie 重置调用（woodie.lua:1221 等 4 处）。
- 游戏运行：未运行。

## 相关链接

- 同类风暴速度组件：[[reference/components/sandstormwatcher]]（月暴等级来源组件 stormwatcher 暂无知识页）。
