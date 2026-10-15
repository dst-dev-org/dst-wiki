---
title: "oceantrawler"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "海洋拖网船的捕鱼容器逻辑：放下后按周期捕获附近海洋鱼，休眠补偿计算，满载溢出鱼群逃逸，并联动诱饵、鱼群事件与生成加成。"
aliases: ["oceantrawler", "海洋拖网", "拖网船", "捕鱼网", "ocean trawler"]
tags: [component, item, loot]
dst_build: "steam-buildid:26230496"
sources: ["components/oceantrawler.lua", "prefabs/ocean_trawler.lua", "actions.lua", "components/schoolspawner.lua", "components/squidspawner.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# oceantrawler

## 快速判断

- 职责与适用：ocean_trawler 实体的组件（配合 container、timer），管理"放下→捕鱼→收起"循环；捕获结果写入容器（`<鱼>_inv` 变体），不是通用容器或渔竿逻辑。[安装](../../../raw/current/scripts/prefabs/ocean_trawler.lua#L162-L163)
- 优先使用：放/收/修走动作系统（LOWER/RAISE/FIX 动作调 `Lower()`/`Raise()`/`Fix()`）；状态查询用 `IsLowered()`/`HasFishEscaped()`/`HasCaughtItem()`。[动作](../../../raw/current/scripts/actions.lua#L5116-L5136)
- 重要副作用：放网时容器被锁（canbeopened=false）并换小地图图标；满载再捕鱼进 overflowfish，收网时弹出；累计溢出有概率整网逃逸（清空容器并置 `trawler_fish_escaped` 标签，需 Fix 恢复）。[放网](../../../raw/current/scripts/components/oceantrawler.lua#L113-L125) [溢出逃逸](../../../raw/current/scripts/components/oceantrawler.lua#L231-L259)
- 修改边界：唤醒补偿要求世界端有 `schoolspawner` 组件（决定鱼种），缺失时休眠捕获直接跳过；逃逸后只有 Fix 清除标记，放网期间不再捕获。[依赖](../../../raw/current/scripts/components/oceantrawler.lua#L313-L317) [逃逸门禁](../../../raw/current/scripts/components/oceantrawler.lua#L312

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| 捕获参数（构造字段） | number | `range=2.5`（搜鱼半径）、`checkperiod=0.75`（醒时检查周期）、`catchfishchance=0.125`、`sleepcatchfishchance=0.0625/SEG_TIME`、`baitcatchfishmodifier=2`、`overflowescapepercent=0.2`。均为可直接覆写的实例字段。[构造](../../../raw/current/scripts/components/oceantrawler.lua#L24-L49) |
| `Lower()`／`Raise()` | — | 切 `lowered`、`trawler_lowered` 标签、SG lower/raise 状态与小地图图标；放网关容器，收网解锁、弹出溢出鱼并刷新网外观。[放/收](../../../raw/current/scripts/components/oceantrawler.lua#L113-L125) [收](../../../raw/current/scripts/components/oceantrawler.lua#L190-L203) |
| `Fix()` | — | 清 `trawler_fish_escaped` 标记（修网动作），只改状态不清容器。[修复](../../../raw/current/scripts/components/oceantrawler.lua#L205-L209) |
| `GetBait(eater_prefab)` | → item 或 nil | 按 oceanfishdef 的鱼食性在容器里找匹配饵料（跳过 oceanfish 标签物品）；捕获概率乘 `baitcatchfishmodifier`，命中则消耗该饵。[饵匹配](../../../raw/current/scripts/components/oceantrawler.lua#L136-L160) |
| `AddFish(fishprefab)`（内部） | — | 未满则 SpawnPrefab(`<鱼>_inv`) 入容器；已满进 `overflowfish` 并跑溢出逃逸判定。醒时与休眠补偿两条路径共用它。[加鱼](../../../raw/current/scripts/components/oceantrawler.lua#L282-L303) |
| `SimulateCatchFish()` | 内部（醒时调用） | 休眠补偿：按已睡时长折算检查次数，概率里乘 `Map:CalcPercentOceanTilesAtPoint(…, 25)` 的海面占比与 16 距离内拖网船数量的均摊系数；命中时对附近鱼群推 `ms_shoalfishhooked`。[补偿](../../../raw/current/scripts/components/oceantrawler.lua#L308-L360) |
| `OnUpdate(dt)` | 内部 | 醒着且放网、未逃逸时找 2.5 内最近海洋鱼（活的、非 limbo/untrappable/库存），按 `catchfishchance × 饵倍率` 捕获：移除原鱼、入网、命中鱼群 home 时推 `ms_shoalfishhooked`。[醒时捕获](../../../raw/current/scripts/components/oceantrawler.lua#L389-L423) |
| `GetOceanTrawlerSpawnChanceModifier(spawnpoint)` | → number | 放网且满载时返回 `TUNING.OCEAN_TRAWLER_SPAWN_FISH_MODIFIER(2)`，否则 1；schoolspawner/squidspawner 用它提高附近海洋生物生成率。[加成](../../../raw/current/scripts/components/oceantrawler.lua#L128-L134) [schoolspawner](../../../raw/current/scripts/components/schoolspawner.lua#L28-L33) [squidspawner](../../../raw/current/scripts/components/squidspawner.lua#L34-L39) |
| `OnSave()`／`OnLoad(data)` | — | 存 lowered、已睡累计时长、overflowfish、fishescaped；OnLoad 恢复小地图图标与逃逸标签。[存读](../../../raw/current/scripts/components/oceantrawler.lua#L63-L95) |

### 网外观与休眠切换

网面符号（net_empty/medium/full/untied）由 `itemget`/`itemlose` 容器事件驱动，逃逸时换成 net_untied；实体休眠取消周期任务并记起始时间，唤醒跑一次 SimulateCatchFish 后恢复任务。[外观](../../../raw/current/scripts/components/oceantrawler.lua#L3-L22) [休眠](../../../raw/current/scripts/components/oceantrawler.lua#L362-L381)

## 典型接入

### 修改拖网船的捕获速度或饵料效果

- 前提与选择：对目标 trawler 实体（ mastersim）直接改实例字段即可，构造不缓存 TUNING；想全局调取改 `TUNING.OCEAN_TRAWLER_SPAWN_FISH_MODIFIER` 只影响"满载吸引生物"，不影响捕鱼概率。[字段覆写](../../../raw/current/scripts/components/oceantrawler.lua#L28-L42)
- 生效与撤销：改 checkperiod 需在放网状态下重新 StartUpdate 才生效（任务周期在启动时固定）；`Reset()` 会清空溢出与逃逸状态但保留容器内容。[任务启动](../../../raw/current/scripts/components/oceantrawler.lua#L222-L226) [Reset](../../../raw/current/scripts/components/oceantrawler.lua#L51-L61)
- 原版依据：动作层 LOWER/RAISE/FIX 是唯一官方入口，客户端表现为 SG 状态与音效；直接调 Lower/Raise 绕过动作仍会正常切状态。[动作绑定](../../../raw/current/scripts/actions.lua#L5116-L5136)

```lua
-- 服务器端：给世界现有拖网船提高醒时捕获概率与搜索半径
AddPrefabPostInit("ocean_trawler", function(inst)
    if not TheWorld.ismastersim then
        return
    end
    if inst.components.oceantrawler ~= nil then
        inst.components.oceantrawler.catchfishchance = 0.25 -- 默认 0.125
        inst.components.oceantrawler.range = 4 -- 默认 2.5
    end
end)
```

- 调整与验证：postinit 时组件已安装（在 prefab master 分支 AddComponent 之后）；捕获仍有随机性，验证建议配合 GetDebugString 或容器物品统计观察。[安装顺序](../../../raw/current/scripts/prefabs/ocean_trawler.lua#L162-L163)

## 常见陷阱与限制

- **休眠补偿需要 schoolspawner：** 无人值守挂机期望"醒来满网"的前提是世界端有 schoolspawner（森林世界默认安装）；组件缺失时 SimulateCatchFish 直接 return，不补鱼。[门禁](../../../raw/current/scripts/components/oceantrawler.lua#L314-L317) [安装](../../../raw/current/scripts/prefabs/forest.lua#L644
- **逃逸是"整网清空"不是漏几条：** 溢出判定命中即清空全部容器物品、溢出列表并标记逃逸；醒着逃逸还会把网内实体弹出。逃逸状态持续到 Fix，期间醒睡两条路径都不再捕鱼。[逃逸](../../../raw/current/scripts/components/oceantrawler.lua#L231-L259)
- **溢出鱼收网才现身：** 满载后的捕获进 `overflowfish`（prefab 名列表），收网时 SpawnPrefab 并随机弹飞；中途被打捞/烧毁不结算这些鱼。[弹出](../../../raw/current/scripts/components/oceantrawler.lua#L176-L188)
- **饵料判定按鱼食性而非"任何食物"：** GetBait 用 oceanfishdef 的 diet 匹配 `edible_*` 标签，且命中后饵被移除；放不匹配的饵只有占位作用。[食性匹配](../../../raw/current/scripts/components/oceantrawler.lua#L143-L157)
- **组件自带 timer：** 构造里 `AddComponent("timer")`；容器由 prefab 安装，WidgetSetup 为 4 槽（ui_cookpot_1x4），"满载"即 4 格。[构造](../../../raw/current/scripts/components/oceantrawler.lua#L45-L48) [容器定义](../../../raw/current/scripts/containers.lua#L1855-L1869)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 oceantrawler.lua，核验放/收/修状态机、醒时捕获与休眠补偿、饵料匹配、溢出与逃逸、生成加成消费与存读；核对 ocean_trawler 安装、动作绑定与 schoolspawner/squidspawner 消费。被注释的 CheckForMalbatross 死代码未纳入结论。
- 结构检查：本页通过 validate_page.py 与 validate_source_links.py（严格）；Lua 示例经 parser_probe 语法检查。
- 游戏运行：未运行；捕获概率体感、满载吸引与逃逸触发未实测。

相关：[[reference/components/schoolspawner]]、[[reference/components/oceanfishingrod]]。

## 相关链接

- [[reference/components/schoolspawner]]：休眠补偿的鱼种来源与生成加成消费方。
- [[reference/components/squidspawner]]：满载拖网提高乌贼生成率。
