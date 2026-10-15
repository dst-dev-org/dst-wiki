---
title: "武器命中火花（weaponsparks 族）"
created: "2026-10-30"
updated: "2026-11-01"
type: prefab
summary: "熔炉血统的武器命中火花四注册：基础／弹跳单段、穿刺／突刺双段镜像（3 帧后对侧补背面）；主模拟初始化来自 Forge 事件数据包，快照内无生成点。"
aliases: ["weaponsparks", "weaponsparks_piercing", "weaponsparks_thrusting", "weaponsparks_bounce", "武器命中火花", "武器命中特效", "熔炉火花", "穿刺火花", "突刺火花", "弹跳火花"]
tags: [prefab, fx, network-sync]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/weaponsparks.lua", "entityscript.lua", "prefabs/lavaarena_lucy.lua", "prefabs/blowdart_lava.lua", "prefabs/spear_gungnir.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 武器命中火花（weaponsparks 族）

## 快速判断

- 职责与适用：武器命中目标的地面火花表现，4 注册共用 `lavaarena_hit_sparks_fx` 资源；**独立工厂**（熔炉 Forge 血统），不走 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]] 协议。[注册](../../../raw/current/scripts/prefabs/weaponsparks.lua#L163-L166)
- 优先使用：查"武器命中火花／熔炉火花"选本页；玩法结算归各武器链，本工厂纯表现。
- 重要副作用：无音效、无灯光、无临时组件；本地表现与背面段都靠 animover 自删。[本地实体](../../../raw/current/scripts/prefabs/weaponsparks.lua#L8-L29)
- 修改边界：**主模拟初始化来自 Forge 事件数据包**（快照 scripts 内不可读）；快照 scripts 内未检索到任何生成点；动画是否重复未核验。[事件数据装载](../../../raw/current/scripts/entityscript.lua#L77-L87)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `weaponsparks` | `MakeSparks(name, fxfn, variation, doubleflip)` → 代理（Transform＋Network）非专服一帧后调 fxfn；主模拟交给事件数据 `master_postinit(inst, "normal")` | `PlaySparksAnim`：anim `hit_3`、FinalOffset 1、泛光、缩放 ±0.7（flip 为 net_bool）；不父挂、贴代理位置。[基础](../../../raw/current/scripts/prefabs/weaponsparks.lua#L8-L29)、[注册](../../../raw/current/scripts/prefabs/weaponsparks.lua#L163-L163) |
| `weaponsparks_piercing` | 同上，`master_postinit(inst, "piercing")`，flip 为 net_tinybyte（doubleflip） | `Piercing_PlaySparksAnim`：anim `hit_3`＋OnGround；**父挂时按父级世界位重算落点**，3 帧后在镜像对侧补播背面段 `hit_2`（glow 隐藏、旋转＋180）。[正面](../../../raw/current/scripts/prefabs/weaponsparks.lua#L60-L93)、[背面](../../../raw/current/scripts/prefabs/weaponsparks.lua#L33-L58) |
| `weaponsparks_thrusting` | 同上，`master_postinit(inst, "piercing")`，flip 为 net_tinybyte | 包装 `Piercing_PlaySparksAnim(proxy, 1)`——同穿刺但整体抬高 1。[包装](../../../raw/current/scripts/prefabs/weaponsparks.lua#L97-L99)、[注册](../../../raw/current/scripts/prefabs/weaponsparks.lua#L165-L165) |
| `weaponsparks_bounce` | 同基础协议，`master_postinit(inst, "normal")`，flip 为 net_bool | `Bounce_PlaySparksAnim`：anim `hit_2`（glow 隐藏）、按 flip 翻转；无背面段、无父挂重算。[弹跳](../../../raw/current/scripts/prefabs/weaponsparks.lua#L103-L127)、[注册](../../../raw/current/scripts/prefabs/weaponsparks.lua#L166-L166) |

四注册在简中 .po 均无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。本地表现统一：bank=`hits_sparks`、build=`lavaarena_hit_sparks_fx`、非网络实体、animover 自删。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 代理非专服延迟一帧建本地表现（注释：等定位落定或临移除时不建）；flip 按是否 doubleflip 用 net_bool／net_tinybyte | 客户端 SetPristine 即返；专服不建表现 | [工厂](../../../raw/current/scripts/prefabs/weaponsparks.lua#L131-L161) |
| 主模拟初始化 | `event_server_data("lavaarena", "prefabs/weaponsparks").master_postinit(inst, variation, doubleflip)`——装载 `lavaarena_event_server/` 事件数据包（快照 scripts 之外） | 服务端；代理寿命与 flip 写入由事件数据决定，快照内未定义 | [装载器](../../../raw/current/scripts/entityscript.lua#L77-L87)、[调用](../../../raw/current/scripts/prefabs/weaponsparks.lua#L155-L155) |
| 生成者 | **快照 scripts 内未检索到 `SpawnPrefab("weaponsparks…")`**；仅依赖声明：熔炉露西 [lavaarena_lucy](../../../raw/current/scripts/prefabs/lavaarena_lucy.lua#L17-L19)、火镖 [blowdart_lava](../../../raw/current/scripts/prefabs/blowdart_lava.lua#L22-L22)（blowdart_lava2 同）、冈格尼尔 [spear_gungnir](../../../raw/current/scripts/prefabs/spear_gungnir.lua#L17-L17) | 生成与 flip 赋值由 Forge 事件数据驱动，未核验；依赖声明不等于生成 | [声明](../../../raw/current/scripts/prefabs/lavaarena_lucy.lua#L17-L19) |
| 穿刺双段几何 | 正面段先 `SetFromProxy`；若代理有父级，再按父级世界位＋偏移重算落点（thrusting 另加 yoffset 1）；3 帧后背面段在 `父级位−偏移`、旋转＋180 处播放，正面翻转看 flip>2、背面翻转看 flip==2 或 4 | 父挂由生成方建立；无父级时背面段不生成 | [重算](../../../raw/current/scripts/prefabs/weaponsparks.lua#L72-L80)、[背面翻转](../../../raw/current/scripts/prefabs/weaponsparks.lua#L53-L55) |
| 回收 | 全部本地实体（含背面段）animover 自删；代理寿命快照内未定义 | 各模拟独立；动画是否重复未核验 | [animover](../../../raw/current/scripts/prefabs/weaponsparks.lua#L28-L28) |

## 典型接入

### 工厂行为参照（非普通世界接入承诺）

- 前提与选择：四个变体的主模拟构造都**无条件**执行事件包 `master_postinit`，其实现未核验；下例只描述工厂代码在生成后一帧的行为，把它当**实现参照**——在 Forge 事件数据可用的环境之外的直接生成属未验证路径，不能承诺普通世界直接可用。
- 生效与撤销：基础／弹跳变体的本地表现为单段动画（animover 自删）；穿刺／突刺需要**父挂代理**才安排双段（工厂只判父级是否存在，[weaponsparks](../../../raw/current/scripts/prefabs/weaponsparks.lua#L73-L73)），flip 只决定正／背面段的翻转方向，不决定是否出双段。
- 原版依据：[基础本地实体](../../../raw/current/scripts/prefabs/weaponsparks.lua#L8-L29)、[无条件 master_postinit](../../../raw/current/scripts/prefabs/weaponsparks.lua#L155-L155)。

```lua
-- 实现参照：事件数据环境下 weaponsparks_bounce 命中后的表现行为
local fx = SpawnPrefab("weaponsparks_bounce")
fx.Transform:SetPosition(x, y, z)   -- 单段表现；animover 自删；代理寿命由事件数据决定
```

- 调整与验证：flip 取值的语义（bool 翻转、tinybyte 四值）由事件数据写入端决定，快照只见消费分支；脱离事件数据自行赋值前先确认与该变体 local 表现的对应关系。

## 常见陷阱与限制

- **误区：按 piercing／thrusting／bounce 名字推断玩法**：穿刺／突刺／弹跳只是表现形态（双段镜像／抬高／单段），穿透、突进等玩法结算全在武器与事件数据链；本工厂无任何组件。[注册](../../../raw/current/scripts/prefabs/weaponsparks.lua#L163-L166)
- **边界：主模拟行为在快照之外**：`master_postinit` 来自 `lavaarena_event_server` 数据包，代理寿命、flip 写入时机、生成调用都不可在快照内核验；引用本页结论时区分"工厂代码已核验"与"事件数据未核验"。[装载器](../../../raw/current/scripts/entityscript.lua#L77-L87)
- **边界：穿刺背面段依赖父挂**：代理无父级时跳过重算与背面段，只留正面单段；thrusting 的 yoffset 也只在父挂路径生效。[重算](../../../raw/current/scripts/prefabs/weaponsparks.lua#L72-L80)
- **边界：flip 两套取值**：normal／bounce 是 net_bool，piercing／thrusting 是 net_tinybyte（>2 正面翻转、2 或 4 背面翻转）；同一字段名不同类型，跨变体赋值会错位。[字段](../../../raw/current/scripts/prefabs/weaponsparks.lua#L147-L147)
- **边界：动画是否重复未核验**：本地实体只调一次 `PlayAnimation` 并监听 animover，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/weaponsparks.lua#L23-L23)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。工厂全文（四注册装配、flip 双类型、四段本地表现函数、穿刺双段几何与翻转分支、事件数据装载入口）、依赖声明与 .po 名称键核验完毕；快照内无生成点经全库检索确认。未覆盖：`lavaarena_event_server` 数据包内容（master_postinit／生成调用／flip 赋值）、`hit_2`／`hit_3` 动画资源内容、实际视觉与晚加入表现。
- 游戏运行：未运行；双段镜像与翻转表现未实测。

返回 [[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
