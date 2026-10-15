---
title: "灵魂治疗特效（wortox_soul_heal_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "灵魂释放范围治疗时挂在每个被治疗者身上的跟随特效：heal 动画两倍速播放，主模拟按动画帧用 colouradder 给目标推红色调并播治疗音；治疗量本身由 DoHeal 结算。"
aliases: ["wortox_soul_heal_fx", "灵魂治疗特效", "灵魂回血特效", "soul heal fx"]
tags: [prefab, fx, network-sync]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/wortox_soul_heal_fx.lua", "components/colouradder.lua", "prefabs/wortox_soul_common.lua", "prefabs/wortox_soul.lua", "prefabs/wortox_soul_spawn.lua", "prefabs/wortox.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 灵魂治疗特效（wortox_soul_heal_fx）

## 快速判断

- 职责与适用：灵魂（wortox_soul，官方译名"灵魂"，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L351880-L351882)）释放触发范围治疗时，对**每个实际被治疗的玩家**生成的跟随特效：bank／build `wortox_soul_heal_fx` 播 `heal`，1.5 倍缩放、DeltaTimeMultiplier 2、FinalOffset 3；主模拟按 FX 自身动画帧用 colouradder 给目标推红色调，并在目标身上播治疗音。回血由 `wortox_soul_common.DoHeal` 的 `health:DoDelta` 完成，FX 无治疗数值。[工厂](../../../raw/current/scripts/prefabs/wortox_soul_heal_fx.lua#L44-L56)
- 优先使用：查"灵魂治疗／回血特效"选本页；治疗链入口见下文"生成链"小节。
- 重要副作用：目标已有 colouradder 时才推色（Setup 不补挂该组件），音效在目标 SoundEmitter 存在时播放；50% 概率水平翻转（主模拟随机）。[翻转](../../../raw/current/scripts/prefabs/wortox_soul_heal_fx.lua#L66-L68)
- 修改边界：`Setup` 在主模拟门后暴露，色调与音效逻辑都在主模拟执行；客户端只是显示动画实体。[端分界](../../../raw/current/scripts/prefabs/wortox_soul_heal_fx.lua#L60-L72)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wortox_soul_heal_fx` | fn：Transform＋AnimState＋Network → 播 `heal` → SetPristine → 客户端早退 → 主模拟随机翻转、挂 `animover` 自删、`persists=false`、暴露 `Setup` | 单一注册；`PlayAnimation("heal")` 未显式传循环参数，实际是否重复未核验。[注册](../../../raw/current/scripts/prefabs/wortox_soul_heal_fx.lua#L77)、[主模拟](../../../raw/current/scripts/prefabs/wortox_soul_heal_fx.lua#L66-L72) |

标题中文为已核验效果描述，非官方译名（NAMES／SKIN_NAMES 未发现该 FX 注册名条目）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成链：wortox_soul_common.DoHeal | 范围内受伤玩家逐个 `health:DoDelta` 后生成 FX：`AddFollower().FollowSymbol(目标, hiteffectsymbol, 0, -50, 0)` 再 `fx:Setup(v)` | 只对 `IsHurt`、非 health_as_oldage、存活可见且有 combat 的范围内玩家生成；满血不生成（注释说明"总是显示"是相对特殊目标选择而言） | [DoHeal](../../../raw/current/scripts/prefabs/wortox_soul_common.lua#L30-L48)、[生成与挂符号](../../../raw/current/scripts/prefabs/wortox_soul_common.lua#L42-L45) |
| 治疗数值 | 与 FX 无读写关系，同源调用：`TUNING.HEALING_MED`（20）减每多目标 2，最低 5，范围 TUNING.WORTOX_SOULHEAL_RANGE（8）＋技能修正 | naughty 倾向乘 TUNING.SKILLS.WORTOX.NAUGHTY_SOULHEAL_RECEIVED_MULT | [治疗量](../../../raw/current/scripts/prefabs/wortox_soul_common.lua#L32-L41)、[tuning](../../../raw/current/scripts/tuning.lua#L4547-L4549)、[[reference/components/health#关键接口]] |
| DoHeal 调用者 | ① wortox_soul 落地到期 `KillSoul_FromPocket`（soulprotector_3 爆裂分支加倍）；② wortox_soul_spawn 飞行超时 `OnTimeout` | 两个宿主都把 wortox_soul_heal_fx 列入 prefabs 依赖 | [到期](../../../raw/current/scripts/prefabs/wortox_soul.lua#L35-L52)、[爆裂](../../../raw/current/scripts/prefabs/wortox_soul.lua#L26-L34)、[超时](../../../raw/current/scripts/prefabs/wortox_soul_spawn.lua#L248-L259) |
| FX 内部：帧驱动色调 | `Setup` 给 FX 加 updatelooper，按 FX 动画帧给目标 colouradder PushColour：帧 <10 线性到 0.5；10-40 二次回落；≥40 PopColour 并移除更新 | colouradder 与 updatelooper 都挂在主模拟侧 | [色调区间](../../../raw/current/scripts/prefabs/wortox_soul_heal_fx.lua#L8-L31)、[[reference/components/colouradder#关键接口]]、[[reference/components/updatelooper#关键接口]] |
| 音效 | `Setup` 在**目标** SoundEmitter 播 `dontstarve/characters/wortox/soul/heal` | 目标无 SoundEmitter 时跳过 | [Setup](../../../raw/current/scripts/prefabs/wortox_soul_heal_fx.lua#L39-L41) |
| 同 bank 辨析 | wortox.lua 的 `wortox_panflute_buff_fx_fn` 复用同 bank／build 播 `heal`（loop=true、缩放 1×0.75、DeltaTimeMultiplier 0.5），是另一个注册名，不是本 FX | 勿以 bank 名反查注册名 | [同名 bank](../../../raw/current/scripts/prefabs/wortox.lua#L1254-L1280) |
| 动画资源 | bank／build `wortox_soul_heal_fx`，anim `heal` | 资源包内容未解包 | [声明与设置](../../../raw/current/scripts/prefabs/wortox_soul_heal_fx.lua#L1-L3)、[L51-L56](../../../raw/current/scripts/prefabs/wortox_soul_heal_fx.lua#L51-L56) |

## 典型接入

### 复用为治疗提示特效

- 前提与选择：主模拟生成并挂到有效目标符号后调 `Setup(target)`；需要变色时目标须已有 colouradder（工厂不补挂），音效要求目标有 SoundEmitter。
- 生效与撤销：FX 的 animover 自删与色调完成（≥40 帧）是两个独立终点；目标中途无效时只移除更新函数。[失效分支](../../../raw/current/scripts/prefabs/wortox_soul_heal_fx.lua#L29-L30)

```lua
local fx = SpawnPrefab("wortox_soul_heal_fx")
local symbol = target.components.combat ~= nil
    and target.components.combat.hiteffectsymbol or "hit"
fx.entity:AddFollower():FollowSymbol(target.GUID, symbol, 0, -50, 0)
fx:Setup(target)
```

- 调整与验证：色调强度写死在 TINT（暗红 154/23/19）与 0.5 系数里，不能参数化；改变播放速度会影响色调节奏（帧驱动）。[常量](../../../raw/current/scripts/prefabs/wortox_soul_heal_fx.lua#L6)

## 常见陷阱与限制

- **误区：FX 自己回血**：治疗在 `DoHeal` 的 `health:DoDelta`；FX 只做色调＋音效，且只对实际进入治疗列表的玩家生成。[治疗](../../../raw/current/scripts/prefabs/wortox_soul_common.lua#L41)
- **误区：跟随符号由 FX 自带**：Follower 是生成方在 SpawnPrefab 后手动 Add 的，工厂只建 Transform／AnimState。[生成方](../../../raw/current/scripts/prefabs/wortox_soul_common.lua#L43-L44)
- **边界：颜色源移除由 colouradder 兜底**：本厂没有显式 OnRemoveEntity 回落，但 colouradder:PushColour 会监听实体颜色源的 onremove 并 PopColour；不能据本厂缺回调断言提前删除会残色。[颜色源监听](../../../raw/current/scripts/components/colouradder.lua#L84-L120)[无回落](../../../raw/current/scripts/prefabs/wortox_soul_heal_fx.lua#L70-L72)
- **边界：色调在主模拟执行**：客户端显示路径（colouradder 的联网同步）未核验；专服同样执行 Setup（ismastersim 分支内）但不渲染。[端分界](../../../raw/current/scripts/prefabs/wortox_soul_heal_fx.lua#L60-L64)
- **边界：另有一个同名 bank 消费者**：panflute buff FX 与本页是不同注册名，调参时勿混改。[辨析](../../../raw/current/scripts/prefabs/wortox.lua#L1254-L1280)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；77 行工厂全文、DoHeal 主流程与数值链、两个 DoHeal 调用者、帧驱动色调区间、同名 bank 辨析、译名检索已核验；动画资源内容、heal 动画时长、colouradder 联网显示路径未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
