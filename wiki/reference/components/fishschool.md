---
title: "fishschool"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "给鱼群实体维护鱼量等级，被渔网罩住时按等级生成鱼并播事件，周期性补充等级。"
aliases: ["fishschool", "鱼群等级", "渔网捕鱼群", "鱼群补充"]
tags: [component]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/fishschool.lua", "prefabs/antchovies_group.lua", "components/fishingnetvisualizer.lua", "stategraphs/SGantchovies.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# fishschool

## 快速判断

- 职责与适用：维护 `fish_level`（0～max，默认满 3）并在收到 `on_pre_net` 时按等级生成鱼；等级归零后播枯竭动画，靠周期任务补充。([fishschool.lua:29-45](../../../raw/current/scripts/components/fishschool.lua#L29-L45) ([fishschool.lua:47-56](../../../raw/current/scripts/components/fishschool.lua#L47-L56)
- 优先使用：`SetNettedPrefab(鱼prefab)` ＋ `StartReplenish(间隔秒)` 初始化；原版鳀鱼群用 10 秒。([antchovies_group.lua:35-37](../../../raw/current/scripts/prefabs/antchovies_group.lua#L35-L37)
- 重要副作用：捕获时在渔网位置逐条 SpawnPrefab 并对鱼推 `on_caught_in_net`；等级与动画状态变化全在本地实体。([fishschool.lua:32-43](../../../raw/current/scripts/components/fishschool.lua#L32-L43)
- 修改边界：`on_pre_net` 由渔网收口时对范围内实体统一推送，组件只被动响应。([fishingnetvisualizer.lua:100-107](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L100-L107)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `StartReplenish(replenish_rate)` | 间隔秒，默认 10 | 先停旧任务再建周期 Replenish。([fishschool.lua:13-16](../../../raw/current/scripts/components/fishschool.lua#L13-L16) |
| `SetNettedPrefab(fishing_net_prefab)` | 鱼 prefab 名字符串 | 未设置时被网也不生成鱼（只清零等级、播枯竭动画）。([fishschool.lua:25-27](../../../raw/current/scripts/components/fishschool.lua#L25-L27) ([fishschool.lua:34-43](../../../raw/current/scripts/components/fishschool.lua#L34-L43) |
| `Replenish()` | 无参 | 等级 +1 封顶；从 0 恢复时重播 `group_pre`＋`group_loop1`。([fishschool.lua:47-56](../../../raw/current/scripts/components/fishschool.lua#L47-L56) |
| `OnPreNet(net)` | net 为渔网实体（事件 data） | 按 fish_level 数量在网坐标生成鱼并推 `on_caught_in_net`，等级清零、播 `group_pst`。([fishschool.lua:29-45](../../../raw/current/scripts/components/fishschool.lua#L29-L45) |

## 典型接入

### 自定义鱼群实体

- 前提与选择：宿主是带 AnimState 的群体实体；鱼的"被捕获"表现由鱼 prefab 自己的 `on_caught_in_net` 处理（鳀鱼状态图转 caught_in_net）。[SGantchovies.lua:16](../../../raw/current/scripts/stategraphs/SGantchovies.lua#L16

```lua
inst:AddComponent("fishschool")
inst.components.fishschool:SetNettedPrefab("myfish")
inst.components.fishschool:StartReplenish(10)
```

- 生效与撤销：构造即播开场动画并监听事件；StopReplenish 可停补充但不复位等级。([fishschool.lua:8-11](../../../raw/current/scripts/components/fishschool.lua#L8-L11) ([fishschool.lua:18-23](../../../raw/current/scripts/components/fishschool.lua#L18-L23)

## 常见陷阱与限制

- 无存读钩子：存档重载后 fish_level 重置为满、补充任务不恢复；动画状态同样回到满员开场。([fishschool.lua:1-12](../../../raw/current/scripts/components/fishschool.lua#L1-L12)
- `on_pre_net` 的推送范围是渔网 `collect_radius + TUNING.MAX_FISH_SCHOOL_SIZE`，靠近网的群也会被触发。([fishingnetvisualizer.lua:101-104](../../../raw/current/scripts/components/fishingnetvisualizer.lua#L101-L104)
- 生成的鱼只被推事件并放在网位置，是否真的"进网"由鱼本体逻辑决定，组件不收集它们。([fishschool.lua:35-39](../../../raw/current/scripts/components/fishschool.lua#L35-L39)

## 验证记录

- 源码审阅：2026-10-28，`steam-buildid:26230496`：审阅组件全文、鳀鱼群安装参数、渔网推送点与鳀鱼状态图事件处理。
- 游戏运行：未运行；未复现捕网到补充的完整循环。
- 结构检查：validate_page.py 通过；validate_source_links.py --strict 通过（本批 16 页合并执行）。

相关：[[reference/components/fishingnetvisualizer]] · [[reference/components/fishingnet]] · [[reference/components/schoolspawner]]
