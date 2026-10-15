---
title: "gestaltcapturable"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "可被织影牢笼捕获的目标标记组件：以 gestaltcapturable tag 暴露可用态，维护瞄准者集合与等级/位面数据，捕获流程由 gestaltcage 驱动并回调宿主。"
aliases: ["gestalt 捕获目标", "可捕获标记", "gestaltcapturable"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/gestaltcapturable.lua", "components/gestaltcage.lua", "prefabs/alterguardian_phase1.lua", "stategraphs/SGalterguardian_phase1.lua", "components/playercontroller.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# gestaltcapturable

## 快速判断

- 职责与适用：给实体打 `gestaltcapturable` tag（enabled 属性自动同步，components/gestaltcapturable.lua:1-4）；OnTargeted/OnUntargeted/OnCaptured 均注明 "called by gestaltcage component"（:67、:83、:98）。
- 优先使用：让自定实体可捕获→挂组件、`SetLevel(n)`（决定生成 `gestalt_cage_filled<n>`，components/gestaltcage.lua:29）、按需 `SetIsPlanar` 与 `SetOnCapturedFn`。
- 重要副作用：首个瞄准者出现推 `"gestaltcapturable_targeted"`、全部离开推 `"gestaltcapturable_untargeted"`（:78、:93）；OnCaptured 向 doer 推 `"gestaltcaptured"`（:100-102）。
- 修改边界：OnRemoveFromEntity 移除 tag（:23-25）；瞄准者 onremove 自动解除（:16、:72）；操作入口是手持牢笼的 POUNCECAPTURE（componentactions.lua:2349、components/playercontroller.lua:2096、:2173）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetEnabled`/`IsEnabled` | `(bool) -> nil` / `-> bool` | 默认 true；false 时 `GestaltCage:Capture` 直接 MISSED（gestaltcage.lua:15-19）。:27-33 |
| `SetLevel`/`GetLevel` | `(number) -> nil` / `-> number` | 捕获成功后决定牢笼档位；原版 gestalt=1（prefabs/gestalt.lua:205-206）、guard/尸体/月鹿/藤蔓=2、天体英雄一阶段=3（prefabs/alterguardian_phase1.lua:739-740）。:35-41 |
| `SetIsPlanar`/`GetIsPlanar` | `(bool) -> nil` / `-> bool\|nil` | 转交给生成牢笼的 `SetIsPlanar`（gestaltcage.lua:31-33）。:43-49 |
| `OnTargeted`/`OnUntargeted`/`OnCaptured` | `(obj[, doer]) -> nil` | 维护 targeters 集合并监听其 onremove；OnCaptured 依次推 doer 事件、调 oncapturedfn。:68-107 |

## 典型接入

### 原版参照：分批启用捕获

- 前提与选择：天体英雄一阶段初始 `SetEnabled(false)`（alterguardian_phase1.lua:742），由状态图在血量阶段启用/关闭（stategraphs/SGalterguardian_phase1.lua:828、:866、:905）——"可捕获窗口"是常用模式。
- 生效与撤销：Capture 链路=牢笼组件 OnTarget→玩家 POUNCECAPTURE→`GestaltCage:Capture`→读 level/isplanar→移除牢笼、生成填充牢笼（gestaltcage.lua:10-37）。
- 无 mod 接入案例，不提供代码。

## 常见陷阱与限制

- 只加 tag 不挂组件：playercontroller 的搜索仍会选中（playercontroller.lua:2173 只看 tag），但 Capture 判 `IsEnabled` 前会因组件缺失走 MISSED 分支（gestaltcage.lua:15-19）。
- SetEnabled(false) 不清 targeters；已处于 targeted 状态的实体被禁用后，targeted/untargeted 事件时序需宿主自行处理。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 109 行，核验 gestaltcage 完整 Capture/OnTarget 链路、6 处原版挂载（gestalt.lua:205、gestalt_guard.lua:274、corpse_gestalt.lua:117、lunar_grazer.lua:468、lunarthrall_plant_gestalt.lua:80、alterguardian_phase1.lua:739）与动作/控制入口。
- 游戏运行：未运行。

## 相关链接

- 捕获执行方：[[reference/components/gestaltcage]]。
