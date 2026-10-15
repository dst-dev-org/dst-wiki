---
title: "绳桥与藤桥的桥面表现（rope_bridge_fx、vine_bridge_fx、vine_bridge_decor_fx）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "两条桥面表现链：绳桥（洞穴玩家套件搭建，双绳本地子实体＋netvar 位编码同步，地震可打坏）与藤桥（温妮玫瑰眼镜的临时桥，主桥＋装饰子实体，创建即排定约 18 秒警告与约 20⅓ 秒真正拆除）；可通行面是地图瓦片，FX 实体只是视觉层，均由世界管理器生成与销毁。"
aliases: ["rope_bridge_fx", "vine_bridge_fx", "vine_bridge_decor_fx", "绳桥", "藤桥", "桥面贴片", "绳桥 FX", "藤桥装饰"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/rope_bridge.lua", "prefabs/vine_bridge.lua", "components/ropebridgemanager.lua", "components/vinebridgemanager.lua", "componentutil.lua", "components/roseinspectableuser.lua", "prefabs/winona.lua", "prefabs/cave.lua", "prefabs/world.lua", "prefabs/dock_damage.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 绳桥与藤桥的桥面表现（rope_bridge_fx、vine_bridge_fx、vine_bridge_decor_fx）

## 快速判断

- 职责与适用：本页收拢 **3 个桥面表现注册**，分属两个工厂：`rope_bridge_fx` 是洞穴绳桥的桥面＋双绳表现（玩家用 `rope_bridge_kit` 搭建、可被地震打坏）；`vine_bridge_fx`＋`vine_bridge_decor_fx` 是地表藤桥的桥面＋散落藤叶装饰（温妮玫瑰眼镜的临时桥，创建时就排定警告与拆除时点）。可通行面都不是 FX 实体——管理器先把地图瓦片改成 `WORLD_TILES.ROPE_BRIDGE`／`CHARLIE_VINE`，FX 只是瓦片上的视觉层。[绳桥注册](../../../raw/current/scripts/prefabs/rope_bridge.lua#L583)、[藤桥注册](../../../raw/current/scripts/prefabs/vine_bridge.lua#L160-L161)
- 重要副作用：两座桥的 FX 实体都 `persists=false`——存档只存管理器的格子数据，读档按瓦片重铺 FX；绳桥的双绳、藤桥的装饰都是**非专用端本地或宿主子实体**，跨端同步只靠宿主身上的小 netvar。
- 修改边界：只覆盖 3 个注册的工厂、生成点、父挂与清理；瓦片寻路属性（`IsValidTileForRopeBridgeAtPoint` 等）、地震系统与玫瑰眼镜技能树不展开。
- 名称：套件有官方名 `STRINGS.NAMES.ROPE_BRIDGE_KIT`＝"洞穴探险者的桥梁套件"（[po](../../../raw/current/scripts/languages/chinese_s.po#L348789-L348792)）；`ROPE_BRIDGE_FX`／`VINE_BRIDGE_FX`／`VINE_BRIDGE_DECOR_FX` 无 `STRINGS.NAMES.*` 键，标题中的"绳桥""藤桥""藤叶装饰"是已核验的效果描述，**非官方译名**。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `rope_bridge_fx` | `fn`：Transform／AnimState／SoundEmitter／Network → 八向、`FX` 标签 → bank／build `rope_bridge`、`bridge_place`、贴地＋`LAYER_BACKGROUND`＋排序 1 → netvar `iconoffset`（tinybyte）与 `animdata`（smallbyte）→ Pristine → 客户端听 `animdatadirty`／`iconoffsetdirty`；主模拟 `persists=false`＋导出 `SetIconOffset`／`SkipPre`／`ShakeIt`／`KillFX` | 联网壳只带桥面动画；两条绳索是客户端按 `animdata` 重建的本地子实体。主模拟还有 0 帧放置音与放置→待机动画推进。[工厂](../../../raw/current/scripts/prefabs/rope_bridge.lua#L231-L283)、[netvar](../../../raw/current/scripts/prefabs/rope_bridge.lua#L250-L251) |
| `vine_bridge_fx` | `fn`：同壳（bank／build `vine_bridge`、`bridge1_pre`）→ Pristine 前播环境循环音 `vine_bridge_lp`（各端本地）→ 主模拟随机 1–3 变体、`bridge{var}_pre` 接 idle、`persists=false`、4–6 个延时 `SpawnDecor`、导出 `SkipPre`／`KillFX`／`ShakeIt` | 无自定义 netvar——变体与动画全靠引擎的联网动画同步；装饰生成走真实 `SpawnPrefab`（联网子实体）。[工厂](../../../raw/current/scripts/prefabs/vine_bridge.lua#L75-L121) |
| `vine_bridge_decor_fx` | `decorfn`：Transform／AnimState／Network → **`DECOR` 标签（非 FX）** → bank／build `vine_bridge`、`extra_1_pre` → 主模拟随机 1–3 变体、50% 概率 `SetScale(-1,1)` 镜像、`persists=false` | 无自删、无宿主 API；生命期完全挂在藤桥实体下。[装饰工厂](../../../raw/current/scripts/prefabs/vine_bridge.lua#L125-L158) |

## 关联机制导航

### 生成点与宿主链

| 生成点 | 涉及注册 | 端、条件与调用 | 证据 |
|---|---|---|---|
| `ropebridgemanager:SpawnBridgeAnim` | `rope_bridge_fx` | 主模拟；`CreateRopeBridgeAtTile` 改瓦片后按格子生成（全库唯一生成点），写位置／按方向旋转／`SetIconOffset`，存进 `bridge_anims_grid`；世界生成期（POPULATING）改调 `SkipPre` 跳过放置动画 | [生成](../../../raw/current/scripts/components/ropebridgemanager.lua#L309-L327)、[改瓦片](../../../raw/current/scripts/components/ropebridgemanager.lua#L134-L167) |
| `rope_bridge_kit` 的 `OnDeploy` | `rope_bridge_fx`（间接） | 主模拟；套件（绳×2＋木板×4 配方）部署时按落点串逐格 `QueueCreateRopeBridgeAtPoint`（延时 0.25s/格），管理器再走上表生成 | [部署](../../../raw/current/scripts/prefabs/rope_bridge.lua#L321-L355)、[排队](../../../raw/current/scripts/components/ropebridgemanager.lua#L169-L184) |
| `vinebridgemanager:SpawnBridgeAnim` | `vine_bridge_fx` | 主模拟；`CreateVineBridgeAtTile` 改瓦片后生成（全库唯一生成点）；旋转映射与绳桥不同（x→±90、z→180/0） | [生成](../../../raw/current/scripts/components/vinebridgemanager.lua#L251-L268)、[改瓦片](../../../raw/current/scripts/components/vinebridgemanager.lua#L71-L107) |
| 玫瑰眼镜 "Vine Bridge" 玫瑰点 | `vine_bridge_fx`（间接） | 主模拟；只有温妮装 `roseinspectableuser`，对查理残留激活玫瑰点 → `RosePoint_VineBridge_Do` 逐格 `QueueCreateVineBridgeAtPoint` 并**同时**排定 `QueueDestroyForVineBridgeAtPoint`：`fxtime`＝18s＋0.25s×i 时警告，`destroytime`＝警告后 70 帧（相对该格生成约 20⅓ 秒）真正销毁 | [玫瑰点配置](../../../raw/current/scripts/componentutil.lua#L655-L690)、[激活入口](../../../raw/current/scripts/components/roseinspectableuser.lua#L113-L135)、[温妮装配](../../../raw/current/scripts/prefabs/winona.lua#L205) |
| 藤桥 `SpawnDecor` | `vine_bridge_decor_fx` | 主模拟；桥生成后 0.9–1.1s，4–6 枚装饰 `SpawnPrefab`＋`SetParent(桥)`＋随机偏移 (−2..2)，索引存回 `inst.decor` | [SpawnDecor](../../../raw/current/scripts/prefabs/vine_bridge.lua#L14-L20)、[排队](../../../raw/current/scripts/prefabs/vine_bridge.lua#L109-L112) |

### 父子、同步与资源

| 对象 | 怎么用 | 条件与边界 | 证据 |
|---|---|---|---|
| 绳索子实体（非注册 prefab） | 客户端 `OnAnimData` 首次收数据时造两个 `CreateRope` 本地实体（`NOCLICK`＋`decor`、非联网），`SetParent` 到桥上，局部偏移 (−0.1,0,±2.6) | 只在非专用端存在；`animdata` 位编码：绳 1 变体（bit 5–6）＋绳 2 变体（bit 3–4）＋动画 id（bit 1–2，place/idle/shake/break），按变体播 `rope_place_`／`rope_support_`／`rope_shake_`／`rope_break_` | [OnAnimData](../../../raw/current/scripts/prefabs/rope_bridge.lua#L77-L111)、[CreateRope](../../../raw/current/scripts/prefabs/rope_bridge.lua#L39-L57)、[位表](../../../raw/current/scripts/prefabs/rope_bridge.lua#L64-L75) |
| 小地图图标（非注册 prefab） | 主模拟 `SetIconOffset` 写 `iconoffset`（0 无／1 居中／2 偏移 (2,0,0)），`OnIconOffset` 建／删 `CreateMinimapIcon`（非联网 `CLASSIFIED` 图标，父挂桥上） | 专用服务器也执行一次以缓存图标；netvar 名字符串源码即拼作 `"rope_bridge_fx.iconffset"`（拼写如此，非本页笔误） | [图标](../../../raw/current/scripts/prefabs/rope_bridge.lua#L189-L221)、[netvar](../../../raw/current/scripts/prefabs/rope_bridge.lua#L250) |
| 藤桥装饰父子 | 装饰是联网实体、引擎子体挂藤桥；`inst.decor` 表**先存任务后存实体**——未到时的槽位是 `DoTaskInTime` 句柄，宿主 API 用 `is_a(EntityScript)` 区分 | SkipPre 遇到未到时槽位会 `Cancel` 任务并重新 `SpawnDecor`；KillFX 对未到时槽位只取消不补生成 | [SkipPre](../../../raw/current/scripts/prefabs/vine_bridge.lua#L22-L38)、[KillFX](../../../raw/current/scripts/prefabs/vine_bridge.lua#L40-L59) |
| 管理器数据 | 两管理器都是世界组件；绳桥只在洞穴装配，藤桥组件在共同 `MakeWorld` 的主模拟分支无地表条件挂载，地表与洞穴都会装上。用 `duration_grid`（血量＋方向）／`marked_for_delete_grid`／`bridge_anims_grid` 按瓦片记账，OnSave／OnLoad 存读格子后重铺 FX | `ropebridgemanager` 只在 [cave.lua](../../../raw/current/scripts/prefabs/cave.lua#L556) 装配；`vinebridgemanager` 在 [world.lua](../../../raw/current/scripts/prefabs/world.lua#L586) 的共同工厂装配，`cave.lua` 也 [调用 `MakeWorld`](../../../raw/current/scripts/prefabs/cave.lua#L581-L586)，组件可用范围不限于地表；当前已核验的原版藤桥触发场景仍是地表玫瑰点。绳桥读档另铺 `dock_damage` 损伤层 | [绳桥读档](../../../raw/current/scripts/components/ropebridgemanager.lua#L342-L379)、[藤桥读档](../../../raw/current/scripts/components/vinebridgemanager.lua#L283-L320) |
| 伤害与修理 | 绳桥：地震推 `startquake` 后按保护扫描 `DamageRopeBridgeAtPoint`（每次 50，血量 200），相邻船坞损伤层可修理（负伤害）；藤桥：`DamageVineBridgeAtPoint` 脚本域无调用者，损伤层生成代码被注释禁用，血量仅记账 | 血量 0 → `QueueDestroy`：无自定义 breakdata 时警告立即发生，销毁延时为 `2+(70+math.random(0,10))*FRAMES`（约 4.3～4.7 秒），并非固定 2 秒；自定义 breakdata 则按 `shaketime`／`fxtime`／`destroytime` 排警告与销毁 | [地震](../../../raw/current/scripts/components/ropebridgemanager.lua#L89-L109)、[损伤层](../../../raw/current/scripts/components/ropebridgemanager.lua#L289-L307)、[修理](../../../raw/current/scripts/prefabs/dock_damage.lua#L15-L17)、[藤桥伤害未接线](../../../raw/current/scripts/components/vinebridgemanager.lua#L231-L249) |
| 资源 | `anim/rope_bridge.zip`、`anim/vine_bridge.zip`；音效 `rifts4/rope_bridge/*` 与 `meta4/charlie_residue/vine_bridge_*` | 只核验声明与工厂字符串；包内帧数未解包 | [绳桥资源](../../../raw/current/scripts/prefabs/rope_bridge.lua#L3-L12)、[藤桥资源](../../../raw/current/scripts/prefabs/vine_bridge.lua#L1-L4) |

### 清理路径

| 注册 | 清理方式 | 边界与证据 |
|---|---|---|
| `rope_bridge_fx`（完好拆除） | 管理器 `DestroyRopeBridgeAtPoint`：移除损伤层 → 瓦片还原（undertile 优先，否则 IMPASSABLE）→ 清格子 → `fx:KillFX()` | 拆除动因：血量归零排程、地震排程、调试命令。[销毁](../../../raw/current/scripts/components/ropebridgemanager.lua#L186-L223) |
| `rope_bridge_fx`（KillFX 自身） | 播 `break_<1-3>`、转 Billboard＋`LAYER_BELOW_GROUND`、播碎裂音、写 break 位码（双绳转 `rope_break_<var>` 并解除父挂）→ 实体监听 `animover`／`OnEntitySleep` 自删 | 自删是"醒着等动画、睡着即删"双路径；`killed` 标志防重入。[KillFX](../../../raw/current/scripts/prefabs/rope_bridge.lua#L151-L168)、[绳索 KillFX](../../../raw/current/scripts/prefabs/rope_bridge.lua#L23-L37) |
| `vine_bridge_fx` | 管理器 `DestroyVineBridgeAtPoint` → `fx:KillFX()`：停循环音、播 `vine_bridge_pst`、装饰播 `extra_{var}_pst` 或取消任务 → 桥监听 `animover`／`OnEntitySleep` 自删 | 装饰没有 animover 自删，随父体移除回收（引擎父子，未运行验证）；创建即排定的约 18 秒警告与约 20⅓ 秒销毁是最常见动因。[KillFX](../../../raw/current/scripts/prefabs/vine_bridge.lua#L40-L59)、[销毁](../../../raw/current/scripts/components/vinebridgemanager.lua#L125-L162) |
| 读档中途拆除 | 管理器 OnSave 存 `marked_for_delete`，OnLoad 对标记瓦片重新排程销毁 | 保证"读档前正在塌的桥读档后继续塌"。[绳桥续排](../../../raw/current/scripts/components/ropebridgemanager.lua#L348-L360) |

## 典型接入

### 复刻"瓦片＋格子记账＋贴片实体"的世界设施表现

- 前提与选择：需要"可通行面跟随地图瓦片、实体可随时重建"的桥／路类设施时，复用本模式：世界组件持有 DataGrid（血量、方向），把瓦片改成专用 `WORLD_TILES`，再按格生成纯视觉 FX；`persists=false` 让实体不进存档，读档按格子重铺。
- 生效与撤销：生成走 `SpawnBridgeAnim` 幂等（有实体则跳过）；撤销必须先还原瓦片、再清格子、最后 `fx:KillFX()`，顺序颠倒会留下"看得见走不过去"的错位表现。[绳桥生成](../../../raw/current/scripts/components/ropebridgemanager.lua#L309-L327)、[绳桥销毁](../../../raw/current/scripts/components/ropebridgemanager.lua#L186-L223)
- 原版依据：[ropebridgemanager 全文](../../../raw/current/scripts/components/ropebridgemanager.lua)、[vinebridgemanager 全文](../../../raw/current/scripts/components/vinebridgemanager.lua)

```lua
local function build_bridge_at(world, x, z, direction)
    local mgr = world.components.ropebridgemanager
    if mgr ~= nil then
        -- 主模拟调用；延时 0.5～0.8s（base_time＋random_time×math.random()）后改瓦片并生成桥面 FX
        mgr:QueueCreateRopeBridgeAtPoint(x, 0, z, { base_time = 0.5, direction = direction })
    end
end
```

- 调整与验证：`direction` 是 {x,z} 单位向量，决定旋转与相邻格搜索；示例只传 `base_time=0.5`，`random_time` 默认 0.3，因此实际是 0.5～0.8 秒而非固定 0.5 秒，需要固定延时须补 `random_time=0`。藤桥换成 `vinebridgemanager` 且调用方需自排 `QueueDestroyForVineBridgeAtPoint`（原版由玫瑰点配置排定警告与销毁）。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：桥的可通行性不在 FX 实体上。** 走不走得过去由瓦片类型与寻路决定；直接 `SpawnPrefab("rope_bridge_fx")` 只会得到一块能看的桥面，不改瓦片、不进管理器格子，也不会被地震／拆除流程管理。[改瓦片](../../../raw/current/scripts/components/ropebridgemanager.lua#L141-L147)
- **边界：绳桥双绳在专用服务器上不存在。** `OnAnimData` 建绳走 `TheNet:IsDedicated()` 早退；绳索是非联网本地实体，服务端只有 `animdata` 位码。自定改绳索动画必须走位码或新 netvar，直接改服务端 AnimState 客户端看不见。[早退](../../../raw/current/scripts/prefabs/rope_bridge.lua#L77-L82)
- **边界：藤桥表现同步靠引擎动画复制，没有变体 netvar。** 变体、镜像缩放、随机起始帧都是主模拟写入、引擎同步；客户端 `SkipPre`（POPULATING 分支）只发生在主模拟。与绳桥"本地重建"是两种相反的同步策略。[藤桥工厂](../../../raw/current/scripts/prefabs/vine_bridge.lua#L96-L112)、[绳桥位码](../../../raw/current/scripts/prefabs/rope_bridge.lua#L92-L95)
- **边界：藤桥装饰表存放两种对象。** `inst.decor` 在生成前是 `DoTaskInTime` 句柄、生成后是实体；遍历时必须先 `is_a(EntityScript)` 判断，直接调 `AnimState` 会在任务槽位上炸。[判断](../../../raw/current/scripts/prefabs/vine_bridge.lua#L25-L32)
- **边界：藤桥的损伤显示被禁用。** `SpawnDamagePrefab` 的生成段整块注释掉，`VINEBRIDGE_HEALTH=200` 只在内存记账、血量归零即排定拆除；可见的"损伤贴片"只有绳桥有（`dock_damage`）。[注释段](../../../raw/current/scripts/components/vinebridgemanager.lua#L236-L242)、[绳桥损伤层](../../../raw/current/scripts/components/ropebridgemanager.lua#L294-L300)
- **误区：藤桥会一直存在。** 温妮玫瑰点在 `QueueCreate` 的同时就为每格排了 `QueueDestroyForVineBridgeAtPoint`：`fxtime`＝18 秒＋0.25 秒×i 时警告，`destroytime`＝警告后 70 帧（相对该格生成约 20⅓ 秒）真正销毁；管理器 API 本身不限制存在时长，限时是调用方约定。[排定](../../../raw/current/scripts/componentutil.lua#L665-L681)、[时长](../../../raw/current/scripts/tuning.lua#L3748-L3749)
- **边界：KillFX 的睡眠直删是共性，`killed` 防重入只属绳桥。** 两座桥都在实体睡眠时直接 `Remove`，醒着播退场动画并设 `OnEntitySleep=Remove`。绳桥另有 `killed` 标志，重复 `KillFX` 被拦下；藤桥的 `KillFX` 与 `ShakeIt` 都没有该检查，重复调用会重播退场或覆盖当前动画，不能概括成同一协议。[绳桥](../../../raw/current/scripts/prefabs/rope_bridge.lua#L151-L168)、[藤桥](../../../raw/current/scripts/prefabs/vine_bridge.lua#L40-L73)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/rope_bridge.lua`、`prefabs/vine_bridge.lua`、`components/ropebridgemanager.lua`、`components/vinebridgemanager.lua`，按需读取 `componentutil.lua` 玫瑰点段、`roseinspectableuser.lua` 激活入口、`dock_damage.lua` 修理段与两处世界装配行。对 3 个完整注册名以 `grep -rn "注册名" --include=*.lua` 在 `raw/current/scripts` 全域反查：`rope_bridge_fx` 命中 5 处（注册、kit 依赖表、cave.lua 依赖表、工厂内两 netvar 字符串、`ropebridgemanager:312` 生成点）；`vine_bridge_fx` 命中 4 处（注册、world.lua 依赖表、`vinebridgemanager:254` 生成点、netvar 无）；`vine_bridge_decor_fx` 命中 3 处（注册、`vine_bridge_fx` 依赖表、`vine_bridge.lua:15` 生成点）——三注册名均无其他生成点。调试命令（`debugcommands.lua`）对绳桥的批量拆除为非玩法入口，本页不展开。`TUNING.ROPEBRIDGE_HEALTH=200`、`ROPEBRIDGE_LENGTH_TILES=6`、`ROPEBRIDGE_EARTHQUAKE_DAMAGE_TAKEN=50`、`ROPEBRIDGE_EARTHQUAKE_TIMETOBREAK=3`、`VINEBRIDGE_DURATION=18`、`VINEBRIDGE_HEALTH=200`、`SKILLS.WINONA.CHARLIE_VINEBRIDGE_LENGTH_TILES=3` 已核对；另核对 `QueueCreateRopeBridgeAtPoint` 默认 `base_time=0.5`／`random_time=0.3`、玫瑰点 `fxtime`／`destroytime`，以及两管理器无自定义 data 时的默认销毁延时 `2+(70+math.random(0,10))*FRAMES`。
- 名称核验：`languages/chinese_s.po` 定向检索——`STRINGS.NAMES.ROPE_BRIDGE_KIT`＝"洞穴探险者的桥梁套件"（锚点见"快速判断"）；`ROPE_BRIDGE_FX`／`VINE_BRIDGE_FX`／`VINE_BRIDGE_DECOR_FX` 无 `STRINGS.NAMES.*` 键，标题使用效果描述。
- 分档说明：`size_class: standard`（3 注册、两座管理器驱动的桥、本地重建与引擎同步两种策略、多条清理路径，单页可承载），实测行数与字节见结构检查。
- 未覆盖：瓦片寻路与 `IsValidTileFor*` 地图函数；地震系统全貌与 `quake_blocker` 保护；玫瑰眼镜技能树与查理残留玩法；绳索／装饰动画包内容与观感；读档重铺时 POPULATING 分支的完整时序。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录（0 error 0 warning 后入库）。

## 相关链接

- [[reference/components/ropebridgemanager|ropebridgemanager]]、[[reference/components/vinebridgemanager|vinebridgemanager]]（两条生成与销毁链的权威机制页）
- [[reference/prefabs/ice_crack_grid_fx|海冰网格裂纹]]（同为管理器按瓦片铺贴片；对照查重与清除策略）
- [[reference/prefabs/explode_small|小爆裂、换装烟与火药爆裂]]（同为一次性／多注册的贴地表现族）
- [[reference/components/undertile|undertile]]（桥拆除时瓦片还原的协作组件）
- 返回 [[reference/prefabs/fx-structure-index|建筑、装置与遗迹演出]]
