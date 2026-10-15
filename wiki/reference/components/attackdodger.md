---
title: "attackdodger"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "在 combat 受击流程中按回调或冷却资格闪避，将本次普通与特殊伤害清空。"
aliases: ["attackdodger", "受击闪避", "周期闪避攻击", "伍迪鹅形态闪避", "闪避冷却"]
tags: [component, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/attackdodger.lua", "components/combat.lua", "prefabs/woodie.lua", "prefabs/skilltree_woodie.lua", "prefabs/mermking_gear_buffs.lua", "prefabs/merm.lua", "stategraphs/SGmerm.lua", "tuning.lua", "modutil.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# attackdodger

## 快速判断

- 职责与适用：服务端受击组件，由 [[reference/components/combat]] 的 `GetAttacked_Internal`（`GetAttacked` 转调的内部路径）主动查询；目标有 health、damage 非 nil 且未重定向时，成功将 `damage = 0`、`spdamage = nil`；攻击者"穿在受害者身上"（宿主 inventory 的 `IsItemEquipped`，slurper/bat_boss 类）时跳过闪避。[消费点](../../../raw/current/scripts/components/combat.lua#L615-L621)。
- 优先使用：原版伍迪鹅形态依靠冷却实现间歇闪避；已有宿主宜调原配置，不另挂一份或常驻人物本体。[原版创建](../../../raw/current/scripts/prefabs/woodie.lua#L953)。
- 重要副作用：`Dodge` 先调用宿主回调再开启任务冷却；组件自己不发事件、不播动画、不移动实体。特效或 SG 反应来自宿主回调。[实现](../../../raw/current/scripts/components/attackdodger.lua#L37)。
- 修改边界：条件回调和可用冷却按“或”放行，不能当作“条件且冷却”的通用门禁；移除会取消当前保存的冷却任务。[CanDodge](../../../raw/current/scripts/components/attackdodger.lua#L30)、[移除](../../../raw/current/scripts/components/attackdodger.lua#L49)。

（26148036 核验：combat 受击主体拆入 `GetAttacked_Internal`，新增 `isequipped` 前置——攻击者被穿在受害者身上时无视闪避、护甲与鞍防御；查询点与逻辑本体未变——[combat.lua:607-622](../../../raw/current/scripts/components/combat.lua#L607-L622)）

## 关键接口

| 接口／字段 | 默认／约定 | 条件、作用与证据 |
|---|---|---|
| `SetCanDodgeFn(fn)` | 默认 nil；`fn(inst, attacker)` | `CanDodge` 每次先调用它；按 Lua 真值使用结果，不强制 boolean。[setter](../../../raw/current/scripts/components/attackdodger.lua#L22)、[查询](../../../raw/current/scripts/components/attackdodger.lua#L30) |
| `SetCooldownTime(n)` | 默认 nil；原版为秒数 | 只赋值，不取消／重排正在运行的任务，也不改变 `oncooldown`。非 nil 时 `Dodge` 排任务；数值 0 在 Lua 中仍是真值。[setter](../../../raw/current/scripts/components/attackdodger.lua#L26)、[排任务](../../../raw/current/scripts/components/attackdodger.lua#L42) |
| `CanDodge(attacker)` | 返回真值／false／nil | `candodge` 为回调结果；`notincooldown` 为配置冷却且 `oncooldown == false`。当前表达式的放行结果等价于 `candodge or notincooldown`，并非两者都要通过。[完整实现](../../../raw/current/scripts/components/attackdodger.lua#L30) |
| `SetOnDodgeFn(fn)` | 默认 nil；`fn(inst, attacker)` | `Dodge` 中调用，返回值忽略；回调时本次冷却尚未开启。[定义](../../../raw/current/scripts/components/attackdodger.lua#L18)、[执行顺序](../../../raw/current/scripts/components/attackdodger.lua#L37) |
| `Dodge(attacker)` | 无成功返回约定 | 不调用 CanDodge，直接执行回调并按配置启动冷却；到点清任务引用并设 `oncooldown = false`。[执行](../../../raw/current/scripts/components/attackdodger.lua#L37)、[到点](../../../raw/current/scripts/components/attackdodger.lua#L1) |
| `OnRemoveFromEntity()` | 取消保存的 `cooldowntask` | 不重置其他字段；组件无存取档实现。正常通过实体 `RemoveComponent` 触发。[组件](../../../raw/current/scripts/components/attackdodger.lua#L49)、[实体移除](../../../raw/current/scripts/entityscript.lua#L670) |

### 冷却与条件如何组合

默认两项均未配置时不闪避；只配冷却时初始可闪避，随后等任务结束；只配条件时由条件结果决定。两项都有时，条件 false 不能否决已经可用的冷却，条件 true 能在冷却中继续放行。需要严格 AND 门禁时应另行设计，不直接靠两个 setter 叠加。[依据](../../../raw/current/scripts/components/attackdodger.lua#L30)。

## 典型接入

### 延长伍迪鹅形态已有技能的闪避冷却

独立模组的完整 `modmain.lua`；普通生存模式服务端，无新增资源。原版 `woodie` 变鹅且激活 `woodie_curse_goose_3` 时才创建组件并设置冷却；读档技能激活路径也会补建。使用组件 post-init 包装后续 setter，避免在人物 prefab post-init 时组件尚不存在，或设置值被原版紧接着覆盖。[变形路径](../../../raw/current/scripts/prefabs/woodie.lua#L953)、[读档路径](../../../raw/current/scripts/prefabs/skilltree_woodie.lua#L329)、[注入注册](../../../raw/current/scripts/modutil.lua#L567)、[构造后调用顺序](../../../raw/current/scripts/entityscript.lua#L655)。

```lua
AddComponentPostInit("attackdodger", function(component, inst)
    if not GLOBAL.TheWorld.ismastersim or inst.prefab ~= "woodie" then
        return
    end
    local old_setcooldown = component.SetCooldownTime
    component.SetCooldownTime = function(self, seconds)
        old_setcooldown(self, seconds ~= nil and seconds * 2 or nil)
    end
end)
```

倍率 `2` 使原版 5 秒配置成为 10 秒；只影响之后为该伍迪组件调用 setter 的数值，不缩放已经启动的剩余任务。不会授予技能，也不为其他形态创建组件；原版保留 `OnDodgeAttack` 特效回调，切换战斗形态时先移除旧组件并取消其任务。[原值](../../../raw/current/scripts/tuning.lua#L7121)、[特效](../../../raw/current/scripts/prefabs/woodie.lua#L916)、[切换清理](../../../raw/current/scripts/prefabs/woodie.lua#L923)。停用该段并重新加载世界恢复原版配置；不应手动移除伍迪的技能组件作为撤销。示例未进游戏运行。

### 原版鱼人王冠 buff 的事件协作

`mermking_buff_crown` 附着非玩家且有 `mermguard` 标签的实体时添加组件，配置冷却并在闪避回调中发送 `attackdodged`；buff 脱离时移除组件。原版 `mermguard` 的工厂指定 `SGmerm`，其监听事件进入 `dodge_attack`；动画／位移由状态图实现，组件本身没有 SG 动作接口。[buff 挂载及撤销](../../../raw/current/scripts/prefabs/mermking_gear_buffs.lua#L66)、[实际宿主注册](../../../raw/current/scripts/prefabs/merm.lua#L1649)、[宿主 SG](../../../raw/current/scripts/prefabs/merm.lua#L1273)、[SG 事件](../../../raw/current/scripts/stategraphs/SGmerm.lua#L96)、[状态](../../../raw/current/scripts/stategraphs/SGmerm.lua#L710)。

## 常见陷阱与限制

- 直接调 `Dodge` 不会检查资格，也不会替调用者清空某次伤害；实际清伤在 combat 中。它先于护甲处理，将本次特殊伤害表整体清空，但不覆盖绕过 GetAttacked 的生命扣减。[消费点](../../../raw/current/scripts/components/combat.lua#L615-L621)。
- 闪避后 combat 仍继续护甲、声音和事件流程；普通无额外伤害来源时可走 `blocked`，不能据此假设受击完全无副作用。重定向目标非 nil 时跳过闪避（包括原版招架路径），攻击者被 `IsItemEquipped` 穿在受害者身上时同样跳过。[后续处理](../../../raw/current/scripts/components/combat.lua#L630)、[blocked](../../../raw/current/scripts/components/combat.lua#L730)。
- 冷却中强行多次 Dodge 会覆盖保存的任务引用而不取消旧任务；旧任务可能提前解除冷却，移除也只取消当前引用。保留“先 CanDodge 再 Dodge”的消费模式，并留意条件 true 可绕过冷却的组合。[任务覆盖](../../../raw/current/scripts/components/attackdodger.lua#L37)、[回调解锁](../../../raw/current/scripts/components/attackdodger.lua#L1)。
- 冷却不存档，卸载／重建会回到初始可用状态；原版伍迪形态切换与 buff 移除也会删除组件。不要把该字段当永久能力状态。[构造默认值](../../../raw/current/scripts/components/attackdodger.lua#L8)、[原版清理](../../../raw/current/scripts/prefabs/woodie.lua#L929)。

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件对照 build-26148036 更新报告逐条核验，锚点重定位 12 处、结论修订 2 处（GetAttacked_Internal 消费点与穿身豁免，见正文"26148036 核验"标注）；未覆盖：运行验证。

- 2026-10-21，`steam-buildid:25643519`：完整审阅组件 1–57 行，覆盖全部 setter、真值组合、回调先后、冷却任务及移除；定向核验 combat 入口／后续事件、伍迪宿主和读档技能激活、王冠 buff 与 SG 事件协作、组件 post-init 顺序及调参案例。
- 主源码 SHA256：`7C48E861D6E3B8F29021E33F3E06C731AAC3DB80DA2F43EA5CE2468E58640ADC`。
- 静态检查：本页 `validate_page.py`（含 sources 存在性）、`validate_source_links.py --strict` 与完整 modmain 示例的 `parser_probe --syntax-only` 通过；未检查公共导航。
- 未覆盖：鱼人 buff 的全部授予来源、其他模组条件回调和重复组件冲突、全部伤害来源、联机表现；未运行游戏。结构与 Lua 语法检查不证明运行效果。

相关：[[reference/components/combat]]、[[reference/components/health]]。
