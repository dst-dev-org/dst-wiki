---
title: "鹅形态水花与羽毛（weregoose_splash）"
created: "2026-10-31"
updated: "2026-11-08"
type: prefab
summary: "十个通用表注册承载鹅形态水上移动表现：三档水花（run／step／walk）与羽毛拖尾由 SGwilson 三个 helper 按水陆分流，非扑翼状态另有周期涟漪，起飞羽毛沿轨迹落点；海猴划船复用本名水花。全部父挂取位即解挂，不持续跟随。"
aliases: ["weregoose_splash", "weregoose_splash_med1", "weregoose_splash_med2", "weregoose_splash_less1", "weregoose_splash_less2", "weregoose_ripple1", "weregoose_ripple2", "weregoose_feathers1", "weregoose_feathers2", "weregoose_feathers3", "鹅形态水花", "水上羽毛", "涟漪特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26474873"
sources: ["fx.lua", "stategraphs/SGwilson.lua", "prefabs/woodie.lua", "stategraphs/SGpowdermonkey.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 鹅形态水花与羽毛（weregoose_splash）

## 快速判断

- 职责与适用：十个通用表注册承载鹅形态（伍迪月鹅）水上／陆地移动表现——三档水花、三段羽毛拖尾与两段涟漪；另有海猴划船复用。工厂协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目组](../../../raw/current/scripts/fx.lua#L732-L863)
- 优先使用：查"鹅形态水花／水上羽毛／涟漪特效"到本页；变身烟见 [[reference/prefabs/werebeaver_transform_fx|变身切换烟]]。
- 重要副作用：带鸟形水声的水花共 5 注册（`weregoose_splash` 与 med／less 各两变体），羽毛与涟漪无声；涟漪渲染在背景层。[水花条目](../../../raw/current/scripts/fx.lua#L768-L779)
- 修改边界：十名条目 fn 均在挂有父级时**立即解挂并定位于当帧世界坐标**——父挂只为取位，表现不随宿主移动。[解挂 fn](../../../raw/current/scripts/fx.lua#L773-L779)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `weregoose_splash` | 通用表条目带 fn：海面着色＋解挂取位 | bank=`splash_weregoose_fx`、build=`splash_water_drop`，anim=`idle`，sound=鸟形水声。[条目](../../../raw/current/scripts/fx.lua#L768-L779) |
| `weregoose_splash_med1`／`_med2` | 同上协议 | anim=`stationary`／`stationary2`，带水声。[条目](../../../raw/current/scripts/fx.lua#L782-L807) |
| `weregoose_splash_less1`／`_less2` | 同上协议 | anim=`stationary_small`／`stationary_small2`，带水声。[条目](../../../raw/current/scripts/fx.lua#L810-L835) |
| `weregoose_ripple1`／`_ripple2` | 同上协议，另设 `LAYER_WORLD_BACKGROUND` 背景层 | anim=`no_splash`／`no_splash2`，**无声**。[条目](../../../raw/current/scripts/fx.lua#L838-L863) |
| `weregoose_feathers1`／`_feathers2`／`_feathers3` | 条目带 fn：仅解挂取位（无着色） | bank／build=`weregoose_fx`，anim=`trail1～3`，**无声**。[条目](../../../raw/current/scripts/fx.lua#L732-L765) |

十名在简中 .po 均无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名（伍迪官方名见 [po:351765-351766](../../../raw/current/scripts/languages/chinese_s.po#L351765-L351766)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 代理＋一帧本地表现，fn(inst, proxy) 同步调用 | 消费者父挂代理 → 本地表现挂代理父级 → fn 解挂取位 | [[reference/mechanisms/generic-fx-factory]] |
| 移动水花 helper（SGwilson） | `DoGooseRunFX`：水上落 `weregoose_splash`、陆地落 `weregoose_feathers1～3`（随机），父挂玩家；`DoGooseStepFX`／`DoGooseWalkFX`：水上分别落 `splash_med1～2`／`splash_less1～2` | 服务端；先 `drownable:IsOverWater()` 分流水陆 | [三 helper](../../../raw/current/scripts/stategraphs/SGwilson.lua#L151-L169) |
| helper 接线状态 | RunFX：`run` 两处帧（1／9 帧；另一处调用在长注释块内未启用）、`death` 15 帧（goose 分支）、`reviver_rebirth`；StepFX：`idle` onupdate（idle_loop 第 5／14 帧）、`run_stop`、`acting_run_stop`；WalkFX：`run` 两处迷糊帧（4／17 帧，goosegroggy 分支）、`run_stop`、`acting_run_stop` | 服务端；全部父挂玩家实体 | [death](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3828-L3833)、[idle](../../../raw/current/scripts/stategraphs/SGwilson.lua#L4354-L4362)、[run 1 帧](../../../raw/current/scripts/stategraphs/SGwilson.lua#L11526-L11532)、[run 9 帧与迷糊 WalkFX](../../../raw/current/scripts/stategraphs/SGwilson.lua#L11672-L11694)、[run_stop](../../../raw/current/scripts/stategraphs/SGwilson.lua#L11804-L11806)、[acting_run_stop](../../../raw/current/scripts/stategraphs/SGwilson.lua#L23389-L23391)、[reviver_rebirth](../../../raw/current/scripts/stategraphs/SGwilson.lua#L15394-L15394) |
| 停留涟漪（woodie） | `OnNewGooseState`：新状态不在扑翼清单或迷糊时，起 0.7 秒周期任务 `DoRipple` 落 `ripple1～2`（随机、父挂、水上判定在 helper 内）；进入扑翼状态则取消任务 | 服务端；扑翼清单= idle／run_start／run／weregoose_takeoff | [任务与清单](../../../raw/current/scripts/prefabs/woodie.lua#L975-L1002) |
| 变回时水上水花 | `transform_goose_person` animover：水上变回先落 `weregoose_splash`（父挂）再落变身烟 | 服务端；变身烟见 [[reference/prefabs/werebeaver_transform_fx|变身切换烟]] | [水上分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L3367-L3370) |
| 起飞羽毛 | `weregoose_takeoff` onupdate：每 7 帧在起飞点（y 随时间抬升）落点式生成 `weregoose_feathers1～3`（随机） | 服务端；不经父挂，直接 SetPosition | [onupdate](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22474-L22482) |
| 跨生物复用（海猴） | SGpowdermonkey 划船状态 7 帧在身后 1 距离落点 `weregoose_splash` | 服务端；猴子无父挂取位，直接定位 | [spawnsplash](../../../raw/current/scripts/stategraphs/SGpowdermonkey.lua#L3-L11)、[划船帧](../../../raw/current/scripts/stategraphs/SGpowdermonkey.lua#L348-L348) |
| 资源 | 羽毛一套（weregoose_fx）、水花一套（splash_weregoose_fx／splash_water_drop，五组 anim 分档） | 动画时长与循环性未核验 | [条目组](../../../raw/current/scripts/fx.lua#L732-L863) |

## 典型接入

### 给水生生物移动接同款水花

- 前提与选择：宿主需有 `drownable` 组件供 `IsOverWater()` 判定；父挂取位写法照抄原版 helper——挂到宿主实体即可，条目 fn 会立即解挂定位。
- 生效与撤销：一次性动画，animover 自删；不随宿主移动。
- 原版依据：[DoGooseStepFX](../../../raw/current/scripts/stategraphs/SGwilson.lua#L151-L155)。

```lua
-- 服务端：在自定义生物的移动帧事件里调用
local function DoStepSplash(inst)
    if inst.components.drownable ~= nil and inst.components.drownable:IsOverWater() then
        SpawnPrefab("weregoose_splash_med"..tostring(math.random(2))).entity:SetParent(inst.entity)
    end
end
```

- 调整与验证：换 `weregoose_splash` 出大水花、`weregoose_ripple1～2` 出无声背景涟漪；陆地目标不要接水花组（无水陆分流时表现仍会播放）。

## 常见陷阱与限制

- **边界：父挂≠跟随**：十条目 fn 均在挂有父级时立即解挂并定位于当帧世界坐标，表现不随宿主移动；持续跟随需求需自行换非通用表工厂。[解挂 fn](../../../raw/current/scripts/fx.lua#L773-L779)
- **边界：无声注册**：feathers 三段与 ripple 两段无 sound 字段；带声水花为 5 注册。按"都是水花"统一补声会与原版表现不一致。[条目组](../../../raw/current/scripts/fx.lua#L732-L863)
- **边界：涟漪在背景层**：ripple 条目另设 `LAYER_WORLD_BACKGROUND`，叠放次序与水花不同；前景遮挡判断勿混用。[条目](../../../raw/current/scripts/fx.lua#L838-L849)
- **边界：命名带 weregoose 非鹅专属**：海猴划船复用 `weregoose_splash` 本名；跨生物复用不改注册。[海猴](../../../raw/current/scripts/stategraphs/SGpowdermonkey.lua#L348-L348)
- **边界：涟漪与移动水花无统一互斥门禁**：周期涟漪只在非扑翼清单状态（如 `run_stop`、攻击、工作）或迷糊时起；但 `run_stop` 状态自身同时接 StepFX／WalkFX 帧事件、迷糊时扑翼状态也照起涟漪任务——两类表现可以并存，不要假设互斥。扑翼清单含 idle／run_start／run／weregoose_takeoff。[任务与清单](../../../raw/current/scripts/prefabs/woodie.lua#L975-L1002)
- **边界：起飞羽毛是落点式**：takeoff 路径不经父挂，直接在轨迹点 SetPosition；与其他父挂路径的取位语义不同。[onupdate](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22474-L22482)

## 验证记录

- 源码审阅：2026-10-31，Steam buildid 26230496。十注册配置（bank／build／anim／sound／fn 与解挂逻辑）、SGwilson 三 helper 与六类接线状态、woodie 周期涟漪与扑翼清单、变回水花、起飞羽毛、海猴复用、po 键缺失核验。未覆盖：动画资源内容、扑翼声与涟漪的音频细节、wereplayer 技能树触发链、晚加入时序。
- 游戏运行：未运行；水花与羽毛视觉未实测。

返回 [[reference/prefabs/fx-transition-index|变身、召唤、传送与退场]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-04）。
