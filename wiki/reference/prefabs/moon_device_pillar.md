---
title: "月装置支柱、顶盖、一阶段破碎与天体裂缝裂纹层（moon_device_pillar、moon_device_top、moon_device_break_stage1、moon_fissure_fx）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "月亮虹吸器的阶段支柱与顶盖、首次破碎动画，以及天体裂缝按月相显隐的裂纹层四个独立注册：工厂、生成条件、父挂、端与清理。"
aliases: ["moon_device_pillar", "moon_device_top", "moon_device_break_stage1", "moon_fissure_fx", "月装置支柱", "月装置顶盖", "月装置破碎", "天体裂缝裂纹层", "月亮虹吸器支柱", "月亮虹吸器顶盖", "月装置外围表现"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/moon_device.lua", "prefabs/moon_fissure.lua", "prefabs/moon_altar.lua", "components/retrofitforestmap_anr.lua", "entityscript.lua", "simutil.lua", "map/forest_map.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 月装置支柱、顶盖、一阶段破碎与天体裂缝裂纹层（moon_device_pillar、moon_device_top、moon_device_break_stage1、moon_fissure_fx）

## 快速判断

- 职责与适用：四注册都是纯表现，没有生命、战斗、可工作或容器组件；它们分别是装置阶段 2／3 的四向支柱与顶端、装置首次破碎动画、天体裂缝按月相切层的裂纹。均非 `fx.lua` 通用表项，不能套用 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]] 的代理／自删协议。
- 优先使用：给宿主叠按月相切层的裂纹，看 `moon_fissure_fx` 与宿主 `UpdateState`；复现装置首破，看 `moon_device_break_stage1`；把支柱／顶端当装饰复用，看“典型接入”。
- 重要副作用：支柱／顶端联网父挂、无标签、不持久；首破 FX 生成 1.5 秒后 `ErodeAway`；裂纹层联网、`DECOR`／`NOCLICK`、无自删。首破同帧的清场、伤害与月坛换裂缝属装置本体，不在这四个注册里。
- 修改边界：装置本体／construction1／2／placer／meteor_spawner 只作生成条件；stage2／3 破碎见 [[reference/prefabs/moonstorm_glass_fx]]；月坛连接与月脉冲见 [[reference/prefabs/moon_altar_link_fx]]。

## 注册与构造

四注册都由各自文件内的工厂函数直接建实体；公共骨架为 Transform／AnimState／Network → `SetPristine` → 客户端早退 → 主模拟 `persists = false`。

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `moon_device_pillar` | `pillarfn`：`SetEightFaced` → bank `moon_device_stages`／build `moon_device`／`stage2_idle`（未传循环参数）→ override 为 `moon_device` → 客户端早退 → 主模拟 `persists = false`；无组件、无标签。 | [工厂 L121-L145](../../../raw/current/scripts/prefabs/moon_device.lua#L121-L145)、[注册 L638](../../../raw/current/scripts/prefabs/moon_device.lua#L638-L638) |
| `moon_device_top` | `topfn`：同骨架；旋转 45，循环 `stage3_idle`（显式 `true`），同样 override 为 `moon_device`、`persists = false`、无组件无标签。 | [工厂 L147-L172](../../../raw/current/scripts/prefabs/moon_device.lua#L147-L172)、[注册 L639](../../../raw/current/scripts/prefabs/moon_device.lua#L639-L639) |
| `moon_device_break_stage1` | `break_stage1_fn`：bank 与 build 均为 `moon_device_break`、`stage1_break`（显式 `false`）→ 标签 `FX` → 客户端早退 → 主模拟 `persists = false` 且 `DoTaskInTime(1.5, ErodeAway)`；无朝向／层／排序调用。 | [工厂 L567-L591](../../../raw/current/scripts/prefabs/moon_device.lua#L567-L591)、[注册 L643](../../../raw/current/scripts/prefabs/moon_device.lua#L643-L643) |
| `moon_fissure_fx` | `fx_fn`：bank 与 build 均为 `moon_fissure_fx`、循环 `crack_idle` → `DECOR`／`NOCLICK` → 客户端早退 → 主模拟 `persists = false`；无朝向／层／排序／灯光调用。`AddNetwork` 注释称联网是为了在其上触发动画。 | [工厂 L220-L243](../../../raw/current/scripts/prefabs/moon_fissure.lua#L220-L243)、[注册 L292](../../../raw/current/scripts/prefabs/moon_fissure.lua#L292-L292) |

- `SetPrefabNameOverride(nameoverride)` 只写 `inst.nameoverride`，注释明确“只改描述与名称显示、不覆盖 prefab 本身”；`GetBasicDisplayName` 在其非空时回退到 `STRINGS.NAMES`。支柱／顶端的 `prefab` 仍是各自注册名，不能据显示名判定实体是 `moon_device`。[语义 L846-L850](../../../raw/current/scripts/entityscript.lua#L846-L850)、[读取 L687-L692](../../../raw/current/scripts/entityscript.lua#L687-L692)
- 声明不等于生成：支柱／顶端注册沿用装置的整份 `assets`／`prefabs`（含本页外的破碎 2／3、天体英雄、陨石生成器等）；`moon_fissure_fx` 传的也是含宿主 `anim/moon_fissure.zip` 的整份 `assets`，且没有 `prefabs` 参数。[装置声明 L1-L32](../../../raw/current/scripts/prefabs/moon_device.lua#L1-L32)、[裂纹注册 L292](../../../raw/current/scripts/prefabs/moon_fissure.lua#L292-L292)
- 名称：简中 `.po` 定向检索四个注册名的 `STRINGS.NAMES` 键均无命中，上述中文都是效果描述，非官方译名。宿主 `moon_device`＝“月亮虹吸器”、`moon_fissure`＝“天体裂缝”命中官方键，仅用于说明来源。[月亮虹吸器 L345829-L345832](../../../raw/current/scripts/languages/chinese_s.po#L345829-L345832)、[天体裂缝 L345854-L345857](../../../raw/current/scripts/languages/chinese_s.po#L345854-L345857)

## 关联机制导航

装置本体与天体裂缝本体只取与本页四注册有关的生成、父挂和层控制，其余生命周期不在本页。

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成点：装置阶段 | 成品 `moon_device` 不带构造参数、`level = 3`；建造阶段 1／2 带 `construction_data`。`level >= 2` 构造时生成四柱，`level == 3` 再加顶端与自毁任务；施工完成由 `OnConstructed` 换成品，再在新实例上补播 `stage2_idle_pre`／`stage3_idle_pre`。 | 主模拟；单例与“必须落在月坛连接点上”的校验属本体，本页只取其对生成时机的影响。 | [等级 L402-L406](../../../raw/current/scripts/prefabs/moon_device.lua#L402-L406)、[生成条件 L453-L468](../../../raw/current/scripts/prefabs/moon_device.lua#L453-L468)、[升级 L69-L88](../../../raw/current/scripts/prefabs/moon_device.lua#L69-L88) |
| 生成点：首破 | 仅 `level == 3`：`BREAK_DELAY`（9.5 秒）前 1 秒播预警音，到点进入 `breaksequence`；先落 `alterguardian_phase1fallfx`，9 帧后 `break_device` 依次执行 stage3／stage2／stage1 破碎，stage1 在装置世界坐标 y=0 生成本 FX、`moon_geyser_explode` 与灼痕。 | 主模拟；落点式，不父挂。 | [延迟 L48](../../../raw/current/scripts/prefabs/moon_device.lua#L48-L48)、[时序 L467-L468](../../../raw/current/scripts/prefabs/moon_device.lua#L467-L468)、[序列 L329-L335](../../../raw/current/scripts/prefabs/moon_device.lua#L329-L335)、[首破 L289-L305](../../../raw/current/scripts/prefabs/moon_device.lua#L289-L305) |
| 生成点：裂缝宿主 | `moon_fissure` 由世界生成的 `moon_fissure` 类型放置；装置破碎的 `ClearArea` 也会在半径 15 内每个月坛（`moon_altar`／`_cosmic`／`_astral`）位置生成一枚并把月坛移除；锤毁月坛与旧档改装也会生成。裂纹层只由宿主 `fn` 生成，没有其他生成点。 | 主模拟；宿主可存档，裂纹层 `persists = false`。 | [类型映射 L233](../../../raw/current/scripts/map/forest_map.lua#L233-L233)、[装置侧生成 L251-L263](../../../raw/current/scripts/prefabs/moon_device.lua#L251-L263)、[锤毁月坛](../../../raw/current/scripts/prefabs/moon_altar.lua#L282-L294)、[旧档改装](../../../raw/current/scripts/components/retrofitforestmap_anr.lua#L693-L697)、[裂纹层生成 L210-L211](../../../raw/current/scripts/prefabs/moon_fissure.lua#L210-L211) |
| 父挂与端 | 支柱：`addpillar` 生成后 `entity:SetParent(装置.entity)`，再按 `-2.7`／`0`／`+2.7` 四个偏移与 0／90／180／270 旋转摆放；顶端：`spawntop` 生成后父挂；裂纹层：宿主生成后 `SetParent`。 | 主模拟生成；客户端通过联网实体与父级关系拿到实例。代码走 `entity:SetParent`，不经 `AddChild`，`EntityScript.children` 不登记。 | [支柱 L97-L119](../../../raw/current/scripts/prefabs/moon_device.lua#L97-L119)、[顶端 L174-L179](../../../raw/current/scripts/prefabs/moon_device.lua#L174-L179)、[裂纹层父挂](../../../raw/current/scripts/prefabs/moon_fissure.lua#L210-L211)、[AddChild L921-L935](../../../raw/current/scripts/entityscript.lua#L921-L935) |
| 清理 | 支柱／顶端：破碎时 `stage2_break`／`stage3_break` 下一帧逐个 `Remove`；两者不单独存档。首破 FX：1.5 秒后 `ErodeAway`，默认腐蚀 1 秒，线程结束才 `Remove`；不监听 `animover`。裂纹层：无自删，宿主被装上月坛部件时直接 `Remove` 宿主。 | 主模拟；宿主删除时 `entity:SetParent` 子实体的引擎级联未核验。 | [柱／顶移除 L190-L204](../../../raw/current/scripts/prefabs/moon_device.lua#L190-L204)、[ErodeAway 调用 L588](../../../raw/current/scripts/prefabs/moon_device.lua#L588-L588)、[实现 L575-L596](../../../raw/current/scripts/simutil.lua#L575-L596)、[宿主移除 L69-L75](../../../raw/current/scripts/prefabs/moon_fissure.lua#L69-L75) |
| 月相层、数据与排序 | 宿主 `UpdateState` 把 `low`／`med`／`high`／`full` 从宿主层隐藏，并按 `lightstate_data` 在裂纹层上显示；`backing` 按亮灯与否在两者间对调；`delayed_transition` 在两侧同播 `transition`→`crack_idle`，月相变化先取消旧任务再随机 0～1 秒后过渡。`MOON_STATES` 把月相映射 1～5；`_level` 是 `net_tinybyte`（`moonfissure.level`，脏事件名 `leveldirty`）；本 prefab 不监听 `leveldirty`，工厂在客户端早退前注册 0 周期 `OnUpdateLight` 并调用一次，主模拟另有睡眠取消／唤醒恢复，`aurafn` 与 `getstatus` 也读 `_level`。排序上宿主 `SetFinalOffset(3)`，裂纹层未设层／排序。 | 层显隐与过渡只在主模拟；客户端读 `_level` 做灯光插值，跨端层同步未核验。 | [层控制 L85-L108](../../../raw/current/scripts/prefabs/moon_fissure.lua#L85-L108)、[过渡 L110-L125](../../../raw/current/scripts/prefabs/moon_fissure.lua#L110-L125)、[状态映射 L36-L43](../../../raw/current/scripts/prefabs/moon_fissure.lua#L36-L43)、[netvar L174-L175](../../../raw/current/scripts/prefabs/moon_fissure.lua#L174-L175)、[宿主 FinalOffset L172](../../../raw/current/scripts/prefabs/moon_fissure.lua#L172-L172) |
| 关联 Prefab | `moon_device_break_stage2`／`_stage3` 是通用表注册，细节见 [[reference/prefabs/moonstorm_glass_fx]]；装置内连接件 `moon_altar_link_contained` 是另一个 Prefab（本页不展开），与 [[reference/prefabs/moon_altar_link_fx]] 的通用表 FX 不是同一对象。 | — | [连接件生成 L181-L188](../../../raw/current/scripts/prefabs/moon_device.lua#L181-L188) |

## 典型接入

### 把装置支柱挂到自定义宿主

- 前提与选择：仅主模拟；目标是已存在、联网的宿主实体。原版 `addpillar` 就是“生成 → 父挂 → 设偏移与旋转”三步，复用支柱照抄即可。[原版 L97-L104](../../../raw/current/scripts/prefabs/moon_device.lua#L97-L104)
- 生效与撤销：父挂后随宿主移动；宿主删除的级联未核验，自建时显式 `Remove`（原版在破碎时显式移除柱／顶）。支柱 `persists = false`，不单独存档，宿主重建时需再次生成。
- 原版依据：四向偏移与旋转见 `spawnpillars`（L106-L119）；支柱工厂已内置八向、动画与名称 override，调用方不必再设。

```lua
local function AttachPillar(host, x, z, rotation)
    local pillar = SpawnPrefab("moon_device_pillar")
    pillar.entity:SetParent(host.entity)
    pillar.Transform:SetPosition(x, 0, z)
    pillar.Transform:SetRotation(rotation)
    return pillar
end
```

- 调整与验证：偏移取原版 2.7（四向为 `-2.7`／`+2.7`），旋转取 0／90／180／270；返回值由调用方保存以便显式清理。不要拿它做 `prefab == "moon_device"` 判断：显示名被 override，`prefab` 仍是 `moon_device_pillar`。

## 常见陷阱与限制

- **误区：显示名是 `moon_device` 就等于生成的是本体。** override 只写 `nameoverride`，`prefab`、标签、组件、网络身份都不变；只有名称读取走 override（证据见上文 L846-L850）。
- **依赖声明不是生成点。** 支柱／顶端的 `prefabs` 表与装置共用，含本页外对象；据此推断“支柱会生成天体英雄”等是错误的。
- **父挂不等于自动清理。** 支柱、顶端与裂纹层都用 `entity:SetParent`，没有 `AddChild`（首破 FX 是落点式）；原版只在破碎时机显式移除柱／顶，裂纹层没有任何移除调用。宿主 `Remove` 时子实体循环只覆盖 `EntityScript.children`，引擎侧级联未核验；自建宿主应在移除前显式清理。[AddChild L921-L935](../../../raw/current/scripts/entityscript.lua#L921-L935)、[子实体循环 L1779-L1784](../../../raw/current/scripts/entityscript.lua#L1779-L1784)
- **层的显隐只在主模拟。** `UpdateState`／`delayed_transition` 都在主模拟路径上，脚本没有 netvar 事件或 RPC 让客户端重放；联网注释只说“在其上触发动画”。跨端是否一致未运行核验，照抄时不能让客户端依赖这套调用。[主模拟 L214](../../../raw/current/scripts/prefabs/moon_fissure.lua#L214-L214)
- **`leveldirty` 是引擎脏事件，但本 prefab 没有监听。** `net_tinybyte(..., "leveldirty")` 的第三参数就是引擎在该实体上触发的脏事件名（`components/lavaarenamusic.lua`、`quagmire_music.lua` 有 `ListenForEvent("leveldirty", …)` 的用法）；本文件不监听该事件，但并非只写不读：两端周期 `OnUpdateLight`、主模拟 `aurafn` 与 `getstatus` 都读取 `_level`。需要即时响应脏事件时再自行 `ListenForEvent("leveldirty")`。
- **月相切换有随机延迟且会取消未执行任务。** 连续变化只保留最后一次；0～1 秒内看到的仍是旧层。
- **首破 FX 不结算伤害。** 同一帧的 `ClearArea` 才清理半径 15 内的可破坏物、对近距战斗单位结算 `ALTERGUARDIAN_PHASE1_ROLLDAMAGE`、把月坛换成 `moon_fissure`、弹飞物品；`moon_device_break_stage1` 只是动画。[清场 L251-L281](../../../raw/current/scripts/prefabs/moon_device.lua#L251-L281)
- **首破 FX 的寿命不是 1.5 秒。** 1.5 秒后才开始默认 1 秒的腐蚀，线程结束才 `Remove`；没有 `animover` 收尾。[调用 L588](../../../raw/current/scripts/prefabs/moon_device.lua#L588-L588)
- **装置不是静态摆件。** `level == 3` 的自毁任务在构造时排入，静态放置也会在 9.5 秒后破碎并生成本页首破 FX。
- **同文件的 `moon_fissure_plugged` 才是 `animover` 驱动。** 它的 toot 动画链与本页裂纹层无关，不要混用清理路径。[L275-L284](../../../raw/current/scripts/prefabs/moon_fissure.lua#L275-L284)

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394（实读 `raw/current/BUILD.txt`）。逐个核验四注册的工厂参数（bank／build／anim 与循环标志、override、标签、层／排序）、端分界、生成点、父挂与清理，并追到 `ErodeAway`、`AddChild`／`Remove`、`SetPrefabNameOverride` 的实现。全 `raw/current/scripts` 反查四个注册名，命中仅注册行、父挂／生成点（`moon_device.lua` L98、L176、L294 与 `moon_fissure.lua` L210）及依赖声明；`moon_fissure` 反查另见世界生成类型表、静态布局、旧存档改装组件与装置侧 `ClearArea`，均不直接生成本页 FX。简中 `.po` 的四个 `STRINGS.NAMES` 键均无命中，宿主键只用于说明中文来源。未覆盖：动画 zip 内容与帧长、客户端层显隐／过渡同步、`entity:SetParent` 子实体在宿主删除时的引擎行为、模组生成、专用服务器与晚加入。
- 游戏运行：未运行。
- 结构检查：`validate_page.py`（standard 档）与 `validate_source_links.py --strict` 通过；文内 Lua 片段经 `parser_probe.exe --syntax-only` 通过。

返回 [[reference/prefabs/fx-structure-index|建筑、装置与遗迹演出]]。
