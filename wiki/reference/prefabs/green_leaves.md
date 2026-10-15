---
title: "树叶飘落与砍伐（green_leaves 族）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "通用表四色八注册的树叶表现：绿叶／红叶／橙叶／黄叶各配飘落与砍伐两动画，由桦树林落叶树、越冬树与庆典广场树按树色与事件调用。"
aliases: ["green_leaves", "red_leaves", "orange_leaves", "yellow_leaves", "green_leaves_chop", "red_leaves_chop", "orange_leaves_chop", "yellow_leaves_chop", "落叶", "树叶", "四季树叶", "树叶飘落", "砍树叶", "落叶特效", "秋天落叶"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fx.lua", "fx.lua", "prefabs/deciduoustrees.lua", "prefabs/winter_tree.lua", "prefabs/carnival_plaza.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 树叶飘落与砍伐（green_leaves 族）

## 快速判断

- 职责与适用：落叶树（桦树林树）与越冬树的树叶纯表现：`*_leaves` 为飘落、`*_leaves_chop` 为砍伐飞叶，四色两动画共 8 注册；无玩法组件。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L364-L425)
- 优先使用：查"落叶／砍树飞叶"选本页；树的砍伐回调与可工作流程见 [[reference/components/workable|workable]]。
- 重要副作用：无；全部条目带 `leaf_rustle` 叶响音效。[条目](../../../raw/current/scripts/fx.lua#L364-L390)
- 修改边界：随生成一次性落点；动画是否循环未核验。[代理回收](../../../raw/current/scripts/prefabs/fx.lua#L156-L157)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `green_leaves_chop`／`red_leaves_chop`／`orange_leaves_chop`／`yellow_leaves_chop` | 通用表条目 → 加载器代理＋一帧后本地实体 | bank=`tree_leaf_fx`，build=`tree_leaf_fx_<color>`，anim=`chop`，音效 `dontstarve_DLC001/fall/leaf_rustle`。[循环四条](../../../raw/current/scripts/fx.lua#L364-L390) |
| `green_leaves`／`red_leaves`／`orange_leaves`／`yellow_leaves` | 同上协议 | 同 bank／build，anim 换 `fall`；音效相同。[四条](../../../raw/current/scripts/fx.lua#L399-L425) |

定向检索简中 .po 未发现这 8 个注册名的 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目，差异仅 build 色与 chop／fall 动画 | 全族同协议 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L364-L425) |
| 生成者：桦树林落叶树 | builds 表按树当前 build 存 `fx`／`chopfx`（normal=绿叶，red／orange／yellow=秋色；barren 双 nil 不出叶）；`SpawnLeafFX` chop 分支出 chopfx、否则出 fx，按成长阶段调 y，砍伐叶另随机抬升 | 服务端；摇摆周期任务按 `MIN~MAX_SWAY_FX_FREQUENCY` 随机点名 fall 版 | [builds 表](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L45-L110)、[SpawnLeafFX](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L168-L197)、[周期](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L232-L236)、[砍伐回调](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L1371-L1371) |
| 生成者：越冬树（winter_deciduoustree 等） | 落叶过渡 11 帧后 `OnDropLeavesFX` 出 fall 版；`deciduous_onchop` 动态拼名 `.."_chop"` 出砍伐版，stage 4 大树 y−0.3 | 服务端；`DECIDUOUS_COLORFUL_FX` 仅 red/orange/yellow，`leaf_id` 为 nil 时回退 `green_leaves` | [色表](../../../raw/current/scripts/prefabs/winter_tree.lua#L806-L820)、[落叶](../../../raw/current/scripts/prefabs/winter_tree.lua#L861-L868)、[生成](../../../raw/current/scripts/prefabs/winter_tree.lua#L822-L827)、[砍伐](../../../raw/current/scripts/prefabs/winter_tree.lua#L944-L948) |
| 生成者：鸦年华广场树 | 鸦年华（carnival）广场树每次被敲必生成一项：`orange_leaves_chop` 或 `yellow_leaves_chop` 各 50%，y 抬升随机 | 服务端；不使用绿／红 | [调用](../../../raw/current/scripts/prefabs/carnival_plaza.lua#L43-L45) |
| 依赖声明 | deciduoustrees 的 prefabs 表声明四色＋chop＋purple 全套 | 仅声明；声明不等于生成 | [声明](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L23-L33) |
| 资源 | bank=`tree_leaf_fx`，build 分四色 `tree_leaf_fx_<color>`；资源注释标注绿叶=春夏、红橙黄=秋 | 动画时长与循环性未核验 | [资产注释](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L8-L11) |

## 典型接入

### 借用树叶表现

- 前提与选择：装饰性飘叶用 `*_leaves`，模拟砍伐用 `*_leaves_chop`；配色应与树色语义一致（绿=春夏、红橙黄=秋），越冬树叶色需自行映射 `leaf_id`。
- 生效与撤销：落点一次性动画，animover 自删；不跟随树体。
- 原版依据：落叶树点名 [SpawnLeafFX](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L168-L197)。

```lua
local fx = SpawnPrefab("yellow_leaves_chop")
fx.Transform:SetPosition(x, y + math.random() * 2, z)
```

- 调整与验证：树的摇摆周期由 `TUNING.MIN/MAX_SWAY_FX_FREQUENCY`（1～2 个时段）控制，模组改频需调 tuning 而非逐树计时（[摇摆任务](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L232-L236)）。

## 常见陷阱与限制

- **误区：四色对应四季各一**：资源注释只标绿叶为春夏、红／橙／黄同属秋季 build；冬季落叶树走 barren build（fx 双 nil）不生成叶特效。[builds](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L45-L110)
- **边界：越冬树的绿是回退值**：`DECIDUOUS_COLORFUL_FX` 只注册红／橙／黄三色，`leaf_id` 缺失时落叶／砍伐都回退 `green_leaves`；给越冬树配绿 leaf_id 并无独立绿色 colorful 项。[色表](../../../raw/current/scripts/prefabs/winter_tree.lua#L806-L820)
- **边界：purple_leaves 不在本族**：紫叶两条目用 `tree_monster_fx` bank（不同资源族），见 [[reference/prefabs/sugarwood_leaf_fx|特殊树叶与死亡烟]]；别按名称前缀并入。[紫叶条目](../../../raw/current/scripts/fx.lua#L392-L397)
- **误区：广场树会落全部四色**：鸦年华广场树每次必出其一，且只在橙／黄 chop 两名中二选一。[调用](../../../raw/current/scripts/prefabs/carnival_plaza.lua#L43-L45)
- **边界：动画是否循环未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。八条目配置、加载器协议、落叶树 builds／SpawnLeafFX／摇摆周期／砍伐回调、越冬树落叶与砍伐及色表回退、广场树分支逐条核验；.po 名称键定向核验无结果。未覆盖：树色随季节切换的完整状态机、动画资源内容、晚加入时序。
- 游戏运行：未运行；落叶频率与砍伐飞叶表现未实测。

返回 [[reference/prefabs/fx-table-world-index|通用表 FX·环境与物件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
