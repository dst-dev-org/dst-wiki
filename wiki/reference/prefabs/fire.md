---
title: "火焰、火光与烟雾特效族（fire 等）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "原版火焰、冷火、角色火、眼焰与烟雾 FX 的工厂链、firefx 等级表、heater 热度与宿主生成清理；通用燃烧的 fire 有直接与 helper 两路生成点。"
aliases: ["fire", "campfirefire", "portable_campfirefire", "coldfirefire", "nightlight_flame", "pigtorch_flame", "nightstickfire", "character_fire", "character_fire_flicker", "firefx_light", "floatinglanternfire", "thurible_smoke", "smoke_plant", "firehit", "eyeflame", "火焰特效", "燃烧特效", "火焰与烟雾"]
tags: [prefab, fx]
size_class: complex
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fire.lua", "prefabs/campfirefire.lua", "prefabs/coldfirefire.lua", "prefabs/nightlight_flame.lua", "prefabs/pigtorch_flame.lua", "prefabs/nightstickfire.lua", "prefabs/character_fire.lua", "prefabs/firefx_light.lua", "prefabs/floatinglanternfire.lua", "prefabs/thurible_smoke.lua", "prefabs/smoke_plant.lua", "prefabs/firehits.lua", "prefabs/eyeflame.lua", "components/firefx.lua", "components/burnable.lua", "components/heater.lua", "standardcomponents.lua", "entityscript.lua", "prefabs/carrat.lua", "prefabs/campfire.lua", "prefabs/firepit.lua", "prefabs/coldfire.lua", "prefabs/coldfirepit.lua", "prefabs/portablefirepit.lua", "prefabs/deerclops.lua", "prefabs/floatinglantern.lua", "prefabs/hound.lua", "prefabs/nightlight.lua", "prefabs/nightstick.lua", "prefabs/oceanvine.lua", "prefabs/oceanvine_cocoon.lua", "prefabs/pigtorch.lua", "prefabs/thurible.lua", "prefabs/rock_light.lua", "prefabs/tumbleweed.lua", "prefabs/abigail_flower.lua", "prefabs/cotl_tabernacle.lua", "prefabs/stageusher.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 火焰、火光与烟雾特效族（fire 等）

## 快速判断

- 职责与适用：13 个文件、15 个注册；9 个注册由 [`firefx`](../../../raw/current/scripts/components/firefx.lua#L88-L135) 等级表驱动动画／灯／声，6 个注册另挂 `heater` 供热，其余为纯灯光或独立 VFX／动画。
- 优先使用：不要为“会燃烧的实体”直接生成这些 FX；走 [`burnable:AddBurnFX`](../../../raw/current/scripts/components/burnable.lua#L135-L137)＋`SetFXLevel`，通用档由 [`Make*Burnable`](../../../raw/current/scripts/standardcomponents.lua#L158-L198) 选 `fire`，角色档选 `character_fire`。只做火焰实体时按 [[reference/components/firefx|firefx]] 接入。
- 重要副作用：`heater` 是真温度（`fire` 30–240，`coldfirefire` 为负且吸热）；`fire` 熄灭时按 5 格内火等级总和抑制 fireOut 声（[test](../../../raw/current/scripts/prefabs/fire.lua#L56-L67)）。
- 修改边界：`firefx`／`heater` 只在主模拟；`firehit` 的 `master_postinit` 与 `campfirefire` 的 quagmire 分支依赖缺失的 `event_data`，不推断行为；bank／build／sound 仅核验声明。

## 注册与构造

| 注册名 | 工厂与初始化分界 | FX 差异（levels） | 生成点与清理 |
|---|---|---|---|
| `fire` | `fn`：bank/build `fire`、`FINALOFFSET_MAX`、`FX`＋`HASHEATER` → SetPristine；主模拟 `firefx` 6 档＋`heater.heatfn`（[L28-L72](../../../raw/current/scripts/prefabs/fire.lua#L28-L72)） | 6 档暗黄→亮黄→橙；L4／5 同 `level4`，L5 起 forestfire 声；无 `pst`（[levels](../../../raw/current/scripts/prefabs/fire.lua#L18-L26)） | 无直接 `SpawnPrefab("fire")`；`AddBurnFX("fire")` 有 12 处调用（如 [carrat](../../../raw/current/scripts/prefabs/carrat.lua#L789)、leafpile、moontree、palmconetree）与 `burnfx.generic` helper 两路，均经 [`SpawnFX`](../../../raw/current/scripts/components/burnable.lua#L496-L539) 生成，随 `KillFX` 删。 |
| `campfirefire`／`portable_campfirefire` | 共用 [`MakeFire`](../../../raw/current/scripts/prefabs/campfirefire.lua#L37-L82)：`SetLevel(1)`、`usedayparamforsound`；quagmire 分支缺事件数据（[L72-L74](../../../raw/current/scripts/prefabs/campfirefire.lua#L72-L74)） | `campfirefire` 4 档 r2–5；`portable_campfirefire` 3 档 r2.5／3／3.5，热度 70／77.5／85（[两表](../../../raw/current/scripts/prefabs/campfirefire.lua#L15-L33)） | 篝火类 `AddBurnFX`：[campfire](../../../raw/current/scripts/prefabs/campfire.lua#L165)、[firepit](../../../raw/current/scripts/prefabs/firepit.lua#L199)（(0,25,0) 偏移）、[stagehand](../../../raw/current/scripts/prefabs/stageusher.lua#L223)、[abigail_flower](../../../raw/current/scripts/prefabs/abigail_flower.lua#L373-L374)、[cotl_tabernacle](../../../raw/current/scripts/prefabs/cotl_tabernacle.lua#L277-L278)、[portablefirepit](../../../raw/current/scripts/prefabs/portablefirepit.lua#L177)。 |
| `coldfirefire` | 同构 `fn`，bank/build `coldfire_fire`；`heater`＋`SetThermics(false,true)`＋`firefx`（[L51-L58](../../../raw/current/scripts/prefabs/coldfirefire.lua#L51-L58)） | 4 档 `_redux`，冷色 `00B7FF`，热度 −10…−40（[levels](../../../raw/current/scripts/prefabs/coldfirefire.lua#L18-L24)） | [coldfire](../../../raw/current/scripts/prefabs/coldfire.lua#L135)、[coldfirepit](../../../raw/current/scripts/prefabs/coldfirepit.lua#L141)。 |
| `nightlight_flame` | 同构 `fn`，build `campfire_fire`，`SetMultColour(0,0,0,.6)` 压暗后加 `firefx`；无 heater | 4 档，nightlight 声，暗粉 `FDB3B3`（[levels](../../../raw/current/scripts/prefabs/nightlight_flame.lua#L13-L19)） | [nightlight](../../../raw/current/scripts/prefabs/nightlight.lua#L120) 绑 `fire_marker`。 |
| `pigtorch_flame` | 同构 `fn`，build `campfire_fire`＋`firefx`；无 heater | 3 档，半径 3／4／5，falloff .44（[levels](../../../raw/current/scripts/prefabs/pigtorch_flame.lua#L13-L18)） | [pigtorch](../../../raw/current/scripts/prefabs/pigtorch.lua#L125) offset(−5,40,0) 绑 `fire_marker`。 |
| `nightstickfire` | Transform＋Light＋Network，无 AnimState／firefx；固定灯 r4／i.75／f.6／`FCFBED`，`persists=false`（[L10-L22](../../../raw/current/scripts/prefabs/nightstickfire.lua#L10-L22)） | 无等级；纯灯光 | [nightstick](../../../raw/current/scripts/prefabs/nightstick.lua#L39-L44) 装备时生成挂持有者，卸下／移除时删（[L53-L55](../../../raw/current/scripts/prefabs/nightstick.lua#L53-L55)）。 |
| `character_fire` | `commonfn` 出公共外观 → [`firefn`](../../../raw/current/scripts/prefabs/character_fire.lua#L114-L128) 主模拟加 `firefx`＋`heater.heatfn` | 3 档带 `pre`／`pst`／`pst_fast` 与 `_controlled_burn`；半径 2／3／4，热度 50／65／80（[levels](../../../raw/current/scripts/prefabs/character_fire.lua#L18-L71)） | 默认角色档（[helper](../../../raw/current/scripts/standardcomponents.lua#L245-L282)）；代表宿主另有 [cocoon](../../../raw/current/scripts/prefabs/oceanvine_cocoon.lua#L77-L83) 直生 `SetLevel(5)`→3、[oceanvine_cocoon 换名](../../../raw/current/scripts/prefabs/oceanvine_cocoon.lua#L335-L339)、[oceanvine](../../../raw/current/scripts/prefabs/oceanvine.lua#L316-L318) 改 `fxdata[1].prefab`、[rock_light](../../../raw/current/scripts/prefabs/rock_light.lua#L190)、[tumbleweed](../../../raw/current/scripts/prefabs/tumbleweed.lua#L498) 直接 `AddBurnFX`（未穷举）。 |
| `character_fire_flicker` | 同 `commonfn` → [`flickerfn`](../../../raw/current/scripts/prefabs/character_fire.lua#L130-L144)；heater 固定 `heat=65`，无 heatfn | 1／2 档空表，仅 3 档 `loop_fire_flicker`（[levels](../../../raw/current/scripts/prefabs/character_fire.lua#L73-L88)） | 无独立宿主 helper；仅 [deerclops](../../../raw/current/scripts/prefabs/deerclops.lua#L691) 替换 `fxdata[1].prefab`（[L430](../../../raw/current/scripts/prefabs/deerclops.lua#L430) 给 level 3）。 |
| `firefx_light` | 仅 Transform＋Light＋Network，无 bank／build／anim；`persists=false`（[L1-L19](../../../raw/current/scripts/prefabs/firefx_light.lua#L1-L19)） | 无等级；参数由 `firefx:SetLevel` 写入 | [`firefx` 构造时生成并挂父](../../../raw/current/scripts/components/firefx.lua#L18-L19)；`AttachLightTo` 可转挂宿主；各 prefab 列表只是预载声明。 |
| `floatinglanternfire` | `MakeFire` 包装，4 档；非专服 `updatelooper` 按父级 `camerafade` 覆写 alpha（[L26-L32](../../../raw/current/scripts/prefabs/floatinglanternfire.lua#L26-L32)）；灯位偏移 (0,−6,0) 防裁剪 | 半径 1／1.34／1.67／2，falloff .4，yoth_2026 声（[levels](../../../raw/current/scripts/prefabs/floatinglanternfire.lua#L16-L22)） | [floatinglantern](../../../raw/current/scripts/prefabs/floatinglantern.lua#L441) 绑 `flames_wide`、followaschild＋followlayered。 |
| `thurible_smoke` | Transform＋SoundEmitter＋Network；两套 VFX 发射器（烟＋手影），EnvelopeManager 定色／缩放；`staysthroughvirtualrooms`（[fn](../../../raw/current/scripts/prefabs/thurible_smoke.lua#L191-L215)） | 非 firefx；烟 1.1s／手影 1.7s，专服跳过粒子（[InitParticles](../../../raw/current/scripts/prefabs/thurible_smoke.lua#L120-L187)） | [thurible](../../../raw/current/scripts/prefabs/thurible.lua#L44-L84) 生成并跟随，`turnoff`／`OnRemove` 删。 |
| `smoke_plant` | AnimState 播 `smoke_loop`＋琥珀色灯＋`PlaySound(...,"smolder")` 命名声部（循环属性在声音资源，快照不可证），`persists=false`（[L16-L39](../../../raw/current/scripts/prefabs/smoke_plant.lua#L16-L39)） | 无等级；闷烧指示 | [`StartWildfire` 生成](../../../raw/current/scripts/components/burnable.lua#L251-L258)，`StopSmoldering` 停声并删（[L431-L440](../../../raw/current/scripts/components/burnable.lua#L431-L440)）。 |
| `firehit` | Transform＋Network＋`variation` netvar；命中动画由本地非联网子实体播（延迟一帧、variation/2 选 anim、奇偶翻转），`animover` 自删（[PlayFXAnim](../../../raw/current/scripts/prefabs/firehits.lua#L6-L29)） | 无 firefx；纯命中表现 | 快照无生成点；`master_postinit` 来自 [event_server_data](../../../raw/current/scripts/prefabs/firehits.lua#L54)（缺口）。`lavaarena_firebomb`／`spear_gungnir`／`spear_lance` 数组只是预载。 |
| `eyeflame` | 完全非联网（无 Network／SetPristine），两发射器（烟＋火），`persists=false`（[L112-L191](../../../raw/current/scripts/prefabs/eyeflame.lua#L112-L191)） | 无等级；本地表现，专服不生成 | [hound](../../../raw/current/scripts/prefabs/hound.lua#L361-L383)、`warg`、`livingtree_halloween` 按 `_eyeflames` netvar 脏标记成对生成并删除。 |

### firefx 等级表（9 个 firefx 注册）

| 注册名 | 档 | anim（按档） | radius | intensity | falloff | colour | 声／音量 | heater |
|---|---:|---|---|---|---|---|---|---|
| `fire` | 6 | level1/2/3/4/4/5（另 `_controlled_burn`） | 2/3/4/5/6/7 | .75/.8×2/.9×3 | .33×3/.25/.2×2 | C5C5AA/FFFFC0×2/FFBE79×3 | campfire×4→forestfire×2；.1/.3/.6/1/1/1 | 30/70/120/180/220/240（20） |
| `campfirefire` | 4 | level1–4 | 2/3/4/5 | .8 | .33 | FFFFC0 | campfire；.1/.3/.6/1 | 70/85/100/115（20） |
| `portable_campfirefire` | 3 | level1/level1a/level2 | 2.5/3/3.5 | .8 | .33 | FFFFC0 | campfire；.1/.2/.3 | 70/77.5/85（20） |
| `coldfirefire` | 4 | level1_redux–4_redux | 2/3/4/5 | .8 | .33 | 00B7FF | coldfire（DLC001）；.1/.3/.6/1 | −10/−20/−30/−40（−20） |
| `nightlight_flame` | 4 | level1–4 | 2/3/4/5 | .8 | .33 | FDB3B3 | nightlight；.1/.3/.6/1 | — |
| `pigtorch_flame` | 3 | level1–3 | 3/4/5 | .8 | .44 | FFFFC0 | campfire；.1/.6/1 | — |
| `character_fire` | 3 | loop_small/med/large（另 pre／pst／pst_fast、`_controlled_burn`） | 2/3/4 | .6/.75/.8 | .7/.5/.33 | C5C5AA/FFFFC0/C5C5AA | campfire/treefire/forestfire；均 1 | 50/65/80（40） |
| `character_fire_flicker` | 3 | 1／2 空表；3 档 loop_fire_flicker（pre／pst 同名） | 3 | .75 | .5 | C5C5AA | treefire；1 | 65（固定 `heat`） |
| `floatinglanternfire` | 4 | level1–4 | 1/1.34/1.67/2 | .8 | .4 | FFFFC0 | floating_LP；.1/.3/.6/1 | — |

- 斜线值按档序，`×n` 表示连续同值；`colour` 为 `Light:SetColour` 的 RGB 源值；`heater` 括号内是等级缺失时的回退值（[GetHeatFn](../../../raw/current/scripts/prefabs/fire.lua#L14-L16)）。

## 关联机制导航

| 类型／对象 | 本页如何使用 | 端与条件 | 入口 |
|---|---|---|---|
| 组件 `firefx` | 等级表→动画、灯参数与火声；`SetLevel`／`SetPercentInLevel`／`Extinguish`；构造时生成 `firefx_light` 并挂父 | 仅主模拟；休眠停 loop 声、唤醒补播 | [[reference/components/firefx]]、[SetLevel](../../../raw/current/scripts/components/firefx.lua#L88-L135) |
| 组件 `heater` | 6 个注册用 `heatfn` 或固定 `heat` 供热；`coldfirefire` 为吸热 | 仅主模拟；`HASHEATER` 在 pristine 前加 | [[reference/components/heater#携带范围与地面热源的分工]]、[GetHeat](../../../raw/current/scripts/components/heater.lua#L53-L56) |
| 组件 `burnable` | `AddBurnFX`→`SpawnFX` 生成 FX，`SetFXLevel` 驱档，`KillFX` 清理 | 主模拟；`fxlevel` 默认 1，`Make*Burnable` 设 1／2／3（corpse 1／2） | [[reference/components/burnable]]、[SpawnFX](../../../raw/current/scripts/components/burnable.lua#L496-L539) |
| 全局 helper `burnfx` | `generic="fire"`／`character="character_fire"` 映射档位；`Make*Burnable` 顺带设 burn 时长与回调 | 主模拟；脚本全局函数，非公开 API | [burnfx](../../../raw/current/scripts/standardcomponents.lua#L158-L198)、[角色 helper](../../../raw/current/scripts/standardcomponents.lua#L245-L282) |
| API `event_server_data` | `firehit` 与 quagmire 分支取事件包；快照无 `*_event_server/` | 仅主模拟 | [定义](../../../raw/current/scripts/entityscript.lua#L77-L86) |
| 资源／缺失类别 | bank/build：`fire`、`campfire_fire`、`coldfire_fire`、`fire_large_character`、`lavaarena_hits_variety`；`smoke_plant` 用 `smoke_out`。无 Replica／SG／Brain | 快照只含 scripts，资源未验证 | 各文件 assets 表 |

## 典型接入

### 给自定义世界实体复用通用燃烧与 fire 特效

- 前提与选择：自定义世界实体（有 Transform、可被点燃）沿用通用 `fire` 六档表；`MakeSmallBurnable` 已封装 `burnable`、`AddBurnFX("fire", …)` 与默认 ignite／extinguish／burnt 回调。
- 生效与撤销：只在主模拟加组件；`SetFXLevel` 可后续改档，移除或 `Extinguish` 时 burnable 负责 `KillFX`。

```lua
-- modmain.lua：自建实体走原版通用燃烧档，特效由 burnable 生成 fire
local function AddBurnable(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    GLOBAL.MakeSmallBurnable(inst, GLOBAL.TUNING.SMALL_BURNTIME)
end

AddPrefabPostInit("my_custom_entity", AddBurnable)
```

- 调整与验证：换 `MakeMediumBurnable`／`MakeLargeBurnable` 即改 `fxlevel` 3／4（`fire` 热度 120／180）；角色外观改 `Make*BurnableCharacter`；偏移／跟随用 `burnable:AddBurnFX("fire", Vector3(0, 1, 0), "swap_fire")`。helper 非公开 API，换版本需复查；未运行验证。

## 常见陷阱与限制

- **直接生成 `character_fire` 不出动画**：工厂不调 `SetLevel`，level 为 nil；除显式调用（cocoon、vault_torch、willow_flame_fx 等）外，通常由 burnable 的 `SpawnFX` 按 `fxlevel` 调 `SetLevel`。`SetLevel` 把等级 clamp 到表长（[L100](../../../raw/current/scripts/components/firefx.lua#L100)），cocoon 传 5 实际落到 3；flicker 的 1／2 档是空表，`SetLevel(1/2)` 会取 nil 的 anim／radius，deerclops 先 `MakeLargeBurnableCharacter`（level 3）再换 prefab 才避开。
- **熄灭动画要表里有 `pst`**：`fire` 无 `pst`，`KillFX` 立即删除；`character_fire` 有 `pst`／`pst_fast`，保留熄火动画并延迟删除（[KillFX](../../../raw/current/scripts/components/burnable.lua#L560-L573)）。
- **`SetBurningFX` 是空转**：只写 `self.fxprefab`，无消费者也无调用方；换 FX 用 `fxdata[i].prefab`（deerclops 做法）或 `AddBurnFX`。`firefx_light` 也无自身参数，须经 `firefx:SetLevel` 才有半径／颜色。
- **端差异**：`firefx`／`heater` 只在主模拟；`eyeflame` 与 `firehit` 的可见体是非联网本地实体，服务端或专服生成看不到；`thurible_smoke` 专服跳过粒子；`floatinglanternfire` 的淡出只在非专服生效。
- **事件数据缺口**：`firehit` 的 `variation` 写入与 `campfirefire` 的 quagmire 行为都在缺失的 `*_event_server/` 中，prefab 数组与 `master_postinit` 调用不能证明生成或效果。

## 验证记录

- 源码审阅：2026-11-04，`steam-buildid:26335394`；逐个核验 13 个范围文件与 `firefx`／`burnable`／`heater`／`standardcomponents`／`entityscript`，抽查宿主 campfire、nightstick、thurible、floatinglantern、deerclops、cocoon、hound、abigail_flower、cotl_tabernacle、stageusher、oceanvine、rock_light、tumbleweed 等（生成点表列代表宿主，未穷举全部 `AddBurnFX` 调用）；全 scripts 反查 15 个注册名，区分直接生成、`AddBurnFX` 通路与仅预载。`fire` 非死代码：`SpawnPrefab("fire")` 与 `"fire"..` 0 命中，但 `AddBurnFX("fire")` 有 12 处直接调用（carrat、leafpile、moontree、palmconetree 等），另有 helper→fxdata→`SpawnFX`；`"fire"` 其余仅预载列表与 tag 判断。
- 名称：`STRINGS.NAMES.FIRE`＝“火”（[chinese_s.po](../../../raw/current/scripts/languages/chinese_s.po#L342770-L342772)），只对应通用 `fire`；其余 14 个注册无 NAMES 键，标题用已核验效果描述，不是官方译名。
- 未覆盖：未运行游戏；`lavaarena`／`quagmire` 事件包不在快照，`firehit` 与 quagmire 分支未核验；anim／sound／贴图不在快照，未验证实际资源；未核验皮肤与 mod 兼容。
- 结构检查：`validate_page.py`、`validate_source_links.py --strict` 与 Lua 片段 `parser_probe --syntax-only` 均通过（见交付报告）。按 `complex` 保留：13 文件 15 注册、9 个 `firefx` 等级表（最多 6 档）与 6 个 `heater` 热度／声画／宿主链需在一页内整体对照，拆页会切断同一工厂的等级与生成清理依据；实测约 16.7 KiB、111 行，在 complex 的 400 行／32 KiB 内。

## 相关链接

- [[reference/components/firefx|firefx]]、[[reference/components/burnable]]、[[reference/components/heater#携带范围与地面热源的分工]]
- 返回 [[reference/prefabs/fx-world-index|特效 Prefab 索引·生物与环境]]
