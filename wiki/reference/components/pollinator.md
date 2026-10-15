---
title: "pollinator"
created: "2026-10-18"
updated: "2026-10-28"
type: component
summary: "记录授粉者访问的花实体与收集阈值，区分寻花、回巢及无家蜜蜂造花的宿主流程。"
aliases: ["蜜蜂采三朵花回巢", "授粉收集数量", "蜜蜂造花", "pollinator"]
tags: [component, creature, ai, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/pollinator.lua", "prefabs/bee.lua", "brains/beebrain.lua", "behaviours/findflower.lua", "stategraphs/SGbee.lua", "actions.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# pollinator

## 快速判断

- 职责与适用：授粉者侧记录已访问的 flower 实体、目标及收集阈值；组件不自动寻路、计时或给农作物加成。[完整组件](../../../raw/current/scripts/components/pollinator.lua#L1-L47)
- 优先使用：CanPollinate／Pollinate 管访问记录；改收集门槛用 collectcount，但 HasCollectedEnough 判断是 `#flowers > collectcount`，默认 5 实际需 6 个不同实体。[访问与门槛](../../../raw/current/scripts/components/pollinator.lua#L17-L30)
- 重要副作用：CreateFlower 随机选一份已访问实体的 prefab，在授粉者位置生成同种花并清空记录；不保证固定普通花，内部也不查密度。[创建](../../../raw/current/scripts/components/pollinator.lua#L32-L40)
- 修改边界：本页服务端原工蜂；Brain／SG 决定回家或造花，killerbee 及其他授粉者不因名称相近而自动适用。只加组件不构成完整生物 AI。[工蜂安装](../../../raw/current/scripts/prefabs/bee.lua#L260-L279) [Brain](../../../raw/current/scripts/brains/beebrain.lua#L69-L81) [造花宿主](../../../raw/current/scripts/stategraphs/SGbee.lua#L171-L179)

## 关键接口

| 接口／字段 | 参数或默认 | 条件、作用与证据 |
|---|---|---|
| `flowers`／`target` | 空数组／nil | 访问记录按实体引用去重，不按 Prefab 品种去重；Pollinate 成功记入后清 target。组件无 OnSave／OnLoad，不自动持久化这些引用。[初值](../../../raw/current/scripts/components/pollinator.lua#L1-L11) [记录](../../../raw/current/scripts/components/pollinator.lua#L17-L26) |
| `CanPollinate(flower)` | 条件表达式结果 | 要求非 nil、带 flower 标签且不在记录；不内建距离、实体 IsValid 或争抢检查。[实现](../../../raw/current/scripts/components/pollinator.lua#L24-L26) |
| `Pollinate(flower)` | 无显式返回 | 符合资格才入表并清 target，不直接造花／返回 true。POLLINATE 动作转发结果，不能据 nil 推断没有发生记录。[实现](../../../raw/current/scripts/components/pollinator.lua#L17-L22) [动作](../../../raw/current/scripts/actions.lua#L2863-L2871) |
| `collectcount`／`HasCollectedEnough()` | 默认 5／boolean | 使用严格大于，不是大于等于；改变门槛影响 Brain 回家判断及满足条件的造花。[实现](../../../raw/current/scripts/components/pollinator.lua#L28-L30) |
| `CreateFlower()` | 无显式返回 | 收集足够且授粉者在有效地面才创建；不检查密度、父花 IsValid 或 SpawnPrefab 是否成功。创建后 planted=true、位置为授粉者当前位置，并清空数组。[实现](../../../raw/current/scripts/components/pollinator.lua#L32-L40) |
| `distance`／`maxdensity`／`CheckFlowerDensity()` | 5／4／boolean | 搜索当前半径内排除 FX／NOBLOCK／INLIMBO／DECOR 的实体，要求计数 < maxdensity；实际没有 flower 必需标签，不能把值解释成纯花数量。[实现](../../../raw/current/scripts/components/pollinator.lua#L42-L47) |

FindFlower 行为负责选择最近花、检查 CanPollinate 和附近其他授粉者占用，向 locomotor 推 POLLINATE；组件不代办这些步骤。工蜂 Brain 收集足够后尝试回家，无家蜜蜂在特定落地状态超时才额外检查密度并 CreateFlower，不是每次 Pollinate 自动生一朵。[行为](../../../raw/current/scripts/behaviours/findflower.lua#L14-L45) [回家](../../../raw/current/scripts/brains/beebrain.lua#L69-L81) [SG 造花](../../../raw/current/scripts/stategraphs/SGbee.lua#L171-L179)

## 典型接入

### 普通工蜂访问 3 个不同花实体，即达到原回巢收集阈值

放目标模组 `modmain.lua`，无需新资源。只改 bee 的门槛为 2，使第 3 个不同花实体满足严格大于；保留原寻花、夜间／冬季／战斗优先级与回家行为，不直接生成花或蜂蜜。[工蜂](../../../raw/current/scripts/prefabs/bee.lua#L260-L279) [注册](../../../raw/current/scripts/prefabs/bee.lua#L318) [Brain 顺序](../../../raw/current/scripts/brains/beebrain.lua#L57-L81) [PostInit](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("bee", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local pollinator = inst.components.pollinator
    if pollinator ~= nil then
        pollinator.collectcount = 2
    end
end)
```

达到阈值只满足 Brain 的收集条件，不保证立即回家、蜂蜜增产或无家时立刻造花；无家造花仍受 SG、有效地面和密度条件影响。停用并重新初始化恢复默认门槛，已有访问数组也不由组件存档保留；不假定收集进度能跨实体重建延续。

如果目标只是修改采收产量／作物压力，应查 [[reference/components/pickable]]／[[reference/components/farmplantstress]]，不要因“授粉”一词把 pollinator 当通用作物增产 API。

## 常见陷阱与限制

- **collectcount=3 表示采 3 朵**：源码用 >，实际需 4 个不同实体；同一朵重复调用不会增加记录。[条件](../../../raw/current/scripts/components/pollinator.lua#L24-L30)
- **直接 CreateFlower 已含所有生成门禁**：内部没有密度／已访问实体有效性／生成结果保护；复用原宿主条件，且新模组花必须核验实际注册。[组件](../../../raw/current/scripts/components/pollinator.lua#L32-L47) [宿主](../../../raw/current/scripts/stategraphs/SGbee.lua#L171-L177)
- **maxdensity 是范围内花数量上限**：实际搜索未要求 flower，其他未排除实体也进入计数；只改数字无法证明新花布局符合预期。[搜索](../../../raw/current/scripts/components/pollinator.lua#L42-L47)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对本页 sources 变更文件 actions.lua（R-01）：POLLINATE 动作未被本次 hunk 触及（变更集中在施法门控／拾取／腐化域），结论无修订；锚点重定位 1 处（POLLINATE.fn +3 行）；未覆盖：运行验证。

- 源码审阅：2026-10-18，Steam buildid 25643519；覆盖实体去重／门槛／返回、造花对象／位置／记录清理、实际密度过滤、工蜂安装、寻花行为和回家／SG 造花消费者。未全面审校其他授粉生物、蜂箱产蜜或全部 SG 状态。
- 结构检查：格式、来源、链接／行号、大小与登记通过；完整示例通过已有 Full Moon Lua 5.1 syntax-only，不证明实际回家、寻花或造花成功。
- 游戏运行：未运行；多蜂争花、昼夜／季节、战斗、无家落地密度、自定义花及存读后重建待实测。
