---
title: "FX · 灯光、火焰与音效代理"
created: "2026-11-07"
updated: "2026-11-07"
type: index
tags: [overview]
---

# FX · 灯光、火焰与音效代理

持续光、可携带灯火、火焰与环境音等表现实体。

## 光源与火焰

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/wormlight|灯具与生物光源的附属光／FX 实体]] | 16 注册：蠕虫灯、矿灯、船灯、红灯笼、女王火炬、哈奇音乐地光与生物光 | `lanternlight`、`miniboatlanternlight`、`slurperlight`、`squideyelight` 等 / active |
| [[reference/prefabs/cavelight_small|光源实体（洞穴、切斯特、读书、出生与中庭）]] | 13 注册：洞穴相位光、变身光柱、读书光与中庭灯 | `cavelight`、`cavelightmoon`、`chesterlight`、`booklight`、`spawnlight_multiplayer`、`atrium_light` / active |
| [[reference/prefabs/fire|火焰、火光与烟雾特效族]] | 15 注册：firefx 等级表、heater 热度与宿主燃烧链 | `campfirefire`、`character_fire`、`floatinglanternfire`、`thurible_smoke` 等 / active |
| [[reference/prefabs/emberlight|火球（emberlight）]] | 薇洛火球术产物：光照供热、烹饪传播与定时熄灭 | 火球、薇洛火球、火球术 / active |
| [[reference/prefabs/nightmarelightfx|暗影光源、幻灵头与盘旋秃鹫 FX 族]] | 8 注册：梦魇灯座/裂缝父挂动画、虚影头跟随与秃鹫盘旋影子 | `gestalt_head`、`circlingbuzzard_lunar` 等 / active |
| [[reference/prefabs/firepit_firebird_puff_fx|带翅膀的火盆添柴烟焰]] | 火盆皮肤添柴烟焰；燃料分档与代理同步 | — / active |
| [[reference/prefabs/lunarflame_puff_small|月焰消退烟团（lunarflame_puff_small）]] | 灭火烟团 12 注册（尺寸×高度）；水晶冠秃鹫进食燃烧尸体生成，复用附身座狼喷焰资源 | lunarflame_puff_tiny、lunarflame_puff_med、lunarflame_puff_large、lunarflame_puff_tiny_low、lunarflame_puff_tiny_high、lunarflame_puff_small_low、lunarflame_puff_small_high、lunarflame_puff_med_low、lunarflame_puff_med_high、lunarflame_puff_large_low、lunarflame_puff_large_high、月焰烟团 / active |

## 声音代理

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/prefabs/grottopool_sfx|洞穴水池声音发射器]] | 本地声音载体、控制器配对及淡变 | 洞穴水池声音 / active |
| [[reference/prefabs/rock_break_fx|岩石碎裂音效代理]] | 岩石碎裂纯音效；网络壳寿命为 1 秒 | — / active |
| [[reference/prefabs/archive_ambient_sfx|远古档案机关声画与基座]] | 5 注册：警戒光球无 AnimState、双层贴地面贴片、环境声区域世界生成铺设、探测仪底座持久实体（arhivepoweron 源码拼写陷阱） | `archive_security_pulse_sfx`、`archive_switch_pad`、`archive_switch_base`、`archive_ambient_sfx`、`archive_resonator_base` / active |

## 关联入口

主体页内的光照变体见 [[reference/prefabs/heatrock#皮肤差异与附属灯|暖石附属灯 heatrocklight]]、[[reference/prefabs/stafflight#光照脉动与消失|矮星光照]]、[[reference/prefabs/staffcoldlight#月台附属变体-staffcoldlightfx|月台附属特效 staffcoldlightfx]]。提灯皮肤光点见 [[reference/prefabs/fx-skin-index|皮肤与装备挂件]]。

返回 [[reference/prefabs/fx-index|FX 主题总索引]]、[[reference/prefabs/index|Prefab 索引]]。
