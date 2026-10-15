---
title: "worker"
created: "2026-10-16"
updated: "2026-10-28"
type: component
summary: "给执行者配置自身工作动作与基础工作量；已有工具优先，徒手交互还需动作与状态支持。"
aliases: ["自身工作能力", "海狸人砍树效率", "无工具工作量", "worker"]
tags: [component, actions, creature, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/worker.lua", "actions.lua", "prefabs/woodie.lua", "components/tool.lua", "modutil.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# worker

## 快速判断

- 职责与适用：挂在执行者，提供动作对应的**自身基础工作量**；原版用于 Woodie 海狸形态，不负责工作目标、耐久或动作动画。[worker.lua:1](../../../raw/current/scripts/components/worker.lua#L1)、[woodie.lua:833](../../../raw/current/scripts/prefabs/woodie.lua#L833
- 优先使用：已有自身工作宿主用 `SetAction(action, amount)`；修改手持工具用 [[reference/components/tool]]，人物倍率用 [[reference/components/workmultiplier]]。[actions.lua:1703](../../../raw/current/scripts/actions.lua#L1703
- 重要副作用：DoToolWork 优先读 invobject.tool，之后才用 doer.worker，最后默认 1；两种基础量不相加，工具返回 0 在 Lua 中也不会回退到 worker。[actions.lua:1703](../../../raw/current/scripts/actions.lua#L1703
- 修改边界：服务端配置；单加 worker 只建 actions 表，不加工具标签或提供完整徒手交互。原版海狸还配置动作选择、形态与其他系统。[worker.lua:1](../../../raw/current/scripts/components/worker.lua#L1)、[woodie.lua:367](../../../raw/current/scripts/prefabs/woodie.lua#L367)、[:1296](../../../raw/current/scripts/prefabs/woodie.lua#L1296)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `actions` | Action → 数值的实例表，初始空 | 没有来源／key 叠加；SetAction 覆盖该动作。[worker.lua:1](../../../raw/current/scripts/components/worker.lua#L1 |
| `SetAction(action, effectiveness?)` | 默认 1；无返回值 | nil／false 取默认，0 会保留；不校验 TOOLACTIONS、不加标签。不是删除动作接口。[worker.lua:10](../../../raw/current/scripts/components/worker.lua#L10 |
| `GetEffectiveness(action)` | 未登记返回 0 | 消费者仍须决定何时读本组件；不是对目标执行一次工作。[worker.lua:6](../../../raw/current/scripts/components/worker.lua#L6 |
| `CanDoAction(action)` | boolean | 仅判断该键非 nil，登记 0 仍为 true；标准 DoToolWork 不先调用本方法。[worker.lua:14](../../../raw/current/scripts/components/worker.lua#L14)、[actions.lua:1703](../../../raw/current/scripts/actions.lua#L1703 |

量的单位是一次工作结算的基础量，不是秒或“点击速度”；完整倍率、反弹、目标修正与完成路径见 [[reference/components/workmultiplier#关键流程]]。

## 典型接入

### 将 Woodie 海狸形态的自身砍树工作量乘以 1.5

目标模组 `modmain.lua`，以下可独立追加，无需 GLOBAL 代理或新资源；**复用原版海狸完整宿主**。不在人物普通形态强挂 worker，不改技能树的基础配置；包装读取方法以保留原版后来 SetAction 写入的数值。

`AddComponentPostInit` 在组件构造后、AddComponent 返回前调用；Woodie 变海狸时动态添加 worker、再写 CHOP／MINE 等动作，离开时移除。故不能仅在普通角色 PrefabPostInit 改一次，也不能在组件 PostInit 只 SetAction 后等待原版不再覆盖。[modutil.lua:567](../../../raw/current/scripts/modutil.lua#L567)、[entityscript.lua:655](../../../raw/current/scripts/entityscript.lua#L655)、[woodie.lua:841](../../../raw/current/scripts/prefabs/woodie.lua#L841)、[:862](../../../raw/current/scripts/prefabs/woodie.lua#L862)

```lua
AddComponentPostInit("worker", function(self, inst)
    if not GLOBAL.TheWorld.ismastersim or inst.prefab ~= "woodie" then
        return
    end
    local old_GetEffectiveness = self.GetEffectiveness
    self.GetEffectiveness = function(component, action)
        local amount = old_GetEffectiveness(component, action)
        return action == GLOBAL.ACTIONS.CHOP and amount * 1.5 or amount
    end
end)
```

仅改变 Woodie 的 worker CHOP 读取：原基础 4 且无技能修正时变 6，已有技能修正仍保留；MINE／DIG／HAMMER 不变。原版动作若带有 tool invobject，仍优先用工具。其他人物／普通人形不受影响，不承诺总耗时乘 1.5。每次重新添加 worker 自动装入本包装，原版形态退出删除旧组件；停用模组后不再包装。读档恢复海狸由原版 OnPreLoad 调 onbecamebeaver 处理。[woodie.lua:847](../../../raw/current/scripts/prefabs/woodie.lua#L847)、[:1608](../../../raw/current/scripts/prefabs/woodie.lua#L1608)

### 新增无工具工作者时先核对宿主

原版 onbecamebeaver 同时调用 SetWereWorker、SetWereActions、SetWereFighter 等，并切换形态；SetWereActions 配置客户端动作选择回调。worker 的小表不能替代这一链路。[woodie.lua:379](../../../raw/current/scripts/prefabs/woodie.lua#L379)、[:1296](../../../raw/current/scripts/prefabs/woodie.lua#L1296)

本页不提供“给任意角色加组件就能徒手砍树”的 demo。新人物或生物需继续核验其动作提供／选择、执行状态、匹配目标、网络和清理；目标工作接口在 [[reference/components/workable]]，不复制原版海狸的完整变身实现。

## 常见陷阱与限制

- **把 worker 当工具增强组件**：有 tool invobject 时优先读取 tool；通常人物工作加成选 workmultiplier，避免把基础来源混为两次加成。[actions.lua:1703](../../../raw/current/scripts/actions.lua#L1703
- **GetEffectiveness 未登记为 0，所以会自动用默认 1**：组件存在时返回的 0 是真值，DoToolWork 会使用 0；不能靠数值 0 表示动作未提供或纯查询，工作目标仍可能收到事件。目标结算见 [[reference/components/workable#关键接口]]。
- **SetAction(nil 值) 删除能力**：第二参数 nil 实际写 1，CanDoAction 看键非 nil；此组件没有 RemoveAction、来源倍率或标签同步接口。[worker.lua:10](../../../raw/current/scripts/components/worker.lua#L10
- **自身 actions 会保存**：组件没有保存／恢复钩子，原版由形态初始化重建。直接改一次现存组件可能在下次变身／读档丢失；上例在每次构造时包装，仍需游戏回归。[worker.lua:1](../../../raw/current/scripts/components/worker.lua#L1)、[woodie.lua:1608](../../../raw/current/scripts/prefabs/woodie.lua#L1608

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 漂移文件 actions.lua、prefabs/woodie.lua、entityscript.lua 对照 build-26148036 更新报告（R-01）逐条核验，锚点重定位 8 处、结论修订 0 处（DoToolWork 读取顺序未变；woodie 漂移为 MooseFighting 受击回调改经 `OnAttacked_MooseFighting`＋`IsEquipmentOnAttackedOrBlocked` 过滤，不触及 worker／海狸变身链；entityscript 漂移为 GetWet 系与距离函数，AddComponentPostInit 位置未变）；未覆盖：运行验证。

- 源码审阅：2026-10-16，Steam buildid 25643519；覆盖组件完整实现、DoToolWork 基础量选取、组件 PostInit 时序、海狸创建／移除与读档恢复，以及本例包装位置。未全量审阅海狸动作状态图或提供新宿主工程。
- 结构检查：本批元数据、来源、源码行锚点、大小、链接与唯一登记通过；发布代码通过 Full Moon 3.0.0／Lua 5.1 语法、原 token 字节与精确往返检查，章节读取完整。
- 游戏运行：未运行；变身往返、技能修正、读档和其他模组包装叠加待实测。

相关：[[reference/components/tool#关键接口|工具基础量]]；[[reference/components/workmultiplier#关键流程|工作量结算]]；[[reference/components/efficientuser#关键接口|消耗倍率]]。
