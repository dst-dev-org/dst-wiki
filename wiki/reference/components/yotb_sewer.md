---
title: "yotb_sewer"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "猫年缝纫机组件：按容器内服装配方缝制新服装，支持拒绝退料、火中保产物与断线续缝。"
aliases: ["缝纫机", "yotb_sewer", "服装缝制", "缝纫配方"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/yotb_sewer.lua", "prefabs/yotb_sewingmachine.lua", "actions.lua", "yotb_sewing.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# yotb_sewer

## 快速判断

- 职责与适用：挂在带 `container` 的缝纫机实体上：容器装满且关闭时打 `readytosew` 标签，执行 YOTB_SEW 动作后按 `yotb_sewing.lua` 配方结算产物（[监听](../../../raw/current/scripts/components/yotb_sewer.lua#L26-L30)、[配方](../../../raw/current/scripts/yotb_sewing.lua#L51-L63)）。
- 优先使用：动作层用 `CanSew`/`IsSewing`/`StartSewing`（[动作](../../../raw/current/scripts/actions.lua#L5580-L5596)）；宿主 prefab 只需注册四个表现回调（[注册](../../../raw/current/scripts/prefabs/yotb_sewingmachine.lua#L179-L183)）。
- 重要副作用：`StartSewing` 会 `DestroyContents()` 清空容器并临时禁止打开（[清空](../../../raw/current/scripts/components/yotb_sewer.lua#L113-L122)）；完成时产物带随机速度弹出。
- 修改边界：配方无效时进入 `TUNING.REJECTION_SEWING_TIME` 拒绝流程并把原料全部丢出（[拒绝](../../../raw/current/scripts/components/yotb_sewer.lua#L116-L117)、[doreject](../../../raw/current/scripts/components/yotb_sewer.lua#L60-L72)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `CanSew()` | → `bool` | 容器存在且已满（[实现](../../../raw/current/scripts/components/yotb_sewer.lua#L86-L88)）。 |
| `IsSewing()` / `IsDone()` / `GetTimeToSew()` | → `bool`/`number` | 基于 targettime 与 done 字段（[实现](../../../raw/current/scripts/components/yotb_sewer.lua#L74-L84)）。 |
| `StartSewing(doer)` | → `nil` | 记录 ingredient_prefabs，配方有效则定时出产物，否则定时拒绝；期间容器不可开（[实现](../../../raw/current/scripts/components/yotb_sewer.lua#L90-L124)）。 |
| `StopSewing(reason)` | reason: "fire" 等 | 取消任务；reason=="fire" 时把产物原样投放（[实现](../../../raw/current/scripts/components/yotb_sewer.lua#L130-L147)）。 |
| `onstartsewing` / `oncontinuesewing` / `oncontinuedone` / `ondonesewing` | fn(inst) 字段 | 宿主表现回调，分别在开始/续缝/续完成/完成时调用（[消费](../../../raw/current/scripts/prefabs/yotb_sewingmachine.lua#L180-L183)）。 |
| `OnSave()` / `OnLoad(data)` / `LongUpdate(dt)` | 组件存档钩子 | 存产物/剩余时间/原料表；LongUpdate 推进或直接结算（[实现](../../../raw/current/scripts/components/yotb_sewer.lua#L149-L237)）。 |

## 典型接入

### 给自定义容器接入缝纫

- 前提与选择：实体须有 `container` 组件且容器 widget 槽位与配方原料匹配；配方数据在 `yotb_costumes.lua` 的 costumes 集。
- 生效与撤销：`itemget`/`onclose` 与 `itemlose`/`onopen` 自动维护 `readytosew` 标签（[标签维护](../../../raw/current/scripts/components/yotb_sewer.lua#L3-L13)）；完成后 `canbeopened` 恢复 true。
- 原版依据：缝纫机唯一挂载点并注册回调（[挂载](../../../raw/current/scripts/prefabs/yotb_sewingmachine.lua#L179-L183)）；产物以物理速度向上弹出（[dosew](../../../raw/current/scripts/components/yotb_sewer.lua#L46-L52)）。

```lua
inst:AddComponent("yotb_sewer")
inst.components.yotb_sewer.onstartsewing = OnStartSewing
inst.components.yotb_sewer.ondonesewing = OnDoneSewing
-- 缝纫由 ACTIONS.YOTB_SEW 触发（目标须有 "sewingmachine" 标签）
```

- 调整与验证：缝制时长来自配方的 `time`（缺省 1 秒，[配方](../../../raw/current/scripts/yotb_sewing.lua#L62)）；动作入口要求目标带 `sewingmachine` 标签（[动作](../../../raw/current/scripts/actions.lua#L5581)）。

## 常见陷阱与限制

- `StartSewing` 前提是 `targettime == nil`：缝纫中重复触发动作会被 `IsSewing` 短路（[动作](../../../raw/current/scripts/actions.lua#L5582-L5585)），但直接调用组件会跳过该守卫的前提判断（组件内只查 targettime，[实现](../../../raw/current/scripts/components/yotb_sewer.lua#L91)）。
- OnLoad 依赖 `data.product ~= nil` 才恢复；拒绝流程（product 为 nil）不存档，重载后容器保持锁定为 `canbeopened=false`（[OnLoad 尾部](../../../raw/current/scripts/components/yotb_sewer.lua#L195-L197)），需依赖宿主表现回调补救。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文与 yotb_sewingmachine 挂载、actions YOTB_SEW、yotb_sewing 配方结算；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
