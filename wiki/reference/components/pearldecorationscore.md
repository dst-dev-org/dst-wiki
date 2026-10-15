---
title: "pearldecorationscore"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "蟹后小屋装饰评分组件：按占地网格周期统计 19 类装饰（地板/花卉/蜂箱/奖杯鱼/挂饰等）换算 0~100 分，推世界事件驱动解锁与商店配方。"
aliases: ["珍珠装饰评分", "蟹后小屋评分", "pearldecorationscore"]
tags: [component, building]
dst_build: "steam-buildid:26230496"
sources: ["components/pearldecorationscore.lua", "prefabs/hermithouse.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# pearldecorationscore

## 快速判断

- 职责与适用：挂在每座蟹后小屋（hermithouse）上（[装配](../../../raw/current/scripts/prefabs/hermithouse.lua#L852-L857)），仅在与珍珠 Friendship 解锁并启用后统计周围装饰对总分 0~100 的贡献；默认关闭，`Enable` 后才开始计数。
- 优先使用：查询得分 `GetScore`／`IsEnabled`（[L199-L201](../../../raw/current/scripts/components/pearldecorationscore.lua#L199-L201)、[L639-L641](../../../raw/current/scripts/components/pearldecorationscore.lua#L639-L641)）；判"实体是否算进这块地"用 `IsPointWithin`／`IsEntityWithin`（[L430-L469](../../../raw/current/scripts/components/pearldecorationscore.lua#L430-L469)，小屋搬家判定也在用，[消费](../../../raw/current/scripts/prefabs/hermithouse.lua#L669)）。
- 重要副作用：评分更新与启停推世界事件 `pearldecorationscore_updatescore`／`_updatestatus`／`_evaluatescores`；小屋、茶铺、寄居蟹都监听（[hermithouse](../../../raw/current/scripts/prefabs/hermithouse.lua#L299-L300)、[teashop](../../../raw/current/scripts/prefabs/hermitcrab_teashop.lua#L427)、[hermitcrab](../../../raw/current/scripts/prefabs/hermitcrab.lua#L1549)）。
- 修改边界：组件只读周围世界（FindEntities + 地块查询）不改装饰物；分值表与上限全在 `TUNING.HERMITCRAB_DECOR_*`（[tuning](../../../raw/current/scripts/tuning.lua#L9156-L9160)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `Enable(on_load)` / `Disable()` | — | 启动/停止周期评分；Enable 注册 FindEntities 缓存标签（全局一次）、监听 `onterraform` 与搬家/传送事件强制重算；Disable 解绑并清零重算（[L203-L286](../../../raw/current/scripts/components/pearldecorationscore.lua#L203-L286)）。 |
| `IsPointWithin(x, y, z, ignore_overhang)` | 支持 Vector3 | 判点是否在占地网格内；水上点向 0.166 出肩的相邻陆块回退（[L435-L469](../../../raw/current/scripts/components/pearldecorationscore.lua#L435-L469)）。 |
| `GetScore()` / `SetScore(score)` | 0~`HERMITCRAB_DECOR_MAX_SCORE`(100) | SetScore 变化时推 `pearldecorationscore_updatescore`（[L631-L641](../../../raw/current/scripts/components/pearldecorationscore.lua#L631-L641)）。 |
| `GetLastDecorScore(key)` / `...PercentToMax/Min(key)` / `GetDecorScoreLevel(key, reverse)` | key 为 `PEARL_DECORATION_TYPES` | 读上次评分的分项分值、占上限比例与 HIGH/MED/LOW 档（负向 key 用 min 阈值，[L596-L629](../../../raw/current/scripts/components/pearldecorationscore.lua#L596-L629)）。 |
| `decor_data`（分项规则表） | 按 key 设 max_score／min_score | 每类上限防刷分；`FISHING_MARKERS`/`SPAWNER`/`JUNK` 是负向 min_score（占用钓鱼点/刷怪点/垃圾倒扣）（[L34-L55](../../../raw/current/scripts/components/pearldecorationscore.lua#L34-L55)）。 |
| `decor_fns`（实体→分项判定） | `{key, fn(ent) → points}` | 逐实体顺序匹配首个命中的分项；`UNIQUE_DECORATION` 按 prefab 一次性计分（[L61-L196](../../../raw/current/scripts/components/pearldecorationscore.lua#L61-L196)、[唯一物判定](../../../raw/current/scripts/components/pearldecorationscore.lua#L424-L428)）。 |
| `OnUpdate(dt)` | `HERMITCRAB_DECOR_UPDATE_TIME` 10s | 强制或到期时重算：占地网格→半径内实体→分项累加→挂饰/宠物/全鱼收集/负向项→`SetScore`（[L470-L589](../../../raw/current/scripts/components/pearldecorationscore.lua#L470-L589)）。 |
| OnSave/OnLoad/LoadPostPass | `{enabled, score}` | Load 后补推 updatestatus/updatescore；移除组件走 Disable（[L643-L673](../../../raw/current/scripts/components/pearldecorationscore.lua#L643-L673)）。 |
| `FlagForConstructionRemoval()` | — | 房屋升级替换旧屋时标记，避免触发 Disable 副作用（[L675-L677](../../../raw/current/scripts/components/pearldecorationscore.lua#L675-L677)、[消费](../../../raw/current/scripts/prefabs/hermithouse.lua#L460-L467)）。 |

## 关键流程

### 一次评分（OnUpdate 到期）

1. `UpdateOccupiedGrid`：小屋所在图块坐标变化时用 `GetHermitCrabOccupiedGrid`（[componentutil](../../../raw/current/scripts/componentutil.lua#L1880)）重建占地网格，统计分数地块数并推 `ms_updatepearldecorationscore_tiles`（[L319-L340](../../../raw/current/scripts/components/pearldecorationscore.lua#L319-L340)）。
2. 半径内实体逐个匹配 `decor_fns`；没进网格的实体只有水树特殊按 11×11 图块覆盖数计分（[L501-L531](../../../raw/current/scripts/components/pearldecorationscore.lua#L501-L531)、[L352-L370](../../../raw/current/scripts/components/pearldecorationscore.lua#L352-L370)）。
3. 房屋本体：tier5（带 container）基础分＋容器内挂饰计数（wagstaff/冬日/普通）；寄居蟹的 critter 宠物；奖杯鱼集齐 16 种给大奖；钓鱼标记被陆地压住倒扣（[L534-L576](../../../raw/current/scripts/components/pearldecorationscore.lua#L534-L576)）。
4. 冬日盛宴期间 `_yule` 皮肤建筑/装饰逐项加成（[L477-L519](../../../raw/current/scripts/components/pearldecorationscore.lua#L477-L519)）。
5. 推 `pearldecorationscore_evaluatescores`（hermitcrab 在此触发对话/剧情，[L1549](../../../raw/current/scripts/prefabs/hermitcrab.lua#L1549)）后 `SetScore` 收口。

### 启用链路

小屋 5 级且 `friendlevels` 满级时 `Enable`（[hermithouse 解锁检查](../../../raw/current/scripts/prefabs/hermithouse.lua#L249-L250)）；小屋搬到装饰区或新屋建成时事件驱动再启用（[L517-L528](../../../raw/current/scripts/prefabs/hermithouse.lua#L517-L528)）。

## 常见陷阱与限制

- **评分范围是占地网格不是圆**：`IsPointWithin` 查 `tiles:GetDataAtPoint`，实体出网格只对水树有覆盖数豁免；把装饰放远一格就完全不计分（[L430-L469](../../../raw/current/scripts/components/pearldecorationscore.lua#L430-L469)）。
- **每类有上限**：`decor_data.max_score` 截断超配，模组加装饰种类必须同时补 TUNING 分值与判定 fn，否则不参与（[L34-L55](../../../raw/current/scripts/components/pearldecorationscore.lua#L34-L55)、[L502-L525](../../../raw/current/scripts/components/pearldecorationscore.lua#L502-L525)）。
- **FindEntities 缓存标签全局注册一次**：`REGISTERED_FIND_DECOR_TAGS` 是文件级单例，排除 FX/玩家/INLIMBO 等、只收 `inspectable` 与 `dock_woodpost`，不在其中的标签实体根本进不了扫描（[L212-L219](../../../raw/current/scripts/components/pearldecorationscore.lua#L212-L219)）。
- **更新与睡眠**：实体睡眠停更新、唤醒重启；`LongUpdate` 只推进倒计时不重算（[L590-L594](../../../raw/current/scripts/components/pearldecorationscore.lua#L590-L594)、[L679-L687](../../../raw/current/scripts/components/pearldecorationscore.lua#L679-L687)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、hermithouse 装配/解锁/升级/搬家消费点、teashop 与 hermitcrab 事件监听、componentutil 网格函数、tuning 段；未覆盖 19 类分值逐项数值与 hermitcrab 对话分支。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/hermitcrab_relocation_manager]]（搬家与钓鱼标记管理）。
