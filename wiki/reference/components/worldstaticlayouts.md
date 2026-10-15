---
title: "worldstaticlayouts"
created: "2026-10-26"
updated: "2026-10-28"
type: "component"
summary: "world 实体上的必放静态布局登记表：RegisterLayout 登记名字与放置函数，读档后 PlaceLayouts 统一执行并断言失败，placedlayouts 随档持久化防重放。"
aliases: ["worldstaticlayouts", "世界静态布局登记", "必放布局", "RegisterLayout"]
tags: ["component", "worldgen"]
dst_build: "steam-buildid:26230496"
sources: ["components/worldstaticlayouts.lua", "prefabs/world.lua", "prefabs/cave.lua", "maputil.lua", "constants.lua", "savefileupgrades.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
size_class: "simple"
---

# worldstaticlayouts

## 快速判断

- 职责与适用：挂在 world 实体上的"生成/读档后必须补放的静态布局"执行器：登记的放置函数在读档后统一跑一次，成败有断言、已放名单随档走。可选装饰性摆放不适用本组件（失败即崩）。[worldstaticlayouts.lua:18-33](../../../raw/current/scripts/components/worldstaticlayouts.lua#L18-L33)
- 优先使用：`AddComponent` 由 MakeWorld 基座统一完成（全部世界都有），接入方只需在世界 prefab 主端（master_postinit 或 AddPrefabPostInit）`RegisterLayout(name, fn)`，`fn(theworld)` 成功返回 truthy。[prefabs/world.lua:599](../../../raw/current/scripts/prefabs/world.lua#L599)、[cave.lua:559](../../../raw/current/scripts/prefabs/cave.lua#L559
- 重要副作用：`PlaceLayouts` 结束即把登记表置 nil——之后任何再登记直接崩（源码注释明示有意为之）；放置函数返回 falsy 触发 assert 中止世界加载。[worldstaticlayouts.lua:22-32](../../../raw/current/scripts/components/worldstaticlayouts.lua#L22-L32)
- 修改边界：仅主端（组件挂在 master 分支）；登记必须发生在 `OnLoadPostPass` 调 `PlaceLayouts` 之前——prefab 构造期登记安全。[prefabs/world.lua:399-401](../../../raw/current/scripts/prefabs/world.lua#L399-L401)、[:572](../../../raw/current/scripts/prefabs/world.lua#L572)

## 关键接口

| 接口 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `RegisterLayout(layoutname, fn)` | fn(theworld)→truthy | 首次登记生效，同名重复登记被忽略；按登记顺序入队。[worldstaticlayouts.lua:11-16](../../../raw/current/scripts/components/worldstaticlayouts.lua#L11-L16) |
| `PlaceLayouts()` | 无参 | 按序执行未放条目；fn 成功记入 placedlayouts，失败 assert("Your world failed to place a required layout %s...")；末尾置 nil 登记表。[worldstaticlayouts.lua:18-33](../../../raw/current/scripts/components/worldstaticlayouts.lua#L18-L33) |
| `OnSave()` | 放过才存 | placedlayouts 非空才返回 `{ placedlayouts = ... }`，空表不写数据。[worldstaticlayouts.lua:35-41](../../../raw/current/scripts/components/worldstaticlayouts.lua#L35-L41) |
| `OnLoad(data)` | 存档恢复 | 恢复 placedlayouts，使已放布局在读档后跳过重放。[worldstaticlayouts.lua:43-49](../../../raw/current/scripts/components/worldstaticlayouts.lua#L43-L49) |

## 典型接入

### 原版例：cave 注册 VAULT 布局

cave 主端 `RegisterLayout(VIRTUALROOMSETS.VAULT, CreateLayout_Vault)`；键名取常量 `VIRTUALROOMSETS.VAULT = "VAULT"`。[cave.lua:559](../../../raw/current/scripts/prefabs/cave.lua#L559)、[constants.lua:3049-3053](../../../raw/current/scripts/constants.lua#L3049-L3053)

`CreateLayout_Vault` 的放置链：取 virtualroommanager 大厅房间集的 MARKER 实体定位 archive_portal（缺失即失败返回）→ `obj_layout.LayoutForDefinition` 取 "Vault_Lobby"／"Vault_Vault" 静态布局 → 以传送门瓦坐标为锚，`StaticLayoutPlacer.TryToPlaceStaticLayoutNear` 螺旋扫描＋不可通行瓦过滤放置两份布局，任一失败返回 false 由 PlaceLayouts 断言。[cave.lua:487-516](../../../raw/current/scripts/prefabs/cave.lua#L487-L516)

位置过滤 `StaticLayoutPlacer.CreateVRSPositionFilter(roomsetname)`（全局表导出自 [maputil.lua:534-551](../../../raw/current/scripts/maputil.lua#L534-L551)）：按房间集名查 `VIRTUALROOM_BOUNDINGBOXES`，返回 `(tx, ty)` 过滤函数，内部调 `virtualroommanager:WillBoundingBoxCollideAt(boundingbox, tx, ty)` 判碰撞；无 manager 时恒 false（视为不碰撞）。[maputil.lua:432-448](../../../raw/current/scripts/maputil.lua#L432-L448)、[constants.lua:3081](../../../raw/current/scripts/constants.lua#L3081

`WillBoundingBoxCollideAt` 语义见 [[reference/components/virtualroommanager]]；金库整套装配与迁移背景见 [[reference/mechanisms/vault-virtualroom-migration]]。

存档迁移：升级器把已废弃 vaultroommanager 的 `spawnedlayouts` 标志改写为本组件 `placedlayouts = { VAULT = true }`（cave 存档），另有 dev 分支补丁按 vaultmarker 实体补写。[savefileupgrades.lua:1438-1444](../../../raw/current/scripts/savefileupgrades.lua#L1438-L1444)、[:1618-1625](../../../raw/current/scripts/savefileupgrades.lua#L1618-L1625)

### 注册自定义必放布局

- 前提与选择：模组 `modmain.lua`；AddPrefabPostInit 在世界构造期执行，早于读档后的 PlaceLayouts。
- 生效与撤销：放置函数返回 true 记入 placedlayouts 并随档持久化；返回 false／nil 中止加载。

```lua
AddPrefabPostInit("cave", function(inst)
    if GLOBAL.TheWorld.ismastersim then
        inst.components.worldstaticlayouts:RegisterLayout("MY_SETPIECE", function(world)
            local setpiece = GLOBAL.SpawnPrefab("my_setpiece")
            setpiece.Transform:SetPosition(0, 0, 0) -- 换成锚点选择逻辑
            return true -- 必须返回 truthy，否则读档断言
        end)
    end
end)
```

- 调整与验证：锚点选择可参照 cave 例（标记实体定位＋瓦坐标）；需要避开虚拟房间碰撞时用 `StaticLayoutPlacer.CreateVRSPositionFilter`。[cave.lua:502-513](../../../raw/current/scripts/prefabs/cave.lua#L502-L513)

## 常见陷阱与限制

- **只用于必放布局**：fn 失败不是重试而是 assert 崩加载；可选/装饰摆放自行在 worldgen 或后初始化里容错。[worldstaticlayouts.lua:22-27](../../../raw/current/scripts/components/worldstaticlayouts.lua#L22-L27)
- **PlaceLayouts 之后登记必崩**：desiredlayouts/layoutorder 被置 nil，晚期 RegisterLayout 索引 nil 报错——源码注释明示用崩溃暴露过晚登记。[worldstaticlayouts.lua:30-32](../../../raw/current/scripts/components/worldstaticlayouts.lua#L30-L32)
- **同名重复登记静默忽略**：以第一次为准，改绑函数需先确认原键名。[worldstaticlayouts.lua:12-15](../../../raw/current/scripts/components/worldstaticlayouts.lua#L12-L15)
- **placedlayouts 键名即协议**：迁移器写死 `"VAULT"` 字符串键，自定义布局读旧档判定同样按登记名匹配。[savefileupgrades.lua:1439-1443](../../../raw/current/scripts/savefileupgrades.lua#L1439-L1443)

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖 worldstaticlayouts.lua 全部 51 行、world.lua 组件挂载与 OnLoadPostPass 钩子、cave.lua 的 VAULT 登记＋CreateLayout_Vault 全函数、maputil CreateVRSPositionFilter 及 StaticLayoutPlacer 导出、constants 键值、savefileupgrades 两处迁移段。未覆盖：世界生成／读档的运行验证。
- 游戏运行：未运行。
- 结构检查：2026-10-26 执行 validate_page.py（simple 档）与 validate_source_links.py（--strict）通过；页内 Lua 示例经 parser_probe.exe --syntax-only 语法检查通过。

相关：[[reference/mechanisms/vault-virtualroom-migration]]；[[reference/components/virtualroommanager]]。
