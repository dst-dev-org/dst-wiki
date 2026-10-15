---
title: "plantresearchable"
created: "2026-10-18"
updated: "2026-10-28"
type: component
summary: "提供植物研究键与阶段并向研究者发送学习事件，区分客户端候选、成长阶段和人物图鉴登记。"
aliases: ["研究胡萝卜同时学会种子", "植物图鉴研究", "植物研究阶段", "plantresearchable"]
tags: [component, callback, event, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/plantresearchable.lua", "prefabs/farm_plants.lua", "prefabs/farm_plant_defs.lua", "prefabs/weed_plants.lua", "componentactions.lua", "actions.lua", "prefabs/player_common.lua", "prefabs/player_common_extensions.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# plantresearchable

## 快速判断

- 职责与适用：挂在被研究植物上，以回调给出 plant／stage，LearnPlant 向 doer 发事件；组件本身不保存人物图鉴或改变植物成长。[完整组件](../../../raw/current/scripts/components/plantresearchable.lua#L1-L29)
- 优先使用：配置 `SetResearchFn`，正常研究由 PLANTREGISTRY_RESEARCH 调用 LearnPlant；按需直接学习时仍需有效研究者及人物事件消费者。[动作](../../../raw/current/scripts/actions.lua#L5460-L5487) [人物转发](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L929-L934)
- 重要副作用：IsRandomSeed 只检查回调第一个返回是否 nil，不检查 israndomseed 标签；LearnPlant 仅在 plant、stage 都为真值时发 learnplantstage，且无显式成功返回。[判定／事件](../../../raw/current/scripts/components/plantresearchable.lua#L17-L27)
- 修改边界：研究源方法在本页服务端执行；普通动作候选还依赖客户端可读 Getter、plantinspector 装备和本地图鉴，单加服务端组件不会自动补齐 UI／网络入口。[候选](../../../raw/current/scripts/componentactions.lua#L60-L83) [原双端接入](../../../raw/current/scripts/prefabs/farm_plants.lua#L817-L852)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetResearchFn(fn)` | `(plant_entity) -> plant_key,stage` | 实际保存字段拼写为 `reasearchinfofn`；优先用 setter，不写成 researchinfofn。[实现](../../../raw/current/scripts/components/plantresearchable.lua#L7-L15) |
| `GetResearchInfo()` | 原样返回回调结果；无回调则无返回 | 原作物回调委托 GetPlantRegistryKey／GetResearchStage；它应为查询，不在其中消费物品或发奖励。[接口](../../../raw/current/scripts/components/plantresearchable.lua#L11-L15) [原回调](../../../raw/current/scripts/prefabs/farm_plants.lua#L676-L678) |
| `IsRandomSeed()` | 首个返回 nil 时 true | 有 plant 但缺 stage 不算随机种子，却仍不能由 LearnPlant 发有效事件；不要用 false／任意字符串代替缺失键。[实现](../../../raw/current/scripts/components/plantresearchable.lua#L17-L27) |
| `LearnPlant(doer)` | 无显式返回 | 向 doer 发 `learnplantstage`，data 为 `{plant=key,stage=stage}`；不内建 doer=nil、眼镜／距离、已知与否或图鉴定义检查。[实现](../../../raw/current/scripts/components/plantresearchable.lua#L22-L27) |

原 farm_plant_carrot 的研究键是 `carrot`，不是 Prefab 名；通用植物定义把 product 设为蔬菜键，宿主再用 product 作为 plantregistrykey。研究阶段是对外 1 起始编号，底层网络值减 1；成熟／腐烂还按是否巨型映射，不能直接照抄 growable.stage。杂草另有自己的成熟／采后映射。[定义](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L254-L268) [宿主键](../../../raw/current/scripts/prefabs/farm_plants.lua#L821-L824) [阶段映射](../../../raw/current/scripts/prefabs/farm_plants.lua#L394-L410) [杂草](../../../raw/current/scripts/prefabs/weed_plants.lua#L24-L38)

动作对随机种子只说未识别台词；其他植物调用 LearnPlant，再可能说研究台词，最终返回 true，并未核验登记是否新增。人物服务端监听事件，交给 [[reference/components/plantregistryupdater]] 处理；肥料研究则走 [[reference/components/fertilizerresearchable]]。[动作](../../../raw/current/scripts/actions.lua#L5468-L5487) [事件监听](../../../raw/current/scripts/prefabs/player_common.lua#L692-L695)

## 典型接入

### 实际研究种植胡萝卜时，同时登记它的种子阶段

放目标模组 `modmain.lua`，无需新资源。包装原 LearnPlant，先保留当前阶段学习，再向同一研究者发种子阶段事件。仅本例已核验的原作物定义中，研究阶段 1 是种子；不推广到任意模组植物／杂草。[种子阶段定义](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L280-L288) [组件安装](../../../raw/current/scripts/prefabs/farm_plants.lua#L851-L852) [工厂注册](../../../raw/current/scripts/prefabs/farm_plants.lua#L926-L937) [PostInit](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("farm_plant_carrot", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local research = inst.components.plantresearchable
    if research == nil then return end
    local original = research.LearnPlant
    research.LearnPlant = function(self, doer)
        original(self, doer)
        local plant = self:GetResearchInfo()
        if plant ~= nil and doer ~= nil
                and doer.components.plantregistryupdater ~= nil then
            doer:PushEvent("learnplantstage", {plant = plant, stage = 1})
        end
    end
end)
```

生效条件是原 LearnPlant 实际被执行；仅查看、已知阶段走失败候选或随机种子分支不会触发本例。当前阶段就是种子时会重复提交同一项，是否新增由登记层去重；不额外解锁全部阶段、不发放种子、不改变生长。停用并重新初始化恢复原方法，已保存的图鉴知识不回滚。

## 常见陷阱与限制

- **只加组件就出现完整研究入口**：客户端还要有与回调一致的键／阶段 Getter 及相关标签；新图鉴条目的数据／展示也须另行接入，不能靠返回一个新字符串完成。[候选约定](../../../raw/current/scripts/componentactions.lua#L60-L77) [原双端字段](../../../raw/current/scripts/prefabs/farm_plants.lua#L817-L852)
- **查询回调可以顺便消费／奖励**：原动作先 IsRandomSeed，再 LearnPlant，会重复读取；保持查询纯粹，效果放明确的学习／登记入口。[动作顺序](../../../raw/current/scripts/actions.lua#L5468-L5474)
- **返回 nil 表示学习失败**：LearnPlant 本就无显式返回；事件提交与登记新增分别判断。组件也没有自有存读或移除标签的钩子，动态卸载须安排清理。[完整组件](../../../raw/current/scripts/components/plantresearchable.lua#L1-L29)

## 验证记录

- 版本沿用：2026-10-25，`steam-buildid:26148036`；原结论复用、锚点迁移 4 处（actions.lua PLANTREGISTRY_RESEARCH 三区间、player_common_extensions.lua OnLearnPlantStage）；依据：actions.lua／componentactions.lua 枢纽 diff 审阅＋变更清单命中 actions.lua、componentactions.lua、player_common.lua、player_common_extensions.lua，动作与人物转发代码同旧版，player_common 删除的是 rifts4 虫洞消化音效、与本页无关。未运行游戏。
- 源码审阅：2026-10-18，Steam buildid 25643519；覆盖回调拼写／返回、随机判定、事件／人物转发、动作候选、原作物／杂草键与研究阶段映射，以及胡萝卜附带种子学习例。未全面审校新植物图鉴 UI／注册、所有研究装备或角色技能。
- 结构检查：格式、来源、链接／行号、大小与登记通过；完整示例通过已有 Full Moon Lua 5.1 syntax-only，不证明客户端显示或档案保存效果。
- 游戏运行：未运行；已知／未知阶段、巨型／腐烂、眼镜候选、远端客户端、重登和其他模组包装顺序待实测。
