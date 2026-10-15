---
title: "漂浮物前后水环（float_fx_front）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "实体浮在海面时的前后两层水环动画：非联网 FX 由 floater 组件在非专用机器本地成对生成、显式循环播放，离开水面移除或随主体 ErodeAway 腐蚀淡出。"
aliases: ["float_fx_front", "float_fx_back", "float_fx", "漂浮水环", "浮物水环"]
tags: [prefab, fx, server-client]
size_class: simple
dst_build: "steam-buildid:26335394"
sources: ["prefabs/float_fx.lua", "components/floater.lua", "prefabs/pumpkin_bomb.lua", "simutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 漂浮物前后水环（float_fx_front）

## 快速判断

- 职责与适用：物品/实体浮在海面时脚下的**前后两层水环动画实体**：非联网（无 AddNetwork），Transform＋AnimState＋SoundEmitter，海洋着色 SetOceanBlendParams；front 层正常深度、back 层压入 LAYER_BACKGROUND。成对使用。[工厂](../../../raw/current/scripts/prefabs/float_fx.lua#L11-L42)
- 优先使用：查"物品浮在水上的一圈水"选本页；主消费者是 floater 组件（客户端落水表现），另 pumpkin_bomb 碎块直接复用。
- 重要副作用：工厂即显式循环播放（`PlayAnimation(anim, true)`），**无自删路径**——寿命完全由生成方管理（移除或 ErodeAway）。[动画](../../../raw/current/scripts/prefabs/float_fx.lua#L26-L33)
- 修改边界：persists=false 在 SetPristine 前设置，双端生成都生效；floater 生成路径只装在非专用机器上（landeddirty 监听包在 `not TheNet:IsDedicated()` 内）。[监听](../../../raw/current/scripts/components/floater.lua#L11-L28)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `float_fx_front` | common_fn(true)：CreateEntity → Transform＋AnimState＋SoundEmitter → persists=false → NOBLOCK/FX/ignorewalkableplatforms 标签 → build `float_fx`＋海洋着色 → bank `float_front`、播 `idle_front_small`（显式循环） | 正常渲染层；默认 small 尺寸动画。[工厂](../../../raw/current/scripts/prefabs/float_fx.lua#L26-L28)、[注册](../../../raw/current/scripts/prefabs/float_fx.lua#L52) |
| `float_fx_back` | common_fn(false)：同上，bank `float_back`、播 `idle_back_small`（显式循环） | 追加 SetLayer(LAYER_BACKGROUND) 背景层。[工厂](../../../raw/current/scripts/prefabs/float_fx.lua#L30-L33)、[注册](../../../raw/current/scripts/prefabs/float_fx.lua#L53) |

两工厂的 SetPristine/mastersim 分支后无任何主模拟初始化（无 AddNetwork 的遗留结构）；两注册名均无 NAMES 条目，标题中文为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：floater（宿主侧） | OnLandedClient 成对生成并 `SetParent`（或独立定位），按 `SetSize` 的 size 覆写播 `idle_front/back_"..size`；离开水面 OnNoLongerLandedClient 移除 | 仅非专用机器；由 `_is_landed` net_bool 的 landeddirty 驱动 | [[reference/components/floater]]；[生成](../../../raw/current/scripts/components/floater.lua#L178-L194)、[移除](../../../raw/current/scripts/components/floater.lua#L224-L236) |
| 净值：floater._is_landed / _erode_time | landeddirty 触发生成/移除；erodetimedirty 对两个 fx 调 ErodeAway(腐蚀时长) | net_bool/net_float，主模拟写、非专用读 | [净值与监听](../../../raw/current/scripts/components/floater.lua#L11-L28)、[L43-L44](../../../raw/current/scripts/components/floater.lua#L43-L44) |
| API：ErodeAway | 腐蚀淡出替代直接移除：默认 1 秒逐帧 SetErosionParams 后 Remove，并回调 floater:Erode | 传入时长可调；simutil 全局函数 | [实现](../../../raw/current/scripts/simutil.lua#L575-L597) |
| 生成者：pumpkin_bomb | 炸碎的南瓜碎块落水时在碎块上成对生成本 fx（显式 small 动画、SetParent）；腐蚀时随碎块一起 ErodeAway | 碎块为非联网本地实体（[碎块工厂](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L194-L216)）；承载爆炸特效的实体联网但其 OnEntityWake 在 SetPristine 前设置（[L301-L312](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L302-L313)），各机唤醒都本地抛掷 | [碎块落水](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L236-L250)、[腐蚀](../../../raw/current/scripts/prefabs/pumpkin_bomb.lua#L252-L260) |
| 资源 | anim/float_fx.zip（build `float_fx`；bank `float_front`/`float_back` 两个） | 声明与使用已核验，包内容未解包 | [资产声明](../../../raw/current/scripts/prefabs/float_fx.lua#L1-L4) |

## 典型接入

### 复用成对漂浮水环

- 前提与选择：非联网实体，SpawnPrefab 后按需 SetParent 或定位；必须自己负责移除——这是与大多数"播完自删"FX 的关键差异。
- 生效与撤销：显式循环动画不会自然结束；离开水面直接 Remove，要淡出用 `ErodeAway(fx)`。
- 原版依据：floater 的生成/移除/腐蚀三段式即标准用法；pumpkin_bomb 是独立定位（SetParent 到碎块）的变体。

```lua
local front = SpawnPrefab("float_fx_front")
front.entity:SetParent(inst.entity)
front.AnimState:PlayAnimation("idle_front_small", true) -- 显式循环
-- back 同理：SpawnPrefab("float_fx_back") + "idle_back_small"
-- 离开水面：front:Remove()；要淡出：ErodeAway(front)
```

- 调整与验证：floater 用 `idle_front_"..size` 覆写尺寸（size 由 SetSize 设置，源码注释 small/med/large）；bank 内非 small 动画是否齐全未解包核验。

## 常见陷阱与限制

- **边界：没有自删路径**：工厂无 animover 监听、动画显式循环（[工厂](../../../raw/current/scripts/prefabs/float_fx.lua#L26-L33)），忘记移除会永久留环；照抄"播完即删"的其他 FX 习惯会泄漏实体。
- **边界：非联网、各机本地生成**：专用服务器上 floater 不挂监听不生成（[floater](../../../raw/current/scripts/components/floater.lua#L11-L18)）；各客户端看到的是自己的本地水环。
- **误区：bank 叫 float_fx**：build 才是 `float_fx`，front/back 各用 `float_front`/`float_back` 两个 bank；换 bank 时别照抄 build 名。[工厂](../../../raw/current/scripts/prefabs/float_fx.lua#L23-L32)
- **边界：SoundEmitter 未被工厂使用**：添加了组件但无任何播放调用；需要水声要自己播。

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；54 行工厂全文、floater 组件落水/移除/腐蚀链与净值、pumpkin_bomb 碎块路径、ErodeAway 实现、NAMES 检索（无命中）已核验；动画包内容与 size 覆写动画齐全性、水面实际渲染效果未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-water-index|水面、航海与船件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
