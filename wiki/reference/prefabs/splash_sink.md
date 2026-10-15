---
title: "沉没与单点水花（splash_sink 族）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "实体沉没水花 splash_sink（SINKENTITY 选档协议）、海洋漩涡传送门水花 oceanwhirlportal_splash 与 pigshrine 泥花 mudpuddle_splash 三个单触发系统注册。"
aliases: ["splash_sink", "oceanwhirlportal_splash", "mudpuddle_splash", "沉没水花", "下沉水花", "漩涡传送门水花", "泥坑水花"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "ocean_util.lua", "prefabs/world.lua", "prefabs/oceanwhirlbigportal.lua", "prefabs/oceanwhirlportal.lua", "prefabs/bootleg.lua", "stategraphs/SGwilson.lua", "prefabs/tumbleweed.lua", "prefabs/monkeyhut.lua", "prefabs/waterplant_seed.lua", "components/flotsamgenerator.lua", "components/floater.lua", "components/oceanwhirlportalphysics.lua", "components/wagpunk_arena_manager.lua", "prefabs/wagpunk_floor_kit.lua", "prefabs/pigshrine.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 沉没与单点水花（splash_sink 族）

## 快速判断

- 职责与适用：三个各只有单一触发系统的落点表现注册——`splash_sink`（实体被水吞没／离场的下沉水花，SINKENTITY 选档协议主用）、`oceanwhirlportal_splash`（海洋漩涡传送门传送船只时入口／出口成对水花）、`mudpuddle_splash`（pigshrine 建筑建成／受击／被砸泥花）。机制互不相同，按小节分工；协议公共部分见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L1868-L1874)、[传送门花](../../../raw/current/scripts/fx.lua#L2030-L2039)、[泥花](../../../raw/current/scripts/fx.lua#L1816-L1822)
- 优先使用：实体从世界消失于水面需要水花→`splash_sink` 或整套 `GetSinkEntityFXPrefabs` 协议；漩涡传送门表现→`oceanwhirlportal_splash`；猪神殿类泥花借用→`mudpuddle_splash`。
- 重要副作用：无组件纯表现；`splash_sink` 与传送门花 fn 设海洋混合参数（贴水面着色）；泥花 fn=FinalOffset3。
- 修改边界：全部落点式一次性；SINKENTITY 出图回退是 `splash_ocean`、虚空档是 fallingswish 烟，都不是本页注册；泥花唯一消费者是 pigshrine。[协议](../../../raw/current/scripts/ocean_util.lua#L188-L208)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `splash_sink` | 通用表条目 → 加载器代理＋一帧后本地实体 | bank/build=`splash_water_drop`，anim=`idle_sink`，音效 `water/splash/small`；fn=`SetOceanBlendParams`。[条目](../../../raw/current/scripts/fx.lua#L1868-L1874) |
| `oceanwhirlportal_splash` | 同上协议 | 复用 `merm_king_splash` 资源（同名独立注册另有条目 [L2055-L2059](../../../raw/current/scripts/fx.lua#L2055-L2059)）；fn 蓝染 `SetMultColour(0.5,0.5,1,1)`＋FinalOffset1＋OceanBlend。[条目](../../../raw/current/scripts/fx.lua#L2030-L2039) |
| `mudpuddle_splash` | 同上协议 | bank/build=`mudsplash`，anim=`anim`，音效 `dontstarve/creatures/pengull/splash`，fn=FinalOffset3。[条目](../../../raw/current/scripts/fx.lua#L1816-L1822) |

简中 `languages/chinese_s.po` 定向检索无这 3 个注册名的显示名键；标题中文为已核验效果描述，非官方译名。

## 关联机制导航

### splash_sink：SinkEntity 选档协议与离场点

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 协议：SINKENTITY_PREFABS | `OCEAN={splash_sink}`、`VOID={fallingswish_*}`、`FALLBACK={splash_ocean}` 三档；`GetSinkEntityFXPrefabs` 按地图边界与 tile 选档，出图强制回退 | ocean_util 全局；world.lua 预载保证可生成 | [表](../../../raw/current/scripts/ocean_util.lua#L188-L192)、[选档](../../../raw/current/scripts/ocean_util.lua#L193-L208)、[预载](../../../raw/current/scripts/prefabs/world.lua#L292-L296) |
| 消费者：SinkEntity | 先掉落库存，再按选档逐个落花，随后不可替代物冲岸、其余 `Remove` | 服务端；实体真正离场前的最后表现 | [流程](../../../raw/current/scripts/ocean_util.lua#L210-L250)、fx 在 [L228-L233](../../../raw/current/scripts/ocean_util.lua#L228-L233) |
| 复用：oceanwhirlbigportal | 大传送门 `SplashWhirlportal` 对被传送者走同一 `GetSinkEntityFXPrefabs` | doer 传送时机 | [调用](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L209-L219) |
| 玩家浮具状态 | SGwilson `float_let_go`：手持浮具离手落水，普通分支在无替代浮具且原浮具仍有效时落一枚，wx78 分支在手持物带 `playerfloater`、`LetGo` 后仍有效时落一枚；`float_hop_pre` 踏水跳第 6 帧落一枚于 (x,0,z) | 服务端状态图帧事件；两分支都是条件生成点，不是到达状态即必出 | [float_let_go](../../../raw/current/scripts/stategraphs/SGwilson.lua#L26556-L26680)、[hop](../../../raw/current/scripts/stategraphs/SGwilson.lua#L26762-L26792) |
| 杂物离场点 | 风滚草离开有效地面 [tumbleweed](../../../raw/current/scripts/prefabs/tumbleweed.lua#L87-L93)；猴屋子猴落无效地面移除前 [monkeyhut](../../../raw/current/scripts/prefabs/monkeyhut.lua#L83-L97)；水草种弹命中不可通行点 [两处](../../../raw/current/scripts/prefabs/waterplant_seed.lua#L110-L118)、[onhit](../../../raw/current/scripts/prefabs/waterplant_seed.lua#L228-L235)；漂浮物到期 [flotsamgenerator](../../../raw/current/scripts/components/flotsamgenerator.lua#L205-L216) | 全部"实体即将 Remove"语义 | [[reference/components/flotsamgenerator]] |
| 漩涡打捞重定位 | winchtarget 被漩涡／wagpunk 场地移到海面点时落花：oceanwhirlportalphysics、wagpunk_floor_kit、wagpunk_arena_manager | 服务端；打捞失败则就地 Salvage | [[reference/components/winchtarget]]、[[reference/components/oceanwhirlportalphysics]]、[三处](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L193-L196)、[L56-L66](../../../raw/current/scripts/prefabs/wagpunk_floor_kit.lua#L56-L66)、[L1946-L1956](../../../raw/current/scripts/components/wagpunk_arena_manager.lua#L1946-L1956) |

### oceanwhirlportal_splash 与 mudpuddle_splash

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 消费者：oceanwhirlportal | `CheckForBoatsTick` 每 0.5 秒检查范围内船只，传送成立时在入口 (sx,sy,sz) 与出口实际选定点各落一枚 | 服务端周期任务；出口 tracker 有 blocker 直接跳过；出口位置受阻先搜索附近可传送偏移，仅搜索失败才放弃 | [周期](../../../raw/current/scripts/prefabs/oceanwhirlportal.lua#L10-L10)、[任务注册](../../../raw/current/scripts/prefabs/oceanwhirlportal.lua#L255-L255)、[出口门禁](../../../raw/current/scripts/prefabs/oceanwhirlportal.lua#L104-L133)、[双落](../../../raw/current/scripts/prefabs/oceanwhirlportal.lua#L138-L139) |
| 传送门来源 | bootleg 投掷生成入口／出口双向门户对（`CreateOceanWhirlportal`） | 服务端投掷落地 | [bootleg](../../../raw/current/scripts/prefabs/bootleg.lua#L11-L21) |
| 消费者：pigshrine | 建成 onbuilt、被砸 onhammered、被敲 workleft>0 的 onhit 三处于建筑位置落泥花 | 服务端建筑事件；当前唯一消费者 | [onbuilt](../../../raw/current/scripts/prefabs/pigshrine.lua#L142-L152)、[onhammered](../../../raw/current/scripts/prefabs/pigshrine.lua#L155-L166)、[onhit](../../../raw/current/scripts/prefabs/pigshrine.lua#L169-L180) |
| 资源 | `splash_water_drop`、`merm_king_splash`（共享）、`mudsplash` 三组 bank/build | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L1868-L1874) |

pigshrine 尚无 Wiki 页（Wiki 待建），源码入口 prefabs/pigshrine.lua；oceanwhirlportal 亦无本体页，机制侧见 [[reference/components/oceanwhirlportalphysics|漩涡物理组件]]。

## 典型接入

### 借用沉没水花或整套选档表现

- 前提与选择：只缺水花直接生成 `splash_sink`；要与原版沉没表现完全一致（含出图回退）可复用 `GetSinkEntityFXPrefabs`（require "ocean_util" 后的全局函数）。
- 生效与撤销：一次性落点动画，animover 自删；实体移除逻辑仍由调用方负责，本花不清理任何东西。
- 原版依据：SinkEntity [选档与落花](../../../raw/current/scripts/ocean_util.lua#L210-L250)。

```lua
-- 只借水花：
SpawnPrefab("splash_sink").Transform:SetPosition(px, py, pz)

-- 或复用选档协议（与 SinkEntity 同步）：
local fx_prefabs = GetSinkEntityFXPrefabs(entity, px, py, pz)
if fx_prefabs ~= nil then
    for _, prefab in ipairs(fx_prefabs) do
        SpawnPrefab(prefab).Transform:SetPosition(px, py, pz)
    end
end
```

- 调整与验证：协议返回的是 Prefab 名列表（OCEAN 档即 `{"splash_sink"}`），生成后仍需自行完成实体 Remove／冲岸等收尾。

## 常见陷阱与限制

- **误区：splash_sink 是通用落水花**：物品落水的冷热分支在 floater→[[reference/prefabs/splash|splash 族]]；`splash_sink` 的语义是"实体在该点消失"，生成点几乎全部紧邻 `Remove`／传送。[对比](../../../raw/current/scripts/components/floater.lua#L150-L174)
- **边界：选档协议的三档不全是本页注册**：出图边界回退 FALLBACK=`splash_ocean`（旧海洋水花），VOID 档是 fallingswish 虚空烟；只有海面 tile 命中才是 `splash_sink`。[表](../../../raw/current/scripts/ocean_util.lua#L188-L208)
- **边界：传送门花的蓝色来自 fn 染色**：资源与 `merm_king_splash` 注册完全同 bank/build/anim，颜色差异全在 `SetMultColour(0.5,0.5,1,1)`；不能从资源名推断外观。[条目](../../../raw/current/scripts/fx.lua#L2030-L2039)
- **边界：mudpuddle_splash 唯一消费者是 pigshrine**：检索范围为 rg 全 scripts 根字面名＋动态 SpawnPrefab 拼名清单（无 mudpuddle 前缀拼接命中）；借用时注意其泥水音效与 pengull 水花声共用。[检索声明](../../../raw/current/scripts/fx.lua#L1816-L1822)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。三条目配置、SinkEntity 协议链（表／选档／SinkEntity／world 预载／大传送门复用）、SGwilson 两个浮具状态（含两分支门禁）、tumbleweed／monkeyhut／waterplant_seed／flotsamgenerator、漩涡与 wagpunk 三处重定位、oceanwhirlportal 传送流程（含出口 tracker blocker 与受阻后的偏移搜索）、bootleg 来源、pigshrine 三触发逐点核验；简中 .po 定向核验 3 注册名无显示名键；动画时长未核验。
- 游戏运行：未运行；下沉视觉、传送门蓝花与泥花音效未实测。

返回 [[reference/prefabs/fx-table-world-index|通用表 FX·环境与物件]]、[[reference/prefabs/index|Prefab 索引]]。
