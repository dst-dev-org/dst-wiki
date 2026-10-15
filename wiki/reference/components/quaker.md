---
title: "quaker"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "洞穴地震组件：主端按 等待→警告→震动的状态机给每名玩家投落重物/小动物，管理碎片落点伤害与碎裂规则，双端经 netvar 同步震动音强。"
aliases: ["地震", "落石", "quaker"]
tags: [component, state-machine]
dst_build: "steam-buildid:26230496"
sources: ["components/quaker.lua", "prefabs/cave_network.lua", "components/caveins.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# quaker

## 快速判断

- 职责与适用：装在洞穴网络世界（[cave_network](../../../raw/current/scripts/prefabs/cave_network.lua#L9-L10)），双端存在：主端跑地震状态机与碎片模拟，客户端只消费 `quaker._quakesoundintensity`/`_miniquakesoundintensity` 两个 netvar 播声（[L76-L77](../../../raw/current/scripts/components/quaker.lua#L76-L77)、[L730-L736](../../../raw/current/scripts/components/quaker.lua#L730-L736)）。
- 优先使用：外部触发走世界事件——`ms_miniquake`（局部震动，caveins/minotaur/传送杖在用）、`ms_forcequake`（月书等）、`explosion`（提前下一场地震）（[L743-L746](../../../raw/current/scripts/components/quaker.lua#L743-L746)、[消费例](../../../raw/current/scripts/components/caveins.lua#L73)）。
- 重要副作用：碎片实体自 y=35 落下并带 `warningshadow` 预警影；重碎片落地砸碎可破坏物/攻击无 epic 标签生物/弹起物品；地震结束给全程玩家 `survive_earthquake` 成就（[L484-L492](../../../raw/current/scripts/components/quaker.lua#L484-L492)、[L192-L237](../../../raw/current/scripts/components/quaker.lua#L192-L237)）。
- 修改边界：碎片分配依赖节点 tags（`_tagdebris`），默认表与配置在构造末尾硬编码（[L753-L811](../../../raw/current/scripts/components/quaker.lua#L753-L811)）；`SetQuakeData`/`SetDebris`/`SetTagDebris` 均主端专用。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetQuakeData(data)` | `{warningtime, quaketime, debrispersecond, nextquake, mammals}`（可为函数） | 配置并启动下一场倒计时；`nextquake` 除以 `TUNING.QUAKE_FREQUENCY_MULTIPLIER`；默认 警告 7s、震动 10~15s、5~6 块/秒、间隔 1~3 日、哺乳动物 1 只（[L698-L707](../../../raw/current/scripts/components/quaker.lua#L698-L707)、[默认配置](../../../raw/current/scripts/components/quaker.lua#L805-L811)）。 |
| `SetDebris(data)` / `SetTagDebris(tag, data)` | `{weight, loot[]}` 数组 | 全局碎片表；按节点 tags 覆盖（空表＝无碎片），缺省回退全局表；原版给 `lunacyarea` 换月岛碎片、`nocavein` 置空禁落（[L709-L719](../../../raw/current/scripts/components/quaker.lua#L709-L719)、[L94-L115](../../../raw/current/scripts/components/quaker.lua#L94-L115)、[L780-L803](../../../raw/current/scripts/components/quaker.lua#L780-L803)）。 |
| `IsQuaking()` | → bool | 大震音强 >1 或 mini 震动 netvar 为真（[L721-L723](../../../raw/current/scripts/components/quaker.lua#L721-L723)）。 |
| `ms_miniquake` 事件字段 | `{pos/target, rad, minrad, num, duration, debrisfn?}` | 不进状态机：分片计时逐个落点 `SpawnDebris`、镜头震动、置 mini 音强 netvar，结束自动复位（[L611-L640](../../../raw/current/scripts/components/quaker.lua#L611-L640)）。 |
| `ms_forcequake` 事件 | 可带 `SetQuakeData` 同构 data | 震动中忽略；否则立即 `StartQuake`（[L654-L658](../../../raw/current/scripts/components/quaker.lua#L654-L658)、[消费](../../../raw/current/scripts/prefabs/books.lua#L764)）。 |
| `pausequakes`/`unpausequakes` 事件 | `{source, reason}` | SourceModifierList 暂停源：警告阶段被暂停时重新排下一场（[L524-L528](../../../raw/current/scripts/components/quaker.lua#L524-L528)、[L682-L692](../../../raw/current/scripts/components/quaker.lua#L682-L692)）。 |
| 落点保护 | `quake_blocker` 标签 | 选点排除 `TUNING.QUAKE_BLOCKER_RANGE` 内的防震物；玩家位置 `CanPointHaveQuaker` 为否不参与（[L377-L400](../../../raw/current/scripts/components/quaker.lua#L377-L400)、[L83-L86](../../../raw/current/scripts/components/quaker.lua#L83-L86)）。 |
| OnSave/OnLoad | time/state/debrispersecond/mammalsremaining | 按状态恢复对应阶段剩余时间（[L817-L838](../../../raw/current/scripts/components/quaker.lua#L817-L838)）。 |

## 关键流程

### 地震生命周期

1. 等待：`SetNextQuake` 排 `nextquake` 定时；`explosion` 事件可把等待中的剩余时间减 `data.damage` 提前（[L642-L650](../../../raw/current/scripts/components/quaker.lua#L642-L650)）。
2. 警告：7 秒警告（音强 1、全员镜头震动＋随机 ANNOUNCE_QUAKE 语音），结束转 `StartQuake`（[L515-L537](../../../raw/current/scripts/components/quaker.lua#L515-L537)）。
3. 震动：音强 2，给每位可震动玩家按 `1/_debrispersecond` 周期循环排落点任务；`quaketime` 到点 `EndQuake(continue=true)` 回到等待（[L494-L513](../../../raw/current/scripts/components/quaker.lua#L494-L513)、[L474-L492](../../../raw/current/scripts/components/quaker.lua#L474-L492)）。
4. 碎片生成 `SpawnDebris`：按落点节点 tags 抽 loot；`rabbit/mole/carrat` 受 `_mammalsremaining` 限额（sg 进 `fall` 态）；落兔时按 `RABBITKING_LUCKY_ODDS_QUAKER` 运气改生成 lucky 兔子王并交给 [[reference/components/rabbitkingmanager]]（[L322-L369](../../../raw/current/scripts/components/quaker.lua#L322-L369)、[L329-L340](../../../raw/current/scripts/components/quaker.lua#L329-L340)）。
5. 落地判定 `_GroundDetectionUpdate`（每帧）：无效地面/洞口直接移除或碎裂；重碎片砸工作台/攻击/弹物，普通碎片仅攻击与碎裂；75% 概率或密度不足时保留（恢复可拾取、推 `stopfalling`），否则首弹后 0.4~0.6s 碎裂（[L173-L314](../../../raw/current/scripts/components/quaker.lua#L173-L314)）。
6. 影裂加成：影裂激活时按运气把该玩家的落点换成 `cavein_boulder`（半径与密度覆盖，[L402-L421](../../../raw/current/scripts/components/quaker.lua#L402-L421)）。

## 常见陷阱与限制

- **Charlie 竞技场静音**：客户端玩家在该区域时跳过震动音（`CanHearQuaker`），但主端模拟照常（[L556-L566](../../../raw/current/scripts/components/quaker.lua#L556-L566)）。
- **mini 震动不改状态机**：`ms_miniquake` 与大震互不感知（源码 FIXME 承认音效会被覆盖），并行调用只叠落点（[L638-L640](../../../raw/current/scripts/components/quaker.lua#L638-L640)）。
- **玩家中途加入**：震动中进服的玩家也会被排落点任务，离服则取消（[L660-L680](../../../raw/current/scripts/components/quaker.lua#L660-L680)）。
- **deactivate 期碎片不保留**：`persists=false` 的碎片只有在成功"保留"分支才转持久；读档不会恢复下落中的碎片（[L343-L344](../../../raw/current/scripts/components/quaker.lua#L343-L344)、[L269-L277](../../../raw/current/scripts/components/quaker.lua#L269-L277)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、cave_network 装配、caveins/books/minotaur/staff_common 触发点、rabbitkingmanager 联动段；未覆盖 cavein_boulder 与 warningshadow prefab 细节。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/rabbitkingmanager]]、[[reference/components/caveweather]]（同装配于 cave_network）。
