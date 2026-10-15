---
title: "moonsparkchargeable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "月风暴火花充电组件：让带 fueled 的装备被 moonstorm_spark 电击时按百分比回充，月镜护目镜的原版挂载件。"
aliases: ["月暴充电", "moonsparkchargeable", "火花充电"]
tags: [component, energy]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/moonsparkchargeable.lua", "prefabs/hats.lua", "prefabs/moonstorm_spark.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# moonsparkchargeable

## 快速判断

- 职责与适用：给可充电装备（需同时有 fueled 组件）标记"可被月风暴火花充电"；`moonstorm_spark` 按标签找到宿主后调用 `DoSpark` 一次回充固定百分比。[DoSpark](../../../raw/current/scripts/components/moonsparkchargeable.lua#L18-L23)
- 优先使用：做"被雷/被火花劈中回充耐久"类装备时挂本组件并保留 fueled；原版唯一挂载点是 [moonstorm_goggles](../../../raw/current/scripts/prefabs/hats.lua#L2303)。
- 重要副作用：构造即加 `moonsparkchargeable` 标签，OnRemoveFromEntity 摘除；标签是 moonstorm_spark 的 `CHARGE_MUST_TAGS` 查找条件。[标签查找](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L15)
- 修改边界：只与 fueled 协作（`GetPercent/SetPercent`），不支持其他燃料接口；`doer` 参数原版不使用。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `fueled_percent` | `number`（默认 `TUNING.MOONSTORM_SPARKCHARGE_DEFAULT` = 0.1） | 每次 DoSpark 给 fueled 增加的百分比；0.1 即 10%。[默认值](../../../raw/current/scripts/tuning.lua#L9242) |
| `SetFueledPercent(amount)` | `amount: number -> nil` | 覆写单次回充量；grep 全库无原版调用者，属预留接口。[实现](../../../raw/current/scripts/components/moonsparkchargeable.lua#L14-L16) |
| `DoSpark(doer)` | `doer: entity（未使用）-> nil` | `fueled_percent ~= 0` 且有 fueled 时执行 `SetPercent(clamp(GetPercent() + fueled_percent, 0, 1))`。[实现](../../../raw/current/scripts/components/moonsparkchargeable.lua#L18-L23) |
| `moonsparkchargeable`（标签） | — | 构造加、卸载摘；供 spark 检索，建议在 prefab pristine 态显式加。[标签](../../../raw/current/scripts/components/moonsparkchargeable.lua#L7) |

## 典型接入

### 让自定义装备被月风暴火花回充（原版参照）

- 前提与选择：服务端；宿主需有 `fueled` 组件（原版护目镜先挂 fueled 再挂本组件，([hats.lua:2296-2303](../../../raw/current/scripts/prefabs/hats.lua#L2296-L2303)）；客户端 pristine 态加同名标签保持网络端一致（([hats.lua:2280-2283](../../../raw/current/scripts/prefabs/hats.lua#L2280-L2283)）。
- 生效与撤销：`moonstorm_spark` 每 5~15 秒对半径 4 内带标签实体逐个 `DoSpark`（([moonstorm_spark.lua:42-45](../../../raw/current/scripts/prefabs/moonstorm_spark.lua#L42-L45)）；`fueled` 耗尽即装备移除，组件随实体消失。
- 原版依据：护目镜 fueltype 为 USAGE，回充走 fueled 的通用百分比接口，不涉及专用燃料类型。

```lua
-- 服务端：自定义可充电装备挂载（pristine 态还需 AddTag("moonsparkchargeable")）
inst:AddComponent("fueled")
inst.components.fueled.fueltype = FUELTYPE.USAGE
inst.components.fueled:InitializeFuelLevel(TUNING.MOONSTORM_GOGGLES_PERISHTIME)
inst:AddComponent("moonsparkchargeable")
inst.components.moonsparkchargeable:SetFueledPercent(0.2) -- 每次火花回充 20%
```

- 调整与验证：`SetFueledPercent` 是原版未使用的预留口，改动即生效于后续 DoSpark；数值语义是"占 fueled 总量的百分比"，不是秒数。

## 常见陷阱与限制

- **只挂组件不加 fueled：** DoSpark 有 fueled 才生效，缺 fueled 时静默无效果（不做任何事，也不报错）。[守卫](../../../raw/current/scripts/components/moonsparkchargeable.lua#L19)
- **标签与组件不同步：** 若只在 master 分支挂组件而漏加 pristine 标签，客户端判定与 spark 检索（服务端 FindEntities 也按标签）都会失效；按组件内注释建议在 pristine 态加标签。[注释](../../../raw/current/scripts/components/moonsparkchargeable.lua#L6-L7)
- **fueled_percent 置 0 当开关：** `DoSpark` 对 0 直接跳过，可作为"禁用充电"的合法值，但标签仍在、spark 仍会调用一次空操作。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（25 行），核对 hats.lua 护目镜挂载与 pristine 标签、moonstorm_spark.lua 检索与调用、tuning.lua 默认值；grep 确认 `SetFueledPercent` 无原版调用者。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 通过；Lua 示例经 parser_probe --syntax-only 解析通过。
- 游戏运行：未运行；火花回充未实测。

相关：[[reference/components/fueled]]
