---
title: "hunger"
created: "2026-10-12"
updated: "2026-10-28"
type: component
summary: "修改饥饿上限、补充和耗速，区分可撤销耗速倍率、暂停及饥饿伤害。"
aliases: ["饥饿上限", "饱食度", "饥饿消耗", "饥饿耗速", "降低饥饿消耗", "饥饿伤害", "hunger"]
tags: [component, lifecycle, mechanism, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/hunger.lua", "components/health.lua", "prefabs/player_common.lua", "prefabs/wintersfeastbuff.lua", "util/sourcemodifierlist.lua", "modutil.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# hunger

## 快速判断

- 职责与适用：食用者的饥饿值、按时间消耗与归零伤害；食物收益来源见 [[reference/components/edible]]／[[reference/components/eater]]。[hunger.lua:146](../../../raw/current/scripts/components/hunger.lua#L146
- 优先使用：初始化 SetMax／SetRate，日常补充 DoDelta；可撤销效果用 burnratemodifiers，不覆盖弃用 burnrate。[hunger.lua:24](../../../raw/current/scripts/components/hunger.lua#L24)、[:48](../../../raw/current/scripts/components/hunger.lua#L48)
- 重要副作用：SetMax 补满并经属性更新 replica，但不发 hungerdelta／跨零事件；SetCurrent／DoDelta 才执行这些事件。[hunger.lua:8](../../../raw/current/scripts/components/hunger.lua#L8)、[:104](../../../raw/current/scripts/components/hunger.lua#L104)
- 修改边界：服务端；耗速单位为饥饿点／秒。倍率 0 只禁止自然耗值，饥饿已为零仍可能掉血；Pause 才跳过整个 DoDec。[hunger.lua:128](../../../raw/current/scripts/components/hunger.lua#L128)、[:146](../../../raw/current/scripts/components/hunger.lua#L146)

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `SetMax(amount)` | 正数上限推荐域 | max 和 current 同设 amount；运行中保留值需先记录，再 SetCurrent。[hunger.lua:48](../../../raw/current/scripts/components/hunger.lua#L48 |
| `SetRate(rate)`／`SetKillRate(rate)` | 点／秒／生命点／秒 | 基础耗速与饥饿归零时伤害速率；不是每次 tick 的量。[hunger.lua:53](../../../raw/current/scripts/components/hunger.lua#L53)、[:146](../../../raw/current/scripts/components/hunger.lua#L146) |
| `DoDelta(delta, overtime?, ignore_invincible?)` | 相对增减；无成功返回约定 | redirect 可接管；无敌默认拦截，传送即使 ignore_invincible 为真也拦截，再进 SetCurrent。[hunger.lua:128](../../../raw/current/scripts/components/hunger.lua#L128 |
| `SetCurrent(current, overtime?)`／`SetPercent(p, overtime?)` | 绝对值／基础 max 比例 | SetCurrent 限制 0～max，发 hungerdelta；跨零发 startstarving／stopstarving，绕过 DoDelta 的前置拦截。[hunger.lua:98](../../../raw/current/scripts/components/hunger.lua#L98 |
| `burnratemodifiers` | 乘法 SourceModifierList | `SetModifier(source, multiplier, key?)`／`RemoveModifier(source, key?)`，同来源同 key 替换，不重复乘；实体来源删除自动撤销。[hunger.lua:36](../../../raw/current/scripts/components/hunger.lua#L36)、[sourcemodifierlist.lua:64](../../../raw/current/scripts/util/sourcemodifierlist.lua#L64 |
| `Pause()`／`Resume()`／`IsPaused()` | 单开关 | Pause 取消周期任务并跳过消耗／饥饿伤害；Resume 重建一秒任务。多来源暂停不自动计数。[hunger.lua:75](../../../raw/current/scripts/components/hunger.lua#L75 |
| `SetOverrideStarveFn(fn)` | `(inst, dt)` | 归零后的更新由回调替代默认 health:DoDelta，需自行处理伤害／替代效果。[hunger.lua:61](../../../raw/current/scripts/components/hunger.lua#L61)、[:156](../../../raw/current/scripts/components/hunger.lua#L156) |

## 典型接入

### 让人物自然饥饿消耗降低 20%

目标模组 `modmain.lua`，普通生存模式；已有玩家组件，无新增资源。独立来源／key 保留角色基础耗速与其他 buff；AddPlayerPostInit 只筛 player 标签，仍必须判断服务端和组件。[modutil.lua:586](../../../raw/current/scripts/modutil.lua#L586)、[player_common.lua:2763](../../../raw/current/scripts/prefabs/player_common.lua#L2763

```lua
AddPlayerPostInit(function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.hunger ~= nil then
        inst.components.hunger.burnratemodifiers:SetModifier(
            inst, 0.8, "dstwiki_hunger_passive")
    end
end)
```

0.8 是乘数，仅本来源使自然耗速乘 80%；与其他来源继续相乘，不改变手动扣值或归零伤害。持续到该人物实例删除；暂时技能／装备在失效时用同列表 `RemoveModifier(inst, "dstwiki_hunger_passive")`，不要 Reset 全列表。重载生成时重新注册；本例不改变上限。[hunger.lua:154](../../../raw/current/scripts/components/hunger.lua#L154)、[sourcemodifierlist.lua:103](../../../raw/current/scripts/util/sourcemodifierlist.lua#L103

原版冬季盛宴 buff 用 buff 实体作 source，开始设置倍率 0，结束成对移除；玩家工厂的 no_hunger 模式则调用 Pause。两者边界不同。[wintersfeastbuff.lua:56](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L56)、[:62](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L62)、[player_common.lua:2767](../../../raw/current/scripts/prefabs/player_common.lua#L2767

## 生命周期

- 构造时启动一秒周期任务；LongUpdate 消耗时间但传 ignore_damage=true，不补算这段饥饿伤害；组件移除取消任务。[hunger.lua:18](../../../raw/current/scripts/components/hunger.lua#L18)、[:167](../../../raw/current/scripts/components/hunger.lua#L167)、[:190](../../../raw/current/scripts/components/hunger.lua#L190)
- 保存只记录非满的当前值，不保存 max、rate、Pause 或修饰列表；OnLoad 恢复 current 后 DoDelta(0)。宿主保存／加载分发见 [entityscript.lua:1928](../../../raw/current/scripts/entityscript.lua#L1928)、[:2005](../../../raw/current/scripts/entityscript.lua#L2005)。[hunger.lua:177](../../../raw/current/scripts/components/hunger.lua#L177

## 常见陷阱与限制

- **倍率 0 当永久免饿死**：DoDec 的归零分支仍走 override 或 health:DoDelta；要禁止整个自然流程用 Pause，并协调其他来源的恢复权限。[hunger.lua:146](../../../raw/current/scripts/components/hunger.lua#L146
- **直接改 current 就有饥饿事件**：属性 setter 只更新 replica；按需求调用 SetCurrent／DoDelta，避免漏 startstarving／stopstarving。[hunger.lua:12](../../../raw/current/scripts/components/hunger.lua#L12)、[:104](../../../raw/current/scripts/components/hunger.lua#L104)
- **每分钟耗三点写 SetRate(3)**：这里按秒消费，应为 `3 / 60`；最终还乘 burnrate 和修饰列表。[hunger.lua:154](../../../raw/current/scripts/components/hunger.lua#L154
- **每 tick 写 SetMax 保持上限**：会持续补满，不同于上限设定；初始化和运行期变更分开。[hunger.lua:48](../../../raw/current/scripts/components/hunger.lua#L48

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 3 处（player_common hunger 构造／no_hunger、entityscript 存／读两处）；无结论修订，仅锚点迁移——依据：全局枢纽 diff 审阅（player_common 本版删除的是虫王消化音效段，hunger 构造区仅行移）＋变更清单命中 player_common.lua、entityscript.lua，hunger.lua、wintersfeastbuff.lua、sourcemodifierlist.lua 未变。
- 源码审阅：2026-10-12，Steam buildid 25643519；覆盖自然耗值、饥饿伤害、事件、暂停、保存恢复与独立修饰案例；未验证每种人物特殊 redirect／override。
- 结构检查：页面／索引、链接与源码行锚点检查通过；Lua 经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；耗速、归零／补充、重载和多模组修饰协作待实测。

相关：[[reference/components/health]]；[[reference/components/eater]]。
