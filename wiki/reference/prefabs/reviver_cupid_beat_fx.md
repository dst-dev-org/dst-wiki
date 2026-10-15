---
title: "告密的心丘比特皮肤心跳特效（reviver_cupid_beat_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "reviver 告密的心 cupid 系皮肤的两个伴随特效：beat 心形粒子爆发（与宿主心跳重播同步、仅落地时有）与 glow 发光覆盖动画（挂 highlightchildren 随高亮），同文件两注册。"
aliases: ["reviver_cupid_beat_fx", "reviver_cupid_glow_fx", "告密的心特效", "丘比特皮肤特效", "天使之心", "Cherub's Heart", "心跳粒子"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/reviver_cupid_beat_fx.lua", "prefabs/reviver.lua", "prefabs/skinprefabs.lua", "prefabskin.lua", "components/highlight.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 告密的心丘比特皮肤心跳特效（reviver_cupid_beat_fx）

## 快速判断

- 职责与适用：reviver 物品（告密的心，官方译名，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L348004-L348007)）"cupid" 系皮肤（如 reviver_cupid"天使之心"，[SKIN_NAMES](../../../raw/current/scripts/languages/chinese_s.po#L406099-L406102)）的两个联网伴随特效：`reviver_cupid_beat_fx` 是心形粒子爆发（非专服本地 VFXEffect，爆发与宿主心跳动画重播同步），`reviver_cupid_glow_fx` 是 bloodpump 发光覆盖动画（挂进宿主 highlightchildren）。二者无玩法作用。[注册](../../../raw/current/scripts/prefabs/reviver_cupid_beat_fx.lua#L159-L160)
- 优先使用：查"告密的心皮肤特效／心跳粒子"选本页；皮肤接线与心跳循环见下文导航表。
- 重要副作用：beat 粒子只在物品**落地期间**存在（拾入背包即删）；glow 覆盖动画常驻并参与高亮传播。[拾起删除](../../../raw/current/scripts/prefabs/reviver.lua#L37-L46)
- 修改边界：beat 粒子发射器专服不建（InitParticles 早退）；glow 的皮肤符号替换由 reviver_init_fn 对实例设置，不在工厂内。[专服早退](../../../raw/current/scripts/prefabs/reviver_cupid_beat_fx.lua#L53-L57)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `reviver_cupid_beat_fx` | fn：Transform＋Network（本体无 AnimState）→ FX 标签 → InitParticles 配发射器 → SetPristine → 客户端早退 → 主模拟 `persists=false` | 单发射器、上限 4 粒、寿命 1 秒、旋转粒子；贴图 fx/heartglow.tex＋vfx_particle_add 加法混合＋bloom；色与缩放 envelope 写死后置 nil。[工厂](../../../raw/current/scripts/prefabs/reviver_cupid_beat_fx.lua#L95-L114)、[发射器](../../../raw/current/scripts/prefabs/reviver_cupid_beat_fx.lua#L51-L71) |
| `reviver_cupid_glow_fx` | glowfn：Transform＋AnimState＋Network → bank/build `bloodpump`、OverrideSymbol bloodpump01→bloodpumpglow、Hide Shadow、播 `idle`、LightOverride 0.3 → SetPristine → 客户端挂 OnEntityReplicated → 主模拟 `persists=false` | 客户端复制完成时若父级是 reviver，把自身登记进父级 highlightchildren。[工厂](../../../raw/current/scripts/prefabs/reviver_cupid_beat_fx.lua#L127-L155)、[复制回调](../../../raw/current/scripts/prefabs/reviver_cupid_beat_fx.lua#L118-L125) |

只有 glow 注册调用 PlayAnimation，未显式传循环参数；beat 没有 AnimState。实际动画是否重复未核验。`reviver_lifecrystal`（"生命水晶"，[译名](../../../raw/current/scripts/languages/chinese_s.po#L406140-L406142)）皮肤复用 glow 注册而 beat 位留空。[皮肤表](../../../raw/current/scripts/prefabs/skinprefabs.lua#L10654-L10666)

标题中文为官方物品名＋已核验效果描述（"丘比特皮肤心跳特效"部分非官方译名，cupid 系 8 个皮肤名见 [SKIN_NAMES 组](../../../raw/current/scripts/languages/chinese_s.po#L406099-L406139)）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 皮肤声明：skinprefabs | cupid 系 8 个皮肤（_cupid／_2／_2alt／_3／_3alt／_4／_4alt／_alt）都在 prefabs 与 fx_prefab 列两特效；lifecrystal 只列 glow | fx_prefab[1] 是 beat、[2] 是 glow | [reviver_cupid](../../../raw/current/scripts/prefabs/skinprefabs.lua#L10554-L10564)、[cupid 系全表](../../../raw/current/scripts/prefabs/skinprefabs.lua#L10554-L10652) |
| 皮肤接线：reviver_init_fn | 主模拟专供：`reviver_beat_fx = skin_fx[1]`（空串则 nil）；glow 立即生成并 SetParent、OverrideItemSkinSymbol 换发光符号、`highlightchildren = { glow }`、PlayBeatAnimation 换成同时重播 glow 的版本 | 换肤清理由 reviver_clear_fn 移除并还原 | [init](../../../raw/current/scripts/prefabskin.lua#L3062-L3086)、[playbeat 替换](../../../raw/current/scripts/prefabskin.lua#L3057-L3060)、[clear](../../../raw/current/scripts/prefabskin.lua#L3087-L3101) |
| 心跳循环：reviver.lua | 每 0.75-1.5 秒重播宿主 idle＋播 ghost/bloodpump 音；落地（ondropped→startbeat）时生成 beat fx：SetParent＋AddFollower＋FollowSymbol(bloodpump01, -5, -30, 0)；拾起（onpickup）删 beat fx | beat fx 只在地面；glow 从换肤起常驻 | [beat 循环](../../../raw/current/scripts/prefabs/reviver.lua#L10-L14)、[startbeat](../../../raw/current/scripts/prefabs/reviver.lua#L16-L28)、[落地／拾起](../../../raw/current/scripts/prefabs/reviver.lua#L30-L46) |
| 粒子与心跳同步 | EmitterManager 回调检测父级 AnimState 当前时间 <0.1 秒且处于等待 → 爆发 1 粒（含角度随机） | 父级是 reviver 本体，idle 重播即心跳拍点 | [爆发同步](../../../raw/current/scripts/prefabs/reviver_cupid_beat_fx.lua#L73-L90) |
| 高亮传播 | glow 进宿主 highlightchildren（主模拟 init 设置；客户端由 OnGlowFXReplicated 补登记）；[[reference/components/highlight#关键接口]] 对 children 传播高亮色 | highlightchildren 是宿主普通 Lua 字段，不是组件 | [登记](../../../raw/current/scripts/prefabs/reviver_cupid_beat_fx.lua#L118-L125)、[消费](../../../raw/current/scripts/components/highlight.lua#L88-L89) |
| 动画资源 | glow 用 bloodpump bank（与 reviver 本体同包），符号 bloodpumpglow；beat 粒子用 fx/heartglow.tex | 资源包内容未解包 | [glow 设置](../../../raw/current/scripts/prefabs/reviver_cupid_beat_fx.lua#L134-L140)、[贴图声明](../../../raw/current/scripts/prefabs/reviver_cupid_beat_fx.lua#L8-L12) |

## 典型接入

### 复用"与宿主动画同步的粒子拍点"

- 前提与选择：把本 fx SetParent 到一个会周期重播短动画的宿主，EmitterManager 回调即可跟随拍点爆发；或按皮肤协议走 reviver_init_fn 的 fx_prefab 表。
- 生效与撤销：粒子发射器随实体存在；宿主 Remove 时子实体随父删除。

```lua
local fx = SpawnPrefab("reviver_cupid_beat_fx")
fx.entity:SetParent(host.entity) -- host 需有 AnimState 且周期重播 idle
```

- 调整与验证：爆发判定阈值 0.1 秒写死；envelope 名 `cupid_beat1_colourenvelope`／`cupid_beat1_scaleenvelope` 全局注册，复用配置时应整体改名防冲突。[envelope](../../../raw/current/scripts/prefabs/reviver_cupid_beat_fx.lua#L5-L6)

## 常见陷阱与限制

- **误区：beat 粒子常驻飘散**：每次心跳爆发 1 粒、寿命 1 秒、并存上限 4；且物品入背包即删（宿主 onpickup 移除 beat fx）。[发射上限](../../../raw/current/scripts/prefabs/reviver_cupid_beat_fx.lua#L62)、[拾起](../../../raw/current/scripts/prefabs/reviver.lua#L37-L46)
- **误区：glow 的发光符号是工厂行为**：OverrideItemSkinSymbol 由 reviver_init_fn 对实例设置，工厂只 Override 普通符号；脱离皮肤协议单独 SpawnPrefab 的 glow 不带皮肤发光符号。[init 覆盖](../../../raw/current/scripts/prefabskin.lua#L3074-L3078)
- **边界：beat 的 FollowSymbol 由宿主添加**：beat fx 工厂没有 Follower，复用时必须自己 AddFollower＋FollowSymbol 才会贴符号。[宿主调用](../../../raw/current/scripts/prefabs/reviver.lua#L23-L25)
- **边界：客户端 highlightchildren 靠复制回调补**：主模拟的 highlightchildren 不联网，glow 在客户端复制完成后自行登记；父级不是 reviver 时跳过。[回调](../../../raw/current/scripts/prefabs/reviver_cupid_beat_fx.lua#L118-L125)
- **边界：lifecrystal 无 beat 粒子**：fx_prefab[1] 为空串时 reviver_beat_fx=nil，startbeat 跳过生成。[空串分支](../../../raw/current/scripts/prefabskin.lua#L3070-L3072)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；160 行工厂全文与两注册、reviver.lua 心跳循环全路径、skinprefabs cupid 系皮肤表、reviver_init_fn／clear_fn 接线、highlight 消费点、译名检索已核验；贴图内容、粒子实际视觉、换肤与晚加入时序未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
