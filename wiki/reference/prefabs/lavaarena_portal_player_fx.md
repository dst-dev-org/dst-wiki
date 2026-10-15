---
title: 熔炉登场、复活与场景余烬特效（lavaarena_portal_player_fx 等）
created: 2026-11-01
updated: 2026-11-01
type: prefab
summary: 熔炉（The Forge/lavaarena）出身的 6 个通用表 FX：玩家登场传送门（树内无生成点，接线在事件数据包）、尸体复活（REVIVE_CORPSE 动作链父挂玩家实体）、生物传送出生烟（SpawnSmokeFx 随机拼名 3 注册）、竞技场熔渣格栅余烬（服务端周期父挂）；全部纯表现，无玩法作用。
aliases: ["lavaarena_portal_player_fx", "lavaarena_player_revive_from_corpse_fx", "lavaarena_creature_teleport_smoke_fx_1", "lavaarena_creature_teleport_smoke_fx_2", "lavaarena_creature_teleport_smoke_fx_3", "ember_short_fx", "熔炉传送门特效", "玩家登场特效", "尸体复活特效", "传送烟雾", "出生烟", "余烬特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/fx.lua", "prefabs/lavaarena_portal.lua", "prefabs/player_common_extensions.lua", "prefabs/player_common.lua", "components/revivablecorpse.lua", "actions.lua", "prefabs/lavaarena_creature_spawn_fx.lua", "prefabs/lavaarena_floorgrate.lua", "map/static_layouts/events/lava_arena.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 熔炉登场、复活与场景余烬特效（lavaarena_portal_player_fx 等）

## 快速判断

- 职责与适用：`fx.lua` 通用注册表中熔炉（The Forge / lavaarena）出身的 6 个动画特效，按生成链分四组：玩家登场传送门 1 名、玩家尸体复活 1 名、生物传送出生烟 3 名（随机拼名）、竞技场熔渣格栅余烬 1 名。公共工厂协议（网络代理＋非专服本地实体、父挂语义、回收边界）只在 [[reference/mechanisms/generic-fx-factory#关键流程|通用 FX 工厂·关键流程]] 维护，本页只写注册差异与生成者。
- 优先使用：查"熔炉玩家登场／尸体复活表现／刷怪出生烟／余烬"选本页；查生物传送柱动画本体（非烟）用 [[reference/prefabs/lavaarena_creature_teleport_small_fx|熔炉生物传送出生特效]]，两族由同一文件（lavaarena_creature_spawn_fx）先后生成。
- 重要副作用：全部为纯表现 FX——注册字段只含动画、声音、泛光与朝向，无伤害、照明或标签玩法作用；尸体复活的实际重生逻辑在 `DoActualRezFromCorpse`，FX 只在那一刻父挂出场。[生成点](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L433-L438)
- 修改边界：`lavaarena_portal_player_fx` 在本脚本树内没有生成调用（生成在熔炉事件数据包）；其余 5 名的生成点均在树内核验。FX 条目均为通用表数据，增删改走 fx.lua 表项＋工厂协议。[事件数据接线](../../../raw/current/scripts/prefabs/lavaarena_portal.lua#L78)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lavaarena_portal_player_fx` | 通用表条目 → prefabs/fx.lua `MakeFx`（协议见工厂页），无专属 fn | bank/build `lavaarena_player_teleport`，anim `idle`（源码注释：开头 6 空帧用于声画对齐），sound `dontstarve/common/lava_arena/portal_player`，bloom。[注册](../../../raw/current/scripts/fx.lua#L880-L885) |
| `lavaarena_player_revive_from_corpse_fx` | 同上；fn=`FinalOffset1`（`SetFinalOffset(1)`，[定义](../../../raw/current/scripts/fx.lua#L1-L3)） | bank/build `lavaarena_player_revive_fx`，anim `player_revive`，sound `dontstarve/common/revive`，bloom＋fourfaced＋autorotate（有父级时一次性复制父级旋转）。[注册](../../../raw/current/scripts/fx.lua#L888-L896) |
| `lavaarena_creature_teleport_smoke_fx_1`／`_2`／`_3` | 同上，三注册仅 anim 不同 | bank/build `lavaarena_creature_teleport_smoke_fx`，anim `smoke_1/2/3`；无声音、无 bloom、无 fn，纯落点烟。[注册](../../../raw/current/scripts/fx.lua#L908-L911)、[913-916](../../../raw/current/scripts/fx.lua#L914-L916)、[919-922](../../../raw/current/scripts/fx.lua#L920-L922) |
| `ember_short_fx` | 同上；`animqueue=true`（听 animqueueover 回收），fn 追加队列 `PushAnimation("loop", false)`、`PushAnimation("pst", false)` | bank `ember_particles`、build `lavaarena_ember_particles_fx`，anim `pre`，bloom；pre→loop→pst 队列播完自删。[注册](../../../raw/current/scripts/fx.lua#L899-L905) |

六个注册名均无 `nameoverride`／NAMES 显示名条目，`languages/chinese_s.po` 定向检索无对应显示名键；标题中文为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 公共工厂协议 | 6 名全部经 `MakeFx`：主模拟隐形代理、非专服一帧后建本地动画实体；代理 1 秒自删，本地实体 animover／animqueueover 自删 | 专服无本地表现；父挂式本地实体成为宿主子实体 | [[reference/mechanisms/generic-fx-factory#关键流程|协议·关键流程]]、[加载器](../../../raw/current/scripts/prefabs/fx.lua#L165-L172) |
| 生成者：lavaarena_portal | `prefabs` 数组依赖声明 `lavaarena_portal_player_fx`；master_postinit 经 `event_server_data("lavaarena", ...)` 接线 | 树内无 `SpawnPrefab` 调用，登场时机与条件在事件数据包（本快照不含），不能据依赖声明断定生成行为 | [声明](../../../raw/current/scripts/prefabs/lavaarena_portal.lua#L10)、[接线](../../../raw/current/scripts/prefabs/lavaarena_portal.lua#L78) |
| 生成者：尸体复活链 | `ACTIONS.REVIVE_CORPSE.fn` → `revivablecorpse:Revive` 推事件 `respawnfromcorpse` → `OnRespawnFromPlayerCorpse` 延迟 0 帧 → `DoActualRezFromCorpse` 父挂 FX 到玩家实体（仅当仍带 `corpse` 标签），随后切回 SGwilson 进 `corpse_rebirth` 状态 | 服务端动作触发；FX 父挂式，复活玩家起身动画期间跟随 | [动作](../../../raw/current/scripts/actions.lua#L4133-L4139)、[组件](../../../raw/current/scripts/components/revivablecorpse.lua#L76-L79)、[监听](../../../raw/current/scripts/prefabs/player_common.lua#L2658)、[生成点](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L433-L438)、[调度](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L749-L762) |
| 生成者：SpawnSmokeFx | `lavaarena_creature_spawn_fx.lua` 主模拟 0-0.2 秒随机延迟调用，动态拼名 `"lavaarena_creature_teleport_smoke_fx_"..tostring(math.random(3))` 落点在传送特效原位置 | 服务端生成；与传送柱/装饰火花同文件分工，见该页注册表 | [拼名生成](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L72-L74)、[调度](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L100)、[[reference/prefabs/lavaarena_creature_teleport_small_fx]] |
| 生成者：lavaarena_floorgrate | 服务端 `DoPeriodicTask(21.1 + math.random()*9, SpawnEmber, math.random()*30)`：每 21.1-30.1 秒（首次 0-30 秒）父挂一枚余烬到格栅实体 | 格栅由熔炉竞技场静态布局 `lava_arena.lua` 布置；FX 随格栅实体挂载 | [周期任务](../../../raw/current/scripts/prefabs/lavaarena_floorgrate.lua#L91)、[SpawnEmber](../../../raw/current/scripts/prefabs/lavaarena_floorgrate.lua#L40-L42)、[布局](../../../raw/current/scripts/map/static_layouts/events/lava_arena.lua#L517) |
| 资源 | anim `lavaarena_player_teleport.zip`、`lavaarena_player_revive_fx.zip`、`lavaarena_creature_teleport_smoke_fx.zip`、`lavaarena_ember_particles_fx.zip`（bank/build 引用即用）；音效 `dontstarve/common/lava_arena/portal_player`、`dontstarve/common/revive` | 注册字段已核验引用关系，anim 包内容未解包 | [各注册行](../../../raw/current/scripts/fx.lua#L880-L922) |

## 典型接入

### 复用"周期父挂的环境表现"与"随机拼名变体"

- 前提与选择：模组端 `AddPrefabPostInit` 任意已加载实体或自建实体主模拟逻辑内 `SpawnPrefab`；跟随宿主用父挂式（`entity:SetParent`），一次性落点用 `SetPosition`。
- 生效与撤销：本地表现为各非专服模拟一帧后自动出现、动画队列播完自删，宿主侧无需清理代理；父挂只保证跟随宿主移动，`SetParent` 未经过 `AddChild` 的 `children` 登记，宿主提前移除时引擎是否级联清理未核验，不能当作确定的随父删除。[AddChild](../../../raw/current/scripts/entityscript.lua#L921-L935)、[Remove](../../../raw/current/scripts/entityscript.lua#L1779-L1783)
- 原版依据：父挂＋周期生成同 [floorgrate](../../../raw/current/scripts/prefabs/lavaarena_floorgrate.lua#L40-L42)；随机变体拼名同 [SpawnSmokeFx](../../../raw/current/scripts/prefabs/lavaarena_creature_spawn_fx.lua#L72-L74)。

```lua
local function SpawnEmber(host)
    SpawnPrefab("ember_short_fx").entity:SetParent(host.entity)  -- 父挂式，随宿主
end

local smoke = SpawnPrefab("lavaarena_creature_teleport_smoke_fx_" .. tostring(math.random(3)))
smoke.Transform:SetPosition(x, y, z)                             -- 落点式，三变体随机拼名
```

- 调整与验证：变体名必须对应实际注册名（smoke 仅 1-3）；`ember_short_fx` 的循环段由注册 fn 队列驱动，改名 anim 会破坏 pre→loop→pst 链。

## 常见陷阱与限制

- **边界：传送门特效树内查不到生成点**：对 `lavaarena_portal_player_fx` 在 scripts 根全域（含 prefabs/components/stategraphs/brains/map）rg 仅命中 fx.lua 注册行与 lavaarena_portal 依赖声明；熔炉 prefab 的运行时接线在 `event_server_data("lavaarena", ...)` 事件数据包，本快照不含，登场条件未核验。[接线](../../../raw/current/scripts/prefabs/lavaarena_portal.lua#L78)
- **辨析：出生烟不是传送柱**：`teleport_smoke_fx_1..3` 是传送点落地的短烟；传送柱动画本体是 `lavaarena_creature_teleport_small_fx`／`_medium_fx`（专属工厂文件，含音量衰减协议）。两者由 SpawnSmokeFx 与 makespawnfx 分开生成，勿混用注册名。
- **辨析：同 bank 异注册**：`fumarole_cook_fx`（[fx.lua L3997](../../../raw/current/scripts/fx.lua#L3997-L4005)）复用同一 `lavaarena_creature_teleport_smoke_fx` bank/build，但走 anim 函数随机选段并随机缩放，属独立注册名，消费者与场景均不同。
- **边界：复活特效只在尸体复活动作链出现**：触发源是 `REVIVE_CORPSE` 动作（玩家对尸体施救），与幽灵复活（touchstone／attune 系）无关；FX 无疗效，重生血量比例等玩法归 `revivablecorpse` 与 `DoActualRezFromCorpse`。[动作](../../../raw/current/scripts/actions.lua#L4133-L4139)
- **边界：余烬节奏是服务端周期任务**：间隔含随机量（21.1-30.1 秒、首延迟 0-30 秒），且只在 `lavaarena_floorgrate` 主模拟侧计时；不能当作固定频率或客户端表现。[任务](../../../raw/current/scripts/prefabs/lavaarena_floorgrate.lua#L91)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。fx.lua 6 条注册逐字段核对；4 条生成链（portal 依赖声明与事件接线、REVIVE_CORPSE 动作→事件→父挂生成、SpawnSmokeFx 动态拼名、floorgrate 周期父挂）读到实际 SpawnPrefab 行；对 6 个注册名及动态拼名片段 `teleport_smoke_fx_"` 在 scripts 根全域 rg 反查，除上述生成点外无其他活跃调用（`ember_short_fx`、`portal_player_fx` 无第二生成点）。事件数据包内实际调用参数、anim zip 内容、复活 FX 的动画时长未核验。
- 游戏运行：未运行；validation 保持 source-reviewed。
- 结构检查：validate_page、validate_source_links --strict 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-event-index|节庆、小游戏与事件模式]]。
