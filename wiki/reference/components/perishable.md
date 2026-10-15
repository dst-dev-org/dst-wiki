---
title: "perishable"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "配置随时间腐败、暂停和变质替换，保留新鲜度比例并处理持有者倍率。"
aliases: ["保质期", "腐败时间", "新鲜度", "变成腐烂食物", "perishable"]
tags: ["component", "item", "food", "lifecycle", "callback"]
dst_build: "steam-buildid:26230496"
sources: ["components/perishable.lua", "modutil.lua", "prefabs/umbrella.lua", "prefabs/veggies.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# perishable

## 快速判断

- 职责与适用：物品随时间变化的剩余保质期；不要求可食用，原版草伞也使用。按使用次数消耗见 [[reference/components/finiteuses]]。[prefabs/umbrella.lua:139](../../../raw/current/scripts/prefabs/umbrella.lua#L139
- 优先使用：初始化 SetPerishTime + StartPerishing；改已有最大时间保留比例用 SetNewMaxPerishTime。[components/perishable.lua:188](../../../raw/current/scripts/components/perishable.lua#L188 [components/perishable.lua:204](../../../raw/current/scripts/components/perishable.lua#L204
- 重要副作用：Perish 先回调、发 perished，再在可行时创建替换物并迁移堆叠／持有槽位。[components/perishable.lua:277](../../../raw/current/scripts/components/perishable.lua#L277
- 修改边界：服务端倒计时；运行中有直接 owner、湿度、天气与全局倍率，保鲜见 [[reference/components/preserver]]。[components/perishable.lua:60](../../../raw/current/scripts/components/perishable.lua#L60

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `SetPerishTime(seconds)` | 正数秒推荐域 | 同时补满剩余时间；若已在更新则重排任务。[components/perishable.lua:188](../../../raw/current/scripts/components/perishable.lua#L188 |
| `SetNewMaxPerishTime(seconds)` | 正数秒 | 保存原 GetPercent 后按新最大量应用；不同于旧存档比例迁移。[components/perishable.lua:204](../../../raw/current/scripts/components/perishable.lua#L204 |
| `StartPerishing()`／`StopPerishing()` | 开始／取消任务 | 添加组件不会自动启动；周期约 10 秒加随机量，非逐帧精确倒计时。[components/perishable.lua:262](../../../raw/current/scripts/components/perishable.lua#L262 [components/perishable.lua:322](../../../raw/current/scripts/components/perishable.lua#L322 |
| `SetPercent(pct)`／`GetPercent()` | 0～1 夹值／比例 | SetPercent 发 perishchange，但不立即调用 Perish；等后续更新。[components/perishable.lua:223](../../../raw/current/scripts/components/perishable.lua#L223 [components/perishable.lua:150](../../../raw/current/scripts/components/perishable.lua#L150 |
| `SetLocalMultiplier(mult)` | 数值 | 属性实际拼写为 localPerishMultiplyer，使用方法避免拼错；与环境倍率共同作用。[components/perishable.lua:196](../../../raw/current/scripts/components/perishable.lua#L196 [components/perishable.lua:127](../../../raw/current/scripts/components/perishable.lua#L127 |
| `SetOnPerishFn(fn)`／`onperishreplacement` | `(inst)`／替换 Prefab 名称 | 回调可移除原实体；无回调删除且无有效替换时，不保证原物品消失。[components/perishable.lua:277](../../../raw/current/scripts/components/perishable.lua#L277 |

## 典型接入

### 调整原版胡萝卜最大保质期，保留初始化时比例

目标模组 `modmain.lua`；复用 carrot 资源和 spoiled_food 替换，保持原版计时任务。原版先 SetPerishTime、StartPerishing，再由 postinit 扩展。[prefabs/veggies.lua:602](../../../raw/current/scripts/prefabs/veggies.lua#L602 [modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593

```lua
AddPrefabPostInit("carrot", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.perishable ~= nil then
        inst.components.perishable:SetNewMaxPerishTime(8 * GLOBAL.TUNING.TOTAL_DAY_TIME)
    end
end)
```

8 个游戏日对应的基础秒数；环境仍影响实际腐败速度。OnLoad 恢复保存的绝对 time 后比例可能不同，本例不迁移旧存档。停用后新实例恢复原最大时间。腐败后的替换物不因停用自动变回胡萝卜。[components/perishable.lua:329](../../../raw/current/scripts/components/perishable.lua#L329

## 常见陷阱与限制

- **新加组件不开始腐败**：需要初始化时间并 StartPerishing，不能只设替换名。[components/perishable.lua:262](../../../raw/current/scripts/components/perishable.lua#L262
- **动态改 SetPerishTime 不补新鲜度**：该方法同时重置 remain；想保留比例改用 SetNewMaxPerishTime。[components/perishable.lua:188](../../../raw/current/scripts/components/perishable.lua#L188
- **移组件自动取消周期任务**：OnRemoveFromEntity 仅移标签，StopPerishing 在 OnRemoveEntity；移组件前显式 Stop，删除实体与移组件不同。[components/perishable.lua:50](../../../raw/current/scripts/components/perishable.lua#L50 [components/perishable.lua:56](../../../raw/current/scripts/components/perishable.lua#L56

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 8 处（SetPerishTime／SetLocalMultiplier／SetNewMaxPerishTime／SetPercent／StartPerishing／Perish／StopPerishing／OnSave 各 -2，显示标签同步）；本页结论未命中本版两处变化（Update 酸雨分支 :98 追加 `CanPointHaveAcidRain` 位置门控——无酸雨区不再加速腐烂；AddTime 移除 updatetask 门控即未启动也可加时，C-15 修复类），逐访问条件保留。依据：组内共享 perishable diff＋变更清单命中 components/perishable.lua（umbrella／veggies／modutil 未变）。
- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：重设时间与保留比例、Perish 回调／替换顺序、绝对时间读档与胡萝卜保质期示例，依据见上文对应锚点；未交付旧存档比例迁移或所有环境倍率组合。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；本页 Lua 代码经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/edible]]；[[reference/components/preserver]]；[[reference/components/stackable#拆分与自定义状态]]。
