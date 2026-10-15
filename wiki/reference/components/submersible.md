---
title: "submersible"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "物品沉水组件：可沉物品在沉船/落地海面时转为 underwater_salvageable 沉底物，并在浅水/近岸自动重定位或爬回陆地。"
aliases: ["沉水", "submersible", "沉底物品", "underwater_salvageable 转换"]
tags: [component, item, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/submersible.lua", "prefabs/winch.lua", "components/winchtarget.lua", "prefabs/treasurechest.lua", "prefabs/veggies.lua", "prefabs/archive_resonator.lua", "components/retrofitforestmap_anr.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# submersible

## 快速判断

- 职责与适用：给"沉入海里"的可交互物品（沉宝箱、巨型作物、棋子残片、月坛碎片、玻璃尖刺、贝壳簇、土豆袋、雪人、海树果、cavein_boulder 等）安装；触发源是 `onsink`（沉船）与 `on_landed`（落海面）事件。[构造监听](../../../raw/current/scripts/components/submersible.lua#L42-L49)
- 优先使用：主动让物品沉底用 `MakeSunken(x, z)`（原版绞盘卸货即此路径）；物品自然落海靠 `OnLanded` 自动触发。[沉底](../../../raw/current/scripts/components/submersible.lua#L181-L195) [绞盘](../../../raw/current/scripts/prefabs/winch.lua#L367-L372)
- 重要副作用：`Submerge` 可能改变物品位置——周围是浅水/近岸时先尝试搬到纯水点，无解则带上 `splash_green` 搬到陆地随机可行走点；沉底后物品被装进新建的 `underwater_salvageable` 容器实体并推 `on_submerge`。[重定位](../../../raw/current/scripts/components/submersible.lua#L84-L179) [装罐](../../../raw/current/scripts/components/submersible.lua#L185-L192)
- 修改边界：依赖 `inventoryitem`（GetContainer／owner 判定）与世界 Map 的海洋判定；`force_no_repositioning` 为 true 时跳过重定位搜索直接沉在原地。[门禁](../../../raw/current/scripts/components/submersible.lua#L101)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GetUnderwaterObject()` | `-> container 实体或 nil` | 仅当物品在容器里且该容器带 `underwater_salvageable` 标签时返回容器（即"已沉底"判定）。[实现](../../../raw/current/scripts/components/submersible.lua#L56-L71) [消费者](../../../raw/current/scripts/prefabs/archive_resonator.lua#L135) |
| `Submerge()` | `-> has_moved: boolean` | 主入口：已沉底则直接返回；先判 `IsSurroundedByWater(MAX_WALKABLE_PLATFORM_RADIUS+0.2)`，不满足且 `force_no_repositioning` 为 false 时按 6 间距邻点搜水点/陆地并可能移动，最后 `MakeSunken` 或搬陆地。[实现](../../../raw/current/scripts/components/submersible.lua#L84-L179) [邻点扫描](../../../raw/current/scripts/components/submersible.lua#L22-L40) |
| `MakeSunken(x, z, ignore_boats?, nosplash?)` | `仅海面有效` | 生成 `underwater_salvageable`、GiveItem 装入、推 `on_submerge`、默认带 `splash_green`；非海面坐标静默无效。绞盘卸货用 `ignore_boats=true, nosplash=true`。[实现](../../../raw/current/scripts/components/submersible.lua#L185-L191) [绞盘调用](../../../raw/current/scripts/prefabs/winch.lua#L369-L371) |
| `OnLanded()` | 事件回调 | 无持有者且脚下是海面时触发 Submerge；retrofit 地图修复也用它重放旧沉物。[实现](../../../raw/current/scripts/components/submersible.lua#L73-L82) [retrofit](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L532-L533) |
| `force_no_repositioning` | `boolean，存读字段` | 从船上沉下时自动置 true（位置必可达）；`winchtarget` 打捞后复位为 false。[置位](../../../raw/current/scripts/components/submersible.lua#L1-L8) [复位](../../../raw/current/scripts/components/winchtarget.lua#L21-L22) |
| `OnSave`／`OnLoad` | 标准存读钩子 | 只持久化 `force_no_repositioning`。[实现](../../../raw/current/scripts/components/submersible.lua#L197-L206) |

## 典型接入

### 让自定义物品可以沉海并被打捞

- 前提与选择：物品需有 `inventoryitem`；沉底容器 `underwater_salvageable` 是公共 prefab，打捞方（绞盘/trawler）按该实体操作，本组件不定义打捞协议。[沉底实体](../../../raw/current/scripts/components/submersible.lua#L183-L193)

```lua
-- 服务端，prefab master_postinit 内
inst:AddComponent("inventoryitem")
inst:AddComponent("submersible")
inst:ListenForEvent("on_submerge", function(inst, data)
    -- data.underwater_object 是承载它的沉底容器
end)
```

- 调整与验证：原版巨型作物（veggies）与沉宝箱即此组合；想禁止"爬回陆地"行为需在 sink 场景外主动置 `force_no_repositioning`，如绞盘卸货那样成对管理。[作物](../../../raw/current/scripts/prefabs/veggies.lua#L900) [宝箱](../../../raw/current/scripts/prefabs/treasurechest.lua#L721)

## 常见陷阱与限制

- **Submerge 会挪动位置：** 在浅水/近岸落下的物品可能被搬到别处水点甚至陆地；依赖物品原坐标的逻辑要在 `on_submerge` 前记录，或用 `force_no_repositioning` 锁定。[搬移分支](../../../raw/current/scripts/components/submersible.lua#L101-L167)
- **onsink 默认强制原地沉：** 从船上沉下（`data.boat ~= nil`）时组件自设 `force_no_repositioning = true`，不会触发重定位；这与"落海"路径不同。[沉船分支](../../../raw/current/scripts/components/submersible.lua#L1-L8)
- **MakeSunken 不校验当前位置：** 传陆地坐标静默失败（非海面直接 return），不报错不回退；调用方需先确认海面。[海面门禁](../../../raw/current/scripts/components/submersible.lua#L182)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（208 行），grep 全库确认 AddComponent 挂载清单（12 个 prefab 文件）与 winch/winchtarget/archive_resonator/retrofit 调用点。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 本批 16 页通过。
- 游戏运行：未运行。
