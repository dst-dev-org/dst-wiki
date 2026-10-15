---
title: "eater"
created: "2026-10-12"
updated: "2026-10-28"
type: component
summary: "配置食用者饮食、三维吸收与进食回调，区分可尝试食用、实际接受和食物消耗顺序。"
aliases: ["食用者", "角色饮食", "挑食", "只能吃肉", "食物吸收倍率", "进食回调", "eater"]
tags: [component, food, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/eater.lua", "components/edible.lua", "prefabs/wathgrithr.lua", "prefabs/wormwood.lua", "prefabs/player_common.lua", "constants.lua", "modutil.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# eater

## 快速判断

- 职责与适用：食用者侧饮食与吸收、进食事件和食物消耗；物品类型／基础数值／食物回调归 [[reference/components/edible]]。[eater.lua:198](../../../raw/current/scripts/components/eater.lua#L198
- 优先使用：SetDiet 区分可尝试与实际接受（SetCanEatHorrible/Gears/Nitre/Raw 已标 Deprecated）；SetAbsorptionModifiers 改食用者三维倍率；附加已成功进食效果优先监听人物 oneat。[eater.lua:52](../../../raw/current/scripts/components/eater.lua#L52)、[:270](../../../raw/current/scripts/components/eater.lua#L270)
- 重要副作用：Eat 先算收益，再发人物 oneat／oneatfn，然后食物 OnEaten／HandleEatRemove；正常流程默认只消费一件，成功返回 true，拒绝路径返回 nil。[eater.lua:198](../../../raw/current/scripts/components/eater.lua#L198)、[edible.lua:216](../../../raw/current/scripts/components/edible.lua#L216
- 修改边界：服务端；CanEat 真不保证 Eat 成功，实际要过 PrefersToEat。改人物不要为单食物效果覆盖整个 Eat。[eater.lua:304](../../../raw/current/scripts/components/eater.lua#L304

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `SetDiet(caneat, preferseating?)` | FOODGROUP 表／FOODTYPE 值的列表；省略第二参复用第一表 | caneat 变更更新 *_eater 标签；偏好用于实际进食，默认两者 OMNI。SetCanEatHorrible/Gears/Nitre/Raw 已标 Deprecated（仍可调用、重复调用幂等化，单次调用行为与旧版等价），迁移方向即本接口（26148036 核验：四接口移至文件尾，原版脚本已无调用方）。[eater.lua:7](../../../raw/current/scripts/components/eater.lua#L7)、[:52](../../../raw/current/scripts/components/eater.lua#L52)、[:339](../../../raw/current/scripts/components/eater.lua#L339)、[constants.lua:2049](../../../raw/current/scripts/constants.lua#L2049 |
| `CanEat(food)`／`PrefersToEat(food)` | 命中为 true；否则可能 nil 或 false | 前者匹配 edible 标签，后者再检查特殊食物、腐败禁食、偏好标签与 preferseating。[eater.lua:288](../../../raw/current/scripts/components/eater.lua#L288 |
| `Eat(food, feeder?)` | feeder 默认 self.inst；成功 true，拒绝 nil | 需要实际 edible 食物；触发三维／食物效果与消耗，别再手动删食物。[eater.lua:198](../../../raw/current/scripts/components/eater.lua#L198 |
| `SetAbsorptionModifiers(health, hunger, sanity)` | 三个数值倍率；默认全 1 | 普通收益计算直接相乘，无可省略／函数回调分支；0 关闭该项基础收益，负收益也按该倍率算。[eater.lua:57](../../../raw/current/scripts/components/eater.lua#L57)、[:211](../../../raw/current/scripts/components/eater.lua#L211) |
| `SetOnEatFn(fn)`／`oneat` 事件 | fn `(inst, food, feeder)`；事件 `{food, feeder}` | 单字段替换；事件发在 oneatfn 前，均位于三维收益之后、食物回调与删除之前。[eater.lua:162](../../../raw/current/scripts/components/eater.lua#L162)、[:270](../../../raw/current/scripts/components/eater.lua#L270) |
| `SetStrongStomach(flag)`／`SetIgnoresSpoilage(flag)` | 布尔约定 | 前者可跳过 monstermeat 的负生命／精神；后者影响 edible 腐败收益，二者不等同。[eater.lua:93](../../../raw/current/scripts/components/eater.lua#L93)、[:113](../../../raw/current/scripts/components/eater.lua#L113)、[:166](../../../raw/current/scripts/components/eater.lua#L166) |
| `SetRefusesSpoiledFood(flag)`／`SetPrefersEatingTag(tag)` | 开关／偏好标签 | PrefersToEat 额外门禁；偏好标签积累（26148036 起重复同标签去重），食品至少有一个偏好标签才继续。[eater.lua:85](../../../raw/current/scripts/components/eater.lua#L85)、[:304](../../../raw/current/scripts/components/eater.lua#L304) |
| `eatwholestack`／`cacheedibletags` | 默认 false／默认未赋值 | 整堆食用以 stack_mult 放大三维并改变删除；GetEdibleTags 首次默认缓存，SetDiet 不清该缓存；26148036 新增 ResetEdibleTagsCache() 显式清空。[eater.lua:28](../../../raw/current/scripts/components/eater.lua#L28)、[:172](../../../raw/current/scripts/components/eater.lua#L172)、[:194](../../../raw/current/scripts/components/eater.lua#L194) |

## 典型接入

### 成功进食后额外回复两点精神

目标模组 `modmain.lua`，普通生存模式；已有人物 eater／sanity，无新资源。监听 oneat 保留原 oneatfn；只提供每次成功进食的额外效果，不改食物基础值，也不替换人物饮食。[modutil.lua:586](../../../raw/current/scripts/modutil.lua#L586)、[eater.lua:270](../../../raw/current/scripts/components/eater.lua#L270

```lua
AddPlayerPostInit(function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.eater == nil then return end
    inst:ListenForEvent("oneat", function(player, data)
        if data ~= nil and data.food ~= nil
            and player.components.sanity ~= nil then
            player.components.sanity:DoDelta(2)
        end
    end)
end)
```

每次成功 Eat 调用增加 2 点，整堆食用仍只触发一次事件；此例明确绕开该次 eater.sanityabsorption 的基础倍率，但仍受 sanity 自身 ignore／redirect／上限限制。事件中食物通常尚待 OnEaten／消耗，不保存其引用作延迟任务，不重复 Remove。监听随人物实例绑定，不另建周期任务。[eater.lua:244](../../../raw/current/scripts/components/eater.lua#L244)、[:270](../../../raw/current/scripts/components/eater.lua#L270)；精神入口见 [[reference/components/sanity#关键接口]]。

### 自建角色饮食与吸收如何分工

- 薇格弗德 SetDiet 的 caneat 是 OMNI，preferseating 才是 MEAT／GOODIES：可尝试许多食物，但最终可能拒绝。自建角色在已建立 eater 的服务端 master_postinit 配置，若是改原版角色则用 AddPrefabPostInit；不要把 `CanEat` 当最终成功判定。[wathgrithr.lua:162](../../../raw/current/scripts/prefabs/wathgrithr.lua#L162
- 虫木用 SetAbsorptionModifiers(0, 1, 1) 关闭基础食物生命收益；这不会禁止食物 oneaten 自定义回调自行回血。数值贡献与附加回调必须分别核验。[wormwood.lua:800](../../../raw/current/scripts/prefabs/wormwood.lua#L800)、[eater.lua:275](../../../raw/current/scripts/components/eater.lua#L275

## 进食顺序与恢复

1. PrefersToEat 门禁；edible getters → foodmemory 倍率 → 各项 absorption → custom_stats_mod_fn → stack_mult → 各三维 DoDelta。[eater.lua:203](../../../raw/current/scripts/components/eater.lua#L203
2. 人物 oneat → oneatfn → 食物 OnEaten（含 oneaten 回调／事件等）→ HandleEatRemove → 更新最后进食时间／食物记忆；事件触发不代表所有后续回调已完成。[eater.lua:270](../../../raw/current/scripts/components/eater.lua#L270
3. OnSave 仅保存距上次进食的时间，OnLoad 据当前时间恢复；饮食／吸收配置不由组件存档。移除组件清 caneat 标签；宿主按钩子分发。[eater.lua:48](../../../raw/current/scripts/components/eater.lua#L48)、[:71](../../../raw/current/scripts/components/eater.lua#L71)、[entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670)、[:1928](../../../raw/current/scripts/entityscript.lua#L1928)

## 常见陷阱与限制

- **吸收 setter 只填 hunger 一项**：三个字段全被赋值，后续直接相乘可能因 nil 出错；提供三个明确倍率。[eater.lua:57](../../../raw/current/scripts/components/eater.lua#L57
- **运行中 SetDiet 自动清所有饮食缓存**：只更新 caneat 属性标签；已生成的 cacheedibletags 仍在。运行期改饮食后调 ResetEdibleTagsCache()（26148036 核验：新增接口，SetDiet 仍不自动清缓存；原版 bat／bat_boss 改食性后即调用），再按需取 GetEdibleTags；其他消费者和客户端动作仍需验证。[eater.lua:52](../../../raw/current/scripts/components/eater.lua#L52)、[:172](../../../raw/current/scripts/components/eater.lua#L172)、[:194](../../../raw/current/scripts/components/eater.lua#L194)
- **SetOnEatFn 追加原逻辑**：直接替换 oneatfn；优先独立事件监听，需要替换时保存并串联旧函数，不能自行再 Eat。[eater.lua:162](../../../raw/current/scripts/components/eater.lua#L162
- **strongstomach 免疫一切食物副作用**：这里只控制相应负生命／精神基础计算；不阻止 edible 回调、腐败或独立效果。[eater.lua:166](../../../raw/current/scripts/components/eater.lua#L166)、[:211](../../../raw/current/scripts/components/eater.lua#L211)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的五个变更文件（eater／constants／entityscript／player_common／wormwood）对照 build-26148036 报告条目 C-12、R-06、R-08、P-82～P-83、P-140 逐条核验：SetCanEatHorrible/Gears/Nitre/Raw 标 Deprecated 且幂等化（原版脚本无残留调用方，SetCanEatRawMeat 不在其列仍正常）、SetPrefersEatingTag 重复同标签去重、新增 ResetEdibleTagsCache（bat／bat_boss 运行期改食性后调用）；Eat／PrefersToEat／oneat 事件数据与缓存行为不变，FOODGROUP／FOODTYPE 仅行号平移；锚点重定位 17 处、结论修订 3 处（见正文"26148036 核验"标注）；player_common 虫胃音效迁 player_classified、wormwood 新增 plantcreature 标签，与本页结论无关；未覆盖：运行验证。

- 源码审阅：2026-10-12，Steam buildid 25643519；覆盖饮食门禁、数值顺序、事件／回调／消耗、缓存与保存、上述接入；未审校所有人物特例、动作／客户端预判和自定义食品分支。
- 结构检查：页面／索引、链接与源码行锚点检查通过；Lua 经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；饮食拒绝、整堆／单件收益、回调顺序、重载和多人兼容待实测。

相关：[[reference/components/edible]]；[[reference/components/health]]；[[reference/components/hunger]]；[[reference/components/perishable]]。
