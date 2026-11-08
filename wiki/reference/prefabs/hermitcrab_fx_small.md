---
title: "寄居蟹隐士迁移、传送门与涟漪特效（hermitcrab_fx_small 等）"
created: "2026-11-01"
updated: "2026-11-08"
type: prefab
summary: "寄居蟹隐士迁移三档、导弹爆炸、岩树碎屑、宝库与中庭传送门和首领涟漪的九种通用表表现。"
aliases: ["hermitcrab_fx_small", "hermitcrab_fx_med", "hermitcrab_fx_tall", "missile_explosion_fx", "tree_rock_chop", "tree_rock_fall", "vault_portal_fx", "atrium_portal_fx", "boss_ripple_fx", "寄居蟹隐士迁移特效", "宝库传送特效", "中庭传送特效", "首领涟漪"]
tags: [prefab, fx]
dst_build: "steam-buildid:26474873"
sources: ["fx.lua", "components/hermitcrab_relocation_manager.lua", "prefabs/hermitcrab_shell.lua", "prefabs/hermitcrab_teashop.lua", "prefabs/lantern_posts_defs.lua", "prefabs/meatrack2.lua", "stategraphs/SGhermitcrab.lua", "prefabs/forest.lua", "prefabs/wagboss_missile.lua", "prefabs/tree_rocks.lua", "prefabs/vault_markers.lua", "prefabs/atrium_markers.lua", "prefabs/vault_orb_refined.lua", "prefabs/vault_teleporter.lua", "prefabs/charliearena_teleporter.lua", "prefabs/staff_common.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilsonghost.lua", "stategraphs/SGmalbatross.lua", "stategraphs/SGeyeofterror.lua", "prefabs/malbatross.lua", "prefabs/eyeofterror.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 寄居蟹隐士迁移、传送门与涟漪特效（hermitcrab_fx_small 等）

## 快速判断

- 九名分属迁移、命中、岩树、传送和首领水面动作；游戏实体搬迁、瓦片与伤害由调用方执行。工厂代理与本地动画生命周期见 [[reference/mechanisms/generic-fx-factory#关键流程]]。
- `atrium_portal_fx` 的本地 fn 在父实体是本地玩家时推 `shroudensummoned` 并解挂，不能一概说九项均无脚本副作用。[条目](../../../raw/current/scripts/fx.lua#L4020-L4032)
- 特效中文是按调用与动画核验的效果描述，非官方译名；简中 po 的九个注册名显示键未命中。角色官方简中名为寄居蟹隐士。[名称](../../../raw/current/scripts/languages/chinese_s.po#L343780-L343782)

## 注册与构造

| 注册名 | bank／build；anim | 其他字段与依据 |
|---|---|---|
| `hermitcrab_fx_small` | `hermitcrab_fx`／同左；`hermitcrab_fx_small` | sound `rifts5/hermit_island/whirlpool_up_s`；fn `FinalOffset3`。[条目](../../../raw/current/scripts/fx.lua#L3915-L3920) |
| `hermitcrab_fx_med` | 同上；`hermitcrab_fx_med` | sound `rifts5/hermit_island/whirlpool_up_m`；fn `FinalOffset3`。[条目](../../../raw/current/scripts/fx.lua#L3923-L3928) |
| `hermitcrab_fx_tall` | 同上；`hermitcrab_fx_tall` | sound `rifts5/hermit_island/whirlpool_up_l`；fn `FinalOffset3`。[条目](../../../raw/current/scripts/fx.lua#L3931-L3936) |
| `missile_explosion_fx` | `missile_fx`／同左；`impact` | sound `rifts5/wagstaff_boss/missile_explode`；无 fn。[条目](../../../raw/current/scripts/fx.lua#L3939-L3943) |
| `tree_rock_chop` | `tree_rock_fx`／同左；`chop` | sound 行注释，无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L3946-L3950) |
| `tree_rock_fall` | `tree_rock_fx`／同左；`fall` | sound 行注释，无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L3953-L3957) |
| `vault_portal_fx` | 同名 bank／build；`activate` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L3960-L3963) |
| `atrium_portal_fx` | 同名 bank／build；`activate` | 无 sound；fn 检查本地玩家父实体、推事件并解挂。[条目](../../../raw/current/scripts/fx.lua#L4020-L4032) |
| `boss_ripple_fx` | `malbatross_ripple`／同左；`idle` | 无 sound；fn 设地下层、船尾迹排序和海水混合参数。[条目](../../../raw/current/scripts/fx.lua#L2654-L2663) |

## 关联机制导航

| 生成者 | 条件、落点及依据 |
|---|---|
| 隐士迁移管理器 | 记录的房屋选 tall、蜂箱和隐士选 small、晾肉架选 med；离开阶段按 `fxprefab` 落旧实体完整世界坐标，到达阶段落目标 x/z、y=0。无 FX、休眠或处于 limbo 时走跳过相应表现的分支，搬迁仍由管理器执行。[映射](../../../raw/current/scripts/components/hermitcrab_relocation_manager.lua#L315-L326)、[到达](../../../raw/current/scripts/components/hermitcrab_relocation_manager.lua#L227-L235)、[离开](../../../raw/current/scripts/components/hermitcrab_relocation_manager.lua#L251-L262) |
| 隐士号角／岛设施／隐士动作 | 号角目的地有效且允许传送时在出发、到达点用 med；茶店、灯柱、晾肉架退出有效区域并未废弃时延迟落 med 后切为破损；隐士 `dancebusy` 的茶店传送分支生 small。[号角](../../../raw/current/scripts/prefabs/hermitcrab_shell.lua#L97-L123)、[到达](../../../raw/current/scripts/prefabs/hermitcrab_shell.lua#L44-L58)、[茶店](../../../raw/current/scripts/prefabs/hermitcrab_teashop.lua#L258-L267)、[灯柱](../../../raw/current/scripts/prefabs/lantern_posts_defs.lua#L62-L71)、[晾肉架](../../../raw/current/scripts/prefabs/meatrack2.lua#L491-L500)、[隐士](../../../raw/current/scripts/stategraphs/SGhermitcrab.lua#L992-L1002) |
| 导弹／岩树 | WAGboss 导弹 `Detonate` 在 x/z、`max(0,y)` 生成爆炸后处理范围实体；岩树砍伐在树位 y 加随机偏移生成 chop，声音另在树上播放。[导弹](../../../raw/current/scripts/prefabs/wagboss_missile.lua#L522-L534)、[岩树](../../../raw/current/scripts/prefabs/tree_rocks.lua#L593-L607) |
| 宝库传送 | 房间 marker 到达回调用 `vault_portal_fx`；精炼宝珠到达目标或自身坐标时生 FX，非玩家同伴由宝库传送器生成，玩家／幽灵由状态生成；玩家状态接受 `data.fxprefab` 替代默认名。[marker](../../../raw/current/scripts/prefabs/vault_markers.lua#L230-L232)、[宝珠](../../../raw/current/scripts/prefabs/vault_orb_refined.lua#L10-L22)、[同伴](../../../raw/current/scripts/prefabs/vault_teleporter.lua#L189-L200)、[玩家](../../../raw/current/scripts/stategraphs/SGwilson.lua#L20195-L20203)、[幽灵](../../../raw/current/scripts/stategraphs/SGwilsonghost.lua#L596-L602) |
| 中庭／查理传送 | 中庭 marker 到达回调落 `atrium_portal_fx`，查理竞技场传送器通过字段选该名；手杖 `notlightningteleport` 分支在前后各生一枚。玩家／幽灵查理传送状态把代理父挂，供本地 fn 判断并触发屏幕事件。[marker](../../../raw/current/scripts/prefabs/atrium_markers.lua#L186-L188)、[传送器](../../../raw/current/scripts/prefabs/charliearena_teleporter.lua#L322-L322)、[手杖结束](../../../raw/current/scripts/prefabs/staff_common.lua#L293-L299)、[手杖开始](../../../raw/current/scripts/prefabs/staff_common.lua#L396-L399)、[玩家](../../../raw/current/scripts/stategraphs/SGwilson.lua#L28790-L28793)、[幽灵](../../../raw/current/scripts/stategraphs/SGwilsonghost.lua#L655-L658) |
| 首领水面涟漪 | 鹅首领 `spawnripple` 与恐怖之眼 `spawn_ground_fx` 在非视觉地面落该名；后者视觉地面改用 `slide_puff`。此处判定不等于检查是否有船。[鹅](../../../raw/current/scripts/stategraphs/SGmalbatross.lua#L15-L20)、[眼](../../../raw/current/scripts/stategraphs/SGeyeofterror.lua#L121-L129) |

## 常见陷阱与限制

- `tree_rock_fall` 在本快照的全 scripts 检索中只见 fx.lua 注册和 `tree_rocks.lua` 的依赖声明，未找到生成点；不据此断言其他版本或模组不会使用。
- 函数按代理父实体创建本地动画；`atrium_portal_fx` 内只有本地玩家父实体命中才推事件和解挂，落点方式不保证触发该事件。

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。逐项核对九条 bank/build/anim/sound/fn，针对全 `raw/current/scripts` 检索完整名称、`hermitcrab_fx_` 与 `portal_fx` 等前缀，追踪 `fxprefab` 字段及动态选名；依赖声明不当生成。未解包动画或实测跨端显示。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-water-index|水面、航海与船件]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-03）。
