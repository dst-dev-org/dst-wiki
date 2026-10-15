---
title: "explosive"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "爆炸实体的范围伤害、整堆倍率、点燃与工作量副作用，以及燃尽触发和销毁顺序。"
aliases: ["explosive", "火药爆炸", "爆炸范围", "爆炸不点火", "整堆爆炸"]
tags: [component, item, callback, event]
dst_build: "steam-buildid:26230496"
sources: ["components/explosive.lua", "components/burnable.lua", "prefabs/gunpowder.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# explosive

## 快速判断

- 职责与适用：OnBurnt 一次性范围处理工作量、点燃、combat 伤害及事件，最后删自身；不负责弹道／倒计时。[explosive.lua:31](../../../raw/current/scripts/components/explosive.lua#L31
- 优先使用：已有火药 PostInit 改 explosivedamage／explosiverange／lightonexplode；燃尽由 [[reference/components/burnable]] 触发，或在已核验服务端入口直接调 OnBurnt，不必真的燃烧。[gunpowder.lua:70](../../../raw/current/scripts/prefabs/gunpowder.lua#L70)、[burnable.lua:291](../../../raw/current/scripts/components/burnable.lua#L291
- 重要副作用：普通伤害／工作量按自身堆数放大，不点燃不等于不破坏。onexplodefn 在扫描／伤害／删除之前，不是纯爆后通知。[explosive.lua:44](../../../raw/current/scripts/components/explosive.lua#L44)、[:68](../../../raw/current/scripts/components/explosive.lua#L68)
- 修改边界：服务端改组件，保留工厂资源／导火索／删除链；SetAttacker 用于仇恨建议，GetAttacked 的来源仍是爆炸实体，不自动套人物武器增伤。设 knockbackdata 后伤害与击退解耦：非 notpvp 目标只受击退不掉血，forcepvp 控制击退事件是否也到被炸玩家。[explosive.lua:58](../../../raw/current/scripts/components/explosive.lua#L58)、[:62-64](../../../raw/current/scripts/components/explosive.lua#L62-L64)、[:116-121](../../../raw/current/scripts/components/explosive.lua#L116-L121)

## 关键接口

| 接口／字段 | 参数／默认 | 条件、副作用与证据 |
|---|---|---|
| `OnBurnt(target?)` | 无显式返回 | 无已爆门禁；按当前状态结算后 Remove，不在其他入口重复调用或安排递归爆炸。可选 `target` 仅把 `ExplodeEnt` 的处理对象缩小到该实体（如虫腹内爆炸）；闪光、爆炸回调、当前点码头伤害与世界 "explosion" 事件仍照常执行，省略时保持全范围搜索。[explosive.lua:31](../../../raw/current/scripts/components/explosive.lua#L31)、[:157](../../../raw/current/scripts/components/explosive.lua#L157) |
| `explosiverange` | 默认 3 | FindEntities 中心半径，排除 INLIMBO／notarget，再排自身／无效对象；不是目标物理边缘扩展半径。[explosive.lua:5](../../../raw/current/scripts/components/explosive.lua#L5)、[:140](../../../raw/current/scripts/components/explosive.lua#L140) |
| `explosivedamage` | 默认 200 | 普通基础量×自身 StackSize；再施 damagetypebonus／explosiveresist，送 combat:GetAttacked，不等于最终扣血。[explosive.lua:48](../../../raw/current/scripts/components/explosive.lua#L48)、[:98-102](../../../raw/current/scripts/components/explosive.lua#L98-L102) |
| `buildingdamage` | 默认 10 | 对可工作目标 WorkedBy(self.inst,量)，不是建筑生命伤害；乘自身件数／类型倍率，物品目标还可乘目标件数且受处理配额限制。[explosive.lua:68](../../../raw/current/scripts/components/explosive.lua#L68 |
| `lightonexplode` | 默认 true | 点燃无 fueled、可燃、未燃烧／未 burnt 对象；false 只关此分支，不撤销传播或工作量／伤害。[explosive.lua:87](../../../raw/current/scripts/components/explosive.lua#L87 |
| `SetOnExplodeFn(fn)` | `(inst)`，返回忽略 | 闪光后、读堆数／位置前；不要删自身、重调 OnBurnt 或假定已结算。[explosive.lua:14](../../../raw/current/scripts/components/explosive.lua#L14)、[:44](../../../raw/current/scripts/components/explosive.lua#L44) |
| `SetAttacker(attacker)` | 实体引用 | self.attacker 优先于 pvpattacker，用于受击后 SuggestTarget，不替换本次 GetAttacked 来源。[explosive.lua:18](../../../raw/current/scripts/components/explosive.lua#L18)、[:117](../../../raw/current/scripts/components/explosive.lua#L117) |
| `SetKnockbackData(knockbackdata)` | `{forcelanded?, forcepvp?, radius?, strengthmult?}`（26230496 新增） | 设置后 OnBurnt 对过滤通过的目标追加 `PushEventImmediate("knockback", { knocker, radius, strengthmult, forcelanded })`；伤害仍只对 notpvp 目标结算，forcepvp 只放宽击退事件的到达。不设置＝旧版行为（无击退事件）；SG 侧消费者为 commonstates 击退框架。pumpkin_bomb 用 `{forcelanded=false, forcepvp=true, radius=TUNING.PUMPKIN_BOMB_RANGE, strengthmult=1}`。[explosive.lua:26-28](../../../raw/current/scripts/components/explosive.lua#L26-L28)、[:119-121](../../../raw/current/scripts/components/explosive.lua#L119-L121) |
| `SetPvpAttacker(attacker)` | 实体引用 | 非 nil 时过滤其他 player 实体，但保留此人物／非 player；不是全局 PVP 开关，不禁环境破坏。[explosive.lua:22](../../../raw/current/scripts/components/explosive.lua#L22)、[:59](../../../raw/current/scripts/components/explosive.lua#L59) |
| `skip_camera_flash` | truthy 时跳过 | 仅跳过本组件 ScreenFlash／ShakeCamera，不控制工厂自建 FX。[explosive.lua:32](../../../raw/current/scripts/components/explosive.lua#L32 |

### 结算与事件顺序

闪光／摇屏→onexplodefn→读堆数／位置→当前点码头伤害→逐目标工作量→复查有效性→点燃→combat 伤害（设 knockbackdata 时随后对同目标推即时 knockback 事件）→仇恨建议→目标 explosion→按自身件数发世界 explosion→若自身带 health 则发世界 entity_death／自身 death→Remove。[explosive.lua:31](../../../raw/current/scripts/components/explosive.lua#L31)、[:53](../../../raw/current/scripts/components/explosive.lua#L53)、[:132](../../../raw/current/scripts/components/explosive.lua#L132)、[:147](../../../raw/current/scripts/components/explosive.lua#L147)

目标事件为 `{explosive=inst}`，世界事件为 `{pt=爆炸点, damage=单件基础量}` 且整堆发多次，不能当一次总伤害通知。特殊伤害另由 SpDamageUtil 收集，现按 damagetypebonus 倍率处理后还会乘自身 stacksize，不再与普通伤害的堆数放大分开；不把普通伤害倍率机械套到它。[explosive.lua:104-111](../../../raw/current/scripts/components/explosive.lua#L104-L111)、[:132](../../../raw/current/scripts/components/explosive.lua#L132)、[:147-148](../../../raw/current/scripts/components/explosive.lua#L147-L148)

（26148036 核验：`OnBurnt` 新增可选 `target` 参数，指定时仅缩小 `ExplodeEnt` 的处理对象（码头伤害/世界事件/闪光仍执行），省略时保持全范围搜索；spdamage 在 damagetypebonus 倍率后追加乘 stacksize——依赖堆叠爆炸旧单倍特殊伤害的用法需调整；世界事件 "explosion" 新增 `pt` 字段，已有监听兼容。）

（26230496 核验：新增 `SetKnockbackData`，设值后目标过滤的"是否受击"与"是否受击退"分离——`notpvp` 判定改用实体 isplayer 标志；伤害只对 notpvp 目标走 GetAttacked，knockback 即时事件按 `notpvp or forcepvp` 推送，玩家判定由标签改为实体标志；未设 knockbackdata 时行为与旧版一致。pumpkin_bomb 为首个官方消费者。）

## 典型接入

### 火药降低单件伤害且不主动点燃周围对象

放在目标模组 `modmain.lua`；复用 gunpowder 的资源、随机导火索、爆炸 FX 和堆叠，仅改两字段，不覆盖 onexplodefn／burnable 回调。[gunpowder.lua:11](../../../raw/current/scripts/prefabs/gunpowder.lua#L11)、[:60](../../../raw/current/scripts/prefabs/gunpowder.lua#L60)

```lua
AddPrefabPostInit("gunpowder", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local explosive = inst.components.explosive
    if explosive ~= nil then
        explosive.explosivedamage = 100
        explosive.lightonexplode = false
    end
end)
```

100 是单件基础普通伤害，十件仍按 1000 进入修饰；范围仍为 3、buildingdamage 仍为 10。本例仍工作破坏、伤害生物／码头、消耗整堆，不作为安全爆竹；火药还带 propagator，本例不关闭它或外部火源。停用后新建／载入实体恢复工厂值，已有伤害不回滚。[explosive.lua:48](../../../raw/current/scripts/components/explosive.lua#L48)、[:68](../../../raw/current/scripts/components/explosive.lua#L68)、[gunpowder.lua:60](../../../raw/current/scripts/prefabs/gunpowder.lua#L60

### 燃烧触发与直接触发

burnable 的 DoneBurning 先发 onburnt、调普通烧毁回调，实体仍有效才调 explosive:OnBurnt。原版火药将普通 onburnt 设 nil，避免先被默认处理删除；进入鼹鼠物品栏则直接 OnBurnt。新对象需核对自己的触发入口，不能仅挂组件就假定自动点火／定时爆炸。[burnable.lua:291](../../../raw/current/scripts/components/burnable.lua#L291)、[gunpowder.lua:26](../../../raw/current/scripts/prefabs/gunpowder.lua#L26)、[:64](../../../raw/current/scripts/prefabs/gunpowder.lua#L64)

## 常见陷阱与限制

- **lightonexplode=false 便能无损爆炸**：只关点燃，workable 和伤害照常；纯 FX 用独立效果流程，不直接复用完整 OnBurnt。[explosive.lua:68](../../../raw/current/scripts/components/explosive.lua#L68
- **SetAttacker 后按人物增伤算爆伤**：源码特意传 self.inst 给 GetAttacked；人物引用主要影响后续仇恨，不用普通武器公式替代本页。[explosive.lua:117](../../../raw/current/scripts/components/explosive.lua#L117
- **onexplodefn 返回 false 能取消**：返回被忽略，后面继续读实体、扫描、删除；门禁应放在调用 OnBurnt 前。[explosive.lua:44](../../../raw/current/scripts/components/explosive.lua#L44
- **加 health 就有完整普通死亡结算**：末尾手动发两个事件后 Remove，不调用 health:Kill；掉落／死亡回调按工厂另核验。[explosive.lua:151](../../../raw/current/scripts/components/explosive.lua#L151

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件对照 build-26148036 更新报告逐条核验，锚点重定位 15 处、结论修订 2 处（OnBurnt 可选参数与 stacksize/pt，见正文"26148036 核验"标注）；未覆盖：运行验证。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖完整 explosive、燃尽触发及火药工厂／鼹鼠触发，核对堆数、筛选、事件对象与回调顺序。未审校全部消费者、抗性／特殊伤害／码头内部及 PVP 全链。
- 结构检查：发布时核对字段、来源、大小、链接／锚点与唯一登记；完整 modmain 示例仅作 Lua 5.1 syntax-only 检查。
- 游戏运行：未运行；整堆爆炸、工作量配额、传播、PVP／多人表现、伤害和读档待实测。

相关：[[reference/components/burnable]]；[[reference/components/propagator]]；[[reference/components/workable]]；[[reference/components/combat]]；[[reference/components/stackable]]。
