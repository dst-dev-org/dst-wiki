---
title: "worldroutes"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "在世界实体上按名字缓存跨地形巡游路线点列并随存档持久化，供巡游类生物读取路线。"
aliases: ["worldroutes", "世界路线缓存", "巡游路线", "按名存取路线"]
tags: [component, data]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/worldroutes.lua", "prefabs/world.lua", "prefabs/wanderingtrader.lua", "components/worldroutefollower.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# worldroutes

## 快速判断

- 职责与适用：把"路线名 → Vector3 点列"的世界级缓存装在 `TheWorld` 上并随存档持久化；组件只做存取与存读，不算路。([worldroutes.lua:1-18](../../../raw/current/scripts/components/worldroutes.lua#L1-L18)
- 优先使用：写路线用 `SetRoute(routename, route)`，读用 `GetRoute(routename)`；巡游跟随配合 worldroutefollower 的 `FollowRoute` 取点。([worldroutes.lua:12-18](../../../raw/current/scripts/components/worldroutes.lua#L12-L18) ([worldroutefollower.lua:240-241](../../../raw/current/scripts/components/worldroutefollower.lua#L240-L241)
- 重要副作用：无事件、无更新循环；仅构造时断言与存读钩子。([worldroutes.lua:5-6](../../../raw/current/scripts/components/worldroutes.lua#L5-L6)
- 修改边界：仅主模拟、必须挂在 `TheWorld` 本体（构造双断言）；森林世界由 `master_postinit` 安装。([worldroutes.lua:5-6](../../../raw/current/scripts/components/worldroutes.lua#L5-L6) [world.lua:587](../../../raw/current/scripts/prefabs/world.lua#L587

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetRoute(routename, route)` | route 为 Vector3 数组 | 覆盖同名缓存；不校验点是否可通行。([worldroutes.lua:12-14](../../../raw/current/scripts/components/worldroutes.lua#L12-L14) |
| `GetRoute(routename)` | route 数组或 nil | 返回缓存引用，消费方可直接索引。([worldroutes.lua:16-18](../../../raw/current/scripts/components/worldroutes.lua#L16-L18) |
| `OnSave()` / `OnLoad(data)` | 保存 `{routes={[名]={{x,z}…}}}`；空返回 nil | OnLoad 重建 `Vector3(v[1], 0, v[2])`，y 归零。([worldroutes.lua:20-52](../../../raw/current/scripts/components/worldroutes.lua#L20-L52) |

## 典型接入

### 为巡游生物注册并读取自定义路线

- 前提与选择：服务端逻辑；原版路线由 wanderingtrader 用拓扑寻路或随机游走生成后写回组件，模组可直接注册自定义点列。[wanderingtrader.lua:315](../../../raw/current/scripts/prefabs/wanderingtrader.lua#L315 ([wanderingtrader.lua:348-360](../../../raw/current/scripts/prefabs/wanderingtrader.lua#L348-L360)
- 生效与撤销：SetRoute 覆盖同名；随世界存档持久化，无独立清理入口。

```lua
AddPrefabPostInit("world", function(inst)
    inst:DoTaskInTime(0, function()
        if not TheWorld.ismastersim then
            return
        end
        if TheWorld.components.worldroutes == nil then
            TheWorld:AddComponent("worldroutes")
        end
        TheWorld.components.worldroutes:SetRoute("mypatrol", {
            Vector3(0, 0, 0),
            Vector3(60, 0, 60),
        })
    end)
end)
```

- 调整与验证：路线合法性（跨海、绕障）由生成方负责；原版拓扑寻路无结果时回退随机游走。([wanderingtrader.lua:351-357](../../../raw/current/scripts/prefabs/wanderingtrader.lua#L351-L357)

## 常见陷阱与限制

- 组件无 `OnUpdate`、无事件；"路线跟随"行为在 worldroutefollower，不要期望本组件驱动移动。([worldroutes.lua:3-18](../../../raw/current/scripts/components/worldroutes.lua#L3-L18)
- 点列被消费方按引用读取；注册后再改动表会影响所有读者。
- 存读只保留 x/z：手工构造的 y 高度不会被持久化。[worldroutes.lua:26](../../../raw/current/scripts/components/worldroutes.lua#L26 [worldroutes.lua:48](../../../raw/current/scripts/components/worldroutes.lua#L48

## 验证记录

- 源码审阅：2026-10-28，`steam-buildid:26230496`：审阅组件全文；核验世界安装点、wanderingtrader 两条生成路径与守卫、worldroutefollower 读取。
- 游戏运行：未运行；未测跨分片与超长路线存读。
- 结构检查：validate_page.py 通过；validate_source_links.py --strict 通过（本批 16 页合并执行）。
