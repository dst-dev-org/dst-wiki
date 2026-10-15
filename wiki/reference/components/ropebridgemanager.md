---
title: "ropebridgemanager"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "洞穴世界绳桥瓦片管理器：把地皮换成 ROPE_BRIDGE 并维护血量/破坏/动画四张网格，处理地震按整桥伤害与 quake_blocker 保护。"
aliases: ["ropebridgemanager", "绳桥管理器", "绳桥瓦片", "绳桥地震破坏"]
tags: [component, mechanism, worldgen]
dst_build: "steam-buildid:26230496"
sources: ["components/ropebridgemanager.lua", "prefabs/rope_bridge.lua", "prefabs/dock_damage.lua", "prefabs/cave.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# ropebridgemanager

## 快速判断

- 职责与适用：仅洞穴世界安装（[安装](../../../raw/current/scripts/prefabs/cave.lua#L556)）的主模拟组件，管理 `WORLD_TILES.ROPE_BRIDGE` 瓦片：创建即改地皮并登记血量网格，破坏恢复地皮，地震时对未受保护的整条桥扣血。桥的视觉是独立 `rope_bridge_fx`，建造入口是 `rope_bridge` 套件道具。[断言](../../../raw/current/scripts/components/ropebridgemanager.lua#L5) [套件调用](../../../raw/current/scripts/prefabs/rope_bridge.lua#L340-L352)
- 优先使用：排队建格 `QueueCreateRopeBridgeAtPoint(x,y,z,data)`；扣血/修复统一走 `DamageRopeBridgeAtPoint(x,y,z,damage)`（damage 负值为修）；显式拆格 `QueueDestroyForRopeBridgeAtPoint`/`DestroyRopeBridgeAtPoint`。[创建](../../../raw/current/scripts/components/ropebridgemanager.lua#L169-L184) [伤害](../../../raw/current/scripts/components/ropebridgemanager.lua#L260-L287)
- 重要副作用：创建/破坏直接 `SetTile` 改世界地图并联动 undertile（保留原瓦片）与 TempTile 警告广播；血量到 0 自动排破坏，地震按 `quake_blocker`（凹陷支柱）保护整条线。[地震保护](../../../raw/current/scripts/components/ropebridgemanager.lua#L43-L88)
- 修改边界：网格在世界 `worldmapsetsize` 后按地图尺寸初始化；OnSave 压缩保存两张网格，OnLoad 重启进行中的破坏并重建 fx/伤害贴片，兼容旧版纯血量存档。[存读](../../../raw/current/scripts/components/ropebridgemanager.lua#L333-L379)

## 关键接口

| 接口 | 条件、作用与证据 |
|---|---|
| `QueueCreateRopeBridgeAtPoint(x,y,z,data?)` | 预登记后按 `base_time+random*rand`（默认 0.5+0.3）延迟建格；data 可给 base_time/random_time/direction/icon_offset。已有数据的瓦片忽略（防重复排队）。[实现](../../../raw/current/scripts/components/ropebridgemanager.lua#L169-L184) |
| `CreateRopeBridgeAtTile(tile_x,tile_y,x?,z?,direction,icon_offset?)` | `SetTile` 为 ROPE_BRIDGE（先记 undertile，源码注明顺序是绕过 SetTile 清 undertile 的 terraforming 回调），duration 网格写 `{ROPEBRIDGE_HEALTH(200), direction}`，生成 rope_bridge_fx（按 direction 定 0/180/±90 朝向）。[实现](../../../raw/current/scripts/components/ropebridgemanager.lua#L134-L167) [血量](../../../raw/current/scripts/tuning.lua#L8466-L8469) |
| `DamageRopeBridgeAtTile(tx,ty,damage)` | 血量钳制 [0,200] 后更新 dock_damage 贴片百分比；血量 0 时以 `DEFAULT_BREAKDATA`（3 秒特效、破坏 +70 帧）排破坏；无数据或已 0 返回 nil，否则返回新血量。[实现](../../../raw/current/scripts/components/ropebridgemanager.lua#L265-L287) [修复消费](../../../raw/current/scripts/prefabs/dock_damage.lua#L22-L41) |
| `QueueDestroyForRopeBridgeAtPoint(x,y,z,data?)` | 标记 marked_for_delete，默认 2+70~80 帧后销毁并广播瓦片变更警告；quake 路径额外在 shaketime 抖动、fxtime 警告。[实现](../../../raw/current/scripts/components/ropebridgemanager.lua#L225-L258) |
| `DestroyRopeBridgeAtPoint(x,y,z,data?)` | 非 ROPE_BRIDGE 瓦片返回 false；恢复 undertile 记录的原瓦片（无记录则 `IMPASSABLE`），清四张网格并 KillFX。[实现](../../../raw/current/scripts/components/ropebridgemanager.lua#L186-L223) |
| `OnStartQuake`（经 `startquake` 世界 net 事件） | 地震延迟 debrisperiod 后对全桥执行 `OnQuaked`：沿线（方向向量，最长 `ROPEBRIDGE_LENGTH_TILES`=6 格）检测 `quake_blocker` 标签实体（半径 `QUAKE_BLOCKER_RANGE`），保护判定作用于整条桥，未保护的每格扣 `ROPEBRIDGE_EARTHQUAKE_DAMAGE_TAKEN`(50)。[注册](../../../raw/current/scripts/components/ropebridgemanager.lua#L110-L112) [伤害](../../../raw/current/scripts/components/ropebridgemanager.lua#L89-L102) [保护扫描](../../../raw/current/scripts/components/ropebridgemanager.lua#L43-L88) |

## 典型接入

### 用套件协议扩展绳桥

- 前提与选择：服务端、洞穴世界。批量铺桥应照抄 rope_bridge 套件的写法——逐格 QueueCreate 并给递增 base_time（原版每格 0.25 秒递增产生“铺开”节奏）与中点 icon_offset；直接调 CreateRopeBridgeAtTile 会立即成格、没有过渡。[套件写法](../../../raw/current/scripts/prefabs/rope_bridge.lua#L340-L352)
- 生效与撤销：QueueCreate 预登记即占位（重复排队被忽略）；撤销=QueueDestroy，会自动恢复 undertile 原瓦片。[防重](../../../raw/current/scripts/components/ropebridgemanager.lua#L172-L183)

```lua
-- 服务端（洞穴世界）：在 (x,y,z) 排队生成一格朝 +x 的绳桥
TheWorld.components.ropebridgemanager:QueueCreateRopeBridgeAtPoint(
    x, y, z,
    { base_time = 0.5, random_time = 0.0, direction = Vector3(1, 0, 0) })
```

- 调整与验证：`direction` 用于地震保护扫描与 fx 朝向，必须与实际桥线一致；验证改瓦片可见、`DamageRopeBridgeAtPoint` 返回值递减（[验证口径](../../../raw/current/scripts/components/ropebridgemanager.lua#L285)）。

## 常见陷阱与限制

- **无 undertile 时拆桥留洞：** Destroy 恢复瓦片依赖 undertile 组件记录；Create 时若 undertile 不存在（`if undertile` 守卫）则原瓦片未保存，销毁后瓦片变成 `IMPASSABLE`（虚空）。自定义世界装桥必须确认 undertile 存在。[守卫](../../../raw/current/scripts/components/ropebridgemanager.lua#L136-L147) [恢复缺省](../../../raw/current/scripts/components/ropebridgemanager.lua#L200-L208)
- **伤害接口只认登记过的瓦片：** 直接 SetTile 出来的 ROPE_BRIDGE 瓦片没有 duration 数据，Damage 返回 nil 且地震不对其结算；务必走 Create/QueueCreate。[早退](../../../raw/current/scripts/components/ropebridgemanager.lua#L269-L272)
- **保护按“方向线”而非半径圆：** quake_blocker 只需位于同一条桥线上任意一格的 `QUAKE_BLOCKER_RANGE` 内，整条线（至多 6 格）全部免疫；垂直相邻的另一条桥不受保护。[线扫描](../../../raw/current/scripts/components/ropebridgemanager.lua#L55-L87)
- **破坏计时在存档中重启：** mid-destruction 的格子在 OnLoad 按 1～10 帧随机重启销毁任务，不会保留原进度；地震中的 3 秒特效也不跨档。[重启](../../../raw/current/scripts/components/ropebridgemanager.lua#L352-L359)
- **修复链沿四方向传染：** dock_damage 的修复会递归修复相邻绳桥格；把 dock_damage 预制件用于非桥场景时注意该传染。[递归](../../../raw/current/scripts/prefabs/dock_damage.lua#L15-L21)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 ropebridgemanager.lua，核验创建/伤害/破坏/地震全链路、undertile 联动、存读兼容与 rope_bridge 套件、dock_damage 修复两个消费方；核对 cave 安装与 TUNING。未运行游戏；quake_blocker 的提供者（support_pillar）与地震组件 caveins 的协作仅核验事件名与标签。

相关：[[reference/components/caveins]]、[[reference/components/undertile]]。
