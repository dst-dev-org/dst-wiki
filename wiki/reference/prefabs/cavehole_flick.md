---
title: "洞穴弹飞、重置烟与表情效果（cavehole_flick 等）"
created: "2026-11-01"
updated: "2026-11-08"
type: prefab
summary: "洞穴弹飞预警、宝箱重置、鼹鼠地表、表情和海洋迁移等八个通用表注册。"
aliases: ["slurper_respawn", "pandorachest_reset", "cavehole_flick_warn", "cavehole_flick", "mole_move_fx", "emote_fx", "tears", "spawn_fx_ocean_static", "洞穴弹飞特效", "宝箱重置烟", "隆起的土堆", "表情特效", "海洋迁移水花"]
tags: [prefab, fx]
dst_build: "steam-buildid:26474873"
sources: ["fx.lua", "prefabs/fx.lua", "prefabs/saddle.lua", "prefabs/slurper.lua", "prefabs/waxwell.lua", "prefabs/treasurechest.lua", "prefabs/chest_mimic.lua", "prefabs/cave_hole.lua", "stategraphs/SGmole.lua", "stategraphs/SGwilson.lua", "stategraphs/SGhermitcrab.lua", "stategraphs/SGwx78_possessedbody.lua", "play_commonfn.lua", "play_the_vault.lua", "emotes.lua", "components/worldmigrator.lua", "components/playerspawner.lua", "prefabs/player_common.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 洞穴弹飞、重置烟与表情效果（cavehole_flick 等）

## 快速判断

- 八名为通用表动画，分属多个不相关生成链；洞穴击退、宝箱替换和迁移由各自调用者处理。公共工厂协议见 [[reference/mechanisms/generic-fx-factory#关键流程]]。
- `mole_move_fx` 的 `nameoverride` 实际引用 `STRINGS.NAMES.MOLE_UNDERGROUND`，简中为“隆起的土堆”；其他标题中文为效果描述，非官方译名。[键](../../../raw/current/scripts/languages/chinese_s.po#L345530-L345532)

## 注册与构造

| 注册名 | bank／build；anim | 其他字段及证据 |
|---|---|---|
| `slurper_respawn` | `die_fx`／`die`；`small` | 黑色 tint、alpha 1；deathpoof 声；无 fn。[条目](../../../raw/current/scripts/fx.lua#L513-L519) |
| `pandorachest_reset` | `attune_fx`／同左；`attune_in` | 黑色 tint、alpha 0.6；sound 注释，无 fn。[条目](../../../raw/current/scripts/fx.lua#L522-L528) |
| `cavehole_flick_warn` | `attune_fx`／同左；`attune_in` | 黑色 tint、alpha 0.8；无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L531-L536) |
| `cavehole_flick` | `statue_ruins_fx`／同左；`transform_nightmare` | alpha 0.8；暗影消散声，fn=UsePointFiltering。[条目](../../../raw/current/scripts/fx.lua#L539-L545)、[helper](../../../raw/current/scripts/fx.lua#L20-L22) |
| `mole_move_fx` | `mole_fx`／`mole_move_fx`；`move` | nameoverride 与 description 指向鼹鼠地下状态；无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L548-L556) |
| `emote_fx` | 同名 bank／build／anim | autorotate、fn=FinalOffset1；无 sound。[条目](../../../raw/current/scripts/fx.lua#L564-L569) |
| `tears` | `tears_fx`／`tears`；`tears_fx` | autorotate、fn=FinalOffset1；无 sound。[条目](../../../raw/current/scripts/fx.lua#L572-L577) |
| `spawn_fx_ocean_static` | `pond_splash_fx`／同左；`pond_splash` | 水花声、静态计时，fn 三轴缩放 2 与 FinalOffset1。[条目](../../../raw/current/scripts/fx.lua#L613-L623) |

## 关联机制导航

| 生成者／关系 | 条件、落点及证据 |
|---|---|
| `slurper_respawn` | 暗影鞍破损、啜食者遗迹重生（respawner 未休眠）、麦斯威尔宠物重换皮肤（玩家有效且非 nofx）分别在对应实体位置落烟。[鞍](../../../raw/current/scripts/prefabs/saddle.lua#L17-L25)、[重生](../../../raw/current/scripts/prefabs/slurper.lua#L412-L417)、[换肤](../../../raw/current/scripts/prefabs/waxwell.lua#L87-L97) |
| `pandorachest_reset` | 潘多拉箱与箱怪监听世界事件后进行重置或转换；调用点检查休眠，在箱或原位置落烟，动画与重置声由箱体处理。[箱](../../../raw/current/scripts/prefabs/treasurechest.lua#L461-L475)、[转箱怪](../../../raw/current/scripts/prefabs/treasurechest.lua#L497-L512)、[箱怪](../../../raw/current/scripts/prefabs/chest_mimic.lua#L163-L175)、[追踪器](../../../raw/current/scripts/prefabs/chest_mimic.lua#L530-L536) |
| 洞穴弹飞 | 持续检查时先延迟 2 秒警告，再延迟 2 秒弹飞；死亡、幽灵或 wereplayer 条件可阻断，弹飞还要求洞穴实体有效。FX 落玩家 x/z、y=0，伪受击及 knockback 由洞穴发送。[计时](../../../raw/current/scripts/prefabs/cave_hole.lua#L97-L98)、[警告与弹飞](../../../raw/current/scripts/prefabs/cave_hole.lua#L201-L261) |
| 鼹鼠 | `SGmole` 地下 walk 循环在 0、5、10、15、20、25 帧于自身位置落 `mole_move_fx`。[helper](../../../raw/current/scripts/stategraphs/SGmole.lua#L73-L75)、[状态](../../../raw/current/scripts/stategraphs/SGmole.lua#L329-L346) |
| 表情与眼泪 | 玩家、隐士蟹、附身 WX 状态图按 data.fx 选名、按需延迟；data.fx 非 false 且未指定时用 `emote_fx`。表情数据的 sad 指定 `tears`、延迟 17 帧；舞台剧另直接生成 `emote_fx`。这些路径对代理父挂并绑定符号，不据此保证本地动画持续跟随符号。[数据](../../../raw/current/scripts/emotes.lua#L34-L34)、[玩家](../../../raw/current/scripts/stategraphs/SGwilson.lua#L23981-L23988)、[玩家 helper](../../../raw/current/scripts/stategraphs/SGwilson.lua#L339-L349)、[隐士蟹](../../../raw/current/scripts/stategraphs/SGhermitcrab.lua#L3892-L3902)、[附身 WX](../../../raw/current/scripts/stategraphs/SGwx78_possessedbody.lua#L4388-L4395)、[剧本](../../../raw/current/scripts/play_commonfn.lua#L410-L425) |
| 海洋迁移水花 | `worldmigrator` 对 `oceanwhirlbigportal` ID 选 `spawn_fx_ocean_static` 并随玩家跨世界请求传递；`playerspawner` 消费覆盖名，在玩家位置生成代理并开始 13 帧淡出。映射本身不是直接生成。[映射](../../../raw/current/scripts/components/worldmigrator.lua#L9-L11)、[迁移事件](../../../raw/current/scripts/components/worldmigrator.lua#L198-L208)、[播放](../../../raw/current/scripts/components/playerspawner.lua#L134-L156) |

## 常见陷阱与限制

- 鼹鼠地表动画带 nameoverride 和 description，并非只有视觉字段；依托的地下描述由查看者上下文决定。[工厂](../../../raw/current/scripts/prefabs/fx.lua#L70-L91)
- 眼泪是表情数据动态选名，不能因没有字面 `SpawnPrefab("tears")` 而判无生成点。

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。逐项核验八条 bank/build/anim/sound/fn 与特殊字段；在 `raw/current/scripts` 全树检索完整注册名及 `cavehole`、`emote`、`slurper_respawn`、`spawn_fx_ocean` 前缀，追查动态字段并区分依赖声明。动画素材及模组动态调用未覆盖。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-world-index|天气、地形与生态痕迹]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-06）。
