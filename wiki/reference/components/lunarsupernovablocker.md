---
title: "lunarsupernovablocker"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "把实体标记为天体英雄超新星的合法遮挡物：为每个超新星来源挂腿部光束特效并轻微闪烁染色，来源失效或出界时自动清除。"
aliases: ["超新星遮挡物标记", "超新星掩体", "lunarsupernovablocker"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/lunarsupernovablocker.lua", "prefabs/wagboss_robot_leg.lua", "stategraphs/SGalterguardian_phase4_lunarrift.lua", "prefabs/wagboss_util.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# lunarsupernovablocker

## 快速判断

- 职责与适用：机器人腿部等遮挡物专用——维护"超新星来源→特效实体"映射，来源有效期间用 colouradder 做闪烁染色；战斗机制读 `lunarsupernovablocker` tag 判定掩体（prefabs/wagboss_util.lua:132 `BLOCKER_TAGS`）。
- 优先使用：想提供超新星掩体即挂组件并保证 pristine tag；来源增删只由天体英雄状态图驱动（stategraphs/SGalterguardian_phase4_lunarrift.lua:226-227 调 AddSource）。
- 重要副作用：每个来源生成 `wagboss_robot_leg_fx` 子实体并朝向来源（components/lunarsupernovablocker.lua:32-41）；首来源开启组件更新，末来源关闭并回调。
- 修改边界：服务器组件；tag 在构造时加、OnRemoveFromEntity 时移除（:12、:15-17）；来源清理完全依赖 OnUpdate 轮询，原版无外部 RemoveSource 调用者。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `AddSource` | `(source) -> nil` | 记录来源（须 sg 带 `supernovaburning` state tag），生成 fx 子实体定向；首来源时 StartUpdatingComponent + `onstartblockingfn`。lunarsupernovablocker.lua:27-55 |
| `RemoveSource` | `(source) -> nil` | 移除 fx；清空后 StopUpdatingComponent、`colouradder:PopColour("lunarsupernovablocker")`、`onstopblockingfn`。:57-70 |
| `SetOnStartBlockingFn`/`SetOnStopBlockingFn` | `(fn) -> nil` | 遮挡开始/结束回调；原版用于切 jiggle/idle 动画（prefabs/wagboss_robot_leg.lua:33-42、:66-67）。:19-25 |
| `OnUpdate` | `(dt) -> nil` | 来源需 `k.sg:HasStateTag("supernovaburning")` 且 IsValid；竞技场屏障升起时来源须在 `Map:IsPointInWagPunkArena` 内，否则超 `WagBossUtil.SupernovaNoArenaRangeSq`（24²）移除。:81-106、prefabs/wagboss_util.lua:188-189 |
| tag `lunarsupernovablocker` | 布尔 | 状态图 `FindEntities_Registered` 的 must tag（SGalterguardian_phase4_lunarrift.lua:213-219）；组件构造/移除时同步。:12、:15-17 |

## 典型接入

### 原版参照：天体英雄最终阶段掩体

- 前提与选择：仅 `wagboss_robot_leg` 挂载（prefabs/wagboss_robot_leg.lua:65），落地完成后注册两个回调。
- 生效与撤销：boss 超新星状态每帧对范围内带 tag 实体 AddSource（SGalterguardian_phase4_lunarrift.lua:219-227）；来源离开超新星状态/出界/失效由组件 OnUpdate 移除（:88-105）。
- 无 mod 接入案例，不提供代码。

## 常见陷阱与限制

- 同坐标来源的 fx 会 `Hide()`（:37-41、:98-99），遮挡方向仅在来源与挂载者错位时可见。
- 非超新星状态实体作来源会被 OnUpdate 立即移除（:89），无法用普通实体伪造遮挡源。
- 闪烁幅度随机 `easing.inOutQuad(math.random(), 0.15, 0.1, 1)`（:72-79），依赖宿主带 colouradder。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 108 行并核验挂载（wagboss_robot_leg.lua:65-67）、唯一 AddSource 调用方（SGalterguardian_phase4_lunarrift.lua:226-227）与 wagboss_util 辅助常量；RemoveSource 全库仅组件内部调用。
- 游戏运行：未运行。

## 相关链接

- 同属裂隙5月灼体系的灼烧组件：[[reference/components/lunarfissureburning]]（超新星本体灼烧组件 lunarsupernovaburning 暂无知识页）。
