---
title: "dockmanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级码头瓦片管理器：在地图网格上登记 MONKEY_DOCK 瓦片的根连接、耐久与破损实体，处理建造、损伤修复、断裂队列与沉没清算。"
aliases: ["dockmanager", "码头管理器", "monkey dock", "码头瓦片", "码头破坏"]
tags: [component, building, data]
dst_build: "steam-buildid:26230496"
sources: ["components/dockmanager.lua", "prefabs/world.lua", "tuning.lua", "prefabs/dock_kit.lua", "prefabs/dock_damage.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# dockmanager

## 快速判断

- 职责与适用：挂在世界的 master 组件（断言 ismastersim），用四张 DataGrid 跟踪猴岛码头瓦片（WORLD_TILES.MONKEY_DOCK）的根连接性、删除标记、耐久与破损装饰实体；所有世界组件都能通过 `TheWorld.components.dockmanager` 调它。[安装](../../../raw/current/scripts/prefabs/world.lua#L585 [断言](../../../raw/current/scripts/components/dockmanager.lua#L10
- 优先使用：建码头用 `CreateDockAtPoint/AtTile(x,y,z, dock_tile_type)`；修/伤用 `DamageDockAtPoint(x,y,z, damage)`（负值=修复）；建完校验连通用 `ResolveDockSafetyAtPoint`。[创建](../../../raw/current/scripts/components/dockmanager.lua#L205-L228) [损伤](../../../raw/current/scripts/components/dockmanager.lua#L361-L388)
- 重要副作用：破坏码头会把瓦片还原为 undertile 底瓦（默认 OCEAN_COASTAL），并对瓦片上实体推 `onsink`/`abandon_ship`/`onpresink`、落下物品 SetLanded 或销毁；断裂破坏延迟 70～80 帧执行并留裂纹过渡。[销毁](../../../raw/current/scripts/components/dockmanager.lua#L231-L320) [队列](../../../raw/current/scripts/components/dockmanager.lua#L322-L353)
- 修改边界：只认 MONKEY_DOCK 瓦片；连通性判定以"非码头陆地/临时瓦片"为根（FARMING_SOIL 除外）；网格随世界存档（ZipAndEncodeSaveData），删除中瓦片重载自动续爆。[根判定](../../../raw/current/scripts/components/dockmanager.lua#L43-L46) [存读](../../../raw/current/scripts/components/dockmanager.lua#L421-L466)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `CreateDockAtPoint(x,y,z, dock_tile_type)`／`AtTile(tx,ty,…)` | → bool | 瓦片直接 SetTile 成码头类型；有 undertile 组件时先读底瓦并在 SetTile 后回写（绕过农耕回调清空底瓦的行为），然后登记根数据与满耐久（`MONKEYISLANDDOCK_HEALTH=300`）。[创建](../../../raw/current/scripts/components/dockmanager.lua#L205-L228) [耐久常量](../../../raw/current/scripts/tuning.lua#L6676 |
| `DamageDockAtPoint(x,y,z, damage)`／`AtTile(tx,ty,…)` | → 新耐久或 nil | 无数据或已到 0 返回 nil；耐久 clamp 到 [0,300]，按比例挂/更新 `dock_damage` 实体（`setdamagepecent`），归零自动进 QueueDestroy。负 damage 即修复（会移除破损实体）。[损伤](../../../raw/current/scripts/components/dockmanager.lua#L366-L388) [修复消费](../../../raw/current/scripts/prefabs/dock_damage.lua#L27-L30) |
| `QueueDestroyForDockAtPoint(x,y,z, dont_toss_loot?)` | — | 标记删除、播 `fx_dock_crackle`、70+随机10 帧后真删；对海面底瓦上的实体（视觉在地表者）推 `abandon_ship`，玩家另推 `onpresink`。[队列](../../../raw/current/scripts/components/dockmanager.lua#L322-L353) |
| `DestroyDockAtPoint(x,y,z, dont_toss_loot?)` | → bool | 仅对 MONKEY_DOCK 生效：清破损实体、还原底瓦、清网格数据；若还原成海瓦，则按 walkableplatform 拆除协议对附近实体推 `onsink`、非溺亡实体落物品或 DestroyEntity，再对邻接码头跑连通复检，最后抛木板/原木碎片。[销毁](../../../raw/current/scripts/components/dockmanager.lua#L231-L320) |
| `ResolveDockSafetyAtPoint(x,y,z)` | → bool | 对单瓦跑连通复检（是否连根），dock_kit 建造时用它判定"不安全则不能建成"。[复检](../../../raw/current/scripts/components/dockmanager.lua#L355-L359) [kit 消费](../../../raw/current/scripts/prefabs/dock_kit.lua#L39-L42) |
| 连通判定（内部 `_TestForBreaking`/`bfs_for_root`） | — | BFS 找"邻接陆地"的根瓦：断根的瓦片进删除标记并随机 1～10 帧后排爆；visited 结果按是否连通回写缓存。邻接 8 向。[BFS](../../../raw/current/scripts/components/dockmanager.lua#L88-L139) [复检入口](../../../raw/current/scripts/components/dockmanager.lua#L176-L189) |
| `SpawnDamagePrefab(tile_index, health)`（内部） | — | 耐久低于满值时挂 dock_damage 装饰并按百分比设置裂纹；满值时移除。OnLoad 后按存档耐久重建装饰。[装饰](../../../raw/current/scripts/components/dockmanager.lua#L396-L415) |
| `GetCoordsFromIndex(index)` | → x,z | 网格索引转瓦片坐标（依赖 WIDTH）。[换算](../../../raw/current/scripts/components/dockmanager.lua#L390-L394) |
| `OnSave()`／`OnLoad(data)` | — | 存根/删除标记/耐久三张网格（ZipAndEncodeSaveData）；重载时对已标记瓦片重新排随机延迟销毁，耐久网格重建破损装饰。[存读](../../../raw/current/scripts/components/dockmanager.lua#L421-L466) |

### 与伤害源的既有接线

爆炸/炮弹/迫击炮落地时按总伤调 `DamageDockAtPoint`；玩家用锤修（dock_damage 交互）传负伤；dock_kit 建造前 `ResolveDockSafetyAtPoint` 防孤块。这些都是既有消费者而非本组件接口。[爆炸](../../../raw/current/scripts/components/explosive.lua#L53-L56) [炮弹](../../../raw/current/scripts/prefabs/cannonballs.lua#L134-L137) [迫击炮](../../../raw/current/scripts/prefabs/mortarball.lua#L155-L158)

## 典型接入

### 模组动态搭建或拆除码头

- 前提与选择：目标世界需有 dockmanager（默认世界都有）；直接给组件传坐标即可，无需自己 SetTile。想用非猴岛码头瓦片也可传自定义 dock_tile_type，但销毁/根判定只按 MONKEY_DOCK 逻辑清理（其余类型不会被 DestroyDockAtPoint 删除）。[瓦片检查](../../../raw/current/scripts/components/dockmanager.lua#L233-L236)
- 生效与撤销：建造后网格立即参与断裂复检；撤销用 `DestroyDockAtPoint`（走完整沉没清算与碎片）或负伤修满。批量拆除时组件内部自动连锁断裂，无需逐瓦调用。
- 原版依据：建造 kits（dock_kit）通过 action 调 CreateDockAtPoint 后再 ResolveDockSafetyAtPoint；孤岛码头会被立刻回拆。[kit](../../../raw/current/scripts/prefabs/dock_kit.lua#L39-L42)

```lua
-- 服务器端：在指定点建造一块码头并确认未形成孤块
local function BuildSafeDock(x, z)
    local dm = TheWorld.components.dockmanager
    if dm == nil then
        return false
    end
    dm:CreateDockAtPoint(x, 0, z, WORLD_TILES.MONKEY_DOCK)
    return dm:ResolveDockSafetyAtPoint(x, 0, z)
end
```

- 调整与验证：坐标按世界坐标，组件内部换算瓦片；孤立建造会立即被复检排队销毁，测试时应紧邻陆地或已有码头。[孤块复检](../../../raw/current/scripts/components/dockmanager.lua#L151-L170)

## 常见陷阱与限制

- **CreateDock 不做安全检查：** 只写瓦片与数据；安全判定是调用方（dock_kit）的职责，模组直接 Create 后必须自查 ResolveDockSafetyAtPoint，否则孤块会在几帧内爆掉。[职责划分](../../../raw/current/scripts/components/dockmanager.lua#L205-L228)
- **DamageDock 传 0 无效果：** damage=0 时新耐久等于旧值但仍会刷新破损装饰；修复传负值且上限 clamp 到 300。想判断瓦片是否存在应看返回值是否为 nil。[返回语义](../../../raw/current/scripts/components/dockmanager.lua#L370-L374)
- **非海面底瓦的码头拆除没有沉没清算：** 还原成陆地瓦时跳过 onsink/DestroyEntity 分支，仅清数据；实体悬空行为交还游戏物理。[分支条件](../../../raw/current/scripts/components/dockmanager.lua#L263-L264)
- **`GetCoordsFromIndex` 的取整方式：** 用 `math.modf(index/WIDTH)` 取商，index 是整数网格索引；传瓦片坐标会得到错误结果。[实现](../../../raw/current/scripts/components/dockmanager.lua#L390-L394)
- **删除中存档会续爆：** marked_for_delete 网格重载后按随机 1～10 帧重排销毁任务——排队期间保存世界的"取消拆除"只能靠手动清标记（无公开接口）。[重载续爆](../../../raw/current/scripts/components/dockmanager.lua#L444-L452)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 dockmanager.lua，核验四网格、创建/损伤/队列/销毁全链路、BFS 连通判定、沉没清算、碎片与存读续爆；核对 world 安装、dock_kit/dock_damage/explosive/cannonballs/mortarball 消费与 MONKEYISLANDDOCK_HEALTH 常量。undertile 组件内部回调未展开。
- 结构检查：本页通过 validate_page.py 与 validate_source_links.py（严格）；Lua 示例经 parser_probe 语法检查。
- 游戏运行：未运行；连锁断裂与沉没清算表现未实测。

相关：[[reference/components/undertile]]。

## 相关链接

- [[reference/components/undertile]]：码头底瓦的保存与还原依赖。
