---
title: "ghostgestalter"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "幽灵显形变形目标端：用激活标签把结构体暴露给 MUTATE 动作，DoMutate 委托宿主回调完成幽灵与显形者互变。"
aliases: ["幽灵变形器", "ghostgestalter", "月晷幽灵变形"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/ghostgestalter.lua", "componentactions.lua", "actions.lua", "prefabs/moondial.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# ghostgestalter

## 快速判断

- 职责与适用：挂在可"变形幽灵"的结构体上（原版宿主为月晷 moondial）；四个布尔字段经 watcher 同步成激活标签，`DoMutate` 把执行权交给宿主的 `domutatefn`（[组件全文](../../../raw/current/scripts/components/ghostgestalter.lua#L17-L47)）。
- 优先使用：宿主设置 `forcerightclickaction` 与 `domutatefn`（[月晷](../../../raw/current/scripts/prefabs/moondial.lua#L204-L206)）；动作候选要求执行者是温蒂且技能 `wendy_lunar_3` 已激活（[候选](../../../raw/current/scripts/componentactions.lua#L624-L627)）。
- 重要副作用：`activatable_forceright` 标签同时影响通用候选兜底——带此标签的实体即使右键不可用也能进候选（[兜底](../../../raw/current/scripts/componentactions.lua#L148)）。
- 修改边界：`OnRemoveFromEntity` 摘除四个标签；无存读钩子，变形状态由宿主月晷自行持久化（[摘标签](../../../raw/current/scripts/components/ghostgestalter.lua#L36-L41)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `DoMutate(doer)` | `doer: entity -> success, reason` | 有 `domutatefn` 时返回其 `(inst, doer)` 结果，否则返回 nil。[实现](../../../raw/current/scripts/components/ghostgestalter.lua#L43-L47) |
| `domutatefn` | `fn(inst, doer) -> success, reason` | 宿主回调；月晷内拒绝洞穴（"CAVE"）、未召唤幽灵（"NOGHOST"）、非夜晚（"NOTNIGHT"）等（[月晷回调](../../../raw/current/scripts/prefabs/moondial.lua#L116-L135)）。 |
| `forcerightclickaction` | `bool（watcher）` | 同步 `activatable_forceright` 标签；是原版候选唯一消费的标签。[watcher](../../../raw/current/scripts/components/ghostgestalter.lua#L9-L11) |
| `OnActivate` | 构造第二参数存入 | 仅存储；全库未发现读取者。[构造](../../../raw/current/scripts/components/ghostgestalter.lua#L17-L19) |

## 典型接入

### 让结构体接受温蒂的幽灵变形（原版参照）

- 前提与选择：主机端加组件并设 `domutatefn`；候选由组件名自动注册，无需额外标签申报（[候选](../../../raw/current/scripts/componentactions.lua#L624-L627)）。
- 生效与撤销：MUTATE 动作函数直接透传 `DoMutate(act.doer)` 的返回（[执行](../../../raw/current/scripts/actions.lua#L6253-L6262)）；成功/失败原因由 `domutatefn` 决定，动作侧不加工。

```lua
AddPrefabPostInit("my_structure", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return inst
    end
    inst:AddComponent("ghostgestalter")
    inst.components.ghostgestalter.domutatefn = function(inst, doer)
        -- 自定义条件检查，返回 true 或 false, "原因码"
        return true
    end
end)
```

- 调整与验证：候选限定 `wendy_lunar_3` 技能已激活，其他角色看不到动作；需要放开角色限制时需改候选函数，组件本身不校验执行者。

## 常见陷阱与限制

- **三个标签无原版 MUTATE 消费者**：`standingaction`/`quickaction`/`forcenopickupaction` 同步的 `standingactivation`/`quickactivation`/`activatable_forcenopickup` 标签中，前两个只被 ACTIONS.ACTIVATE 的状态选择消费（[SG](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L403-L404)），`activatable_forcenopickup` 全库未发现消费者；本组件的 MUTATE 路径只认 `activatable_forceright`。
- **组件不承载变形逻辑**：成功条件、月相限制、幽灵互变全部在宿主 `domutatefn`（[月晷回调](../../../raw/current/scripts/prefabs/moondial.lua#L116-L135)）；复用组件不会自动获得月晷行为。
- **OnActivate 是死字段**：构造函数接收回调但无任何读取者，传 nil 安全，别依赖它被调用。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、MUTATE 候选与执行、月晷宿主挂接与 domutatefn 条件、四个标签的全库消费者检索。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
