---
title: "instrument"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "演奏前／听众／结束回调与范围搜索；效果筛选、动画声音和按次消耗由原乐器／动作链协作。"
aliases: ["排箫听众搜索半径扩大百分之五十", "乐器演奏范围", "instrument"]
tags: [component, item, actions, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/instrument.lua", "prefabs/panflute.lua", "actions.lua", "componentactions.lua", "stategraphs/SGwilson.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# instrument

## 快速判断

- 职责与适用：Play(musician) 先 onplayed，再以演奏者为中心找听众逐个 onheard，最后 onfinishedplaying；组件不自动催眠、播放声音或扣耐久。[完整执行](../../../raw/current/scripts/components/instrument.lua#L33-L51)
- 优先使用：SetRange 配搜索半径，三个 setter 配宿主回调；改原排箫范围保留原听众筛选、技能缓存、完成处理与 finiteuses 动作消费。[设置](../../../raw/current/scripts/components/instrument.lua#L8-L22) [原安装](../../../raw/current/scripts/prefabs/panflute.lua#L127-L142)
- 重要副作用：onheard(listener,musician,instrument) 的第一个参数是听众；组件只排除乐器自己和 FX／DECOR／INLIMBO，不自动排除演奏者或检查视线／敌友，原排箫另筛选。[搜索](../../../raw/current/scripts/components/instrument.lua#L33-L46) [原听众条件](../../../raw/current/scripts/prefabs/panflute.lua#L29-L56)
- 修改边界：onfinishedplaying 在同一次 Play 调用末尾同步执行，不是动画完成事件；实际排箫 SG 在 30 帧执行动作，声音／退出清理归 SG。[组件](../../../raw/current/scripts/components/instrument.lua#L47-L51) [状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L9271-L9323)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `SetRange(radius)`／`range` | 默认 15，FindEntities 使用以 musician 坐标为中心的世界距离半径。setter 直接赋值，不做范围限制；扩大候选不保证都生效。[默认](../../../raw/current/scripts/components/instrument.lua#L1-L6) [setter](../../../raw/current/scripts/components/instrument.lua#L20-L22) [搜索](../../../raw/current/scripts/components/instrument.lua#L38-L46) |
| `SetOnPlayedFn(fn)` | fn(item,musician)，在查听众前调用；原排箫设置睡眠时长和本次技能标志，不能覆盖后只保留自定义特效。[调用](../../../raw/current/scripts/components/instrument.lua#L34-L37) [原准备](../../../raw/current/scripts/prefabs/panflute.lua#L9-L27) |
| `SetOnHeardFn(fn)` | fn(listener,musician,item)，每个候选独立执行，返回值不控制整体成败；原排箫按 PVP／状态／标签筛选，再照料或施加睡意／困倦。[回调](../../../raw/current/scripts/components/instrument.lua#L38-L46) [消费者](../../../raw/current/scripts/prefabs/panflute.lua#L29-L56) |
| `SetOnFinishedPlayingFn(fn)` | fn(item,musician)，即使未命中听众也可执行；原宿主在此按技能另排延迟任务，延迟不是组件提供。[调用](../../../raw/current/scripts/components/instrument.lua#L47-L51) [原延迟](../../../raw/current/scripts/prefabs/panflute.lua#L83-L90) |
| `SetAssetOverrides(build,symbol,sound)`／`GetAssetOverrides()` | 只存取三个字段，由 SG 的 HandleInstrumentAssets 消费；不自动加载资源或选择乐器状态。[字段](../../../raw/current/scripts/components/instrument.lua#L24-L31) [消费者](../../../raw/current/scripts/stategraphs/SGwilson.lua#L679-L693) |

PLAY 动作调用库存物品组件，候选直接提供 PLAY；服务端 SG 根据 flute／horn／bell／whistle 标签选状态，没有这些标签时可无对应状态。乐器组件本身不设通用权限、冷却或存读。[动作](../../../raw/current/scripts/actions.lua#L2857-L2861) [候选](../../../raw/current/scripts/componentactions.lua#L2754-L2756) [路由](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1272-L1281)

## 典型接入

### 原排箫听众搜索半径扩大百分之五十

目标模组 `modmain.lua`，不新增资源；原 panflute 已有 flute 标签和完整演奏、消费链。只改已配置半径，不替换 OnPlayed／HearPanFlute／OnFinishedPlaying。[原标签](../../../raw/current/scripts/prefabs/panflute.lua#L101-L117) [挂载](../../../raw/current/scripts/prefabs/panflute.lua#L127-L142)

```lua
AddPrefabPostInit("panflute", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local instrument = inst.components.instrument
    if instrument ~= nil then
        instrument:SetRange(instrument.range * 1.5)
    end
end)
```

1.5 乘原配置半径，不是面积倍率，也不改变演奏动作距离、持续时间或次数。范围内实体仍须通过原 PVP、冻结／固定／化石及听众标签条件；原回调还包括农作物照料，不只影响睡眠。以演奏者而非地上乐器为中心，直接脚本调用 Play 也采用新半径。[范围消费](../../../raw/current/scripts/components/instrument.lua#L38-L46) [原效果](../../../raw/current/scripts/prefabs/panflute.lua#L29-L56)

原 finiteuses 按 PLAY 配一次消耗与技能修正，本例不在 Play 回调中再次 Use，避免双扣；动作消费机制复用 [[reference/components/finiteuses]]。组件不保存 range，构造时重建，停用并重载恢复原值；其他模组后续 setter 可覆盖，不称为独立来源倍率。[原消费与修正](../../../raw/current/scripts/prefabs/panflute.lua#L92-L99) [配置](../../../raw/current/scripts/prefabs/panflute.lua#L137-L142)

## 常见陷阱与限制

- **onheard 的 inst 是乐器**：实际第一个是听众，第三个才是乐器；参数弄反会把效果或缓存读写到错误对象。[调用](../../../raw/current/scripts/components/instrument.lua#L41-L44)
- **onfinishedplaying 表示动画已经结束**：它只在组件执行尾部；与 SG 动画／声音结束不是同一时点，要延后由实际宿主安排。[组件](../../../raw/current/scripts/components/instrument.lua#L47-L51) [SG](../../../raw/current/scripts/stategraphs/SGwilson.lua#L9283-L9323)
- **Play 返回 true 说明至少催眠一个目标**：正常执行末尾固定返回 true，回调返回值不作汇总；听众数量、效果成功与动作返回分别判断。[执行](../../../raw/current/scripts/components/instrument.lua#L33-L51)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 中变更文件 actions.lua、componentactions.lua、stategraphs/SGwilson.lua（instrument.lua／panflute.lua 本版未变）对照 build-26148036 更新报告核验：PLAY 定义／fn、INVENTORY.instrument 候选与 SG 路由／play_flute 状态均无语义变化（整体行移）；锚点重定位 6 处、结论无修订；未覆盖：运行验证。
- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整 instrument、PLAY／库存候选、排箫工厂及各回调／消费修正、服务端状态路由／资源选择和完整 play_flute。有限次数链复用同版本已核验页；未完整审校客户端预测、其他乐器、所有技能／延迟灵魂逻辑或引擎搜索边界。
- 结构检查：本批格式、来源、源码引用／行号、大小与登记通过；完整范围配置例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；主机／远端、听众边界、PVP／骑乘／冻结、农作照料、技能免耗、耗尽与其他模组覆盖待实测。
