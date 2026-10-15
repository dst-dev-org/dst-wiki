---
title: "sewing"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "针线包／胶带的衣物时间耐久修补，区分 fueled 单位、可缝补标签和工具消费。"
aliases: ["针线包", "胶带修衣服", "缝补装备", "时间耐久修复", "sewing"]
tags: [component, item, equipment, actions, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/sewing.lua", "components/fueled.lua", "components/finiteuses.lua", "actions.lua", "componentactions.lua", "prefabs/sewingkit.lua", "prefabs/sewing_tape.lua", "tuning.lua", "stategraphs/SGwilson.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# sewing

## 快速判断

- 职责与适用：挂在**缝补工具／耗材**上，给有 needssewing 标签的目标 fueled 加量；不是 armor 或 finiteuses 的统一修理接口。[sewing.lua:6](../../../raw/current/scripts/components/sewing.lua#L6)；次数修复见 [[reference/components/repairable]]。
- 优先使用：`repair_value` 设置一次恢复的 fueled 单位，`onsewn` 加后续事件。保持 SEW 动作对目标 fueled 的检查，不仅凭标签直接调用。[sewing.lua:3](../../../raw/current/scripts/components/sewing.lua#L3)、[actions.lua:1015](../../../raw/current/scripts/actions.lua#L1015
- 重要副作用：先恢复目标，再扣工具一次 finiteuses；没有次数组件时才消耗一件 stackable，随后回调并尝试授予缝补成就。[sewing.lua:10](../../../raw/current/scripts/components/sewing.lua#L10
- 修改边界：服务端配置；候选受 needssewing 与骑乘／持有关系影响，目标标签应由 fueled 的 USAGE／缺损／no_sewing 规则维护，不手工贴标签伪造能力。[componentactions.lua:1708](../../../raw/current/scripts/componentactions.lua#L1708)、[fueled.lua:5](../../../raw/current/scripts/components/fueled.lua#L5

## 关键接口

| 接口／字段 | 参数／返回 | 条件、副作用与证据 |
|---|---|---|
| `repair_value` | number；默认 1 | 直接传 fueled:DoDelta，不是百分比；按目标实际 rate 换算可用时长。默认 rate=1 的原版时间耐久用燃料单位表达秒数。[fueled.lua:89](../../../raw/current/scripts/components/fueled.lua#L89)[sewing.lua:1](../../../raw/current/scripts/components/sewing.lua#L1)、[fueled.lua:305](../../../raw/current/scripts/components/fueled.lua#L305)、[:325](../../../raw/current/scripts/components/fueled.lua#L325) |
| `DoSewing(target, doer)` | 有 needssewing：true；没有：nil | 不自行判定 fueled 是否存在，动作先检查；有效 doer 用于回调／成就，不当可省略参数。[sewing.lua:6](../../../raw/current/scripts/components/sewing.lua#L6)、[actions.lua:1015](../../../raw/current/scripts/actions.lua#L1015 |
| `onsewn` | `(tool, target, doer)`；返回忽略 | 恢复与消费之后调用；有限工具最后一次可能已触发 onfinished 并被删除，不能假定工具仍有效。[sewing.lua:12](../../../raw/current/scripts/components/sewing.lua#L12)、[finiteuses.lua:64](../../../raw/current/scripts/components/finiteuses.lua#L64 |
| 目标 `fueled.no_sewing` | 布尔开关 | true 隐藏 USAGE 缺损目标的 needssewing 标签；不是修改 accepting 或燃料类型就能等价替代。[fueled.lua:35](../../../raw/current/scripts/components/fueled.lua#L35)、[:62](../../../raw/current/scripts/components/fueled.lua#L62) |

### 与普通修复的区别

| 需求 | 推荐入口 |
|---|---|
| USAGE 衣物补充时间耐久 | sewing + 目标 fueled；需要 USAGE（主或次类型）、未满、未禁缝补，由 fueled 维护 needssewing。[fueled.lua:5](../../../raw/current/scripts/components/fueled.lua#L5)、[:20](../../../raw/current/scripts/components/fueled.lua#L20)、[:62](../../../raw/current/scripts/components/fueled.lua#L62) |
| 生命／工作量／保鲜／次数材料修复 | [[reference/components/repairable]] + [[reference/components/repairer]]，按目标类型与材料匹配 |
| armor.condition 护甲耐久 | [[reference/components/armor#关键接口]]，不因物品穿在身上就可缝补 |

DoDelta 在 0～maxfuel 内封顶、发 percentusedchange，分段变化时调用 sectionfn；SEW 不走 TakeFuel 的接收燃料流程，也没有给这次 DoDelta 传 doer。[fueled.lua:305](../../../raw/current/scripts/components/fueled.lua#L305)、[sewing.lua:10](../../../raw/current/scripts/components/sewing.lua#L10

## 典型接入

### 针线包每次恢复量翻倍，仍使用五次

放在已启用模组 `modmain.lua`，可独立追加，无 GLOBAL 代理或新增资源。注册 Prefab 是 **sewing_kit**，不是源码文件名 sewingkit；原版已配五次 finiteuses、耗尽删除和 onsewn 的人物 repair 事件。本例只改修补量，不覆盖这些协作。[sewingkit.lua:33](../../../raw/current/scripts/prefabs/sewingkit.lua#L33)、[:6](../../../raw/current/scripts/prefabs/sewingkit.lua#L6)、[:54](../../../raw/current/scripts/prefabs/sewingkit.lua#L54)

```lua
AddPrefabPostInit("sewing_kit", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local sewing = inst.components.sewing
    if sewing ~= nil then
        sewing.repair_value = sewing.repair_value * 2
    end
end)
```

原版基础量 TUNING.SEWINGKIT_REPAIR_VALUE 为 `total_day_time * 5`；翻倍为十个游戏日的**燃料量**，受目标剩余容量封顶，实际可穿多久还取决于 rate／来源倍率、消耗条件。每次仍直接 Use(1)，不读取 efficientuser 的 SEW 倍率。胶带 sewing_tape 不受本例影响；停用模组后新建／载入工具恢复默认配置，已补充耐久不回退。[tuning.lua:2366](../../../raw/current/scripts/tuning.lua#L2366)、[sewing.lua:12](../../../raw/current/scripts/components/sewing.lua#L12)、[fueled.lua:325](../../../raw/current/scripts/components/fueled.lua#L325

### 两种原版工具消费模式

针线包用 finiteuses，最后一次由 onfinished 删除；胶带用 stackable，每次拆一份删除。两者都通过 onsewn 发人物 repair 事件，成就调用在组件中；胶带另有 boatpatch，不是 sewing 自身兼容了船漏修复。[sewingkit.lua:33](../../../raw/current/scripts/prefabs/sewingkit.lua#L33)、[sewing_tape.lua:39](../../../raw/current/scripts/prefabs/sewing_tape.lua#L39)、[sewing.lua:18](../../../raw/current/scripts/components/sewing.lua#L18

## 常见陷阱与限制

- **工具没有次数／堆叠也会自动删除**：两个组件都没有时仍修补、回调并返回 true，但不会自动消耗工具；要多次用保留 finiteuses，要一次性消耗配 stackable／明确策略。[sewing.lua:12](../../../raw/current/scripts/components/sewing.lua#L12
- **同时挂次数与堆叠会双扣**：finiteuses 优先，stackable 是 elseif；不可据此设计同时减少次数与件数。[sewing.lua:12](../../../raw/current/scripts/components/sewing.lua#L12
- **给任意目标贴 needssewing**：有标签却没有 fueled 时直接 DoSewing 会报错；SEW 动作虽检查 fueled，也不能替代合理的燃料类型和标签维护。已有 USAGE 物品满值后标签自动移除。[actions.lua:1015](../../../raw/current/scripts/actions.lua#L1015)、[fueled.lua:62](../../../raw/current/scripts/components/fueled.lua#L62
- **成功即有足额收益**：修补量不按净恢复调整消费；容量封顶仍扣一次。负 repair_value 还会减少 fuel，组件不拒绝，不作为正常修补配置。[sewing.lua:10](../../../raw/current/scripts/components/sewing.lua#L10)、[fueled.lua:308](../../../raw/current/scripts/components/fueled.lua#L308
- **onsewn 返回 false 可取消或再次扣一次**：调用已在恢复与消费之后，返回忽略；不要二次 Remove／Use。临时删除 sewing 组件也不会撤销过去的修补值。[sewing.lua:18](../../../raw/current/scripts/components/sewing.lua#L18

## 验证记录

- 版本沿用：2026-10-25，`steam-buildid:26148036`；原结论复用、锚点迁移 2 处（actions.lua SEW、tuning.lua SEWINGKIT_REPAIR_VALUE，componentactions.lua 候选行未变）；依据：actions.lua／componentactions.lua 枢纽 diff 审阅＋变更清单命中 actions.lua、componentactions.lua、tuning.lua，SEW 动作、needssewing 候选与原版修补量（total_day_time*5）同旧版；prefabs/sewing_mannequin.lua 本版仅新增 canwearhat 标签，不在本页 sources。未运行游戏。
- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖完整 sewing、SEW 候选与执行、fueled 标签／加量／消费、finiteuses 耗尽及针线包／胶带工厂；未审校所有衣物、成就授予内部条件或船补组件。
- 结构检查：发布时核对格式、来源、大小、链接、源码锚点及唯一登记；完整 modmain 示例以已有 Full Moon／Lua 5.1 作 syntax-only 检查，不验证运行时状态、资源或成就。
- 游戏运行：未运行；濒满／耗尽、骑乘目标、最后一次回调、多人缝补、读档和其他模组覆盖待实测。

相关：[[reference/components/fueled]]；[[reference/components/finiteuses]]；[[reference/components/stackable]]；[[reference/components/repairable]]。
