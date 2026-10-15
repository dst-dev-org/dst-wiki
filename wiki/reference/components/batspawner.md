---
title: "batspawner：未完整导出的蝙蝠追踪与选点代码"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "辨认未导出的蝙蝠追踪／选点代码；洞穴黄昏蝙蝠出巢应查 childspawner 的宿主接线。"
aliases: [batspawner, 蝙蝠生成器, 蝙蝠休眠删除, 蝙蝠生成选点]
tags: [component, lifecycle, callback]
dst_build: "steam-buildid:26230496"
sources: [components/batspawner.lua, entityscript.lua]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# batspawner：未完整导出的蝙蝠追踪与选点代码

## 快速判断

- 职责与适用：当前文件只定义局部 `BatSpawner` 类及配置、追踪、选点方法；没有实际生成蝙蝠、执行上限控制或定时生成的函数，不能据名字推断为完整世界生成系统。[batspawner.lua:1-55](../../../raw/archive/build-26148036/scripts/components/batspawner.lua#L1-L55)
- 优先使用：洞穴黄昏出蝙蝠先查 [[reference/components/childspawner#洞穴黄昏刷新蝙蝠由谁控制|childspawner 的昼夜宿主接线]]；本页用于辨认这份代码的边界，不提供添加组件的接入示例。文件末尾没有 `return BatSpawner`，而常规组件加载会读取模块返回值并将其作为构造器使用。[batspawner.lua:40-55](../../../raw/archive/build-26148036/scripts/components/batspawner.lua#L40-L55)、[entityscript.lua:52-60](../../../raw/archive/build-26148036/scripts/entityscript.lua#L52-L60)、[entityscript.lua:626-634](../../../raw/archive/build-26148036/scripts/entityscript.lua#L626-L634)
- 重要副作用：`StartTracking` 把目标设为不存档，并由宿主监听目标的 `entitysleep`，目标休眠时删除它；不会同步清除追踪表项。[batspawner.lua:28-38](../../../raw/archive/build-26148036/scripts/components/batspawner.lua#L28-L38)
- 修改边界：未发现原版注册／消费者，端与宿主不能从本文件确定。构造虽调用 `StartUpdatingComponent(self)`，但全文没有 `OnUpdate` 或停止更新／移除清理实现。[batspawner.lua:1-55](../../../raw/archive/build-26148036/scripts/components/batspawner.lua#L1-L55)

## 关键接口

以下是文件中现存局部类的方法，不代表该模块可由 `AddComponent("batspawner")` 正常实例化。

| 方法／字段 | 参数、返回／默认值 | 实现及限制 |
|---|---|---|
| `SetSpawnTimes(times)` | 原样写入 `spawntime`；默认 `TUNING.BIRD_SPAWN_DELAY` | 未校验形状，文件没有读取它执行生成的逻辑；不据此补造时间区间参数契约。[batspawner.lua:7](../../../raw/archive/build-26148036/scripts/components/batspawner.lua#L7)、[batspawner.lua:20-22](../../../raw/archive/build-26148036/scripts/components/batspawner.lua#L20-L22) |
| `SetMaxBats(max)` | 原样写入 `batcap`；默认 6 | 本文件只在调试字符串中读取上限，没有阻止新增或驱逐多余蝙蝠。[batspawner.lua:6-25](../../../raw/archive/build-26148036/scripts/components/batspawner.lua#L6-L25) |
| `StartTracking(inst)` | 待追踪实体 | `bats[inst]` 保存监听函数；宿主在该实体上监听 `entitysleep`，回调见表项仍存在便 `inst:Remove()`。不创建目标、不检查其 prefab。[batspawner.lua:28-38](../../../raw/archive/build-26148036/scripts/components/batspawner.lua#L28-L38) |
| `GetSpawnPoint(pt)` | 支持向量相加及 `Get()` 的点；返回选中点或无返回值 | 随机方向、半径 `6 + random * 6`，调用 `FindValidPositionByFan` 并传入 12；候选须 `Map:IsPassableAtPoint` 且不在 `GroundCreep` 上，无有效偏移则不返回点。[batspawner.lua:40-55](../../../raw/archive/build-26148036/scripts/components/batspawner.lua#L40-L55) |
| `GetDebugString()`、`bats` | 表键数量／容量文本；初始空表 | 输出 `GetTableSize(bats)`，不是有效存活蝙蝠的重新统计；文件中没有移除表项的路径。[batspawner.lua:4-18](../../../raw/archive/build-26148036/scripts/components/batspawner.lua#L4-L18)、[batspawner.lua:28-38](../../../raw/archive/build-26148036/scripts/components/batspawner.lua#L28-L38) |

## 常见陷阱与限制

- **配置值不等于可用机制**：初始 `timetospawn = 0`、`battypes = {"bat"}` 只是赋值；没有对应计时消费、选 prefab 或生成调用。本文不推荐通过补一个导出就恢复使用，也不补写缺失算法。[batspawner.lua:1-55](../../../raw/archive/build-26148036/scripts/components/batspawner.lua#L1-L55)
- **追踪不是完整成员生命周期**：目标睡眠删除后表项仍在；没有 `onremove` 清理、停止追踪方法、存读接口或专用清理钩子。不能依赖这份表计数实现有效数量上限或恢复存档。[batspawner.lua:28-55](../../../raw/archive/build-26148036/scripts/components/batspawner.lua#L28-L55)
- **选点不是生成**：`GetSpawnPoint` 只返回地面候选点，不创建实体，也没有飞行动作、蝙蝠 AI 或洞穴进出口协作。地面查询使用 `TheWorld.Map` 与 `GroundCreep`，不能当作纯数学圆周采样器。[batspawner.lua:40-55](../../../raw/archive/build-26148036/scripts/components/batspawner.lua#L40-L55)

## 验证记录

- 检索分流补充：2026-10-27；黄昏出巢的宿主证据集中在上述 childspawner 章节，本页保留现有未导出代码的结论和来源，不新增生成机制说明。
- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖 `batspawner.lua` 全部 56 行及常规组件的模块加载／实例化入口。简单档仅记录现存实现与不可直接接入的边界。
- 消费者定位：在当前 `scripts` 的 Lua 文件中按 `batspawner`／`BatSpawner` 精确检索，仅命中本文件；未据此声称其他历史版本或动态构造名称也无调用。
- 游戏运行：未运行；未实测模块加载报错、休眠删除或地面选点，未追溯历史来源、引擎更新调度和选点辅助函数内部。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；无新增 Lua 示例，未运行语法探针。

相关：需要原版实体暂存与释放时参见 [[reference/components/hideout]]；常规巢穴计数生成见 [[reference/components/childspawner]]，两者不作为本组件的等价替代。
