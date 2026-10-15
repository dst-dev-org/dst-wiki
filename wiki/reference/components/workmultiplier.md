---
title: "workmultiplier"
created: "2026-10-16"
updated: "2026-10-28"
type: component
summary: "按执行者和动作叠加工作量倍率，区分工具基础量、耐久消费与反弹后的特殊工作量回调。"
aliases: ["人物砍树效率", "玩家工作量加倍", "工作效率Buff", "工作量倍率", "workmultiplier"]
tags: [component, actions, callback, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/workmultiplier.lua", "util/sourcemodifierlist.lua", "actions.lua", "components/workable.lua", "components/tool.lua", "components/worker.lua", "prefabs/player_common.lua", "prefabs/foodbuffs.lua", "prefabs/wolfgang.lua", "components/mightiness.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# workmultiplier

## 快速判断

- 职责与适用：挂在执行者，按动作叠加**工作量**倍率；原版玩家服务端已有，食物工作 Buff 也按需给目标添加。[workmultiplier.lua:3](../../../raw/current/scripts/components/workmultiplier.lua#L3)、[player_common.lua:2821](../../../raw/current/scripts/prefabs/player_common.lua#L2821)、[foodbuffs.lua:29](../../../raw/current/scripts/prefabs/foodbuffs.lua#L29
- 优先使用：人物工作增强用 `AddMultiplier(action, factor, source)`，撤销同源用 RemoveMultiplier；物品基础工作量归 [[reference/components/tool]]，耐久消费归 [[reference/components/efficientuser]]。[actions.lua:1703](../../../raw/current/scripts/actions.lua#L1703
- 重要副作用：不同来源相乘，同来源覆盖；倍率只改变已有动作的量，不授予工作动作、不自动改动画频率或每次耐久消费。特殊回调返回的是最终候选工作量，不是乘数。[workmultiplier.lua:13](../../../raw/current/scripts/components/workmultiplier.lua#L13)、[:31](../../../raw/current/scripts/components/workmultiplier.lua#L31)
- 修改边界：服务端配置；DoToolWork 消费本组件，其他直接 `WorkedBy` 路径不自动读取。反弹、特殊回调与目标修正会继续改量。[actions.lua:1719](../../../raw/current/scripts/actions.lua#L1719)、[workable.lua:123](../../../raw/current/scripts/components/workable.lua#L123

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `GetMultiplier(action)` | Action → number；缺省 1 | 每动作独立列表，不把 MINE 自动映射到副动作。[workmultiplier.lua:9](../../../raw/current/scripts/components/workmultiplier.lua#L9 |
| `AddMultiplier(action, multiplier, source)` | 倍率 number；来源实体或名称 | 不支持 key 参数；同动作同来源只有默认槽。不同来源相乘，source=nil 不加，倍率 nil／1 撤该槽。[workmultiplier.lua:13](../../../raw/current/scripts/components/workmultiplier.lua#L13)、[sourcemodifierlist.lua:64](../../../raw/current/scripts/util/sourcemodifierlist.lua#L64 |
| `RemoveMultiplier(action, source)` | 无返回值 | 删除该动作的整个来源，保留其他来源。[workmultiplier.lua:21](../../../raw/current/scripts/components/workmultiplier.lua#L21 |
| `SetSpecialMultiplierFn(fn?)` | `fn(inst, action, target, tool, numworks, recoil)` | 单槽替换／nil 清除，不是多来源回调列表。[workmultiplier.lua:27](../../../raw/current/scripts/components/workmultiplier.lua#L27 |
| `ResolveSpecialWorkAmount(...)` | 上述回调参数除 inst；返回工作量 | 没回调或返回 nil／false 时用原 numworks；0 会保留。函数自身不做数值／正负校验。[workmultiplier.lua:31](../../../raw/current/scripts/components/workmultiplier.lua#L31 |

## 关键流程

标准 CHOP／MINE／HAMMER／DIG 进入 DoToolWork 后，先验证目标能工作且 action 匹配，并检查限制／拟态；随后依次结算：[actions.lua:1693](../../../raw/current/scripts/actions.lua#L1693)、[:1748](../../../raw/current/scripts/actions.lua#L1748)、[:1767](../../../raw/current/scripts/actions.lua#L1767)、[:1780](../../../raw/current/scripts/actions.lua#L1780)、[:1793](../../../raw/current/scripts/actions.lua#L1793)

1. 基础量依次选 invobject.tool 的 GetEffectiveness、doer.worker 的 GetEffectiveness、默认 1；存在的返回 0 也会选中。基础量乘执行者 `GetMultiplier(workaction)`。[actions.lua:1703](../../../raw/current/scripts/actions.lua#L1703
2. 目标 `ShouldRecoil(doer, invobject, numworks)` 处理反弹／坚硬门禁，可能改量为 0；特殊回调在此**之后**执行，收到 recoil 与当前 numworks。不要无视这两个值强行返回固定正数，否则可能绕过目标原本的门禁。[actions.lua:1719](../../../raw/current/scripts/actions.lua#L1719)、[workable.lua:189](../../../raw/current/scripts/components/workable.lua#L189
3. `WorkedBy_Internal` 再应用目标 workmultiplierfn、扣剩余量、事件及完成回调；反弹不再算第二遍。事件、剩余≤1 的收尾与掉落位置见 [[reference/components/workable#关键接口]]。[actions.lua:1732](../../../raw/current/scripts/actions.lua#L1732)、[workable.lua:130](../../../raw/current/scripts/components/workable.lua#L130

**工作量与耐久是两条链**：只把工作量乘 2，单次动作消费不会自动乘 2；目标通常更少次数完成，但受剩余量收尾等影响，不承诺精确省一半总耐久。若设计要求每次扣次随量增加，另外配置 efficientuser；原版 mightiness 同时对两个组件设置工作效果倍率，不说明二者是一项功能。[mightiness.lua:341](../../../raw/current/scripts/components/mightiness.lua#L341)、[[reference/components/efficientuser#关键接口]]。

## 典型接入

### 所有玩家每次砍树工作量翻倍，保留其他来源与耐久规则

目标模组 `modmain.lua`；以下可独立追加，无需 GLOBAL 代理或新资源。复用玩家原有动作、工具与目标，不新增徒手动作；不替换特殊回调。原版工作食物效果也用实体来源 Add／Remove 配对。[modutil.lua:586](../../../raw/current/scripts/modutil.lua#L586)、[foodbuffs.lua:29](../../../raw/current/scripts/prefabs/foodbuffs.lua#L29

```lua
local source = "my_mod_chop_work"

AddPlayerPostInit(function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.workmultiplier == nil then
        inst:AddComponent("workmultiplier")
    end
    inst.components.workmultiplier:AddMultiplier(GLOBAL.ACTIONS.CHOP, 2, source)
end)
```

`2` 是工作量乘数；其他来源仍乘算，同名来源重复设置是更新。影响所有玩家，新实例／读档初始化时重施，不保存为组件数据。永久加成随实例存在；若将其改成临时效果，失效时调用 `RemoveMultiplier(GLOBAL.ACTIONS.CHOP, "my_mod_chop_work")`。实体来源删除可自动清理，卸装／脱附仍需显式撤销；不移除玩家共享组件。停用后不再添加，不恢复已经完成的工作或已消耗的物品次数。[sourcemodifierlist.lua:84](../../../raw/current/scripts/util/sourcemodifierlist.lua#L84)、[:103](../../../raw/current/scripts/util/sourcemodifierlist.lua#L103)

### 特殊工作量：先保留原版宿主约束

Wolfgang 的特殊回调在未反弹、量非 0 且强壮等条件下概率返回 99999，否则不返回值，Resolve 会保留原量；可能同时推送声音状态。它在服务端角色初始化注册，不是可随意覆盖的公共乘数槽。[wolfgang.lua:181](../../../raw/current/scripts/prefabs/wolfgang.lua#L181)、[:677](../../../raw/current/scripts/prefabs/wolfgang.lua#L677)

普通稳定加成优先 AddMultiplier；确需包装特殊回调时核对旧返回值、调用次数与副作用，不将回调返回值再乘成第二轮倍率。本页不提供强制清除一切目标的通用例子。

## 常见陷阱与限制

- **把 `SetSpecialMultiplierFn` 当“返回倍率”**：返回 2 就把候选量设成 2；要放大应在核验旧逻辑后计算量，保留 recoil／0 门禁。返回 nil／false 才回退，返回 0 不回退。[workmultiplier.lua:31](../../../raw/current/scripts/components/workmultiplier.lua#L31
- **同源第四参数 key 能隔离两个加成**：本接口不接收 key；额外实参不提供独立槽位。用不同实体或模组命名来源区分效果，移除只撤自己的来源。[workmultiplier.lua:13](../../../raw/current/scripts/components/workmultiplier.lua#L13
- **改完倍率，所有“挖矿相关”动作都变快**：本组件没有副动作映射；REMOVELUNARBUILDUP 走独立动作实现，未走此 DoToolWork。[workmultiplier.lua:13](../../../raw/current/scripts/components/workmultiplier.lua#L13)、[actions.lua:6630](../../../raw/current/scripts/actions.lua#L6630
- **删组件来关闭自己的一项 Buff**：会丢失其他来源；本组件没有 OnRemoveFromEntity 的 Reset，也无 OnSave／OnLoad。优先 RemoveMultiplier 成对撤销，存档恢复由效果所有者负责；不要照搬 efficientuser 的移除钩子。[workmultiplier.lua:1](../../../raw/current/scripts/components/workmultiplier.lua#L1)、[foodbuffs.lua:38](../../../raw/current/scripts/prefabs/foodbuffs.lua#L38
- **0／负数是安全禁用开关**：倍率不校验范围；零量可能仍触发目标工作事件。若要禁用工作，应核验动作／目标门禁，而非靠工作量阻止所有副作用。见 [[reference/components/workable#关键接口]]。

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 漂移文件 actions.lua、prefabs/player_common.lua、prefabs/wolfgang.lua 对照 build-26148036 更新报告（R-01、C-10 关联）逐条核验，锚点重定位 13 处、结论修订 0 处（DoToolWork 结算链与四种动作未变；wolfgang 漂移为 OnHitOther 加 `from_doattack` 前置与 `IsRangedWeapon` 判定，属攻击经验获取，不触及 `SetSpecialMultiplierFn` 特殊回调；player_common 漂移仅删虫腹音效接线）；未覆盖：运行验证。

- 源码审阅：2026-10-16，Steam buildid 25643519；覆盖倍率与特殊回调完整实现、来源列表、上述四种标准动作路径、目标后置修正、玩家初始化／原版食物与 Wolfgang／mightiness 模式。未全量审校所有直接工作调用或人物状态图。
- 结构检查：本批元数据、来源、源码行锚点、大小、链接与唯一登记通过；发布代码通过 Full Moon 3.0.0／Lua 5.1 语法、原 token 字节与精确往返检查，章节读取完整。
- 游戏运行：未运行；工作次数／耗时、反弹与临界剩余量、多来源撤销、读档和其他模组兼容待实测。

相关：[[reference/components/worker#快速判断|自身基础量]]；[[reference/components/tool#关键接口|工具基础量]]；[[reference/components/efficientuser#关键接口|耐久与治疗消费者]]。
