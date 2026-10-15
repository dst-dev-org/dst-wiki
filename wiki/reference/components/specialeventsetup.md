---
title: "specialeventsetup"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级节日事件装配器：读档对比前后激活事件，为万圣节补发饰品、为猫猫年布置捉迷藏基特猫，并推送事件装配钩子。"
aliases: ["specialeventsetup", "节日事件", "万圣节饰品补发", "猫猫年活动"]
tags: [component, lifecycle, worldgen]
dst_build: "steam-buildid:26230496"
sources: ["components/specialeventsetup.lua", "constants.lua", "prefabs/mound.lua", "prefabs/forest.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# specialeventsetup

## 快速判断

- 职责与适用：森林世界主模拟组件（[安装](../../../raw/current/scripts/prefabs/forest.lua#L671)），在世界初始化（OnPostInit）时对比“存档时的激活事件”与“本版本全局激活事件”，新增的做装配、消失的做关闭；当前版本全局激活事件硬编码在常量里（本 build 为 `SPECIAL_EVENTS.HALLOWED_NIGHTS`）。[断言](../../../raw/current/scripts/components/specialeventsetup.lua#L9) [OnPostInit](../../../raw/current/scripts/components/specialeventsetup.lua#L303-L322) [全局事件](../../../raw/current/scripts/constants.lua#L858-L859)
- 优先使用：模组监听世界事件 `ms_setupspecialevent`/`ms_shutdownspecialevent` 接入自己的节日（装配时 event 参数为事件枚举）；万圣节挖坟蝙蝠概率读公开字段 `halloween_bat_grave_spawn_chance`。[模组钩子](../../../raw/current/scripts/components/specialeventsetup.lua#L283-L284) [坟场消费](../../../raw/current/scripts/prefabs/mound.lua#L100-L113)
- 重要副作用：装配会直接 SpawnPrefab 修改世界（饰品散布、基特猫藏匿点），一次性执行不做增量；关闭路径目前只推事件，万圣节残留饰品清理是 TODO。[装配](../../../raw/current/scripts/components/specialeventsetup.lua#L274-L285) [关闭 TODO](../../../raw/current/scripts/components/specialeventsetup.lua#L290-L292)
- 修改边界：无 OnRemoveFromEntity；有 OnSave/OnLoad（蝙蝠概率、prev_event/prev_extra_events，兼容旧 halloweentrinkets 字段）。[存读](../../../raw/current/scripts/components/specialeventsetup.lua#L328-L345)

## 关键接口

| 接口 | 条件、作用与证据 |
|---|---|
| `OnPostInit()` | 先调 `oceanfishdef.SpecialEventSetup()`（鱼类事件数据），再对比 prev 与当前事件集合做装配/关闭。世界初始化后自动执行一次。[实现](../../../raw/current/scripts/components/specialeventsetup.lua#L303-L322) [事件集合工具](../../../raw/current/scripts/constants.lua#L1114) |
| `SetupNewSpecialEvent(event)` | `YOT_CATCOON`→基特猫捉迷藏布置；`HALLOWED_NIGHTS`→饰品补发；其他事件只推钩子。无返回值。[实现](../../../raw/current/scripts/components/specialeventsetup.lua#L274-L285) |
| `ShutdownPrevSpecialEvent(event)` | 目前无内置清理（万圣节 TODO），仅推 `ms_shutdownspecialevent`。[实现](../../../raw/current/scripts/components/specialeventsetup.lua#L287-L296) |
| `ms_setupspecialevent`／`ms_shutdownspecialevent`（世界事件） | 模组接入点：data 为事件枚举。原版脚本内无消费者。[推送](../../../raw/current/scripts/components/specialeventsetup.lua#L284) [关闭推送](../../../raw/current/scripts/components/specialeventsetup.lua#L295) |
| `halloween_bat_grave_spawn_chance`（公开字段） | 挖坟蝙蝠累计概率：mound 每次挖坟失败累加 0.1～0.2，出蝙蝠即清零；存档保存。[字段](../../../raw/current/scripts/components/specialeventsetup.lua#L27) [消费](../../../raw/current/scripts/prefabs/mound.lua#L100-L113) |

### 万圣节饰品补发

判定“今年没发过”后，把 `trinket_32..trinket_46`（各 2 份）加 6 个万圣节挂饰洗牌，遍历 topology 节点每 4 个取 1 个、每节点取最多 5 个随机点，在地面且 1 半径无实体的位置逐个放置（[实现](../../../raw/current/scripts/components/specialeventsetup.lua#L57-L94)，[饰品编号](../../../raw/current/scripts/constants.lua#L1608-L1609)）。旧档兼容：先扫描世界中现存饰品数量（>15 个视为已发过），旧 `halloweentrinkets` 年份标记只用于触发迁移（[迁移](../../../raw/current/scripts/components/specialeventsetup.lua#L34-L58)）。

### 猫猫年基特猫布置

8 种 kitcoon 变体各有专属生物群系与藏匿点数据（[数据表](../../../raw/current/scripts/components/specialeventsetup.lua#L236-L246)）；先经 `ms_collectallkitcoons` 收编现存个体避免重复生成，再逐只按“群系内合格藏匿点 → 补生成 fallback prefab → 应急小物（rocks/twigs/cutgrass/log）→ 全图任意点 → 出生点附近”的兜底链安置，全失败则直接把猫放出生点旁（[兜底链](../../../raw/current/scripts/components/specialeventsetup.lua#L98-L225)）。数据表对模组无钩子（源码 TODO 明示，[注释](../../../raw/current/scripts/components/specialeventsetup.lua#L248)）。

## 典型接入

### 模组节日接入装配/关闭钩子

- 前提与选择：服务端、森林世界（洞穴不装本组件）。监听世界事件即可在原版事件启停时机执行自己的装配；事件枚举自 `SPECIAL_EVENTS`。注意组件只对 `YOT_CATCOON`/`HALLOWED_NIGHTS` 有内置装配，其余枚举仅推事件——模组内容全部自行处理。[分发](../../../raw/current/scripts/components/specialeventsetup.lua#L277-L281)
- 生效与撤销：OnPostInit 每次世界加载各触发一次（新增装配、移除关闭）；世界运行中不会重复触发。[触发时机](../../../raw/current/scripts/components/specialeventsetup.lua#L309-L321)

```lua
-- modmain.lua：接入模组节日事件的装配与关闭
AddPrefabPostInit("world", function(inst)
    if not TheWorld.ismastersim then
        return
    end
    inst:ListenForEvent("ms_setupspecialevent", function(world, event)
        if event == SPECIAL_EVENTS.HALLOWED_NIGHTS then
            -- 自己的万圣节内容装配
        end
    end)
    inst:ListenForEvent("ms_shutdownspecialevent", function(world, event)
        -- 对应清理
    end)
end)
```

- 调整与验证：`SPECIAL_EVENTS` 枚举见 constants；验证改 `WORLD_SPECIAL_EVENT` 常量或用旧档切换事件观察装配日志（[枚举](../../../raw/current/scripts/constants.lua#L858)）。

## 常见陷阱与限制

- **钩子只在装配时机推一次：** 世界加载完成后新增事件不会实时触发（对比逻辑只在 OnPostInit）；想运行中切换需自己处理实体与再触发。[单次触发](../../../raw/current/scripts/components/specialeventsetup.lua#L303-L322)
- **饰品补发可能漏发：** 放置要求节点 `i%4==0` 且点上方地面、1 半径无实体；世界拥挤时 count 会小于预期，且不重试。[放置条件](../../../raw/current/scripts/components/specialeventsetup.lua#L72-L91)
- **猫猫年数据表无模组钩子：** `kitten_hiding_data` 是函数局部表，TODO 注释明确未提供扩展点；新增 kitcoon 变体需整体接管 `_SetupYearOfTheCatcoon` 或补发生成的猫。[TODO](../../../raw/current/scripts/components/specialeventsetup.lua#L248)
- **当前激活事件由编译期常量决定：** `WORLD_SPECIAL_EVENT` 写死在 constants.lua，运行时改它不影响已初始化的组件对比结果（OnSave 又以它为准回写）；跨年维护需跟进该常量。[常量](../../../raw/current/scripts/constants.lua#L858-L859) [回写](../../../raw/current/scripts/components/specialeventsetup.lua#L332-L333)
- **prev_event 迁移链路有三代格式：** halloweentrinkets（年份字符串）→ prev_event/prev_extra_events；混用旧档时按迁移分支执行，新模组不要写 halloweentrinkets。[迁移](../../../raw/current/scripts/components/specialeventsetup.lua#L337-L345)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 specialeventsetup.lua，核验事件对比流程、万圣节与猫猫年装配实现、存读迁移、钩子推送与 mound 消费点；核对 constants.lua 全局事件与饰品编号常量。未运行游戏；各节日实体的具体玩法归各自页面。

相关：[[reference/components/carnivalevent]]、[[reference/components/regrowthmanager]]。
