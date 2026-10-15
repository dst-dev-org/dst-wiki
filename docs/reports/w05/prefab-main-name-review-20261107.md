# Prefab 主名称逐页复检报告

- 检视日期：2026-11-07
- 源码快照：`raw/current/BUILD.txt` = **26335394**
- 固定范围：启动时 `wiki/reference/prefabs/` 下 `type: prefab`、`status: active` 的 353 页；不含 19 个 `type: index` 索引页，也不含本轮开始后并行新增的页面。清单见 [fixed-list.txt](../../../output/w05-prefab-name-review-20261107/fixed-list.txt)。
- 候选证据：[all-pages.json](../../../output/w05-prefab-name-review-20261107/all-pages.json)，由 `inspect_prefab_names.py` 对同一快照的 `languages/chinese_s.po` 批量定位；本报告按 18 批复核。

## 口径

- 以同版本原版简中实显名为准；已有名称证据且标题、族页摘要、正文首次介绍、主要检索别名、对应叶索引行一致时复用，不重复追查。
- 单对象页核对标题主名称与官方名；族页核对族名与成员官方名是否在摘要／首次介绍／aliases／叶索引行可检索。旧称、俗称有依据时保留在 aliases。
- FX 页按现行规则不要求官方 `STRINGS.NAMES`／po 名：标题用注册名＋效果描述，检查标题/别名含注册名、叶索引行存在且指向一致；没有把 FX 的描述名误记为“证据不足”。
- 只修确认影响阅读或检索的问题；未逐条审计普通 aliases、未追查正文次要对象、未重做机制复审。

## 结果

| 状态 | 页数 |
|---|---:|
| 通过 | 351 |
| 已修 | 2 |
| 证据不足 | 0 |
| **合计** | **353** |

上表是固定清单的主名称结论。按用户后续要求，又对全文同类旧译/俗称做了补充清理：另改 20 页并同步相关叶索引，不改变 353 页“通过/已修/证据不足”的主名称判定。

### 第二轮：全文同名旧译补充清理（2026-11-07）

| 范围 | 处理 | 依据 |
|---|---|---|
| 族页成员官方名 | `role-tools` 的 8 件枕头盔甲、8 件手持枕头/剧场服装官方名补入 aliases 与正文表，并更新 `staffs-character-index`；`character-gear-extra` 补“伯尼！”；`living-tools` 补“空鸟嘴壶”，同步 `tools-field-index` | `chinese_s.po` 的 `BODYPILLOW_*`／`HAND_PILLOW_*`／`COSTUME_*`（L339805–339822、L341715–341752、L343715–343732）、`BERNIE_BIG`（L339385–339387）、`PREMIUMWATERINGCAN`（L347280–347282） |
| FX 标题旧译 | `dreadstone_spawn_fx` 恐惧石→绝望石；`fumarole_ember` 查莉→查理；`toadstool_cap_absorbfx` 蛤蟆菇帽→毒菌蟾蜍帽；`wathgrithr_spirit` 武神→薇格弗德；`abigail_attack_shadow_fx` 纯辉→辉煌闪现。标题、H1、摘要/首次介绍、aliases 与对应叶索引同步 | `DREADSTONE`（L342259–342262）、`CHARLIE`（L340789–340792）、`TOADSTOOL_CAP`（L350425–350427）、`WATHGRITHR`（L351874–351877）、`SLINGSHOTAMMO_PUREBRILLIANCE_DEBUFF`／`PUREBRILLIANCE`（L349529–349532、L347384–347387） |
| 全文同名旧译 | 查莉→查理（9 页/索引）、恐惧石→绝望石（armor-rift、hats-armor、wearables-index、dreadstone 页）、纯辉→辉煌闪现/纯粹辉煌（4 页）、蛤蟆菇→毒菌蟾蜍（hats-environment、toadstool 页） | 同上 .po 条目；旧译仅在确有检索价值时保留为 alias |
| 术语旧译 | `abigail_retaliation` 标题/摘要/正文/aliases 与 `fx-status-index`、`wendy_gravestone_rebirth_fx` 链接中的“怨咒”改“惊扰”；标题与 H1 同步 | `STRINGS.SKILLTREE.WENDY.WENDY_SHADOW_2_DESC` 简中“惊扰伤害”（L367266–367270） |
| 补充官方名 alias | `charlie_boss_aoe_flame_fx` 的 `charlie_boss_projectile` 官方名“群聚恐怖”补入 aliases 与 `fx-creature-index` | `CHARLIE_BOSS_PROJECTILE` 名称见 `chinese_s.po` L340819–340822 |
| 事件帽叶索引 | `wearables-index` 的 hats-event 行：鬼魂花帽→幽魂花冠、木偶面具→玩偶面具、鹦鹉帽→波莉·罗杰的帽子、龙舞帽→幸运兽脑袋；摘要“龙舞三件套”改“幸运兽三件套” | `GHOSTFLOWERHAT`（L343204–343207）、`MASK_DOLLHAT`（L345099–345102）、`POLLY_ROGERSHAT`（L347114–347117）、`DRAGON*HAT`（L342189–342252） |

第二轮为 `abigail_attack_shadow_fx`、`dreadstone_spawn_fx`、`fumarole_ember` 三页补入 `languages/chinese_s.po` 并按规则记录基线；其余页 sources 未变。

## 检查点

| 批次 | 范围 | 页数 | 通过 | 已修 | 证据不足 | 检查点 |
|---|---|---:|---:|---:|---:|---|
| B01 | 1–20 | 20 | 20 | 0 | 0 | 已完成 |
| B02 | 21–40 | 20 | 20 | 0 | 0 | 已完成 |
| B03 | 41–60 | 20 | 20 | 0 | 0 | 已完成 |
| B04 | 61–80 | 20 | 20 | 0 | 0 | 已完成 |
| B05 | 81–100 | 20 | 20 | 0 | 0 | 已完成 |
| B06 | 101–120 | 20 | 18 | 2 | 0 | 已完成 |
| B07 | 121–140 | 20 | 20 | 0 | 0 | 已完成 |
| B08 | 141–160 | 20 | 20 | 0 | 0 | 已完成 |
| B09 | 161–180 | 20 | 20 | 0 | 0 | 已完成 |
| B10 | 181–200 | 20 | 20 | 0 | 0 | 已完成 |
| B11 | 201–220 | 20 | 20 | 0 | 0 | 已完成 |
| B12 | 221–240 | 20 | 20 | 0 | 0 | 已完成 |
| B13 | 241–260 | 20 | 20 | 0 | 0 | 已完成 |
| B14 | 261–280 | 20 | 20 | 0 | 0 | 已完成 |
| B15 | 281–300 | 20 | 20 | 0 | 0 | 已完成 |
| B16 | 301–320 | 20 | 20 | 0 | 0 | 已完成 |
| B17 | 321–340 | 20 | 20 | 0 | 0 | 已完成 |
| B18 | 341–353 | 13 | 13 | 0 | 0 | 已完成 |

18 批固定清单逐页留痕；每批完成后继续下一批，未等待确认。

## 逐页结果

### B01（1–20）

| # | 页面 | 主对象／标题 | 状态 | 名称证据与处理 |
|---:|---|---|---|---|
| 1 | [abigail_attack_fx](../../../wiki/reference/prefabs/abigail_attack_fx.md) | 阿比盖尔攻击环特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 2 | [abigail_attack_shadow_fx](../../../wiki/reference/prefabs/abigail_attack_shadow_fx.md) | 温蒂、阿比盖尔与纯辉标记特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 3 | [abigail_retaliation](../../../wiki/reference/prefabs/abigail_retaliation.md) | 阿比盖尔反击、怨咒与暗影增益特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 4 | [abigailsummonfx](../../../wiki/reference/prefabs/abigailsummonfx.md) | 阿比盖尔之花召唤、收回与升级特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 5 | [abysspillar_fx](../../../wiki/reference/prefabs/abysspillar_fx.md) | 深渊石柱坍塌与重建特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 6 | [acidraindrop](../../../wiki/reference/prefabs/acidraindrop.md) | 酸雨落点烟与蘑菇树酸烟 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 7 | [acidsmoke_fx](../../../wiki/reference/prefabs/acidsmoke_fx.md) | 酸雨烟雾特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 8 | [alterguardian_laserscorch](../../../wiki/reference/prefabs/alterguardian_laserscorch.md) | 天体英雄激光、灼烧与地面特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 9 | [alterguardian_spintrail_fx](../../../wiki/reference/prefabs/alterguardian_spintrail_fx.md) | 天体英雄与月脉冲特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 10 | [alterguardian_summon_fx](../../../wiki/reference/prefabs/alterguardian_summon_fx.md) | 天体英雄召唤光柱 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 11 | [amulets](../../../wiki/reference/prefabs/amulets.md) | 宝石护符族 | 通过 | 族页：族名与标题一致；PO 成员官方名 6 个（首个 重生护符）；主要检索别名与叶索引可用 |
| 12 | [archive_ambient_sfx](../../../wiki/reference/prefabs/archive_ambient_sfx.md) | 远古档案机关声画实体 | 通过 | FX：PO 关联名 ARCHIVE_SWITCH_BASE=华丽基座；标题／别名含注册名，叶索引一致 |
| 13 | [archive_chandelier](../../../wiki/reference/prefabs/archive_chandelier.md) | 远古吊灯与石像鬼石化特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 14 | [armor-boss](../../../wiki/reference/prefabs/armor-boss.md) | Boss 与特殊护甲族 | 通过 | 族页：族名与标题一致；PO 成员官方名 4 个（首个 鳞甲）；主要检索别名与叶索引可用 |
| 15 | [armor-classic](../../../wiki/reference/prefabs/armor-classic.md) | 传统身体护甲族 | 通过 | 族页：族名与标题一致；PO 成员官方名 10 个（首个 草甲）；主要检索别名与叶索引可用 |
| 16 | [armor-rift](../../../wiki/reference/prefabs/armor-rift.md) | 裂隙三护甲族 | 通过 | 族页：族名与标题一致；PO 成员官方名 4 个（首个 绝望石盔甲）；主要检索别名与叶索引可用 |
| 17 | [armor_lunarplant_glow_fx](../../../wiki/reference/prefabs/armor_lunarplant_glow_fx.md) | 装备与帽饰跟随光效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 18 | [atrium_gate_activatedfx](../../../wiki/reference/prefabs/atrium_gate_activatedfx.md) | 远古大门充能光层与距离化警示声 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 19 | [attackfx_handpillow_petals](../../../wiki/reference/prefabs/attackfx_handpillow_petals.md) | 枕头大战攻击碎屑与作物挖起表现族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 20 | [attune_out_fx](../../../wiki/reference/prefabs/attune_out_fx.md) | 复活调谐得失特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |

### B02（21–40）

| # | 页面 | 主对象／标题 | 状态 | 名称证据与处理 |
|---:|---|---|---|---|
| 21 | [axe](../../../wiki/reference/prefabs/axe.md) | 斧头族 | 通过 | PO AXE=斧头、GOLDENAXE=黄金斧头、MOONGLASSAXE=月光玻璃斧；标题／摘要／首次介绍／别名／叶索引一致 |
| 22 | [backpack](../../../wiki/reference/prefabs/backpack.md) | 背包 | 通过 | PO BACKPACK=背包；标题／摘要／首次介绍／别名／叶索引一致 |
| 23 | [balloon_held_child](../../../wiki/reference/prefabs/balloon_held_child.md) | 手持气球双层代理与彩带庆典 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 24 | [balloon_pop_body](../../../wiki/reference/prefabs/balloon_pop_body.md) | 气球爆裂 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 25 | [bat_bosshat_fx](../../../wiki/reference/prefabs/bat_bosshat_fx.md) | 猎犬蝙蝠帽层、影分身演出、蝙蝠棒皮肤扇翅与惊树蝠群 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 26 | [batbat](../../../wiki/reference/prefabs/batbat.md) | 蝙蝠棒 | 通过 | PO BATBAT=蝙蝠棒；标题／摘要／首次介绍／别名／叶索引一致 |
| 27 | [bearger_swipe_fx](../../../wiki/reference/prefabs/bearger_swipe_fx.md) | 熊獾挥击地面特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 28 | [beargerfur_sack_frost_fx](../../../wiki/reference/prefabs/beargerfur_sack_frost_fx.md) | 极地熊獾桶霜气与伯尼大火 FX | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 29 | [bee_poof_big](../../../wiki/reference/prefabs/bee_poof_big.md) | 蜂蜜飞溅、蜂群烟与病害蝇群烟 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 30 | [beefalo_transform_fx](../../../wiki/reference/prefabs/beefalo_transform_fx.md) | 皮弗娄牛驯化变身特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 31 | [bile_splash](../../../wiki/reference/prefabs/bile_splash.md) | 胆汁弹溅落 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 32 | [bishop_targeting_fx](../../../wiki/reference/prefabs/bishop_targeting_fx.md) | 发条主教预警瞄准圈 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 33 | [boat_grass_erode](../../../wiki/reference/prefabs/boat_grass_erode.md) | 草／海带船侵蚀与垫料特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 34 | [boat_mast_sink_fx](../../../wiki/reference/prefabs/boat_mast_sink_fx.md) | 船件沉没与受击特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 35 | [boat_water_fx](../../../wiki/reference/prefabs/boat_water_fx.md) | 船尾航迹水花 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 36 | [boatrace_start_flag](../../../wiki/reference/prefabs/boatrace_start_flag.md) | 龙舟竞赛旗帜、烟花与起跑浮标 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 37 | [book_fx](../../../wiki/reference/prefabs/book_fx.md) | 读书施法书本特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 38 | [boomerang](../../../wiki/reference/prefabs/boomerang.md) | 回旋镖 | 通过 | PO BOOMERANG=回旋镖；标题／摘要／首次介绍／别名／叶索引一致 |
| 39 | [bramblefx_armor](../../../wiki/reference/prefabs/bramblefx_armor.md) | 荆棘尖刺反伤与陷阱穿刺 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 40 | [brilliance_projectile_fx](../../../wiki/reference/prefabs/brilliance_projectile_fx.md) | 亮茄魔杖辉光弹 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |

### B03（41–60）

| # | 页面 | 主对象／标题 | 状态 | 名称证据与处理 |
|---:|---|---|---|---|
| 41 | [bullkelp_root](../../../wiki/reference/prefabs/bullkelp_root.md) | 公牛海带茎 | 通过 | PO BULLKELP_ROOT=公牛海带茎；标题／摘要／首次介绍／别名／叶索引一致 |
| 42 | [bundle_unwrap](../../../wiki/reference/prefabs/bundle_unwrap.md) | 开包特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 43 | [burntground](../../../wiki/reference/prefabs/burntground.md) | 焦地渐隐与蜂蜜拖尾减速地面 FX | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 44 | [candybag](../../../wiki/reference/prefabs/candybag.md) | 糖果袋 | 通过 | PO CANDYBAG=糖果袋；标题／摘要／首次介绍／别名／叶索引一致 |
| 45 | [cane](../../../wiki/reference/prefabs/cane.md) | 步行手杖 | 通过 | PO CANE=步行手杖；标题／摘要／首次介绍／别名／叶索引一致 |
| 46 | [cane_ancient_fx](../../../wiki/reference/prefabs/cane_ancient_fx.md) | 远古手杖影子拖尾 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 47 | [cane_candy_fx](../../../wiki/reference/prefabs/cane_candy_fx.md) | 拐杖糖雪花特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 48 | [cane_chess_fx](../../../wiki/reference/prefabs/cane_chess_fx.md) | 城堡手杖棋子拖尾 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 49 | [cane_harlequin_fx](../../../wiki/reference/prefabs/cane_harlequin_fx.md) | 愚人手杖彩纸特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 50 | [cane_rose_fx](../../../wiki/reference/prefabs/cane_rose_fx.md) | 玫瑰红手杖花瓣特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 51 | [cane_sharp_fx](../../../wiki/reference/prefabs/cane_sharp_fx.md) | 腐败手杖紫烟特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 52 | [cane_victorian_fx](../../../wiki/reference/prefabs/cane_victorian_fx.md) | 山羊头手杖闪光特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 53 | [cannon_aoe_range_fx](../../../wiki/reference/prefabs/cannon_aoe_range_fx.md) | 船炮瞄准范围圈特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 54 | [cannonball_used](../../../wiki/reference/prefabs/cannonball_used.md) | 船炮与迫击炮弹命中表现 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 55 | [carnival_confetti_fx](../../../wiki/reference/prefabs/carnival_confetti_fx.md) | 狂欢节庆典特效族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 56 | [carnival_sparkle_bush](../../../wiki/reference/prefabs/carnival_sparkle_bush.md) | 皮肤火星与随身连缀 FX 族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 57 | [cave_vent_ground_fx](../../../wiki/reference/prefabs/cave_vent_ground_fx.md) | 喷气孔地面特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 58 | [cavehole_flick](../../../wiki/reference/prefabs/cavehole_flick.md) | 洞穴弹飞、重置烟与表情效果 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 59 | [cavein_dust_low](../../../wiki/reference/prefabs/cavein_dust_low.md) | 洞穴落石尘土与拾荒疯猪碎装表现族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 60 | [cavelight_small](../../../wiki/reference/prefabs/cavelight_small.md) | 光源实体 | 通过 | FX：PO 关联名 ATRIUM_LIGHT=远古灯柱；标题／别名含注册名，叶索引一致 |

### B04（61–80）

| # | 页面 | 主对象／标题 | 状态 | 名称证据与处理 |
|---:|---|---|---|---|
| 61 | [channel_absorb_fire_fx](../../../wiki/reference/prefabs/channel_absorb_fire_fx.md) | 薇洛打火机吸火引导特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 62 | [character-gear-extra](../../../wiki/reference/prefabs/character-gear-extra.md) | 角色专属装备补遗 | 通过 | 族页：族名与标题一致；PO 成员官方名 7 个（首个 伯尼）；主要检索别名与叶索引可用 |
| 63 | [charlie_boss_aoe_flame_fx](../../../wiki/reference/prefabs/charlie_boss_aoe_flame_fx.md) | 藏影女王 Boss 战表现特效族 | 通过 | FX：PO 关联名 CHARLIE_BOSS_PROJECTILE=群聚恐怖；标题／别名含注册名，叶索引一致 |
| 64 | [charlie_snap](../../../wiki/reference/prefabs/charlie_snap.md) | 暗影拍指 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 65 | [clothing](../../../wiki/reference/prefabs/clothing.md) | BODY 槽衣物与背心族 | 通过 | 族页：族名与标题一致；PO 成员官方名 12 个（首个 充气背心）；主要检索别名与叶索引可用 |
| 66 | [collapse_big](../../../wiki/reference/prefabs/collapse_big.md) | 建筑倒塌特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 67 | [confetti_fx](../../../wiki/reference/prefabs/confetti_fx.md) | 大赛庆祝彩纸 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 68 | [crab_king_bubble1](../../../wiki/reference/prefabs/crab_king_bubble1.md) | 蟹王伴随表现特效族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 69 | [crabking_claw_shadow](../../../wiki/reference/prefabs/crabking_claw_shadow.md) | 地面贴片与投影阴影 FX 族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 70 | [crater_steam_fx1](../../../wiki/reference/prefabs/crater_steam_fx1.md) | 温泉蒸汽 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 71 | [cutless](../../../wiki/reference/prefabs/cutless.md) | 木头短剑 | 通过 | PO CUTLESS=木头短剑；标题／摘要／首次介绍／别名／叶索引一致 |
| 72 | [damp_trail](../../../wiki/reference/prefabs/damp_trail.md) | 地面痕迹与瞬时提示的外围 FX 族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 73 | [daywalker2_swipe_fx](../../../wiki/reference/prefabs/daywalker2_swipe_fx.md) | 拾荒疯猪挥击地面特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 74 | [deer_ice_circle](../../../wiki/reference/prefabs/deer_ice_circle.md) | 无眼鹿冰火法阵 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 75 | [deerclops_icelance_ping_fx](../../../wiki/reference/prefabs/deerclops_icelance_ping_fx.md) | 晶体独眼巨鹿冰圈特效组 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 76 | [deerclops_icespike_fx](../../../wiki/reference/prefabs/deerclops_icespike_fx.md) | 独眼巨鹿冰刺 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 77 | [deerclops_laserscorch](../../../wiki/reference/prefabs/deerclops_laserscorch.md) | 独眼巨鹿激光族：焦痕、烟尾与命中闪光 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 78 | [dirt_puff](../../../wiki/reference/prefabs/dirt_puff.md) | 作物挖掘烟 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 79 | [diseaseflies](../../../wiki/reference/prefabs/diseaseflies.md) | 病害蝇群表现 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 80 | [dr_warm_loop_1](../../../wiki/reference/prefabs/dr_warm_loop_1.md) | 探测杖距离环与 WX-78 特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |

### B05（81–100）

| # | 页面 | 主对象／标题 | 状态 | 名称证据与处理 |
|---:|---|---|---|---|
| 81 | [dreadstone_spawn_fx](../../../wiki/reference/prefabs/dreadstone_spawn_fx.md) | 暗影融合体、恐惧石与诅咒特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 82 | [dumbbells](../../../wiki/reference/prefabs/dumbbells.md) | 哑铃族 | 通过 | 族页：族名与标题一致；PO 成员官方名 7 个（首个 哑铃）；主要检索别名与叶索引可用 |
| 83 | [electricchargedfx](../../../wiki/reference/prefabs/electricchargedfx.md) | 电充闪光特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 84 | [electrocute_fx](../../../wiki/reference/prefabs/electrocute_fx.md) | 电击表现与连锁闪电 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 85 | [emberlight](../../../wiki/reference/prefabs/emberlight.md) | 火球 | 通过 | PO EMBERLIGHT=火球；标题／摘要／首次介绍／别名／叶索引一致 |
| 86 | [event-and-carnival-gear](../../../wiki/reference/prefabs/event-and-carnival-gear.md) | Boss 与狂欢节装备 | 通过 | 族页：族名与标题一致；PO 成员官方名 4 个（首个 派对气球）；主要检索别名与叶索引可用 |
| 87 | [event-weapons](../../../wiki/reference/prefabs/event-weapons.md) | 事件武器族 | 通过 | 族页：族名与标题一致；PO 成员官方名 5 个（首个 兔王棍）；主要检索别名与叶索引可用 |
| 88 | [explode_small](../../../wiki/reference/prefabs/explode_small.md) | 小爆裂、换装烟与火药爆裂 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 89 | [eye_charge](../../../wiki/reference/prefabs/eye_charge.md) | 外围载体：眼弹、诱鱼区、黏着块与速爆弹 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 90 | [eyeofterror_arrive_fx](../../../wiki/reference/prefabs/eyeofterror_arrive_fx.md) | 恐怖之眼降临传送门 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 91 | [fallingswish_clouds](../../../wiki/reference/prefabs/fallingswish_clouds.md) | 坠落云与线条 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 92 | [farm-hoe](../../../wiki/reference/prefabs/farm-hoe.md) | 园艺锄 | 通过 | PO FARM_HOE=园艺锄、GOLDEN_FARM_HOE=黄金园艺锄；标题／摘要／首次介绍／别名／叶索引一致 |
| 93 | [farm_plant_happy](../../../wiki/reference/prefabs/farm_plant_happy.md) | 作物表情与活动彩纸 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 94 | [fence_gate_anim](../../../wiki/reference/prefabs/fence_gate_anim.md) | 宿主挂接的附属表现实体 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 95 | [fire](../../../wiki/reference/prefabs/fire.md) | 火焰、火光与烟雾特效族 | 通过 | FX：PO 关联名 FIRE=火；标题／别名含注册名，叶索引一致 |
| 96 | [firepit_firebird_puff_fx](../../../wiki/reference/prefabs/firepit_firebird_puff_fx.md) | 带翅膀的火盆添柴烟焰 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 97 | [firesplash_fx](../../../wiki/reference/prefabs/firesplash_fx.md) | 龙蝇火系与暗影斩击特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 98 | [fishing-tools](../../../wiki/reference/prefabs/fishing-tools.md) | 钓鱼与水上工具族 | 通过 | 族页：族名与标题一致；PO 成员官方名 8 个（首个 渔网）；主要检索别名与叶索引可用 |
| 99 | [flamethrower_fx](../../../wiki/reference/prefabs/flamethrower_fx.md) | 薇洛暗焰喷射布焰代理 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 100 | [float_fx_front](../../../wiki/reference/prefabs/float_fx_front.md) | 漂浮物前后水环 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |

### B06（101–120）

| # | 页面 | 主对象／标题 | 状态 | 名称证据与处理 |
|---:|---|---|---|---|
| 101 | [flotsam_puddle](../../../wiki/reference/prefabs/flotsam_puddle.md) | 冲滩浮渣伴生水洼与拾取碎裂 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 102 | [forcefieldfx](../../../wiki/reference/prefabs/forcefieldfx.md) | 铥矿皇冠护盾特效 | 已修 | 标题／H1／优先使用／名称行与 `fx-status-index.md` 叶索引由“图勒皇冠”改为官方“铥矿皇冠”；旧译“图勒皇冠”保留 alias |
| 103 | [fossilized_break_fx](../../../wiki/reference/prefabs/fossilized_break_fx.md) | 化石碎裂与化石化特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 104 | [fossilspike_base](../../../wiki/reference/prefabs/fossilspike_base.md) | 化石骨刺地面基座层 | 通过 | FX：PO 关联名 FOSSILSPIKE=化石笼子、FOSSILSPIKE2=化石骨刺；标题／别名含注册名，叶索引一致 |
| 105 | [fumarole-tools](../../../wiki/reference/prefabs/fumarole-tools.md) | 气孔工具族 | 通过 | 族页：族名与标题一致；PO 成员官方名 5 个（首个 炽热斧）；主要检索别名与叶索引可用 |
| 106 | [fumarole_ember](../../../wiki/reference/prefabs/fumarole_ember.md) | 喷气孔与查莉仪式圈特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 107 | [fx_boat_crackle](../../../wiki/reference/prefabs/fx_boat_crackle.md) | 海洋平台破坏开裂与爆裂声画 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 108 | [fx_book_moon](../../../wiki/reference/prefabs/fx_book_moon.md) | 读书施法特效族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 109 | [fx_book_mount](../../../wiki/reference/prefabs/fx_book_mount.md) | 书本挂载特效族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 110 | [gelblob_attach_fx](../../../wiki/reference/prefabs/gelblob_attach_fx.md) | 恶液接触附着黏液 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 111 | [gestalt_trail](../../../wiki/reference/prefabs/gestalt_trail.md) | 宿主附着与跟随的外围 FX 族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 112 | [ghost_transform_overlay_fx](../../../wiki/reference/prefabs/ghost_transform_overlay_fx.md) | 幽灵复活转生光效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 113 | [ghostflower_spirit1_fx](../../../wiki/reference/prefabs/ghostflower_spirit1_fx.md) | 幽灵花灵体飘光 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 114 | [ghostlyelixir_slowregen_fx](../../../wiki/reference/prefabs/ghostlyelixir_slowregen_fx.md) | 幽灵灵药环绕与滴落特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 115 | [giant-crops](../../../wiki/reference/prefabs/giant-crops.md) | 巨型作物 heavy 持握 | 已修 | 补入 PO `CARROT_OVERSIZED_ROTTEN`=巨型腐烂胡萝卜 到 aliases 与 `storage-heavy-index` 叶索引行；标题/摘要/首次介绍保持“巨型作物”族名 |
| 116 | [glass_fx](../../../wiki/reference/prefabs/glass_fx.md) | 玻璃化、灰烬消散与睡眠炸弹爆裂 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 117 | [glasscutter](../../../wiki/reference/prefabs/glasscutter.md) | 玻璃刀 | 通过 | PO GLASSCUTTER=玻璃刀；标题／摘要／首次介绍／别名／叶索引一致 |
| 118 | [gnarwail_horn](../../../wiki/reference/prefabs/gnarwail_horn.md) | 一角鲸的角 | 通过 | PO GNARWAIL_HORN=一角鲸的角；标题／摘要／首次介绍／别名／叶索引一致 |
| 119 | [goosplash](../../../wiki/reference/prefabs/goosplash.md) | 黏液飞溅与贴地溅落 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 120 | [grasspartfx](../../../wiki/reference/prefabs/grasspartfx.md) | 小型一次性外围 FX 族 | 通过 | FX：PO 关联名 CHARLIERESIDUE=黑暗痕迹、CHARLIEROSE=脆弱玫瑰；标题／别名含注册名，叶索引一致 |

### B07（121–140）

| # | 页面 | 主对象／标题 | 状态 | 名称证据与处理 |
|---:|---|---|---|---|
| 121 | [green_leaves](../../../wiki/reference/prefabs/green_leaves.md) | 树叶飘落与砍伐 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 122 | [grottopool_sfx](../../../wiki/reference/prefabs/grottopool_sfx.md) | 洞穴水池瀑布声发射器 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 123 | [groundpound_fx](../../../wiki/reference/prefabs/groundpound_fx.md) | 震地尘土 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 124 | [groundpoundring_fx](../../../wiki/reference/prefabs/groundpoundring_fx.md) | 冲击环、爆点与碎块表现特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 125 | [halloween_firepuff_1](../../../wiki/reference/prefabs/halloween_firepuff_1.md) | 万圣夜火焰绒与月绒烟 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 126 | [hambat](../../../wiki/reference/prefabs/hambat.md) | 火腿棒 | 通过 | PO HAMBAT=火腿棒；标题／摘要／首次介绍／别名／叶索引一致 |
| 127 | [hammer](../../../wiki/reference/prefabs/hammer.md) | 锤子 | 通过 | PO HAMMER=锤子；标题／摘要／首次介绍／别名／叶索引一致 |
| 128 | [hats-armor](../../../wiki/reference/prefabs/hats-armor.md) | 防护与战斗帽族 | 通过 | 族页：族名与标题一致；PO 成员官方名 24 个（首个 启迪之冠）；主要检索别名与叶索引可用 |
| 129 | [hats-environment](../../../wiki/reference/prefabs/hats-environment.md) | 环境与功能帽族 | 通过 | 族页：族名与标题一致；PO 成员官方名 32 个（首个 刮地皮头盔）；主要检索别名与叶索引可用 |
| 130 | [hats-event](../../../wiki/reference/prefabs/hats-event.md) | 事件装扮与角色帽族 | 通过 | 族页：族名与标题一致；PO 成员官方名 28 个（首个 气球帽）；主要检索别名与叶索引可用 |
| 131 | [hats](../../../wiki/reference/prefabs/hats.md) | 帽子跟随层与帽灯特效族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 132 | [heatrock](../../../wiki/reference/prefabs/heatrock.md) | 暖石 | 通过 | PO HEATROCK=暖石；标题／摘要／首次介绍／别名／叶索引一致 |
| 133 | [heavy-items](../../../wiki/reference/prefabs/heavy-items.md) | 重型障碍持握族 | 通过 | PO GLASSBLOCK=玻璃城堡、GLASSSPIKE=玻璃尖刺、OCEANTREENUT=疙瘩树果；标题／摘要／首次介绍／别名／叶索引一致 |
| 134 | [hermitcrab_fx_small](../../../wiki/reference/prefabs/hermitcrab_fx_small.md) | 寄居蟹隐士迁移、传送门与涟漪特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 135 | [hitsparks_fx](../../../wiki/reference/prefabs/hitsparks_fx.md) | 命中火花特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 136 | [hotcold_fx](../../../wiki/reference/prefabs/hotcold_fx.md) | 小惊吓的冷热寻物指示 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 137 | [houndwarning_lvl1](../../../wiki/reference/prefabs/houndwarning_lvl1.md) | 威胁预警音效与暗影窥视表现 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 138 | [ice_crack_grid_fx](../../../wiki/reference/prefabs/ice_crack_grid_fx.md) | 海冰网格裂纹 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 139 | [ice_puddle](../../../wiki/reference/prefabs/ice_puddle.md) | 宿主附属的贴地与贴水 FX 族 | 通过 | FX：PO 关联名 NITRE_FORMATION=硝石堆；标题／别名含注册名，叶索引一致 |
| 140 | [icebox](../../../wiki/reference/prefabs/icebox.md) | 冰箱 | 通过 | PO ICEBOX=冰箱；标题／摘要／首次介绍／别名／叶索引一致 |

### B08（141–160）

| # | 页面 | 主对象／标题 | 状态 | 名称证据与处理 |
|---:|---|---|---|---|
| 141 | [icebox_coffin_bat_fx](../../../wiki/reference/prefabs/icebox_coffin_bat_fx.md) | 冰冷坟墓开箱蝙蝠特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 142 | [icebox_crystal_fx](../../../wiki/reference/prefabs/icebox_crystal_fx.md) | 水晶冰箱持续霜气特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 143 | [icebox_kitchen_fx](../../../wiki/reference/prefabs/icebox_kitchen_fx.md) | 知更鸟蛋冰箱持续霜气特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 144 | [icebox_porcelain_fx](../../../wiki/reference/prefabs/icebox_porcelain_fx.md) | 陶瓷冰箱持续霜气特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 145 | [icebox_victorian_frost_fx](../../../wiki/reference/prefabs/icebox_victorian_frost_fx.md) | 维多利亚冰箱持续霜气特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 146 | [icepack](../../../wiki/reference/prefabs/icepack.md) | 保鲜背包 | 通过 | PO ICEPACK=保鲜背包；标题／摘要／首次介绍／别名／叶索引一致 |
| 147 | [icespike_fx_1](../../../wiki/reference/prefabs/icespike_fx_1.md) | 月矿采掘、旧冰刺与雕像转变 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 148 | [icing_splat_fx](../../../wiki/reference/prefabs/icing_splat_fx.md) | 糖霜溅斑 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 149 | [impact](../../../wiki/reference/prefabs/impact.md) | 命中与落点伤害表现及传送门投影 FX 族 | 通过 | FX：PO 关联名 LUNARRIFT_TERRAFORMER_EXPLOSION=月形爆裂；标题／别名含注册名，叶索引一致 |
| 150 | [ink_puddle_land](../../../wiki/reference/prefabs/ink_puddle_land.md) | 墨水与胆汁洼地 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 151 | [ink_splash](../../../wiki/reference/prefabs/ink_splash.md) | 墨水弹溅落 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 152 | [itemmimic_revealed_shadow](../../../wiki/reference/prefabs/itemmimic_revealed_shadow.md) | 道具拟态怪影尾粒子 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 153 | [junk_break_fx](../../../wiki/reference/prefabs/junk_break_fx.md) | 退化、破裂与改造特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 154 | [king_cane_fx](../../../wiki/reference/prefabs/king_cane_fx.md) | 宿主持有与跟随的附属 FX 族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 155 | [krampus](../../../wiki/reference/prefabs/krampus.md) | 坎普斯 | 通过 | PO KRAMPUS=坎普斯；标题／摘要／首次介绍／别名／叶索引一致 |
| 156 | [krampus_sack](../../../wiki/reference/prefabs/krampus_sack.md) | 坎普斯背包 | 通过 | PO KRAMPUS_SACK=坎普斯背包；标题／摘要／首次介绍／别名／叶索引一致 |
| 157 | [lantern](../../../wiki/reference/prefabs/lantern.md) | 提灯 | 通过 | PO LANTERN=提灯；标题／摘要／首次介绍／别名／叶索引一致 |
| 158 | [lantern_crystal_fx_ground](../../../wiki/reference/prefabs/lantern_crystal_fx_ground.md) | 盐灯持续光点特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 159 | [lantern_flower_fx_held](../../../wiki/reference/prefabs/lantern_flower_fx_held.md) | 纸灯笼花瓣飘落特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 160 | [lantern_gothic_fx_held](../../../wiki/reference/prefabs/lantern_gothic_fx_held.md) | 哥特灯笼光屑飘落特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |

### B09（161–180）

| # | 页面 | 主对象／标题 | 状态 | 名称证据与处理 |
|---:|---|---|---|---|
| 161 | [lantern_insect_fx_held](../../../wiki/reference/prefabs/lantern_insect_fx_held.md) | 蝇光灯光屑飘落特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 162 | [lantern_tesla_fx_held](../../../wiki/reference/prefabs/lantern_tesla_fx_held.md) | 特斯拉线圈提灯电光特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 163 | [lantern_winter_fx_held](../../../wiki/reference/prefabs/lantern_winter_fx_held.md) | 冬季盛宴提灯落雪特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 164 | [lavaarena_beetletaur_fossilized_break_fx_right](../../../wiki/reference/prefabs/lavaarena_beetletaur_fossilized_break_fx_right.md) | 熔炉独眼巨猪与犀牛兄弟化石碎裂特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 165 | [lavaarena_creature_teleport_small_fx](../../../wiki/reference/prefabs/lavaarena_creature_teleport_small_fx.md) | 熔炉生物传送出生特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 166 | [lavaarena_portal_player_fx](../../../wiki/reference/prefabs/lavaarena_portal_player_fx.md) | 熔炉登场、复活与场景余烬特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 167 | [lavae_move_fx](../../../wiki/reference/prefabs/lavae_move_fx.md) | 岩浆虫与哈奇移动拖尾 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 168 | [leafpile_spawn_fx](../../../wiki/reference/prefabs/leafpile_spawn_fx.md) | 落叶堆生成与碎裂特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 169 | [lighter](../../../wiki/reference/prefabs/lighter.md) | 薇洛的打火机 | 通过 | PO LIGHTER=薇洛的打火机；标题／摘要／首次介绍／别名／叶索引一致 |
| 170 | [lightning](../../../wiki/reference/prefabs/lightning.md) | 闪电与雷声表现 | 通过 | FX：PO 关联名 LIGHTNING=闪电；标题／别名含注册名，叶索引一致 |
| 171 | [lightning_rod_fx](../../../wiki/reference/prefabs/lightning_rod_fx.md) | 避雷针电弧动画 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 172 | [living-tools](../../../wiki/reference/prefabs/living-tools.md) | 生活工具族 | 通过 | 族页：族名与标题一致；PO 成员官方名 8 个（首个 刷子）；主要检索别名与叶索引可用 |
| 173 | [lucy](../../../wiki/reference/prefabs/lucy.md) | 露西斧 | 通过 | PO LUCY=露西斧；标题／摘要／首次介绍／别名／叶索引一致 |
| 174 | [lunar_goop_cloud_fx](../../../wiki/reference/prefabs/lunar_goop_cloud_fx.md) | 牧者黏液雾云 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 175 | [lunar_goop_trail_fx](../../../wiki/reference/prefabs/lunar_goop_trail_fx.md) | 牧者黏液拖尾 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 176 | [lunarflame_puff_small](../../../wiki/reference/prefabs/lunarflame_puff_small.md) | 月焰消退烟团 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 177 | [lunarmerm_thorns_fx](../../../wiki/reference/prefabs/lunarmerm_thorns_fx.md) | 鱼人荆棘反伤与翻土标记 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 178 | [lunarplant-tools](../../../wiki/reference/prefabs/lunarplant-tools.md) | 亮茄工具族 | 通过 | PO PICKAXE_LUNARPLANT=亮茄粉碎者、SHOVEL_LUNARPLANT=亮茄锄铲；标题／摘要／首次介绍／别名／叶索引一致 |
| 179 | [lunarrift_crystal_spawn_fx](../../../wiki/reference/prefabs/lunarrift_crystal_spawn_fx.md) | 裂隙晶体生成烟尘 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 180 | [lunarrift_terraformer_visual](../../../wiki/reference/prefabs/lunarrift_terraformer_visual.md) | 裂隙地皮改造视觉层与河灯影层 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |

### B10（181–200）

| # | 页面 | 主对象／标题 | 状态 | 名称证据与处理 |
|---:|---|---|---|---|
| 181 | [madscience_lab_goop](../../../wiki/reference/prefabs/madscience_lab_goop.md) | 实验台三层特效与粒子杂项 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 182 | [marionette_appear_fx](../../../wiki/reference/prefabs/marionette_appear_fx.md) | 舞台剧木偶出场与退场特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 183 | [maxwell_smoke](../../../wiki/reference/prefabs/maxwell_smoke.md) | 麦斯威尔烟 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 184 | [merm_splash](../../../wiki/reference/prefabs/merm_splash.md) | 鱼人水花 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 185 | [messagebottle_break_fx](../../../wiki/reference/prefabs/messagebottle_break_fx.md) | 消息瓶与歌贝特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 186 | [miasma_cloud](../../../wiki/reference/prefabs/miasma_cloud.md) | 瘴气 | 通过 | FX：PO 关联名 MIASMA=瘴气；标题／别名含注册名，叶索引一致 |
| 187 | [mining_fx](../../../wiki/reference/prefabs/mining_fx.md) | 采石碎屑 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 188 | [minotaur_blood1](../../../wiki/reference/prefabs/minotaur_blood1.md) | 沃尔夫冈力量变身与远古守护者血滴 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 189 | [monkey_cursed_pre_fx](../../../wiki/reference/prefabs/monkey_cursed_pre_fx.md) | 猴诅咒与变形特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 190 | [monkeyisland_portal_fxloot](../../../wiki/reference/prefabs/monkeyisland_portal_fxloot.md) | 猴岛传送门假掉落 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 191 | [moon-altar-pieces](../../../wiki/reference/prefabs/moon-altar-pieces.md) | 天体祭坛部件与吸引人的结构 | 通过 | 既有修正，按现状核验：PO 三个 `MOON_ALTAR_ROCK_*` 均为“吸引人的结构”；“岩石包裹”保留 alias |
| 192 | [moon_altar_break](../../../wiki/reference/prefabs/moon_altar_break.md) | 月坛破碎演出 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 193 | [moon_altar_link_fx](../../../wiki/reference/prefabs/moon_altar_link_fx.md) | 月坛连接光效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 194 | [moon_device_pillar](../../../wiki/reference/prefabs/moon_device_pillar.md) | 月装置支柱、顶盖、一阶段破碎与天体裂缝裂纹层 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 195 | [moonstorm_glass_fx](../../../wiki/reference/prefabs/moonstorm_glass_fx.md) | 月之装置破碎与月暴玻璃爆裂 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 196 | [moonstorm_ground_lightning_fx](../../../wiki/reference/prefabs/moonstorm_ground_lightning_fx.md) | 月风暴地面闪电 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 197 | [moose_nest_fx_idle](../../../wiki/reference/prefabs/moose_nest_fx_idle.md) | 麋鹿鹅巢电弧特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 198 | [mossling_spin_fx](../../../wiki/reference/prefabs/mossling_spin_fx.md) | 麋鹿鹅幼崽旋转电光 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 199 | [multitool_axe_pickaxe](../../../wiki/reference/prefabs/multitool_axe_pickaxe.md) | 多用斧镐 | 通过 | PO MULTITOOL_AXE_PICKAXE=多用斧镐；标题／摘要／首次介绍／别名／叶索引一致 |
| 200 | [mushtree_tall_burntfx](../../../wiki/reference/prefabs/mushtree_tall_burntfx.md) | 蘑菇树焦壳与绽放焦壳 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |

### B11（201–220）

| # | 页面 | 主对象／标题 | 状态 | 名称证据与处理 |
|---:|---|---|---|---|
| 201 | [mushtree_tall_webbed_burntfx](../../../wiki/reference/prefabs/mushtree_tall_webbed_burntfx.md) | 生物残骸与植物／菌类次级表现 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 202 | [nightmarelightfx](../../../wiki/reference/prefabs/nightmarelightfx.md) | 暗影光源、幻灵头与盘旋秃鹫的外围 FX 族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 203 | [nightstick](../../../wiki/reference/prefabs/nightstick.md) | 晨星锤 | 通过 | PO NIGHTSTICK=晨星锤；标题／摘要／首次介绍／别名／叶索引一致 |
| 204 | [nightsword](../../../wiki/reference/prefabs/nightsword.md) | 暗夜剑 | 通过 | PO NIGHTSWORD=暗夜剑；标题／摘要／首次介绍／别名／叶索引一致 |
| 205 | [nightsword_cotl_fx](../../../wiki/reference/prefabs/nightsword_cotl_fx.md) | 咩咩之刃攻击烟尘特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 206 | [nightsword_curve_fx](../../../wiki/reference/prefabs/nightsword_curve_fx.md) | 扭曲匕首攻击火花特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 207 | [nightsword_lightsbane_fx](../../../wiki/reference/prefabs/nightsword_lightsbane_fx.md) | 魔光剑攻击像素粒子特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 208 | [nightsword_sharp_fx](../../../wiki/reference/prefabs/nightsword_sharp_fx.md) | 腐败刀锋攻击雪粒特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 209 | [nightsword_wizard_fx](../../../wiki/reference/prefabs/nightsword_wizard_fx.md) | 诅咒之剑攻击暗影微粒特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 210 | [ocean_splash_med1](../../../wiki/reference/prefabs/ocean_splash_med1.md) | 海面水花特效族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 211 | [oceanhorror_ripples](../../../wiki/reference/prefabs/oceanhorror_ripples.md) | 海上暗影生物的水面涟漪与船只挂点 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 212 | [oceantree_ripples_short](../../../wiki/reference/prefabs/oceantree_ripples_short.md) | 海树波纹、根须、倒伏与落叶表现 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 213 | [oldager_become_younger_front_fx](../../../wiki/reference/prefabs/oldager_become_younger_front_fx.md) | 年龄转换特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 214 | [opalstaff](../../../wiki/reference/prefabs/opalstaff.md) | 唤月者魔杖 | 通过 | PO OPALSTAFF=唤月者魔杖；标题／摘要／首次介绍／别名／叶索引一致 |
| 215 | [petrified_trunk_break_fx](../../../wiki/reference/prefabs/petrified_trunk_break_fx.md) | 树木石化特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 216 | [pickaxe](../../../wiki/reference/prefabs/pickaxe.md) | 鹤嘴锄 | 通过 | PO GOLDENPICKAXE=黄金鹤嘴锄、PICKAXE=鹤嘴锄；标题／摘要／首次介绍／别名／叶索引一致 |
| 217 | [piggyback](../../../wiki/reference/prefabs/piggyback.md) | 猪皮包 | 通过 | PO PIGGYBACK=猪皮包；标题／摘要／首次介绍／别名／叶索引一致 |
| 218 | [planar_hit_fx](../../../wiki/reference/prefabs/planar_hit_fx.md) | 位面命中与抵抗表现 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 219 | [player_float_hop_water_fx](../../../wiki/reference/prefabs/player_float_hop_water_fx.md) | 玩家出水水环 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 220 | [pocketwatch](../../../wiki/reference/prefabs/pocketwatch.md) | 旺达怀表族 | 通过 | 族页：族名与标题一致；PO 成员官方名 11 个（首个 钟表匠工具）；主要检索别名与叶索引可用 |

### B12（221–240）

| # | 页面 | 主对象／标题 | 状态 | 名称证据与处理 |
|---:|---|---|---|---|
| 221 | [pocketwatch_warp_marker](../../../wiki/reference/prefabs/pocketwatch_warp_marker.md) | 怀表与传送门附属表现 FX 族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 222 | [pocketwatch_weapon_fx](../../../wiki/reference/prefabs/pocketwatch_weapon_fx.md) | 警钟暗影微粒特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 223 | [positronbeam_back](../../../wiki/reference/prefabs/positronbeam_back.md) | 月亮石充能光束与光柱 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 224 | [pouch-bags](../../../wiki/reference/prefabs/pouch-bags.md) | 小袋与专用袋族 | 通过 | PO POTATOSACK=土豆袋、SEEDPOUCH=种子袋、SPICEPACK=厨师袋；标题／摘要／首次介绍／别名／叶索引一致 |
| 225 | [pumpkincarving_shatter_fx](../../../wiki/reference/prefabs/pumpkincarving_shatter_fx.md) | 南瓜雕刻碎裂动画 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 226 | [pumpkincarving_swap_fx](../../../wiki/reference/prefabs/pumpkincarving_swap_fx.md) | 南瓜雕刻换装代理 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 227 | [quagmire_portal_player_fx](../../../wiki/reference/prefabs/quagmire_portal_player_fx.md) | 暴食远古大门玩家出场与撒盐特效族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 228 | [queen-torch](../../../wiki/reference/prefabs/queen-torch.md) | 女王火炬 | 通过 | PO QUEEN_TORCH=女王火炬；标题／摘要／首次介绍／别名／叶索引一致 |
| 229 | [rabbithouse_yule_glow_fx](../../../wiki/reference/prefabs/rabbithouse_yule_glow_fx.md) | 倒挂树屋光晕特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 230 | [rain](../../../wiki/reference/prefabs/rain.md) | 天气降水表现特效 | 通过 | FX：PO 关联名 RAIN=短路；标题／别名含注册名，叶索引一致 |
| 231 | [raindrop](../../../wiki/reference/prefabs/raindrop.md) | 落点水花与呼气粒子 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 232 | [ranged-weapons](../../../wiki/reference/prefabs/ranged-weapons.md) | 远程手持武器族 | 通过 | 族页：族名与标题一致；PO 成员官方名 8 个（首个 火焰吹箭）；主要检索别名与叶索引可用 |
| 233 | [redlantern](../../../wiki/reference/prefabs/redlantern.md) | 红灯笼 | 通过 | PO REDLANTERN=红灯笼；标题／摘要／首次介绍／别名／叶索引一致 |
| 234 | [reticuleaoe](../../../wiki/reference/prefabs/reticuleaoe.md) | 瞄准圈、准星与 AoE 目标指示特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 235 | [reviver_cupid_beat_fx](../../../wiki/reference/prefabs/reviver_cupid_beat_fx.md) | 告密的心丘比特皮肤心跳特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 236 | [rock_break_fx](../../../wiki/reference/prefabs/rock_break_fx.md) | 岩石碎裂音效代理 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 237 | [rocky_snip_fx](../../../wiki/reference/prefabs/rocky_snip_fx.md) | 岩甲 Boss 剪击冲刺特效与枯萎变体 | 通过 | FX：PO 关联名 ROCKY_BOSS_SHADOW=枯萎岩石大师；标题／别名含注册名，叶索引一致 |
| 238 | [role-tools](../../../wiki/reference/prefabs/role-tools.md) | 角色与活动专用工具族 | 通过 | 族页：族名与标题一致；PO 成员官方名 25 个（首个 气球）；主要检索别名与叶索引可用 |
| 239 | [rope_bridge_fx](../../../wiki/reference/prefabs/rope_bridge_fx.md) | 绳桥与藤桥的桥面表现 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 240 | [rose_petals_fx](../../../wiki/reference/prefabs/rose_petals_fx.md) | 玫瑰花瓣飘落 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |

### B13（241–260）

| # | 页面 | 主对象／标题 | 状态 | 名称证据与处理 |
|---:|---|---|---|---|
| 241 | [round_puff_fx_sm](../../../wiki/reference/prefabs/round_puff_fx_sm.md) | 通用圆烟、滑行烟与木屑 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 242 | [ruins-bat](../../../wiki/reference/prefabs/ruins-bat.md) | 铥矿棒 | 通过 | PO RUINS_BAT=铥矿棒；标题／摘要／首次介绍／别名／叶索引一致 |
| 243 | [ruins_cavein_obstacle_rubble_fx](../../../wiki/reference/prefabs/ruins_cavein_obstacle_rubble_fx.md) | 废墟落石碎屑与炼金打印舱皮肤 FX | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 244 | [ruinsnightmare_horn_attack](../../../wiki/reference/prefabs/ruinsnightmare_horn_attack.md) | 潜伏梦魇穿地犄角突袭 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 245 | [saddle_shadow_fx](../../../wiki/reference/prefabs/saddle_shadow_fx.md) | 梦魇鞍具的跟随特效与暗影脚印 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 246 | [sand_puff](../../../wiki/reference/prefabs/sand_puff.md) | 沙尘与暗影烟团 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 247 | [sanity_raise](../../../wiki/reference/prefabs/sanity_raise.md) | 影岩升降标记动画 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 248 | [scorchedground](../../../wiki/reference/prefabs/scorchedground.md) | 地面焦痕、跟随换装与附着状态的外围 FX 族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 249 | [shadow-battleaxe](../../../wiki/reference/prefabs/shadow-battleaxe.md) | 暗影槌 | 通过 | PO SHADOW_BATTLEAXE=暗影槌；标题／摘要／首次介绍／别名／叶索引一致 |
| 250 | [shadow_bishop_fx](../../../wiki/reference/prefabs/shadow_bishop_fx.md) | 暗影主教羽群、裂隙门粒子与墨荒替身／附体特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 251 | [shadow_despawn](../../../wiki/reference/prefabs/shadow_despawn.md) | 暗影消散、舞台影手与暗影憎恶光束的外围 FX 族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 252 | [shadow_glob_fx](../../../wiki/reference/prefabs/shadow_glob_fx.md) | 暗影地面痕特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 253 | [shadow_pillar](../../../wiki/reference/prefabs/shadow_pillar.md) | 暗影囚牢·影柱法术体系 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 254 | [shadow_shield1](../../../wiki/reference/prefabs/shadow_shield1.md) | 骨头盔甲格挡护盾 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 255 | [shadow_teleport_in](../../../wiki/reference/prefabs/shadow_teleport_in.md) | 暗影传送、孢子咳出与宝盒玩家特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 256 | [shadow_trap](../../../wiki/reference/prefabs/shadow_trap.md) | 暗影陷阱与减速表现 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 257 | [shadowhand_arm](../../../wiki/reference/prefabs/shadowhand_arm.md) | 暗影之手与恐惧之手的伸缩手臂 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 258 | [shadowthrall_projectile_fx](../../../wiki/reference/prefabs/shadowthrall_projectile_fx.md) | 墨荒弹幕投射物 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 259 | [sharkboi_swipe_fx](../../../wiki/reference/prefabs/sharkboi_swipe_fx.md) | 大霜鲨攻击冰面特效族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 260 | [shatter](../../../wiki/reference/prefabs/shatter.md) | 一次性碎裂与爆裂表现族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |

### B14（261–280）

| # | 页面 | 主对象／标题 | 状态 | 名称证据与处理 |
|---:|---|---|---|---|
| 261 | [shock_fx](../../../wiki/reference/prefabs/shock_fx.md) | 电击附着动画 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 262 | [shovel](../../../wiki/reference/prefabs/shovel.md) | 铲子与黄金铲子 | 通过 | PO GOLDENSHOVEL=黄金铲子、SHOVEL=铲子；标题／摘要／首次介绍／别名／叶索引一致 |
| 263 | [shovel_dirt](../../../wiki/reference/prefabs/shovel_dirt.md) | 铲土飞尘 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 264 | [shrouden_eye_fx](../../../wiki/reference/prefabs/shrouden_eye_fx.md) | 暗影憎恶死亡水洼之眼 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 265 | [sinkhole_spawn_fx](../../../wiki/reference/prefabs/sinkhole_spawn_fx.md) | 地陷尘爆与落石预警族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 266 | [sleepcloud](../../../wiki/reference/prefabs/sleepcloud.md) | 睡眠云与孢子云 | 通过 | FX：PO 关联名 SPORECLOUD=孢子云；标题／别名含注册名，叶索引一致 |
| 267 | [slingshot_ice_aoe_fx](../../../wiki/reference/prefabs/slingshot_ice_aoe_fx.md) | 弹弓 AOE 命中圈 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 268 | [slingshotammo_hitfx_rock](../../../wiki/reference/prefabs/slingshotammo_hitfx_rock.md) | 弹弓弹药命中表现 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 269 | [slingshotammo_purebrilliance_debuff_fx](../../../wiki/reference/prefabs/slingshotammo_purebrilliance_debuff_fx.md) | 弹弓元素弹药附带表现 | 通过 | FX：PO 关联名 SLINGSHOTAMMO_HORRORFUEL_DEBUFF_FX=群聚恐怖、SLINGSHOTAMMO_PUREBRILLIANCE_DEBUFF=辉煌闪现；标题／别名含注册名，叶索引一致 |
| 270 | [small_puff](../../../wiki/reference/prefabs/small_puff.md) | 变身与出生烟 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 271 | [snowball_shatter_fx](../../../wiki/reference/prefabs/snowball_shatter_fx.md) | 雪球碎裂动画 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 272 | [sparks](../../../wiki/reference/prefabs/sparks.md) | 电击命中火花 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 273 | [spat_splat_fx](../../../wiki/reference/prefabs/spat_splat_fx.md) | 唾沫溅斑 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 274 | [spawn_fx_small](../../../wiki/reference/prefabs/spawn_fx_small.md) | 生成烟雾 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 275 | [spear](../../../wiki/reference/prefabs/spear.md) | 长矛 | 通过 | PO SPEAR=长矛；标题／摘要／首次介绍／别名／叶索引一致 |
| 276 | [spear_gungnir_lungefx](../../../wiki/reference/prefabs/spear_gungnir_lungefx.md) | 突刺与装备跟随电光 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 277 | [spell_fire_throw](../../../wiki/reference/prefabs/spell_fire_throw.md) | 火焰失败、月化结束与暗影火爆裂 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 278 | [spellmasterybuff](../../../wiki/reference/prefabs/spellmasterybuff.md) | 法术精通 buff 与法球 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 279 | [spider_heal_fx](../../../wiki/reference/prefabs/spider_heal_fx.md) | 蜘蛛治疗与突变 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 280 | [splash](../../../wiki/reference/prefabs/splash.md) | 落水水花 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |

### B15（281–300）

| # | 页面 | 主对象／标题 | 状态 | 名称证据与处理 |
|---:|---|---|---|---|
| 281 | [splash_green](../../../wiki/reference/prefabs/splash_green.md) | 海面绿花与冲滩水洼 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 282 | [splash_sink](../../../wiki/reference/prefabs/splash_sink.md) | 沉没与单点水花 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 283 | [spooked_spider_rock_fx](../../../wiki/reference/prefabs/spooked_spider_rock_fx.md) | 惊吓蜘蛛虫群特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 284 | [staff_castinglight](../../../wiki/reference/prefabs/staff_castinglight.md) | 施法光源 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 285 | [staff_lunarplant_fx](../../../wiki/reference/prefabs/staff_lunarplant_fx.md) | 装备与物品持握跟随层 FX 族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 286 | [staffcastfx](../../../wiki/reference/prefabs/staffcastfx.md) | 施法与怀表回溯表现 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 287 | [staffcoldlight](../../../wiki/reference/prefabs/staffcoldlight.md) | 极光 | 通过 | PO STAFFCOLDLIGHT=极光；标题／摘要／首次介绍／别名／叶索引一致 |
| 288 | [stafflight](../../../wiki/reference/prefabs/stafflight.md) | 矮星 | 通过 | PO STAFFLIGHT=矮星；标题／摘要／首次介绍／别名／叶索引一致 |
| 289 | [staffs-more](../../../wiki/reference/prefabs/staffs-more.md) | 法杖补充族 | 通过 | 族页：族名与标题一致；PO 成员官方名 10 个（首个 国王魔杖）；主要检索别名与叶索引可用 |
| 290 | [staffs](../../../wiki/reference/prefabs/staffs.md) | 基础法杖族 | 通过 | 族页：族名与标题一致；PO 成员官方名 7 个（首个 火魔杖）；主要检索别名与叶索引可用 |
| 291 | [stalker_shield1](../../../wiki/reference/prefabs/stalker_shield1.md) | 复活的骨架骨盾格挡 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 292 | [sugarwood_leaf_fx](../../../wiki/reference/prefabs/sugarwood_leaf_fx.md) | 特殊树叶与死亡烟 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 293 | [superjump_fx](../../../wiki/reference/prefabs/superjump_fx.md) | 超级跳落点特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 294 | [sword_lunarplant](../../../wiki/reference/prefabs/sword_lunarplant.md) | 亮茄剑 | 通过 | PO SWORD_LUNARPLANT=亮茄剑；标题／摘要／首次介绍／别名／叶索引一致 |
| 295 | [tentaclespike](../../../wiki/reference/prefabs/tentaclespike.md) | 触手尖刺 | 通过 | PO TENTACLESPIKE=触手尖刺；标题／摘要／首次介绍／别名／叶索引一致 |
| 296 | [terrarium_fx](../../../wiki/reference/prefabs/terrarium_fx.md) | 盒中泰拉召唤光柱 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 297 | [terrariumchest_fx](../../../wiki/reference/prefabs/terrariumchest_fx.md) | 显眼箱子微光与传送塔沙柱 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 298 | [throwables](../../../wiki/reference/prefabs/throwables.md) | 投掷炸弹与雪球 | 通过 | 族页：族名与标题一致；PO 成员官方名 4 个（首个 亮茄炸弹）；主要检索别名与叶索引可用 |
| 299 | [toadstool_cap_absorbfx](../../../wiki/reference/prefabs/toadstool_cap_absorbfx.md) | 蛤蟆菇帽毒雾吸收与释放 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 300 | [tools-misc](../../../wiki/reference/prefabs/tools-misc.md) | 探具与仪式工具族 | 通过 | 族页：族名与标题一致；PO 成员官方名 5 个（首个 指南针）；主要检索别名与叶索引可用 |

### B16（301–320）

| # | 页面 | 主对象／标题 | 状态 | 名称证据与处理 |
|---:|---|---|---|---|
| 301 | [tophat_shadow_fx](../../../wiki/reference/prefabs/tophat_shadow_fx.md) | 魔术师高礼帽影子特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 302 | [torch](../../../wiki/reference/prefabs/torch.md) | 火炬 | 通过 | PO TORCH=火炬；标题／摘要／首次介绍／别名／叶索引一致 |
| 303 | [trap_fumarole_burn_fx](../../../wiki/reference/prefabs/trap_fumarole_burn_fx.md) | 喷气孔陷阱贴身灼烧与藤蔓陷阱底座 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 304 | [trident](../../../wiki/reference/prefabs/trident.md) | 刺耳三叉戟 | 通过 | PO TRIDENT=刺耳三叉戟；标题／摘要／首次介绍／别名／叶索引一致 |
| 305 | [umbrella](../../../wiki/reference/prefabs/umbrella.md) | 伞族 | 通过 | 族页：族名与标题一致；PO 成员官方名 4 个（首个 花伞）；主要检索别名与叶索引可用 |
| 306 | [utility-and-throwables](../../../wiki/reference/prefabs/utility-and-throwables.md) | 实用与投掷装备 | 通过 | 族页：族名与标题一致；PO 成员官方名 5 个（首个 恶液罐）；主要检索别名与叶索引可用 |
| 307 | [vault_compass_visual](../../../wiki/reference/prefabs/vault_compass_visual.md) | 宝库设施表现族：路标指针、净化雾、地纹贴片与冷火 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 308 | [vault_pillar_guard_swipe_fx](../../../wiki/reference/prefabs/vault_pillar_guard_swipe_fx.md) | 远古戍卫塔挥击与砸地特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 309 | [voidcloth_boomerang](../../../wiki/reference/prefabs/voidcloth_boomerang.md) | 阴郁回旋镖 | 通过 | PO VOIDCLOTH_BOOMERANG=阴郁回旋镖；标题／摘要／首次介绍／别名／叶索引一致 |
| 310 | [voidcloth_boomerang_fx](../../../wiki/reference/prefabs/voidcloth_boomerang_fx.md) | 虚布武器持握层与回旋镖投射物 | 通过 | FX：PO 关联名 VOIDCLOTH_SCYTHE=暗影收割者；标题／别名含注册名，叶索引一致 |
| 311 | [voidcloth_boomerang_launch_fx](../../../wiki/reference/prefabs/voidcloth_boomerang_launch_fx.md) | 虚布回旋镖与皮弗娄牛复活雷光 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 312 | [voidcloth_scythe](../../../wiki/reference/prefabs/voidcloth_scythe.md) | 暗影收割者 | 通过 | PO VOIDCLOTH_SCYTHE=暗影收割者；标题／摘要／首次介绍／别名／叶索引一致 |
| 313 | [wagboss_beam_fx](../../../wiki/reference/prefabs/wagboss_beam_fx.md) | W.A.R.B.O.T. 轨道光束与机械 Boss 战伴随特效族 | 通过 | FX：PO 关联名 ALTERGUARDIAN_CONTAINED=月亮精华提取器；标题／别名含注册名，叶索引一致 |
| 314 | [wagboss_missile_target_fx](../../../wiki/reference/prefabs/wagboss_missile_target_fx.md) | WAGboss 导弹落点与无人机电束电球 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 315 | [wagdrone_rolling_collide_small_fx](../../../wiki/reference/prefabs/wagdrone_rolling_collide_small_fx.md) | 滚动无人机碰撞火花与灵药护盾 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 316 | [wagpunksteam_hat_up](../../../wiki/reference/prefabs/wagpunksteam_hat_up.md) | WAGpunk 装备蒸汽 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 317 | [wanda_attack_pocketwatch_old_fx](../../../wiki/reference/prefabs/wanda_attack_pocketwatch_old_fx.md) | 旺达怀表攻击与治疗特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 318 | [warg_mutated_breath_fx](../../../wiki/reference/prefabs/warg_mutated_breath_fx.md) | 附身座狼喷焰与灼烬 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 319 | [waterballoon_splash](../../../wiki/reference/prefabs/waterballoon_splash.md) | 水球溅落 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 320 | [waterplant_destroy](../../../wiki/reference/prefabs/waterplant_destroy.md) | 暗影手与海上水植特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |

### B17（321–340）

| # | 页面 | 主对象／标题 | 状态 | 名称证据与处理 |
|---:|---|---|---|---|
| 321 | [waterplant_pollen_fx](../../../wiki/reference/prefabs/waterplant_pollen_fx.md) | 海草花粉喷云与海面诱饵云 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 322 | [wathgrithr_shield](../../../wiki/reference/prefabs/wathgrithr_shield.md) | 战斗圆盾 | 通过 | PO WATHGRITHR_SHIELD=战斗圆盾；标题／摘要／首次介绍／别名／叶索引一致 |
| 323 | [wathgrithr_spirit](../../../wiki/reference/prefabs/wathgrithr_spirit.md) | 武神之魂与战歌特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 324 | [wave_med](../../../wiki/reference/prefabs/wave_med.md) | 海面波浪与浪花 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 325 | [weaponsparks](../../../wiki/reference/prefabs/weaponsparks.md) | 武器命中火花 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 326 | [wendy_gravestone_rebirth_fx](../../../wiki/reference/prefabs/wendy_gravestone_rebirth_fx.md) | 温蒂墓碑重生双幽灵动画 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 327 | [werebeaver_transform_fx](../../../wiki/reference/prefabs/werebeaver_transform_fx.md) | 变身切换烟 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 328 | [weregoose_splash](../../../wiki/reference/prefabs/weregoose_splash.md) | 鹅形态水花与羽毛 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 329 | [weremoose_smash_fx](../../../wiki/reference/prefabs/weremoose_smash_fx.md) | 伍迪麋鹿形态重击特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 330 | [whip](../../../wiki/reference/prefabs/whip.md) | 三尾猫鞭 | 通过 | PO WHIP=三尾猫鞭；标题／摘要／首次介绍／别名／叶索引一致 |
| 331 | [willow_shadow_flame](../../../wiki/reference/prefabs/willow_shadow_flame.md) | 薇洛暗焰 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 332 | [winona_battery_high_shatterfx](../../../wiki/reference/prefabs/winona_battery_high_shatterfx.md) | 薇诺娜宝石发电机设备特效族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 333 | [winters_feast_depletefood](../../../wiki/reference/prefabs/winters_feast_depletefood.md) | 冬季盛宴餐桌食物枯竭特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 334 | [wintersfeastbuff_fx](../../../wiki/reference/prefabs/wintersfeastbuff_fx.md) | 冬季盛宴增益粒子与烤炉火焰 FX | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 335 | [witchbroom_fx](../../../wiki/reference/prefabs/witchbroom_fx.md) | 活扫帚轨迹、沃比烘干架与训练哨增益 FX | 通过 | FX：PO 关联名 WITCHBROOM=女巫扫帚；标题／别名含注册名，叶索引一致 |
| 336 | [woby_dash_shadow_fx](../../../wiki/reference/prefabs/woby_dash_shadow_fx.md) | 沃比暗影冲刺特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 337 | [wormlight](../../../wiki/reference/prefabs/wormlight.md) | 灯具与生物光源的附属光／FX 实体 | 通过 | FX：PO 关联名 WORMLIGHT=发光浆果、WORMLIGHT_LESSER=小发光浆果；标题／别名含注册名，叶索引一致 |
| 338 | [wormwood_plant_fx](../../../wiki/reference/prefabs/wormwood_plant_fx.md) | 沃姆伍德开花植物特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 339 | [wortox-nabbag](../../../wiki/reference/prefabs/wortox-nabbag.md) | 强抢袋 | 通过 | PO WORTOX_NABBAG=强抢袋；标题／摘要／首次介绍／别名／叶索引一致 |
| 340 | [wortox_eat_soul_fx](../../../wiki/reference/prefabs/wortox_eat_soul_fx.md) | 进食灵魂特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |

### B18（341–353）

| # | 页面 | 主对象／标题 | 状态 | 名称证据与处理 |
|---:|---|---|---|---|
| 341 | [wortox_portal_fx](../../../wiki/reference/prefabs/wortox_portal_fx.md) | 灵魂跳跃出入场特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 342 | [wortox_soul_heal_fx](../../../wiki/reference/prefabs/wortox_soul_heal_fx.md) | 灵魂治疗特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 343 | [wortox_soul_in_fx](../../../wiki/reference/prefabs/wortox_soul_in_fx.md) | 灵魂入体特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 344 | [wortox_soul_spawn_fx](../../../wiki/reference/prefabs/wortox_soul_spawn_fx.md) | 沃拓克斯传送、灵魂与分身特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 345 | [wortox_soulecho_buff_fx](../../../wiki/reference/prefabs/wortox_soulecho_buff_fx.md) | 沃拓克斯灵魂增益与装备跟随特效族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 346 | [wurt_casting_fx](../../../wiki/reference/prefabs/wurt_casting_fx.md) | 沃特施法特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 347 | [wurt_shadow_merm_planar_fx](../../../wiki/reference/prefabs/wurt_shadow_merm_planar_fx.md) | 沃特位面增益与投泥带外围特效族 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 348 | [wurt_terraformer_fx_shadow](../../../wiki/reference/prefabs/wurt_terraformer_fx_shadow.md) | 沃特沼泽改造、水盾与暗影鱼人特效 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 349 | [wx78_big_spark](../../../wiki/reference/prefabs/wx78_big_spark.md) | WX-78 电火花、照明光柱与扫描底盘特效族 | 通过 | FX：PO 关联名 WX78_BACKUPBODY_INVENTORY=备份底盘；标题／别名含注册名，叶索引一致 |
| 350 | [wx78_shield_full](../../../wiki/reference/prefabs/wx78_shield_full.md) | WX78 护盾动画 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 351 | [wx78_taser_projectile_fx](../../../wiki/reference/prefabs/wx78_taser_projectile_fx.md) | WX-78 电击蓄能爆发 | 通过 | FX：未定位到本注册名的 NAMES 键；标题／别名含注册名，叶索引一致 |
| 352 | [yellowstaff](../../../wiki/reference/prefabs/yellowstaff.md) | 唤星者魔杖 | 通过 | PO YELLOWSTAFF=唤星者魔杖；标题／摘要／首次介绍／别名／叶索引一致 |
| 353 | [yotr_fightring_torch](../../../wiki/reference/prefabs/yotr_fightring_torch.md) | 枕头大战坑与兔人火炬 | 通过 | FX：PO 关联名 YOTR_FIGHTRING=枕头大战坑、YOTR_FIGHTRING_BELL=枕头大战钟；标题／别名含注册名，叶索引一致 |

## 已修详情

### forcefieldfx：标题与叶索引旧译名改回官方名

- 位置：[forcefieldfx.md](../../../wiki/reference/prefabs/forcefieldfx.md) 的 `title`、H1、“优先使用”和“名称”行；[fx-status-index.md](../../../wiki/reference/prefabs/fx-status-index.md#L30) 叶索引行。
- 依据：`STRINGS.NAMES.RUINSHAT` 在 `raw/current/scripts/languages/chinese_s.po` 第 348814–348817 行为“铥矿皇冠”；本页 `FORCEFIELDFX` 无 `STRINGS.NAMES.*` 键，旧标题“图勒皇冠护盾特效”使用旧译“图勒”，与玩家看到的宿主名不一致。
- 处理：标题／H1／“优先使用”／“名称”行与叶索引行改为“铥矿皇冠护盾特效”；旧译“图勒皇冠护盾特效”保留在 aliases 作检索旧称。只改名称相关行，未改机制、sources 或基线。
- 校验：`validate_page.py --check-indexes` 对页面与索引 `passed: true`；`validate_source_links.py --strict` 对页面 0 issue；`inspect_prefab_names.py --reuse` 复核无新增名称候选。

### giant-crops：补入腐烂变体的官方名

- 位置：[giant-crops.md](../../../wiki/reference/prefabs/giant-crops.md) 的 aliases；[storage-heavy-index.md](../../../wiki/reference/prefabs/storage-heavy-index.md#L29) 叶索引行。
- 依据：`STRINGS.NAMES.CARROT_OVERSIZED_ROTTEN` 在 `raw/current/scripts/languages/chinese_s.po` 第 340670–340672 行为“巨型腐烂胡萝卜”；页面标题明确覆盖 oversized／waxed／rotten 三变体，原 aliases 有“巨型胡萝卜”与 `carrot_oversized_rotten`，但缺腐烂变体的简中名，精确搜索“巨型腐烂胡萝卜”无命中。
- 处理：aliases 与叶索引行补“巨型腐烂胡萝卜”；未改族名、机制、sources 与基线，未逐条补其他族页成员别名。
- 校验：`validate_page.py --check-indexes` 对页面与索引 `passed: true`；`validate_source_links.py --strict` 对页面 0 issue。
- 第二轮的 19 页与相关索引逐项处理见上方“第二轮”表。

## 校验（第二轮后）

- `validate_page.py --check-indexes`：22 个改动页 + 9 个受影响叶索引 `passed: true`，0 issues，导航检查通过。
- `validate_source_links.py --strict`：22 个改动页 0 error／0 warning。
- `validate_source_hashes.py check`：22 个改动页 0 affected／0 unrecorded；3 个新增 `.po` 来源的页面已 `record`。
- 报告自身：353 行逐页结果齐全，359 个本地链接全部可解析。

## 证据不足

无。353 页均有结论：75 个非 FX 页均有至少 1 个 `STRINGS.NAMES.*` 候选并核对至标题／摘要／首次介绍／别名／叶索引；278 个 FX 页按现行规则不要求官方 NAMES 键，已逐页核对注册名与效果描述、别名和叶索引行。

## 保留旧称（不视为缺口）

- “女武神”“木偶”“图勒”等：属于已建立的俗称/旧译，继续保留在 aliases 或正文描述中，不作为标题主名（`wathgrithr_spirit` 标题已改“薇格弗德”，`ruins-bat` 标题/主名仍“铥矿棒”，`marionette_appear_fx` 的“木偶”是效果描述）。

## 已知未纳入（不影响本轮主名称结论）

- 旧称作检索别名保留：`forcefieldfx` 的“图勒皇冠护盾特效”、`ruins-bat` 的“图勒棍棒”、`dreadstone_spawn_fx` 的“恐惧石特效”、`armor-rift` 的“恐惧石甲”、`toadstool_cap_absorbfx` 的“蛤蟆菇帽吸收/释放”、`wathgrithr_spirit` 的“武神之魂/英灵”。这些只在 aliases，标题/摘要/索引主名已用官方名。
- `role-tools`、`character-gear-extra`、`living-tools` 的成员官方名、`hats-event` 叶索引旧俗称、五个 FX 标题旧译与“怨咒→惊扰”已在第二轮修复，不再列为缺口。

## 范围外

- 未修改 Skill、工具箱或 raw；未运行游戏；未提交、未推送。
- 第二轮为 3 页补入 `languages/chinese_s.po` 并按规则 `record`；其余页 sources 未变，未刷新其他基线。
- 复检期间装备索引由并行会话拆分为五个叶索引；本轮更新 `staffs-character-index`、`tools-field-index`、`fx-shadow-index`、`fx-world-index`、`fx-status-index`、`wearables-index`、`storage-heavy-index`、`fx-transition-index` 等受影响行，未重做其他页的索引登记。

