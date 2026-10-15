---
title: "healer"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "治疗物品的当前生命恢复、自定义门禁与用后消耗，区分治疗倍率和真实恢复量。"
aliases: ["治疗物品", "药膏回血", "治疗回调", "治疗失败", "healer"]
tags: [component, item, callback, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/healer.lua", "components/health.lua", "components/health_replica.lua", "components/stackable.lua", "actions.lua", "componentactions.lua", "prefabs/healingsalve.lua", "tuning.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# healer

## 快速判断

- 职责与适用：挂在治疗**物品**，给已有 health 的目标恢复当前生命；不负责生命上限惩罚、持续再生或复活。[healer.lua:21](../../../raw/current/scripts/components/healer.lua#L21)；惩罚恢复见 [[reference/components/maxhealer]]，持续回复见 [[reference/components/health#关键接口]]。
- 优先使用：`SetHealthAmount(n)` 设置基础治疗点数；`SetCanHealFn` 决定是否接受并返回失败原因；`SetOnHealFn` 承接额外效果。[healer.lua:9](../../../raw/current/scripts/components/healer.lua#L9
- 重要副作用：基础量乘**执行者** efficientuser 的 HEAL 倍率；通过门禁后，即使实际未回血也回调、消耗一件并返回 true。[healer.lua:36](../../../raw/current/scripts/components/healer.lua#L36
- 修改边界：服务端配置；普通使用走 HEAL 动作。直接 `Heal(target, doer)` 必须给有效 doer，不自动检查死亡、幽灵、距离或目标是否缺血。[actions.lua:3217](../../../raw/current/scripts/actions.lua#L3217)、[healer.lua:21](../../../raw/current/scripts/components/healer.lua#L21

## 关键接口

| 接口／字段 | 参数／返回 | 条件、副作用与证据 |
|---|---|---|
| `SetHealthAmount(health)`／`health` | 基础 HP 点数；默认 HEALING_SMALL=3 | 不夹取负数，不修改目标上限；原版普通药膏设为 HEALING_MED=20。[healer.lua:1](../../../raw/current/scripts/components/healer.lua#L1)、[tuning.lua:2025](../../../raw/current/scripts/tuning.lua#L2025)、[healingsalve.lua:56](../../../raw/current/scripts/prefabs/healingsalve.lua#L56 |
| `SetCanHealFn(fn)` | `fn(item, target, doer) → valid, reason?` | 无 health 先返回 false；回调否决则返回 false, reason，不治疗、不执行 onheal、不消费。[healer.lua:17](../../../raw/current/scripts/components/healer.lua#L17)、[:21](../../../raw/current/scripts/components/healer.lua#L21) |
| `SetOnHealFn(fn)` | `fn(item, target, doer)`；返回值忽略 | 在尝试 DoDelta 后、消费前调用；不是实际恢复成功通知，也不能靠返回 false 撤销治疗。[healer.lua:13](../../../raw/current/scripts/components/healer.lua#L13)、[:38](../../../raw/current/scripts/components/healer.lua#L38) |
| `Heal(target, doer)` | 缺 health／门禁失败：false；完成：true | `health.canheal` 为真才 DoDelta(delta, false, item.prefab)，但不据返回量判定成功。堆叠大于一时拆一件删除，否则删除自身。[healer.lua:36](../../../raw/current/scripts/components/healer.lua#L36)、[stackable.lua:109](../../../raw/current/scripts/components/stackable.lua#L109 |

### 动作门禁与实际恢复

HEAL 对缺省 target 使用 doer；执行阶段排除死亡和 playerghost，并优先选 healer，其次才是 maxhealer。候选动作根据 `replica.health:CanHeal()` 或物品 healerbuffs 标签，以及骑乘／持有关系生成；CanHeal 只查 cannotheal 标签，**不检查缺血量**。[actions.lua:3217](../../../raw/current/scripts/actions.lua#L3217)、[componentactions.lua:1346](../../../raw/current/scripts/componentactions.lua#L1346)、[:2736](../../../raw/current/scripts/componentactions.lua#L2736)、[health_replica.lua:200](../../../raw/current/scripts/components/health_replica.lua#L200

`healerbuffs` 允许不可常规治疗目标出现动作；healer 仍跳过其 HP 恢复并执行 onheal，原版酸蚀药膏在此添加 Debuff。需要 Buff 的药品不要一律加“必须缺血”的门禁。[healingsalve.lua:79](../../../raw/current/scripts/prefabs/healingsalve.lua#L79)、[healer.lua:38](../../../raw/current/scripts/components/healer.lua#L38

## 典型接入

### 普通治疗药膏改为基础回复四十点生命

放在已启用模组的 `modmain.lua`，可独立追加，无需 GLOBAL 代理。仅修改普通 `healingsalve`，复用原版工厂、资源、堆叠和动作，不改酸蚀等变体；注册名与配置依据见 [工厂](../../../raw/current/scripts/prefabs/healingsalve.lua#L21)、[注册](../../../raw/current/scripts/prefabs/healingsalve.lua#L268)。

```lua
AddPrefabPostInit("healingsalve", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local healer = inst.components.healer
    if healer ~= nil then
        healer:SetHealthAmount(40)
    end
end)
```

40 是单份物品的基础 HP，仍乘执行者 HEAL 倍率并受目标 health 的 redirect、无敌、deltamodifier、可用上限封顶影响；不是保证净增 40。每次新建／载入实例重新配置，停用模组后按原版初始化，不返还已消费药品。无需自行 Remove 或额外 DoDelta。[healer.lua:36](../../../raw/current/scripts/components/healer.lua#L36)、[health.lua:613](../../../raw/current/scripts/components/health.lua#L613)、[:565](../../../raw/current/scripts/components/health.lua#L565)

## 常见陷阱与限制

- **满血或回血被拦截仍耗药**：Heal 不比较前后生命；若需求是不允许这类使用，合并保留已有 canhealfn 后增加服务端门禁，不能只靠 onheal 的返回值。若有 Buff 用途，分别定义接受条件。[healer.lua:28](../../../raw/current/scripts/components/healer.lua#L28
- **把倍率挂在被治疗者**：这里读取 doer.components.efficientuser；仅增加目标倍率不影响他人给它治疗。倍率不是药品消费折扣，仍消耗一份。见 [[reference/components/efficientuser#倍率实际作用在哪里]]。
- **直接 Heal(target) 复用动作安全性**：有 health 且通过回调后会访问 doer.components；漏 doer 可报错。直接调用也绕过动作的死亡／幽灵检查；常规用药保留动作流程。[healer.lua:36](../../../raw/current/scripts/components/healer.lua#L36)、[actions.lua:3219](../../../raw/current/scripts/actions.lua#L3219
- **附加 finiteuses 就能多次用药**：healer 不读取次数，成功仍删除物品／拆一份；反复治疗工具需另行设计消费策略，不仅是加组件。[healer.lua:46](../../../raw/current/scripts/components/healer.lua#L46

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：sources 变更文件 actions.lua／componentactions.lua／tuning.lua 对照更新报告 R-01/R-04 与 hunk 地图逐 hunk 核验（health.lua、health_replica.lua、stackable.lua、healingsalve.lua 本批未变更）。锚点重定位 6 处：HEAL 动作移至 actions.lua:3207／:3209（+3 行移），EQUIPPED 段 healer 候选移至 componentactions.lua:2724（+45），tuning HEALING_MED 按"=20"内容校正至 :2025（旧锚点 2013 行实际为 HEALING_SMALL，属旧页错位一并修正）；USEITEM 段候选 :1346 行号不变。HEAL 分派、门禁与候选无语义变化。未覆盖：运行验证。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖 healer 全部实现、HEAL 分派与普通门禁、药膏普通／Buff 工厂、health 实际增量处理及堆叠取用；未审校所有特殊角色、药品或模组覆盖。
- 结构检查：发布时检查本页格式、来源、大小、链接、源码锚点和唯一登记；本页完整 modmain 示例使用已有 Full Moon／Lua 5.1 探针作 syntax-only 检查，不证明 API 绑定或行为。
- 游戏运行：未运行；满血、不可治疗、倍率叠加、多人用药、存档及其他模组覆盖仍待实测。

相关：[[reference/components/health]]；[[reference/components/maxhealer]]；[[reference/components/stackable]]；[[reference/components/debuffable]]。
