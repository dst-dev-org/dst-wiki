---
title: "薇洛打火机吸火引导特效（channel_absorb_fire_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "薇洛打火机技能（饥饿打火机）引导吸火的四个特效：channel_absorb_fire_fx 挂在持械符号上持续播放直到 KillFX，channel_absorb_fire／_smoulder／_embers 是每次扑灭燃烧／阴燃／回收以太余烬时播一次的标记；玩法作用（灭火、拾取）在 lighter 的 UpdateSnuff。"
aliases: ["channel_absorb_fire_fx", "channel_absorb_fire", "channel_absorb_smoulder", "channel_absorb_embers", "吸火特效", "饥饿打火机", "打火机吸火"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/channel_absorb_fire_fx.lua", "prefabs/lighter.lua", "prefabs/willow_ember.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 薇洛打火机吸火引导特效（channel_absorb_fire_fx）

## 快速判断

- 职责与适用：[[reference/components/channelcastable|channelcastable]] 引导灭火（薇洛技能 `willow_attuned_lighter`，官方名"饥饿打火机"）的表现族：引导期间打火机上持续播 `channel_absorb_fire_fx`；每 0.3 秒扫描一次 2.5 距离内的燃烧／阴燃实体与以太余烬，扑灭或回收时往玩家手上播对应的一次性标记特效（[注册](../../../raw/current/scripts/prefabs/channel_absorb_fire_fx.lua#L91-L94)、[引导入口](../../../raw/current/scripts/prefabs/lighter.lua#L65-L78)）。
- 优先使用：查"怎么灭火/回收余烬"看 lighter 的 UpdateSnuff；想复用"挂在持械符号上的引导特效"直接借鉴 fire_fx 的 Follower 用法。
- 重要副作用：特效本体无玩法组件——真正灭火（burnable:Extinguish／SmotherSmolder）与余烬拾取都在 lighter 的扫描任务里；且回收余烬会给目标加 `snuffed` 标签并在 10 帧后塞进背包（[UpdateSnuff](../../../raw/current/scripts/prefabs/lighter.lua#L23-L63)）。
- 修改边界：四个注册名同一 bank/build；`channel_absorb_fire_fx` 需要调用方 KillFX 收尾，三个一次性特效 animover 自删；引导入口仅主模拟。[KillFX](../../../raw/current/scripts/prefabs/channel_absorb_fire_fx.lua#L6-L12)、[自删](../../../raw/current/scripts/prefabs/channel_absorb_fire_fx.lua#L67)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `channel_absorb_fire_fx` | fn：Transform＋AnimState＋Follower＋Network → bank/build `channel_absorb_fire_fx`、播 `channel_pre`（未显式传循环参数，实际是否重复未核验）、bloom、LightOverride 1、FX＋NOCLICK → SetPristine → 主模拟 `PushAnimation("channel_loop")`（未显式传循环参数）、`persists=false`、暴露 KillFX；KillFX 幂等（`killed` 标志）：改听 animover 并播 `channel_pst` 后自删 | 引导持握特效；未调用 KillFX 时工厂没有实体删除路径；末帧表现未运行核验。[fn](../../../raw/current/scripts/prefabs/channel_absorb_fire_fx.lua#L14-L44)、[KillFX](../../../raw/current/scripts/prefabs/channel_absorb_fire_fx.lua#L6-L12) |
| `channel_absorb_fire` | CommonFX("channel_fire")：同 bank/build，播单段动画（未显式传循环参数）→ 主模拟 animover 即 Remove | 扑灭正在燃烧的实体时播 | [CommonFX](../../../raw/current/scripts/prefabs/channel_absorb_fire_fx.lua#L46-L70)、[选用](../../../raw/current/scripts/prefabs/lighter.lua#L33-L36) |
| `channel_absorb_smoulder` | CommonFX("channel_smoulders")，同上 | 扑灭阴燃（SmotherSmolder）时播 | [选用](../../../raw/current/scripts/prefabs/lighter.lua#L37-L39) |
| `channel_absorb_embers` | CommonFX("channel_embers")，同上 | 吸回以太余烬（willow_ember 标签）时播 | [选用](../../../raw/current/scripts/prefabs/lighter.lua#L29-L32) |

四个注册名均无 NAMES 条目；标题中文为已核验技能／效果描述（"饥饿打火机"为官方技能译名，[SKILLTREE](../../../raw/current/scripts/languages/chinese_s.po#L366799-L366802)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 宿主：lighter（薇洛的打火机，官方译名 [NAMES](../../../raw/current/scripts/languages/chinese_s.po#L344100-L344102)） | 引导开始：0.3 秒周期 `UpdateSnuff`、播 `lighter_absorb_LP`（"channel_loop" 通道）、生成 channel_absorb_fire_fx 并 `Follower:FollowSymbol(玩家, "swap_object", 56, -40, 0)`；结束/卸下/移除时 KillFX | 仅装备并激活 `willow_attuned_lighter` 技能后才装 channelcastable | [OnStartChanneling](../../../raw/current/scripts/prefabs/lighter.lua#L65-L78)、[技能门控](../../../raw/current/scripts/prefabs/lighter.lua#L116-L124)、[清理](../../../raw/current/scripts/prefabs/lighter.lua#L80-L93)、[OnRemoveEntity](../../../raw/current/scripts/prefabs/lighter.lua#L301-L306) |
| 扫描：UpdateSnuff | 找 2.5 距离内带 `smolder`/`fire`/`willow_ember` 标签且不在 INLIMBO 的实体：余烬→加 `snuffed` 标签＋embers 特效＋10 帧后拾取；燃烧→Extinguish＋fire 特效；阴燃→SmotherSmolder＋smoulder 特效；每次播 `ember_absorb` 音效 | 主模拟任务；特效统一 FollowSymbol 到玩家 swap_object | [扫描](../../../raw/current/scripts/prefabs/lighter.lua#L23-L63)、[标签集](../../../raw/current/scripts/prefabs/lighter.lua#L19-L21)、[拾取](../../../raw/current/scripts/prefabs/lighter.lua#L48-L58) |
| 关联 Prefab：willow_ember | 以太余烬（官方译名，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L351395-L351397)）被吸回时播 idle_pst 并入包，摘 `snuffed` 后重播 idle | 需目标带 `willow_ember` 标签（[打标](../../../raw/current/scripts/prefabs/willow_ember.lua#L937)） | [选用](../../../raw/current/scripts/prefabs/lighter.lua#L29-L32) |
| 组件：Follower | 引导特效与一次性特效都挂在玩家 `swap_object` 符号（手持物位置） | 客户端跟随依赖玩家 AnimState 符号存在 | [FollowSymbol](../../../raw/current/scripts/prefabs/lighter.lua#L46)、[[reference/components/follower]] |
| 资源 | anim/channel_absorb_fire_fx.zip（bank/build 同名）；音效 `meta3/willow_lighter/ember_absorb`、`lighter_absorb_LP` | 声明即核验范围 | [assets](../../../raw/current/scripts/prefabs/channel_absorb_fire_fx.lua#L1-L4)、[音效](../../../raw/current/scripts/prefabs/lighter.lua#L44) |

## 典型接入

### 复用"挂在手持物上的引导特效"

- 前提与选择：主模拟生成 `channel_absorb_fire_fx` 后立即 `Follower:FollowSymbol(用户, "swap_object", 偏移…)`；持续表现靠 `channel_pre→channel_loop`，退出必须调 `KillFX()`。
- 生效与撤销：KillFX 幂等可重复调用；忘记调用会留下无自删任务的特效实体（工厂不自动结束）。
- 原版依据：lighter 的 OnStartChanneling／OnStopChanneling／OnRemoveEntity 三处成对调用即官方节奏。

```lua
inst.snuff_fx = SpawnPrefab("channel_absorb_fire_fx")
inst.snuff_fx.Follower:FollowSymbol(user.GUID, "swap_object", 56, -40, 0)
-- 结束时：
inst.snuff_fx:KillFX()
```

- 调整与验证：偏移 56/-40 对应打火机持械位；一次性标记换 `channel_absorb_fire` 等三个注册名即可，无需管理生命周期（animover 自删）。

## 常见陷阱与限制

- **边界：引导特效不自动结束**：`channel_loop` 之后工厂没有 animover 回退——不调 KillFX 就没有工厂内建的实体删除路径；三个一次性特效则相反，无需 KillFX。[对比](../../../raw/current/scripts/prefabs/channel_absorb_fire_fx.lua#L6-L12)
- **误区：特效会灭火**：灭火/回收是 lighter 扫描任务的行为，特效只是伴随标记；只生成特效不会改动任何 burnable 状态（[职责分界](../../../raw/current/scripts/prefabs/lighter.lua#L23-L43)）。
- **边界：扫描按标签不按组件**：`willow_ember` 标签优先于 burnable 分支——带该标签的余烬即使可燃也走回收；`snuffed` 标签在拾取后才摘除，期间不会被重复扫描（[分支顺序](../../../raw/current/scripts/prefabs/lighter.lua#L29-L41)）。
- **边界：无 NPC／怪物的 NAME 译名**：四个注册名在 po 无条目，写文档时引用"饥饿打火机"技能名需注明 SKILLTREE 出处。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；工厂全文（四工厂与 KillFX 幂等）、lighter 的技能门控／引导开关／UpdateSnuff 全程、willow_ember 标签、技能与物品译名已核验；`channel_*` 动画实际帧数、channelcaster 组件内部、引导动作的客户端预测未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-character-index|角色技能、施法与读书]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
