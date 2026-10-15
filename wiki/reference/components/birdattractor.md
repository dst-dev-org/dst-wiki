---
title: "birdattractor"
created: "2026-10-24"
updated: "2026-10-28"
type: component
summary: "给玩家提供按键累加的附近飞鸟上限与生成延迟修正，复用羽毛帽的来源增删和世界排程刷新。"
aliases: ["birdattractor", "羽毛帽吸引鸟", "玩家附近鸟数量", "吸引飞鸟来源", "maxbirds"]
tags: [component, creature, equipment, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/birdattractor.lua", "components/birdspawner.lua", "util/sourcemodifierlist.lua", "prefabs/player_common.lua", "prefabs/hats.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# birdattractor

## 快速判断

- 职责与适用：玩家侧保存 `spawnmodifier` 来源表；由世界 [[reference/components/birdspawner]] 读取，分别加到 `maxbirds`、`mindelay`、`maxdelay`。组件本身不生成鸟、不搜索物品，也不是一定生效的诱鸟区域。[完整定义](../../../raw/current/scripts/components/birdattractor.lua#L1-L15) [读取方](../../../raw/current/scripts/components/birdspawner.lua#L116-L122)
- 优先使用：对已有玩家组件按独有来源增删三个键；参考羽毛帽的穿戴／卸下流程，变更后调用世界 `birdspawner:ToggleUpdate()` 重排。没有世界生成器则不假定存在同等效果。[穿戴](../../../raw/current/scripts/prefabs/hats.lua#L557-L568) [卸下](../../../raw/current/scripts/prefabs/hats.lua#L572-L582)
- 重要副作用：这些是**加算差值**，不是倍率或最终上限；当前羽毛帽分别加 `3`、`-3`、`-5`，基础限额由 4 鸟变为 7 鸟，常规下轮延迟由 5～15 秒变为 2～10 秒，仍受世界门禁和落点影响。[默认与差值](../../../raw/current/scripts/tuning.lua#L1078-L1083) [自然数量判定](../../../raw/current/scripts/components/birdspawner.lua#L154-L165)
- 修改边界：原版玩家在 `SetPristine` 后的服务端分支添加；没有自己的网络、存档、更新或移除钩子。不要为了撤销一次效果移除整个玩家组件，应撤销自己的来源。[端界](../../../raw/current/scripts/prefabs/player_common.lua#L2611-L2614) [挂载](../../../raw/current/scripts/prefabs/player_common.lua#L2668-L2672) [定义](../../../raw/current/scripts/components/birdattractor.lua#L3-L15)

## 关键接口

| 字段／来源表接口 | 约定与证据 |
|---|---|
| `spawnmodifier` | `SourceModifierList(inst,0,additive)`；跨来源相同键相加。组件没有 `SetMaxBirds` 或直接生成接口。[构造](../../../raw/current/scripts/components/birdattractor.lua#L3-L8) [按键合计](../../../raw/current/scripts/util/sourcemodifierlist.lua#L155-L165) |
| `spawnmodifier:SetModifier(source,delta,key)` | key 使用 `"maxbirds"`、`"mindelay"`、`"maxdelay"`；相同来源／键是替换，delta=0 或 nil 会移除该键。用稳定来源区分效果，不把三个键塞成同一个无 key 的值。[源码提示](../../../raw/current/scripts/components/birdattractor.lua#L6-L7) [写入](../../../raw/current/scripts/util/sourcemodifierlist.lua#L64-L98) |
| `spawnmodifier:RemoveModifier(source,key?)` | 省 key 撤销该来源全部键；只撤自己的来源。实体来源被 Remove 时列表会自动撤销，但没有自动重排世界任务的回调。[显式移除](../../../raw/current/scripts/util/sourcemodifierlist.lua#L103-L121) [自动移除](../../../raw/current/scripts/util/sourcemodifierlist.lua#L84-L91) |
| `spawnmodifier:CalculateModifierFromKey(key)` | 返回某一键的累计差值。不要用 `Get()` 取三个含不同单位的键的混合合计；世界消费者明确逐键读取。[按键函数](../../../raw/current/scripts/util/sourcemodifierlist.lua#L155-L165) [消费者](../../../raw/current/scripts/components/birdspawner.lua#L116-L122) |
| `GetDebugString()` | 显示三键差值，不是最终世界配置，也不代表附近已生成这些鸟。[实现](../../../raw/current/scripts/components/birdattractor.lua#L10-L12) |

## 典型接入

### 让自定义装备增加持有者附近的自然飞鸟

这是目标模组现有装备的服务端生命周期指引，不是带资源的新装备 demo；穿戴骨架与持有者语义复用 [[reference/components/equippable]]。

1. 穿戴回调取得 `owner.components.birdattractor`；按装备实体作为来源设置三键差值。原羽毛帽先执行共同穿戴逻辑再登记，不替换掉原显示或其他穿戴效果。[原版接线](../../../raw/current/scripts/prefabs/hats.lua#L557-L568) [工厂安装](../../../raw/current/scripts/prefabs/hats.lua#L586-L600)
2. 完成登记后，若世界组件存在则 `ToggleUpdate()`；它会刷新**所有活跃玩家**的已排任务，不只该 owner。不要每帧刷新，也不要把一次刷新理解为立即生成鸟。[羽毛帽刷新](../../../raw/current/scripts/prefabs/hats.lua#L565-L567) [公共入口](../../../raw/current/scripts/components/birdspawner.lua#L441-L442) [实际重排](../../../raw/current/scripts/components/birdspawner.lua#L195-L213)
3. 卸下时移除装备来源，再刷新世界排程。效果恢复与资源销毁由宿主决定；修改后的数量和延迟不能要求即刻补齐或删除已有鸟。[原版撤销](../../../raw/current/scripts/prefabs/hats.lua#L572-L582) [后续数量检查](../../../raw/current/scripts/components/birdspawner.lua#L154-L165)

世界的自然调度、天气倍率、选鸟与落点规则统一见 [[reference/components/birdspawner#关键接口]]；本页只维护玩家来源配置，不复制刷鸟实现。

## 常见陷阱与限制

- **把差值写成目标值：** `maxbirds=7` 是基础上限再加 7，不是最终设为 7；延迟也逐键加到基础秒数。所有来源叠加后应保证最小／最大延迟非负且有序，组件没有夹取或范围校验。[加算](../../../raw/current/scripts/components/birdspawner.lua#L116-L122) [排程使用](../../../raw/current/scripts/components/birdspawner.lua#L172-L180)
- **设来源后不刷新：** 当前已排任务保留原到期时刻；修正值在下次排程时才重读。原羽毛帽显式刷新，不是 SourceModifierList 自动通知世界。[读取时机](../../../raw/current/scripts/components/birdspawner.lua#L175-L180) [原版刷新](../../../raw/current/scripts/prefabs/hats.lua#L565-L567)
- **吸引能越过禁刷鸟／夜晚：** 自然排程先要求非夜晚且基础 `_maxbirds>0`；玩家差值不能把基础禁用重新打开。直接生成是另一条路径，不能据此推导全世界不可能再出现鸟。[全局门禁](../../../raw/current/scripts/components/birdspawner.lua#L191-L213)
- **附近数量是玩家独占配额：** 生成器在玩家周围半径 64 统计带 bird 标签的实体，不是只统计该玩家曾生成的鸟；多人区域可重叠，实际生成仍受落点资格限制。[计数](../../../raw/current/scripts/components/birdspawner.lua#L154-L161)
- **来源自动消失就立即恢复排程：** 实体来源的 onremove 只重算来源表；如需马上应用撤销，应在宿主退出流程显式移除并刷新，不依赖整组件卸载。组件没有保存来源，读档需由装备／效果重新登记。[来源清理](../../../raw/current/scripts/util/sourcemodifierlist.lua#L84-L91) [原版卸装](../../../raw/current/scripts/prefabs/hats.lua#L572-L582)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 9 处（tuning BIRD_SPAWN 默认段 +4、player_common 端界／挂载段 -14~-12、hats.lua 羽毛帽段 -1）；无结论修订，仅锚点迁移——BIRD_SPAWN_MAX=4、延迟 5～15 秒、羽毛帽差值 3/-3/-5 均未变（tuning-hunk-map 5 改值键不含），birdattractor.lua、birdspawner.lua、sourcemodifierlist.lua 未变。依据：tuning-hunk-map 键级审阅＋变更清单命中 tuning.lua、player_common.lua、hats.lua（本版改动不涉羽毛帽穿戴／卸下内容）。
- 源码审阅：2026-10-24，Steam buildid 25643519；完整定义与来源表的按键累加／移除、玩家主模拟挂载、羽毛帽双向生命周期、自然鸟数与延迟消费者及当前差值常量。未审校所有装备、模组来源或天气事件全过程。
- 结构检查：本页格式、来源、本地链接与源码引用严格检查通过；本批两页共90处引用无错误／警告。无完整 Lua 示例，导航登记由主代理统一检查。
- 游戏运行：未运行；多人重叠计数、来源销毁与刷新时序、读档装备重挂和异常延迟值未实测。

相关：[[reference/components/birdspawner]]、[[reference/components/equippable]]。
