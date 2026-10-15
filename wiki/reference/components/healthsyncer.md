---
title: "healthsyncer"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "把实体生命百分比和持续上升／下降趋势同步给客户端，供没有 replica.health 的 HUD 读取；不修改生命数据。"
aliases: ["生命百分比同步", "生命趋势同步", "船血条同步", "healthsyncer"]
tags: [component, network-sync, ui, server-client, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/healthsyncer.lua", "components/health.lua", "components/health_replica.lua", "components/sleepingbag.lua", "prefabs/boat.lua", "prefabs/yotd_boats.lua", "prefabs/player_common.lua", "prefabs/player_classified.lua", "widgets/boatmeter.lua", "widgets/teammatehealthbadge.lua", "widgets/teamstatusbars.lua", "widgets/statusdisplays.lua", "widgets/healthbadge.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# healthsyncer

## 快速判断

- 职责与适用：把宿主 `healthdelta` 的百分比写进 `net_float`，把"持续掉血／回血"归为 `net_tinybyte` 状态，客户端再转成 `clienthealthdirty`／`clienthealthstatusdirty`。给客户端读不到 `health` 组件的实体（船）和队友 HUD 提供只读数据。[构造](../../../raw/current/scripts/components/healthsyncer.lua#L21-L77) [状态计算](../../../raw/current/scripts/components/healthsyncer.lua#L97-L123)
- 优先使用：读比例用 `GetPercent()`，读趋势箭头用 `GetOverTime()`；需要绝对数值时由宿主另给 `max_health`。玩家自身用 `replica.health`＋`healthbadge`，宠物显示用 [[reference/components/pethealthbar]]，改生命用 [[reference/components/health]]。
- 重要副作用：服务端每模拟帧 `OnUpdate` 推断趋势；构造期登记腐蚀／回血来源并监听 debuff 的 `onremove`；`_healthpct` 构造时默认 1，首次同步前客户端读到满血。[默认值](../../../raw/current/scripts/components/healthsyncer.lua#L24-L26) [更新注册](../../../raw/current/scripts/components/healthsyncer.lua#L71)
- 修改边界：只在服务端写网络值；组件内部按 `TheWorld.ismastersim` 分派，需双端安装；不写入 `health`，没有 `OnRemoveFromEntity`，动态移除组件不安全（见陷阱）。[分派](../../../raw/current/scripts/components/healthsyncer.lua#L28-L76)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `_healthpct`／`GetPercent()` | `net_float`，0..1 比例，默认 1 | 服务端在 `healthdelta` 值变化时写入，客户端读；`InitServer` 在零延迟用 `health:GetPercent()` 校正初值，无 `health` 时取 1。是百分比，不是当前／最大生命值。[字段](../../../raw/current/scripts/components/healthsyncer.lua#L24-L26) [初始化](../../../raw/current/scripts/components/healthsyncer.lua#L17-L19) [读取](../../../raw/current/scripts/components/healthsyncer.lua#L82-L84) |
| `_status`／`GetOverTime()` | `net_tinybyte`（0..4）→ 返回 -2..2 | 服务端只写 0／1／2／3；返回映射：`<=0` 或 `>=5` 为 0，1／2 原样，3／4 减 5。是趋势指示，不是回血速率或预计净值。[映射](../../../raw/current/scripts/components/healthsyncer.lua#L86-L92) [写入](../../../raw/current/scripts/components/healthsyncer.lua#L113-L122) |
| `OnUpdate(dt)` | 服务端更新回调 | 每模拟帧判断 down（冻结／过热／饥饿／火伤／腐蚀任一）与 up（睡眠回复／回血来源／装备 `regen` 标签，且 `health:IsHurt()`；睡眠标记见 [player_classified](../../../raw/current/scripts/prefabs/player_classified.lua#L1365)、[床具写入](../../../raw/current/scripts/components/sleepingbag.lua#L1-L10)）；down 优先，只有值变化才写网络值并推事件。[更新](../../../raw/current/scripts/components/healthsyncer.lua#L97-L123) |
| `max_health` | 普通 Lua 字段，不联网 | 组件自身不读也不写；原版船在共同初始化设为 `data.max_health or TUNING.BOAT.HEALTH`，供 `boatmeter` 把百分比换算成数值。客户端要一致必须双端各自赋同一来源。[船](../../../raw/current/scripts/prefabs/boat.lua#L416-L417) [消费](../../../raw/current/scripts/widgets/boatmeter.lua#L113-L125) |
| `clienthealthdirty`／`clienthealthstatusdirty` | 客户端实体事件 | netvar dirty 回调在客户端转发，主机端写值时也会直接推同一事件；`clienthealthdirty` 的 payload 只有 `percent`，状态事件无 payload。[回调](../../../raw/current/scripts/components/healthsyncer.lua#L1-L7) [客户端接线](../../../raw/current/scripts/components/healthsyncer.lua#L73-L76) |
| 腐蚀／回血来源登记 | 服务端集合 `corrosives`／`hots`／`small_hots` | 监听宿主实体的 `startcorrosivedebuff`／`starthealthregen`／`startsmallhealthregen`／`stopsmallhealthregen`，并在 debuff 实体 `onremove` 时清理；这些表只在服务端存在，客户端读不到。[登记](../../../raw/current/scripts/components/healthsyncer.lua#L32-L69) |

## 典型接入

### 给自建生物补齐客户端可读的生命百分比与趋势

- 前提与选择：目标实体服务端已有 `health`；用 `AddPrefabPostInit` 双端安装，组件自行按端分派；`max_health` 用双端同源常量。原版船先在共同初始化加组件，再由 `boatmeter` 轮询百分比。[postinit](../../../raw/current/scripts/mainfunctions.lua#L373-L403)
- 生效与撤销：服务端 `healthdelta` 与每帧 `OnUpdate` 驱动网络值；读取方轮询 `GetPercent`／`GetOverTime`，或监听上表的两个 client 事件（`boatmeter` 是轮询，`teammatehealthbadge` 是事件）。读取方的监听要自行解绑；不要动态 `RemoveComponent`（见陷阱）。
- 原版依据：船 `prefabs/boat.lua` 共同初始化加组件与 `max_health`；lavaarena 队友条由 `player_common` 仅在该模式给玩家加组件、`teammatehealthbadge` 监听事件。[船安装](../../../raw/current/scripts/prefabs/boat.lua#L416-L417) [玩家安装](../../../raw/current/scripts/prefabs/player_common.lua#L2529-L2531) [队友条](../../../raw/current/scripts/widgets/teammatehealthbadge.lua#L70-L82)

```lua
-- modmain.lua：双端安装；max_health 不联网，双端必须用同一来源保证一致
AddPrefabPostInit("spider_warrior", function(inst)
    if inst.components.healthsyncer ~= nil then
        return
    end
    local syncer = inst:AddComponent("healthsyncer")
    syncer.max_health = GLOBAL.TUNING.SPIDER_WARRIOR_HEALTH
end)
```

- 调整与验证：`max_health` 只用于把比例换算成绝对值，组件不校验；示例只含安装，消费者接线按上一条另写。未运行。

## 常见陷阱与限制

- **趋势当净值或回血速率**：down 只看有限状态（冻结／过热／饥饿／火伤／腐蚀），up 还要求 `health:IsHurt()`；不累计 `DoDelta` 数值，满血时不会显示上升箭头。[更新](../../../raw/current/scripts/components/healthsyncer.lua#L97-L123)
- **所有下降都映射为大下降**：`OnUpdate` 只产生状态 3（→ -2），状态 4（→ -1"小下降"）在本组件没有写入点，别按 -1 设计分支。[写入](../../../raw/current/scripts/components/healthsyncer.lua#L113-L117)
- **普通玩家身上没有这个组件**：`player_common` 只在 `lavaarena` 模式添加；不要把 `components.healthsyncer ~= nil` 当"能否读生命"的判断，普通玩家用 `replica.health`。[玩家安装](../../../raw/current/scripts/prefabs/player_common.lua#L2529-L2531) [复制组件](../../../raw/current/scripts/components/health_replica.lua#L1-L9)
- **`max_health` 不是同步字段**：客户端与其他玩家看不到服务端赋的值；只在服务端设置会让 HUD 的绝对值错乱。船的做法是在共同初始化用同一 `data` 赋值。[船](../../../raw/current/scripts/prefabs/boat.lua#L416-L417)
- **首次显示可能是满血**：`_healthpct` 构造即 `set(1)`，客户端在零延迟就转发一次 dirty；`boatmeter` 的 `Enable` 又立即刷新，所以进入时先看到 100%，等同步值到达才回落。[默认值](../../../raw/current/scripts/components/healthsyncer.lua#L24-L26) [立即刷新](../../../raw/current/scripts/widgets/boatmeter.lua#L74-L77)
- **动态移除组件不安全**：没有 `OnRemoveFromEntity`；`RemoveComponent` 只把更新回调拆下，不注销 `healthdelta`／dirty 监听。残留的 `OnHealthDelta` 随后读 `inst.components.healthsyncer`（已为 nil）并报错，客户端 `OnHealthPctDirty` 同样报错；只有不读组件的 `OnStatusDirty` 仍会推 `clienthealthstatusdirty`。撤销请删除宿主实体（`EntityScript:Remove` 清监听），停用请另设开关。[回调取值](../../../raw/current/scripts/components/healthsyncer.lua#L1-L15) [移除](../../../raw/current/scripts/entityscript.lua#L670-L684) [宿主删除](../../../raw/current/scripts/entityscript.lua#L1739-L1743)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 4 处（player_classified 睡眠标记、player_common lavaarena 安装、mainfunctions postinit、entityscript OnRemoveEntity 段）；无结论修订，仅锚点迁移——依据：全局枢纽 diff 审阅＋变更清单命中 player_common.lua、player_classified.lua、widgets/healthbadge.lua、entityscript.lua、mainfunctions.lua，healthsyncer 自身、boat／yotd_boats 与各 widget 未变；healthbadge 本版改动（OnUpdate 附近）不在本页引用面。
- 源码审阅：2026-10-21，Steam buildid 25643519；完整阅读 `components/healthsyncer.lua`（125 行）与三个安装点（`boat.lua`、`yotd_boats.lua`、`player_common.lua`，后者仅 lavaarena），核验 `healthdelta`／`IsHurt` 生产、睡眠回复与腐蚀／回血事件来源、两个消费者（`boatmeter` 轮询、`teammatehealthbadge` 事件）及 `teamstatusbars` 组队循环、`statusdisplays` 平台开关、`RemoveComponent` 与更新停止。未覆盖 YOTD 船变体差异、lavaarena 完整队伍流程与全部 HUD 布局。
- 结构检查：`validate_page.py` 无 issue；`validate_source_links.py --strict` 为 0 error／0 warning（本页 32 处源码引用）；示例 335 B 经 `parser_probe.exe` 完成 Lua 5.1 syntax-only。
- 游戏运行：未运行；客户端百分比／箭头表现、多人同步、`max_health` 双端一致性、移除组件后的行为均未覆盖。

相关：[[reference/components/health]]；[[reference/components/healthbar]]；[[reference/components/pethealthbar]]；[[reference/components/healthtrigger]]。
