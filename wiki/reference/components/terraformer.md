---
title: "terraformer"
created: "2026-10-22"
updated: "2026-10-28"
type: component
summary: "检查铲地或犁地资格、替换地图地块，并委派地皮掉落与后续效果。"
aliases: ["terraformer", "草叉铲地", "犁地", "恢复地块", "铲除农田"]
tags: [component, item, callback, event, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/terraformer.lua", "components/map.lua", "componentutil.lua", "componentactions.lua", "actions.lua", "prefabs/pitchfork.lua", "prefabs/farm_plow.lua", "prefabs/turfs.lua", "components/undertile.lua", "components/farming_manager.lua", "components/finiteuses.lua", "bufferedaction.lua", "mainfunctions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# terraformer

## 快速判断

- 职责与适用：服务端工具或犁地实体对一个世界坐标执行地图换地块；普通草叉与农用犁均为真实宿主。[组件实现](../../../raw/current/scripts/components/terraformer.lua#L12-L48)、[草叉构造](../../../raw/current/scripts/prefabs/pitchfork.lua#L56-L83)、[农用犁构造](../../../raw/current/scripts/prefabs/farm_plow.lua#L212-L243)。
- 优先使用：修改落地地块用 `turf`；修改掉落／完成效果用 `onterraformfn`；`plow` 选择犁地门禁。铺设手中地皮走 `deployable`，不走草叉动作。[Terraform](../../../raw/current/scripts/components/terraformer.lua#L16-L32)、[地皮部署](../../../raw/current/scripts/prefabs/turfs.lua#L9-L17)。
- 重要副作用：先改地图，再调用回调；普通铲地给同格 `soil` 实体发送 `collapsesoil`，随后工具与可选执行者各收到无载荷 `onterraform`。[Terraform 后半段](../../../raw/current/scripts/components/terraformer.lua#L26-L47)。
- 修改边界：组件不自行扣耐久、消费物品或保存配置；原版在服务端添加。移除特定目标实体由 [[reference/components/terraformerremoveable]] 承接，属于另一个动作。

## 关键接口

| 接口／字段 | 签名或已核验约定 | 条件、作用与证据 |
|---|---|---|
| `Terraform(pt, doer?)` | `pt:Get()` 返回世界坐标；返回 boolean | 根据 `plow` 选择地图门禁；失败返回 false 且不改地块；成功返回 true。[实现](../../../raw/current/scripts/components/terraformer.lua#L12-L48) |
| `turf` | 可选地块 ID | 目标优先级是此字段 → `undertile:GetTileUnderneath(tile_x, tile_y)` → `WORLD_TILES.DIRT`；是地图地块 ID，不是 `turf_*` Prefab 名。[选取目标](../../../raw/current/scripts/components/terraformer.lua#L21-L26) |
| `plow` | 可选布尔标志；构造时未赋值 | 真值用 `CanPlowAtPoint`，否则用 `CanTerraformAtPoint`；前者也跳过组件末尾整格 `soil` 坍塌循环。[组件](../../../raw/current/scripts/components/terraformer.lua#L3-L40) |
| `onterraformfn` | `fn(inst, pt, old_tile, old_turf_data)` | 可选；第四参是 `GroundTiles.turf[old_tile]` 数据或 nil；替代默认 `HandleDugGround`，返回值不影响成功，也不能撤销已完成的 SetTile。[调用](../../../raw/current/scripts/components/terraformer.lua#L26-L32) |

### 动作、地图门禁与事件分工

1. POINT 右键收集以工具的 `plow` **标签**选择地图检查，再加入 `TERRAFORM`；动作执行转到服务端组件，组件读取的是 `plow` **字段**。[候选动作](../../../raw/current/scripts/componentactions.lua#L2223-L2229)、[动作执行](../../../raw/current/scripts/actions.lua#L2873-L2877)。自制手持犁地工具需让两处语义一致；原版部署后的农用犁直接调用组件，不依靠 POINT。
2. 普通铲地要求陆地且不在 `TERRAFORM_IMMUNE` 中；犁地要求 `CanPlantAtPoint`，即陆地且非 `GROUND_HARD`。两者均检查可见 `terraformblocker` 的额外间距。[地图门禁](../../../raw/current/scripts/components/map.lua#L119-L160)、[可种植地块](../../../raw/current/scripts/components/map.lua#L167-L175)。这不等同于铺地皮只接受 DIRT 的检查。[铺地门禁](../../../raw/current/scripts/components/map.lua#L162-L164)。
3. `Map:SetTile` 在调用中向**世界实体**发送 `onterraform`，payload 为 `{x, y, original_tile, tile}`，此处 x/y 是地图格坐标。组件随后发出的工具／执行者同名事件没有此表，监听对象与时机不可混用。[地图包装](../../../raw/current/scripts/components/map.lua#L3-L8)、[组件事件](../../../raw/current/scripts/components/terraformer.lua#L42-L47)。
4. 默认掉落由 `HandleDugGround` 查询原地块数据后生成 `turf_..name`，无数据时生成塌陷特效；`nospawnturf` 仅有注释，组件没有读取它的分支。[掉落处理](../../../raw/current/scripts/componentutil.lua#L31-L46)、[完整组件](../../../raw/current/scripts/components/terraformer.lua#L3-L48)。

## 典型接入

### 普通草叉铲地后固定露出裸地

已有启用的服务端模组，将以下完整接入代码放入 `modmain.lua`，沿用原版 `pitchfork` 的资源、装备、动作、默认掉落与耐久配置。所有可铲地块的落地目标均固定为 DIRT；原先农田下层是其他地块时，也不再恢复该下层。Prefab postinit 在原版工厂完成后执行。[草叉注册](../../../raw/current/scripts/prefabs/pitchfork.lua#L103-L125)、[postinit 顺序](../../../raw/current/scripts/mainfunctions.lua#L387-L399)。

```lua
AddPrefabPostInit("pitchfork", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local terraformer = inst.components.terraformer
    if terraformer ~= nil then
        terraformer.turf = GLOBAL.WORLD_TILES.DIRT
    end
end)
```

该配置影响模组启用期间创建／载入的普通草叉，不影响金草叉；不绕过原版铲地门禁。停用后重建的草叉恢复默认目标选择，已经改成 DIRT 的地图格不会自动恢复。组件没有 OnSave／OnLoad，配置每次由构造和 postinit 重建；下层地块另由 `undertile` 保存。[完整组件](../../../raw/current/scripts/components/terraformer.lua#L3-L50)、[下层存取与保存](../../../raw/current/scripts/components/undertile.lua#L41-L75)。

### 农用犁：延时换地块与道具返还由宿主管理

原版 `farm_plow` 设置 `turf=FARMING_SOIL`、`plow=true`，回调 `OnTerraform` 生成土堆碎屑并调用 Finished，而不是默认返还旧地皮；计时完成才直接 `Terraform(inst:GetPosition())`。[配置](../../../raw/current/scripts/prefabs/farm_plow.lua#L230-L243)、[回调](../../../raw/current/scripts/prefabs/farm_plow.lua#L81-L102)、[计时调用](../../../raw/current/scripts/prefabs/farm_plow.lua#L155-L165)。

部署时 `farm_plow_item` 已先扣 `finiteuses:Use(1)`，将剩余道具保存记录交给场景实体后删除道具；Finished 根据记录返还剩余物品，犁地失败也调用 Finished。[部署消费](../../../raw/current/scripts/prefabs/farm_plow.lua#L248-L259)、[返还路径](../../../raw/current/scripts/prefabs/farm_plow.lua#L48-L68)。宿主保存道具记录并在载入时恢复钻地流程，不能只复制一个 terraformer 组件就获得这套生命周期。[宿主存档](../../../raw/current/scripts/prefabs/farm_plow.lua#L175-L191)。

## 常见陷阱与限制

- **回调返回 false 取消换地**：地图已先修改，组件忽略返回值；资格判断必须放在调用前，不能用完成回调充当事务校验。[先换地后回调](../../../raw/current/scripts/components/terraformer.lua#L16-L32)。
- **指定 `turf` 等于使用一块地皮**：这里只改地图 ID；原版铺地皮另由 `turfs.lua` 的 `ondeploy` 消费一件堆叠物品，并通过 `GroundTiles.turf` 生成注册项。[部署消费](../../../raw/current/scripts/prefabs/turfs.lua#L9-L17)、[注册循环](../../../raw/current/scripts/prefabs/turfs.lua#L80-L87)。
- **直接调用自动扣草叉耐久**：扣次发生在成功的 BufferedAction `OnUsedAsItem` 链；普通草叉为 TERRAFORM 配置 `.125`，`finiteuses:SetConsumption` 同时映射 TERRAFORM_REMOVE。直接调用 `Terraform` 不执行这条动作扣次链。[成功动作](../../../raw/current/scripts/bufferedaction.lua#L22-L37)、[草叉配置](../../../raw/current/scripts/prefabs/pitchfork.lua#L63-L79)、[消耗映射](../../../raw/current/scripts/components/finiteuses.lua#L27-L34)。详见 [[reference/components/finiteuses]]。
- **农田下层永不变化**：地图世界事件由 farming_manager 消费，进入 FARMING_SOIL 时记录原地块，离开时清除下层；该记录不是 terraformer 自有字段。[消费者](../../../raw/current/scripts/components/farming_manager.lua#L153-L178)、[监听注册](../../../raw/current/scripts/components/farming_manager.lua#L612)。

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对本页 sources 变更文件 actions.lua／componentactions.lua（R-01/R-04）、componentutil.lua（R-05）、components/map.lua（C-23）、mainfunctions.lua（R-09）：TERRAFORM 候选／执行、地图门禁与 HandleDugGround 均未被本次 hunk 触及（C-23 变化全在 map.lua 923 行后，R-09 插入仅使 postinit 锚点后移），结论无修订；锚点重定位 3 处；未覆盖：运行验证。

- 源码审阅：2026-10-22，Steam buildid 25643519；覆盖完整组件、POINT／动作门禁、地块检查与世界事件、地皮掉落／铺设、下层消费者、两种宿主及存档分工；示例只改变普通草叉目标地块。
- 结构与语法：本次两页定向页面校验、源码锚点严格校验通过；本页完整 `modmain.lua` 示例通过现有 Full Moon 的 Lua 5.1 语法检查。
- 游戏运行：未运行；多人动作显示、农田恢复效果、存档重载和其他模组兼容仍需实际验证。

相关：[[reference/components/terraformerremoveable]]；[[reference/components/deployable]]；[[reference/components/finiteuses]]。

## 相关链接

- [[reference/components/undertile]]：临时地皮底层 tile 记忆网格。
