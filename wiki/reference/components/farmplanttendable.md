---
title: "farmplanttendable"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "控制植物能否被照料及成功回调，区分对话入口、阶段重置和实际快乐压力处理。"
aliases: ["与植物对话", "照料农作物", "植物交互奖励", "farmplanttendable"]
tags: [component, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/farmplanttendable.lua", "prefabs/farm_plants.lua", "prefabs/farm_plant_defs.lua", "prefabs/weed_plants.lua", "actions.lua", "componentactions.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# farmplanttendable

## 快速判断

- 职责与适用：维护 tendable 开关与 `tendable_farmplant` 标签，允许一次成功照料；具体收益由 ontendtofn 实现，不内建浇水、施肥或无压力。[构造与接口](../../../raw/current/scripts/components/farmplanttendable.lua#L2-L31)
- 优先使用：切换交互用 `SetTendable`，实际执行用 `TendTo(doer)`；改原植物收益包装 ontendtofn 并保留返回。[实现](../../../raw/current/scripts/components/farmplanttendable.lua#L22-L31)
- 重要副作用：只有开关真、回调存在且返回真值，才关闭开关并返回 true；没有回调不会自动成功。原农作物回调仅清 happiness 压力。[判定](../../../raw/current/scripts/components/farmplanttendable.lua#L26-L31) [农作物回调](../../../raw/current/scripts/prefabs/farm_plants.lua#L115-L119)
- 修改边界：本页服务端；人物交互候选排除 fire／smolder 和 mime，但直接 TendTo 不含这些门禁，魔法照料可不传 doer。[候选](../../../raw/current/scripts/componentactions.lua#L377-L385) [魔法调用](../../../raw/current/scripts/prefabs/farm_plants.lua#L613-L614)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetTendable(tendable)` | boolean | 构造默认 true，属性监听同步增删标签；不要手动只改标签。[属性回调](../../../raw/current/scripts/components/farmplanttendable.lua#L2-L24) |
| `ontendtofn` | `(plant,doer?) -> 真值或失败值` | 原农作物清 happiness 后 true，杂草只安排开心 FX 后 true；同组件不保证相同农作收益。[农作物](../../../raw/current/scripts/prefabs/farm_plants.lua#L115-L119) [杂草](../../../raw/current/scripts/prefabs/weed_plants.lua#L3-L9) |
| `TendTo(doer?)` | 成功 true；否则无显式返回 | 回调先执行，再 tendable=false；回调若临时设 true 仍会被成功结尾覆盖。[实现](../../../raw/current/scripts/components/farmplanttendable.lua#L26-L31) |
| `OnSave()`／`OnLoad(data)` | `{tendable=值}` | 只保存开关，不保存回调或快乐压力；有 data 时直接采用 data.tendable，缺字段会成 nil。[存读](../../../raw/current/scripts/components/farmplanttendable.lua#L33-L41) |

原植物在 pristine 前标标签、服务端安装组件／回调；成长阶段通过 [[reference/components/growable]] 阶段 fn 重新 SetTendable，因此不是“一株终生只照料一次”。着火还可将 happiness 标为压力并按当前阶段恢复照料资格。[注册](../../../raw/current/scripts/prefabs/farm_plants.lua#L817-L833) [安装](../../../raw/current/scripts/prefabs/farm_plants.lua#L867-L868) [阶段例](../../../raw/current/scripts/prefabs/farm_plants.lua#L415-L424) [点燃](../../../raw/current/scripts/prefabs/farm_plants.lua#L93-L101)

## 典型接入

### 成功照料种植胡萝卜时，给实际照料者 1 点精神

放目标模组 `modmain.lua`，原种植胡萝卜名为 `farm_plant_carrot`，不是地面野胡萝卜；原工厂按 plant_def.prefab 注册。保留原快乐压力处理，doer=nil 的魔法照料不奖励。即时精神用 [[reference/components/sanity#关键接口|DoDelta]]，无新资源。[名称生成](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L254-L258) [Prefab 注册](../../../raw/current/scripts/prefabs/farm_plants.lua#L926-L937) [PostInit](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("farm_plant_carrot", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local tendable = inst.components.farmplanttendable
    if tendable == nil or tendable.ontendtofn == nil then return end
    local original = tendable.ontendtofn
    tendable.ontendtofn = function(plant, doer)
        local success = original(plant, doer)
        if success and doer ~= nil and doer.components.sanity ~= nil then
            doer.components.sanity:DoDelta(1)
        end
        return success
    end
end)
```

原 `TendTo` 成功后关开关，本例不强行重开；后续阶段／着火等重开后可再次奖励，不能称为每株只奖励一次。若直接调用 ontendtofn 会绕过开关，玩法执行应走 TendTo。停用后重新初始化不装奖励包装，已变更精神不回滚。人物原动作成功还会尝试播放对植物的台词。[动作](../../../raw/current/scripts/actions.lua#L1553-L1563)

## 常见陷阱与限制

- **回调做了效果却忘记 return true**：开关不关闭、动作不成功，可能重复产生效果；保留原返回或明确返回 true。[TendTo](../../../raw/current/scripts/components/farmplanttendable.lua#L26-L31)
- **只移除 tendable_farmplant 标签**：状态与标签脱节，后续存读或阶段重置可改变结果；用 SetTendable。组件没有 OnRemoveFromEntity 清标签实现，动态卸载需由调用方安排清理。[完整组件](../../../raw/current/scripts/components/farmplanttendable.lua#L2-L43)
- **开心 FX 证明农作物无压力**：组件回调可只做外观；原作物也只清 happiness，其余压力、养分、浇水仍属其他消费者。[两种回调](../../../raw/current/scripts/prefabs/weed_plants.lua#L3-L9) [作物](../../../raw/current/scripts/prefabs/farm_plants.lua#L115-L119)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对本页 sources 变更文件 actions.lua、componentactions.lua（R-01/R-04）：farmplanttendable 候选（componentactions.lua:377 未移位）与照料动作未被本次 hunk 触及，结论无修订；锚点重定位 1 处（动作台词段 +3 行）；未覆盖：运行验证。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖开关／标签、回调返回、存读、动作过滤及台词、作物／杂草差异、阶段重置和无 doer 调用。sanity API 复用当前已核验页；未审校全部压力／阶段和乐器消费者。
- 结构检查：格式、来源、链接／行号、大小与登记通过；完整示例通过已有 Full Moon Lua 5.1 syntax-only，不证明网络或奖励次数。
- 游戏运行：未运行；生长／点燃／读档重置、默剧演员、魔法照料、多人同时交互及实际精神收益待实测。
