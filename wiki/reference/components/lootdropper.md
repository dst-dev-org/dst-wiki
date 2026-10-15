---
title: "lootdropper"
created: "2026-10-12"
updated: "2026-10-28"
type: component
summary: "组合固定、概率、加权与配方返还掉落，区分生成名单和实际散落实体。"
aliases: ["额外掉落", "概率掉落", "掉落表", "拆除返材料", "死亡掉落", "lootdropper"]
tags: [component, loot, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/lootdropper.lua", "prefabs/rocks.lua", "prefabs/treasurechest.lua", "entityscript.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# lootdropper

## 快速判断

- 职责与适用：保存多来源掉落规则，生成 Prefab 名单或生成地面实体；组件自身不监听死亡自动掉物。[lootdropper.lua:1](../../../raw/current/scripts/components/lootdropper.lua#L1 [entityscript.lua:2181](../../../raw/current/scripts/entityscript.lua#L2181
- 优先使用：增补一项用 `AddChanceLoot`；预览名单用 `GenerateLoot`，实际散落用 `DropLoot`，只生成一件用 `SpawnLootPrefab`。[lootdropper.lua:68](../../../raw/current/scripts/components/lootdropper.lua#L68 [lootdropper.lua:186](../../../raw/current/scripts/components/lootdropper.lua#L186 [lootdropper.lua:382](../../../raw/current/scripts/components/lootdropper.lua#L382
- 重要副作用：SetLoot 只清部分来源；默认还可能追加共享表／配方返还／烧毁木炭，DropLoot 再做烧焦转换与节日附加。[lootdropper.lua:30](../../../raw/current/scripts/components/lootdropper.lua#L30 [lootdropper.lua:242](../../../raw/current/scripts/components/lootdropper.lua#L242 [lootdropper.lua:415](../../../raw/current/scripts/components/lootdropper.lua#L415
- 修改边界：服务端配置；实际触发点来自目标工作回调／死亡流程。改变名单不会自动增加触发点或处理容器内容。[rocks.lua:168](../../../raw/current/scripts/prefabs/rocks.lua#L168 [treasurechest.lua:53](../../../raw/current/scripts/prefabs/treasurechest.lua#L53

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetLoot(prefabs)` | Prefab 名数组 | 替换固定名单，清本地 chance／random／抽取数；不清共享表、setup、fallback 或 recipe。[lootdropper.lua:30](../../../raw/current/scripts/components/lootdropper.lua#L30 |
| `AddChanceLoot(prefab, chance)` | 数值概率 | 每条独立测试，≥1 必加；小于 1 的概率经幸运来源处理，不是互斥加权。[lootdropper.lua:68](../../../raw/current/scripts/components/lootdropper.lua#L68 [lootdropper.lua:173](../../../raw/current/scripts/components/lootdropper.lua#L173 |
| `SetChanceLootTable(name)` | 全局共享表名／nil | 使用 LootTables[name]；nil 清引用。更改共享表会影响所有引用者。[lootdropper.lua:22](../../../raw/current/scripts/components/lootdropper.lua#L22 |
| `AddRandomLoot(prefab, weight)`＋`numrandomloot` | 权重＋抽取数 | 需设置抽取数；每次抽取可重复，同一 prefab 可出现多份；被作祟时可能换 haunted 表。[lootdropper.lua:41](../../../raw/current/scripts/components/lootdropper.lua#L41 [lootdropper.lua:87](../../../raw/current/scripts/components/lootdropper.lua#L87 [lootdropper.lua:193](../../../raw/current/scripts/components/lootdropper.lua#L193 |
| `SetLootSetupFn(fn)` | `(lootdropper)` | 每次 GenerateLoot 先调用，不是 fn(inst)；反复 append 会累积。[lootdropper.lua:37](../../../raw/current/scripts/components/lootdropper.lua#L37 [lootdropper.lua:189](../../../raw/current/scripts/components/lootdropper.lua#L189 |
| `droprecipeloot` | 默认 true | 按 AllRecipes[inst.prefab] 返材料，含 HAMMER 比例／耐久／烧毁差异；并非按真实建造所用配方追溯。[lootdropper.lua:114](../../../raw/current/scripts/components/lootdropper.lua#L114 [lootdropper.lua:242](../../../raw/current/scripts/components/lootdropper.lua#L242 |
| `GenerateLoot()` | Prefab 名数组 | 执行 setup 和随机，可能改变内部标志，不是无副作用的完整最终掉落预览。[lootdropper.lua:186](../../../raw/current/scripts/components/lootdropper.lua#L186 |
| `DropLoot(pt?, prefabs?)` | 无显式返回 | 不传名单时 GenerateLoot；传入名单仍转换烧焦／追加节日掉落，可能原地修改此数组。[lootdropper.lua:415](../../../raw/current/scripts/components/lootdropper.lua#L415 |
| `SpawnLootPrefab(prefab, pt?, …)` | 新实体／失败 nil | 抛落、继承湿度；向掉物发 on_loot_dropped，向掉落源发 loot_prefab_spawned；controlled burn 可使掉物冒烟。26148036 起 FlingItem 新增 `nofling` 字段可整体跳过抛掷物理，落点传送失败时回退原位上方传送，正常抛落行为不变。[lootdropper.lua:382](../../../raw/current/scripts/components/lootdropper.lua#L382 [抛落开关](../../../raw/current/scripts/components/lootdropper.lua#L335) |

## 典型接入

### 普通岩石挖完额外掉一份金块并保留原掉落

目标模组 `modmain.lua`；仅 `rock1`，沿用现有资源和最终工作分支。原版通过共享 rock1 表生成普通掉落，最终 onwork 调 DropLoot；所以增补本地必掉条目即可，不再包完成回调。[rocks.lua:322](../../../raw/current/scripts/prefabs/rocks.lua#L322 [rocks.lua:168](../../../raw/current/scripts/prefabs/rocks.lua#L168 [modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593

```lua
AddPrefabPostInit("rock1", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.lootdropper ~= nil then
        inst.components.lootdropper:AddChanceLoot("goldnugget", 1)
    end
end)
```

一条必掉记录在每次 GenerateLoot 添加一个名字，原 DropLoot 负责实际生成；不另掉一次、不覆盖原共享表。工厂初始化与 PostInit 在重建实例时重新配置；停用后新／重建实例不再添加。替换目标前检查 setup、升级或其他模组是否随后 SetLoot 清本地条目；“必加名单”不保证 SpawnPrefab 成功或免除烧焦转换。[lootdropper.lua:204](../../../raw/current/scripts/components/lootdropper.lua#L204 [lootdropper.lua:384](../../../raw/current/scripts/components/lootdropper.lua#L384

## 常见陷阱与限制

- **SetLoot({}) 清空全部掉落**：共享表、setup、fallback、配方返还与烧毁／节日附加仍需分别审阅。ClearChanceLoot 只清本地 chance 和 fallback，不清共享引用。[lootdropper.lua:30](../../../raw/current/scripts/components/lootdropper.lua#L30 [lootdropper.lua:82](../../../raw/current/scripts/components/lootdropper.lua#L82
- **fallback 表示这次一件都没掉**：droppingchanceloot 不在每次 GenerateLoot 重置，且≥1 条目不置位；不可用它判定本次总名单为空。[lootdropper.lua:202](../../../raw/current/scripts/components/lootdropper.lua#L202 [lootdropper.lua:229](../../../raw/current/scripts/components/lootdropper.lua#L229
- **生物加组件后死亡一定掉**：还需现有死亡链调用 DropDeathLoot／DropLoot；尸体模式可能先存名单。本页仅核验宿主入口，不提供全生物 SG／尸体流程。[entityscript.lua:2181](../../../raw/current/scripts/entityscript.lua#L2181
- **拆容器的内容也在掉落表里**：木箱另调 container:DropEverything 后 Remove；这是完成回调协作，不能重复放进固定名单。[treasurechest.lua:53](../../../raw/current/scripts/prefabs/treasurechest.lua#L53

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 4 处（lootdropper SpawnLootPrefab 两处与 DropLoot、entityscript DropDeathLoot）；结论补充 1 处：FlingItem 26148036 起新增 `nofling` 抛掷开关与落点传送失败回退（lootdropper.lua:335、:352-376），本页"抛落、继承湿度、事件推送"结论在默认配置下不变。依据：变更清单命中 components/lootdropper.lua（改动均在 FlingItem）＋全局枢纽 diff 审阅（entityscript 距离函数改动不涉死亡掉落入口）。
- 源码审阅：2026-10-12，Steam buildid 25643519；覆盖上述名单来源／生成顺序、掉落实体／事件、烧焦与节日、岩石／木箱消费者和死亡宿主入口。完整节日配置、所有幸运来源、尸体状态图与物理轨迹未全量核验。
- 结构检查：元数据、证据行、链接／章节、大小与登记通过；发布示例及岩石组合经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；实体实际落点、掉落次数、特殊活动与模组冲突待实测。

相关：[[reference/components/workable#工作顺序与原版完成位置]]；[[reference/components/container#关键接口]]；[[reference/components/builder#关键接口]]。
