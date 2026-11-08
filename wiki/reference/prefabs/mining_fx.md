---
title: "采石碎屑（mining_fx）"
created: "2026-10-30"
updated: "2026-11-08"
type: prefab
summary: "共享采矿表现的普通与冰色两注册：commonstates 的 PlayMiningFX 按目标标签选名（frozen→冰、月晶与水晶→组外注册、其余→普通），玩家啃咬、猎犬与月兔直落点、蚁狮陷坑等十余处调用。"
aliases: ["mining_fx", "mining_ice_fx", "采矿碎屑", "挖矿石特效", "冰屑特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/fx.lua", "fx.lua", "stategraphs/commonstates.lua", "stategraphs/SGwilson.lua", "stategraphs/SGhound.lua", "stategraphs/SGmoonpig.lua", "prefabs/antlion_sinkhole.lua", "prefabs/eyeofterror_sinkhole.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 采石碎屑（mining_fx）

## 快速判断

- 职责与适用：挖矿／敲击动作的碎屑表现，两注册（普通／冰色 build）；选名集中在 commonstates 的 `PlayMiningFX`，按目标标签分流。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目](../../../raw/current/scripts/fx.lua#L340-L350)
- 优先使用：查"采矿碎屑／冰屑特效"选本页；可工作矿石的挖矿结算读 [[reference/components/workable|workable]]。
- 重要副作用：无音效字段（PlayMiningFX 有 `nosound` 参数，声在其内部另播）。[选名](../../../raw/current/scripts/stategraphs/commonstates.lua#L2448-L2458)
- 修改边界：目标位置一次性落点（不父挂）；动画是否重复未核验。[落点](../../../raw/current/scripts/stategraphs/SGhound.lua#L771-L771)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `mining_fx` | 通用表条目 → 加载器代理＋一帧后本地实体，无 fn | bank／build=`mining_fx`，anim=`anim`。[条目](../../../raw/current/scripts/fx.lua#L340-L344) |
| `mining_ice_fx` | 同上协议 | bank 同 `mining_fx`、build 换 `mining_ice_fx`（同 bank 异 build 的冰色变体）。[条目](../../../raw/current/scripts/fx.lua#L346-L350) |

两注册在简中 .po 均无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 通用表条目，无附加字段 | 两名同协议 | [[reference/mechanisms/generic-fx-factory]]；[条目](../../../raw/current/scripts/fx.lua#L340-L350) |
| 选名入口：PlayMiningFX | commonstates.lua 声明的**全局函数** `PlayMiningFX(inst, target, nosound)`：目标有效时按标签选名——`frozen`→`mining_ice_fx`；`moonglass`／`LunarBuildup`→组外 `mining_moonglass_fx`；`crystal`→组外 `mining_crystal_fx`；**其余→`mining_fx`**；判定按此顺序短路（`mining_charged_moonglass_fx` 不在其分支内） | 服务端；共享 helper，被多状态图引用 | [选名](../../../raw/current/scripts/stategraphs/commonstates.lua#L2448-L2458) |
| 生成者：玩家啃咬 | SGwilson `gnaw` 状态（海狸啃咬同时处理采矿与挖掘）：mine 分支调 PlayMiningFX、DIG 分支落 [[reference/prefabs/shovel_dirt|shovel_dirt]] | 服务端；动作分支互斥 | [gnaw 双分支](../../../raw/current/scripts/stategraphs/SGwilson.lua#L5490-L5498) |
| 生成者：直落点代表例 | 猎犬月变挖月基 [SGhound](../../../raw/current/scripts/stategraphs/SGhound.lua#L771-L771)、月兔挖月基 [SGmoonpig](../../../raw/current/scripts/stategraphs/SGmoonpig.lua#L273-L273)——直接 `SpawnPrefab("mining_fx")` 于月基位置 | 服务端；不经 PlayMiningFX（固定普通版） | [对照](../../../raw/current/scripts/stategraphs/commonstates.lua#L2448-L2458) |
| 其他调用面 | 蚁狮／眼球陷坑 [antlion_sinkhole](../../../raw/current/scripts/prefabs/antlion_sinkhole.lua#L148-L148)、[eyeofterror_sinkhole](../../../raw/current/scripts/prefabs/eyeofterror_sinkhole.lua#L125-L125)；隐士蟹、WX78、鱼人、暗影麦斯威尔等状态图（`PlayMiningFX` 调用约十余处，代表例非穷举） | 服务端 | [调用面](../../../raw/current/scripts/stategraphs/commonstates.lua#L2448-L2458) |
| 依赖声明 | antlion_sinkhole prefabs 声明两注册 | 仅声明 | [声明](../../../raw/current/scripts/prefabs/antlion_sinkhole.lua#L14-L15) |
| 同名组外注册 | `mining_moonglass_fx`／`mining_charged_moonglass_fx`／`mining_crystal_fx` 是 fx.lua 内**其他条目**；PlayMiningFX 只选用其中 moonglass 与 crystal 两名，`mining_charged_moonglass_fx` 不在其分支 | 独立注册 | [相邻条目](../../../raw/current/scripts/fx.lua#L1855-L1861) |
| 资源 | bank=`mining_fx`，build 分 `mining_fx`／`mining_ice_fx` | 动画时长与循环性未核验 | [条目](../../../raw/current/scripts/fx.lua#L340-L350) |

## 典型接入

### 复用采石碎屑

- 前提与选择：挖矿动作应调 `PlayMiningFX(inst, target)` 复用标签分流（冻结目标自动换冰色、月晶水晶自动换组外注册）；固定普通版可直 Spawn。
- 生效与撤销：落点一次性动画，animover 自删。
- 原版依据：[PlayMiningFX](../../../raw/current/scripts/stategraphs/commonstates.lua#L2448-L2458)。

```lua
PlayMiningFX(inst, target)        -- 按目标标签选普通／冰／组外注册
-- 或固定普通版：
local fx = SpawnPrefab("mining_fx")
fx.Transform:SetPosition(target.Transform:GetWorldPosition())
```

- 调整与验证：`nosound` 传 true 可静音；标签判定顺序 frozen→moonglass→crystal→默认，冻结目标即使带月晶标签也出冰色。

## 常见陷阱与限制

- **边界：三名相邻注册不属本页**：PlayMiningFX 只选用 `mining_moonglass_fx`（月晶分支）与 `mining_crystal_fx`（水晶分支），**不选择 `mining_charged_moonglass_fx`**；引用时按目标标签对应，不要当本页注册。[选名](../../../raw/current/scripts/stategraphs/commonstates.lua#L2450-L2458)
- **边界：判定顺序短路**：frozen 优先于 moonglass／crystal；多标签目标只出首个命中的变体。[顺序](../../../raw/current/scripts/stategraphs/commonstates.lua#L2450-L2458)
- **边界：直落点固定普通版**：SGhound／SGmoonpig 不走 PlayMiningFX，直接生成 `mining_fx`——冰色变体只在走 helper 的路径出现。[对照](../../../raw/current/scripts/stategraphs/SGhound.lua#L771-L771)
- **边界：动画是否重复未核验**：条目仅 `PlayAnimation` 一次，未显式传循环参数，实际是否重复由动画数据决定。[播放](../../../raw/current/scripts/prefabs/fx.lua#L63-L65)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496。两注册配置、加载器协议、PlayMiningFX 选名与标签顺序、SGwilson gnaw 双分支、SGhound／SGmoonpig 直落点、其余调用面清点核验。前轮"无生成点"结论系检索截断所致（head 截断了拼接与非字面调用），本轮回填纠正。未覆盖：动画资源内容、十余处调用点逐条审阅、晚加入时序。
- 游戏运行：未运行；碎屑视觉未实测。

返回 [[reference/prefabs/fx-material-index|采集、制作、碎裂与生成烟]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-03）。
