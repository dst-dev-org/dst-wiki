---
title: "灵魂入体特效（wortox_soul_in_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "灵魂投射物命中接收者时在落点生成的联网特效：经 net_entity 同步目标，客户端按 FX 动画帧给目标推红暗色调并播灵魂音；发放灵魂由宿主 OnHit 完成。"
aliases: ["wortox_soul_in_fx", "灵魂入体特效", "灵魂命中特效", "soul in fx"]
tags: [prefab, fx, network-sync]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/wortox_soul_in_fx.lua", "prefabs/wortox_soul_spawn.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 灵魂入体特效（wortox_soul_in_fx）

## 快速判断

- 职责与适用：灵魂投射物（wortox_soul_spawn，灵魂物族官方译名"灵魂"，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L351880-L351882)）命中接收者时，在命中点生成的联网 FX：bank／build `wortox_soul_ball` 从 `idle_pst` 第 6 帧起播、0.8 缩放、FinalOffset 3；主模拟 `Setup(target)` 记录目标并给目标推红暗色调，同时在目标身上播 `soul/spawn` 音。把灵魂物品塞进接收者背包的是宿主 OnHit，不是本 FX。[工厂](../../../raw/current/scripts/prefabs/wortox_soul_in_fx.lua#L70-L88)
- 优先使用：查"灵魂命中／入体特效"选本页；投射物飞行途中的目标变色是另一个机制，见下文辨析。
- 重要副作用：目标 AnimState 高亮／乘法色被临时改写（10 帧渐变后恢复）；目标中途无效则立即停止。[色调](../../../raw/current/scripts/prefabs/wortox_soul_in_fx.lua#L25-L41)
- 修改边界：net_entity `_target` 联网同步目标；专服不做本地色调（Setup 里跳过），客户端经 `targetdirty` 事件执行。[同步](../../../raw/current/scripts/prefabs/wortox_soul_in_fx.lua#L86-L94)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `wortox_soul_in_fx` | fn：Transform＋AnimState＋Network → 播 `idle_pst`＋SetFrame(6) → net_entity `_target` → SetPristine → 客户端挂 `targetdirty` 监听后早退 → 主模拟挂 `animover` 自删、`persists=false`、暴露 `Setup` | 单一注册；`PlayAnimation("idle_pst")` 未显式传循环参数，实际是否重复未核验。[注册](../../../raw/current/scripts/prefabs/wortox_soul_in_fx.lua#L103)、[主模拟](../../../raw/current/scripts/prefabs/wortox_soul_in_fx.lua#L96-L98) |

`Setup`（主模拟）：`_target:set(target)`，非专服再本地直接调 `OnTargetDirty`，最后在目标 SoundEmitter 播音。[Setup](../../../raw/current/scripts/prefabs/wortox_soul_in_fx.lua#L60-L68)

标题中文为已核验效果描述，非官方译名（NAMES／SKIN_NAMES 未发现该 FX 注册名条目）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成点：wortox_soul_spawn.OnHit | 投射物命中接收者时在命中点生成、`SetPosition` 后 `fx:Setup(target)`；随后把 wortox_soul 物品给目标（背包开）或掉落 | 目标是灵魂投射物 OnHitFn 的 target | [OnHit](../../../raw/current/scripts/prefabs/wortox_soul_spawn.lua#L70-L87) |
| Replica／同步：`_target` | `net_entity`＋`targetdirty` 事件；客户端监听后取 `_target:value()` 作为染色目标 | 客户端主路径；专服不建 updatelooper／不染色 | [netvar](../../../raw/current/scripts/prefabs/wortox_soul_in_fx.lua#L86)、[客户端监听](../../../raw/current/scripts/prefabs/wortox_soul_in_fx.lua#L90-L94) |
| 目标色调 | 帧驱动：FX 动画帧 <10 时 k=(帧/10)²，PushColour(加红 (1-k)·0.7, 乘法色 k·0.7+0.3)；到帧后 PopColour | 无 [[reference/components/highlight#关键接口]] 组件才染（有则只复位乘法色）；updatelooper 驱动 | [PushColour](../../../raw/current/scripts/prefabs/wortox_soul_in_fx.lua#L9-L16)、[更新区间](../../../raw/current/scripts/prefabs/wortox_soul_in_fx.lua#L25-L41)、[[reference/components/updatelooper#关键接口]] |
| 清理协议 | `animover` 自删（主模拟）；提前删除时 `OnRemoveEntity` 对目标 PopColour | 目标中途无效只移除更新函数并摘掉 OnRemoveEntity | [自删](../../../raw/current/scripts/prefabs/wortox_soul_in_fx.lua#L96)、[回落](../../../raw/current/scripts/prefabs/wortox_soul_in_fx.lua#L43-L47)、[失效](../../../raw/current/scripts/prefabs/wortox_soul_in_fx.lua#L38-L40) |
| 同族辨析：投射物本体 tint | wortox_soul_spawn 自带 15 帧渐变 tint 逻辑（同 TINT 常量、同 highlight 门控），飞行途中染色的是投射物不是本 FX | 两套机制共存，勿混改 | [投射物 tint](../../../raw/current/scripts/prefabs/wortox_soul_spawn.lua#L297-L330) |
| 动画资源 | bank／build `wortox_soul_ball`（与灵魂物品同包），anim `idle_pst` 起播帧 6 | 资源包内容未解包 | [设置](../../../raw/current/scripts/prefabs/wortox_soul_in_fx.lua#L77-L82) |

## 典型接入

### 复用为"飞向目标后生效"的提示特效

- 前提与选择：主模拟生成后 `SetPosition`＋`Setup(target)`；客户端自动经 netvar 跟进，无需各端重复调用。
- 生效与撤销：色调 10 帧内完成、FX 按 animover 退场；提前 Remove 由 OnRemoveEntity 恢复目标外观。

```lua
local fx = SpawnPrefab("wortox_soul_in_fx")
fx.Transform:SetPosition(x, y, z)
fx:Setup(target)
```

- 调整与验证：染色强度写死（0.7 系数＋TINT 暗红 154/23/19），不能参数化；目标带 [[reference/components/highlight#关键接口]] 组件时自动跳过染红，只复位乘法色。[门控](../../../raw/current/scripts/prefabs/wortox_soul_in_fx.lua#L10-L15)

## 常见陷阱与限制

- **误区：FX 负责发放灵魂**：`GiveItem`／掉落都在宿主 OnHit；FX 只表现命中。[宿主](../../../raw/current/scripts/prefabs/wortox_soul_spawn.lua#L77-L84)
- **误区：专服也会染色**：`Setup` 只在非专服本地调 `OnTargetDirty`，专服只有 netvar 赋值；色调全部由客户端（含主机客户端）执行。[端分界](../../../raw/current/scripts/prefabs/wortox_soul_in_fx.lua#L62-L64)
- **边界：起播帧 6**：`SetFrame(6)` 硬编码从 `idle_pst` 第 6 帧起播，动画总长与该帧截取意图未核验。[起播](../../../raw/current/scripts/prefabs/wortox_soul_in_fx.lua#L79-L80)
- **边界：目标失效即放弃**：tint 期间目标 IsValid 失败只移除更新函数，不补播任何表现；与 wortox_soul_heal_fx 不同，本厂有 OnRemoveEntity 回落。[对比](../../../raw/current/scripts/prefabs/wortox_soul_in_fx.lua#L43-L47)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；103 行工厂全文、OnHit 生成点与灵魂发放、netvar 同步与客户端监听、帧驱动色调与 highlight 门控、投射物 tint 辨析、译名检索已核验；动画资源内容、idle_pst 时长未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
