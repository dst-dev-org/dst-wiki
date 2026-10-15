---
title: "海洋平台破坏开裂与爆裂声画（fx_boat_crackle 族）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "船沉、码头拆除与海冰破碎共用的 crackle 开裂预警与 pop 爆裂声画：通用表 5 注册的生成管线、boat_crackle 选名字段与端边界。"
aliases: ["fx_boat_crackle", "fx_boat_pop", "fx_dock_crackle", "fx_dock_pop", "fx_ice_crackle", "开裂预警", "码头拆除声画", "海冰破碎", "船沉爆裂"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "mainfunctions.lua", "prefabs/boat.lua", "prefabs/world.lua", "prefabs/yotd_boats.lua", "prefabs/crabking.lua", "prefabs/cannonballs.lua", "prefabs/mortarball.lua", "prefabs/oceanwhirlbigportal.lua", "stategraphs/SGboat.lua", "stategraphs/SGboat_ice.lua", "stategraphs/SGcrabking_cannontower.lua", "components/dockmanager.lua", "components/oceanicemanager.lua", "components/sharkboimanager.lua", "components/explosive.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 海洋平台破坏开裂与爆裂声画（fx_boat_crackle 族）

## 快速判断

- 职责与适用：`fx.lua` 通用表中海洋可行走平台（木质船系、码头、海冰）**破坏流程的开裂预警与爆裂声画 5 注册**：船沉 crackle/pop（2）、码头拆除 crackle/pop（2）、海冰 crackle（1，其 pop 在组外 `fx_ice_pop`）；语义为 crackle＝破坏预警或开裂过程、pop＝破裂瞬间，均纯表现无玩法作用，工厂协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。
- 优先使用：查"船沉断裂声""码头破碎预警""海冰开裂"读本页；草／海带材质船的侵蚀残影与垫料屑在姊妹页 [[reference/prefabs/boat_grass_erode|草／海带船侵蚀与垫料特效]]（两船型不出本族 crackle）。
- 重要副作用：crackle 都是落点独立实体（不父挂平台）；码头／海冰的 crackle 落点后由管理器延迟任务执行破坏，间隔与 fx 无联动。[排队](../../../raw/current/scripts/components/dockmanager.lua#L322-L331)
- 修改边界：全部服务端逻辑生成（SG／世界组件），各模拟端按工厂各自建本地表现；`inst.boat_crackle` 字段按船型选名——冰船换 `mining_ice_fx`，草船／海獭窝船不设该字段。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `fx_boat_crackle` | 无 sound、无 fn 的纯视觉条目 | bank `fx_boat_crack`、build `fx_boat_crackle` 播 `crackle`；沉没分层声在船 SG 时间线（船 emitter），fx 自身无声；[条目](../../../raw/current/scripts/fx.lua#L1831-L1835) |
| `fx_boat_pop` | 无 sound、无 fn | bank/build `fx_boat_pop` 播 `pop`；沉没双声（damage＋sink）在 postsinkfn 的船 emitter；[条目](../../../raw/current/scripts/fx.lua#L1837-L1841) |
| `fx_dock_crackle` | `sound` 字段由工厂加 emitter 播 creak；fn 再 `AddSoundEmitter` 并按 2/14/25/29/33/45/52 帧分层播 `monkeyisland/dock/damage`（intensity 0.1→0.3） | bank/build `fx_dock_crackleandpop` 播 `crackle`；[条目](../../../raw/current/scripts/fx.lua#L2873-L2889) |
| `fx_dock_pop` | 仅 `sound` 字段 | 同 bank/build 播 `pop`，声 `monkeyisland/dock/break`；[条目](../../../raw/current/scripts/fx.lua#L2891-L2896) |
| `fx_ice_crackle` | 无 sound 字段；fn 自建 SoundEmitter 按同帧序分层播 `dontstarve_DLC001/common/iceboulder_hit` | bank/build `fx_ice_crackleandpop` 播 `crackle`；[条目](../../../raw/current/scripts/fx.lua#L3519-L3548) |

5 个注册名无显示名键（`chinese_s.po` 定向检索无 NAMES 条目），标题中文为已核验效果描述，**非官方译名**。相关注册 `fx_ice_pop`（bank/build `fx_dock_crackleandpop`，与码头 pop 同包，声 `iceboulder_smash`，[fx.lua L3273-L3279](../../../raw/current/scripts/fx.lua#L3273-L3279)）是海冰侧的"pop"，不在本批建页清单，生成点在下方海冰小节一并记录；条目归属见 [[reference/prefabs/junk_break_fx|废料／退化特效]]。

## 触发窗口与生成者

### 船沉：crackle 在断开、pop 在离场

- **选名字段**：`inst.boat_crackle` 决定 snapping 生成何物——木质船 `boat`／`boat_pirate`／`boat_ancient` 与龙舟赛船（dragonboat_pack／shadowboat）设 `"fx_boat_crackle"`（[boat.lua L780/L920/L998](../../../raw/current/scripts/prefabs/boat.lua#L779-L998)、[yotd_boats.lua L392](../../../raw/current/scripts/prefabs/yotd_boats.lua#L392)）；冰船设 `"mining_ice_fx"`（[boat.lua L1361](../../../raw/current/scripts/prefabs/boat.lua#L1361)，注册见 [[reference/prefabs/mining_fx|mining_fx 族]]）；草船／海獭窝船不设。全域 `boat_crackle\s*=` 检索仅此 5 处赋值。
- **crackle 时机**：SGboat `death → ready_to_snap（0.75 秒，全船 abandon_ship）→ snapping → popping`；`snapping` 进入即按 `boat_crackle` 在船中心落一枚（[SGboat.lua L41-L72](../../../raw/current/scripts/stategraphs/SGboat.lua#L41-L72)）。
- **pop 时机**：`popping` 或 `InstantlyBreakBoat`（[boat.lua L463-L480](../../../raw/current/scripts/prefabs/boat.lua#L463-L480)；蟹王 [crabking.lua L1039](../../../raw/current/scripts/prefabs/crabking.lua#L1039)、漩涡传送门 [oceanwhirlbigportal.lua L240](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L240)、鲨鱼人 [sharkboimanager.lua L315](../../../raw/current/scripts/components/sharkboimanager.lua#L315) 调用）按 `sinkloot → postsinkfn → inst:Remove()` 收尾；`fx_boat_pop` 由三木质船的 `postsinkfn` 在船中心落点（[boat.lua L796-L801](../../../raw/current/scripts/prefabs/boat.lua#L796-L801)、[L935-L940](../../../raw/current/scripts/prefabs/boat.lua#L935-L940)、[L1014-L1019](../../../raw/current/scripts/prefabs/boat.lua#L1014-L1019)），随即船 Remove；龙舟赛船未定义 postsinkfn，无 pop（[yotd_boats.lua L341-L344](../../../raw/current/scripts/prefabs/yotd_boats.lua#L341-L344)）。
- **冰船辨析**：SGboat_ice 同构且更密——snapping 进入落一枚，时间线 2～82 帧再在平台半径内随机点重复生成共 10 处（[SGboat_ice.lua L6-L14](../../../raw/current/scripts/stategraphs/SGboat_ice.lua#L6-L14)、[L79-L155](../../../raw/current/scripts/stategraphs/SGboat_ice.lua#L79-L155)），合计 11 枚；但字段值是 `mining_ice_fx`，不出本族。

### 码头：crackle 预警 → 延迟拆除 → pop

- **生成链**：`dockmanager:QueueDestroyForDockAtPoint(x, y, z)` 在给定点落 `fx_dock_crackle`，同帧排 `(70 + random(0,10))*FRAMES` 后的拆除任务（[dockmanager.lua L322-L331](../../../raw/current/scripts/components/dockmanager.lua#L322-L331)）；到点 `DestroyDockAtPoint` 还瓦片、对台上实体推 `onsink`；`dont_toss_loot` 不为真时才落 `fx_dock_pop` 与木板／木段碎片（[L307-L317](../../../raw/current/scripts/components/dockmanager.lua#L307-L317)）。
- **入口**：瓦片耐久归零 `DamageDockAtTile`（炸弹 [explosive.lua L55](../../../raw/current/scripts/components/explosive.lua#L55)、炮弹 [cannonballs.lua L136](../../../raw/current/scripts/prefabs/cannonballs.lua#L136)、迫击炮 [mortarball.lua L157](../../../raw/current/scripts/prefabs/mortarball.lua#L157) 经 `DamageDockAtPoint` 进入）；相邻断连级联 `_TestForBreaking`（[dockmanager.lua L292-L305](../../../raw/current/scripts/components/dockmanager.lua#L292-L305)，经 `start_destroy_for_tile` 延迟 1～10 帧回到排队入口）。
- **船体复用**：蟹王炮塔 `breach_thrust` 撞击船平台时直接在炮塔位置落 `fx_dock_crackle` 并扣船血（[SGcrabking_cannontower.lua L215-L224](../../../raw/current/scripts/stategraphs/SGcrabking_cannontower.lua#L215-L224)）——码头预警表现复用于船体受创。

### 海冰：crackle 预警，pop 在组外

- **生成链**：`oceanicemanager:QueueDestroyForIceAtPoint(x, y, z, data)` 落 `fx_ice_crackle` 于 (x, 0, z)，同时按临陆方向铺 `ice_crack_grid_fx` 裂纹，`data.destroytime` 或 `(70 + random(0,10))*FRAMES` 后 `DestroyIceAtPoint`（[oceanicemanager.lua L414-L449](../../../raw/current/scripts/components/oceanicemanager.lua#L414-L449)）；破裂瞬间（非 silent）落组外注册 `fx_ice_pop`、按临陆方向推浮冰 `boat_ice` 并撒冰屑（[L363-L378](../../../raw/current/scripts/components/oceanicemanager.lua#L363-L378)）。
- **入口**：蟹王战斗（即时销毁或排队，[crabking.lua L789-L793](../../../raw/current/scripts/prefabs/crabking.lua#L789-L793)）、鲨鱼人竞技场 [sharkboimanager.lua L643-L645](../../../raw/current/scripts/components/sharkboimanager.lua#L643-L645)、瓦片耐久归零 `DamageIceAtTile`（[oceanicemanager.lua L472-L473](../../../raw/current/scripts/components/oceanicemanager.lua#L472-L473)）。

## 关联机制导航

| 类型／对象 | 本族如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议：`MakeFx`（prefabs/fx.lua） | 5 条目均按公共工厂展开为网络代理＋各端本地动画实体 | 专服只有代理；本地实体 animover 自删 | [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]] |
| 宿主 Prefab：`boat` 系 | `boat_crackle` 字段选名 + `postsinkfn` 落 pop，均挂 SGboat／SGboat_ice 沉没序 | 服务端；默认 SG 绑定 [L586](../../../raw/current/scripts/prefabs/boat.lua#L586-L586)、冰船经 data 指定 SGboat_ice（[L1316](../../../raw/current/scripts/prefabs/boat.lua#L1316-L1316)）；boat 系 Wiki 待建 | [注册段](../../../raw/current/scripts/prefabs/boat.lua#L1639-L1660) |
| 组件：dockmanager | crackle 排队与 pop 拆除的执行者 | 世界组件，服务端 | [[reference/components/dockmanager]] |
| 组件：oceanicemanager | 海冰 crackle、裂纹与延迟破坏的执行者 | 世界组件，服务端 | [[reference/components/oceanicemanager]] |
| 关联 Prefab：`ice_crack_grid_fx` | 海冰预警伴随裂纹（本批外注册） | `QueueDestroyForIceAtPoint` 按临陆方向铺设，去重后落点 | [spawncracks](../../../raw/current/scripts/components/oceanicemanager.lua#L383-L412)；见 [[reference/prefabs/ice_crack_grid_fx|海冰网格裂纹]] |
| 关联 Prefab：`mining_ice_fx` | 冰船 snapping 的替代 crackle | `boat_crackle` 字段选名 | [[reference/prefabs/mining_fx|mining_fx 族]] |
| 资源 | bank/build `fx_boat_crack`（build `fx_boat_crackle`）、`fx_boat_pop`、`fx_dock_crackleandpop`、`fx_ice_crackleandpop` | 声明与使用已核验，包内容未解包 | 各条目行号见注册表 |

## 典型接入

### 复刻"预警 → 延迟破坏"节奏

- 前提与选择：crackle→pop 的间隔由 SG 状态时长或管理器延迟任务决定，fx 之间无联动；模组想复用该节奏可直接调世界组件的排队接口（服务端），无需自己拼两个 fx 的时序。
- 生效与撤销：排队即写删除标记并落 crackle，相邻断连瓦片会级联排队；无公开取消接口，撤销需在延迟窗口内直接操作管理器内部网格，不建议。

```lua
-- modmain.lua：服务端监听自定义事件；调用方须在世界存在后传入码头坐标。
AddPrefabPostInit("world", function(world)
    if not world.ismastersim then return end
    world:ListenForEvent("my_mod_queue_dock_break", function(_, data)
        if data == nil or type(data.x) ~= "number" or type(data.z) ~= "number" then return end
        local manager = world.components.dockmanager
        if manager ~= nil then
            local x, y, z = world.Map:GetTileCenterPoint(data.x, 0, data.z)
            manager:QueueDestroyForDockAtPoint(x, y, z)
        end
    end)
end)
```

- 调整与验证：事件名为示例，调用方需自行在服务端发送并提供目标坐标；延迟帧数 `(70 + random(0,10))*FRAMES` 写死在管理器内（[dockmanager.lua L331](../../../raw/current/scripts/components/dockmanager.lua#L331)）。`dont_toss_loot` 为真时同时抑制 pop FX 和碎片，不抑制已排队的 crackle 与拆除。示例的 `AddPrefabPostInit("world")` 能命中世界实例：`MakeWorld` 的 fn 在 `common_postinit` 之后执行 `inst:SetPrefabName("world")`（[world.lua L555](../../../raw/current/scripts/prefabs/world.lua#L555)），PostInit 先按该名派发；又因 `inst.prefab ~= name`，再按请求名（地表 `forest`）派发一次（[mainfunctions.lua L392-L407](../../../raw/current/scripts/mainfunctions.lua#L392-L407)）。因此 `world` 与 `forest` 两处 Hook 都会命中，`world` 不是按存档 `map.prefab` 查表；未在游戏内执行。

## 常见陷阱与限制

- **误区：fx_boat_crackle 自带断裂声**：注册无 sound／fn，creak 与 damage 分层声都在 SGboat snapping 时间线（船 emitter）播放；fx 只提供开裂动画。
- **边界：crackle 与 pop 不成对联动**：间隔由 SG 状态时长（船沉，animover 驱动）或管理器延迟任务（码头／海冰 70+rand 帧）决定；各自独立落点、独立自删。
- **误区：冰船也用 fx_boat_crackle**：冰船 `boat_crackle = "mining_ice_fx"` 且 SGboat_ice 会在 snapping 时间线重复生成（随机平台内点位）；本族 crackle 仅木质系船与龙舟赛船，草船／海獭窝船两者皆无。
- **边界：fx_dock_crackle 双声源**：工厂 sound 字段（creak）与 fn 手动加 emitter 的分层 damage 声并存；重复 `AddSoundEmitter` 的幂等性是引擎行为、未在脚本快照内核验（原版 sound＋fn 条目均依赖此模式）。只替换 sound 字段不会去掉 fn 里的分层声。
- **边界：海冰无本族 pop**：破裂瞬间的声画是组外注册 `fx_ice_pop`；只做"预警"判断时，看到 crackle 后的破坏由 `DestroyIceAtPoint` 完成，不经过本族任何注册。

## 验证记录

- 源码审阅：2026-11-01，steam-buildid:26335394。fx.lua 5 条目区段全文；生成者 boat.lua（三木质船字段与 postsinkfn、InstantlyBreakBoat 及三个调用方）、SGboat／SGboat_ice 沉没序、yotd_boats dragonboat、dockmanager 排队→拆除全链、oceanicemanager 排队→破坏全链、SGcrabking_cannontower 复用点、外部触发（crabking／sharkboimanager／explosive／cannonballs／mortarball）核验、世界 PostInit 按 `SetPrefabName("world")` 派发；本页 5 名全域反查无其他生成点，`boat_crackle` 赋值全域仅 5 处。未核验：动画包内部时长、`start_destroy_for_tile` 的世界生成触发上下文、多客户端表现。
- 游戏运行：未运行。
- 结构检查：validate_page 通过（0 issue；100 行、13678 字节，standard 档内）；validate_source_links --strict 通过（38 处引用、0 error／0 warning，两处越界锚点已修正后复检）；Lua 示例另存临时文件经 parser_probe --syntax-only 通过（只查语法，未执行，临时文件已删）。

返回 [[reference/prefabs/fx-water-index|水面、航海与船件]]。
