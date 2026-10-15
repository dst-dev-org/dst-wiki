---
title: "滚动无人机碰撞火花与灵药护盾（wagdrone_rolling_collide_small_fx 等）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "滚动无人机两种碰撞火花与幽灵灵药玩家受击护盾的通用表表现及各自生成条件。"
aliases: ["wagdrone_rolling_collide_small_fx", "wagdrone_rolling_collide_med_fx", "elixir_player_forcefield", "滚动无人机碰撞火花", "灵药玩家护盾特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "stategraphs/SGwagdrone_rolling.lua", "prefabs/wagdrone_rolling.lua", "prefabs/ghostly_elixirs.lua", "prefabs/player_common.lua", "componentutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 滚动无人机碰撞火花与灵药护盾（wagdrone_rolling_collide_small_fx 等）

## 快速判断

- 三名为通用表一次性动画表现：滚动无人机 small／med 碰撞火花和玩家灵药受击护盾。它们由不同生成者触发；攻击、反冲和减伤不是 FX 自身的玩法。公共工厂与回收见 [[reference/mechanisms/generic-fx-factory#关键流程]]。
- 中文为根据动画及调用语境核验的效果描述，非官方译名；简中 `languages/chinese_s.po` 定向检索三名显示名键无命中。

## 注册与构造

| 注册名 | bank／build | anim／sound／fn | 条目 |
|---|---|---|---|
| `wagdrone_rolling_collide_small_fx` | `hits_sparks`／`lavaarena_hit_sparks_fx` | `hit_1`；`rifts5/wagdrone_rolling/collide`；fn 隐藏 `glow` | [条目](../../../raw/current/scripts/fx.lua#L3895-L3902) |
| `wagdrone_rolling_collide_med_fx` | 同上 | `hit_2`；同声；fn 隐藏 `glow` | [条目](../../../raw/current/scripts/fx.lua#L3905-L3912) |
| `elixir_player_forcefield` | `abigail_shield`／同左 | `player_shield`；无 sound、fn 字段 | [条目](../../../raw/current/scripts/fx.lua#L3889-L3892) |

## 关联机制导航

| 生成者 | 条件、落点和证据 |
|---|---|
| 滚动无人机强制反冲 | 状态图处于 running 收到 `forced_spinning_recoil` 时，helper 计算与目标距离、必要时移动无人机，再于计算点 y=1 生成 small；未父挂。[事件](../../../raw/current/scripts/stategraphs/SGwagdrone_rolling.lua#L142-L145)、[helper](../../../raw/current/scripts/stategraphs/SGwagdrone_rolling.lua#L11-L30) |
| 滚动无人机旋转范围判定 | `DoSpinningAOE` 检查目标距离与可工作对象，标记 spark 时生成 med：重合时随机落点，否则按碰撞方向距无人机半径落点，y=0；未父挂。[筛选](../../../raw/current/scripts/stategraphs/SGwagdrone_rolling.lua#L161-L195)、[spark 生成](../../../raw/current/scripts/stategraphs/SGwagdrone_rolling.lua#L265-L284) |
| 同类碰撞与硬甲 | 范围判定中另一滚动无人机处于 running 时，small 在两者中点 y=1；持硬甲目标分支则在方向点或重合随机点 y=0 生成 small，再按条件攻击和反冲。[同类](../../../raw/current/scripts/stategraphs/SGwagdrone_rolling.lua#L289-L295)、[硬甲](../../../raw/current/scripts/stategraphs/SGwagdrone_rolling.lua#L313-L345) |
| 玩家灵药受击 | shield 或 retaliation 的玩家分支监听 `attacked`；回调检查事件非重定向、攻击者存在，且攻击者的物品持有人或攻击者自身不等于受击者本人（其他玩家不因此被排除）。头槽物品须有已充能 `rechargeable`，然后生成护盾代理并 `AddChild` 挂玩家，玩家自身播放另一护盾声、移除减伤 modifier。之后若 `elixir_buff` 存在才继续反击判断并使帽子进入 10 秒充能。具体灵药链见 [[reference/prefabs/ghostlyelixir_slowregen_fx#生成链与生命周期]]。[判定](../../../raw/current/scripts/componentutil.lua#L114-L122)、[受击及槽位](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L6-L25)、[放电](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L81-L81)、[两药监听](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L231-L245)、[反击药](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L262-L281) |

无人机 Prefab 主模拟侧装载该状态图；依赖数组声明两火花。玩家公共预载数组声明护盾，但声明本身不构成生成。[端分界](../../../raw/current/scripts/prefabs/wagdrone_rolling.lua#L179-L182)、[状态图](../../../raw/current/scripts/prefabs/wagdrone_rolling.lua#L222-L222)、[无人机依赖](../../../raw/current/scripts/prefabs/wagdrone_rolling.lua#L10-L11)、[玩家依赖](../../../raw/current/scripts/prefabs/player_common.lua#L2223-L2223)

## 常见陷阱与限制

- 碰撞的声音属于火花条目；灵药护盾条目无声，其生成回调单独让玩家播放护盾声。直接生成护盾不会施加减伤或触发反击。
- 强制反冲 helper 的落点公式使用 `dx / dist` 和 `dz / dist`，重合时虽另选反冲角，源码未对零距离除法单独处理；该边界未在游戏运行核验。[公式](../../../raw/current/scripts/stategraphs/SGwagdrone_rolling.lua#L11-L26)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394；逐项核验三项 bank／build／anim／sound／fn；对 `raw/current/scripts` 全树检索三全名、`wagdrone_rolling_collide`／`elixir_player_forcefield` 前缀及拼接线索，跟读各生成点、依赖和状态图安装。未核验动画外观、时长、多人及零距离运行表现。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-table-index|通用表 FX 索引]]。
