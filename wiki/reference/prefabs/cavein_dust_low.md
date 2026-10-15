---
title: "洞穴落石尘土与拾荒疯猪碎装表现族（cavein_dust_low 等 9 注册）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "两个共享工厂的 9 个一次性表现注册：cavein_boulder.lua 的洞穴落石尘土两档（低档散落／破阵、高档顶起巨石成阵）与 daywalker2_items.lua 的拾荒疯猪 7 个装备碎裂（含 spike_loot 变体读档后结算掉落 scraphat）；落石链与掉装链触发、动态拼名生成点、混合载体辨析。"
aliases: ["cavein_dust_low", "cavein_dust_high", "daywalker2_object_break_fx", "daywalker2_spike_break_fx", "daywalker2_spike_loot_fx", "daywalker2_cannon_break_fx", "daywalker2_armor1_break_fx", "daywalker2_armor2_break_fx", "daywalker2_cloth_break_fx", "cavein_debris", "落石尘土", "洞穴尘土", "掉落物碎裂", "碎装表现", "拾荒疯猪碎装", "地震落石"]
tags: [prefab, fx, server-client]
size_class: complex
dst_build: "steam-buildid:26335394"
sources: ["prefabs/cavein_boulder.lua", "prefabs/daywalker2_items.lua", "prefabs/daywalker2.lua", "stategraphs/SGdaywalker2.lua", "components/caveins.lua", "components/quaker.lua", "components/heavyobstaclephysics.lua", "prefabs/warningshadow.lua", "map/bunches.lua", "fx.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 洞穴落石尘土与拾荒疯猪碎装表现族（cavein_dust_low 等 9 注册）

## 快速判断

- 职责与适用：两个共享工厂、两个互不相干宿主的 **9 个一次性表现注册**。①`cavein_boulder.lua` 的洞穴落石尘土两档：`cavein_dust_low`（低档，散落／破阵／砸近玩家）与 `cavein_dust_high`（高档，仅阵型中央顶起的巨石成形时）；②`daywalker2_items.lua` 的拾荒疯猪（Daywalker2）7 个装备碎裂：`daywalker2_object_break_fx`、`daywalker2_spike_break_fx`、`daywalker2_spike_loot_fx`（唯一带掉落 loot 的变体）、`daywalker2_cannon_break_fx`、`daywalker2_armor1_break_fx`、`daywalker2_armor2_break_fx`、`daywalker2_cloth_break_fx`。注册行：[cavein 两档](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L693-L695)、[daywalker2 七注册](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L71-L77)
- 三种载体模式：①**联网动画壳＋声音可选**（cavein 两档——AnimState／SoundEmitter 在联网壳上，主模拟驱动；`SkipToFull`／`PlaySoundFX` 是壳方法，调用方决定是否跳帧与放声）；②**联网动画壳＋本地物理移动**（daywalker2 全部 7 注册——`MakeInventoryPhysics` 后由主模拟 `SetMotorVel` 抛出，落地滑行）；③②的 loot 变体：`spike_loot_fx` 在动画结束时结算**掉落 `scraphat`**（官方名"拾荒尖帽"），其余 6 注册零掉落。
- 重要副作用：cavein 尘土壳 `persists=false`＋动画长度＋0.5 秒自删；daywalker2 六个无 loot 注册 `persists=false`＋`animover` 侵蚀（`ErodeAway`），`spike_loot_fx` **未写 `persists=false`**（飞行中存档则读档后 `OnLoad=OnAnimOver` 直接结算掉落）。全部 9 注册零伤害。
- 修改边界：只覆盖 9 个注册的工厂、生成点、触发链、端模型与清理；宿主本体 `cavein_boulder`（可挖可携带的重物＋五石成阵系统）与拾荒疯猪 Boss 玩法只在辨析内说明不展开。清单外发现：`cavein_debris`（蚁狮沉洞警告碎屑，通用表条目）与 `warningshadow`（落石落点预警阴影）在同一条链上，按发现并入简条。
- 名称：宿主官方名 `STRINGS.NAMES.CAVEIN_BOULDER`＝"岩石"（[po](../../../raw/current/scripts/languages/chinese_s.po#L340709-L340712)）、`STRINGS.NAMES.DAYWALKER2`＝"拾荒疯猪"（[po](../../../raw/current/scripts/languages/chinese_s.po#L341984-L341987)）、loot `STRINGS.NAMES.SCRAPHAT`＝"拾荒尖帽"（[po](../../../raw/current/scripts/languages/chinese_s.po#L348994-L348997)）；9 个覆盖注册名与 `CAVEIN_DEBRIS`／`WARNINGSHADOW` 均无 `STRINGS.NAMES.*` 键，标题与正文效果描述**非官方译名**。

## 注册与构造

### cavein 尘土（cavein_dust_low、cavein_dust_high）

两档由同一工厂 `MakeFX(name, anim)` 生成（[工厂](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L661-L691)），只差动画名，与宿主 `cavein_boulder` 同文件返回（[返回](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L693-L695)）。

| 注册名 | 工厂壳 | 差异与证据 |
|---|---|---|
| `cavein_dust_low` | Transform／AnimState／SoundEmitter／Network；bank／build `cavein_dust_fx`、播 `dust_low`；标签 `FX`；客户端早退；主模拟 `persists=false`＋动画长度＋0.5 秒自删；挂 `SkipToFull`（跳到第 5–11 帧随机帧）与 `PlaySoundFX`（蚁狮 `ground_break` size=1）两个壳方法 | 低档：散落、破阵、砸近玩家三种落点共用；是否跳帧/放声完全由调用方决定（成阵分支跳帧、落石分支不跳但放声）。[工厂](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L661-L691)、[SkipToFull](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L653-L655)、[放声](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L657-L659) |
| `cavein_dust_high` | 同壳同标签，仅动画换 `dust_high` | 高档：**唯一触发场景是五石成阵中央 raised 石**（见[触发链](#cavein-尘土的高低两档触发链)）；生成端总是先 `SkipToFull` 再按 raised 放声。[生成](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L425-L430) |

### 掉落物碎裂（daywalker2_items.lua 7 注册）

7 注册由同一工厂 `MakeItemBreakFx(name, anim, loot)` 生成（[工厂](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L26-L69)），资源只有一份 `anim/daywalker_phase3.zip`（拾荒疯猪本体动画包的碎裂片段）。

| 注册名 | 动画 | loot | 差异与证据 |
|---|---|---|---|
| `daywalker2_object_break_fx` | `object_break` | 无 | 挥击臂（`equipswing`，item 名 `object`）用尽／被击落时的碎裂。[注册](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L71) |
| `daywalker2_spike_break_fx` | `spike_break` | 无 | 冲撞尖刺（`equiptackle`，item 名 `spike`）用尽时。[注册](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L72) |
| `daywalker2_spike_loot_fx` | `spike_loot` | **`scraphat`** | 唯一 loot 变体：Boss 被击败／钻地时尖刺被"卸下"而非用坏，动画结束掉出 6–20% 耐久的拾荒尖帽。[注册](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L73) |
| `daywalker2_cannon_break_fx` | `cannon_break` | 无 | 臂装炮（`equipcannon`，item 名 `cannon`）用尽／被击落时。[注册](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L74) |
| `daywalker2_armor1_break_fx` | `armor1_break` | 无 | 三件护甲碎裂：Boss `defeat` 状态第 36 帧一次性环形抛出（见[生成点表](#生成点与宿主调用)）。[注册](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L75) |
| `daywalker2_armor2_break_fx` | `armor2_break` | 无 | 同上。[注册](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L76) |
| `daywalker2_cloth_break_fx` | `cloth_break` | 无 | 同上（碎布片）。[注册](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L77) |

工厂共性：Transform／AnimState／Network（**无 SoundEmitter**——碎裂声 `use_break` 由宿主代播）＋`MakeInventoryPhysics` 物理体＋标签 `FX`／`NOCLICK`；bank `daywalker`／build `daywalker_phase3`。主模拟按有无 loot 分岔（[分岔](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L51-L62)）：loot 变体速度 3–4、15 帧后 `OnHitGround` 半速滑行，`inst.loot` 记目标、`OnLoad=OnAnimOver`（读档即结算）；无 loot 变体速度 4–5、20 帧后落地，`persists=false`。统一监听 `animover` → `OnAnimOver`（[结算](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L10-L24)）：有 loot 则 `SpawnPrefab("scraphat")` 落地、`armor:SetPercent(0.06–0.2)`、自删；无 loot 则停物理后 `ErodeAway` 侵蚀消失。

### 本体辨析（两个宿主，不计 FX 覆盖）

- `cavein_boulder`（[工厂](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L568-L649)）：**混合结构重物实体**，不是 FX——`heavyobstaclephysics`（可被顶起／下落）、`workable`（MINE，敲碎出 rock＋燧石共享掉落表并生成 `rock_break_fx`，[OnWorked](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L100-L118)）、`inventoryitem`（SetSinks 入水下沉）、`equippable`（BODY 槽重物携带、减速）、`submersible`、小地图图标与**五石成阵系统**（5 块未成阵巨石聚齐即组成"中央顶起＋四方平铺"阵型，[成阵](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L406-L469)）。标签 `heavy`／`boulder`／`caveindebris`。
- 拾荒疯猪（`daywalker2`，官方名见[验证记录](#验证记录)）：三件装备以**字符串 item 名**存于 Boss 实体（`equipswing`／`equiptackle`／`equipcannon`，值域 `object`／`spike`／`cannon`，各 `TUNING.DAYWALKER2_ITEM_USES` 次耐久，[SetEquip](../../../raw/current/scripts/prefabs/daywalker2.lua#L295-L334)、[item 表](../../../raw/current/scripts/prefabs/daywalker2.lua#L247-L249)）；耐久用尽或 Boss 被击倒／钻地时经 `DropItem`／`DropItemAsLoot` 拼名生成碎裂 FX。Boss 战斗玩法不展开。

## 关联机制导航

### 生成点与宿主调用

| 注册 | 生成语句（file:line） | 消费文件与条件 |
|---|---|---|
| `cavein_dust_low` | 动态选择：[cavein_boulder.lua#L425](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L425-L430)（成阵，raised=false 的四方槽位）；直接：[L498](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L497-L501)（落石砸近玩家）、[L505](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L503-L516)（落石未入阵） | 全部在 `cavein_boulder.lua` 内：成阵分支每石一枚（跳帧、raised 才放声）；两条落石分支放声但不跳帧，砸近玩家分支随后移除巨石本体 |
| `cavein_dust_high` | [cavein_boulder.lua#L425](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L425-L430) | 唯一生成点＝成阵分支中 `v.raised == true` 的中央槽位；`PlaySoundFX` 只对它调用（[L428-L430](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L428-L430)） |
| `daywalker2_object_break_fx`／`daywalker2_spike_break_fx`／`daywalker2_cannon_break_fx` | 动态拼名 `SpawnPrefab("daywalker2_"..item.."_break_fx")`：[daywalker2.lua#L381](../../../raw/current/scripts/prefabs/daywalker2.lua#L365-L392)（`DropItem`） | `DropItem(action)` 的消费方：SG `dropitem` 状态（耐久用尽，[SGdaywalker2.lua#L810-L818](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L810-L818)，由 swing／tackle／cannon 各攻击状态转入）；挥击／冲撞／炮击状态收尾（[swing L1322/L1375](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L1322-L1322)、[tackle L1892](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L1891-L1894)、[cannon L1529](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L1528-L1529)）；`defeat` 进场丢 swing（[L2273-L2275](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L2269-L2276)）；钻地 `MakeBuried` 丢三件（[daywalker2.lua#L754-L770](../../../raw/current/scripts/prefabs/daywalker2.lua#L754-L770)） |
| `daywalker2_spike_loot_fx` | 动态拼名 `SpawnPrefab("daywalker2_"..item.."_loot_fx")`：[daywalker2.lua#L381](../../../raw/current/scripts/prefabs/daywalker2.lua#L365-L392)（`DropItemAsLoot`＝`DropItem(action, nosound, true)`） | 两处：`defeat` 第 36 帧 `equiptackle=="spike"` 时卸刺（[SGdaywalker2.lua#L2290-L2294](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L2285-L2314)）；钻地 `MakeBuried` 同条件卸刺（[daywalker2.lua#L758-L764](../../../raw/current/scripts/prefabs/daywalker2.lua#L754-L770)）。原版 `TACKLE_ITEMS` 只有 `"spike"`，两处条件恒真 |
| `daywalker2_armor1_break_fx`／`daywalker2_armor2_break_fx`／`daywalker2_cloth_break_fx` | 循环 `SpawnPrefab(v)`：[SGdaywalker2.lua#L2303-L2314](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L2303-L2314) | 唯一生成点：`defeat` 状态第 36 帧，三件以随机起始角＋120° 间隔环形布在 Boss 周围（半径 0.5–1.5）；同帧隐藏本体 `swap_hunch`／`swap_cloth` 等符号并停用尸装 |

### cavein 尘土的高低两档触发链

落石到尘土的完整链路（两端都汇到 `OnStopFalling`）：

1. **地震链（常规洞穴地震）**：quaker 组件按时间片给玩家落碎屑；暗影裂隙激活时 `DoDropForPlayer` 以 `override_prefab = "cavein_boulder"`（density 0＝落地保留）替换常规碎屑（[quaker.lua#L410-L414](../../../raw/current/scripts/components/quaker.lua#L410-L414)）→ `SpawnDebris` 传送至 y=35、放落点预警阴影 `warningshadow`、推 `startfalling`（[quaker.lua#L322-L368](../../../raw/current/scripts/components/quaker.lua#L322-L368)）。
2. **蚁狮沉洞链**：caveins 组件对目标玩家 `DoTargetAttack` 推 `ms_miniquake`（rad 4、30 发、3.5 秒、`debrisfn` 固定返回 `cavein_boulder`，[caveins.lua#L72-L82](../../../raw/current/scripts/components/caveins.lua#L72-L82)）→ quaker `OnMiniQuake` 逐时间片调用 `debrisfn` 生成落石（[quaker.lua#L625-L636](../../../raw/current/scripts/components/quaker.lua#L625-L636)、[_OnMiniQuakeSpawn](../../../raw/current/scripts/components/quaker.lua#L611-L622)）。
3. **落地**：`heavyobstaclephysics` 监听 `stopfalling`（[组件接线](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L239-L239)）回调 `OnStopFallingFn`（[OnStopFalling](../../../raw/current/scripts/components/heavyobstaclephysics.lua#L149-L159)）→ `cavein_boulder.lua` 的 `OnStopFalling`：
   - 1 格内有玩家 → `cavein_dust_low`＋放声＋**移除巨石**（[L497-L501](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L497-L501)）；
   - 无玩家 → `TryFormationAt` 找 2.5 格内 5 块未成阵巨石成阵（[L454-L469](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L454-L469)）→ 成阵时 `CreateFormation` 给**每块**石放尘土：中央 raised 槽位 `cavein_dust_high`＋放声，四方槽位 `cavein_dust_low` 跳帧不放声（[L425-L430](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L425-L430)）；未成阵则 `cavein_dust_low`＋放声，且与已成阵阵位重叠时移除巨石、否则原地转为障碍物（[L503-L516](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L503-L516)）。

**高低两档语义**：`dust_high`＝"巨石被顶起的成形仪式"（只在阵型中央、且总伴随 ground_break 音效）；`dust_low`＝其余一切（散落、破阵、砸近玩家）。世界生成布置的巨石群同样走该链（[bunches 布置](../../../raw/current/scripts/map/bunches.lua#L95-L95)）。

### 层、父子与端模型

| 对象 | 本族如何使用 | 边界 | 证据 |
|---|---|---|---|
| cavein 尘土端模型 | 联网壳复制位置；动画与声音都挂在壳上由主模拟驱动；无本地子实体、无非专服分支 | 专用服务器同样生成（无渲染意义）；`SkipToFull` 只改变起始帧不改变寿命（自删按完整动画长度＋0.5 秒） | [工厂](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L661-L691)、[跳帧](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L653-L655) |
| daywalker2 fx 端模型 | 联网壳＋本地物理；抛出方向／速度在主模拟 `SetMotorVel` 决定，位置随网络复制；`OnHitGround` 在 15／20 帧后把速度减半模拟"落地滑行" | 动画在壳上主模拟播；碎裂声不在 FX 上（宿主代播一次 `use_break`） | [工厂](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L26-L69)、[抛出](../../../raw/current/scripts/prefabs/daywalker2.lua#L381-L390) |
| 落点预警 | 每块下落巨石配一枚 `warningshadow` 预警阴影（清单外并入，另被月雹／废墟落石柱共用，不展开） | 阴影与巨石互听 onremove 联动清理 | [quaker](../../../raw/current/scripts/components/quaker.lua#L359-L362)、[注册](../../../raw/current/scripts/prefabs/warningshadow.lua#L31-L31) |

### 清理路径

| 注册 | 壳 | 附属 | 证据 |
|---|---|---|---|
| `cavein_dust_low`／`cavein_dust_high` | 主模拟动画长度＋0.5 秒定时自删；`persists=false` 不入存档 | 无本地子实体 | [工厂尾](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L682-L683) |
| daywalker2 六个无 loot 注册 | `animover` → 停物理 → `ErodeAway` 侵蚀；`persists=false` | 无 | [OnAnimOver](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L10-L24)、[无 loot 分支](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L58-L62) |
| `daywalker2_spike_loot_fx` | `animover` → 生成 scraphat → 自删；**未写 `persists=false`**，且 `OnLoad=OnAnimOver` | 读档后若还活着即当场结算掉落 | [结算](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L11-L17)、[loot 分支](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L51-L56) |

### 伤害与玩法归属辨析

9 个注册自身都没有 combat／weapon 等玩法组件；表现背后的结算位置：

- **落石砸地伤害**：由 quaker 的落点检测结算——`cavein_boulder` 带 `heavy` 标签，下落砸中的战斗单位挨 30 点（[quaker.lua#L215](../../../raw/current/scripts/components/quaker.lua#L215-L215)）、可破坏物直接 `Destroy`、地上物品被弹飞；尘土 FX 零结算。巨石被玩家挖碎的掉落在本体 `workable` 回调（[OnWorked](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L100-L118)）。
- **scraphat 掉落**：`spike_loot_fx` 的 `OnAnimOver` 给掉落物 `armor:SetPercent(GetRandomMinMax(0.06, 0.2))`（[结算](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L11-L17)）——拾荒尖帽官方名"拾荒尖帽"（[po](../../../raw/current/scripts/languages/chinese_s.po#L348994-L348997)）；耐久 6–20% 是 FX 结算的一部分，不是掉落表。注意 scraphat 的工厂不在本快照 `raw/current/scripts` 内（见[陷阱](#常见陷阱与限制)）。
- **装备碎裂本身**：三件"武器"是 Boss 实体上的字符串状态＋耐久计数（[SetEquip](../../../raw/current/scripts/prefabs/daywalker2.lua#L295-L334)），伤害走 Boss 的 combat；碎裂 FX 只是耐久用尽／被卸下的视觉收尾。
- **cavein_debris（清单外并入）**：蚁狮沉洞**警告阶段**的地面碎屑（注册于通用表 `fx.lua`，[条目](../../../raw/current/scripts/fx.lua#L1687-L1692)；生成于 caveins 的 `DoTargetDebris`，[L34-L53](../../../raw/current/scripts/components/caveins.lua#L34-L53)，3–4 片、逐片外扩、0.4–0.6 秒一片），带自己的 ground_break size≈0 音效；它不落在 9 注册清单内、不随落石出现，通用表工厂协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。

## 典型接入

### 手动播放一次成阵／碎裂表现

- 前提与选择：cavein 尘土位置定位＋可选跳帧／放声，直接 `SpawnPrefab` 即用；daywalker2 碎裂要自己给抛出方向与速度（工厂不设初速以外的朝向），loot 结算逻辑已内置。
- 生效与撤销：主模拟生成；cavein 尘土想要"成形仪式"观感就在生成后立刻调 `SkipToFull()`，想要"从零播起"就不调；`PlaySoundFX()` 由调用方按需要触发。
- 原版依据：[成阵写法](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L425-L430)、[动态拼名写法](../../../raw/current/scripts/prefabs/daywalker2.lua#L381-L390)。

```lua
local function PlayCaveinDust(x, z, raised)
    local fx = SpawnPrefab(raised and "cavein_dust_high" or "cavein_dust_low")
    if fx ~= nil then
        fx.Transform:SetPosition(x, 0, z)
        fx:SkipToFull()     -- 跳过起始帧（落石分支不调用）
        if raised then
            fx:PlaySoundFX() -- 仅顶起巨石时带 ground_break 音效
        end
    end
end
```

- 调整与验证：替换点是注册名与 `raised` 语义；`SkipToFull`／`PlaySoundFX` 是 cavein 两档壳上的实例方法，别的 FX 没有。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：`cavein_dust_high` 只有一个生成点。** 它不是"更大的爆炸"：成阵分支里 raised 槽位之外永远不会出高档尘土；直接 `SpawnPrefab("cavein_dust_high")` 可用，但语义是"顶起巨石的成形仪式"。[唯一生成点](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L425-L430)
- **边界：落石砸近玩家时巨石无声消失。** 1 格内有玩家的落地分支只放尘土＋音效并 `Remove` 本体——**不掉落 rocks／flint**（挖碎才有掉落表）；不要把这条分支当成"巨石摔碎了给奖励"。[分支](../../../raw/current/scripts/prefabs/cavein_boulder.lua#L497-L501)
- **误区：daywalker2 碎裂 FX 会掉装备。** 只有 `spike_loot_fx` 结算掉落 `scraphat`；其余 6 注册都是 `persists=false` 的纯表现。`spike_break_fx`（用尽碎裂）与 `spike_loot_fx`（卸下掉落）共享 spike 动画语义但结局完全不同。[对照](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L51-L62)
- **边界：`daywalker2_spike_loot_fx` 未写 `persists=false`。** 其余 6 注册都关持久化，唯独 loot 变体没有——理论上可在存档里留下一个飞行中的碎裂 FX，读档后 `OnLoad=OnAnimOver` 当场结算掉落（耐久随机重roll）。复用该工厂时注意这一差异。[loot 分支](../../../raw/current/scripts/prefabs/daywalker2_items.lua#L51-L56)
- **边界：动态拼名无 nil 保护。** `SpawnPrefab("daywalker2_"..item.."_break_fx")` 之后直接 `fx.Transform:SetRotation`——原版三张 item 表都是单元素、拼名必然命中；若模组向 `SWING_ITEMS` 等表注入未注册名，`DropItem` 会在 SetRotation 处报错。[拼名](../../../raw/current/scripts/prefabs/daywalker2.lua#L381-L390)
- **边界：daywalker2 fx 自带物理且会被抛出 2 格。** 生成位置不是 Boss 原地而是沿朝向偏 2 格（角度偏 ±70–90°随机），护甲三件则是环形 0.5–1.5 格；`NOCLICK`＋`FX` 标签但物理真实存在，密集生成时注意碰撞。[抛出](../../../raw/current/scripts/prefabs/daywalker2.lua#L382-L386)、[环形](../../../raw/current/scripts/stategraphs/SGdaywalker2.lua#L2303-L2314)
- **边界：scraphat 工厂不在快照内。** 全 `raw/current/scripts` 检索 `scraphat` 只命中 loot 生成（daywalker2_items.lua:73）、拆解配方（recipes.lua:1683）、图鉴与台词引用——没有 `Prefab("scraphat")` 注册行（`prefablist.lua` 也无）；工厂可能在快照外的数据／事件包内，不判死代码。[检索记录](#验证记录)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取两个工厂文件（`cavein_boulder.lua` 全文含成阵／OnWorked／MakeFX，`daywalker2_items.lua` 全文）；按需读取 `daywalker2.lua` 的 item 表／SetEquip／DropItem／MakeBuried／scrapbook_deps，`SGdaywalker2.lua` 的 dropitem／defeat／swing／tackle／cannon 状态相关段，`caveins.lua` 全文，`quaker.lua` 的 SpawnDebris／_OnMiniQuakeSpawn／OnMiniQuake／DoDropForPlayer 与落点伤害段，`heavyobstaclephysics.lua` 的 stopfalling 接线，`warningshadow.lua` 注册，`fx.lua` 的 cavein_debris 条目。全 `raw/current/scripts` 反查（`grep -rn "注册名" --include=*.lua`）：`cavein_dust_low` 命中 5 行（依赖表 L20、生成 L425/L498/L505、注册 L694，均在 cavein_boulder.lua）；`cavein_dust_high` 命中 3 行（L21、L425、L695）——两档无其他文件生成点、无动态拼名；7 个 `daywalker2_*_break_fx`／`_loot_fx` 命中 daywalker2.lua 依赖表（L20-L26）、daywalker2.lua:381 动态拼名（`"daywalker2_"..item..(_loot and "_loot_fx" or "_break_fx")`）、SGdaywalker2.lua:2303 三件护甲列表、daywalker2_items.lua 注册行——armor 三件唯一生成点在 defeat 帧 36，object/spike/cannon 走 DropItem／DropItemAsLoot 的 6 处消费；`cavein_boulder` 另命中 caveins.lua（依赖 L15、debrisfn L69）、quaker.lua:411（暗影裂隙 override）、map/bunches.lua:95（世界生成布置）、debugcommands 等非生成引用。清单外发现：`cavein_debris`（fx.lua:1687 注册、caveins.lua:49 生成）与 `warningshadow`（warningshadow.lua:31 注册、quaker.lua:359 生成）已并入正文简条。
- 名称核验：`languages/chinese_s.po` 定向检索——`CAVEIN_BOULDER`＝"岩石"（[L340709-L340712](../../../raw/current/scripts/languages/chinese_s.po#L340709-L340712)）、`DESCRIBE.CAVEIN_BOULDER.GENERIC`＝"我觉得我可以举起这个。"（[L10674-L10677](../../../raw/current/scripts/languages/chinese_s.po#L10674-L10677)）、`DAYWALKER2`＝"拾荒疯猪"（[L341984-L341987](../../../raw/current/scripts/languages/chinese_s.po#L341984-L341987)）、`SCRAPHAT`＝"拾荒尖帽"（[L348994-L348997](../../../raw/current/scripts/languages/chinese_s.po#L348994-L348997)）；`CAVEIN_DUST_LOW`／`CAVEIN_DUST_HIGH`／`DAYWALKER2_*_BREAK_FX`／`_LOOT_FX`／`CAVEIN_DEBRIS`／`WARNINGSHADOW` 均无 `STRINGS.NAMES.*` 键，正文使用已核验效果描述。
- 分档说明：`size_class: complex`——9 注册分属 2 个工厂文件＋2 条互不相干的宿主链（cavein 地震／成阵链横跨 quaker／caveins／heavyobstaclephysics 三组件；daywalker2 掉装链横跨 Boss 实体与 SG 多状态），同一注册在两文件共用工厂下仍有 loot／非 loot 与高／低档的行为分岔，`scraphat` 工厂缺失的检索结论需要整体保留，超出 standard 单页容量。
- 未覆盖：`cavein_boulder` 成阵玩法全貌（顶起石的搬运收益、`dropraisedboulder` 事件驱动的倒伏）、拾荒疯猪 Boss 战斗规则与三件武器的攻击数值、`warningshadow` 在月雹／废墟落石柱的复用细节、蚁狮沉洞系统（sinkhole 生成）、scraphat 本体装备属性（工厂不在快照）、动画资源内容与观感。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/ruins_cavein_obstacle_rubble_fx|废墟落石碎屑与实验室吊舱皮肤 FX]]（同主题落石碎屑；对照遗迹侧落石柱）
- [[reference/prefabs/rock_break_fx|岩石碎裂音效代理]]（巨石被挖碎时由 OnWorked 生成的音效代理）
- [[reference/prefabs/shatter|一次性碎裂与爆裂表现族]]（同为宿主回调生成的一次性碎裂；含 brokentool 掉装代理对照）
- 返回 [[reference/prefabs/fx-world-index|特效 Prefab 索引·环境与世界]]
