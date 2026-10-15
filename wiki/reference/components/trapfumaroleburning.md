---
title: "trapfumaroleburning"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "熔岩trap点燃状态：站在活陷阱上的实体持续吃火伤害、积累点燃直至被烤熟或引燃，由陷阱动态装卸。"
aliases: ["熔岩陷阱点燃", "trapfumaroleburning", "地热灼烧"]
tags: [component, energy]
dst_build: "steam-buildid:26230496"
sources: ["components/trapfumaroleburning.lua", "prefabs/trap_fumarole.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# trapfumaroleburning

## 快速判断

- 职责与适用：被熔岩陷阱（`trap_fumarole`）动态加到踩上来的实体上（[加组件](../../../raw/current/scripts/prefabs/trap_fumarole.lua#L65-L69)），按实体所在瓦片解析当前陷阱并施加灼烧。
- 优先使用：不需要主动接入——陷阱在 AOE 扫描时对合格实体加组件；想免疫则用其豁免条件。
- 重要副作用：持续火伤害（骑乘时改由坐骑承担）、温度上升、点燃累积（cookable 者被烤熟、可导热者被引燃）。
- 修改边界：离开陷阱 1 秒后组件自移除（[自移除](../../../raw/current/scripts/components/trapfumaroleburning.lua#L135-L146)）；`OnRemoveFromEntity` 清理 IgnitingEntity 登记与特效（[清理](../../../raw/current/scripts/components/trapfumaroleburning.lua#L15-L22)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `OnUpdate(dt)` | 自动周期 | 定位陷阱→豁免检查→伤害与点燃累积→温度与特效（[主循环](../../../raw/current/scripts/components/trapfumaroleburning.lua#L56-L147)）。 |
| `SetFxEnabled(enable)` | bool | 控制灼烧特效与红 tint，内部按需 `AddComponent(colouradder)`（[特效](../../../raw/current/scripts/components/trapfumaroleburning.lua#L24-L51)）。 |
| `cleartime` / `ignitetime` | 数值 | 离开陷阱计时／点燃累积进度，调试字符串可读（[调试](../../../raw/current/scripts/components/trapfumaroleburning.lua#L149-L151)）。 |
| `FIRE_DAMAGE_DT` | `14 * FRAMES` | 火伤害结算间隔，刻意比火焰 0.5 秒略快以免超时（[注释](../../../raw/current/scripts/components/trapfumaroleburning.lua#L53-L54)）。 |
| 豁免条件 | 标签/状态集合 | limbo、`notarget`/`flying`、sg `flight`/`invisible`/`noattack`、已死、`CanBeAttacked` 为假（[集合](../../../raw/current/scripts/components/trapfumaroleburning.lua#L69-L75)）。 |

## 典型接入

### 让单位免疫熔岩陷阱灼烧

- 前提与选择：组件由陷阱自动装卸，模组能做的是命中豁免条件——任一豁免标签/状态即不累积不伤害。
- 生效与撤销：`notarget` 一类标签随时可加可摘，摘除后下一个更新周期恢复灼烧。
- 原版依据：陷阱侧扫描与加组件的门控（[扫描](../../../raw/current/scripts/prefabs/trap_fumarole.lua#L55-L69)）；伤害、点燃倍率等数值全走 tuning（[调参](../../../raw/current/scripts/tuning.lua#L9501-L9511)）。

```lua
-- 服务器：给自定义生物挂豁免标签（选其一即可长期免疫）
inst:AddTag("notarget") -- 或飞行类 "flying"；sg 状态标签 flight/invisible/noattack 同样有效
```

- 调整与验证：点燃阈值 `TUNING.TRAP_FUMAROLE_IGNITE_TIME`，按陷阱温度段乘 `IGNITE_MULTS`（[乘区](../../../raw/current/scripts/components/trapfumaroleburning.lua#L77-L78)）；调参请改 tuning 而非组件。

## 常见陷阱与限制

- 只按实体中心点所在瓦片解析陷阱（[解析](../../../raw/current/scripts/components/trapfumaroleburning.lua#L57-L61)）：大体积实体尾部悬在陷阱外也算全中，原版已注明该近似（[备注](../../../raw/current/scripts/prefabs/trap_fumarole.lua#L27)）。
- 烤熟走 `cookable:Cook(trap)` 并移除原物（[烤熟](../../../raw/current/scripts/components/trapfumaroleburning.lua#L107-L126)）：无 cookable 而可导热的实体（propagator:AcceptsHeat）则是被引燃而非消失。
- 与 [[reference/components/health]] 的火伤害、[[reference/components/propagator]] 导热直接联动，改其一需复核本组件数值。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文、trap_fumarole 装卸链与相关 tuning；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
