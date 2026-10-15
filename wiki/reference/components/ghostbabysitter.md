---
title: "ghostbabysitter"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: 记录照看中幽灵名单的组件（未接线）：可增删幽灵、随存档保存 GUID、加载后向幽灵推 set_babysitter 事件；当前版本无任何挂载者。
aliases: ["ghostbabysitter", "幽灵照看者"]
tags: [component]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/ghostbabysitter.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# ghostbabysitter

## 快速判断

- 职责与适用：在一个实体（按设计意图是墓碑类建筑）上维护 `babysitting` 幽灵集合：`AddGhost`/`RemoveGhost` 增删并启停更新，存档保存幽灵 GUID，加载恢复时向各幽灵推 `set_babysitter` 事件。[LoadPostPass](../../../raw/current/scripts/components/ghostbabysitter.lua#L53-L62)
- 优先使用：**当前快照内无任何消费者**——全库 grep 仅组件自身引用，无 `AddComponent("ghostbabysitter")`、无外部 `updatefn` 赋值者、无 `set_babysitter` 监听者；不能作为可靠接入点。
- 重要副作用：`inactive` 字段是同步字段，变化时增删实体 `inactive` 标签（netvar 回调表注册）；`OnRemoveFromEntity` 清 `inactive` 与 `activatable_forceright` 标签。[netvar 回调](../../../raw/current/scripts/components/ghostbabysitter.lua#L17)
- 修改边界：与温蒂/阿比盖尔的"看护人"机制（实体字段 `ghost_babysitter`、事件 `babysitter_set`）是两套互不相通的代码，不要混淆。[对照](../../../raw/current/scripts/prefabs/wendy.lua#L409)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `AddGhost(ghost)` | ghost: entity | `babysitting[ghost] = true` 并 `StartUpdatingComponent`；重复添加幂等。[实现](../../../raw/current/scripts/components/ghostbabysitter.lua#L31-L34) |
| `RemoveGhost(ghost)` | ghost: entity | 移除记录；集合清空时停止更新。[实现](../../../raw/current/scripts/components/ghostbabysitter.lua#L36-L41) |
| `IsBabysittingGhost(ghost)` | `-> boolean 或 nil` | 查询是否在照看名单。[实现](../../../raw/current/scripts/components/ghostbabysitter.lua#L25-L29) |
| `updatefn` | `function(inst, self, dt)` 或 nil | 每帧回调钩子，组件不赋默认值；当前脚本库无任何赋值者，未赋值时 OnUpdate 为空操作。[消费点](../../../raw/current/scripts/components/ghostbabysitter.lua#L47-L51) |
| `inactive` | boolean（同步） | 构造默认 true；netvar 回调按值增删 `inactive` 标签。[回调](../../../raw/current/scripts/components/ghostbabysitter.lua#L1-L3) |
| `OnSave` / `LoadPostPass` | 存读钩子 | OnSave 输出幽灵 GUID 列表并声明引用实体；LoadPostPass 对恢复的幽灵推 `set_babysitter`（携带本实体）。OnLoad 为空体。[存档](../../../raw/current/scripts/components/ghostbabysitter.lua#L64-L75) |

## 典型接入

### 设计意图还原（未接线，勿当现成能力）

- 前提与选择：如需启用，挂载方需自行管理三件事：赋 `updatefn` 才有每帧行为、在幽灵侧监听 `set_babysitter` 才能接上加载恢复、并自行决定 `inactive`/`forcerightclickaction` 的语义（后者仅由 netvar 回调控制标签，无驱动者）。[事件推送](../../../raw/current/scripts/components/ghostbabysitter.lua#L58)
- 生效与撤销：名单随宿主存档持久化；幽灵引用按 GUID 经 `newents` 恢复，幽灵不存在时静默跳过。

```lua
-- 复刻设计意图（当前原版未接线；自行验证）
inst:AddComponent("ghostbabysitter")
inst.components.ghostbabysitter.updatefn = function(inst, self, dt) end
inst.components.ghostbabysitter:AddGhost(ghost_ent)
```

- 调整与验证：`LoadPostPass` 只推事件不直接写名单，恢复后的 `babysitting` 表内容依赖宿主在事件处理中回填。

## 常见陷阱与限制

- **以为组件驱动幽灵 AI：** 幽灵行为里的看护逻辑走的是另一套：`abigailbrain.lua:161` 读实体字段 `inst.ghost_babysitter`，`follower.lua:78` 据此跳过传送；这些与本组件零交集。[AI 侧字段](../../../raw/current/scripts/brains/abigailbrain.lua#L161)
- **`OnLoad` 是空体：** 存档数据在 `LoadPostPass` 才消费，不要在 OnLoad 里期待还原名单；GUID 恢复依赖引擎 LoadPostPass 时序。[空 OnLoad](../../../raw/current/scripts/components/ghostbabysitter.lua#L77-L80)
- **`activatable_forceright` 标签无来源：** `forcerightclickaction` 有 netvar 回调函数但未注册进回调表，也没有写入者；标签只会在 `OnRemoveFromEntity` 时被清除。[清理](../../../raw/current/scripts/components/ghostbabysitter.lua#L20-L23)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（82 行）；全库 grep（大小写不敏感）确认 `ghostbabysitter`、`set_babysitter`、`updatefn` 赋值均无外部消费者；核对温蒂/阿比盖尔 `ghost_babysitter` 字段机制以划清边界（`wendy.lua:409`、`abigailbrain.lua:161`、`follower.lua:78`）。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。
