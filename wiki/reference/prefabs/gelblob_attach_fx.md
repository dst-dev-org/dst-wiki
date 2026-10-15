---
title: "恶液接触附着黏液（gelblob_attach_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "恶液与暗影憎恶接触目标时挂身的黏液覆盖层：主层贴在目标身上、两根 Connector 连回宿主；对骑手／变形玩家与生物直接减速，普通玩家走 carefulwalker 谨慎步行，全部经共享表防叠加。"
aliases: ["gelblob_attach_fx", "恶液附着", "黏液附着特效", "gelblob 减速", "shrouden gel fx"]
tags: [prefab, fx, network-sync, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/gelblob_attach_fx.lua", "prefabs/gelblob.lua", "prefabs/shrouden.lua", "components/carefulwalker.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 恶液接触附着黏液（gelblob_attach_fx）

## 快速判断

- 职责与适用：恶液 gelblob（官方译名，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L342545-L342547)）与暗影憎恶 shrouden（[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L348734-L348737)）的"接触凝胶"表现＋减速玩法：目标身上挂主层 FX（加 `gelblobbed` 标签），两根 Connector 从目标平滑连回宿主。**是会改变移动速度的玩法 FX**。[SetupBlob](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L289-L308)
- 优先使用：查"贴身减速／接触覆盖层"选本页；宿主的接触检测循环见关联机制导航。
- 重要副作用：骑手／wereplayer 与带 locomotor 的生物直接 `SetExternalSpeedMultiplier(target,"gelblob",0.3)`（[CAREFUL_SPEED_MOD](../../../raw/current/scripts/tuning.lua#L3363)）；普通玩家每 0.1 秒推 `unevengrounddetected` 交给 [[reference/components/carefulwalker#关键接口]] 谨慎步行；同一目标多团恶液经模块级 `ALLTARGETS` 表防叠加。[防叠加](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L234-L246)
- 修改边界：`mainblob`（net_entity）与 `killed`（net_bool）双 netvar 同步；Connector 是双端各自创建的本地实体；宿主睡眠时直接 Remove 不走 splash。`persists=false` 不入存档。[构造](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L362-L402)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `gelblob_attach_fx` | fn：Transform＋AnimState＋DynamicShadow＋Network → 影子 2×1.5、FX 标签 → bank/build `gelblob` 播 `blob_attach_end_pre`、FinalOffset 2 → mainblob／killed netvar → SetPristine → 客户端监听 mainblobdirty／killeddirty＋OnRemoveEntity_Client 早退 → 主模拟 Push `blob_attach_end_loop`（**未显式传循环参数**）、`persists=false`、暴露 SetupBlob／KillFX、OnRemoveEntity_Server | 主层必须经 SetupBlob 挂到目标：SetParent(target)＋目标加 gelblobbed＋按目标类型登记减速；宿主是 shrouden 时主层与 Connector 都 LightOverride 1。[工厂](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L362-L402)、[SetupBlob](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L289-L308) |
| Connector（非注册，本地实体） | CreateConnectorBlob：Transform＋AnimState＋DynamicShadow、不联网、`SetCanSleep(ismastersim)`、播 `blob_attach_middle_pre`＋Push `blob_attach_middle_loop`（**未显式传循环参数**）、暴露 LerpTo／KillFX | 由 OnMainBlobDirty 创建两根，父级设为宿主；宿主旋转下 LerpTo 做旋转变换跟随。[工厂](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L78-L103)、[创建](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L179-L199) |

本页两形态均无 NAMES 中文条目；"恶液／暗影憎恶"为宿主官方译名，"附着黏液"为已核验效果描述。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 宿主：gelblob（恶液） | OnUpdateProximity 接触圈（半径＝目标物理半径＋接触值）内对带 locomotor 的目标 Spawn 并 SetupBlob；移出圈／死亡／despawn 逐个 KillFX，睡眠时直接 Remove | 大体型恶液另有 suspend 吞人逻辑与本 FX 并行；onremove 回调清理 `_targets` 记账 | [接触生成](../../../raw/current/scripts/prefabs/gelblob.lua#L146-L163)、[移出圈清理](../../../raw/current/scripts/prefabs/gelblob.lua#L171-L175)、[死亡](../../../raw/current/scripts/prefabs/gelblob.lua#L310-L323)、[睡眠直接 Remove](../../../raw/current/scripts/prefabs/gelblob.lua#L344-L353) |
| 宿主：shrouden（暗影憎恶） | OnUpdateGel 同构接触圈（3.2／3.35），Spawn＋SetupBlob；蜜渍路径用 `gelblobbed` 标签过滤已附着目标 | 双端同 gelblob；LightOverride 1（黑凝胶无光渲染） | [接触生成](../../../raw/current/scripts/prefabs/shrouden.lua#L183-L208)、[gelblobbed 过滤](../../../raw/current/scripts/prefabs/shrouden.lua#L148-L151) |
| 减速：locomotor 外部乘数 | RegisterTargetLocomotorDebuff 以 key `"gelblob"` 调 [[reference/components/locomotor#关键接口]] 的 SetExternalSpeedMultiplier(0.3)；注销时移除 | 目标须有 locomotor；玩家仅在骑乘或 wereplayer 形态走此路径（mounted/dismounted/startwereplayer/stopwereplayer 事件刷新） | [登记／注销](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L237-L260)、[玩家分流](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L279-L287)、[乘数](../../../raw/current/scripts/tuning.lua#L3363) |
| 减速：carefulwalker 谨慎步行 | 普通玩家路径每 0.1 秒推 `unevengrounddetected`(radius 0.5, period 0.1)，[[reference/components/carefulwalker#关键接口]] TrackTarget 后谨慎步行减速 | 仅玩家且未骑乘、非 wereplayer；FX 删除时取消任务 | [推送](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L262-L270)、[消费](../../../raw/current/scripts/components/carefulwalker.lua#L12-L13) |
| [[reference/components/highlight#关键接口]]：highlightchildren | 宿主有 highlightchildren 表时把主层与两根 Connector 插入，跟随宿主检视高亮；宿主丢失／FX 删除时移除登记 | 依赖宿主暴露该表（gelblob 初始化空表） | [插入](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L201-L208)、[宿主表](../../../raw/current/scripts/prefabs/gelblob.lua#L538) |
| 生命周期：KillFX 双层收尾 | 主层 killed 置位→摘 gelblobbed→脱离父级→播 `splash` 自删；locomotor 减速由 OnRemoveEntity_Server 在实体移除时注销；两根 Connector 走 KillConnector：脱离宿主、播 `blob_attach_middle_pst`、updatelooper 按 k² 缓动把位置缩向宿主、缩放回 1 | 连接段动画时长由 `_animlen` 实测；宿主无效或已睡眠直接 Remove 跳过动画 | [主层收尾](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L310-L336)、[Connector 收尾](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L43-L69)、[killed dirty](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L220-L232) |

## 典型接入

复用"接触附着＋可撤销减速"的完整链路：Spawn 后必须调 `fx:SetupBlob(宿主, 目标)`；结束时必须调 `fx:KillFX()`（或保证宿主走 OnRemoveEntity），不能直接 Remove 目标身上的 FX——直接 Remove 会走 OnRemoveEntity 分支清标签与减速，但没有 splash 动画，且 Connector 不播收尾缩回。[清理分界](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L338-L360)

```lua
local fx = SpawnPrefab("gelblob_attach_fx")
fx:SetupBlob(myhost, mytarget)   -- 挂目标、加标签、登记减速
-- 解除时：
fx:KillFX()                      -- splash + Connector 回缩；locomotor 减速到实体 Remove 才注销
```

- 调整与验证：减速乘数是 tuning 键（[CAREFUL_SPEED_MOD](../../../raw/current/scripts/tuning.lua#L3363)）；接触圈半径与缩放曲线在宿主 OnUpdate 与本文件 [L129-L165](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L129-L165)。

## 常见陷阱与限制

- **误区：挂上就全程减速 0.3**：普通玩家走 carefulwalker 谨慎步行（仅靠太近时生效），骑手／wereplayer／生物才吃 locomotor 0.3 乘数；两路径随 mounted 等事件互斥切换。[分流](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L279-L287)
- **误区：多团恶液减速会叠乘**：locomotor 路径以模块级 `ALLTARGETS` 记账，同目标多个 FX 只登记一次乘数，最后一个注销时才移除；该表不跨存档、不跨会话。[防叠加](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L234-L260)
- **边界：宿主睡眠直接消失**：gelblob OnEntitySleep 对目标 FX 直接 `Remove`（无 splash）；KillFX 内部也先查 IsAsleep 立即删。表现是否突兀取决于宿主睡眠时机。[睡眠路径](../../../raw/current/scripts/prefabs/gelblob.lua#L344-L353)
- **边界：mainblob 失效的客户端兜底**：客户端 `mainblobdirty` 收到 nil（宿主睡眠或传送）时走 OnMainBlobLost 清 Connector 与 updatelooper，后续非 nil 的 dirty 仍可重建 Connector。[兜底](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L121-L127)
- **边界：`blob_attach_end_loop`／`blob_attach_middle_loop` 未显式传循环参数**：实际是否重复未核验；层持续存在由宿主接触循环维持，不靠动画循环。[主层](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L393)、[Connector](../../../raw/current/scripts/prefabs/gelblob_attach_fx.lua#L93-L94)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；405 行工厂全文（主层／Connector 双工厂、接触几何、双路径减速、防叠加表、双层收尾）、gelblob 与 shrouden 宿主循环、carefulwalker 消费端、tuning 乘数、译名检索已核验；恶液吞人（suspend）完整流程、宿主 Brain、`unevengrounddetected` 在玩家 SG 的具体表现未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
