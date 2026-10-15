---
title: "deployable"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "把物品部署到世界，配置合法位置、预览与部署后的物品消费。"
aliases: ["部署物品", "放置墙体", "部署间距", "可放置道具", "deployable"]
tags: ["component", "item", "building", "callback", "server-client"]
dst_build: "steam-buildid:26230496"
sources: ["actions.lua", "components/deployable.lua", "components/inventoryitem_replica.lua", "constants.lua", "prefabs/walls.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# deployable

## 快速判断

- 职责与适用：物品部署资格／位置及回调，不等同配方建造；新增世界实体由 ondeploy 创建。[components/deployable.lua:104](../../../raw/current/scripts/components/deployable.lua#L104 [prefabs/walls.lua:150](../../../raw/current/scripts/prefabs/walls.lua#L150
- 优先使用：SetDeployMode／SetDeploySpacing 用原版枚举；ondeploy 实现结果、位置和材料消费。[components/deployable.lua:64](../../../raw/current/scripts/components/deployable.lua#L64
- 重要副作用：普通 DEPLOY 先从 inventory／container 取一件，再调用 Deploy；组件自身不自动 Remove，且忽略 ondeploy 的返回值。[actions.lua:1625](../../../raw/current/scripts/actions.lua#L1625 [components/deployable.lua:132](../../../raw/current/scripts/components/deployable.lua#L132
- 修改边界：服务端权威位置检查；CUSTOM 的 inst._custom_candeploy_fn 需双方同规则，客户端通过 inventoryitem replica。[components/inventoryitem_replica.lua:327](../../../raw/current/scripts/components/inventoryitem_replica.lua#L327

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `SetDeployMode(mode)`／`SetDeploySpacing(spacing)` | DEPLOYMODE／DEPLOYSPACING 枚举 | 影响合法地图／距离及 replica；不要用未定义数字代替枚举。[components/deployable.lua:64](../../../raw/current/scripts/components/deployable.lua#L64 [constants.lua:2215](../../../raw/current/scripts/constants.lua#L2215 [constants.lua:2236](../../../raw/current/scripts/constants.lua#L2236 |
| `CanDeploy(pt, mouseover?, deployer?, rot?)` | boolean, reason? | 根据模式分派；ANYWHERE 仍检查地图可通过条件，并非任意地形。[components/deployable.lua:104](../../../raw/current/scripts/components/deployable.lua#L104 |
| `inst._custom_candeploy_fn` | `(inst, pt, mouseover, deployer, rot)` | CUSTOM 使用；缺失回退旧 MAST 逻辑；不要只写 deployable.candeployfn。[components/deployable.lua:123](../../../raw/current/scripts/components/deployable.lua#L123 |
| `ondeploy` | `(inst, pt, deployer, rot)` | rot 默认 0；回调负责生成／消费，返回 false 不会让 Deploy 报失败。[components/deployable.lua:132](../../../raw/current/scripts/components/deployable.lua#L132 |
| `keep_in_inventory_on_deploy` | 可选标志 | true 时动作不预先取出物品，回调自行定义消费和归属边界。[actions.lua:1622](../../../raw/current/scripts/actions.lua#L1622 |

## 典型接入

### 将原版石墙物品部署间距设为原版 NONE 档位

目标模组 `modmain.lua`；沿用 wall_stone_item 的 placer、墙体资源与 ondeploywall，不改地图校验模式。原版部署成功后 Get():Remove 消耗一件；工厂与名称见 [prefabs/walls.lua:150](../../../raw/current/scripts/prefabs/walls.lua#L150 和 [prefabs/walls.lua:378](../../../raw/current/scripts/prefabs/walls.lua#L378)。

```lua
AddPrefabPostInit("wall_stone_item", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.deployable ~= nil then
        inst.components.deployable:SetDeploySpacing(GLOBAL.DEPLOYSPACING.NONE)
    end
end)
```

此例演示枚举配置，WALL 模式仍由地图墙体规则验证，不能据间距 NONE 承诺重叠放置；停用后新实例恢复原间距。新增部署物还需完整 Prefab 依赖、placer／预览与结果资源，这里不提供未核验工厂。

## 常见陷阱与限制

- **ondeploy 返回 false 就返还材料**：Deploy 忽略它并返回 true；失败必须在 CanDeploy 前置校验或设计独立事务，不能只返回 false。[components/deployable.lua:138](../../../raw/current/scripts/components/deployable.lua#L138
- **加 deployable 自动消耗**：正常动作先取出一件，但未删除，成功回调仍要消费；特殊 keep_in_inventory 分支连取出也没有。[actions.lua:1622](../../../raw/current/scripts/actions.lua#L1622 [prefabs/walls.lua:159](../../../raw/current/scripts/prefabs/walls.lua#L159
- **覆写 CanDeploy 永远 true**：绕过骑乘、海陆、权限与占位边界；用合法模式／CUSTOM，双方预览一致但最终以服务器验证为准。[components/deployable.lua:89](../../../raw/current/scripts/components/deployable.lua#L89 [components/inventoryitem_replica.lua:347](../../../raw/current/scripts/components/inventoryitem_replica.lua#L347

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 漂移文件 actions.lua、constants.lua 对照 build-26148036 更新报告（R-01、R-06）逐条核验，锚点重定位 5 处、结论修订 0 处（DEPLOY 动作拆件流程未变；constants 漂移为 CONTROL_CUSTOM_START 上移、WORLD_SPECIAL_EVENT 复位与新增 VIRTUALROOMSETS 等，DEPLOYMODE／DEPLOYSPACING 枚举未动）；未覆盖：运行验证。

- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：DEPLOY 拆出一件、CanDeploy 分派、ondeploy 返回值与消费责任、石墙间距示例，依据见上文对应锚点；未交付新部署物的完整工厂与双端预览。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；本页 Lua 代码经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

相关：[[reference/components/stackable]]；[[reference/components/inventoryitem]]；[[reference/components/container]]。
