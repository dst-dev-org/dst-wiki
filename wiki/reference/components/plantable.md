---
title: "plantable"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "为旧农场提供种子的产物与基准时长，区分现代 farmplantable、地面部署及实际消费。"
aliases: ["旧农场种子产物", "胡萝卜种子旧农场种南瓜", "旧种子生长时长", "plantable"]
tags: [component, item, data, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/plantable.lua", "components/grower.lua", "prefabs/seeds.lua", "prefabs/veggies.lua", "prefabs/plant_normal.lua", "componentactions.lua", "actions.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# plantable

## 快速判断

- 职责与适用：旧农场种子侧的数据组件，只有 growtime／product；没有 Plant、部署、消费或成长调度方法。[完整组件](../../../raw/current/scripts/components/plantable.lua#L1-L7)
- 优先使用：改旧农场产物设 product，改其基准秒数设 growtime；执行由 [[reference/components/grower]] 消费字段。现代土堆播种用 [[reference/components/farmplantable]]。[消费者](../../../raw/current/scripts/components/grower.lua#L110-L142)
- 重要副作用：product 指最终产物，不是农田植物 Prefab；grower 固定生成 plant_normal，再将产物和 growtime*growrate 交给 crop。传现代 farm_plant 名称不是同一种语义。[消费者](../../../raw/current/scripts/components/grower.lua#L117-L135)
- 修改边界：本页服务端旧路径。种子上仍同时保留 plantable 和 farmplantable，改前者不改变后者及现行地面部署。[具名种子双路径](../../../raw/current/scripts/prefabs/veggies.lua#L514-L526) [现行部署](../../../raw/current/scripts/prefabs/veggies.lua#L187-L202)

## 关键接口

| 字段 | 默认／消费者约定 | 条件、作用与证据 |
|---|---|---|
| `growtime` | 默认 120 秒 | grower 以 growtime*growrate 作为单株基准时长；应为正数，不是速率或游戏天数。[默认](../../../raw/current/scripts/components/plantable.lua#L1-L5) [使用](../../../raw/current/scripts/components/grower.lua#L128-L132) |
| `product` | 默认 nil；可产物名或 `(item) -> 产物名` | grower 中函数只接 item；非函数值为真值时直接采用，否则回退 item.prefab。回调自身返回 nil 不再回退；须保证后续产物及外观资源有效。[完整选择分支](../../../raw/current/scripts/components/grower.lua#L117-L122) [原作物显示](../../../raw/current/scripts/prefabs/plant_normal.lua#L30-L44) |

组件本身无存读、生命周期清理或网络属性实现；原普通种子用 pickproduct 函数，具名种子直接用蔬菜名，growtime 都由宿主设为 TUNING.SEEDS_GROW_TIME。候选由目标 fertile／fullfertile 标签决定，再走 ACTIONS.PLANT 的取一颗／失败还回流程；不是 plantable 自己删除一颗。[普通种子](../../../raw/current/scripts/prefabs/seeds.lua#L170-L173) [具名种子](../../../raw/current/scripts/prefabs/veggies.lua#L517-L520) [候选](../../../raw/current/scripts/componentactions.lua#L1542-L1546) [动作](../../../raw/current/scripts/actions.lua#L2632-L2649)

## 典型接入

### 胡萝卜种子在旧农场产南瓜，现代播种仍保持胡萝卜

放目标模组 `modmain.lua`，无需新资源；只改 carrot_seeds 的 plantable.product，适用于已有旧农场的项目。旧 plant_normal 工厂声明蔬菜产物依赖并用产物资源显示；不将 product 改成 farm_plant_pumpkin。[具名种子产物](../../../raw/current/scripts/prefabs/veggies.lua#L514-L520) [种子注册](../../../raw/current/scripts/prefabs/veggies.lua#L1087) [原作物产物依赖](../../../raw/current/scripts/prefabs/plant_normal.lua#L18-L20) [PostInit](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("carrot_seeds", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local plantable = inst.components.plantable
    if plantable ~= nil then
        plantable.product = "pumpkin"
    end
end)
```

仅后续实际由 grower:PlantItem 读取该字段时生效；现代 farmplantable.plant 与地面 OnDeploy 仍用原胡萝卜植物。名称／图标／食用配置也未改，这不是完整新种子工程。旧农场已经种出的作物保存自己的产物，不随种子字段追溯变更；停用并重新初始化恢复原种子配置，不回滚现有作物。

原 veggies 中旧 plant_normal_ground／crop:StartGrowing 部署代码位于注释块，现行 OnDeploy 已使用 farmplantable。即使注释或依赖列表还能搜索命中，也不能当当前调用关系。[实际分支与注释](../../../raw/current/scripts/prefabs/veggies.lua#L187-L202) 普通种子现行部署同样生成现代随机种子植物。[普通种子部署](../../../raw/current/scripts/prefabs/seeds.lua#L131-L137)

## 常见陷阱与限制

- **给物品加 plantable 就能到处种植**：它只有两个字段；需要实际消费者、目标资格、动作与资源组装。grower 甚至会删除整个传入实体，正常拆分归动作层。[组件](../../../raw/current/scripts/components/plantable.lua#L1-L7) [消费](../../../raw/current/scripts/components/grower.lua#L139-L146)
- **把 product 当作要生成的植物对象名**：grower 固定创建 plant_normal，product 用于其 crop 的最终产物；与 farmplantable.plant 分开设置。[生成分工](../../../raw/current/scripts/components/grower.lua#L117-L135)
- **改 growtime 能影响现代植物或已种作物**：消费者在种植时取值，现代与存档中的 crop 各有自己的数据／调用链；按需求选入口。[使用时机](../../../raw/current/scripts/components/grower.lua#L128-L135) 单株机制见 [[reference/components/crop]]。

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对本页 sources 变更文件 actions.lua、componentactions.lua（R-01/R-04）：PLANT 动作与 plantable 候选（componentactions.lua:1542 未移位）未被本次 hunk 触及，结论无修订；锚点重定位 1 处（PLANT.fn +3 行）；未覆盖：运行验证。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖两字段默认值、grower 的产物／时长解释、普通／具名种子安装、动作拆分／还回、实际部署与注释边界，以及胡萝卜种子旧路径产南瓜例。未审校所有产物资源、旧农场恢复工程或模组自定义消费者。
- 结构检查：格式、来源、链接／行号、大小与登记通过；完整示例通过已有 Full Moon Lua 5.1 syntax-only，不证明旧农场或现代种植实际效果。
- 游戏运行：未运行；旧设施取得、不同播种路径、堆叠／最后一颗、存读、南瓜外观与其他模组产物覆盖待实测。
