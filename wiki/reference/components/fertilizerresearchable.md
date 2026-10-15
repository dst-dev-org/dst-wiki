---
title: "fertilizerresearchable"
created: "2026-10-18"
updated: "2026-10-28"
type: component
summary: "提供肥料图鉴键并通知研究者学习，区分研究登记、实际养分与施肥消耗。"
aliases: ["施肥自动研究便便", "肥料图鉴", "肥料研究键", "fertilizerresearchable"]
tags: [component, item, callback, event, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/fertilizerresearchable.lua", "components/fertilizer.lua", "prefabs/poop.lua", "prefabs/fertilizer_nutrient_defs.lua", "componentactions.lua", "actions.lua", "prefabs/player_common.lua", "prefabs/player_common_extensions.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# fertilizerresearchable

## 快速判断

- 职责与适用：被研究肥料侧的键查询与学习事件，不配置实际养分、施肥效果或使用次数。[完整组件](../../../raw/current/scripts/components/fertilizerresearchable.lua#L1-L24) 施肥能力见 [[reference/components/fertilizer]]。
- 优先使用：SetResearchFn 配置键；LearnFertilizer(doer) 发人物学习事件，由 [[reference/components/plantregistryupdater]] 登记，别把研究动作当施肥。[事件](../../../raw/current/scripts/components/fertilizerresearchable.lua#L7-L22) [人物消费者](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L936-L941)
- 重要副作用：只在键为真值时发 learnfertilizer，没有新增成功布尔，也不扣肥料；正常研究动作的 true 不证明档案新增。[组件](../../../raw/current/scripts/components/fertilizerresearchable.lua#L17-L22) [动作](../../../raw/current/scripts/actions.lua#L5480-L5487)
- 修改边界：本页服务端调用；普通研究候选另查 plantinspector 装备、客户端 GetFertilizerKey 和本地图鉴。组件不会给任意新物品自动注册图鉴条目。[候选](../../../raw/current/scripts/componentactions.lua#L60-L77) [原双端入口](../../../raw/current/scripts/prefabs/poop.lua#L73-L93)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetResearchFn(fn)` | `(item) -> fertilizer_key` | 实际字段同样拼作 reasearchinfofn；setter 只赋回调，不建立数据定义或网络 Getter。[实现](../../../raw/current/scripts/components/fertilizerresearchable.lua#L7-L15) |
| `GetResearchInfo()` | 原样返回回调结果；无回调则无返回 | 原便便经 GetFertilizerKey 返回 inst.prefab，对应已定义的 poop 键。[原回调](../../../raw/current/scripts/prefabs/poop.lua#L48-L54) [定义](../../../raw/current/scripts/prefabs/fertilizer_nutrient_defs.lua#L1-L6) |
| `LearnFertilizer(doer)` | 无显式返回 | 查键后向 doer 发 `learnfertilizer`，data 为 `{fertilizer=key}`；不检查 doer=nil、已知状态、装备、距离或扣材料。[实现](../../../raw/current/scripts/components/fertilizerresearchable.lua#L17-L22) |

原便便在 pristine 前加 fertilizerresearchable 标签和 GetFertilizerKey，服务端才安装组件／回调及实际 fertilizer。地面／物品栏研究候选共同使用 PlantRegistryResearch，FERTILIZER 失败理由与实际施肥动作不同。[原工厂](../../../raw/current/scripts/prefabs/poop.lua#L73-L99) [候选调用](../../../raw/current/scripts/componentactions.lua#L387-L391) [物品栏](../../../raw/current/scripts/componentactions.lua#L2720-L2722) [研究失败](../../../raw/current/scripts/actions.lua#L5445-L5457)

## 典型接入

### 玩家成功施用便便后，自动登记这项肥料研究

放目标模组 `modmain.lua`，无需新资源。包装 fertilizer.onappliedfn，在原组件的成功应用收尾中学习；只接受有 plantregistryupdater 的 doer，不增加研究装备门槛，也不重复消耗肥料。[原组件顺序](../../../raw/current/scripts/components/fertilizer.lua#L30-L48) [便便研究配置](../../../raw/current/scripts/prefabs/poop.lua#L92-L99) [PostInit](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("poop", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local fertilizer = inst.components.fertilizer
    if fertilizer == nil then return end
    local original = fertilizer.onappliedfn
    fertilizer.onappliedfn = function(item, final_use, doer, target)
        local research = item.components.fertilizerresearchable
        if research ~= nil and doer ~= nil
                and doer.components.plantregistryupdater ~= nil then
            research:LearnFertilizer(doer)
        end
        if original ~= nil then
            return original(item, final_use, doer, target)
        end
    end
end)
```

覆盖原消费者实际进入 OnApplied 的路径，包括部署和合适的目标／自肥；只完成研究事件，不修改养分。学习发生在原 onappliedfn 包装内、组件删除最后一份之前；本例先学习再保留旧回调。重复应用是否新增由登记层判断，不额外发奖励。停用并重新初始化停止自动学习，已保存的图鉴不删除。

本例主动赋予“使用即学习”，会绕过普通研究动作的眼镜候选门槛；它不是声称原版施肥默认自动研究。各施肥路径和消费边界复用 [[reference/components/fertilizer#关键接口]]，不复制一套执行代码。

## 常见陷阱与限制

- **研究组件等于肥料能力**：它只提供键／事件；新物品仍需实际 fertilizer、部署／目标消费者和独立图鉴数据，不能只加标签。[研究组件](../../../raw/current/scripts/components/fertilizerresearchable.lua#L1-L24) [原组装](../../../raw/current/scripts/prefabs/poop.lua#L92-L99)
- **只有服务端 callback 足够让客户端识别**：普通候选读取客户端 GetFertilizerKey 与 ThePlantRegistry；键应与服务器回调一致，不以“能发事件”证明 UI 已接入。[候选](../../../raw/current/scripts/componentactions.lua#L60-L77)
- **LearnFertilizer 返回 nil 就重试并扣一份**：方法没有成功返回，也不消费；普通研究和本例均不能额外 OnApplied。组件无存读或移除标签钩子，动态卸载另安排清理。[完整组件](../../../raw/current/scripts/components/fertilizerresearchable.lua#L1-L24)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的变更文件（actions.lua、componentactions.lua、prefabs/player_common.lua、prefabs/player_common_extensions.lua）对照更新报告 R-01/R-04、P-83/P-84 与 hunk 地图逐条核验，研究候选、PLANTREGISTRY_RESEARCH 分派与 learnfertilizer 转发无语义变化（P-83/P-84 为虫胃音效迁移与图鉴 FX 标签调整，不触及本页锚点区），锚点重定位 4 处、结论修订 0 处；未覆盖：运行验证。

- 源码审阅：2026-10-18，Steam buildid 25643519；覆盖键／拼写／事件、候选与研究动作、人物转发、原便便双端接入、已定义 poop 键及 OnApplied 包装顺序。实际施肥路径复用当前 fertilizer 页；未全面审校所有肥料和自定义图鉴注册／展示。
- 结构检查：格式、来源、链接／行号、大小与登记通过；完整示例通过已有 Full Moon Lua 5.1 syntax-only，不证明实际施肥、客户端登记或档案保存。
- 游戏运行：未运行；部署／自肥／目标施肥、最后一份、无 doer、重复研究、远端玩家与旧回调移除物品的兼容待实测。
