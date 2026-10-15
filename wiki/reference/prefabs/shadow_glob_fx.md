---
title: "暗影地面痕特效（shadow_glob_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "麦克斯韦暗影秘典法术与暗影傀儡消散留下的半透明地面影：主模拟随机 3 变体一次播放自毁，海面可开启客户端涟漪；'shadowtrail' 标签兼作布痕去重占位。"
aliases: ["shadow_glob_fx", "暗影地面痕", "影子拖尾", "shadow trail fx"]
tags: [prefab, fx, server-client]
size_class: standard
dst_build: "steam-buildid:26335394"
sources: ["prefabs/shadow_glob_fx.lua", "prefabs/shadow_trap.lua", "stategraphs/SGshadow_trap.lua", "prefabs/shadow_pillar.lua", "stategraphs/SGshadowwaxwell.lua", "prefabs/waxwelljournal.lua", "prefabs/cane_ancient_fx.lua", "prefabskin.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 暗影地面痕特效（shadow_glob_fx）

## 快速判断

- 职责与适用：地面暗影痕（半透明 alpha 0.5）：主模拟随机播 `shad1~3` 之一、随机缩放 1~1.8 并 50% 水平翻转，播完自删；可选开启海面涟漪子 FX。布痕去重靠 `shadowtrail` 标签（0.7 距离查重）。[工厂](../../../raw/current/scripts/prefabs/shadow_glob_fx.lua#L69-L114)
- 优先使用：查"麦克斯韦暗影法术地面痕／暗影傀儡消散痕"选本页；四个生成点见关联机制导航。
- 重要副作用：本体无玩法组件，但标签被 shadow_trap／shadow_pillar 的 FindEntities 用作占位去重；古代手杖影痕同标签也参与占位。[标签](../../../raw/current/scripts/prefabs/shadow_glob_fx.lua#L76-L78)
- 修改边界：涟漪是客户端任务（非专服监听 netvar），专服只置位 netvar；本体在平台上会被 parent 到平台（随船沉没），离平台的海面才开涟漪。[专服分支](../../../raw/current/scripts/prefabs/shadow_glob_fx.lua#L87-L90)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `shadow_glob_fx` | fn：Transform＋AnimState＋Network＋FX/NOCLICK/shadowtrail 标签 → bank/build `cane_shadow_fx`、播 `shad1`、SetMultColour(1,1,1,.5) → net_bool `_ripple_enabled`(事件 ripple_enabled_dirty) → 非专服监听 dirty → SetPristine → 主模拟：随机重播 shad2/3、随机缩放与翻转、animover 自删、暴露 EnableRipples | 单一注册；变体／缩放／翻转只改主模拟实例。[注册](../../../raw/current/scripts/prefabs/shadow_glob_fx.lua#L116)、[工厂](../../../raw/current/scripts/prefabs/shadow_glob_fx.lua#L69-L114) |

无 NAMES 条目；关联官方译名：NAMES.WAXWELLJOURNAL "暗影秘典"（[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L351279-L351282)）、NAMES.SHADOWWAXWELL "暗影傀儡"（[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L348614-L348617)）。标题中文为已核验效果描述（shadow_glob_fx），非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：shadow_trap 布痕 | 暗影秘典影陷阱法术布下后 `EnableGroundFX` 每 FRAMES 沿 offsets 试点：0.7 内无 shadowtrail 实体才生成；海面非平台时 `EnableRipples(true)` | 影陷阱由 waxwelljournal TrapSpellFn 生成；SG 在 activating 态开启、离开关闭 | [TryFX](../../../raw/current/scripts/prefabs/shadow_trap.lua#L28-L73)、[周期任务](../../../raw/current/scripts/prefabs/shadow_trap.lua#L92)、[开关调用](../../../raw/current/scripts/stategraphs/SGshadow_trap.lua#L129)、[法术](../../../raw/current/scripts/prefabs/waxwelljournal.lua#L56-L68) |
| 生成者：SGshadow_trap trigger/dispell | 影陷阱触发／驱散时原地生成一枚：trigger 缩放 1.8、dispell 1.2，随机翻转；平台则 parent 到平台，否则 EnableRipples | 主模拟；源码注释明示"glob 在地面→随沉船销毁" | [trigger](../../../raw/current/scripts/stategraphs/SGshadow_trap.lua#L164-L177)、[dispell](../../../raw/current/scripts/stategraphs/SGshadow_trap.lua#L203-L212) |
| 生成者：shadow_pillar 布痕 | 影柱法术同 TryFX 模式布痕；shadow_pillar_spell 由 waxwelljournal PillarsSpellFn 生成 | 与影陷阱共用去重与海面逻辑 | [TryFX](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L598-L616)、[法术](../../../raw/current/scripts/prefabs/waxwelljournal.lua#L37-L54)、[实体生成](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L705) |
| 生成者：SGshadowwaxwell DoDespawnFX | 暗影傀儡（Maxwell 影人）消散时与 shadow_despawn 一起生成一枚（缩放 1.3）；在平台上 parent | 主模拟 | [DoDespawnFX](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L32-L48) |
| 涟漪子系统 | EnableRipples(true) 置 net_bool → 各非专服客户端起 DoRipple 任务：海面才生成 splash 涟漪实体（bank splash_weregoose_fx），帧与本体重播对齐，源码将动画帧号与 PERIOD（28*FRAMES，约 0.933）比较，不能解释成“前 28 帧”；仅比较成立时重排 | 客户端本地非联网实体；专服永不生成涟漪 | [OnRippleEnabled](../../../raw/current/scripts/prefabs/shadow_glob_fx.lua#L54-L63)、[DoRipple](../../../raw/current/scripts/prefabs/shadow_glob_fx.lua#L35-L52)、[CreateRipple](../../../raw/current/scripts/prefabs/shadow_glob_fx.lua#L13-L33) |
| 标签：shadowtrail | 本体自带；shadow_trap／shadow_pillar 以 0.7 半径 FindEntities 去重；cane_ancient_fx 影痕同标签也会占位 | 皮肤系统另有 TRAIL_FLAGS { "shadowtrail", "canetrail" } | [查重](../../../raw/current/scripts/prefabs/shadow_trap.lua#L27)、[同标签 FX](../../../raw/current/scripts/prefabs/cane_ancient_fx.lua#L56)、[皮肤标记](../../../raw/current/scripts/prefabskin.lua#L3106) |
| 资源 | anim/cane_shadow_fx.zip（本体）、anim/splash_weregoose_fx.zip＋anim/splash_water_drop.zip（涟漪） | 涟漪带海面混合参数 | [assets](../../../raw/current/scripts/prefabs/shadow_glob_fx.lua#L1-L6)、[混合](../../../raw/current/scripts/prefabs/shadow_glob_fx.lua#L28) |

## 典型接入

### 原地放一枚暗影痕（含海面处理）

- 前提与选择：主模拟 SpawnPrefab 后摆位；落在海面且无平台时调 `EnableRipples(true)` 让客户端补涟漪；在平台上则 parent 到平台实体，使痕迹随船移动／沉没。
- 生效与撤销：本体播完 animover 自删；涟漪由 netvar 驱动，任务随实体移除结束。
- 原版依据：SGshadow_trap trigger 态即完整用例（[生成](../../../raw/current/scripts/stategraphs/SGshadow_trap.lua#L164-L177)）。

```lua
local fx = SpawnPrefab("shadow_glob_fx")
local platform = TheWorld.Map:GetPlatformAtPoint(x, z)
if platform ~= nil then
    fx.entity:SetParent(platform.entity)
    fx.Transform:SetPosition(platform.entity:WorldToLocalSpace(x, 0, z))
else
    fx.Transform:SetPosition(x, 0, z)
    fx:EnableRipples(true)  -- 海面才生效，客户端生成涟漪
end
```

- 调整与验证：缩放用 `AnimState:SetScale`（负 x 即翻转）；需要去重时自查 `shadowtrail` 标签（0.7 半径），工厂不管重复。

## 常见陷阱与限制

- **边界：涟漪只在"海面且无平台"开启**：四个生成方都是平台优先 parent、非平台海面才 EnableRipples；陆地开启时 DoRipple 的 `IsOceanAtPoint` 检查会拦截生成，不报错。[检查](../../../raw/current/scripts/prefabs/shadow_glob_fx.lua#L38)
- **边界：变体与翻转只改主模拟**：shad2/3 重播、随机缩放翻转都在 SetPristine 之后执行，客户端工厂只初始化 shad1；后续由联网 AnimState 状态更新，实际表现未运行核验，不能断言客户端固定只播 shad1。[主模拟块](../../../raw/current/scripts/prefabs/shadow_glob_fx.lua#L98-L106)
- **边界：本体动画未显式传循环参数**：单次播放 animover 即删；涟漪的持续重复由 DoRipple 任务重排实现，与本体重播无关。[自删](../../../raw/current/scripts/prefabs/shadow_glob_fx.lua#L108)
- **误区：shadowtrail 只是视觉标记**：它是生成去重的占位依据——密集布点时 0.7 查重会吞掉候选位置；古代手杖影痕同标签，两类 FX 会互相抑制生成。[查重](../../../raw/current/scripts/prefabs/shadow_trap.lua#L35)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；117 行工厂全文（本体、涟漪子系统、netvar 链）、四个生成点上下文（shadow_trap TryFX／SG trigger+dispell／shadow_pillar TryFX／SGshadowwaxwell DoDespawnFX）、waxwelljournal 两法术、shadowtrail 消费与皮肤标记、译名已核验；动画帧数与实际时长、客户端变体表现、船沉没时 parent 清理的完整链路未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-world-index|天气、地形与生态痕迹]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
