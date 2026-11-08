---
title: "幽灵复活转生光效（ghost_transform_overlay_fx）"
created: "2026-10-29"
updated: "2026-11-08"
type: prefab
summary: "玩家被 reviver 类物品复活（TellTale Heart 路径）时叠加在玩家身上的转生表现：shudder→brace→transform 复活动画＋复刻幽灵灯光的逐帧闪变光效；单注册名，由 SGwilson reviver_rebirth 状态生成并挂到玩家实体。"
aliases: ["ghost_transform_overlay_fx", "复活转生特效", "幽灵转生", "复活光效"]
tags: [prefab, fx, server-client]
size_class: simple
dst_build: "steam-buildid:26474873"
sources: ["prefabs/ghost_transform_overlay_fx.lua", "stategraphs/SGwilson.lua", "prefabs/player_common_extensions.lua", "prefabs/player_common.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 幽灵复活转生光效（ghost_transform_overlay_fx）

## 快速判断

- 职责与适用：玩家被复活（TellTale Heart 等 reviver 路径）时的**叠层表现 FX**：bank/build `player_revive_fx` 播幽灵转生动画（shudder→brace→transform，与玩家自身动画同款）＋复刻幽灵灯光的逐帧闪变光效；单注册名；生成后 SetParent 到玩家实体跟随显示。[工厂](../../../raw/current/scripts/prefabs/ghost_transform_overlay_fx.lua#L36-L82)
- 优先使用：查"复活时玩家身上的转生动画／白光闪变"选本页；复活来源分派与玩家自身状态机在 player_common_extensions 与 SGwilson `reviver_rebirth`。
- 重要副作用：内置逐帧 Light 动画——半径每帧收 0.067、收尽后 `Enable(false)`，强度每帧 ×0.75 衰减；时间点 6/15/30/40/47/64/74 帧依次 Flash 提亮；动画队列播完 animqueueover→1 秒后自删。[光效更新](../../../raw/current/scripts/prefabs/ghost_transform_overlay_fx.lua#L11-L30)、[时间表](../../../raw/current/scripts/prefabs/ghost_transform_overlay_fx.lua#L60-L67)、[自删](../../../raw/current/scripts/prefabs/ghost_transform_overlay_fx.lua#L32-L34)
- 修改边界：构造段（动画首段、Light 初值、任务表）在 SetPristine 前完成，双端各自执行；brace/transform 追加与移除监听仅主模拟。[端分界](../../../raw/current/scripts/prefabs/ghost_transform_overlay_fx.lua#L69-L79)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `ghost_transform_overlay_fx` | fn：Transform＋AnimState＋Light＋Network → NOCLICK/FX → 播 `shudder`（单参）→ Light 复刻玩家幽灵光（intensity .6／radius .5／falloff .6／色 180,195,225，EnableClientModulation 开）→ _kintensity/_kradius 归零＋每帧 OnUpdate＋7 个 Flash 定时 → SetPristine → 主模拟 persists=false＋PushAnimation `brace`/`transform`（显式 false 不循环）＋animqueueover RemoveMe | [工厂](../../../raw/current/scripts/prefabs/ghost_transform_overlay_fx.lua#L36-L82)、[注册](../../../raw/current/scripts/prefabs/ghost_transform_overlay_fx.lua#L84) |

无 NAMES 条目；标题中文为已核验效果描述，非官方译名。player_common 依赖本注册名（[L2202](../../../raw/current/scripts/prefabs/player_common.lua#L2202)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 消费者：SGwilson reviver_rebirth | 生成并 SetParent 到玩家；玩家切 ghost skin、播同款 shudder→brace→transform、隐 HUD、镜头拉远 14、无敌；88 帧 `transform_end`、96 帧 popbloom | 状态标签 pausepredict/silentmorph/ghostbuild | [状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L15349-L15413)、[生成行](../../../raw/current/scripts/stategraphs/SGwilson.lua#L15363) |
| 触发链 | `respawnfromghost` 事件→OnRespawnFromGhost：source 带 `reviver` 标签→DoActualRez(inst, nil, item=source)→非 pocketwatch 分支→`reviver_rebirth` | TellTale Heart 路径；amulet／复活石／雕像／传送门／WX-78 备份身体等其余来源走各自状态，脚本树内仅此一处生成（grep 单命中） | [事件分派](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L569-L606)、[reviver 判定](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L593-L594)、[DoActualRez](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L327)、[Telltale 分支](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L411-L412) |
| 光效时间线 | Flash(intensity, radius) 以非 nil 参数覆盖 _kintensity/_kradius（不累加）；OnUpdate 每帧：半径>0 时按平方缓动收半径→0 后关灯；强度每帧 ×0.75 衰减 | 双端各自跑（任务表在 SetPristine 前设置）；参数写死无对外接口 | [Flash](../../../raw/current/scripts/prefabs/ghost_transform_overlay_fx.lua#L6-L9)、[OnUpdate](../../../raw/current/scripts/prefabs/ghost_transform_overlay_fx.lua#L11-L30)、[调度](../../../raw/current/scripts/prefabs/ghost_transform_overlay_fx.lua#L60-L67) |
| 资源 | anim/player_revive_fx.zip（bank/build 同名；动画与玩家幽灵转生共用一套） | 声明与使用已核验，包内容未解包 | [声明](../../../raw/current/scripts/prefabs/ghost_transform_overlay_fx.lua#L1-L4) |

## 典型接入

### 复用"实体上的转生闪光"

- 前提与选择：主模拟 SpawnPrefab 后 SetParent 到目标实体；fx 自带动画与光效，无参数可调。
- 生效与撤销：动画队列 shudder→brace→transform 播完 animqueueover 触发 1 秒自删；光效随时间线自动收束关灯。
- 原版依据：reviver_rebirth 的"生成＋SetParent"即全部用法（[生成行](../../../raw/current/scripts/stategraphs/SGwilson.lua#L15363)）。

```lua
local fx = SpawnPrefab("ghost_transform_overlay_fx")
fx.entity:SetParent(inst.entity)  -- 挂到宿主实体上跟随显示
```

- 调整与验证：光效参数写死在工厂；要改亮度需改工厂值或事后操作 `fx.Light`（非原版用法）。

## 常见陷阱与限制

- **边界：只有 reviver 路径生成**：amulet／复活石／雕像／传送门等复活走 DoActualRez 内各自状态分派，均不生成本 FX；不要把它当"通用复活特效"。[分派](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L327-L412)
- **边界：动画是否重复未核验**：`shudder` 单参播放（未显式传循环参数）；`brace`/`transform` PushAnimation 显式 false；设计预期播完自删，别依赖循环。
- **边界：客户端也跑光效任务**：任务表在 SetPristine 前设置，各端独立计算 Light；mod 侧不要重复叠加同样的闪变逻辑。[任务表](../../../raw/current/scripts/prefabs/ghost_transform_overlay_fx.lua#L58-L67)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；85 行工厂全文（工厂、Flash/OnUpdate 光效、端分界）、SGwilson reviver_rebirth 状态、respawnfromghost→OnRespawnFromGhost→DoActualRez 触发链、player_common 依赖、NAMES 检索（无命中）已核验；anim zip 内容、ghost 皮肤模式细节、晚加入时序未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-transition-index|变身、召唤、传送与退场]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-03）。
