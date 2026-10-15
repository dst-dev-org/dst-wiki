---
title: "落叶堆生成与碎裂特效（leafpile_spawn_fx 族）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "万圣夜叶堆（leafpile）的通用表表现族：聚叶与碎裂两动画 × 默认／红／黄三 build 共 6 注册，由叶堆自身按 build 拼名在放置、生长与自毁／敲碎／清扫时生成。"
aliases: ["leafpile_spawn_fx", "leafpile_shatter_fx", "leafpile_red_spawn_fx", "leafpile_red_shatter_fx", "leafpile_yellow_spawn_fx", "leafpile_yellow_shatter_fx", "落叶堆特效", "叶堆碎裂特效", "落叶堆生成特效", "万圣夜落叶堆"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/fx.lua", "prefabs/leafpile.lua", "components/leafpilemanager.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 落叶堆生成与碎裂特效（leafpile_spawn_fx 族）

## 快速判断

- 职责与适用：万圣夜活动落叶堆（leafpile）的纯表现 FX：`*_spawn_fx` 为落点聚叶、`*_shatter_fx` 为碎裂（语义以叶堆调用点为准，见下），三 build 色 × 两动画共 6 注册；无组件、无玩法作用。[六条目](../../../raw/current/scripts/fx.lua#L3717-L3763)
- 优先使用：要"落叶聚拢／叶堆碎裂"表现 → 按色 `SpawnPrefab` 对应名；生成协议见 [[reference/mechanisms/generic-fx-factory#关键流程]]，叶堆实体与生成门控见 [[reference/components/leafpilemanager#关键接口]]。
- 重要副作用：条目自带叶响音效（spawn=`leaf_rustle_spawn_grow`，shatter=`leaf_rustle_sfx_os`）与 `SetFinalOffset(2)`；本地实体动画结束自删。[条目](../../../raw/current/scripts/fx.lua#L3717-L3763)
- 修改边界：生成点全在叶堆服务端（主模拟）代码；放置与碎裂两路径有 `not inst:IsAsleep()` 门，生长（`SetLeafSize` 非即时换档）无门、休眠叶堆到点仍出 fx；红黄仅换 build 贴图，动画／声效／fn 相同。[放置／碎裂](../../../raw/current/scripts/prefabs/leafpile.lua#L308-L341)、[生长](../../../raw/current/scripts/prefabs/leafpile.lua#L373-L413)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `leafpile_spawn_fx`／`leafpile_shatter_fx` | 通用表条目 → 加载器代理＋一帧后本地实体，协议同全表 | bank=`leafpile`、build=`leafpile`；spawn anim=`leaves_fall`，shatter anim=`leaves_shatter`；共用 `fn=FinalOffset2`。[两条](../../../raw/current/scripts/fx.lua#L3717-L3731) |
| `leafpile_red_spawn_fx`／`leafpile_red_shatter_fx` | 同上 | 仅 build 换 `leafpile_red_build`，anim／声／fn 不变。[两条](../../../raw/current/scripts/fx.lua#L3733-L3747) |
| `leafpile_yellow_spawn_fx`／`leafpile_yellow_shatter_fx` | 同上 | 仅 build 换 `leafpile_yellow_build`。[两条](../../../raw/current/scripts/fx.lua#L3749-L3763) |

定向检索简中 `languages/chinese_s.po` 无这 6 个注册名的显示名键；标题中文为已核验效果描述，非官方译名。三色对应叶堆 `SetBuildType` 在 `none`／`red`／`yellow` 中随机三选一的 `inst.build`（scrapbook 数据强制 `none`）。[选色](../../../raw/current/scripts/prefabs/leafpile.lua#L452-L462)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目：代理／落点复制／音效／回收全走公共协议 | 全族一致；落点式（生成者只 `SetPosition`，不父挂，不随叶堆移动） | [[reference/mechanisms/generic-fx-factory#关键流程]] |
| 生成者：放置（`OnPlaced`） | `leafpilemanager:TryToCreateLeafpileAtPoint` 生成并登记 `leafpile` 后调用：叶堆 `Hide` 并延迟 59 帧（`LEAVES_FALL_ANIM_SYNC`≈1.97 秒）显形播 `spawn_to_small`（注释标明与 fx 的 `leaves_fall` 对齐），同时出 spawn fx | 服务端；`build=="none"` 用默认名，否则拼 `"leafpile_"..build.."_spawn_fx"`；叶堆休眠时整体跳过 | [OnPlaced](../../../raw/current/scripts/prefabs/leafpile.lua#L308-L327)、[调用方](../../../raw/current/scripts/components/leafpilemanager.lua#L90-L100)、[同步常数](../../../raw/current/scripts/prefabs/leafpile.lua#L78-L78) |
| 生成者：生长（`SetLeafSize`） | timer 到点 `GrowLeafPile` 非即时换档（small→medium→large）时再出 spawn fx，59 帧后播转段动画 | 服务端；读档 `SetLeafSize(…, instant=true)` 不出 fx | [SetLeafSize](../../../raw/current/scripts/prefabs/leafpile.lua#L373-L413) |
| 生成者：碎裂（`PlayShatter`） | 三条移除路径先出 shatter fx 再 `Remove` 叶堆：瓦片失效自毁（`OnInvalidTile`，首睡／首醒 `OnEntity_Init` 与世界 `onterraform` 经 `CheckCurrentTile` 触发）、workable 一击完成（`OnWorkFinished`）、清扫（`OnSweepedFn`，先掉落战利品） | 服务端；拼名规则同上，休眠时不出 | [PlayShatter](../../../raw/current/scripts/prefabs/leafpile.lua#L329-L341)、[瓦片检查](../../../raw/current/scripts/prefabs/leafpile.lua#L290-L298)、[清扫](../../../raw/current/scripts/prefabs/leafpile.lua#L433-L437)、[敲碎](../../../raw/current/scripts/prefabs/leafpile.lua#L428-L431) |
| 组件：sweepable／workable | 清扫与敲碎是碎裂 fx 的两个玩家触发入口 | 叶堆主模拟挂载 | [[reference/components/sweepable#关键接口]]、[[reference/components/workable#关键接口]] |
| 关联 Prefab：`leafpile` | 宿主实体：三个生成者方法都挂在叶堆主模拟实体上，fx 与本体共用 bank | 服务端装配（`ismastersim` 分支后）；本体页未建 | 源码 [prefabs/leafpile.lua](../../../raw/current/scripts/prefabs/leafpile.lua#L480-L568)（叶堆实体页 Wiki 待建） |
| 依赖声明 | leafpile 的 `prefabs` 表声明全部 6 名，`assets` 声明三 build 动画 zip | 仅预载声明；声明不等于生成 | [声明](../../../raw/current/scripts/prefabs/leafpile.lua#L19-L32)、[资产](../../../raw/current/scripts/prefabs/leafpile.lua#L1-L5) |
| 资源 | bank=`leafpile` 与叶堆本体共用；build 为 `leafpile`／`leafpile_red_build`／`leafpile_yellow_build` | 动画时长与是否循环未核验 | [资产](../../../raw/current/scripts/prefabs/leafpile.lua#L1-L5) |

## 典型接入

### 借用叶堆碎裂表现

- 前提与选择：服务端主模拟 `SpawnPrefab`（非专服模拟经代理各自建本地表现）；需要"叶堆被破坏"语义时选 shatter 名，出生／生长感选 spawn 名。落点式一次性，无需清理。
- 生效与撤销：本地实体动画结束 animover 自删；条目音效已含。

```lua
local fx = SpawnPrefab("leafpile_shatter_fx")
fx.Transform:SetPosition(x, y, z) -- 落点式，动画播完自删
```

- 调整与验证：红／黄是叶堆随机变体色，借用时按场景选色；动画时长未核验，与自建动画衔接需实测。

## 常见陷阱与限制

- **误区：衰减／烧毁也播碎裂**：`Decay` 走叶堆自身 `decay_<size>` 动画后自删，`OnBurnt` 直接 `Remove`，都不调 `PlayShatter`；碎裂 fx 仅瓦片失效、敲碎、清扫三条路径。[Decay](../../../raw/current/scripts/prefabs/leafpile.lua#L348-L362)、[烧毁](../../../raw/current/scripts/prefabs/leafpile.lua#L272-L276)
- **边界：红黄注册名的生成点无静态文本**：`SpawnPrefab` 实参一律经 `"leafpile_"..inst.build.."_…"` 拼接，按完整注册名文本全域检索命中的只有 fx.lua 注册与 leafpile 声明表；声明不构成生成证据，反查须追拼接点（上文三生成者行）。
- **边界：两个 animover 别混淆**：fx 自删监听在工厂协议的本地表现实体上；`Decay` 给叶堆本体挂的 animover／entitysleep 是叶堆自身清理，与 fx 无关。[协议回收](../../../raw/current/scripts/prefabs/fx.lua#L98-L102)、[Decay 监听](../../../raw/current/scripts/prefabs/leafpile.lua#L357-L358)
- **边界：休眠门控只覆盖放置与碎裂**：放置（`OnPlaced`）与碎裂（`PlayShatter`）以 `not inst:IsAsleep()` 为门，远处休眠叶堆被瓦片失效／敲碎／清扫时服务端实体照常生成／移除，仅无 fx；生长（`SetLeafSize` 非 instant）经 timer 到点触发、无休眠检查，休眠叶堆到点换档仍会出 spawn fx。[门示例](../../../raw/current/scripts/prefabs/leafpile.lua#L308-L327)、[生长无门](../../../raw/current/scripts/prefabs/leafpile.lua#L373-L413)

## 验证记录

- 源码审阅：2026-11-01，steam-buildid:26335394。核验 fx.lua 六条目（3717-3763 行）、prefabs/leafpile.lua 全文（三调用点、拼名、放置与碎裂的休眠门及生长路径无门、59 帧同步常数、选色、依赖声明与 Decay／OnBurnt 边界）、components/leafpilemanager.lua 的 `TryToCreateLeafpileAtPoint`→`OnPlaced` 链、prefabs/fx.lua 回收段。检索范围：对六注册名与 `leafpile` 在 raw/current/scripts 全域（根级与 prefabs／components／stategraphs／brains）grep，六名命中仅 fx.lua 注册与 prefabs/leafpile.lua；`leafpile` 其余命中（prefablist.lua、scrapbook 列表、forest.lua 声明与挂载、tuning.lua、screens、skins_defs_data.lua）为叶堆实体条目或数据声明，非本族 fx 生成点。未核验：动画资源内容（时长／循环）、引擎休眠判定细节。
- 游戏运行：未运行。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；lua 示例通过 parser_probe 语法检查。

返回 [[reference/prefabs/fx-table-world-index|通用表 FX·环境与物件]]、[[reference/prefabs/index|Prefab 索引]]。
