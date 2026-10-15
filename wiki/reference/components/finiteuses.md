---
title: finiteuses
created: "2026-10-11"
updated: "2026-10-28"
type: component
summary: "按动作或显式调用消耗次数，配置耗尽回调并保存剩余次数。"
aliases: ["有限使用次数", "工具耐久", "按次消耗", "finiteuses"]
tags: [component, item, equipment, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/finiteuses.lua", "class.lua", "entityscript.lua", "bufferedaction.lua", "actions.lua", "components/weapon.lua", "prefabs/axe.lua", "prefabs/oar.lua", "prefabs/wateringcan.lua", "modutil.lua", "mainfunctions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# finiteuses

## 快速判断

- 职责与适用：物品按次数消耗；原版斧头在服务端添加并配置动作消耗。[prefabs/axe.lua:94](../../../raw/current/scripts/prefabs/axe.lua#L94
- 优先使用：初始化分别 `SetMaxUses(n)`、`SetUses(n)`；配置动作 `SetConsumption(action, n)`，手动扣次 `Use(n)`。[components/finiteuses.lua:27](../../../raw/current/scripts/components/finiteuses.lua#L27
- 重要副作用：`SetUses` 发物品事件 `percentusedchange`，正到非正时加 `usesdepleted` 并执行物品实体回调。[components/finiteuses.lua:64](../../../raw/current/scripts/components/finiteuses.lua#L64
- 修改边界：修改服务端组件；`SetMaxUses` 只改总量；直接 `Use` 不走动作修正链。下文记录时序、存档和攻击差异。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `total` / `current` | `number`；默认均为 `100` | 写属性会更新可选修理组件状态；[finiteuses.lua:1](../../../raw/current/scripts/components/finiteuses.lua#L1)、[class.lua:36](../../../raw/current/scripts/class.lua#L36 |
| `SetMaxUses(val)` | `val: number`；无返回值 | 不重置 `current`，不发百分比事件；[finiteuses.lua:60](../../../raw/current/scripts/components/finiteuses.lua#L60 |
| `SetUses(val)` / `Use(num?)` | 数值；`num` 省略／nil 按 `1` | 由 `SetUses` 处理耗尽；[finiteuses.lua:64](../../../raw/current/scripts/components/finiteuses.lua#L64 |
| `GetUses()` / `GetPercent()` | 返回 `number` | 比例为 `current / total`，无零分母保护；[finiteuses.lua:81](../../../raw/current/scripts/components/finiteuses.lua#L81)、[:116](../../../raw/current/scripts/components/finiteuses.lua#L116) |
| `SetPercent(amount)` / `Repair(repairvalue)` | 数值；无返回值 | 百分比不截断；修理相加后仅上限限制为 total；[finiteuses.lua:120](../../../raw/current/scripts/components/finiteuses.lua#L120 |
| `SetConsumption(action, uses?)` | `Action` → `number`；nil 删除映射 | MINE 同设 REMOVELUNARBUILDUP，TERRAFORM 同设 TERRAFORM_REMOVE；[finiteuses.lua:27](../../../raw/current/scripts/components/finiteuses.lua#L27 |
| `ClearAllConsumptions()` | 无返回值 | 换空映射表，保留修正回调；[finiteuses.lua:36](../../../raw/current/scripts/components/finiteuses.lua#L36 |
| `SetModifyUseConsumption(fn?)` | `fun(uses:number, action:Action, doer:ent\|nil, target:ent\|nil, inst:ent):number` | nil 清除；参数依次是消耗、动作、执行者、目标、物品；[finiteuses.lua:97](../../../raw/current/scripts/components/finiteuses.lua#L97 |
| `SetOnFinished(fn?)` | `fun(inst:ent)` | nil 清除；回调返回值忽略；[finiteuses.lua:64](../../../raw/current/scripts/components/finiteuses.lua#L64)、[:124](../../../raw/current/scripts/components/finiteuses.lua#L124) |
| `SetDoesNotStartFull(enabled?)` | 布尔推荐域；nil 清除 | 即使满次数也保存当前量；[finiteuses.lua:44](../../../raw/current/scripts/components/finiteuses.lua#L44 |
| `SetIgnoreCombatDurabilityLoss(value?)` | 布尔推荐域；nil 清除 | getter 原样返回 `boolean\|nil`；影响 Weapon 攻击扣次路径；[weapon.lua:111](../../../raw/current/scripts/components/weapon.lua#L111 |

这些类型描述原版调用约定；赋值型 setter 并不执行 Lua 类型检查，不代表任意模组写入都安全。

## 典型接入

### 修改已有斧头总次数，保留初始比例与原版耗尽行为

- 前提与选择：已有启用的服务端模组，将以下完整接入代码放入 `modmain.lua`；使用现有 `axe` 的组件和资源，不新增 Prefab。
- 生效与撤销：影响此模组下新实例的总次数，保留当前初始比例；存档载入后原版 `OnLoad` 会恢复保存的**绝对次数**，此例不迁移旧存档比例。停用后新实例恢复原版定义。
- 原版依据：斧头先设置总次数、当前次数和耗尽回调，再注册 CHOP 消耗；Prefab postinit 在工厂之后执行。[axe.lua:112](../../../raw/current/scripts/prefabs/axe.lua#L112)、[modutil.lua:592](../../../raw/current/scripts/modutil.lua#L592)、[mainfunctions.lua:387](../../../raw/current/scripts/mainfunctions.lua#L387

```lua
AddPrefabPostInit("axe", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local finiteuses = inst.components.finiteuses
    if finiteuses == nil then
        return
    end
    local oldpercent = finiteuses:GetPercent()
    finiteuses:SetMaxUses(200)
    finiteuses:SetUses(200 * oldpercent)
end)
```

- 调整与验证：`200` 为总次数；没有覆盖原版 CHOP 消耗或 onfinished。已审阅接入顺序，未在游戏运行；不保证与其他同时修改斧头的模组兼容。

### 原版不同消费模式

动作工具：[axe.lua:112](../../../raw/current/scripts/prefabs/axe.lua#L112 配 CHOP；回调修正：[oar.lua:57](../../../raw/current/scripts/prefabs/oar.lua#L57)、[:137](../../../raw/current/scripts/prefabs/oar.lua#L137) 修正划船耗次；非满初态：[wateringcan.lua:146](../../../raw/current/scripts/prefabs/wateringcan.lua#L146)。只选不同模式，无完整引用清单。

## 常见陷阱与限制

- **只改上限**：`SetMaxUses` 不重置当前值；新物品要满次数时同时 `SetUses`。直接写 current 不发百分比事件或耗尽回调，优先用方法。[finiteuses.lua:60](../../../raw/current/scripts/components/finiteuses.lua#L60
- **耗尽事件时序**：先发百分比事件，再把非正次数归零；传入负数时事件 payload 可能为负。onfinished 只在正到非正转换触发，组件不自动删实体；斧头通过 `SetOnFinished(inst.Remove)` 实现删除。[finiteuses.lua:64](../../../raw/current/scripts/components/finiteuses.lua#L64)、[axe.lua:115](../../../raw/current/scripts/prefabs/axe.lua#L115
- **攻击没有经过动作回调**：`OnUsedAsItem` 有消耗映射才先乘 efficientuser 倍率，再调用修正函数；武器攻击在自己的 OnAttack 先算 ATTACK 倍率，再直接 `Use`，不会调用动作消费修正函数。修改该回调不能统一修改全部手动／攻击扣次；消费者区别见 [[reference/components/efficientuser#倍率实际作用在哪里]]。[finiteuses.lua:101](../../../raw/current/scripts/components/finiteuses.lua#L101)、[weapon.lua:111](../../../raw/current/scripts/components/weapon.lua#L111
- **非满初始状态的存档**：OnSave 默认满次数时无返回值；初始为空但可补满的物品需考虑 `SetDoesNotStartFull(true)`，原版 wateringcan 如此配置。[finiteuses.lua:44](../../../raw/current/scripts/components/finiteuses.lua#L44)、[wateringcan.lua:146](../../../raw/current/scripts/prefabs/wateringcan.lua#L146
- **范围由调用方负责**：SetPercent 不夹在 0～1，SetUses 不限制上限；确保 total 为正并按实际场景约束输入。[finiteuses.lua:64](../../../raw/current/scripts/components/finiteuses.lua#L64)、[:116](../../../raw/current/scripts/components/finiteuses.lua#L116)

## 生命周期

- 真实钩子：OnSave 返回 `{uses=current}` 或不返回；OnLoad 需非空表，可选 uses 存在才应用；OnRemoveFromEntity 仅移标签。宿主按方法存在性分派，不意味着此组件实现 OnUpdate 等公共编辑器钩子。[entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670)、[:1928](../../../raw/current/scripts/entityscript.lua#L1928)、[:2005](../../../raw/current/scripts/entityscript.lua#L2005)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；相关调用与示例仍适用，本轮仅重定位证据（锚点 3 处），未运行游戏。

- 增量审阅：2026-10-16，同 build；仅复查动作消费与 Weapon ATTACK 倍率的分工，补 efficientuser 入链；本次实质增修同时将原版模式归典型接入、生命周期标题归一，旧依据与其验证范围保留。
- 源码审阅：2026-10-11，Steam buildid 25643519；覆盖此页接口、耗尽／动作／存档路径与斧头 postinit 场景。19 个实际方法的接口约定保存在固定 fork [83 条 reviews](https://github.com/YingSP/lsp-dst/blob/1db05561ea6ea271093ea780b60d49e7499baf40/reviews/components/finiteuses.lua.json)；类型库其他公共模型与引擎内部未因此核验。
- LuaCATS：[固定版本声明](https://github.com/YingSP/lsp-dst/blob/1db05561ea6ea271093ea780b60d49e7499baf40/scripts/components/members/finiteuses.lua) 与页面使用同一 reviews，已发布并固定引用；查询本页不要求克隆类型库。
- 结构检查：2026-10-11，元数据、本地链接和分类大小检查通过；LuaLS 声明／正例无诊断，三处故意错误命中预期。只验证各自范围。
- 游戏运行：未运行，案例仅源码审阅；引擎网络实际传播、多人显示和存档回归需游戏验证。

相关：[[reference/components/tool|工作动作与效率]]；[[reference/components/weapon|攻击扣次]]；[[reference/components/fueled|时间耐久]]；[[reference/components/rechargeable|自动恢复]]；[[reference/components/stackable|实体件数]]。
