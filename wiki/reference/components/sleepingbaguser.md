---
title: "sleepingbaguser"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "人物侧床具使用任务、三维恢复倍率与可睡回调；由床具建立关系并统一唤醒，不能单独替代睡眠状态图。"
aliases: ["人物睡眠收益","睡眠回血倍率","睡眠饥饿消耗","睡袋使用者","睡眠门禁","sleepingbaguser"]
tags: [component, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/sleepingbaguser.lua","components/sleepingbag.lua","prefabs/player_common.lua","stategraphs/SGwilson.lua","entityscript.lua","scheduler.lua","modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# sleepingbaguser

## 快速判断

- 职责与适用：挂在人物，读取床具参数、按周期结算三维与温度回调；原人物已有。床具占用／资源／回调在 [[reference/components/sleepingbag]]。
- 优先使用：人物收益改 SetHealthBonusMult／SetSanityBonusMult，饥饿量改 SetHungerBonusMult；床具专属参数改床具，避免两个位置重复乘算。见[关键接口](#关键接口)。
- 重要副作用：先扣饥饿再回精神，挨饿会跳过治疗并在 tick 末请求床具唤醒；任务不直接清空双方归属。见[每次结算与退出](#每次结算与退出)。
- 修改边界：服务端；ShouldSleep 是供调用者消费的回调，不是 DoSleep 的统一强制门禁。不要直接对任意实体开启本任务。

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetHungerBonusMult(mult)`／`SetHealthBonusMult(mult)`／`SetSanityBonusMult(mult)` | 数值倍率，初始均为 1 | 各自替换单值，不是可叠加来源表。hunger_tick 通常为负，倍率 0.5 即一半睡眠扣量，不是恢复 50% 饥饿。[sleepingbaguser.lua:9](../../../raw/current/scripts/components/sleepingbaguser.lua#L9)、[:16](../../../raw/current/scripts/components/sleepingbaguser.lua#L16)、[:83](../../../raw/current/scripts/components/sleepingbaguser.lua#L83) |
| `SetCanSleepFn(fn)`／`ShouldSleep()` | `(inst) -> success, reason`／原样返回二值 | 无回调返回 true；函数内部不验证时段、饥饿或危险，也不自行发台词。SGwilson bedroll 入口只取第二个返回值作为 failreason，故 false, nil 在该入口不拒绝。[sleepingbaguser.lua:28](../../../raw/current/scripts/components/sleepingbaguser.lua#L28)、[:70](../../../raw/current/scripts/components/sleepingbaguser.lua#L70)、[SGwilson.lua:12419](../../../raw/current/scripts/stategraphs/SGwilson.lua#L12419 |
| `DoSleep(bed)` | 有 sleepingbag 的床具实体 | 存 bed、监听世界 phase，取消旧 tick 后按 bed.tick_period 建周期任务；不先 ShouldSleep、不建立双方占用、不进入睡眠 SG。[sleepingbaguser.lua:40](../../../raw/current/scripts/components/sleepingbaguser.lua#L40)、[sleepingbag.lua:61](../../../raw/current/scripts/components/sleepingbag.lua#L61 |
| `DoWakeUp(nostatechange?)` | boolean? | 取消任务与 phase 监听；非 true 时访问人物 inventory／SG 并切 wakeup，不清 bed 或床具 sleeper。常规从床具 DoWakeUp 调它。[sleepingbaguser.lua:53](../../../raw/current/scripts/components/sleepingbaguser.lua#L53)、[sleepingbag.lua:74](../../../raw/current/scripts/components/sleepingbag.lua#L74 |
| `SleepTick()` | 无参数 | 读取当前床具增量与人物倍率；好睡眠头饰标签 good_sleep_aid 还乘精神收益。不是任意实体可安全调用的通用治疗入口。[sleepingbaguser.lua:79](../../../raw/current/scripts/components/sleepingbaguser.lua#L79 |

## 每次结算与退出

1. 用床具每 tick 量乘人物倍率，先 hunger:DoDelta，再读 IsStarving。
2. 精神未达带惩罚上限时回精神；挨饿仍可走这一步。未挨饿且床具 healthsleep 为真时才调用 health:DoDelta。
3. 调床具 temperaturetickfn（如存在）；最后若挨饿，通过床具 DoWakeUp 退出。干燥／ambient_temp 由其他组件更新，见 [[reference/components/sleepingbag#关键接口]]。[sleepingbaguser.lua:79](../../../raw/current/scripts/components/sleepingbaguser.lua#L79

世界 phase 变化后，若不等于床具 GetSleepPhase 才请求床具醒来；WatchWorldState 注册并不在 DoSleep 内主动验证当前时段。首次 tick 默认为一个 tick_period 后，而非立即恢复。[sleepingbaguser.lua:32](../../../raw/current/scripts/components/sleepingbaguser.lua#L32)、[:40](../../../raw/current/scripts/components/sleepingbaguser.lua#L40)、[entityscript.lua:1537](../../../raw/current/scripts/entityscript.lua#L1537)、[scheduler.lua:295](../../../raw/current/scripts/scheduler.lua#L295

退出应由床具清双向关联并调使用者取消任务；原 SG 的 bedroll／tent onexit、人物 OnDespawn 有额外清理。使用者组件没有 OnRemoveFromEntity／OnSave／OnLoad：**在使用中直接移组件可能留下回调引用缺失组件**，先经床具唤醒，不手写 bed 字段代替退出；配置在初始化重施。[sleepingbaguser.lua:32](../../../raw/current/scripts/components/sleepingbaguser.lua#L32)、[:53](../../../raw/current/scripts/components/sleepingbaguser.lua#L53)、[SGwilson.lua:12492](../../../raw/current/scripts/stategraphs/SGwilson.lua#L12492)、[:12586](../../../raw/current/scripts/stategraphs/SGwilson.lua#L12586)、[player_common.lua:1417](../../../raw/current/scripts/prefabs/player_common.lua#L1417

## 典型接入

### 人物睡床回血翻倍且睡眠饥饿扣量减半

目标模组 `modmain.lua`；普通人物已有 sleepingbaguser，复用原床具和人物动画，不新建资源。改人物侧两个倍率，让所有使用该组件结算的床具受影响；不修改床具 tick 或通用 hunger 消耗速率。[player_common.lua:2829](../../../raw/current/scripts/prefabs/player_common.lua#L2829)、[modutil.lua:586](../../../raw/current/scripts/modutil.lua#L586

```lua
AddPlayerPostInit(function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local user = inst.components.sleepingbaguser
    if user ~= nil then
        user:SetHealthBonusMult(user.health_bonus_mult * 2)
        user:SetHungerBonusMult(user.hunger_bonus_mult * 0.5)
    end
end)
```

对该初始化实例各乘一次，原值 1 时得到 2 和 0.5；其他模组之后再次 setter 仍可覆盖，不保证与任意改动顺序无关。睡眠倍率不是持久来源列表，本例常驻至实例移除，读档创建人物时重施；临时效果须自行保存／恢复基线或实现来源聚合，不能退出时无条件 setter(1) 抹掉他人配置。组件无保存倍率逻辑，也不改变是否能睡、床具次数或独立饥饿系统。[sleepingbaguser.lua:16](../../../raw/current/scripts/components/sleepingbaguser.lua#L16)、[:83](../../../raw/current/scripts/components/sleepingbaguser.lua#L83)

### 只限制睡袋入口时如何接可睡回调

已有服务端角色初始化中可接 SetCanSleepFn；需要拒绝时返回 `false, "ANNOUNCE_..."`，reason 须是目标人物已有或模组已配置的有效字符串键，并保留之前回调的拒绝结果。**本页不提供“禁止一切睡眠”的代码**：当前 bedroll 只消费 reason，tent 入口没有同样调用，DoSleep 也不自动调用。若要全床具／催眠门禁，必须逐入口核验，不滥用此单回调。[sleepingbaguser.lua:70](../../../raw/current/scripts/components/sleepingbaguser.lua#L70)、[SGwilson.lua:12419](../../../raw/current/scripts/stategraphs/SGwilson.lua#L12419)、[:12516](../../../raw/current/scripts/stategraphs/SGwilson.lua#L12516)

## 常见陷阱与限制

- **hunger 倍率越大越省饥饿**：直接乘负增量，通常越大扣得越多；0／负倍率会改变玩法，源码未替 setter 限制取值。[sleepingbaguser.lua:16](../../../raw/current/scripts/components/sleepingbaguser.lua#L16)、[:83](../../../raw/current/scripts/components/sleepingbaguser.lua#L83)
- **cansleepfn 返回 false 就能统一禁睡**：取决于消费者；bedroll 还需要非 nil reason，tent 不走同样门禁。见[可睡回调](#只限制睡袋入口时如何接可睡回调)。
- **组件的 self.healthsleep／dryingrate 是实际配置**：构造虽有同名字段，但当前 SleepTick 读的是 **bed.components.sleepingbag.healthsleep**，干燥由人物 moisture 读床具；不要因存在默认字段就写错对象。[sleepingbaguser.lua:1](../../../raw/current/scripts/components/sleepingbaguser.lua#L1)、[:97](../../../raw/current/scripts/components/sleepingbaguser.lua#L97)；见 [[reference/components/sleepingbag#关键接口]]。
- **每次 SleepTick 必定回血**：先扣饥饿后判饥饿，最后一 tick 可能只有精神／温度处理而不治疗；治疗还受生命组件自身规则，结构／语法检查不能证明实际收益。见[结算流程](#每次结算与退出)。

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 6 处（SGwilson 大改后 +88 偏移、player_common 删 14 行、entityscript +3，区间文本逐段比对一致）；结论修订：无，仅锚点迁移。
- 依据：全局枢纽 hunk 地图（R-08）＋变更清单命中 stategraphs/SGwilson.lua、prefabs/player_common.lua、entityscript.lua。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖倍率、ShouldSleep 二值与实际 SG 消费差异、周期／phase、tick 顺序、双向清理及人物接入。未审校全部角色、好睡眠头饰工厂、所有床具入口或模组自定义状态图。
- 结构／语法：页面格式、来源、大小、链接与唯一登记通过，源码锚点 strict 检查无问题；本页完整 modmain 示例经 Full Moon 3.0.0／Lua 5.1 syntax-only 通过，未检查字节往返或运行效果。
- 游戏运行：未运行；实际回血／饥饿临界、门禁提示、断线退出、多来源倍率及联机兼容待测。

相关：[[reference/components/sleepingbag]]；[[reference/components/health]]；[[reference/components/hunger]]；[[reference/components/sanity]]；[[reference/components/grogginess]]。
