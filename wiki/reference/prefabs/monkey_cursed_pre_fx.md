---
title: "猴诅咒与变形特效（monkey_cursed_pre_fx 等）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "猴诅咒施加与消退、玩家变猴和恢复的六个通用表动画及触发边界。"
aliases: ["monkey_morphin_power_players_fx", "monkey_de_morphin_fx", "monkey_cursed_pre_fx", "monkey_cursed_pst_fx", "monkey_deform_pre_fx", "monkey_deform_pst_fx", "猴诅咒特效", "变猴特效", "解除猴诅咒"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "curse_monkey_util.lua", "components/cursable.lua", "stategraphs/SGwilson.lua", "prefabs/player_common.lua", "prefabs/cursed_monkey_token.lua", "prefabs/wonkey.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 猴诅咒与变形特效（monkey_cursed_pre_fx 等）

## 快速判断

- 六名均为通用表动画：诅咒计数施加／移除各一名，玩家切换猴形态的前后阶段各两名。计数、皮肤和角色交换由调用方完成；代理、本地动画及回收见 [[reference/mechanisms/generic-fx-factory#关键流程]]。
- 简中 `languages/chinese_s.po` 定向检索六个注册名的 `STRINGS.NAMES` 键未命中；标题中文是按源码语境核验的效果描述，非官方译名。

## 注册与构造

下列均由通用表工厂注册；除注明外无 `sound`、`fn` 字段。

| 注册名 | bank／build；anim | 差异与证据 |
|---|---|---|
| `monkey_morphin_power_players_fx` | `cursed_fx`／同左；`idle` | `nofaced`，声 `monkeyisland/wonkycurse/curse_fx`，fn `FinalOffset1`。[条目](../../../raw/current/scripts/fx.lua#L2730-L2737)、[函数](../../../raw/current/scripts/fx.lua#L1-L3) |
| `monkey_de_morphin_fx` | `monkey_change_fx`／同左；`deform_hit` | `nofaced`，fn `FinalOffset1`，无 sound。[条目](../../../raw/current/scripts/fx.lua#L2739-L2745) |
| `monkey_cursed_pre_fx` | `monkey_change_fx`／同左；`cursed_pre` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L2849-L2852) |
| `monkey_cursed_pst_fx` | 同上；`cursed_pst` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L2855-L2858) |
| `monkey_deform_pre_fx` | 同上；`deform_pre` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L2861-L2864) |
| `monkey_deform_pst_fx` | 同上；`deform_pst` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L2867-L2870) |

## 关联机制导航

| 生成者／关系 | 触发、落点及证据 |
|---|---|
| 诅咒计数 | `cursable:ApplyCurse` 增加 MONKEY 计数后调用 `docurse`，后者先生成 `monkey_morphin_power_players_fx` 并将代理父挂持有者，再按计数设置部位／皮肤或安排变猴。`RemoveCurse` 扣减计数后调用 `uncurse`，先父挂 `monkey_de_morphin_fx`，再更新外观或安排恢复。[施加](../../../raw/current/scripts/components/cursable.lua#L8-L23)、[移除](../../../raw/current/scripts/components/cursable.lua#L26-L54)、[施加表现](../../../raw/current/scripts/curse_monkey_util.lua#L81-L91)、[解除表现](../../../raw/current/scripts/curse_monkey_util.lua#L33-L45) |
| 变形调度 | `trytransform` 在 nomorph、silentmorph、busy、pinned、dead 状态延后；变猴时伍迪特殊形态也延后。周期任务调用 `monkeychanger_pre`，骑乘时先下骑再选择变猴或恢复状态。[门禁](../../../raw/current/scripts/curse_monkey_util.lua#L3-L23)、[骑乘](../../../raw/current/scripts/stategraphs/SGwilson.lua#L23125-L23158) |
| 变猴状态 | `changetomonkey` 进入时在玩家当前位置生成 cursed_pre；动画结束调用 `ChangeToMonkey`。`changetomonkey_pst` 进入时生成 cursed_pst；两者均落点式。[前段](../../../raw/current/scripts/stategraphs/SGwilson.lua#L23177-L23206)、[后段](../../../raw/current/scripts/stategraphs/SGwilson.lua#L23221-L23230)、[交换入口](../../../raw/current/scripts/prefabs/player_common.lua#L2348-L2351) |
| 恢复状态 | `changefrommonkey` 进入时在玩家当前位置生成 deform_pre；动画结束调用 `ChangeFromMonkey`。`changefrommonkey_pst` 进入时生成 deform_pst；两者均落点式。[前段](../../../raw/current/scripts/stategraphs/SGwilson.lua#L23255-L23284)、[后段](../../../raw/current/scripts/stategraphs/SGwilson.lua#L23299-L23308) |

## 常见陷阱与限制

- 诅咒计数变化时父挂的两种动画不等同角色交换完成；完整变形可因状态门禁推迟，四个状态动画由状态图另外生成。
- `prefabs/cursed_monkey_token.lua` 与 `prefabs/wonkey.lua` 的依赖声明仅保证加载关系，不是生成点。[代币](../../../raw/current/scripts/prefabs/cursed_monkey_token.lua#L12-L13)、[猴角色](../../../raw/current/scripts/prefabs/wonkey.lua#L12-L15)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394；逐项核对六条 bank/build/anim/sound/fn，在 `raw/current/scripts` 全树检索完整注册名、`monkey_cursed`／`monkey_deform`／`curse_monkey_util` 与选名调用，区分生成、状态与依赖声明。未解包动画资源或核验其他模组调用。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-table-character-index|通用表 FX·角色与装备补充]]。
