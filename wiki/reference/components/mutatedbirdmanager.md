---
title: "mutatedbirdmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "空壳弃用组件：只剩类定义与弃用注释，无接口、无原版消费者；裂隙鸟群体逻辑未随版本交付。"
aliases: ["mutatedbirdmanager", "弃用变异鸟管理器", "裂隙鸟管理器"]
tags: [component]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/mutatedbirdmanager.lua", "prefabs/forest.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# mutatedbirdmanager

## 快速判断

- 职责与适用：无可用职责。组件体只剩注释——"Deprecated and cut content! Used to contain migration and buzzard logic. Maybe some day rift birds will get their swarm logic."，迁移与秃鹫逻辑已删，裂隙鸟群体逻辑未实现。[组件全文](../../../raw/current/scripts/components/mutatedbirdmanager.lua#L4-L8)
- 优先使用：无。原版唯一挂载点已被注释：forest.lua 里 `--inst:AddComponent("mutatedbirdmanager") -- NOTES(OMAR): This component is deprecated.`，实际挂载的是相邻的 migrationmanager 与 mutatedbuzzardmanager。[挂载现场](../../../raw/current/scripts/prefabs/forest.lua#L739-L741)
- 重要副作用：无。构造函数不注册字段、事件或更新。
- 修改边界：全库 grep 无 `AddComponent("mutatedbirdmanager")`，也无 `components.mutatedbirdmanager` 读取者；本页仅作考古登记。

## 常见陷阱与限制

- **以为它负责裂隙鸟行为：** 当前快照中组件为空壳，挂上也不会产生任何行为；现役近名组件是 mutatedbuzzardmanager（裂隙秃鹫管理）与 migrationmanager（候鸟迁移），均为独立实现，不在本组件内。[弃用挂载](../../../raw/current/scripts/prefabs/forest.lua#L739-L741)
- **从旧模组照抄挂载代码：** 旧模组若仍写 `AddComponent("mutatedbirdmanager")`，新版本不报错但完全无效；需要同类世界级鸟群管理时应参照 mutatedbuzzardmanager 的现役实现重新实现。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（8 行）与 forest.lua 挂载区，grep 全库确认无 AddComponent 消费者、无字段读取者。未审阅 mutatedbuzzardmanager／migrationmanager 内部实现（不属于本组件范围）。
- 游戏运行：未运行。
