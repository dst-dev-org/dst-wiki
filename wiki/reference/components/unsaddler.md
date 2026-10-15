---
title: "unsaddler"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "作为卸鞍工具的标记参与动作发现；卸鞍执行与工具消耗由动作、坐骑和 finiteuses 协作。"
aliases: ["鞍角卸鞍耐久减半", "卸鞍工具标记", "unsaddler"]
tags: [component, equipment, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/unsaddler.lua", "prefabs/saddlehorn.lua", "componentactions.lua", "actions.lua", "components/rideable.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# unsaddler

## 快速判断

- 职责与适用：无字段／公开业务方法的标记组件；原 saddlehorn 安装它，让动作系统识别工具，不在组件内执行卸鞍。[完整定义](../../../raw/current/scripts/components/unsaddler.lua#L1-L5) [宿主](../../../raw/current/scripts/prefabs/saddlehorn.lua#L40-L54)
- 优先使用：新工具按实际工厂装配标记，耐久归 [[reference/components/finiteuses]]；实际鞍交接归 [[reference/components/rideable]]，不要虚构 unsaddler:Unsaddle 或 SetUses。
- 重要副作用：候选动作要求目标 saddled 且非 right 分支；动作执行另查战斗／死亡／拟态，再交 SetSaddle(doer,nil)，没有“标记自带无条件成功”。[候选](../../../raw/current/scripts/componentactions.lua#L1882-L1886) [执行](../../../raw/current/scripts/actions.lua#L4190-L4202)
- 修改边界：原鞍角的武器攻击耗用与 UNSADDLE 动作耗用分别配置，修改后者不改攻击消耗；无资源／输入注册不构成新物品完整骨架。[宿主配置](../../../raw/current/scripts/prefabs/saddlehorn.lua#L44-L54)

## 关键接口

| 入口 | 条件、作用与证据 |
|---|---|
| `inst:AddComponent("unsaddler")` | 仅提供标记，无 Get／Set／OnSave 业务接口；原工厂在主模拟分界后安装。[定义](../../../raw/current/scripts/components/unsaddler.lua#L1-L5) [实例](../../../raw/current/scripts/prefabs/saddlehorn.lua#L40-L54) |
| 动作候选 `unsaddler` | 两处消费者按 target.saddled／not right 插入 UNSADDLE，不直接修改目标；可见候选不代表执行资格已通过。[消费者一](../../../raw/current/scripts/componentactions.lua#L1882-L1886) [消费者二](../../../raw/current/scripts/componentactions.lua#L2522-L2526) |
| `ACTIONS.UNSADDLE.fn` | 合法分支给操作者发 saddle 事件，调用目标 SetSaddle(doer,nil)，返回 true；不是将鞍自动塞回玩家背包。[执行](../../../raw/current/scripts/actions.lua#L4190-L4202) |
| `finiteuses:SetConsumption(ACTIONS.UNSADDLE, amount)` | 原鞍角设 1；由动作消耗链使用，不由空标记自行扣次数。数值／保存语义复用 finiteuses 页。[原配置](../../../raw/current/scripts/prefabs/saddlehorn.lua#L50-L54) |

玩家卸鞍传入 doer，区别于 nil doer 丢鞍后的 discardedcb 路径；旧鞍会恢复场景并被抛出。不要为“成功卸鞍”再手动复制／生成一个鞍，或同时让标记扣一次、动作再扣一次。[交接](../../../raw/current/scripts/components/rideable.lua#L113-L133)

## 典型接入

### 原鞍角每次卸鞍的工具耗用减半

目标模组 `modmain.lua`，无新资源，沿用原 unsaddler 标记、动作与 finiteuses；只把 UNSADDLE 的基准消费从 1 改为 0.5，不补满现有耐久。[原工具](../../../raw/current/scripts/prefabs/saddlehorn.lua#L44-L62)

```lua
AddPrefabPostInit("saddlehorn", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local uses = inst.components.finiteuses
    if uses ~= nil then
        uses:SetConsumption(GLOBAL.ACTIONS.UNSADDLE, 0.5)
    end
end)
```

0.5 为该动作基准次数消耗，最终是否受其他使用倍率影响按 [[reference/components/finiteuses]]／[[reference/components/efficientuser]] 判断，不承诺所有模式下工具寿命精确翻倍。攻击 attackwear=3、鞍自身丢弃耐久与工具耗尽回调均保留；不添加“完成后再 Use”逻辑。

消费配置随新建／重载重新应用；当前剩余次数走原 finiteuses 保存，停用恢复后续原动作配置，既有耐久不回滚。新卸鞍工具仍需物品工厂、资源和动作上下文，不把这个 PostInit 当新物品完整工程。

## 常见陷阱与限制

- **组件很短所以不用审阅调用方**：标记的意义由候选／动作／宿主配置决定，不能从名称推断使用条件或耐久规则。[候选](../../../raw/current/scripts/componentactions.lua#L1882-L1886) [动作](../../../raw/current/scripts/actions.lua#L4190-L4202)
- **卸鞍会直接归入背包**：SetSaddle 的旧鞍分支恢复场景并 FlingItem，库存拾取是后续流程。[交接](../../../raw/current/scripts/components/rideable.lua#L113-L133)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：sources 中漂移文件 actions.lua、componentactions.lua（更新报告 R-01/R-04）——UNSADDLE 执行与两处候选处理器未变；锚点重定位 7 处；未覆盖：运行验证。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖完整标记定义、两处候选、UNSADDLE 执行、鞍角装配及鞍交接；消耗接口复用已核验 finiteuses。未全面审校所有输入模式、物品动作同步或拟态行为。
- 结构检查：本批格式、来源、链接／行号、大小和登记通过；完整消费例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；动作可见性、战斗／死亡拒绝、卸鞍抛物、分数消耗／倍率、存读和攻击耐久待实测。
