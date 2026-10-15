---
title: "冬季盛宴餐桌食物枯竭特效（winters_feast_depletefood 等）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "冬季盛宴餐桌开餐期间与食物耗尽时的一次性枯竭表现：depletefood 为每名进食者 30% 概率的随机闪粒，food_depleted 为食物耐久耗尽的终局 burst；两名仅餐桌一处生成。"
aliases: ["winters_feast_depletefood", "winters_feast_food_depleted", "冬季盛宴食物枯竭特效", "盛宴餐桌特效", "食物耗尽特效", "winters feast 枯竭特效"]
tags: [prefab, fx, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/fx.lua", "prefabs/table_winters_feast.lua", "components/wintersfeasttable.lua", "components/feasts.lua", "stategraphs/SGwilson.lua", "prefabs/wintersfeastcookedfoods.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 冬季盛宴餐桌食物枯竭特效（winters_feast_depletefood 等）

## 快速判断

- 职责与适用：`fx.lua` 通用表两条一次性动画特效，都是冬季盛宴餐桌（`table_winters_feast`）盛宴流程的纯视觉表现，落点式生成在餐桌位置。`winters_feast_depletefood` 是开餐过程特效：每次食物消耗按进食人数掷概率；`winters_feast_food_depleted` 是终局特效：食物耐久耗尽时播一次 burst。两名语义分界以生成方代码为准，不是同一过程的两段名。选 simple 档：单一消费链、各注册差异仅一两处。
- 优先使用：查"盛宴餐桌食物枯竭／耗尽什么表现"读本页；代理与本地表现分工、`animover` 自删、专服无表现等公共协议只在 [[reference/mechanisms/generic-fx-factory#关键流程|通用 FX 工厂·关键流程]] 维护，本页不重复。
- 重要副作用：无玩法作用，条目无生效 sound——food_depleted 的 sound 被注释（fx.lua L2139），终局音效 `wintersfeast2019/winters_feast/table/fx` 由餐桌自身 SoundEmitter 播放（table_winters_feast.lua L188）。
- 修改边界：po 无显示名键，标题中文为已核验效果描述、非官方译名；触发前提是桌上有带 `finiteuses` 的盛宴食物。普通食物在投食阶段先被餐桌拒绝（`RefuseFood` bump／弹出，`canfeast=false`），不会进入消耗链；若直接调用组件入口，`DepleteFood` 对无 finiteuses 的食物也静默返回，两特效都不出。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `winters_feast_depletefood` | 通用表条目 → MakeFx：bank/build `winters_feast_table_fx`、anim `"1"`（占位）、`bloom = true`；fn 内 `SetLightOverride(1)`、`PlayAnimation(math.random(1, 5))` 覆盖初始动画、`SetFinalOffset(3)` | 开餐过程特效，实际播 1~5 号动画之一（数值实参由引擎按动画名处理，脚本侧不可证）；随机动画＋bloom 是与另一名的表现差异。[条目](../../../raw/current/scripts/fx.lua#L2123-L2132) |
| `winters_feast_food_depleted` | 同表同 bank/build，anim `"burst"`；fn 仅 `SetLightOverride(1)`＋`SetFinalOffset(3)`，无 bloom；`--sound = ,` 注释为空 | 耗尽终局特效，单次 burst。[条目](../../../raw/current/scripts/fx.lua#L2135-L2143) |

资源按工厂规则声明 `anim/winters_feast_table_fx.zip`；bank 内 1~5 号与 burst 动画的内容、时长在动画数据里，脚本不可见。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：餐桌盛宴回调 | 唯一生成文件：`ondepletefoodfn`→`spawnfx` 每名进食者 `math.random() < 0.3` 时延迟 `math.random()*1/3` 秒落点生成 depletefood；`onfinishfoodfn` 延迟 `math.random()*0.5` 秒清桌并生成 food_depleted | 主模拟 SpawnPrefab，`SetPosition` 取餐桌世界坐标，不父挂；专服只有代理 | [spawnfx](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L142-L146)、[OnFinishFood](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L169-L190)；宿主 prefab 页 Wiki 待建（prefabs/table_winters_feast.lua） |
| 触发链：wintersfeasttable | 世界组件 feasts 每 1/3 秒盛宴 tick 对每桌 `DepleteFood(#feasters)`：`finiteuses:Use()` 后剩余 >0 调 ondepletefoodfn、归零调 onfinishfoodfn；人数按桌组统计 | 玩家以 WINTERSFEAST_FEAST 动作开餐推 `feasterstarted` 才启动 tick | 组件消耗链 [[reference/components/wintersfeasttable#典型接入]]、盛宴 tick [[reference/components/feasts#关键接口]]、[SG 入口](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1571-L1577) |
| 前提：盛宴食物 finiteuses | 可触发对象是 wintersfeastcookedfoods 生成的盛宴食物（uses 缺省 `TUNING.WINTERSFEASTBUFF.EATTIME`）；其 `SetOnFinished(Remove)` 被注释，删除改由餐桌负责 | 非盛宴食物先在餐桌投食阶段被拒绝（`RefuseFood`；`canfeast=false`）；直接进入组件入口时，无 finiteuses 也静默跳过 | [uses 配置](../../../raw/current/scripts/prefabs/wintersfeastcookedfoods.lua#L51-L55)、[投食分界](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L208-L229)、[RefuseFood](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L121-L133) |
| 公共工厂 | MakeFx 代理＋本地表现协议：一帧延迟、`animover` 自删、代理 1 秒回收、`persists=false` | 非专服各机建表现 | [[reference/mechanisms/generic-fx-factory#关键流程]] |

## 典型接入

### 复刻餐桌枯竭表现

- 前提与选择：主模拟 `SpawnPrefab`＋落点，与原版生成者一致；表现由通用工厂自动分发，宿主侧无需清理。
- 生效与撤销：本地表现一帧后出现、动画播完 `animover` 自删；代理 1 秒后回收。
- 原版依据：[table_winters_feast.lua:144](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L144)、[:186](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L186)。

```lua
local fx = SpawnPrefab("winters_feast_depletefood")       -- 开餐闪粒（随机 1~5 号动画）
fx.Transform:SetPosition(table_inst.Transform:GetWorldPosition())
local burst = SpawnPrefab("winters_feast_food_depleted")  -- 耗尽终局 burst
burst.Transform:SetPosition(table_inst.Transform:GetWorldPosition())
```

- 调整与验证：概率、延迟与人数统计都在生成方回调内，FX 条目只承载表现；改行为需覆盖 `ondepletefoodfn`／`onfinishfoodfn` 或自建同构回调。

## 常见陷阱与限制

- **误区：两名是同一过程的两段**：depletefood 是每次消耗按人数逐名掷 30% 的过程特效（一桌多人同 tick 可出多枚），food_depleted 是耐久归零那一次的终局特效；分界在 [DepleteFood 的 uses 分支](../../../raw/current/scripts/components/wintersfeasttable.lua#L65-L81)，不按名称推断。
- **边界：专服与普通食物都不出特效**：专服无本地表现；非盛宴食物通常先在投食阶段被 `RefuseFood`，若绕过宿主直接进入组件入口，无 finiteuses 时 `DepleteFood` 静默返回，两名都不生成。[守卫](../../../raw/current/scripts/components/wintersfeasttable.lua#L68)、[投食分界](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L214-L229)
- **边界：终局音效不属于本 FX**：条目 sound 注释为空，`wintersfeast2019/winters_feast/table/fx` 由餐桌 SoundEmitter 在清桌延迟任务内播放；替换 FX 不影响该音效。[音效](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L188)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。fx.lua 两条目逐字段核对（L2123-2144，含注释掉的 sound）；对 `depletefood`／`food_depleted` 子串在 scripts 根级、prefabs/、components/、stategraphs/ 全域 rg（含动态拼名模式 `winters_feast.*deplete` 等），命中仅 fx.lua（注册）、table_winters_feast.lua（生成 2 处）、wintersfeasttable.lua（`DepleteFood` 方法名，非生成）；触发链 wintersfeasttable／feasts／SGwilson／wintersfeastcookedfoods 已读上下文；chinese_s.po 定向检索两注册名与 bank 均无显示名键。动画内容与时长、客户端本地 Remove 的引擎行为未覆盖，公共工厂面见通用 FX 工厂页验证记录。
- 游戏运行：未运行。
- 结构检查：validate_page 与 validate_source_links --strict 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-event-index|节庆、小游戏与事件模式]]、[[reference/prefabs/index|Prefab 索引]]。
