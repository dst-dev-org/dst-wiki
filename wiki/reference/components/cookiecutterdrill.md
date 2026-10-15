---
title: "cookiecutterdrill"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "记录饼干切割者钻船进度并在钻完时触发船体漏水与进食计时的状态组件，进度由状态图驱动。"
aliases: ["cookiecutterdrill", "钻船计时", "饼干切割者钻船", "船体漏水触发"]
tags: [component, ai]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/cookiecutterdrill.lua", "prefabs/cookiecutter.lua", "stategraphs/SGcookiecutter.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# cookiecutterdrill

## 快速判断

- 职责与适用：给饼干切割者维护"本次钻船已持续多久"，并提供完成判定与完成副作用（漏水事件）；OnUpdate 只累加时间。([cookiecutterdrill.lua:52-54](../../../raw/current/scripts/components/cookiecutterdrill.lua#L52-L54)
- 优先使用：进度由 SGcookiecutter 的 drill 状态机调用 ResumeDrilling／PauseDrilling／ResetDrilling／GetIsDoneDrilling／FinishDrilling 驱动，勿在脑或计时器里另起一套。[SGcookiecutter.lua:529](../../../raw/current/scripts/stategraphs/SGcookiecutter.lua#L529 [SGcookiecutter.lua:558](../../../raw/current/scripts/stategraphs/SGcookiecutter.lua#L558 [SGcookiecutter.lua:590](../../../raw/current/scripts/stategraphs/SGcookiecutter.lua#L590
- 重要副作用：`FinishDrilling` 推 `spawnnewboatleak`（med_leak）到所站船，并设置 eater 的 lasteattime。([cookiecutterdrill.lua:35-50](../../../raw/current/scripts/components/cookiecutterdrill.lua#L35-L50)
- 修改边界：实体睡眠自动停更；唤醒不自动续钻（源码注释：由状态图处理）。([cookiecutterdrill.lua:14-17](../../../raw/current/scripts/components/cookiecutterdrill.lua#L14-L17)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `drill_duration` | 数值秒，默认 10 | 消费者覆写为 `TUNING.COOKIECUTTER.DRILL_TIME`（3 秒）。([cookiecutter.lua:199-200](../../../raw/current/scripts/prefabs/cookiecutter.lua#L199-L200) [tuning.lua:4085](../../../raw/current/scripts/tuning.lua#L4085 |
| `GetIsDoneDrilling()` | 布尔 | `drill_progress >= drill_duration`。([cookiecutterdrill.lua:19-21](../../../raw/current/scripts/components/cookiecutterdrill.lua#L19-L21) |
| `ResumeDrilling()` / `PauseDrilling()` / `ResetDrilling()` | 无参 | 开更／停更／进度清零。([cookiecutterdrill.lua:23-33](../../../raw/current/scripts/components/cookiecutterdrill.lua#L23-L33) |
| `FinishDrilling()` | 无参 | 停更清零；设置 `eater.lasteattime`；有 `leak_damage` 且船有 hullhealth 时 `boat.components.health:DoDelta(leak_damage)`；推 `spawnnewboatleak`。([cookiecutterdrill.lua:35-50](../../../raw/current/scripts/components/cookiecutterdrill.lua#L35-L50) |
| `leak_damage` / `leak_type` | 数值／字符串，默认 nil／"med_leak" | leak_damage 未设则不掉船血；leak_type 字段存了但事件里硬编码 "med_leak"。[cookiecutterdrill.lua:7](../../../raw/current/scripts/components/cookiecutterdrill.lua#L7 ([cookiecutterdrill.lua:45-48](../../../raw/current/scripts/components/cookiecutterdrill.lua#L45-L48) |

## 典型接入

### 调整钻船时长与伤害

- 前提与选择：改参数在 AddComponent 后覆写字段（原版即如此改 drill_duration）；漏血另设 leak_damage。([cookiecutter.lua:199-200](../../../raw/current/scripts/prefabs/cookiecutter.lua#L199-L200)

```lua
inst:AddComponent("cookiecutterdrill")
inst.components.cookiecutterdrill.drill_duration = 5
inst.components.cookiecutterdrill.leak_damage = -20
```

- 生效与撤销：状态图在 jump_pst_boat 清零、drill 态续钻并逐帧查完成、drill_pst 时间线里 FinishDrilling。[SGcookiecutter.lua:529](../../../raw/current/scripts/stategraphs/SGcookiecutter.lua#L529 [SGcookiecutter.lua:606](../../../raw/current/scripts/stategraphs/SGcookiecutter.lua#L606 [SGcookiecutter.lua:634](../../../raw/current/scripts/stategraphs/SGcookiecutter.lua#L634

## 常见陷阱与限制

- `FinishDrilling` 判 `boat.components.hullhealth` 却调 `boat.components.health:DoDelta`：两者需同时存在才生效，别假设改 hullhealth 就安全。([cookiecutterdrill.lua:45-47](../../../raw/current/scripts/components/cookiecutterdrill.lua#L45-L47)
- 进度不存档：重载后重新可钻；睡眠即停更，剩余进度保留但不再累加。([cookiecutterdrill.lua:15-17](../../../raw/current/scripts/components/cookiecutterdrill.lua#L15-L17)
- `sound`／`sound_intensity` 字段仅声明未被读取；`leak_type` 同样未参与事件负载。

## 验证记录

- 源码审阅：2026-10-28，`steam-buildid:26230496`：审阅组件全文、cookiecutter 安装与覆写、SGcookiecutter 五处驱动点及 tuning 值。
- 游戏运行：未运行；未复现钻船漏水完整链路。
- 结构检查：validate_page.py 通过；validate_source_links.py --strict 通过（本批 16 页合并执行）。
