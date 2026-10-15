---
title: walkableplatform
created: 2026-10-21
updated: "2026-10-28"
type: component
summary: "把实体注册为可行走平台：按半径维护成员与玩家集合、同步满载和摄像机标记、用 UID 保存，并在移除时推送沉没与清理。"
aliases: ["可行走平台", "平台组件", "船上平台", "平台成员", "IsFull", "平台摄像机", "walkableplatform", "walkableplatformmanager"]
tags: [component, server-client, network-sync]
dst_build: "steam-buildid:26230496"
sources: ["components/walkableplatform.lua", "components/walkableplatformmanager.lua", "components/walkableplatformplayer.lua", "entityscript.lua", "prefabs/boat.lua", "prefabs/yotd_boats.lua", "prefabs/abysspillar.lua", "components/locomotor.lua", "components/map.lua", "components/boatphysics.lua", "components/boatdrifter.lua", "mainfunctions.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# walkableplatform

## 快速判断

- 原版宿主与注册名：`prefabs/boat.lua` 在共用前置 `create_common_pre` 添加，覆盖注册船 `boat`、`boat_grass`、`boat_pirate`、`boat_ancient`、`boat_otterden`、`boat_ice`、`boat_ice_crabking`（[prefabs/boat.lua:1639-1661](../../../raw/current/scripts/prefabs/boat.lua#L1639-L1661)）；`prefabs/yotd_boats.lua` 在 `dragonboat_common` 添加，覆盖 `dragonboat_body` 与 `dragonboat_shadowboat`（[prefabs/yotd_boats.lua:915-922](../../../raw/current/scripts/prefabs/yotd_boats.lua#L915-L922)）；`prefabs/abysspillar.lua` 对应 `abysspillar`（[prefabs/abysspillar.lua:843](../../../raw/current/scripts/prefabs/abysspillar.lua#L843）。
- 职责与适用：把实体登记为“可站立、可被跳上”的平台，负责半径内成员跟随、玩家进出记录、满载标记、平台 UID 与移除清理；不包含浮力、推进或碰撞体本身。[components/walkableplatform.lua:1-18](../../../raw/current/scripts/components/walkableplatform.lua#L1-L18)
- 优先使用：自定义船／浮台加 `walkableplatform`，设 `platform_radius`（默认 4）；需要边缘不可走出时再给 `player_collision_prefab`。[components/walkableplatform.lua:143-151](../../../raw/current/scripts/components/walkableplatform.lua#L143-L151)
- 重要副作用：构造时加 `walkableplatform` 标签（双端参与地图与寻路查询）；服务端每帧重建成员表；`SetIsFull` 增删 `walkableplatform_full`；移除实体时给成员与本平台发 `onsink` 并销毁甲板物品。[components/walkableplatform.lua:24-51](../../../raw/current/scripts/components/walkableplatform.lua#L24-L51)
- 修改边界：成员、玩家集合、`isfull`、UID、清理都只在主模拟端；客户端只读标签与 `doplatformcamerazoom`，摄像机开关逻辑不在本组件。[components/walkableplatform.lua:20-22](../../../raw/current/scripts/components/walkableplatform.lua#L20-L22)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `platform_radius` | `number`，默认 `4` | 成员判定与地图查询都用它；宿主设为物理半径（木船 4、草船 3、水獭巢船 3.75、龙舟 3、深渊柱 0.6）。[walkableplatform.lua:6](../../../raw/current/scripts/components/walkableplatform.lua#L6)、[prefabs/boat.lua:412-413](../../../raw/current/scripts/prefabs/boat.lua#L412-L413) |
| `player_collision_prefab` | `string?` | 非空时构造后延迟 0 秒生成碰撞体并作平台跟随者；深渊柱自建碰撞，所以该字段留空、组件生成是空操作。[walkableplatform.lua:208-222](../../../raw/current/scripts/components/walkableplatform.lua#L208-L222)、[prefabs/abysspillar.lua:434-449](../../../raw/current/scripts/prefabs/abysspillar.lua#L434-L449) |
| `SetIsFull(isfull)` / `IsFull()` | 布尔；无返回值／返回 `boolean` | 仅服务端写；写时增删标签供客户端与 `locomotor` 判断能否跳上，不限制已在平台的单位。[walkableplatform.lua:118-133](../../../raw/current/scripts/components/walkableplatform.lua#L118-L133)、[locomotor.lua:1238-1241](../../../raw/current/scripts/components/locomotor.lua#L1238-L1241) |
| `player_only` / `no_mounts` / `max_hop_distance` | 组件不初始化（源码为注释行） | 由宿主赋值，消费方是 `locomotor` 与 `map`；默认 nil 表示不限制，不要以为组件内建默认值。[walkableplatform.lua:7-9](../../../raw/current/scripts/components/walkableplatform.lua#L7-L9)、[locomotor.lua:1229-1231](../../../raw/current/scripts/components/locomotor.lua#L1229-L1231) |
| `AddPlayerOnPlatform` / `RemovePlayerOnPlatform` / `GetPlayersOnPlatform` | 玩家实体 → `void`；getter 返回表 | 仅服务端由 `walkableplatformplayer` 调用；玩家集合与成员表是两套数据。[walkableplatform.lua:100-114](../../../raw/current/scripts/components/walkableplatform.lua#L100-L114) |
| `GetEntitiesOnPlatform` / `AddEntityToPlatform` / `RemoveObject` | 表／实体 → void | `AddEntityToPlatform` 只把无父实体挂为跟随者，成员登记要等下一次 `SetEntitiesOnPlatform`；`RemoveObject` 是原版未调用、留给模组的移除入口。[walkableplatform.lua:143-151](../../../raw/current/scripts/components/walkableplatform.lua#L143-L151)、[:201-206](../../../raw/current/scripts/components/walkableplatform.lua#L201-L206) |
| `SetEntitiesOnPlatform()` | 无参数（manager 会传 `dt`，被忽略） | 服务端每帧重建成员表：半径内加跟随者、旧的移除；`INLIMBO`／飞行／FX 等标签除外。[walkableplatform.lua:153-199](../../../raw/current/scripts/components/walkableplatform.lua#L153-L199) |
| `StartUpdating` / `StopUpdating` | 无返回值 | 向 `walkableplatformmanager` 注册／注销本平台；宿主在物理或实体睡眠时停、唤醒或构造时开。[walkableplatform.lua:69-75](../../../raw/current/scripts/components/walkableplatform.lua#L69-L75) |
| `GetUID()` / `OnSave` / `OnLoad` | 返回 `number`；存档 `{ uid }` | 首次取 UID 才注册；`OnLoad` 先恢复 uid 再无条件注册。UID 供平台上实体存档写 `puid` 与相对坐标。[walkableplatform.lua:53-67](../../../raw/current/scripts/components/walkableplatform.lua#L53-L67)、[entityscript.lua:272-287](../../../raw/current/scripts/entityscript.lua#L272-L287) |
| `HasPlatformCamera()` | 返回 `inst.doplatformcamerazoom`（`net_bool` 或 nil） | 返回网络布尔对象，真值只表示“有缩放通道”；当前是否缩放要看 `:value()`。只有船类宿主定义该字段。[walkableplatform.lua:20-22](../../../raw/current/scripts/components/walkableplatform.lua#L20-L22)、[prefabs/boat.lua:433](../../../raw/current/scripts/prefabs/boat.lua#L433 |
| `GetEmbarkPosition(x, z, min_dist?)` | 返回 `x, z` | 从平台中心指向登船者，落在半径减 0.5 再减 `min_dist` 的圆周上；供 `embarker`／鼠标登船取点。[walkableplatform.lua:135-141](../../../raw/current/scripts/components/walkableplatform.lua#L135-L141) |
| `CommitPlayersOnPlatform()` | 无参数 | 玩家集合从空到有／从有到空时切 `boatdrifter` 的漂流状态与物理唤醒测试。[walkableplatform.lua:224-236](../../../raw/current/scripts/components/walkableplatform.lua#L224-L236) |

## 关键流程

1. 服务端每帧 `walkableplatformmanager:PostUpdate` 先对每个已注册平台调 `SetEntitiesOnPlatform`：半径内（`TheSim:FindEntities`）且无 `ignorewalkableplatforms`／飞行等标签、无父实体的对象挂为平台跟随者；不在新表里的旧成员移除。[walkableplatformmanager.lua:56-70](../../../raw/current/scripts/components/walkableplatformmanager.lua#L56-L70)
2. 同一次更新里对每个玩家跑 `walkableplatformplayer:TestForPlatform`，以 `GetCurrentPlatform()` 与记录的平台比较，触发 `GetOnPlatform`／`GetOffPlatform`。[walkableplatformplayer.lua:244-271](../../../raw/current/scripts/components/walkableplatformplayer.lua#L244-L271)
3. `GetOnPlatform`：服务端加入 `players_on_platform`；本机 `ThePlayer` 另发 `got_on_platform`、启动船速音乐测试，并在允许时启动船摄像机与缩放监听。[walkableplatformplayer.lua:198-219](../../../raw/current/scripts/components/walkableplatformplayer.lua#L198-L219)
4. `GetOffPlatform`：停摄像机／音乐、服务端移除玩家、清 `Transform:SetIsOnPlatform`，发 `got_off_platform`。[walkableplatformplayer.lua:221-242](../../../raw/current/scripts/components/walkableplatformplayer.lua#L221-L242)
5. 最后 `CommitPlayersOnPlatform` 汇总漂移状态；移除实体时 `OnRemoveEntity` 停更新、删碰撞体、逐级发 `onsink` 并清甲板。[components/walkableplatform.lua:24-51](../../../raw/current/scripts/components/walkableplatform.lua#L24-L51)

## 典型接入

### 自定义可行走平台（服务端权威，参照深渊柱最小集）

- 前提与选择：目标文件 `scripts/prefabs/my_platform.lua`，由 `modmain.lua` 的 `PrefabFiles` 加载；资源可复用 `anim/abyss_pillar.zip`。组件必须在 `entity:SetPristine()` **之前**添加，两端才会都带 `walkableplatform` 标签，与原版 [prefabs/abysspillar.lua:615-626](../../../raw/current/scripts/prefabs/abysspillar.lua#L615-L626) 一致。
- 生效与撤销：构造即在主模拟端 `StartUpdating` 并注册到 manager，玩家可跳上，平台成员由 `SetPlatform` 绑定跟随；`inst:Remove()` 触发沉没与清理。未设 `player_collision_prefab` 时组件不生成碰撞体（见接口表），边缘没有隐形墙，限制需宿主自理。
- 原版依据：船类另加物理圆柱与 `boat` 标签（[prefabs/boat.lua:381-413](../../../raw/current/scripts/prefabs/boat.lua#L381-L413)），并在各变体 `SetPristine` 前设碰撞 prefab，例如木船 [prefabs/boat.lua:770](../../../raw/current/scripts/prefabs/boat.lua#L770)；深渊柱不动物理、自建玩家碰撞（[prefabs/abysspillar.lua:434-449](../../../raw/current/scripts/prefabs/abysspillar.lua#L434-L449)）。

```lua
local assets =
{
    Asset("ANIM", "anim/abyss_pillar.zip"),
}

local function fn()
    local inst = CreateEntity()

    inst.entity:AddTransform()
    inst.entity:AddAnimState()
    inst.entity:AddSoundEmitter()
    inst.entity:AddNetwork()

    inst.AnimState:SetBank("abyss_pillar")
    inst.AnimState:SetBuild("abyss_pillar")
    inst.AnimState:PlayAnimation("idle_a")

    -- 避免本平台再被别的平台收编；walkableplatform 标签由组件添加
    inst:AddTag("ignorewalkableplatforms")

    local platform = inst:AddComponent("walkableplatform")
    platform.platform_radius = 1.5
    -- 需要边缘碰撞时再设 platform.player_collision_prefab，
    -- 且碰撞体 prefab 的网格半径必须与 platform_radius 匹配

    inst.entity:SetPristine()
    if not TheWorld.ismastersim then
        return inst
    end

    inst:AddComponent("inspectable")

    return inst
end

return Prefab("my_platform", fn, assets)
```

- 调整与验证：`platform_radius` 同时决定成员范围、地图跳板取点与登船位置，改动后要复查 `TUNING.MAX_WALKABLE_PLATFORM_RADIUS`（4）的查询余量；需要移动或浮力时另加 `boatphysics`（见 [[reference/components/boatphysics]]，本页不展开）。案例只做语法检查、未运行。

## 常见陷阱与限制

- 不要用 `RemoveComponent("walkableplatform")` 卸载：它只调用 `OnRemoveFromEntity`，本组件没有该方法，于是 `StopUpdating`／注销都不会发生，manager 下一帧对空组件调用 `SetEntitiesOnPlatform` 会报错，且 `walkableplatform` 标签仍留着。要撤销请 `Remove()` 实体，或自己先 `StopUpdating` 再移除。[entityscript.lua:670-685](../../../raw/current/scripts/entityscript.lua#L670-L685)
- `walkableplatformmanager` 清理失效平台的分支写的是 `self.walkableplatform[k] = nil`，而注册表字段叫 `walkable_platforms`；一旦有平台在仍注册时失效，这里不是静默清理而是索引错误。组件自身的 `OnRemoveEntity` 会先 `StopUpdating`，所以正常删除不会触发。[walkableplatformmanager.lua:56-64](../../../raw/current/scripts/components/walkableplatformmanager.lua#L56-L64)
- `HasPlatformCamera()` 返回 `net_bool` 对象，木板船即使当前没缩放也返回真值；把返回值当“是否正在缩放”会永远走缩放分支，正确读法是再取 `:value()`。[components/walkableplatform.lua:20-22](../../../raw/current/scripts/components/walkableplatform.lua#L20-L22)、[walkableplatformplayer.lua:182-188](../../../raw/current/scripts/components/walkableplatformplayer.lua#L182-L188)
- `SetIsFull` 是“拒绝新的 locomotor 跳上”的标记，不踢走已在平台上的单位，也不阻止玩家直接从甲板走下去；原版只有深渊柱用它做占位／预约，船类从不调用。[components/locomotor.lua:1226-1241](../../../raw/current/scripts/components/locomotor.lua#L1226-L1241)、[prefabs/abysspillar.lua:386-410](../../../raw/current/scripts/prefabs/abysspillar.lua#L386-L410)
- 成员按半径球判定，不是按甲板网格：半径过大会把邻近漂浮物、甚至岸上实体挂上平台；`AddEntityToPlatform` 不立即写成员表，若实体随后立即移出半径，可能不会被 `RemovePlatformFollower` 清掉。[components/walkableplatform.lua:143-151](../../../raw/current/scripts/components/walkableplatform.lua#L143-L151)、[:194-198](../../../raw/current/scripts/components/walkableplatform.lua#L194-L198)
- 平台落在可视地面上时，`SetEntitiesOnPlatform` 会置 `boatphysics:SetHalting(true)`，有 `health` 且未死则直接 `Kill()`；不带 `boatphysics` 的自定义平台不会触发这段，但也没有搁浅保护。[components/walkableplatform.lua:185-192](../../../raw/current/scripts/components/walkableplatform.lua#L185-L192)
- `player_only`／`no_mounts`／`max_hop_distance` 在组件里只是注释，读取方在 `locomotor`／`map`；写在其它实体上不生效，必须写在平台实体的同名组件字段上，跳板细节见 [[reference/components/locomotor]]。[components/locomotor.lua:1226-1231](../../../raw/current/scripts/components/locomotor.lua#L1226-L1231)
- 移除时给成员发 `onsink`：可溺水实体（玩家）拿到一次性共享的岸边点，其他实体只带 `boat`；船上物品在 `DestroyObjectsOnPlatform` 里只被 `SetLanded(false, true)` 丢下，带父实体、`amphibiouscreature`、`drownable` 的对象会被跳过。[components/walkableplatform.lua:29-45](../../../raw/current/scripts/components/walkableplatform.lua#L29-L45)、[:80-98](../../../raw/current/scripts/components/walkableplatform.lua#L80-L98)
- 存档顺序耦合：平台上实体的存档记录写 `puid` 与相对坐标，读档时靠 manager 的 UID 表找回平台；平台未能先恢复或 UID 丢失时，实体退回世界坐标。[mainfunctions.lua:450-462](../../../raw/current/scripts/mainfunctions.lua#L450-L462)
- 摄像机开关是客户端选项：`walkableplatformplayer` 只监听 `enableboatcamera`／`enablemovementprediction`，缩放由 `boatphysics` 写 `doplatformcamerazoom`、选项界面发 `EnableBoatCamera` 决定，不是组件自动完成。[prefabs/player_common.lua:946-948](../../../raw/current/scripts/prefabs/player_common.lua#L946-L948)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 锚点重定位 1 处（mainfunctions puid 存档链 424-436→450-462）；无结论修订（R-09 不触及本页 UID 存档）。
- 依据：全局枢纽 hunk 地图（R-09）＋变更清单命中 mainfunctions／player_common／entityscript。

- 源码审阅：2026-10-21，`steam-buildid:25643519`；覆盖构造与双端标签、三处宿主（`boat` 系列、`dragonboat_body`／`dragonboat_shadowboat`、`abysspillar`）的注册名与半径、逐帧成员重建、玩家集合与摄像机流程、`IsFull` 消费方、`OnSave`／`OnLoad` 与 `puid` 存档、`OnRemoveEntity` 清理顺序及移除组件的后果。未覆盖：实机站立／跳板手感、多人并发与断线、平台随波移动的物理细节（属 `boatphysics`）、模组覆盖 manager 的冲突。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict 与示例语法检查通过。

## 相关链接

- [[reference/components/walkableplatformmanager]]：平台 UID 注册表与成员更新驱动。
