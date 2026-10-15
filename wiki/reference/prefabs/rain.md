---
title: "天气降水表现特效（rain／snow／pollen／lunarhail／caverain／caveacidrain）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "七个非联网天气 FX 的工厂、资源、驱动、跟随与池回收：地表雨雪花粉月雹、洞穴雨与酸雨，以及月雹落点。"
aliases: ["rain", "snow", "pollen", "lunarhail", "lunarhaildrop", "caverain", "caveacidrain", "降水粒子", "天气粒子", "月雹落点"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/rain.lua", "prefabs/caverain.lua", "prefabs/caveacidrain.lua", "prefabs/snow.lua", "prefabs/pollen.lua", "prefabs/lunarhail.lua", "prefabs/lunarhaildrop.lua", "prefabs/raindrop.lua", "emitters.lua", "components/weather.lua", "components/caveweather.lua", "components/raindome.lua", "componentutil.lua", "components/map.lua", "components/acidlevel.lua", "components/worldstate.lua", "prefabs/forest_network.lua", "prefabs/cave_network.lua", "gamelogic.lua", "prefablist.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 天气降水表现特效（rain／snow／pollen／lunarhail／caverain／caveacidrain）

## 快速判断

- 职责与适用：`rain`／`snow`／`pollen`／`lunarhail`／`caverain`／`caveacidrain` 是六个“降水表现管理层”本地实体，各持一台 VFX 发射器，按天气组件写入的每 tick 粒子数发射；`lunarhaildrop` 是 `lunarhail` 单独池化的落点动画。七者都只有 `FX` 标签，不结算伤害、湿身或积雪。[rain](../../../raw/current/scripts/prefabs/rain.lua#L87-L112)、[lunarhaildrop](../../../raw/current/scripts/prefabs/lunarhaildrop.lua#L40-L59)
- 优先使用：复现表现时看 `weather`／`caveweather` 如何写 `particles_per_tick` 与 `splashes_per_tick`；链路是“组件每帧写速率 → 实体累计发射/喷溅预算 → 池化落点”。[地表驱动](../../../raw/current/scripts/components/weather.lua#L1005-L1048)、[洞穴驱动](../../../raw/current/scripts/components/caveweather.lua#L651-L718)
- 重要副作用：全部 `persists = false`、`SetCanSleep(false)`、无 `AddNetwork()`，只在非专用服生成；激活玩家时 `SetParent(player.entity)` 跟随，失活解绑，组件移除或喷溅速率归零时清池。[生成开关](../../../raw/current/scripts/components/weather.lua#L211-L216)、[跟随](../../../raw/current/scripts/components/weather.lua#L542-L563)
- 修改边界：酸雨伤害由 `acidlevel` 结算，`caveacidrain` 只做表现；`lunarhaildrop` 也无伤害组件。七个注册名都无可用官方简中名（`STRINGS.NAMES.RAIN` 为 “Short Circuit／短路”，另有 `STRINGS.NAMES.ACIDRAIN`＝酸雨但不对应 `CAVEACIDRAIN`），标题与别名用效果描述＋注册名，非官方。[po](../../../raw/current/scripts/languages/chinese_s.po#L348469-L348472)

## 注册与构造

共同构造：`CreateEntity()` → `AddTag("FX")` → `SetCanSleep(false)`＋`persists = false` → `AddTransform()`；无 `AddNetwork()`／`SetPristine()`。六个管理器用 `AddVFXEffect()`＋`vfx_particle.ksh`；`lunarhaildrop` 只用 AnimState。[rain 构造](../../../raw/current/scripts/prefabs/rain.lua#L87-L112)、[lunarhaildrop 构造](../../../raw/current/scripts/prefabs/lunarhaildrop.lua#L40-L59)

| 注册名 | 工厂与资源 | 初始化差异与门禁 |
|---|---|---|
| `rain` | `Prefab("rain", fn, assets, prefabs)`；`fx/rain.tex`；白、寿命 2 秒、缩放 .1～10；池 `raindrop`。 | 地表雨；速率全由 `weather` 写；发射过雨棚＋`CanPointHaveRain`，喷溅另需可通行，雨棚边缘补落点。[工厂](../../../raw/current/scripts/prefabs/rain.lua#L273)、[发射](../../../raw/current/scripts/prefabs/rain.lua#L139-L163) |
| `snow` | `Prefab("snow", fn, assets)`；`snow.tex`／`wintersnow.tex`／`wintersnow_cane.tex`；寿命约 1.5～7.5 秒（`4.5 + 3×UnitRand()`）；无池、无喷溅。 | 地表雪；冬宴激活时换 `wintersnow_cane.tex`、UV 八帧、旋转、粒子倍率 2；`WINTER_TEXTURE_OLD` 只声明未使用。[工厂](../../../raw/current/scripts/prefabs/snow.lua#L174)、[冬宴分支](../../../raw/current/scripts/prefabs/snow.lua#L125-L157) |
| `pollen` | `Prefab("pollen", fn, assets)`；复用 `fx/snow.tex`；黄、寿命约 0～60 秒（`30 + 30×UnitRand()`）、1000 粒；无池。 | 地表花粉；只排除 `IsImpassableTile`（海洋不在此组，注释与实现不一致），夏季白天高温时降低初速并抬高运动，加速度按时间圆周变化。[工厂](../../../raw/current/scripts/prefabs/pollen.lua#L167)、[发射](../../../raw/current/scripts/prefabs/pollen.lua#L95-L147) |
| `lunarhail` | `Prefab("lunarhail", fn, assets, prefabs)`；`fx/lunarhail.tex`；寿命约 0.56～0.7 秒（`0.7 × (.9 + .1×UnitRand())`）、UV 八帧、加速度 `.2, -6, 1`；双池。 | 地表月雹；粒子只排雨棚，不查地形雨门禁；海面落 `raindrop`，陆地落 `lunarhaildrop`，按 `_next_long_fx` 计数周期性给一次 2～3 秒地面停留。[工厂](../../../raw/current/scripts/prefabs/lunarhail.lua#L263) |
| `caverain` | `Prefab("caverain", fn, assets, prefabs)`；同样 `fx/rain.tex`；寿命 1 秒、缩放 .5～1；池 `raindrop`。 | 洞穴普通雨；初速 `-1 + UnitRand() * -2`，门禁同 `rain`。[工厂](../../../raw/current/scripts/prefabs/caverain.lua#L274)、[发射](../../../raw/current/scripts/prefabs/caverain.lua#L140-L164) |
| `caveacidrain` | `Prefab("caveacidrain", fn, assets, prefabs)`；同样 `fx/rain.tex`；暗黄 `102,102,0,200`、寿命 1 秒、缩放 .5～1；池 `acidraindrop`。 | 洞穴酸雨；门禁 `CanPointHaveAcidRain`（先要求可下普通雨，再排不可通行、vault 与 `nocavein`／`lunacyarea` 节点）。[工厂](../../../raw/current/scripts/prefabs/caveacidrain.lua#L274)、[门禁](../../../raw/current/scripts/prefabs/caveacidrain.lua#L140-L164)、[地图](../../../raw/current/scripts/components/map.lua#L925-L949) |
| `lunarhaildrop` | `Prefab("lunarhaildrop", fn, assets)`；`anim/lunarhaildrop.zip`（`ALT_RENDERPATH`）；bank/build `lunarhaildrop`。 | 落点播 `anim`；无 `delay` 接 `anim_pst`，有则转地面 `ground1/2`（朝向取 `TheCamera.headingtarget`、背景层、sort 3），延迟后播 `groundX_pst`，结束回池或移除。[工厂](../../../raw/current/scripts/prefabs/lunarhaildrop.lua#L62)、[动画](../../../raw/current/scripts/prefabs/lunarhaildrop.lua#L10-L31) |

三处寿命都以 `UnitRand()` 采样；它是 `math.random()*2-1`，取值约 −1～1，因此 `MIN_LIFETIME`／`MAX_LIFETIME` 的常量名不等于实际下界。[UnitRand](../../../raw/current/scripts/emitters.lua#L101-L103)

### 生成点与驱动

原版只在两个天气组件的构造期生成：`weather` 生成四个地表 FX，`caveweather` 生成两个洞穴 FX，均以 `_hasfx = not TheNet:IsDedicated()` 控制；`lunarhaildrop` 只由 `lunarhail` 的喷溅池生成。除这两个组件外，没有世界生成任务、房间或其它管理器另行创建这些实体。[地表构造](../../../raw/current/scripts/components/weather.lua#L211-L216)、[洞穴构造](../../../raw/current/scripts/components/caveweather.lua#L153-L158)、[月雹落点](../../../raw/current/scripts/prefabs/lunarhail.lua#L76-L95)

| 实体 | 构造点 | 原版每帧驱动 | 证据 |
|---|---|---|---|
| `rain` | `weather` | `5 * preciprate` 粒子、`2 * preciprate` 喷溅 | [L1005-L1009](../../../raw/current/scripts/components/weather.lua#L1005-L1009) |
| `snow` | `weather` | `20 * preciprate` 粒子；无喷溅 | [L1043-L1047](../../../raw/current/scripts/components/weather.lua#L1043-L1047) |
| `pollen` | `weather` | 夏季按季节进度 ramp × `0.5`；非夏季或沙暴归零 | [L1068-L1081](../../../raw/current/scripts/components/weather.lua#L1068-L1081) |
| `lunarhail` | `weather` | `4 * lunarhailrate` 粒子、`2 * lunarhailrate` 喷溅 | [L1016-L1018](../../../raw/current/scripts/components/weather.lua#L1016-L1018) |
| `caverain` | `caveweather` | `5 * preciprate` 粒子、`2 * preciprate` 喷溅 | [L705-L708](../../../raw/current/scripts/components/caveweather.lua#L705-L708) |
| `caveacidrain` | `caveweather` | `5 * preciprate` 粒子、`2 * preciprate` 喷溅 | [L714-L715](../../../raw/current/scripts/components/caveweather.lua#L714-L715) |
| `lunarhaildrop` | `lunarhail` 喷溅 | 由 `lunarhail` 的 `splashes_per_tick` 预算逐滴生成／复用 | [L223-L243](../../../raw/current/scripts/prefabs/lunarhail.lua#L223-L243) |

世界加载链路：`forest_network` 添加 `weather`，`cave_network` 添加 `caveweather`；主端载入时 `gamelogic` 以 `savedata.map.prefab.."_network"` 动态生成世界网络实体，组件构造期再生成 FX。[地表装配](../../../raw/current/scripts/prefabs/forest_network.lua#L16-L22)、[洞穴装配](../../../raw/current/scripts/prefabs/cave_network.lua#L8-L12)、[动态世界网络名](../../../raw/current/scripts/gamelogic.lua#L584)

### 落点名与检索范围

- `rain`／`caverain`／`caveacidrain` 的 `SpawnPrefab(prefab)` 不做动态拼名：本地 `GetPooledFx(prefab, pool)` 只被 `SpawnRaindropAtXZ` 调用，实参是字面量 `"raindrop"` 或 `"acidraindrop"`；无字符串连接、无表查名。[rain 池与落点](../../../raw/current/scripts/prefabs/rain.lua#L53-L78)、[caverain](../../../raw/current/scripts/prefabs/caverain.lua#L54-L79)、[caveacidrain](../../../raw/current/scripts/prefabs/caveacidrain.lua#L54-L79)
- `lunarhail` 同样只在两个字面量间选择：`TheWorld.Map:IsOceanAtPoint` 为真取 `"raindrop"`，否则取 `"lunarhaildrop"`；`lunarhaildrop` 自身以 `"lunarhaildrop"` 注册，并在父级 `prefabs` 依赖表中声明。[海陆选择](../../../raw/current/scripts/prefabs/lunarhail.lua#L76-L88)、[依赖表](../../../raw/current/scripts/prefabs/lunarhail.lua#L14-L18)
- 检索范围：在 `raw/current/scripts` 全树对全部 `.lua` 运行 `SpawnPrefab("<七个注册名>")` 与字面注册名检索；`SpawnPrefab` 命中仅有 `weather.lua` 四行与 `caveweather.lua` 两行，`map/`、`worldgen/` 零命中，其余字面量还出现在 `prefablist.lua` 清单、`worldstate.lua` 类型名和 `lunarhail.lua` 子实体名等处。全链路唯一的动态名是世界网络 prefab 的 `savedata.map.prefab.."_network"`，与落点无关。[清单](../../../raw/current/scripts/prefablist.lua#L260-L264)、[类型名](../../../raw/current/scripts/components/worldstate.lua#L152-L157)

### 跟随与清理

- 激活玩家时四个地表 FX 或两个洞穴 FX（按世界）分别 `SetParent(player.entity)` 并调用一次 `OnPostInit()` 预热；玩家失活时 `SetParent(nil)`。[地表](../../../raw/current/scripts/components/weather.lua#L542-L563)、[洞穴](../../../raw/current/scripts/components/caveweather.lua#L410-L429)
- 子实体动画结束且 `pool.valid` 时 `RemoveFromScene()` 回池，否则移除；父级 `OnRemoveEntity` 清池并置 `valid = false`；喷溅速率为 0 时每帧 `ClearPoolEnts`。[rain 回收](../../../raw/current/scripts/prefabs/rain.lua#L53-L83)、[停喷清理](../../../raw/current/scripts/prefabs/rain.lua#L247-L253)、[月雹双池](../../../raw/current/scripts/prefabs/lunarhail.lua#L236-L243)
- 世界移除时 `weather`／`caveweather` 的 `OnRemoveEntity` 对每个仍有效的 FX 调 `Remove()`。[地表清理](../../../raw/current/scripts/components/weather.lua#L853-L867)、[洞穴清理](../../../raw/current/scripts/components/caveweather.lua#L578-L585)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：weather` | 构造并持有四个地表 FX；写速率；挂到激活玩家；雨声与地面 overlay 同步。 | 非专用服；雨/雪/月雹/花粉四类；雨雪按温度切换。 | [[reference/components/weather]]；[构造](../../../raw/current/scripts/components/weather.lua#L211-L216) |
| `组件：caveweather` | 构造并持有 `caverain`／`caveacidrain`；按类型与降水量写速率；区域变化重查门禁。 | 非专用服；普通雨或酸雨；酸雨需 `TUNING.ACIDRAIN_ENABLED` 与影裂传送门激活。 | [[reference/components/caveweather]]；[酸雨起点](../../../raw/current/scripts/components/caveweather.lua#L330-L340) |
| `全局 helper：GetRainDomesAtXZ／IsUnderRainDomeAtXZ` | 雨、洞穴雨、酸雨、月雹与喷溅都用来排除雨棚；前三种还在雨棚边缘补落点。 | 由 `componentutil` 预加载；依赖 `raindome` 激活半径。 | [[reference/components/raindome]]；[定义](../../../raw/current/scripts/components/raindome.lua#L29-L59)、[加载](../../../raw/current/scripts/componentutil.lua#L1) |
| `数据：Map:CanPointHaveRain／CanPointHaveAcidRain` | 普通雨/洞穴雨用前者，酸雨用后者；后者先复用前者再排不可通行、vault、`nocavein`／`lunacyarea`。 | 地表与洞穴共用；月雹不查。 | [普通雨](../../../raw/current/scripts/components/map.lua#L955-L965)、[酸雨](../../../raw/current/scripts/components/map.lua#L925-L949) |
| `子实体：raindrop／acidraindrop／lunarhaildrop` | 分别由 rain/caverain/lunarhail、caveacidrain、lunarhail 池化。 | 前两者动画落点，第三者带地面停留；都非联网、不存档。 | [[reference/prefabs/acidraindrop|酸雨落点烟]]、[[reference/prefabs/raindrop|落点水花]]；[源码](../../../raw/current/scripts/prefabs/raindrop.lua#L19-L41) |
| `组件：acidlevel` | 酸雨玩法结算：监听 `isacidraining` 起周期任务，对装备、背包食物与生命结算伤害。 | `rainimmunity`／`ignoreacidrainticks` 可跳过。 | [[reference/components/acidlevel]]；[监听](../../../raw/current/scripts/components/acidlevel.lua#L114-L119)、[tick](../../../raw/current/scripts/components/acidlevel.lua#L147-L205) |
| `组件：worldstate` | 把 `precipitationchanged` 聚合为 `israining`／`issnowing`／`islunarhailing`／`isacidraining`。 | 类型字符串来自两个天气组件的 netvar。 | [[reference/components/worldstate]]；[聚合](../../../raw/current/scripts/components/worldstate.lua#L152-L157) |

## 常见陷阱与限制

- 直接 `SpawnPrefab("rain")` 不会自己下雨：六个管理器的 `desired_particles_per_second` 初值都是 0，必须由调用方写 `particles_per_tick`／`splashes_per_tick`；`PostInit()` 只按寿命预热，不产生持续速率。[rain 初值](../../../raw/current/scripts/prefabs/rain.lua#L118-L128)、[snow 初值](../../../raw/current/scripts/prefabs/snow.lua#L83-L89)
- 专用服没有这些 FX：`_hasfx = not TheNet:IsDedicated()`，两个组件在专用服把 FX 置 `nil`；不要假定服务端一定有可操作的降水实体。[地表](../../../raw/current/scripts/components/weather.lua#L211-L216)、[洞穴](../../../raw/current/scripts/components/caveweather.lua#L153-L158)
- 表现不等于玩法：`caveacidrain` 只生成 `acidraindrop` 烟，伤害、装备腐蚀与背包食物腐烂由 `acidlevel` 结算；`lunarhaildrop` 只有 Transform／AnimState。[酸雨 tick](../../../raw/current/scripts/components/acidlevel.lua#L147-L205)、[落点构造](../../../raw/current/scripts/prefabs/lunarhaildrop.lua#L40-L59)
- 月雹落点门禁比雨窄：只排雨棚；海面换 `raindrop`，陆地不查 `CanPointHaveRain` 或可通行，落点集合可能不同。[月雹喷溅](../../../raw/current/scripts/prefabs/lunarhail.lua#L223-L231)
- `lunarhaildrop.RestartFx` 只重播动画并随机翻转，不清 `inst.delay`；带长延迟的实例回池后再取出会沿用旧延迟（源码推导，未运行验证）。[重播](../../../raw/current/scripts/prefabs/lunarhaildrop.lua#L33-L38)

## 验证记录

- 源码审阅：2026-11-04，steam-buildid:26335394。逐文件全读七个 prefab；核验 weather/caveweather 构造、驱动、跟随与清理，forest_network/cave_network 装配与 gamelogic 世界网络生成，raindome/componentutil 全局 helper，map 雨门禁，acidlevel 酸雨结算，worldstate 聚合，prefablist 清单，以及 chinese_s.po 名称键。`SpawnPrefab` 反查覆盖 `raw/current/scripts` 全树，`map/`、`worldgen/` 零命中。
- 游戏运行：未运行；动画/贴图外观、音效和 `lunarhaildrop` 延迟复用表现未做运行验证。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 均通过（0 错 0 警）；无 Lua 示例，未跑 `parser_probe.exe`。

## 相关链接

- [[reference/components/weather]]、[[reference/components/caveweather]]、[[reference/components/acidlevel]]、[[reference/components/raindome]]
- 返回 [[reference/prefabs/fx-world-index|天气、地形与生态痕迹]]
