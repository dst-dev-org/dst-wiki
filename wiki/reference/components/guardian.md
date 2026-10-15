---
title: "guardian"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "守护者召唤组件：宿主累计召唤计数达阈值自动召唤或寻回守护者，计数衰减归零或守护者死亡时解除，供巢穴-守护兽绑定。"
aliases: ["守护者召唤", "召唤守护兽", "guardian"]
tags: [component, ai, lifecycle]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/guardian.lua", "prefabs/mooseegg.lua", "actions.lua", "brains/mosslingbrain.lua", "stategraphs/SGmooseegg.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# guardian

## 快速判断

- 职责与适用：维护"宿主 ↔ 守护者实体"的计数召唤关系（文件头注记：守护者由其他生物召唤，components/guardian.lua:1）。计数到 `threshold`（默认 20）自动召唤，衰减到 0 或守护者死亡／移除时解除（lines 43-60、118-137）。
- 优先使用：原版唯一消费者麋鹿蛋巢——蛋孵化时 `SummonGuardian(母亲)`＋`DoDelta(1)`（stategraphs/SGmooseegg.lua:33-34），玩家逗弄走 `ACTIONS.SUMMONGUARDIAN → Call()`（actions.lua:1680-1683）。
- 重要副作用：每次 `DoDelta` 推送 `summonsdelta` 事件（line 49）；每次计数变化都会重置 20 秒的衰减任务（lines 83-91）。
- 修改边界：`OnRemoveFromEntity` 取消衰减并解绑守护者（lines 21-27）；守护者经 `death`／`onremove` 事件自动清引用（lines 17-18、29-41）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `prefab` | string | 要召唤的守护者 prefab 名；未设置时 `SummonGuardian` 只打印警告（lines 5、93-97） |
| `threshold`／`summons` | 默认 20／0 | 计数上限与当前值（lines 11-12）；`DoDelta` 夹在 0..threshold（line 47） |
| `decaytime` | 默认 20 秒 | 无新计数时每次递减 1 的间隔；任务在每次 `DoDelta` 重置（lines 14、78-91） |
| `DoDelta(d)` | 计数增减 | 自动判断：满且无守护者→召唤；归零且有守护者→解除（lines 43-60） |
| `Call(d)`／`Decay(d)` | 缺省 ±1 | 语义化封装（lines 70-76） |
| `SummonGuardian(override)` | 可传已有实体 | 先在 30 距离找同 prefab 实体复用（line 103），否则生成并放到宿主位置、调 `onsummonfn`（lines 108-113） |
| `DismissGuardian()` | 解除 | 有 `ondismissfn` 则回调后清引用，否则直接 `guardian:Remove()`（lines 126-137） |
| `SetGuardian(guardian)` | 绑定／解绑 | 管理 `death`／`onremove` 监听（lines 29-41） |
| `onguardiandeathfn`／`onsummonfn`／`ondismissfn` | 回调 | 死亡回调收 `cause`（lines 118-124）；mooseegg.lua:137-141 三回调齐配 |
| `OnSave`／`OnLoad`／`LoadPostPass` | 存计数＋守护者 GUID 引用 | 载入后经 `SummonGuardian(实体)` 重建绑定（lines 143-172） |

## 典型接入

### 巢穴型守护兽（服务端 master postinit）

- 前提与选择：守护者是普通生物 prefab；召唤优先复用世界现有实体，因此同一守护兽可被多个宿主争抢（line 103 只查距离不查归属）。
- 生效与撤销：计数自然衰减（20 秒 -1）；`DoDelta` 归零即触发解除。守护者死亡后计数保留，可继续 `Call` 重召。
- 原版依据：prefabs/mooseegg.lua:137-141；幼麋鹿脑按守护者状态决定跟随目标（brains/mosslingbrain.lua:145、177-178）。

```lua
inst:AddComponent("guardian")
inst.components.guardian.prefab = "tallbird"
inst.components.guardian.threshold = 10
inst.components.guardian.decaytime = 20
inst.components.guardian.onguardiandeathfn = function(inst, guardian, cause)
    inst:PushEvent("my_guardian_lost")
end
-- 计数：inst.components.guardian:Call(1)；手动衰减：inst.components.guardian:Decay()
```

- 调整与验证：`threshold` 即"满值自动召唤"的触发点，`summonsdelta` 事件可驱动 HUD／语音；保存的 GUID 引用在 `LoadPostPass` 才能解析（lines 164-172）。

## 常见陷阱与限制

- **忘设 prefab 静默失败：** `SummonGuardian` 只 print 不报错（lines 94-97），挂着组件不生效时先查 `prefab` 字段。
- **衰减任务单槽：** `StartDecay` 取消旧任务再建新任务（lines 83-91），等价于"每次计数刷新 20 秒倒计时"，而不是每计数独立倒计时。
- **onrush 复用判定宽松：** 30 距离内任何同名 prefab 都会被认作守护者（line 103），同名野生生物会顶替召唤计划。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。通读 components/guardian.lua 全文（174 行）；核验 mooseegg.lua:137-141、actions.lua:1680-1683、SGmooseegg.lua:33-37、mosslingbrain.lua:145/177-178、mossling.lua:36/96。未运行游戏。
- 结构检查：validate_page.py / validate_source_links.py 通过（见批级自验）。

相关：[[reference/components/herd]]、[[reference/components/herdmember]]
