---
title: "stewer"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "管理厨锅材料结算、料理计时、锅内腐败与收获，区别食谱注册和直接烤制。"
aliases: ["厨锅时间减半", "锅内料理", "烹饪配方", "stewer"]
tags: [component, food, recipe, building, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/stewer.lua", "cooking.lua", "containers.lua", "prefabs/cookpot.lua", "prefabs/portablecookpot.lua", "actions.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# stewer

## 快速判断

- 职责与适用：挂在厨锅宿主，按材料与 `inst.prefab` 对应的食谱算产物，等待后收获；不承担食谱注册。直接烤制分别见 [[reference/components/cooker]]／[[reference/components/cookable]]。[开始](../../../raw/current/scripts/components/stewer.lua#L130-L176)
- 优先使用：仅调新开锅时长用 `cooktimemult`；正常交互保留 COOK／Harvest 与宿主回调。新锅不能只改 Prefab 名而不配置对应食谱与容器。[时间公式](../../../raw/current/scripts/components/stewer.lua#L167-L176)／[食谱按锅名取](../../../raw/current/scripts/cooking.lua#L233-L235)
- 重要副作用：开始时销毁全部材料并关容器；完成后进入锅内腐败计时，收获才生成食物。宿主 onspoil 必须实际处理变质产物。[耗料](../../../raw/current/scripts/components/stewer.lua#L174-L176)／[完成](../../../raw/current/scripts/components/stewer.lua#L75-L99)／[原版变质](../../../raw/current/scripts/prefabs/cookpot.lua#L128-L132)
- 修改边界：服务端；CanCook 只查容器满，StartCooking 不自行完整检查满格／未完成。原版动作还处理正在烹饪与其他玩家占用，不应裸用 CanCook 当网络请求门禁。[判断](../../../raw/current/scripts/components/stewer.lua#L122-L131)／[动作](../../../raw/current/scripts/actions.lua#L2041-L2052)

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `cooktimemult` | 默认 1，倍数 | 新一锅时长=`TUNING.BASE_COOK_TIME * 食谱cooktime * 倍率`，不是秒数；原版便携锅也赋此字段。[默认](../../../raw/current/scripts/components/stewer.lua#L34)／[公式](../../../raw/current/scripts/components/stewer.lua#L167)／[便携锅](../../../raw/current/scripts/prefabs/portablecookpot.lua#L280-L281) |
| `CanCook()`／`StartCooking(doer)` | 满格判断／启动 | 材料每个占用槽记一个 Prefab，不按堆叠数展开；调用 cooking.CalculateRecipe 后立即读取对应食谱。[材料与配方](../../../raw/current/scripts/components/stewer.lua#L122-L148) |
| `IsCooking()`／`IsDone()`／`IsSpoiling()` | 状态判断，部分可 nil | targettime 被烹饪／变质共用，要结合 done；剩余秒数用 GetTimeToCook／GetTimeToSpoil。[状态与时间](../../../raw/current/scripts/components/stewer.lua#L102-L123) |
| `keepspoilage` | 默认未赋值；可为 true | 可腐败原料新鲜度平均值，默认恢复损失的一半；true 保留平均比例，并非禁用锅内腐败。[计算](../../../raw/current/scripts/components/stewer.lua#L150-L164) |
| 宿主回调 | `onstartcooking`、`ondonecooking`、`onspoil`、`onharvest` 等 | 原版配置动画／灯光／变质；donecooking 在 done=true 前，onharvest 在生成／清状态前。[原版组装](../../../raw/current/scripts/prefabs/cookpot.lua#L309-L315)／[完成顺序](../../../raw/current/scripts/components/stewer.lua#L75-L99)／[收获](../../../raw/current/scripts/components/stewer.lua#L276-L284) |
| `Harvest(harvester)` | done 时处理，返回 true | 生成、设置新鲜度、给物品或弹出，清状态并重新开放容器；true 不保证 SpawnPrefab 成功。[收获](../../../raw/current/scripts/components/stewer.lua#L276-L328) |
| `StopCooking(reason)` | 取消并清状态 | 不返还原料，也不负责重新开放容器；reason="fire" 另有生成产物路径，不能当通用安全取消按钮。[停止](../../../raw/current/scripts/components/stewer.lua#L184-L201) |

## 典型接入

### 普通厨锅后续每锅烹饪时间减半

目标模组 `modmain.lua`，复用 cookpot 的食谱、容器、资源与回调，只改时长倍率；原版锅装配和注册名已核验。[组装](../../../raw/current/scripts/prefabs/cookpot.lua#L309-L322)／[注册](../../../raw/current/scripts/prefabs/cookpot.lua#L359)／[PostInit](../../../raw/current/scripts/modutil.lua#L593)

```lua
AddPrefabPostInit("cookpot", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.stewer ~= nil then
        inst.components.stewer.cooktimemult = 0.5
    end
end)
```

是相对基础公式乘 0.5，不承诺叠乘其他模组倍率；不改变料理种类、锅内保质期与收获。仅在 StartCooking 新建任务时生效；已有任务和存档恢复剩余时间不按该字段重算。停用后后续新锅恢复原倍率，已有保存计时照恢复路径执行。[新建任务](../../../raw/current/scripts/components/stewer.lua#L167-L176)／[保存与恢复](../../../raw/current/scripts/components/stewer.lua#L203-L252)

## 常见陷阱与限制

- **StartCooking 自带完整门禁：** 它只检查 targettime 与 container，不检查 CanCook 或 done。自定义入口须保留服务器状态、占用与材料检查；原版容器仅接受 cooking 原料且不收堆叠。[入口](../../../raw/current/scripts/components/stewer.lua#L130-L148)／[动作](../../../raw/current/scripts/actions.lua#L2041-L2052)／[容器](../../../raw/current/scripts/containers.lua#L369-L385)
- **只加 stewer 就能新造厨锅：** recipe 按锅 Prefab 分表，候选测试和优先级由 cooking 计算；缺有效食谱会在开始流程读取条目时出错。本批未提供料理注册 API 或新锅骨架，不能把 cookable.product 当食谱注册。[查表](../../../raw/current/scripts/cooking.lua#L233-L262)／[选方](../../../raw/current/scripts/cooking.lua#L268-L284)／[直接读取](../../../raw/current/scripts/components/stewer.lua#L147-L148)
- **设 spoiledproduct 就会自动换腐烂物：** dospoil 只清计时并调用 onspoil；原版 cookpot 的 spoilfn 明确赋 product=spoiledproduct。替换回调时须保留真实状态处理。[回调](../../../raw/current/scripts/components/stewer.lua#L65-L72)／[宿主赋值](../../../raw/current/scripts/prefabs/cookpot.lua#L128-L132)
- **移除组件就完成停机清理：** OnRemoveFromEntity 只移标签，不取消计时或恢复容器；不要把 RemoveComponent 当取消烹饪。LongUpdate 还能结算煮熟后的剩余腐败时间。[移除](../../../raw/current/scripts/components/stewer.lua#L59-L63)／[离屏](../../../raw/current/scripts/components/stewer.lua#L332-L357)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：sources 变更文件 actions.lua／containers.lua 对照更新报告 R-01/R-07 与 hunk 地图逐 hunk 核验。锚点重定位 3 处（COOK 的 stewer 分支两处移至 actions.lua:2038-2049，+3 行移；containers cookpot 参数区 :369-385 行号不变）；COOK 动作与 cookpot 容器过滤无语义变化，R-07 改动全在 socket_keystone／quagmire_pot 段不触及本页。未覆盖：运行验证。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖普通锅材料／配方消费、开始／完成／变质／收获、倍率／新鲜度与保存恢复、回调分工及半时长示例。料理注册、全部角色加成和专属锅未完整核验。
- 结构／示例检查：本批格式、来源文件、链接／行锚点、导航登记及完整 Lua 5.1 示例语法检查通过。
- 游戏运行：未运行；菜单与多人占用、计时／读档、锅内变质、收获与跨模组倍率需验证。
