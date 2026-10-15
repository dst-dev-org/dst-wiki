---
title: "阿比盖尔攻击环特效（abigail_attack_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "阿比盖尔进攻时的旋转环绕表现族：abigail_attack_fx 竖立跟随、abigail_attack_fx_ground 铺地背景层；实体出厂不带动画，由宿主按攻击等级播 attack{1/2/3}_pre→_loop(显式循环)，收尾走宿主调用的 kill_fx 播 _pst。"
aliases: ["abigail_attack_fx", "abigail_attack_fx_ground", "阿比盖尔攻击特效", "阿比盖尔光环特效", "abigail attack ring"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/abigail_attack_fx.lua", "stategraphs/SGabigail.lua", "prefabs/abigail.lua", "components/avengingghost.lua", "prefabs/player_common.lua", "prefabs/skilltree_wendy.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 阿比盖尔攻击环特效（abigail_attack_fx）

## 快速判断

- 职责与适用：[阿比盖尔](../../../raw/current/scripts/languages/chinese_s.po#L337870-L337872)（官方译名）进入攻击性状态时的环绕表现：宿主 SpawnPrefab 后 AddChild，按当前攻击等级播 `attack{1/2/3}_pre` → `attack{level}_loop`（显式循环）；退出态时调 `kill_fx(level)` 播 `_pst` 并定时自删（[startaura](../../../raw/current/scripts/stategraphs/SGabigail.lua#L22-L51)、[kill_fx](../../../raw/current/scripts/prefabs/abigail_attack_fx.lua#L7-L15)）。
- 优先使用：查攻击等级来源 → abigail.lua 的 UpdateDamage（时段 1/2/3，攻击药水强制 3）；复用表现只需拿 kill_fx 协议（宿主播动画＋收尾）。
- 重要副作用：特效无 combat/伤害；murder buff 时整体换 `abigail_attack_fx_shadow_build` 并覆写漩涡符号；皮肤经 flower 符号 OverrideItemSkinSymbol（[影子覆盖](../../../raw/current/scripts/stategraphs/SGabigail.lua#L39-L45)、[皮肤](../../../raw/current/scripts/stategraphs/SGabigail.lua#L47-L50)）。
- 修改边界：`abigail_attack_fx_ground` 已注册但本快照未发现生成点；实体出厂无动画、无自觉生命周期（除 kill_fx 的定时删除）。[ground 无生成点](../../../raw/current/scripts/prefabs/abigail_attack_fx.lua#L67-L68)、[依赖声明](../../../raw/current/scripts/prefabs/abigail.lua#L26-L27)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `abigail_attack_fx` | normal_fn：Transform＋AnimState＋SoundEmitter＋Light＋Network → bank/build `abigail_attack_fx`（**不播任何初始动画**）→ SetPristine → 主模拟 `persists=false`、`kill_fx=normal_kill_fx`：播 `attack{level}_pst` 后 15 帧自删 | 竖立跟随宿主；SoundEmitter/Light 挂了组件但工厂未配置参数 | [normal_fn](../../../raw/current/scripts/prefabs/abigail_attack_fx.lua#L44-L65)、[收尾](../../../raw/current/scripts/prefabs/abigail_attack_fx.lua#L7-L10) |
| `abigail_attack_fx_ground` | ground_fn：同上，另 `SetOrientation(OnGround)`＋背景层＋SortOrder 3 → 主模拟 `kill_fx=ground_kill_fx`：播 `attack{level}_ground_pst` 后 12 帧自删 | 铺地形态；本快照仅注册与依赖声明，无 SpawnPrefab 调用 | [ground_fn](../../../raw/current/scripts/prefabs/abigail_attack_fx.lua#L17-L42)、[收尾](../../../raw/current/scripts/prefabs/abigail_attack_fx.lua#L12-L15) |

两注册名均无 NAMES 条目；标题中文为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 宿主：SGabigail 常态光环 | startaura：生成＋AddChild＋`attack{level}_pre`→`_loop(true)`；stopaura：`kill_fx(level)`；等级取 `inst.attack_level`（abigail.lua 289-291：day 1／dusk 2／night 3，`ghostlyelixir_attack_buff` 药水按 night 计） | 阿比盖尔本体的红光与 MultColour 在宿主上设置，特效只管环绕动画 | [startaura](../../../raw/current/scripts/stategraphs/SGabigail.lua#L22-L51)、[stopaura](../../../raw/current/scripts/stategraphs/SGabigail.lua#L53-L62)、[等级映射](../../../raw/current/scripts/prefabs/abigail.lua#L289-L291) |
| 宿主：SGabigail 突进攻击 | `abigail_attack` 状态时间轴第 10 帧生成 `sg.mem.abigail_attack_fx`＋AddChild＋pre/loop(true)，onexit 直接 Remove（不走 _pst） | gestalt 形态不生成；loop 态 onexit 同样 Remove | [突进生成](../../../raw/current/scripts/stategraphs/SGabigail.lua#L644-L650)、[移除](../../../raw/current/scripts/stategraphs/SGabigail.lua#L677-L678)、[loop 态移除](../../../raw/current/scripts/stategraphs/SGabigail.lua#L716-L717) |
| 等级热替换 | UpdateDamage 末尾：等级变化且 `_loop` 不匹配时直接重播 `attack{level}_loop(true)` | 仅常态光环路径的 fx 生效 | [热替换](../../../raw/current/scripts/prefabs/abigail.lua#L301-L304) |
| 宿主：复仇之灵（玩家） | [[reference/components/avengingghost|avengingghost 组件]]（玩家装备，player_common 2525；温蒂技能组 avengingghost）StartAvenging：生成 `ghost_attack_fx`＋AddChild，SetGhostDamage 按时段选 attack1/2/3（push=false 时只播 loop） | 伤害与 AnimState 染红在玩家自身；音效 attack_LP 由宿主播 | [组件挂载](../../../raw/current/scripts/prefabs/player_common.lua#L2525)、[技能组](../../../raw/current/scripts/prefabs/skilltree_wendy.lua#L149-L157)、[StartAvenging](../../../raw/current/scripts/components/avengingghost.lua#L110-L135)、[时段动画](../../../raw/current/scripts/components/avengingghost.lua#L95-L107) |
| 皮肤与影子 | murder buff（`abigail_murder_buff`）→ SetBuild `abigail_attack_fx_shadow_build`＋OverrideSymbol fx_swirl/fx_aoe_swirl/fx_swirl_01；宿主皮肤 → flower 符号回退 `abigail_attack_fx` | 仅常态光环路径应用 | [影子覆盖](../../../raw/current/scripts/stategraphs/SGabigail.lua#L39-L45)、[皮肤 flower](../../../raw/current/scripts/stategraphs/SGabigail.lua#L47-L50) |
| 资源 | anim/abigail_attack_fx.zip＋anim/abigail_attack_fx_shadow_build.zip；attack{1/2/3}[_ground]{_pre,_loop,_pst} 系列动画 | 皮肤符号 flower 以原 build 为回退 | [assets](../../../raw/current/scripts/prefabs/abigail_attack_fx.lua#L1-L5) |

## 典型接入

### 复用"等级化环绕攻击表现"

- 前提与选择：主模拟生成后 AddChild（跟随靠父子化），播 `attack{n}_pre` 再 `PushAnimation(anim.."_loop", true)`；结束调 `fx:kill_fx(n)`（注意全小写）播对应 `_pst` 并等 15/12 帧自删。
- 生效与撤销：等级切换直接重播 loop（原版 UpdateDamage 做法）；直接 Remove 不播 _pst 是突进态的官方用法，两者都可复用。
- 原版依据：SGabigail startaura 与 avengingghost 两种宿主分别示范"常态持续"与"状态期临时"两种挂接。

```lua
local fx = SpawnPrefab("abigail_attack_fx")
host:AddChild(fx)
fx.AnimState:PlayAnimation("attack1_pre")
fx.AnimState:PushAnimation("attack1_loop", true)
-- 结束：
fx:kill_fx(1)   -- 播 attack1_pst，15 帧后自动 Remove
```

- 调整与验证：ground 变体无生成点可自行取用（铺地背景层已配好）；影子换装需同步覆写三个漩涡符号，缺一个会露出原 build 漩涡。

## 常见陷阱与限制

- **边界：`abigail_attack_fx_ground` 是已注册未接线**：全脚本快照仅注册（L67-68）与 abigail.lua 依赖声明（L26-27），无任何 SpawnPrefab——复用它不会与原版逻辑冲突，但也不能指望有现成宿主。[证据](../../../raw/current/scripts/prefabs/abigail_attack_fx.lua#L67-L68)
- **误区：收尾函数叫 KillFX**：本文件暴露的是小写 `kill_fx(level)`（大写 KillFX 是 warg/ember 族与 channel 族的协议）；调错函数会直接报 attempt to call nil（[暴露](../../../raw/current/scripts/prefabs/abigail_attack_fx.lua#L39)、[L62](../../../raw/current/scripts/prefabs/abigail_attack_fx.lua#L62)）。
- **边界：出厂不带动画也不带声音**：动画全由宿主驱动，工厂从未触碰自带的 SoundEmitter/Light 参数；"attack_LP 怒吼"等声音来自宿主（[startaura](../../../raw/current/scripts/stategraphs/SGabigail.lua#L30)、[工厂未配置](../../../raw/current/scripts/prefabs/abigail_attack_fx.lua#L44-L65)）。
- **边界：kill_fx 的定时删除依赖实体存活**：normal 15 帧／ground 12 帧后 Remove，若宿主中途再播新动画会与 _pst 混帧——原版通过 stopaura 置 `attack_fx = nil` 防重复调用（[stopaura](../../../raw/current/scripts/stategraphs/SGabigail.lua#L53-L62)）。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；工厂全文（双工厂与 kill_fx 收尾）、SGabigail startaura/stopaura/突进态/热替换、abigail.lua 等级映射、avengingghost 组件宿主与技能组、阿比盖尔译名已核验；ground 变体动画实际帧数、shadow_build 内符号清单、abigaillevelupfx 的 flower 回退路径未核验。
- 2026-11-02：`abigaillevelupfx` 的工厂与花符号路径另见 [[reference/prefabs/abigailsummonfx#注册与构造]]；本页历史核验范围不变。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-combat-index|攻击、弹道与落点]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
