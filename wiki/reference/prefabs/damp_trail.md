---
title: "地面痕迹与瞬时提示的外围 FX 族（damp_trail、poopcloud、petrify_announce）"
created: "2026-11-04"
updated: "2026-11-08"
type: prefab
summary: "三个一次性外围表现注册：森林复活骸骨的地面拖尾 damp_trail、施肥／便便用的本地便便云 poopcloud、森林石化的纯音效公告 petrify_announce；差异在联网壳与本地代理、三段动画与 1 秒兜底清理。"
aliases: ["damp_trail", "poopcloud", "petrify_announce", "地面拖尾", "便便云", "石化公告音", "外围 FX 族"]
tags: [prefab, fx, server-client]
size_class: complex
dst_build: "steam-buildid:26474873"
sources: ["prefabs/damp_trail.lua", "prefabs/stalker.lua", "stategraphs/SGstalker.lua", "tuning.lua", "prefabs/poopcloud.lua", "prefabs/compost.lua", "prefabs/compostwrap.lua", "prefabs/guano.lua", "prefabs/poop.lua", "prefabs/mosquitofertalizer.lua", "scenarios/chest_foodspoil.lua", "scenarios/sleepingspider_spiderambush.lua", "scenarios/statue_enemywave.lua", "scenarios/wes_enemywave.lua", "prefabs/petrify_announce.lua", "components/forestpetrification.lua", "prefabs/forest.lua", "prefabs/compostingbin.lua", "prefabs/soil_amender.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 地面痕迹与瞬时提示的外围 FX 族（damp_trail、poopcloud、petrify_announce）

## 快速判断

- 职责与适用：三个一次性外围表现注册，均只挂 `FX` 标签、不挂玩法组件：`damp_trail` 是森林形态复活骸骨（`stalker_forest`）行走／死亡时的地面潮湿拖尾，按 `_pre → 主体 → _pst` 三段推进；`poopcloud` 是施肥／便便使用时的便便云，网络壳只负责复制与 1 秒清理，视觉是 `SetFromProxy` 的本地实体；`petrify_announce` 是森林石化完成时的纯音效公告，无 `AnimState`，按本地焦点 20 距离归一化后播放 `post_distant`。共同点是“生成方写位置、实体自删”，端模型分三类：联网动画实体／本地代理子实体／本地音效实体。[damp_trail](../../../raw/current/scripts/prefabs/damp_trail.lua#L44-L72)、[poopcloud](../../../raw/current/scripts/prefabs/poopcloud.lua#L27-L52)、[petrify_announce](../../../raw/current/scripts/prefabs/petrify_announce.lua#L31-L57)
- 优先使用：地面残留拖尾选 `damp_trail`（生成同帧必须调 `SetVariation`）；一次性便便／肥料烟选 `poopcloud`；纯声音提示选 `petrify_announce`。生成点与宿主见“关联机制导航”。
- 重要副作用：`damp_trail` 的 `Refresh` 只续当前未进入淡出的实例，宿主每次 `DoTrail` 会取消上一枚的刷新任务；`poopcloud` 的视觉不联网，专用服务器只保留 1 秒网络壳；`petrify_announce` 的本地音效实体在 `PlaySound` 后立即 `Remove`。三者均 `persists=false`。
- 修改边界：只覆盖三个工厂、端分界、生成点、父挂与清理；宿主玩法（stalker 开花、施肥、森林石化周期）、动画包实际内容与多人时序不展开。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `damp_trail` | `fn`：Transform／AnimState／SoundEmitter／Network → `FX` → bank／build `damp_trail`、`LAYER_BACKGROUND`、`SetSortOrder(3)` → Pristine → 主模拟导出 `SetVariation`／`Refresh`、`persists=false`、`DoTaskInTime(0, Remove)` 兜底 | 联网动画实体；生成方必须同帧 `SetVariation(rand, scale, duration)` 才取消兜底并进入三段动画；`SoundEmitter` 未播放声音。[工厂](../../../raw/current/scripts/prefabs/damp_trail.lua#L44-L72)、[三段](../../../raw/current/scripts/prefabs/damp_trail.lua#L6-L21) |
| `poopcloud` | `fn`：Transform／Network → `FX` → 非专用端 `DoTaskInTime(0, PlayCloudAnim)` → Pristine → 主模拟 `persists=false`＋1 秒 `Remove` | 网络壳无 AnimState；`PlayCloudAnim(proxy)` 另造本地实体，`SetFromProxy(proxy.GUID)`，bank `poopcloud`、build `poop_cloud`、anim `idle`、`SetFinalOffset(3)`，`animover` 自删。[工厂](../../../raw/current/scripts/prefabs/poopcloud.lua#L27-L52)、[代理](../../../raw/current/scripts/prefabs/poopcloud.lua#L6-L25) |
| `petrify_announce` | `fn`：Transform／Network → `FX` → 非专用端 `DoTaskInTime(0, PlayPetrifySound)` → Pristine → 主模拟 `SetCanSleep(false)`、`persists=false`＋1 秒 `Remove` | 无 AnimState、无 assets；本地实体只有 Transform＋SoundEmitter，按 `TheFocalPoint` 距离把播放点归一到最多 20，否则 `SetFromProxy`；播 `post_distant` 后立即删。[工厂](../../../raw/current/scripts/prefabs/petrify_announce.lua#L31-L57)、[归一化](../../../raw/current/scripts/prefabs/petrify_announce.lua#L9-L25) |

名称：三个注册名在简中 `.po` 均无 `STRINGS.NAMES.*` 键；标题中的中文是已核验的效果描述，不是官方译名。`damp_trail`／`poopcloud` 各有一组动画资源（`anim/damp_trail.zip`、`anim/poop_cloud.zip`），`petrify_announce` 只有内置音效路径。

## 关联机制导航

### 生成点与宿主

| 生成点 | 涉及注册 | 端、阶段与条件 | 证据 |
|---|---|---|---|
| `stalker_forest` 的 `forest_master_postinit`／`StartBlooming` | `damp_trail` | 主模拟；开花开始时 `OnStartBlooming` 先生成一枚，随后 `inst.DoTrail` 由 `SGstalker` 帧事件重复调用 | [森林后初始化](../../../raw/current/scripts/prefabs/stalker.lua#L1602-L1620)、[开花起点](../../../raw/current/scripts/prefabs/stalker.lua#L1236-L1239)、[注册](../../../raw/current/scripts/prefabs/stalker.lua#L1797) |
| `SGstalker` 的 `walk`／`walk_stop`／`death2` | `damp_trail` | 主模拟；`DoTrail` 包装先判 `inst.foreststalker`，仅森林变体生效；`walk` 帧 1／18、`walk_stop` 帧 1、`death2` 帧 20／38／54 | [守卫](../../../raw/current/scripts/stategraphs/SGstalker.lua#L49-L53)、[walk](../../../raw/current/scripts/stategraphs/SGstalker.lua#L452-L454)、[death2](../../../raw/current/scripts/stategraphs/SGstalker.lua#L622-L636) |
| `compost`／`compostwrap`／`poop`／`mosquitofertalizer` 的 `FuelTaken` | `poopcloud` | 主模拟；取 `taker` 的 burnable fx 或自身位置，`y+1` 生成 | [compost](../../../raw/current/scripts/prefabs/compost.lua#L15-L24)、[compostwrap](../../../raw/current/scripts/prefabs/compostwrap.lua#L27-L36)、[poop](../../../raw/current/scripts/prefabs/poop.lua#L24-L33)、[mosquitofertalizer](../../../raw/current/scripts/prefabs/mosquitofertalizer.lua#L40) |
| `guano` 的 `FuelTaken` | `poopcloud` | 主模拟；直接在 `taker` 世界位置生成（不 `y+1`） | [guano](../../../raw/current/scripts/prefabs/guano.lua#L24-L26) |
| `scenarios/chest_foodspoil`、`sleepingspider_spiderambush`、`statue_enemywave`、`wes_enemywave` | `poopcloud` | 场景脚本；一次性位置生成，`chest_foodspoil` 另把缩放设为 1.5 并播放 `toxic_cloud`；是否在 current 世界流程中启用未核验 | [chest_foodspoil](../../../raw/current/scripts/scenarios/chest_foodspoil.lua#L45-L49)、[sleepingspider](../../../raw/current/scripts/scenarios/sleepingspider_spiderambush.lua#L24-L26)、[statue](../../../raw/current/scripts/scenarios/statue_enemywave.lua#L47-L49)、[wes](../../../raw/current/scripts/scenarios/wes_enemywave.lua#L49-L51) |
| `forestpetrification` 的 `PetrifyForest` | `petrify_announce` | 主模拟；石化完成后按被石化树的平均坐标 `(xsum/count, 0, zsum/count)` 生成；`count` 为 0 时不生成 | [PetrifyForest](../../../raw/current/scripts/components/forestpetrification.lua#L152-L166)、[世界挂载](../../../raw/current/scripts/prefabs/forest.lua#L668) |

### 端、挂接与清理

| 注册名 | 父挂／跟随 | 清理路径 | 证据 |
|---|---|---|---|
| `damp_trail` | 无 `SetParent`、无 `Follower`；生成方写世界坐标 | `SetVariation` 取消 0 帧兜底 → `_pre` 的 `animover` 播主体并起 `duration` 计时 → 到点播 `_pst` → `_pst` 的 `animover` 删除；`Refresh` 只重排未到点的淡出任务；`persists=false` | [三段](../../../raw/current/scripts/prefabs/damp_trail.lua#L6-L21)、[刷新](../../../raw/current/scripts/prefabs/damp_trail.lua#L37-L42) |
| `poopcloud` | 无 `SetParent`；本地代理 `SetFromProxy(proxy.GUID)` 跟随网络壳 | 本地代理 `animover` 自删；网络壳 1 秒 `Remove`；专用服务器不造本地代理；`persists=false` | [代理](../../../raw/current/scripts/prefabs/poopcloud.lua#L17)、[1 秒](../../../raw/current/scripts/prefabs/poopcloud.lua#L48-L49) |
| `petrify_announce` | 无 `SetParent`；本地音效实体用焦点归一化坐标或 `SetFromProxy` | 本地实体 `PlaySound` 后立即 `Remove`；网络壳 1 秒 `Remove`；专用服务器不造本地实体；`persists=false` | [播放与删除](../../../raw/current/scripts/prefabs/petrify_announce.lua#L27-L28)、[1 秒](../../../raw/current/scripts/prefabs/petrify_announce.lua#L53-L54) |

### 依赖、资源与数据

| 类型／对象 | 本注册如何使用 | 条件与边界 | 证据 |
|---|---|---|---|
| 依赖声明 | `stalker.lua` 的 `prefabs_forest` 声明 `damp_trail`，`forest.lua` 的 `prefabs` 声明 `petrify_announce`，`compostingbin.lua`／`soil_amender.lua` 的 `prefabs` 表声明 `poopcloud`；声明不证明生成 | 仅加载保证；实际生成点见上表 | [stalker](../../../raw/current/scripts/prefabs/stalker.lua#L44-L53)、[forest](../../../raw/current/scripts/prefabs/forest.lua#L316)、[compostingbin](../../../raw/current/scripts/prefabs/compostingbin.lua#L9-L14)、[soil_amender](../../../raw/current/scripts/prefabs/soil_amender.lua#L9-L12) |
| `TUNING.STALKER_BLOOM_DECAY` | `damp_trail` 的 `duration` 与宿主刷新周期都用它：值 5 秒，刷新任务每 2.5 秒一次 | 主模拟；只影响 `stalker_forest` 生成链 | [tuning](../../../raw/current/scripts/tuning.lua#L3390)、[DoTrail](../../../raw/current/scripts/prefabs/stalker.lua#L1199-L1203) |
| `TheFocalPoint` | `petrify_announce` 用它把远处音源归一到 20 距离；为空时退回 `SetFromProxy` | 本地焦点实体；专用端不生成本地音效实体 | [归一化](../../../raw/current/scripts/prefabs/petrify_announce.lua#L9-L25) |
| 资源 | `anim/damp_trail.zip`、`anim/poop_cloud.zip`；`petrify_announce` 仅内置音效 `dontstarve/common/together/petrified/post_distant` | 动画包内容、帧数与循环标记未解包核验 | [damp_trail 资源](../../../raw/current/scripts/prefabs/damp_trail.lua#L1-L4)、[poopcloud 资源](../../../raw/current/scripts/prefabs/poopcloud.lua#L1-L4) |

## 典型接入

### 复用 damp_trail 的三段地面拖尾

- 前提与选择：需要“按步或按事件在地面留一枚会淡出的拖尾”时用 `damp_trail`；原版由 `stalker_forest` 在走路帧事件里生成，位置写世界坐标，移动中额外向朝向偏移 1 单位。
- 生效与撤销：必须在 `SpawnPrefab` 同帧调 `SetVariation(rand, scale, duration)`；`rand` 取 1–7，`scale` 为整体缩放，`duration` 为秒。之后可周期调 `Refresh()` 续命，但进入 `_pst` 后不再生效；实体最终由动画 `animover` 删除。
- 原版依据：[DoTrail](../../../raw/current/scripts/prefabs/stalker.lua#L1190-L1203)、[SetVariation](../../../raw/current/scripts/prefabs/damp_trail.lua#L23-L35)

```lua
local function spawn_damp_trail(x, z, variation, scale, duration)
    if not TheWorld.ismastersim then
        return
    end
    local fx = SpawnPrefab("damp_trail")
    fx.Transform:SetPosition(x, 0, z)
    fx:SetVariation(variation, scale, duration) -- 必须同帧调用，否则 0 帧兜底已删除实体
    return fx
end
```

- 调整与验证：`variation` 超出 1–7 时依赖动画包内存在对应段，工厂不校验；`duration` 单位秒。示例只做语法检查，未在游戏内执行。

### 生成一次性便便云与石化公告

- 前提与选择：`poopcloud` 适合任意点的一次性烟；`petrify_announce` 适合“只出声、不要画面”的提示。两者都由服务端生成网络壳，非专用端各自造本地表现。
- 生效与撤销：`poopcloud` 的本地视觉在 `animover` 自删，网络壳 1 秒后删；`petrify_announce` 的本地音效实体播完即删，网络壳 1 秒后删。不要在客户端重复 `SpawnPrefab`，否则会多一层本地表现。
- 原版依据：[poopcloud 工厂](../../../raw/current/scripts/prefabs/poopcloud.lua#L27-L52)、[petrify_announce 工厂](../../../raw/current/scripts/prefabs/petrify_announce.lua#L31-L57)

```lua
-- 仅在服务端调用；客户端重复生成会多造一层本地表现
local function spawn_poop_cloud(x, y, z)
    SpawnPrefab("poopcloud").Transform:SetPosition(x, y, z)
end

local function spawn_petrify_announce(x, z)
    SpawnPrefab("petrify_announce").Transform:SetPosition(x, 0, z)
end
```

- 调整与验证：`poopcloud` 的 `y` 由生成方决定，原版肥料类多为 `y+1`；`petrify_announce` 的 20 距离归一化在本地实体创建时按 `TheFocalPoint` 计算，生成方只需给源位置。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：`damp_trail` 必须同帧 `SetVariation`。** 工厂用 `DoTaskInTime(0, inst.Remove)` 兜底；`SetVariation` 内部直接 `inst.task:Cancel()`，跨帧调用时实体已被 0 帧任务 Remove（`inst.task` 仍是已结束任务的引用，不是 `nil`），随后的 `SetScale`／`PlayAnimation` 会落在已移除实体上。位置也要在 `SetVariation` 前写好。[兜底](../../../raw/current/scripts/prefabs/damp_trail.lua#L69)、[SetVariation](../../../raw/current/scripts/prefabs/damp_trail.lua#L23-L35)
- **边界：`Refresh` 不能复活已进入淡出的拖尾。** `OnStartFade` 已把 `inst.task` 置 `nil` 并播 `_pst`；`Refresh` 只在 `trailname ~= nil and task ~= nil` 时重排任务。宿主 `DoTrail` 每次新建 fx 并取消上一枚的 `_trailtask`，所以只有最新拖尾会被周期刷新。[Refresh](../../../raw/current/scripts/prefabs/damp_trail.lua#L37-L42)、[刷新任务](../../../raw/current/scripts/prefabs/stalker.lua#L1200-L1203)
- **边界：`damp_trail` 的 `SetVariation`／`Refresh` 只在主模拟存在。** 客户端在 Pristine 后早退，不能调用这两个方法；动画状态随联网实体同步（引擎层，未运行核验）。[端分界](../../../raw/current/scripts/prefabs/damp_trail.lua#L59-L63)
- **边界：`poopcloud` 的视觉不在网络壳上。** 网络壳只有 Transform／Network，没有 AnimState；视觉是 `SetFromProxy` 的本地实体。专用服务器不生成视觉，只保留 1 秒网络壳。[网络壳](../../../raw/current/scripts/prefabs/poopcloud.lua#L27-L33)、[代理](../../../raw/current/scripts/prefabs/poopcloud.lua#L6-L25)
- **边界：`petrify_announce` 没有 AnimState。** 它不能当可视 FX 用，只有 SoundEmitter；20 距离归一化只改播放点，不改音量或衰减参数。[工厂](../../../raw/current/scripts/prefabs/petrify_announce.lua#L31-L44)、[归一化](../../../raw/current/scripts/prefabs/petrify_announce.lua#L9-L25)
- **边界：本地代理与网络壳谁先消失取决于动画长度。** `poopcloud` 的本地代理按 `animover` 自删，网络壳固定 1 秒删；动画短于 1 秒时本地先删、壳后删；若动画长于 1 秒则网络壳先 Remove，`SetFromProxy` 的跟随在壳删除后的表现未运行核验。[清理](../../../raw/current/scripts/prefabs/poopcloud.lua#L42-L49)

## 验证记录

- 源码审阅：2026-11-04，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。三个工厂全文、注册名、端分支、父挂／Follower（均无）、清理路径已核；生成点按完整注册名在 `raw/current/scripts` 全域 `grep -rn --include=*.lua` 反查：`damp_trail` 命中 `stalker.lua`（依赖表＋`DoTrail`）、`SGstalker.lua` 帧事件；`poopcloud` 命中 5 个肥料 prefab 的 `FuelTaken`、4 个 scenarios 脚本与依赖表；`petrify_announce` 命中 `forestpetrification.lua` 与 `forest.lua` 依赖表。`TUNING.STALKER_BLOOM_DECAY=5`、`TheFocalPoint` 分支、`SetFromProxy` 代理与 1 秒兜底均已核对。
- 名称核验：`languages/chinese_s.po` 定向检索 `DAMP_TRAIL`／`POOPCLOUD`／`PETRIFY_ANNOUNCE` 无 `STRINGS.NAMES.*` 键；标题使用效果描述。
- 游戏运行：未运行；动画包内容、客户端可见时序、scenarios 是否启用、本地代理在壳先删后的跟随均未实测。
- 分档说明：覆盖 3 个注册、3 个工厂、3 种端模型（联网动画／本地代理／本地音效）与 10 个以上生成点，需整体对照保留，故 `size_class: complex`。
- 结构检查：`validate_page.py` 通过（0 issue；133 行、17,810 字节，低于 complex 档 400 行／32 KiB）；`validate_source_links.py --strict` 通过（errors 0、warnings 0；55 处源码引用）；正文两段 Lua 示例另存临时文件经 `parser_probe --syntax-only` 通过（只查 Lua 5.1 语法，未执行）。
- 未覆盖：三个动画／音效资源的实际帧数、循环标记与观感；未穷举运行时拼接名（本族以字面注册名生成）；`poopcloud` 在晚加入客户端的代理创建时序；`petrify_announce` 在专用服务器无本地焦点时的声音行为。

## 相关链接

- [[reference/prefabs/ink_puddle_land|墨水与胆汁洼地]]（同为一次性贴地 FX；对照生成点与 `animover` 自删）
- [[reference/prefabs/gestalt_trail|宿主附着与跟随的外围 FX 族]]（同为多注册外围 FX 族；对照世界点拖尾与清理）
- [[reference/prefabs/lunar_goop_trail_fx|牧者黏液拖尾]]（三段式拖尾协议的另一种实现）
- [[reference/components/forestpetrification|forestpetrification]]（`petrify_announce` 的生成者）
- [[reference/prefabs/fx-world-index|特效 Prefab 索引·生物与环境]]、[[reference/prefabs/fx-index|特效 Prefab 索引]]

返回 [[reference/prefabs/fx-world-index|天气、地形与生态痕迹]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-06）。
