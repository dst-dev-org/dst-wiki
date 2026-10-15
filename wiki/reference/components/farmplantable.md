---
title: "farmplantable"
created: "2026-10-18"
updated: "2026-10-28"
type: component
summary: "把种子种进农田土堆，明确产物选择、单颗拆分、实体消费与直接部署的区别。"
aliases: ["种子播种", "土堆种植", "普通种子固定种胡萝卜", "farmplantable"]
tags: [component, item, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/farmplantable.lua", "actions.lua", "componentactions.lua", "components/inventory.lua", "prefabs/seeds.lua", "prefabs/veggies.lua", "prefabs/farm_plants.lua", "prefabs/farm_plant_defs.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# farmplantable

## 快速判断

- 职责与适用：种子侧的土堆播种执行器；删除 soil 目标，在原位置生成植物并消费自身。不是锄地器或完整植物成长系统。[Plant](../../../raw/current/scripts/components/farmplantable.lua#L7-L29)
- 优先使用：改土堆播种产物设 `plant`；正常人物播种走 `ACTIONS.PLANTSOIL`，由动作先从物品栏取出一颗。[动作](../../../raw/current/scripts/actions.lua#L4727-L4745) [拆分](../../../raw/current/scripts/components/inventory.lua#L1390-L1403)
- 重要副作用：成功先删土堆，再生成植物／推事件，最后删除传入种子实体；直接对整堆调用会删除整堆。SpawnPrefab 结果未做 nil 检查。[执行顺序](../../../raw/current/scripts/components/farmplantable.lua#L11-L26)
- 修改边界：本页服务端。该路径需要已有 soil；植物本体、人物地面部署与旧农场 plantable 是不同路径，不能只改一个字段便称全部播种生效。[原种子三种接入](../../../raw/current/scripts/prefabs/seeds.lua#L159-L173)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `plant` | Prefab 字符串或 `(seed) -> Prefab名/nil` | 默认 nil；FunctionOrValue 仅传种子，不传 target／planter。返回 nil 会失败，配置须指向已注册植物。[构造](../../../raw/current/scripts/components/farmplantable.lua#L2-L5) [选择](../../../raw/current/scripts/components/farmplantable.lua#L7-L15) |
| `Plant(target, planter)` | 成功 true；正常条件不满足 false | target 必须为可调用 HasTag 的实体且有 soil；不内建 target=nil、生成失败或事务回滚保护。[实现](../../../raw/current/scripts/components/farmplantable.lua#L7-L29) |
| `on_planted` | 植物事件，`{doer,seed,in_soil=true}` | 此时种子尚未删除，稍后即失效；原植物仅在 doer 有 plantkin 时设置 long_life。[发送](../../../raw/current/scripts/components/farmplantable.lua#L15-L25) [消费者](../../../raw/current/scripts/prefabs/farm_plants.lua#L693-L697) |
| `itemplanted` | 世界事件，`{doer,pos}` | 发给 TheWorld，不发给种子；源码注明其他路径也会发送，不能据事件名反推必走本组件。[发送](../../../raw/current/scripts/components/farmplantable.lua#L23-L26) |

人物候选还排除目标 NOCLICK；动作要求 invobject、人物 inventory、soil 目标，优先 quagmire_plantable，再走本组件；返回失败时尝试 GiveItem 还回拆出的种子。直接 Plant 不含这些动作门禁。[候选](../../../raw/current/scripts/componentactions.lua#L1235-L1239) [动作](../../../raw/current/scripts/actions.lua#L4727-L4745)

## 典型接入

### 普通种子种进土堆时，固定生成种植胡萝卜

放目标模组 `modmain.lua`，只改 `seeds` 的服务端 farmplantable.plant；无需新资源。原普通种子返回 farm_plant_randomseed，具名蔬菜种子则用 farm_plant_<name>。胡萝卜原植物已注册，普通种子依赖表也收集可播种蔬菜的植物名。[普通选择](../../../raw/current/scripts/prefabs/seeds.lua#L60-L62) [依赖](../../../raw/current/scripts/prefabs/seeds.lua#L29-L34) [具名种子](../../../raw/current/scripts/prefabs/veggies.lua#L514-L515) [植物名](../../../raw/current/scripts/prefabs/farm_plant_defs.lua#L254-L258) [工厂注册](../../../raw/current/scripts/prefabs/farm_plants.lua#L926-L937) [PostInit](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("seeds", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local plantable = inst.components.farmplantable
    if plantable ~= nil then
        plantable.plant = "farm_plant_carrot"
    end
end)
```

生效于该种子实体后续的土堆播种，拆分／还回仍交原动作；不额外 Remove 或手动调用 Plant。停用并重新初始化后恢复原选择，已经种出的胡萝卜不回滚。替换产物名时须同时核验该植物注册与加载依赖，不能换成仅可食用的 `carrot` 物品。

原普通种子的 plantkin 地面部署仍由 OnDeploy 直接生成随机种子植物，推 in_soil=false；本例没有改变它，也不改变具名种子或旧农场路径。直接地面部署的扩展见 [[reference/components/deployable]]；土堆生成见 [[reference/components/farmtiller]]。[地面路径](../../../raw/current/scripts/prefabs/seeds.lua#L131-L143) [部署接入](../../../raw/current/scripts/prefabs/seeds.lua#L165-L173)

## 常见陷阱与限制

- **直接对物品栏整堆 Plant**：组件删除自身而不 stackable:Get，导致整堆消费；复用动作层拆分，不再重复扣一颗。[删除](../../../raw/current/scripts/components/farmplantable.lua#L25-L26) [单颗取得](../../../raw/current/scripts/components/inventory.lua#L1395-L1403)
- **产物名无效却已删土堆**：Remove 在 SpawnPrefab 之前，随后直接访问 plant.Transform；调用前核验注册／依赖，失败不保证恢复土堆或种子。[顺序](../../../raw/current/scripts/components/farmplantable.lua#L11-L17)
- **把随机种子初始植物当最终品种**：原 seeds 先生成 farm_plant_randomseed，之后由植物工厂替换为品种或杂草，并迁移饮水／压力状态；本例跳过该随机选择，生长管理仍由原植物承担。[随机与替换](../../../raw/current/scripts/prefabs/farm_plants.lua#L134-L166)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对本页 sources 变更文件 actions.lua、componentactions.lua（R-01/R-04）、components/inventory.lua（changed-files 标 M 但两快照字节一致，无内容差异）：PLANTSOIL 候选与执行链未被本次 hunk 触及，结论无修订；锚点重定位 2 处（PLANTSOIL.fn +9 行，候选 1235 未移位）；未覆盖：运行验证。

- 源码审阅：2026-10-18，Steam buildid 25643519；覆盖字段／返回、土堆与种子删除、两个事件、动作拆分与失败还回、普通／具名种子、地面部署及随机植物替换。未审校所有植物、旧农场或饥荒活动种植系统。
- 结构检查：格式、来源、链接／行号、大小与登记通过；完整 modmain 示例通过已有 Full Moon Lua 5.1 syntax-only，不证明资源加载或实际播种成功。
- 游戏运行：未运行；主栏／背包／活动物品拆分、地面部署、plantkin 寿命、存读与其他模组产物覆盖待实测。
