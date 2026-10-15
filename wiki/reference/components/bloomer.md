---
title: "bloomer"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "泛光效果堆叠：按优先级管理多个 bloom 特效来源，栈顶生效并自动联动子实体，来源移除自动弹出。"
aliases: ["泛光堆叠", "bloomer", "bloom 特效"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/bloomer.lua", "prefabs/backpack.lua", "prefabs/amulet.lua", "prefabs/alterguardian_laser.lua", "prefabs/player_common.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# bloomer

## 快速判断

- 职责与适用：把"谁想要什么泛光"做成优先级栈，栈顶应用到 `AnimState:SetBloomEffectHandle`；玩家、猪人、皮划艇背包等 19 处 prefab 注册（[玩家](../../../raw/current/scripts/prefabs/player_common.lua#L2668)）。
- 优先使用：多来源叠加泛光（装备、状态、命中效果）时用 `PushBloom`/`PopBloom`，替代直接操作 AnimState。
- 重要副作用：`PushBloom`/`PopBloom` 会把栈顶变化同步给全部已挂子实体（[联动](../../../raw/current/scripts/components/bloomer.lua#L65-L85)）；实体来源被移除时自动弹出（[自动](../../../raw/current/scripts/components/bloomer.lua#L6)）。
- 修改边界：fx 参数是 shader 句柄字符串（如 `shaders/anim.ksh`），组件不校验；priority 默认 0。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `PushBloom(source, fx, priority)` | source 为实体或任意键 | 入栈/更新并按优先级插入，栈顶变化才重设 shader（[实现](../../../raw/current/scripts/components/bloomer.lua#L87-L127)）。 |
| `PopBloom(source)` | 同上键 | 弹出对应层并恢复新栈顶或清空（[实现](../../../raw/current/scripts/components/bloomer.lua#L129-L148)）。 |
| `GetCurrentFX()` / `GetCurrentFxAndPriority()` | → `string or nil` | 读当前生效层（[实现](../../../raw/current/scripts/components/bloomer.lua#L54-L63)）。 |
| `AttachChild(child)` / `DetachChild(child)` | 子实体 | 子实体跟随宿主栈顶；子实体无 bloomer 时直写 AnimState（[实现](../../../raw/current/scripts/components/bloomer.lua#L23-L52)）。 |
| `GetDebugString()` | 无参数 | 从栈顶打印优先级与来源（[实现](../../../raw/current/scripts/components/bloomer.lua#L150-L157)）。 |

## 典型接入

### 给可叠加状态加/撤泛光

- 前提与选择：双端各自维护（无网络变量）；服务器与客户端都要在对应时机调用。
- 生效与撤销：`PushBloom` 用固定 key，效果结束时配对 `PopBloom`；重复 Push 同 key 即更新。
- 原版依据：护身符给主人加/撤泛光（[参照](../../../raw/current/scripts/prefabs/amulet.lua#L271-L273)）；月耀激光用低优先级 -1 标记目标（[参照](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L463-L464)）。

```lua
inst:AddComponent("bloomer")
inst.components.bloomer:PushBloom("mymod_buff", "shaders/anim.ksh", 1)
-- 状态结束时：
inst.components.bloomer:PopBloom("mymod_buff")
```

- 调整与验证：priority 数值大者优先；FX 子实体（背包挂件）经 [[reference/components/colouradder]] 同款 `AttachChild` 模式跟随（[背包](../../../raw/current/scripts/prefabs/backpack.lua#L25-L27)）。

## 常见陷阱与限制

- 栈按优先级升序排列、栈顶为最高优先级：同优先级时后推者在上（[插入](../../../raw/current/scripts/components/bloomer.lua#L111-L125)）；期望"先到先得"要用不同 priority。
- 无 bloomer 组件的子实体在 `AttachChild` 时直写 `AnimState`，随后宿主 Pop 到空栈会调 `ClearBloomEffectHandle`（[分支](../../../raw/current/scripts/components/bloomer.lua#L36-L40)），混用两种挂法时注意清理路径不同。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖组件全文与 backpack、amulet、alterguardian_laser、player_common 消费链；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
