---
title: "特殊树叶与死亡烟（sugarwood_leaf_fx 等）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "树怪紫叶、糖木叶、松针、月树花瓣、海树与棕榈松叶及催长表现，另列共用通用表的死亡烟。"
aliases: ["die_fx", "pine_needles_chop", "purple_leaves", "purple_leaves_chop", "sugarwood_leaf_fx", "sugarwood_leaf_fx_chop", "sugarwood_leaf_withered_fx", "sugarwood_leaf_withered_fx_chop", "tree_petal_fx_chop", "treegrowthsolution_use_fx", "oceantree_leaf_fx_fall", "oceantree_leaf_fx_chop", "palmcone_leaf_fx_tall", "palmcone_leaf_fx_normal", "palmcone_leaf_fx_short", "特殊树叶", "紫叶", "糖木叶", "死亡烟", "树木催长特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/deciduoustrees.lua", "prefabs/evergreens.lua", "prefabs/winter_tree.lua", "prefabs/quagmire_sugarwoodtree.lua", "prefabs/quagmire_evergreen.lua", "prefabs/moontree.lua", "prefabs/oceantree.lua", "prefabs/oceantree_pillar.lua", "prefabs/watertree_pillar.lua", "prefabs/palmconetree.lua", "prefabs/treegrowthsolution.lua", "components/treegrowthsolution.lua", "prefabs/player_common_extensions.lua", "prefabs/corpses_defs.lua", "stategraphs/SGplayer_hosted.lua", "prefabs/balatro_machine.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 特殊树叶与死亡烟（sugarwood_leaf_fx 等）

## 快速判断

- 本页列 15 个通用表表现注册名：树叶和催长相关 14 名，另有同批盘点的 `die_fx` 死亡烟；玩家死亡／复活和奖励生成由调用方实现。公共代理与本地表现见 [[reference/mechanisms/generic-fx-factory#关键流程]]；四色普通落叶另见 [[reference/prefabs/green_leaves#注册与构造]]。
- 中文为按条目及调用语境核验的效果描述，非官方译名；`languages/chinese_s.po` 定向检索这 15 个注册名未命中显示名键。动画素材外观及实际时长未在游戏验证。

## 注册与构造

均由通用工厂注册。表中 bank 与 build 相同时合写；除明确注明者外无 `sound`、`fn` 字段。叶响完整路径为 `dontstarve_DLC001/fall/leaf_rustle`。

| 注册名 | bank / build；anim | 其他字段与证据 |
|---|---|---|
| `die_fx` | `die_fx` / `die`；`small` | 声 `dontstarve/common/deathpoof`；tint RGB 为 90/255、66/255、41/255；无 fn。[条目](../../../raw/current/scripts/fx.lua#L56-L61) |
| `pine_needles_chop` | `pine_needles`；`chop` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L358-L361) |
| `purple_leaves` | `tree_monster_fx`；`fall` | 叶响、无 fn。[条目](../../../raw/current/scripts/fx.lua#L427-L431) |
| `purple_leaves_chop` | `tree_monster_fx`；`chop` | 叶响、无 fn。[条目](../../../raw/current/scripts/fx.lua#L392-L396) |
| `sugarwood_leaf_fx` | `tree_leaf_fx_quagmire`；`fall` | 叶响、无 fn。[条目](../../../raw/current/scripts/fx.lua#L434-L438) |
| `sugarwood_leaf_fx_chop` | `tree_leaf_fx_quagmire`；`chop` | 叶响、无 fn。[条目](../../../raw/current/scripts/fx.lua#L441-L445) |
| `sugarwood_leaf_withered_fx` | `tree_leaf_fx_quagmire_withered`；`fall` | 叶响、无 fn。[条目](../../../raw/current/scripts/fx.lua#L448-L452) |
| `sugarwood_leaf_withered_fx_chop` | `tree_leaf_fx_quagmire_withered`；`chop` | 叶响、无 fn。[条目](../../../raw/current/scripts/fx.lua#L455-L459) |
| `tree_petal_fx_chop` | `tree_petal_fx`；`chop` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L462-L465) |
| `treegrowthsolution_use_fx` | `treegrowthsolution`；`use` | 声 `waterlogged1/common/use_figjam`，无 fn。[条目](../../../raw/current/scripts/fx.lua#L2629-L2633) |
| `oceantree_leaf_fx_fall` | `oceantree_leaf_fx`；`fall` | 无声；fn 随机缩放 1～1.3、速度 2.75～6.25，并每帧下移。[条目](../../../raw/current/scripts/fx.lua#L2636-L2645)、[更新](../../../raw/current/scripts/fx.lua#L34-L36) |
| `oceantree_leaf_fx_chop` | `oceantree_leaf_fx`；`chop` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L2648-L2651) |
| `palmcone_leaf_fx_tall` | 同名 bank/build；`chop` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L2907-L2910) |
| `palmcone_leaf_fx_normal` | 同名 bank/build；`chop` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L2913-L2916) |
| `palmcone_leaf_fx_short` | 同名 bank/build；`chop` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L2919-L2922) |

## 关联机制导航

| 生成者／触发 | 选名、位置及边界 | 证据 |
|---|---|---|
| 普通落叶树毒化 build | `poison` 表项指向 `purple_leaves`／`purple_leaves_chop`；`SpawnLeafFX` 按普通或 chop 分支读取表字段。燃烧、stump、burnt 或休眠时提前返回；延时执行时重新检查这些门禁。落点按生长阶段调高度。 | [选名](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L100-L109)、[生成](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L168-L197)、[周期](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L232-L236)、[砍伐](../../../raw/current/scripts/prefabs/deciduoustrees.lua#L428-L437) |
| 常青树与越冬树 | `evergreens` 的砍伐回调要求有 growable、树未睡眠、非 old 阶段且 build 非 twiggy，才落 `pine_needles_chop`；越冬常青树砍伐时另有同名调用，y 随机抬升。 | [常青树](../../../raw/current/scripts/prefabs/evergreens.lua#L392-L409)、[越冬树](../../../raw/current/scripts/prefabs/winter_tree.lua#L962-L965) |
| 糖木／暴食事件树 | 四个 sugarwood 名在 `quagmire_sugarwoodtree` 的依赖表声明，但本快照所见树工厂把主模拟后置逻辑交给 `event_server_data("quagmire", ...)`；未能核验该事件包的生成点与选名。`quagmire_evergreen` 同样声明松针，声明不证明生成。 | [四声明](../../../raw/current/scripts/prefabs/quagmire_sugarwoodtree.lua#L11-L20)、[外部后置](../../../raw/current/scripts/prefabs/quagmire_sugarwoodtree.lua#L85-L89)、[松针声明](../../../raw/current/scripts/prefabs/quagmire_evergreen.lua#L10-L13) |
| 月树砍伐 | `tree_petal_fx_chop` 生成于树位，随 short／normal／tall 调整 y；short 与 tall 分别对代理调用 0.75／1.25 缩放。代理缩放是否改变本地表现须按工厂协议核验，不能据此保证观感。 | [生成与尺寸](../../../raw/current/scripts/prefabs/moontree.lua#L154-L174)、[[reference/mechanisms/generic-fx-factory#关键流程]] |
| 海树与海树柱砍伐 | 海树生成 `oceantree_leaf_fx_chop`，y 加随机 0～2；海树柱每次砍伐以 0.58 概率在柱周围 x/z、世界 y=10 生成 `oceantree_leaf_fx_fall`。`watertree_pillar` 中该名为依赖声明。 | [海树](../../../raw/current/scripts/prefabs/oceantree.lua#L300-L304)、[柱](../../../raw/current/scripts/prefabs/oceantree_pillar.lua#L84-L110)、[另一声明](../../../raw/current/scripts/prefabs/watertree_pillar.lua#L25-L26) |
| 棕榈松及其越冬树 | 棕榈松砍伐用 `"palmcone_leaf_fx_"..inst.size`，size 为 short／normal／tall；越冬树用 `inst.statedata.name` 拼名并先检查这三值。落点均为树坐标加 y 偏移。 | [尺寸定义](../../../raw/current/scripts/prefabs/palmconetree.lua#L40-L47)、[生成](../../../raw/current/scripts/prefabs/palmconetree.lua#L155-L162)、[越冬树](../../../raw/current/scripts/prefabs/winter_tree.lua#L969-L973) |
| 树木催长剂 | 物品给 `treegrowthsolution` 组件的 `fx_prefab` 赋 `treegrowthsolution_use_fx`；`GrowTarget` 排除禁止强制生长、树桩、燃烧和烧毁对象后在目标位置生成，再由目标覆写函数或 growable 处理生长，并消耗物品。 | [字段](../../../raw/current/scripts/prefabs/treegrowthsolution.lua#L61-L62)、[组件](../../../raw/current/scripts/components/treegrowthsolution.lua#L6-L28)；[[reference/components/treegrowthsolution#关键接口]] |
| 死亡烟 `die_fx` | 玩家移除时须 `spawnskeleton` 且 `skeleton_prefab` 非空；复活取 source 或玩家坐标；化鬼须非加载且死亡产物不是 CORPSE。尸体侵蚀及托管角色死亡收尾也有生成。Balatro 机器奖励时在奖励点生成，调用方另对代理设缩放。 | [移除](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L148-L152)、[复活](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L327-L334)、[化鬼](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L673-L698)、[尸体](../../../raw/current/scripts/prefabs/corpses_defs.lua#L147-L153)、[托管角色](../../../raw/current/scripts/stategraphs/SGplayer_hosted.lua#L125-L132)、[落烟](../../../raw/current/scripts/stategraphs/SGplayer_hosted.lua#L172-L174)、[机器](../../../raw/current/scripts/prefabs/balatro_machine.lua#L91-L102)、[唱片](../../../raw/current/scripts/prefabs/balatro_machine.lua#L113-L123)、[其他奖励](../../../raw/current/scripts/prefabs/balatro_machine.lua#L150-L167) |

## 常见陷阱与限制

- `pine_needles` 的 fall 条目在块注释中；本页记录的是活跃注册 `pine_needles_chop`。[注释与条目](../../../raw/current/scripts/fx.lua#L352-L361)
- 糖木四名在所选 scripts 的完整名及 `sugarwood_leaf` 前缀检索中只见注册与依赖声明；缺少事件包源码不等于断言其无生成点。不得把糖木状态与落叶时机猜成已核验。
- 代理的变换与本地效果的转换、回收细节见 [[reference/mechanisms/generic-fx-factory#常见陷阱与限制]]；海树叶的 `fall_speed` 是实体每帧位置变化，不能把粒子寿命或代理一秒回收当作本地实体清理时间。

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。逐项检查 15 条目 bank/build/anim/sound/fn 与随机位移；在 `raw/current/scripts` 全域递归检索全部完整名及 `sugarwood_leaf`、`purple_leaves`、`palmcone_leaf_fx_`、`oceantree_leaf_fx`、`treegrowthsolution` 等前缀，追查表字段与拼名，依赖声明与实际生成分开。事件包不可见；未验证动画素材、运行时外观及时长。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-world-index|天气、地形与生态痕迹]]。
