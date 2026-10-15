---
title: "parryweapon"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "配置武器招架角度与成功回调，通过玩家施法状态图重定向符合条件的攻击。"
aliases: ["parryweapon", "武器招架", "盾牌格挡角度", "combat_parry"]
tags: [component, callback, state-machine, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/parryweapon.lua", "components/combat.lua", "prefabs/wathgrithr_shield.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "actions.lua", "componentactions.lua", "components/aoespell.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# parryweapon

## 快速判断

- 职责与适用：挂在招架武器上，提供方向判定和回调；真正免去持有者本次受伤的是玩家 SG 设置的 `combat.redirectdamagefn`，不是组件自身扣血／吸收。[组件](../../../raw/current/scripts/components/parryweapon.lua#L35)、[SG 重定向](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5764)、[受击分支](../../../raw/current/scripts/components/combat.lua#L615)。
- 优先使用：已有原版盾牌只调 `SetParryArc`；新增武器必须一并提供地点施法、双端标签和 SG 接入。完整调参示例见“典型接入”。
- 重要副作用：成功回调可修改宿主资源；原版 `wathgrithr_shield` 会回补充能进度，并在对应技能下记录反击加成。[OnParry](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L96)。
- 修改边界：实际组件及伤害判定在服务端；`parryweapon` 标签应在 pristine 前预置，客户端以标签选择预测动作。卸组件只删标签，不替 SG 清理伤害重定向。[组件生命周期](../../../raw/current/scripts/components/parryweapon.lua#L1)、[原版工厂](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L156)。

## 关键接口

| 接口／字段 | 默认／约定 | 条件、作用与证据 |
|---|---|---|
| `SetParryArc(arc)` | 默认 `178`，单位度，完整扇区角 | 判定阈值为 `arc / 2`；角差等于阈值也不通过该分支。setter 不校验数值。[定义](../../../raw/current/scripts/components/parryweapon.lua#L15)、[阈值](../../../raw/current/scripts/components/parryweapon.lua#L49) |
| `SetOnPreParryFn(fn)`／`OnPreParry(doer)` | `fn(item, doer)` | 纯动画／音效入口，可被绕过；原版 SG 在前摇第 3 帧调用，不适合作为必达的消耗或初始化入口。[定义](../../../raw/current/scripts/components/parryweapon.lua#L19)、[SG](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5783) |
| `SetOnParryFn(fn)` | `fn(item, doer, attacker, damage)` | 角度与刺激筛选通过后调用，返回值不影响成功结果；不收到传入的攻击武器和 `stimuli`。[定义](../../../raw/current/scripts/components/parryweapon.lua#L24)、[调用](../../../raw/current/scripts/components/parryweapon.lua#L63) |
| `EnterParryState(doer, rot, duration)` | 方向为度、持续时间为秒 | 只向 doer 推送 `combat_parry`，数据为 `weapon/direction/duration`；不直接跳状态、设计时器或检查装备。[实现](../../../raw/current/scripts/components/parryweapon.lua#L35) |
| `TryParry(doer, attacker, damage, weapon, stimuli)` | 返回 boolean；attacker 须可访问标签、Transform | 拒绝非 nil 且非 `"stun"` 的刺激和 `groundspike`；先判攻击者所在方向，失败且攻击者有 locomotor 时，再判攻击者朝向与防御者反向是否接近。成功触发回调并返回 true；自身不检查招架时间或是否装备。[完整判定](../../../raw/current/scripts/components/parryweapon.lua#L39) |

### 原版动作与清理链

1. 地点动作由 `aoespell` 收集 `CASTAOE`；服务端动作先 `CanCast` 再 `CastSpell`。双端玩家 SG 看到武器 `parryweapon` 标签选择 `parry_pre`。[动作收集](../../../raw/current/scripts/componentactions.lua#L2025)、[执行](../../../raw/current/scripts/actions.lua#L4535)、[服务端选择](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1304)、[客户端选择](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L598)。
2. `parry_pre` 仅在同步执行 `PerformBufferedAction()` 前后临时监听 `combat_parry`。盾牌 SpellFn 在这个窗口发送事件；监听器设置方向、时间、物品及重定向闭包。闭包每次仍检查该物品已装备并有组件。[监听窗口](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5755)、[盾牌施法](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L85)、[aoespell 调用](../../../raw/current/scripts/components/aoespell.lua#L11)。
3. 招架成功时 `combat` 取得重定向物品，跳过持有者伤害分支；仍可发送带 `redirected` 的受击事件，SG 转入 `parry_hit`。[combat](../../../raw/current/scripts/components/combat.lua#L585)、[事件](../../../raw/current/scripts/components/combat.lua#L715)、[SG](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1884)。
4. 前摇结束转 `parry_idle`，其计时到点回 idle；退出会取消任务，未转交给后续招架状态时清掉 `redirectdamagefn`。受击和击退状态也有对应转交／清理，不要独立覆盖这个单一回调槽。[前摇退出](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5804)、[计时](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5834)、[清理](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5887)、[受击退出](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5942)、[击退退出](../../../raw/current/scripts/stategraphs/SGwilson.lua#L6023)。

## 典型接入

### 收窄原版女武神盾牌的实际招架角度

独立模组的完整 `modmain.lua` 如下；普通生存模式、服务端生效，无新增资源。实际注册名为 `wathgrithr_shield`，原版已配置双端标签、aoetargeting、aoespell、装备限制和服务端 parryweapon。[工厂](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L167)、[组件装配](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L198)、[注册](../../../raw/current/scripts/prefabs/wathgrithr_shield.lua#L220)、[注入入口](../../../raw/current/scripts/modutil.lua#L593)。

```lua
AddPrefabPostInit("wathgrithr_shield", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    if inst.components.parryweapon ~= nil then
        inst.components.parryweapon:SetParryArc(120)
    end
end)
```

`120` 是完整扇区度数；只调整每个新建／读档重建盾牌的服务端判定，仍需原版可装备者正常施放招架。原版充能、技能回调与持续时间保持原样；客户端瞄准弧的外观不会随此字段自动变化。停用该段并重新加载世界恢复原版工厂值；不需要移除组件或操作 SG。示例未进游戏运行。

## 常见陷阱与限制

- 直接在 idle 调 `EnterParryState` 不会建立原版监听窗口，不能据此获得招架；新增动作必须配合 SG，而非只挂组件。[监听边界](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5776)。
- 招架不是“前方所有攻击”：`stimuli`、地刺标签、攻击者朝向的第二判定都会改变结果；调用 `TryParry` 前须提供有效 attacker，组件不作 nil 防护。[判定](../../../raw/current/scripts/components/parryweapon.lua#L39)。
- 不要在成功回调重复手动抵消伤害或移除持有者的 combat；原版伤害路由已由 SG 接管。重定向还会绕过持有者的 [[reference/components/attackdodger]] 判定。[受击分支](../../../raw/current/scripts/components/combat.lua#L615)。
- 组件没有自身存取档／定时清理；移除只删除标签。原版 SG 负责结束状态和清重定向；新宿主须自行提供等价生命周期。[移除](../../../raw/current/scripts/components/parryweapon.lua#L11)、[SG 清理](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5887)。

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；相关调用与示例仍适用，本轮仅重定位证据（锚点 18 处），未运行游戏。

- 2026-10-21，`steam-buildid:25643519`：完整审阅组件 1–69 行，覆盖所有方法、标签增删、角度／刺激筛选与回调；定向审阅盾牌工厂、CASTAOE、双端 SG 的动作选择及招架状态、combat 重定向和上述调参案例。
- 主源码 SHA256：`8C52A00571DBB11FA25505D47A3F60467E8C72488879A7005CEA1946B91B8169`。
- 静态检查：本页 `validate_page.py`（含 sources 存在性）、`validate_source_links.py --strict` 与完整 modmain 示例的 `parser_probe --syntax-only` 通过；未检查公共导航。
- 未覆盖：熔炉事件专用实现、其他模组 SG／重定向槽兼容性、联机延迟表现、全部伤害来源与特殊技能组合；未运行游戏。页面／源码链接及完整 Lua 示例的静态检查不替代运行验证。

相关：[[reference/components/combat]]、[[reference/components/aoespell]]、[[reference/components/aoetargeting]]、[[reference/components/rechargeable]]。
