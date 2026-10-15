---
title: "闪电与雷声表现（lightning、thunder、thunder_close、thunder_far、moonstorm_lightning）"
created: "2026-11-02"
updated: "2026-11-04"
type: prefab
summary: "天气雷击的闪电、无动画雷声、远近雷声与月暴闪电五注册，以及阻挡、月暴、真实雷击结算和世界屏闪的分工。"
aliases: ["lightning", "thunder", "thunder_close", "thunder_far", "moonstorm_lightning", "闪电", "雷声表现", "天气雷击特效", "远近雷声", "月暴闪电"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/lightning.lua", "prefabs/thunder_close.lua", "prefabs/thunder_far.lua", "prefabs/moonstorm_lightning.lua", "components/weather.lua", "components/moonstormmanager.lua", "components/ambientlighting.lua", "componentutil.lua", "prefabs/forest_network.lua", "prefabs/books.lua", "prefabs/altar_prototyper.lua", "prefabs/battlesongdefs.lua", "prefabs/chessjunk.lua", "stategraphs/SGmoose.lua", "stategraphs/SGmossling.lua", "prefabs/lightninggoat.lua", "prefabs/resurrectionstone.lua", "prefabs/staff_common.lua", "debugkeys.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 闪电与雷声表现（lightning、thunder、thunder_close、thunder_far、moonstorm_lightning）

## 快速判断

- 本页覆盖五个注册：`lightning`、`thunder` 由 `prefabs/lightning.lua` 构造联网实体（前者有动画，后者无动画）；`thunder_close`、`thunder_far` 是另行注册的远近雷声，只有网络壳＋本地声源和（近雷的）世界屏闪；`moonstorm_lightning` 是同构的月暴变体，并在落点生成或充能月玻璃。屏闪、震屏及雷声是表现；触电／点火等玩法由天气的选名和 `StrikeLightningAtPoint` 处理，月玻璃是月暴变体自己的副作用。[工厂](../../../raw/current/scripts/prefabs/lightning.lua#L63-L129)、[远近雷声](../../../raw/current/scripts/prefabs/thunder_close.lua#L34-L68)、[天气](../../../raw/current/scripts/components/weather.lua#L666-L729)
- `STRINGS.NAMES.LIGHTNING` 的简中值为「闪电」；`thunder` 未命中同名显示名键，「雷声表现」是已核验效果描述而非官方译名。`thunder_close`、`thunder_far`、`moonstorm_lightning` 同样没有 `STRINGS.NAMES` 键，均以注册名指称。[名称](../../../raw/current/scripts/languages/chinese_s.po#L344729-L344732)

## 注册与构造

| 注册 | 构造、声音与服务端清理 | 证据 |
|---|---|---|
| `lightning` | Transform、AnimState、Network、SoundEmitter；bank/build `lightning`、anim `anim`、2 倍缩放、bloom、LightOverride 1，播放 `dontstarve/rain/thunder_close` 并加 `FX` tag。非专服另排本地雷声，主模拟排屏闪并 0.5 秒后移除。 | [工厂](../../../raw/current/scripts/prefabs/lightning.lua#L63-L100)、[资源](../../../raw/current/scripts/prefabs/lightning.lua#L1-L4)、[注册](../../../raw/current/scripts/prefabs/lightning.lua#L128-L129) |
| `thunder` | Transform、Network；无 AnimState、无该工厂自带的 SoundEmitter 或 FX tag。非专服排同一本地雷声函数；主模拟排屏闪并 0.5 秒后移除。 | [工厂](../../../raw/current/scripts/prefabs/lightning.lua#L102-L129) |

两工厂均是联网实体，非专服的 `PlayThunderSound` 另创建短暂非联网声音实体，在距本地焦点超过 10 时将声源拉近到 10 距离；按雷击点与本地焦点的平方距离和 140 距离计算强度，声音实体播放后即移除。`StartFX` 按玩家距离计算屏闪和震屏，调用排在主模拟端。[雷声](../../../raw/current/scripts/prefabs/lightning.lua#L8-L46)、[屏闪](../../../raw/current/scripts/prefabs/lightning.lua#L45-L57)

### 远近雷声（thunder_close、thunder_far）

无世界雷击事件时天气改生成这两个纯表现注册，二者都没有 AnimState、没有伤害或点火逻辑；世界屏闪由 `thunder_close` 推 `screenflash` 事件、由环境光组件消费，`thunder_far` 不推屏闪。

| 注册名 | 工厂与网络端 | 声音、屏闪与清理 | 证据 |
|---|---|---|---|
| `thunder_close` | 仅 `AddNetwork`＋`FX` 标签；主模拟 `persists = false`。两端都在创建时排 `ScreenFlash`（推世界 `screenflash` 0.5）；非专服客户端监听随机 net_byte 的 dirty 回调。 | 客户端据随机值在 `TheFocalPoint` 下、半径 10 的随机角度处建非联网声源播 `dontstarve/rain/thunder_close` 后立即移除；主模拟在 0.1～0.3 秒后写随机值，并在该延迟后再等 1 秒移除本体。 | [工厂](../../../raw/current/scripts/prefabs/thunder_close.lua#L1-L66)、[屏闪消费者](../../../raw/current/scripts/components/ambientlighting.lua#L275-L275) |
| `thunder_far` | 仅 `AddNetwork`＋`FX` 标签；主模拟 `persists = false`，2 秒后移除。 | 非专服在 `TheFocalPoint` 下建非联网声源播 `dontstarve/rain/thunder_far` 后立即移除；不推 `screenflash`。 | [工厂](../../../raw/current/scripts/prefabs/thunder_far.lua#L1-L41) |

### 月暴闪电（moonstorm_lightning）

`moonstorm_lightning` 是与 `lightning` 同构的月暴变体：动画换 `Moonstorm_LightningStrike`／`moonstorm_lightningstrike`、anim `strike`，保留 bloom、LightOverride 1、`FX` 标签、非专服本地雷声与逐玩家屏闪／震屏，主模拟 0.5 秒后移除；本工厂没有 `lightning` 的 2 倍缩放。额外玩法副作用：主模拟生成时在落点半径 4 内找带 `moonstorm_glass` 标签的实体，命中则让它的掉落器弹出 `moonglass_charged`，否则在落点生成 `moonstorm_glass`。生成点有两处：天气选名在月暴范围内强制覆盖为它，月暴管理器也会在月暴内玩家附近的可走落点定期生成。[工厂](../../../raw/current/scripts/prefabs/moonstorm_lightning.lua#L75-L116)、[玻璃分支](../../../raw/current/scripts/prefabs/moonstorm_lightning.lua#L58-L73)、[天气覆盖](../../../raw/current/scripts/components/weather.lua#L666-L673)、[管理器](../../../raw/current/scripts/components/moonstormmanager.lua#L879-L888)

## 关联机制导航

| 生成者／路径 | 条件与关联 | 证据 |
|---|---|---|
| weather 事件入口 | 主模拟监听世界 `ms_sendlightningstrike`。默认 `lightning`；阻挡器命中或重定向进入阻挡范围时选 `thunder`；月暴范围会在选名时覆盖为 `moonstorm_lightning`。避雷针与普通目标可改变落点。最后按选中 prefab 在落点生成。 | [监听](../../../raw/current/scripts/components/weather.lua#L821-L823)、[选名及生成](../../../raw/current/scripts/components/weather.lua#L630-L729)；[[reference/components/weather]] |
| 真实雷击 | 普通目标路径调用 `StrikeLightningAtPoint`，函数仅在选名为 `lightning` 时处理附近可触电或可燃对象；避雷针有独立事件分支。直接生成本页 FX 不自动执行天气选目标和雷击结算。 | [天气](../../../raw/current/scripts/components/weather.lua#L701-L729)、[结算](../../../raw/current/scripts/componentutil.lua#L1065-L1090) |
| 自然天气 | 雷电模式、降水率与目标共同决定是否发出世界雷击事件；未发出时按降水率生成 `thunder_close`（>0.5）或 `thunder_far`，它们不是本页 `thunder`，也不走天气选名和雷击结算。 | [天气更新](../../../raw/current/scripts/components/weather.lua#L1095-L1117) |
| 事件调用者 | 书、战歌、祭坛、象棋废料、麋鹿鹅与幼崽、伏特羊、试金石和法杖逻辑可推送同一事件；调试键亦可推送。事件先交天气处理，不能保证最终选 `lightning`。 | [书](../../../raw/current/scripts/prefabs/books.lua#L355-L370)、[战歌](../../../raw/current/scripts/prefabs/battlesongdefs.lua#L25-L28)、[祭坛](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L148-L148)／[二](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L332-L332)／[三四](../../../raw/current/scripts/prefabs/altar_prototyper.lua#L416-L426)、[象棋](../../../raw/current/scripts/prefabs/chessjunk.lua#L113-L113)、[麋鹿鹅](../../../raw/current/scripts/stategraphs/SGmoose.lua#L406-L406)、[幼崽](../../../raw/current/scripts/stategraphs/SGmossling.lua#L74-L81)、[羊](../../../raw/current/scripts/prefabs/lightninggoat.lua#L166-L166)、[试金石](../../../raw/current/scripts/prefabs/resurrectionstone.lua#L101-L101)、[法杖](../../../raw/current/scripts/prefabs/staff_common.lua#L304-L304)／[二](../../../raw/current/scripts/prefabs/staff_common.lua#L404-L404)、[调试](../../../raw/current/scripts/debugkeys.lua#L640-L640) |
| 依赖与易混资源 | `forest_network` 预载四名（`thunder_close`、`thunder_far`、`thunder`、`lightning`）是声明，`moonstorm_lightning` 由 `forest.lua` 预载；`books.lua` 的 `fx_over = "lightning"` 拼出书本覆盖用的 `fx_lightning_over_book` 注册名，不是直接生成本页 `lightning`。 | [依赖](../../../raw/current/scripts/prefabs/forest_network.lua#L8-L14)、[月暴预载](../../../raw/current/scripts/prefabs/forest.lua#L413-L413)、[字段](../../../raw/current/scripts/prefabs/books.lua#L350-L353)、[动态注册](../../../raw/current/scripts/prefabs/books.lua#L1097-L1100) |

## 常见陷阱与限制

- `thunder` 无闪电动画，但仍经 `StartFX` 触发玩家屏闪和震屏；它的本地雷声来自延时创建的声音实体，不要因其自身无 SoundEmitter 就断言静音。[工厂](../../../raw/current/scripts/prefabs/lightning.lua#L100-L125)
- 月暴选名逻辑可覆盖阻挡器写入的 `thunder`；具体行为以天气选名顺序为准，不能简化成阻挡永远生成本页雷声。[选名](../../../raw/current/scripts/components/weather.lua#L666-L689)
- `thunder_close`／`thunder_far` 的屏闪与 `lightning`／`thunder` 不是同一条链：近雷推世界 `screenflash` 事件给环境光，`lightning`／`thunder` 则在主模拟对每个玩家调用 `ScreenFlash`＋`ShakeCamera`。远近雷声也没有雷击落点语义，别用它们的工厂去补雷击伤害。[屏闪消费者](../../../raw/current/scripts/components/ambientlighting.lua#L275-L275)、[逐玩家屏闪](../../../raw/current/scripts/prefabs/lightning.lua#L45-L57)

## 验证记录

- 源码审阅：2026-11-02，Steam buildid 26335394。核验两个注册及全部工厂字段、端、音效／屏闪和回收；递归检索整个 `raw/current/scripts` 中 `lightning`、`thunder`、`ms_sendlightningstrike` 与选名／生成字段，列出天气实际生成链及事件入口，区分资产名称和依赖声明。
- 2026-11-04 批 C 独立复核：重读 `weather.lua` 选名与生成段、`componentutil.StrikeLightningAtPoint` 和全部 16 处 `ms_sendlightningstrike` 引用，确认普通目标分支对 `thunder` 也会调用结算函数但函数内部只处理 `lightning`；屏闪强度按雷击点与本地焦点距离表述，锚点起止行按最终版本复核（标题与 `forest_network` 预载数已按独立复审修正）。
- 2026-11-04 扩页补核：新增 `thunder_close`、`thunder_far`、`moonstorm_lightning` 三注册（各自独立工厂文件、天气 L1116 与选名覆盖、月暴管理器 L883 两处生成点、`forest_network` 仅预载声明），核对本地声源父挂、随机 net_byte 时序、世界 `screenflash` 消费者、1 秒／2 秒移除与月玻璃生成分支；动画包内容、其他模组调用和游戏内声画未核验。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-world-index|特效 Prefab 索引·环境与世界]]。
