---
title: "落水水花（splash 族）"
created: "2026-10-30"
updated: "2026-11-08"
type: prefab
summary: "通用表五种水花：基础落水 splash、热物落水 hot_splash（二段蒸汽声）、青蛙入池 frogsplash、旧海洋面 splash_ocean 与灭火撞击 splash_snow_fx 的变体差异与生成者。"
aliases: ["splash", "hot_splash", "frogsplash", "splash_ocean", "splash_snow_fx", "水花", "落水水花", "热水花", "蒸汽水花", "青蛙水花", "海面水花", "雪花溅"]
tags: [prefab, fx]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/fx.lua", "fx.lua", "components/floater.lua", "prefabs/frog.lua", "components/boatleak.lua", "prefabs/wave.lua", "prefabs/oceanfish.lua", "prefabs/wobster.lua", "stategraphs/SGwilson.lua", "prefabs/books.lua", "prefabs/oceanfishingbobber.lua", "prefabs/shadowmeteor.lua", "prefabs/monkeyprojectile.lua", "ocean_util.lua", "prefabs/snowball_item.lua", "prefabs/waterprojectiles.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 落水水花（splash 族）

## 快速判断

- 职责与适用：物品／生物／投射物接触水面或被水系命中时的纯表现水花，五种注册同族不同配置；无玩法组件，加湿等效果由生成者（如 wateryprotection）另行实现。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L70-L110)
- 优先使用：查"落水溅／水花声"选本页；物品落水的冷热分支读 [[reference/components/floater|floater]] 的落水流程。
- 重要副作用：无；`splash`／`hot_splash`／`frogsplash` 带 FinalOffset 1 提升显示层。[fn](../../../raw/current/scripts/fx.lua#L77-L86)
- 修改边界：本族多为一次性落点（`SetPosition`）；玩家 tool_slip 是父挂＋符号跟随用例（代理挂玩家并跟随手持符号）。动画是否循环未核验。[父挂用例](../../../raw/current/scripts/stategraphs/SGwilson.lua#L15100-L15103)、[代理回收](../../../raw/current/scripts/prefabs/fx.lua#L156-L157)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `splash` | 通用表条目 → 加载器代理＋一帧后本地实体，fn=FinalOffset1 | bank／build=`splash`，anim=`splash`，音效 `water/splash/bird`。[条目](../../../raw/current/scripts/fx.lua#L80-L86) |
| `hot_splash` | 同上协议 | 与 `splash` 同 bank／anim，多一段 `sound2`（thumper 蒸汽声，延迟 2×FRAMES）——热物落水的嘶声。[条目](../../../raw/current/scripts/fx.lua#L70-L78) |
| `frogsplash` | 同上协议 | bank／build=`splash`，anim=`splash`，音效换 `dontstarve/frog/splash`。[条目](../../../raw/current/scripts/fx.lua#L104-L110) |
| `splash_ocean` | 同上协议，无 fn | build 换 `splash_ocean`，anim=`idle`；数据行注释"this is for the old ocean"，但当前仍有多处活跃生成点（见下表）。[条目](../../../raw/current/scripts/fx.lua#L321-L326) |
| `splash_snow_fx` | 同上协议，无 fn | build 换 `splash_snow`，anim=`idle`，音效 `firesupressor_impact`（灭火器撞击）。[条目](../../../raw/current/scripts/fx.lua#L646-L651) |

定向检索简中 .po 未发现这 5 个注册名的 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目，配置差异仅 bank／build／anim／音效 | 全族同协议 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L70-L110) |
| 生成者：floater 落水分支 | 物品落水且未被打包／持有：先加湿、降温，温度下降量超过 `TUNING.FLOATER_HOT_SIZZLE_THRESHOLD` 才选 `hot_splash`，否则 `splash` | 服务端物品侧触发；`self.splash` 关闭时不生成 | [落水分支](../../../raw/current/scripts/components/floater.lua#L150-L174)、[[reference/components/floater|floater]] |
| 生成者：frog | 青蛙回巢（OnGoingHome）时在原地播 `frogsplash` | 服务端；与跳跃入池共用音效资源 | [生成](../../../raw/current/scripts/prefabs/frog.lua#L93-L95) |
| `splash` 消费群（代表例，非穷举） | 船漏喷溅 [boatleak](../../../raw/current/scripts/components/boatleak.lua#L181-L184)；波浪扫过 [wave](../../../raw/current/scripts/prefabs/wave.lua#L25-L29)；海鱼入水／挣扎 [oceanfish](../../../raw/current/scripts/prefabs/oceanfish.lua#L64-L70)、[wobster](../../../raw/current/scripts/prefabs/wobster.lua#L100-L106)；洞穴蝙蝠／饼干切割者等 SG 状态 | 服务端各触发点自行定位，多为落点式 | [boatleak 记载](../../../raw/current/scripts/components/boatleak.lua#L113-L159) |
| 父挂用例：tool_slip | 玩家手持工具打滑（SG `tool_slip`）：生成 `splash` 后 `SetParent` 挂玩家，`AddFollower`＋`FollowSymbol` 跟随 `swap_object` 手持符号 | 服务端 SG onenter；代理挂接随网络复制，本地表现对符号动画的持续贴合未核验 | [tool_slip](../../../raw/current/scripts/stategraphs/SGwilson.lua#L15100-L15103) |
| `splash_ocean` 生成点 | 触手书逐根落点 [books](../../../raw/current/scripts/prefabs/books.lua#L250-L257)；海钓空竿 [oceanfishingbobber](../../../raw/current/scripts/prefabs/oceanfishingbobber.lua#L70-L75)；暗影流星落无效地面 [shadowmeteor](../../../raw/current/scripts/prefabs/shadowmeteor.lua#L60-L68)；猴子便便落水／落洞 [monkeyprojectile](../../../raw/current/scripts/prefabs/monkeyprojectile.lua#L12-L20)；沉没出地图回退 [ocean_util](../../../raw/current/scripts/ocean_util.lua#L188-L196) | "old ocean"注释仅指资源年代；以上生成点 26230496 均活跃 | [FALLBACK 注释](../../../raw/current/scripts/ocean_util.lua#L190-L191) |
| `splash_snow_fx` 生成点 | 雪球命中目标：先 `SpreadProtection(target)` 加湿，后生成水花 [snowball_item](../../../raw/current/scripts/prefabs/snowball_item.lua#L42-L46)；水系投射物 OnHitSnow：先生成水花，再 `SpreadProtection`，最后移除 [waterprojectiles](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L95-L99) | 命中位置取目标／投射物位置；灭火器撞击音效 | [投射物依赖](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L8-L8) |
| 资源 | bank=`splash`，build 分 `splash`／`splash_ocean`／`splash_snow` | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L70-L110) |

## 典型接入

### 借用基础水花或热物落水

- 前提与选择：投影物／物品落水需要表现时选 `splash`；模拟"热物体入水嘶声"选 `hot_splash`，其热度语义由生成者判定（floater 用温度下降量阈值，非物体标签）。
- 生效与撤销：一次性落点动画，animover 自删；无需清理。
- 原版依据：floater 冷热分支 [floater.lua](../../../raw/current/scripts/components/floater.lua#L150-L174)。

```lua
local fx = SpawnPrefab(hot and "hot_splash" or "splash")
fx.Transform:SetPosition(x, y, z)
```

- 调整与验证：想复刻 floater 判定需自取物品 `inventoryitem` 温度前后差并与 `TUNING.FLOATER_HOT_SIZZLE_THRESHOLD` 比较；阈值数值以 tuning.lua 为准，本页未展开。

## 常见陷阱与限制

- **误区：splash_ocean 已废弃**：数据行注释"for the old ocean"只说明资源来自旧海洋系统；触手书、海钓空竿、暗影流星、猴子便便与出图沉没回退当前仍在生成它。删改前先核对上述生成点。[生成点](../../../raw/current/scripts/fx.lua#L321-L326)
- **边界：水花不等于加湿**：加湿由生成者调用 `wateryprotection:SpreadProtection` 等实现（雪球／水球路径）；`splash` 本身无任何组件。[雪球](../../../raw/current/scripts/prefabs/snowball_item.lua#L42-L46)
- **误区：hot_splash 自动跟随"火热"标签**：floater 的选择条件是落水前后 `inventoryitem` 温度百分比下降量，携带高温标签但温度已低不会触发热分支。[判定](../../../raw/current/scripts/components/floater.lua#L156-L162)
- **边界：动画是否循环未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。五条目配置、加载器协议、floater 冷热分支、frog／雪球／水球与 splash_ocean 五处生成点逐条核验；`splash` 大消费群仅代表例抽样，未穷举全部调用点。.po 名称键定向核验无结果。
- 游戏运行：未运行；水花视觉与音效触发时机未实测。

返回 [[reference/prefabs/fx-water-index|水面、航海与船件]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-07）。
