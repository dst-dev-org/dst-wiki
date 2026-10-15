---
title: oceanfishinghook
created: 2026-10-21
updated: "2026-10-28"
type: component
summary: 海钓浮标上的鱼群吸引力评分与墙钟更新回调，负责"诱鱼"而不负责捕获
aliases: ["海钓浮标", "诱鱼 charm", "interest 评分", "bobber hook"]
tags: [component, callback, ai]
dst_build: "steam-buildid:26230496"
sources: ["components/oceanfishinghook.lua", "prefabs/oceanfishingbobber.lua", "brains/oceanfishbrain.lua", "components/oceanfishingrod.lua", "prefabs/oceanfishinglure.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# oceanfishinghook

## 快速判断

- 职责与适用：给"水面钩子"实体提供对鱼群的吸引力评分（`charm`→`interest`）、失去兴趣判定，以及供浮标做收线物理的墙钟回调；不含钓竿逻辑，也不执行捕获。
- 优先使用：鱼 AI 侧读 `TestInterest`／`UpdateInterestForFishable`（[挑选目标](../../../raw/current/scripts/brains/oceanfishbrain.lua#L96-L99)、[试探与咬钩](../../../raw/current/scripts/brains/oceanfishbrain.lua#L130-L137)）；浮标侧只调 `SetLureData`。
- 重要副作用：构造时给实体加 `fishinghook` 标签并启动 wall update；`SetLureData` 在 `lure_data.reel_charm ~= nil` 时再启动每帧 `OnUpdate`。
- 修改边界：服务端组件；`lure_data` 没有内建默认值，没调用 `SetLureData` 就被鱼 AI 查询会因 `nil` 索引报错；它不参与咬钩交接。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetLureData(lure_data, lure_fns)` | 参数表＋函数表（可空） | 写入 `lure_data`／`lure_fns`；`reel_charm ~= nil` 才 `StartUpdatingComponent` [SetLureData](../../../raw/current/scripts/components/oceanfishinghook.lua#L21-L28) |
| `lure_data` 字段 | `charm`(0..1)、`reel_charm`(-1..1)、`radius`、`timeofday{day,dusk,night}`、`style`、`weather{default,raining,snowing}`、`dist_max` | 只用于算分与范围；字段语义见 [tuning 注释](../../../raw/current/scripts/tuning.lua#L284-L294) |
| `_ClacCharm(fish)` | 数值（拼写按原版） | `(charm + reel_charm*reel_mod) × 保质百分比 × timeofday × style × weather × charm_mod_fn` [计算](../../../raw/current/scripts/components/oceanfishinghook.lua#L31-L46) |
| `UpdateInterestForFishable(fish)` | 数值 | 首次取 charm，之后每次追加 `charm*0.2`，累加无上限 [累积](../../../raw/current/scripts/components/oceanfishinghook.lua#L60-L71) |
| `HasLostInterest(fish)`／`SetLostInterest(fish)`／`ClearLostInterestList()` | 布尔／无返回／无返回 | 记录为 0 即"没兴趣"；`ClearLostInterestList` 在原版无调用方 [兴趣表](../../../raw/current/scripts/components/oceanfishinghook.lua#L48-L58) |
| `TestInterest(fish)` | 布尔 | 未失兴趣且 `fish:IsNear(inst, lure_data.radius)` [TestInterest](../../../raw/current/scripts/components/oceanfishinghook.lua#L83-L86) |
| `OnUpdate(dt)` | 逐帧（受 `reel_charm` 启动） | 实体速度平方 ≥0.1 时 `reel_mod=1`，否则按 `dt/2` 插值回 0 [OnUpdate](../../../raw/current/scripts/components/oceanfishinghook.lua#L73-L81) |
| `OnWallUpdate(dt)`／`onwallupdate` | 服务端暂停时跳过／函数字段 | 转发 `onwallupdate(inst, dt)`；浮标用它把钩子拖向钓竿 [OnWallUpdate](../../../raw/current/scripts/components/oceanfishinghook.lua#L101-L107) |
| `fishinghook` 标签 | 标签 | 构造时添加；浮标另在 pristine 段添加以获得同步 [标签](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L207) |

## 典型接入

### 用原版浮标做一个"只诱不钓"的鱼群吸引点

- 前提与选择：目标文件 `modmain.lua`；`SpawnPrefab("oceanfishingbobber_none_floater")` 命中已注册浮标（[prefab 表](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L347-L348)），该实体自带 `oceanfishable`＋`oceanfishinghook`（[装配](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L228-L235)）。服务端调用。
- 生效与撤销：`SetLureData` 后鱼 AI 的 `FindFoodAction` 才会用 `TestInterest` 选中它；`reel_charm` 非 nil 会顺带启动 `OnUpdate`。撤销＝`Remove()` 实体；它 `persists=false`，不挂竿时不会自动消失，重载即丢。
- 与捕获的关系：只有吸引力不够。`ACTIONS.EAT.fn` 要双方都有 `oceanfishable` 才把竿交出去（[EAT.fn](../../../raw/current/scripts/actions.lua#L806-L807)），本浮标没有竿，鱼只会来回试探而不会被钩住。
- 原版依据：[浮标落水时从钓竿取 lure 数据](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L52-L55)。

```lua
-- modmain.lua：放一只只负责诱鱼的原版浮标（服务端）
local function SpawnChumBuoy(x, z)
    local buoy = GLOBAL.SpawnPrefab("oceanfishingbobber_none_floater")
    buoy.Transform:SetPosition(x, 0, z)
    buoy.components.oceanfishinghook:SetLureData({
        charm = 0.4,
        reel_charm = 0.3,
        radius = 6,
        style = "spoon",
        timeofday = { day = 1, dusk = 0.5, night = 0.5 },
        weather = { default = 1, raining = 0.5, snowing = 0.5 },
    })
    return buoy
end
```

- 调整与验证：`radius` 是世界单位的试探范围，`charm` 决定每次试探的通过概率（`interest > 0.85` 或 `≥0.2` 且随机命中才咬），`reel_charm` 只在钩子被拖动时加分；必须放在海面，鱼 AI 会检查 `IsOceanAtPoint`。未运行，仅语法检查。

### 在 bobber 链上的位置

- `*_projectile`：只挂 `oceanfishable`（[projectile_fn](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L140-L141)），飞行中不参与诱鱼，落水后生成 `_floater` 并自我移除。
- `*_floater`：`oceanfishable`＋`oceanfishinghook`，pristine 段加 `fishinghook` 标签（[floater_fn](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L207-L235)），落水时从钓竿取 `lure_data`／`lure_fns`。
- 鱼本体 `oceanfish`：只有 `oceanfishable`，没有 hook；咬钩时从浮标接管钓竿，浮标被 `WasEatenByA` 移除（[[reference/components/oceanfishable]]）。

## 常见陷阱与限制

- 忘了 `SetLureData`：`TestInterest` 读 `lure_data.radius`、`_ClacCharm` 读 `charm`，`lure_data` 为 `nil` 时直接报错（[TestInterest](../../../raw/current/scripts/components/oceanfishinghook.lua#L83-L86)）。
- `charm`／`timeofday` 缺一不可：`timeofday` 为 `nil` 时该因子取 `0`，总分被清零，鱼 AI 走 `interest == 0` 分支放弃目标（[计算](../../../raw/current/scripts/components/oceanfishinghook.lua#L38-L42)）。
- `weather` 缺省不等于 1：回落到 `TUNING.OCEANFISHING_LURE_WEATHER_DEFAULT`，雨天／雪天是 0.5（[默认表](../../../raw/current/scripts/tuning.lua#L320)）。
- `style` 与鱼偏好不匹配会乘 0：鱼的 `fish_def.lures` 存在时，`style` 缺失或不匹配即清零，只能靠 `charm_mod_fn` 补（[heavy 诱饵](../../../raw/current/scripts/prefabs/oceanfishinglure.lua#L4-L7)）。
- 兴趣表只增不减：键是鱼的 GUID，`ClearLostInterestList` 在原版无调用方；长期存在的自定义钩子要自己清，否则记录随鱼数量累积。
- `OnUpdate` 依赖 `Physics`：按速度算 `reel_mod`，实体没有物理会逐帧报错（[OnUpdate](../../../raw/current/scripts/components/oceanfishinghook.lua#L73-L81)）。
- 标签要写两处：组件构造时加在服务端，浮标另在 `SetPristine` 前加一次用于同步；自定义实体若在客户端也需要该标签，必须在 pristine 段添加（[标签](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L207)）。
- 移除组件只撤标签：`OnRemoveFromEntity` 不碰更新循环，wall update 由 `EntityScript:RemoveComponent` 统一停（[entityscript](../../../raw/current/scripts/entityscript.lua#L670-L685)）。

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 3 处（tuning +1、actions +3，区间文本逐段比对一致）；结论修订：无（OCEANFISHING_LURE 天气默认表未变）。
- 依据：tuning 键级 hunk 地图＋变更清单命中 tuning.lua、actions.lua。

- 源码审阅：2026-10-21，steam-buildid:25643519，逐行读 `components/oceanfishinghook.lua`（109 行），并与 `oceanfishingbobber.lua`、`oceanfishbrain.lua`、`oceanfishingrod.lua`、`oceanfishinglure.lua`、`tuning.lua` 对照；覆盖 charm 公式、兴趣累积、reel_mod、wall update 转发、标签来源与移除路径，未覆盖鱼群 AI 的实际咬钩概率抽样。
- 游戏运行：未运行。
- 结构检查：validate_page.py 本页 0 error／0 warning（88 行，低于 180 行／14 KiB 上限）；validate_source_links.py --strict 0 error／0 warning，本页源码引用 24 处（唯一目标 21 个，覆盖 7 个源文件）。
