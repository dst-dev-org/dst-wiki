---
title: "手持气球双层代理与彩带庆典（balloon_held_child 等 4 注册）"
created: "2026-11-05"
updated: "2026-11-08"
type: prefab
summary: "Wes 气球家族的表现注册：手持气球的联网代理＋本地动画两层架构（代理只同步颜色/换装/放气 netvar，真实动画在非联网的 balloon_held_child_client 上跑 SGballoonheld 并跟随持握符号），以及派对气球爆开时的彩带云（网络化混合载体，每 1 秒给附近玩家挂回智 debuff）与四只非联网物理彩带气球。"
aliases: ["balloon_held_child", "balloon_held_child_client", "balloonparty_confetti_cloud", "balloonparty_confetti_balloon", "手持气球", "气球派对彩带", "彩带云", "彩带气球", "wes 气球表现", "气球回智"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/balloon_held_child.lua", "prefabs/balloons_common.lua", "prefabs/balloonparty.lua", "prefabs/balloon.lua", "prefabs/balloonspeed.lua", "stategraphs/SGballoonheld.lua", "entityscript.lua", "components/debuffable.lua", "components/debuff.lua", "tuning.lua", "recipes.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 手持气球双层代理与彩带庆典（balloon_held_child 等 4 注册）

## 快速判断

- 职责与适用：Wes 气球家族的 **4 个表现注册**，两组两层结构：①`balloon_held_child`（**联网代理**——Transform＋Network＋4 个 netvar，无动画）与 `balloon_held_child_client`（**非联网本地动画**——bank `balloon2`、SGballoonheld 状态机、FollowSymbol 跟持握符号），共同组成"手持气球"的端分层表现；②`balloonparty_confetti_cloud`（**网络化混合载体**——彩带动画之外每 1 秒给附近玩家挂回智 debuff）与 `balloonparty_confetti_balloon`（**非联网物理动画 FX**——彩带云各端自建的上升小气球）。注册行：[balloon_held_child](../../../raw/current/scripts/prefabs/balloon_held_child.lua#L150)、[balloon_held_child_client](../../../raw/current/scripts/prefabs/balloon_held_child.lua#L151)、[balloonparty_confetti_cloud](../../../raw/current/scripts/prefabs/balloonparty.lua#L308)、[balloonparty_confetti_balloon](../../../raw/current/scripts/prefabs/balloonparty.lua#L309)
- 重要副作用：`balloonparty_confetti_cloud` 是**有玩法作用的混合载体**——`DoPeriodicTask(1, ApplyBuff, 0.5)` 对全表玩家按平面距离 2.6 判定并 `AddDebuff("balloonparty_buff")`（[ApplyBuff](../../../raw/current/scripts/prefabs/balloonparty.lua#L125-L134)），debuff 每 2 秒按"附近庆祝规模"回复理智（调值表 0～4，见下文）；生成它的本体派对气球爆开时另带范围伤害（`BALLOON_DAMAGE`=5、范围 2，[调值](../../../raw/current/scripts/tuning.lua#L5922-L5923)）——伤害结算在气球的 combat 上，与本页 FX 无关。
- 修改边界：只覆盖 4 注册的工厂、生成点、父子与清理；三个持握本体（普通气球、迅捷气球、派对气球）与 `balloonparty_buff` 只在链条中辨析；气球配方、poppable 爆开玩法（见 [[reference/components/poppable|poppable 组件]]）不展开。
- 名称：持握本体有官方名——`BALLOON`＝"气球"、`BALLOONPARTY`＝"派对气球"、`BALLOONSPEED`＝"迅捷气球"（[po](../../../raw/current/scripts/languages/chinese_s.po#L338999-L339007)）；覆盖的 4 注册（`BALLOON_HELD_CHILD`、`BALLOON_HELD_CHILD_CLIENT`、`BALLOONPARTY_CONFETTI_CLOUD`、`BALLOONPARTY_CONFETTI_BALLOON`）在简中 `chinese_s.po` 无 `STRINGS.NAMES` 键，标题中文为已核验效果描述，非官方译名。

## 注册与构造

### 手持气球：联网代理 + 本地动画两层

| 层 | 构造 | 职责分界与证据 |
|---|---|---|
| `balloon_held_child`（联网代理） | Transform／Network（无 AnimState）；标签 `DECOR`；netvar：`_colour_idx`(net_tinybyte)、`_swap_build`/`_swap_sym`(net_hash)、`_deflate_event`(net_event)——事件通道名 `ballon_held_child.*` 为源码原样拼写；非专服 0 秒建本地客户对象；主模拟 `persists=false`、暴露 `SetupFromBaseItem` | 携带跨端状态：颜色索引、换装 build/符号哈希、放气事件；[工厂](../../../raw/current/scripts/prefabs/balloon_held_child.lua#L80-L114)、[netvar](../../../raw/current/scripts/prefabs/balloon_held_child.lua#L88-L91) |
| `balloon_held_child_client`（本地动画） | Transform／AnimState／SoundEmitter（**无 Network**）；bank/build `balloon2`、`idle` 循环、`swap_rope`→balloon2/rope_held、`swap_balloon`→balloon_shapes2/balloon_1、FinalOffset −1、四向；标签 `FX`；`persists=false`；装 SGballoonheld；专服 0 秒自删兜底 | 每台非专服机器各建一份、跟随主人持握符号跑动画；颜色/换装从代理 netvar 读入；[工厂](../../../raw/current/scripts/prefabs/balloon_held_child.lua#L116-L148) |

两层由 `SpawnClientObject` 焊接（[L56-L78](../../../raw/current/scripts/prefabs/balloon_held_child.lua#L56-L78)）：客户对象 `SetParent(持握玩家)`＋`AddFollower:FollowSymbol(玩家, "swap_object", 0,0,0)` 贴手部持握符号，记 `net_proxy` 指回代理，并起**每帧轮询** `updateballonrunningstate`——读主人 SG 的 `moving` 状态标签（无 SG 则读实体 `moving` 标签）驱动 `_isrunning`，变化时 `PushEvent("sg_update_running_state")`。SGballoonheld 五态：`idle`/`start_running`/`running`/`stop_running`/`deflate`，running 第 1 帧播 `wes/characters/wes/speedballoon`（[SG](../../../raw/current/scripts/stategraphs/SGballoonheld.lua#L38-L60)）。

`SetupFromBaseItem`（[L31-L54](../../../raw/current/scripts/prefabs/balloon_held_child.lua#L31-L54)，代理生成后由本体调用）：代理 `SetParent(本体持有者)`、写入 `baseitem.colour_idx`、读本体 AnimState 的 `swap_balloon` 符号 override 写入两个哈希 netvar；**仅当本体带 fueled**（实测只有迅捷气球，[L166-L171](../../../raw/current/scripts/prefabs/balloonspeed.lua#L166-L171)）才挂 `onfueldsectionchanged`——段位变化时更新换装哈希、本地客户对象直接进 `deflate` 态并向各端推 `_deflate_event`（客户端监听同名事件让自家客户对象放气，[L100-L104](../../../raw/current/scripts/prefabs/balloon_held_child.lua#L100-L104)）。`deflate` 态第 13 帧调 `UpdateBalloonSymbol` 换上新符号（[SG](../../../raw/current/scripts/stategraphs/SGballoonheld.lua#L80-L110)）。

### 彩带庆典：confetti 两注册

| 注册名 | 构造 | 差异与证据 |
|---|---|---|
| `balloonparty_confetti_cloud` | Transform／AnimState／SoundEmitter／Network；bank/build `wes_confetti`、`confetti_pre`；`PlaySound("wes/characters/wes/balloon_party")` 在 SetPristine 前（各端构建即响）；标签 `confetti_cloud`＋`FX`；非专服 0 秒建 4 只彩带气球（随机偏移 r=1.0）；主模拟：连推 4 次 `confetti_loop`→`confetti_pst`、`animqueueover` 自删、`DoPeriodicTask(1, ApplyBuff, 0.5)`；**未置 persists=false** | 网络化混合载体：动画随网络复制，debuff 结算在主模拟；贴片生命只有约 5 段动画（约数秒），存档窗口极小但存在（[工厂](../../../raw/current/scripts/prefabs/balloonparty.lua#L136-L188)） |
| `balloonparty_confetti_balloon` | Transform／AnimState＋Physics（**无 Network**）；`SetCanSleep(false)`、`persists=false`；球体半径 0.5、质量 1、弹性 0.5；bank/build `wes_balloon_party`、`float` 循环＋随机起始帧；`BALLOONS.SetColour` 随机 6 色；缩放 0.9–1.0；标签 `CLASSIFIED`＋`FX`；每帧 `SetMotorVel`：水平分量 1 秒后按 inQuad 渐入（4 秒达峰）、垂直从 2.5 升到 6.5；y≥35 或 15 秒自删 | 纯物理动画 FX：各端自建 4 只，上升飘散；不随存档（[工厂](../../../raw/current/scripts/prefabs/balloonparty.lua#L268-L304)、[运动曲线](../../../raw/current/scripts/prefabs/balloonparty.lua#L255-L266)） |

## 关联机制导航

### 生成点与持有者链

| 注册 | 生成语句（file:line） | 条件 |
|---|---|---|
| `balloon_held_child` | [balloons_common.lua L101](../../../raw/current/scripts/prefabs/balloons_common.lua#L93-L113)（`OnEquip_Hand`） | 唯一生成点：任一气球**装备到手部槽**时由本体生成并 `SetupFromBaseItem`；三个持有者都走同一回调——普通气球 [balloon.lua L92-L94](../../../raw/current/scripts/prefabs/balloon.lua#L92-L94)、迅捷气球 [balloonspeed.lua L161-L163](../../../raw/current/scripts/prefabs/balloonspeed.lua#L161-L163)、派对气球 [balloonparty.lua L111-L113](../../../raw/current/scripts/prefabs/balloonparty.lua#L111-L113)。本体获得途径：派对气球配方限 `balloonomancer` 标签（[recipes.lua L375](../../../raw/current/scripts/recipes.lua#L375)） |
| `balloon_held_child_client` | [balloon_held_child.lua L59](../../../raw/current/scripts/prefabs/balloon_held_child.lua#L56-L78) | 代理生成后每台非专服机器 0 秒各建一份；专服不建（工厂门控＋客户工厂自删双保险） |
| `balloonparty_confetti_cloud` | [balloonparty.lua L55](../../../raw/current/scripts/prefabs/balloonparty.lua#L53-L60) | 唯一生成点：本体派对气球爆开（poppable `onpop`）5 帧后在气球原位 |
| `balloonparty_confetti_balloon` | [balloonparty.lua L159-L169](../../../raw/current/scripts/prefabs/balloonparty.lua#L155-L171) | 彩带云工厂内非专服 0 秒任务，一次 4 只 |

### 彩带云的 debuff 链（混合辨析）

`ApplyBuff` 每 1 秒（首拍 0.5 秒）遍历 `AllPlayers` 全表、按**平面距离**（distsq 忽略 y）<2.6² 判定后 `AddDebuff("balloonparty_buff", "balloonparty_buff")`（[L125-L134](../../../raw/current/scripts/prefabs/balloonparty.lua#L125-L134)）——AddDebuff 在玩家无 debuffable 组件时会现装该组件（[entityscript.lua L2140-L2154](../../../raw/current/scripts/entityscript.lua#L2140-L2154)），同名重复施加会进 debuff 的 Extend（[debuffable.lua L103-L107](../../../raw/current/scripts/components/debuffable.lua#L95-L112)）；但 `balloonparty_buff` 工厂没有设置 `onextendedfn`，Extend 只是空回调（[debuff.lua L53-L57](../../../raw/current/scripts/components/debuff.lua#L53-L57)），**不负责续期**。持续生效来自附着实体的 `buff_OnTick` 周期检查附近是否仍有 `confetti_cloud`，没有才 `Stop`，且首次附着会立即执行一次 OnTick（[tick 实现](../../../raw/current/scripts/prefabs/balloonparty.lua#L193-L222)）。`balloonparty_buff`（清单外连带注册，[balloonparty.lua L310](../../../raw/current/scripts/prefabs/balloonparty.lua#L307-L310)）是主模拟专用隐藏实体：父挂目标、`CLASSIFIED`＋`confetti_buff` 标签，每 2 秒（`CONFFETI_PARTY_SANITY_TICKRATE`，[调值](../../../raw/current/scripts/tuning.lua#L5931-L5940)）核一次"2.6 内仍有 confetti_cloud"，按 `CountEntities(4, {CLASSIFIED,confetti_buff})`（附近其他庆祝者）＋`CountEntities(4, {confetti_cloud})`（附近云数）计"派对规模"，从调值表 {0,1,2,3,3.5,4} 取理智增量；离开范围即 debuff Stop（[tick 实现](../../../raw/current/scripts/prefabs/balloonparty.lua#L193-L213)）。

### 父子与清理

| 对象 | 清理方式 | 边界与证据 |
|---|---|---|
| `balloon_held_child`（代理） | 本体卸装 `OnUnequip_Hand` Remove；本体被移除经 `onremove` 监听连带 Remove；本体装备时监听持有者 `attacked`（`ShouldProcOnAttackedOrBlocked` 门控）掉落气球——本体层面见下方 poppable 互链 | [卸装](../../../raw/current/scripts/prefabs/balloons_common.lua#L115-L128)、[本体移除](../../../raw/current/scripts/prefabs/balloons_common.lua#L87-L91) |
| `balloon_held_child_client` | 代理 `onremove` 时由代理移除客户对象（[L76](../../../raw/current/scripts/prefabs/balloon_held_child.lua#L76)）；专服兜底自删；`persists=false` 不存档 | 客户对象随代理生灭，无独立生命周期 |
| `balloonparty_confetti_cloud` | `animqueueover` 自删（约 5 段动画后） | 无兜底定时；生成侧本体爆开后与气球无父子关系（按世界坐标摆放） |
| `balloonparty_confetti_balloon` | 每帧运动回调：y≥35 或 15 秒 Remove | 非联网、各端自建自删，不进存档（[L255-L261](../../../raw/current/scripts/prefabs/balloonparty.lua#L255-L266)） |

## 典型接入

### 复刻"联网代理＋本地动画"两层表现

- 前提与选择：需要**跟随持有者的换装/状态动画**、又不想到处传 net 动画帧时，把"跨端状态"（颜色、符号、离散事件）收进一个轻代理的 netvar，动画实体每台机器本地自建并 FollowSymbol。适合手持物、背饰、光环跟随。
- 生效与撤销：代理由本体装备回调生成并 `SetupFromBaseItem` 喂状态；撤销必须把"代理 Remove → 各端代理 onremove → 客户对象 Remove"这条链闭合（客户对象依赖代理的 onremove 回调清理）。

```lua
-- 装备回调（服务端）：
inst._body = SpawnPrefab("balloon_held_child")
inst._body:SetupFromBaseItem(inst, owner, from_ground, inst.colour_idx)
-- 卸装回调：
if inst._body ~= nil then
    inst._body:Remove()
    inst._body = nil
end
```

- 调整与验证：替换点是 netvar 组与符号名（`swap_object` 是玩家持握符号）；客户对象里的 SG/动画/声音全部本地，不需要网络同步。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：`balloon_held_child` 是可见的气球。** 代理无 AnimState；可见动画全部在每台机器本地的 `balloon_held_child_client` 上。给代理调动画/音效是空操作，要改表现需经 `client_obj` 或直接改客户工厂（[代理工厂](../../../raw/current/scripts/prefabs/balloon_held_child.lua#L80-L114)）。
- **边界：放气链只有迅捷气球会触发。** `SetupFromBaseItem` 的 `onfueldsectionchanged` 监听以本体有 fueled 为前提；普通气球与派对气球本体无 fueled（全文件反查仅 balloonspeed 装配），它们的 `deflate` 态与放气声不会出现，退场走卸装/爆开路径（[fueled 装配](../../../raw/current/scripts/prefabs/balloonspeed.lua#L166-L171)）。
- **边界：running 态的音效不分气球品种。** SGballoonheld 的 running 第 1 帧固定播 `wes/characters/wes/speedballoon`——普通气球手持奔跑也有跑步充气声，这是 SG 层共享行为不是迅捷气球专属（[SG](../../../raw/current/scripts/stategraphs/SGballoonheld.lua#L44-L47)）。
- **边界：彩带云的 debuff 是"全表扫描＋平面距离"。** `ApplyBuff` 遍历 `AllPlayers` 而非 FindEntities，忽略 y 轴——楼上楼下同平面坐标即命中；判定半径 2.6 与 buff 续命半径 2.6 同源（`CONFFETI_CLOUD_DIST`，[L121-L122](../../../raw/current/scripts/prefabs/balloonparty.lua#L121-L122)）。
- **边界：彩带云没关 persists。** 工厂未置 `persists=false`（对照彩带气球的 false）——存档恰好落在其数秒生命周期内时会被存下，读档后动画队列重新推、仍走 animqueueover 自删。这是源码原样行为，复刻短命混合载体时建议显式关掉（[工厂](../../../raw/current/scripts/prefabs/balloonparty.lua#L136-L188)）。
- **边界：代理 onremove 回调清的是 `inst.body`（无下划线）。** balloons_common.lua:105 的回调写 `inst.body = nil`，与各处使用的 `inst._body` 字段名不一致——源码原样记录；实际清理由本体 onremove 与 OnUnequip_Hand 的 `_body:Remove()` 承担，未见由此引发的泄漏路径（[L104-L105](../../../raw/current/scripts/prefabs/balloons_common.lua#L104-L105)）。
- **边界：事件通道名 `ballon_held_child` 为源码原样拼写。** net_event/监听两端都用这个少一撇的名字才成立（[L91](../../../raw/current/scripts/prefabs/balloon_held_child.lua#L88-L91)、[L100](../../../raw/current/scripts/prefabs/balloon_held_child.lua#L100-L104)）；按正确拼写 `balloon_held_child` 监听收不到事件。
- **边界：清单外连带注册。** `balloonparty_buff`（注册 [balloonparty.lua L310](../../../raw/current/scripts/prefabs/balloonparty.lua#L307-L310)）由彩带云的 ApplyBuff 经 AddDebuff 生成，主模拟专用、隐藏、persists=false——不属本页原定清单，按发现并入记录；其工厂不展开，机制见[上方 debuff 链](#彩带云的-debuff-链混合辨析)。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/balloon_held_child.lua`、`prefabs/balloons_common.lua`、`prefabs/balloonparty.lua`、`stategraphs/SGballoonheld.lua`，按需读取 `prefabs/balloon.lua` 与 `prefabs/balloonspeed.lua` 的 equippable/fueled 段、`entityscript.lua` 的 SpawnChild/AddChild/AddDebuff、`components/debuffable.lua` 的 AddDebuff 刷新段、`tuning.lua` 气球与 CONFFETI 调值段、`recipes.lua` 配方行。对 4 个注册名以完整名在 `raw/current/scripts` 全域反查（`grep -rn "注册名" --include=*.lua`）：`balloon_held_child` 命中 balloons_common.lua:101 唯一生成点＋三个持有者依赖表＋prefablist.lua:88＋注册行；`balloon_held_child_client` 命中 balloon_held_child.lua:59 唯一生成点＋依赖表＋注册行；`balloonparty_confetti_cloud` 命中 balloonparty.lua:55 唯一生成点＋依赖表＋注册行；`balloonparty_confetti_balloon` 命中 balloonparty.lua:159/162/165/168（同一 0 秒任务内 4 次）＋依赖表＋注册行。`balloonparty_buff` 反查发现注册于 balloonparty.lua:310、唯一生成点为 ApplyBuff 的 AddDebuff（:131），按清单外发现并入"常见陷阱"末条。
- 名称核验：`languages/chinese_s.po` 定向检索——`BALLOON`＝"气球"、`BALLOONPARTY`＝"派对气球"、`BALLOONSPEED`＝"迅捷气球"（锚点见快速判断）；`BALLOON_HELD_CHILD`、`BALLOON_HELD_CHILD_CLIENT`、`BALLOONPARTY_CONFETTI_CLOUD`、`BALLOONPARTY_CONFETTI_BALLOON` 的 NAMES 键全文 0 命中，标题使用效果描述。
- 分档说明：`standard`（字段省略）——两组两层结构但每层职责单一、状态机只有五态；debuff 链为单层组件协作，standard 预算内可完整承载。
- 未覆盖：poppable 爆开玩法全链（伤害/掉落/proc 门控，仅记触发关系）、`balloonparty_buff` 工厂细节与 debuff 组件生命周期全貌、气球配方的角色门禁与 sg_state 前置、彩带动画内容与观感、迅捷气球 walkspeedmult 数值表、`GetSymbolOverride` 对皮肤变体的实际返回值；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录。
- 第 16 轮补正（2026-11-07）：彩带 buff 不再写“Extend 刷新续期”——该工厂未设 `onextendedfn`，Extend 为空；持续靠 `buff_OnTick` 周期检查附近 `confetti_cloud`，首次附着立即 tick 一次。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/components/poppable|poppable 组件]]（手持气球"受击掉落"门控与爆开玩法的宿主组件页）
- 返回 [[reference/prefabs/fx-skin-index|皮肤与装备挂件]]
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-06）。
