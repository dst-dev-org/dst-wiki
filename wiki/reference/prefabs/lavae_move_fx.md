---
title: "岩浆虫与哈奇移动拖尾（lavae_move_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "岩浆虫／宠物岩浆虫与哈奇移动时留下的网络壳拖尾：主模拟 SetVariation 选 7 种动画与 8 档缩放，非专服延迟一帧在宿主位置生成本地一次性动画并自毁；同文件另一注册 hutch_move_fx。"
aliases: ["lavae_move_fx", "hutch_move_fx", "岩浆虫拖尾", "哈奇黏液拖尾", "move trail fx"]
tags: [prefab, fx, server-client]
size_class: standard
dst_build: "steam-buildid:26335394"
sources: ["prefabs/lavae_move_fx.lua", "stategraphs/SGlavae.lua", "stategraphs/SGchester.lua", "prefabs/lavae.lua", "prefabs/lavae_pet.lua", "prefabs/hutch.lua", "prefabs/chester.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 岩浆虫与哈奇移动拖尾（lavae_move_fx）

## 快速判断

- 职责与适用：移动拖尾网络壳：主模拟只经 `SetVariation` 写 netvar（变体 1~7、缩放量化 0~7 档），非专服监听 dirty 延迟一帧在宿主位置生成非联网本地 FX（bank `lava_trail_fx`、build `lavae_move_fx`、anim `trail1~7`），播完自删。同文件注册 `hutch_move_fx`（青色乘色、不同缩放区间）。[注册](../../../raw/current/scripts/prefabs/lavae_move_fx.lua#L95-L96)、[PlayFX](../../../raw/current/scripts/prefabs/lavae_move_fx.lua#L6-L31)
- 优先使用：要复用"网络壳＋本地一次性 FX"拖尾模式选本页；生成节奏与去重由宿主 SG 控制，不在本工厂。
- 重要副作用：无玩法组件；网络壳 1 秒兜底自移除，本地 FX animover 自移除，全部 persists=false。[壳自删](../../../raw/current/scripts/prefabs/lavae_move_fx.lua#L74-L75)
- 修改边界：专服只建壳不生成本地 FX；每壳只出一个 FX（`_complete` 守卫）。[专服分支](../../../raw/current/scripts/prefabs/lavae_move_fx.lua#L60-L64)、[守卫](../../../raw/current/scripts/prefabs/lavae_move_fx.lua#L33-L41)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `lavae_move_fx` | common_fn → lavae_fn：Transform＋Network＋FX 标签 → net_tinybyte `_rand`(事件 randdirty)／`_scale` → 非专服监听 randdirty → SetPristine → 主模拟：暴露 SetVariation，persists=false，1 秒自移除 | `_min_scale .5`／`_max_scale 1.3`；无乘色。[common_fn](../../../raw/current/scripts/prefabs/lavae_move_fx.lua#L49-L78)、[参数](../../../raw/current/scripts/prefabs/lavae_move_fx.lua#L80-L85) |
| `hutch_move_fx` | 同上 → hutch_fn | `_min_scale .3`／`_max_scale 1.5`；`_colour {.6,1,1,1}` 青色乘色（PlayFX 时 SetMultColour）。[参数](../../../raw/current/scripts/prefabs/lavae_move_fx.lua#L87-L93)、[乘色](../../../raw/current/scripts/prefabs/lavae_move_fx.lua#L26-L28) |

官方译名：NAMES.LAVAE "岩浆虫"、NAMES.LAVAE_PET "超级可爱岩浆虫"、NAMES.HUTCH "哈奇"、NAMES.CHESTER "切斯特"（行号见验证记录）。标题"拖尾"为已核验结构描述，动画视觉内容（火焰／黏液质感）未解包资源核验。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| SG：SGlavae SpawnMoveFx | walk 态 DoPeriodicTask 每 3 帧一次、40% 概率生成 `lavae_move_fx`；7 变体带"最近 4 个不重复"记忆；缩放＝宿主缩放×(1.3−rand²) | 岩浆虫与宠物岩浆虫共用 SGlavae（宠物是否实际出痕未运行验证） | [SpawnMoveFx](../../../raw/current/scripts/stategraphs/SGlavae.lua#L44-L69)、[walk 态](../../../raw/current/scripts/stategraphs/SGlavae.lua#L232-L257)、[接线](../../../raw/current/scripts/prefabs/lavae.lua#L145) |
| SG：SGchester walktimeline | AddWalkStates 步进时间线多点生成 `hutch_move_fx`（起跳／落点用不同 scale 参数） | 仅 `inst.leave_slime` 时——哈奇置 true，切斯特未置，切斯特永不出痕 | [时间线](../../../raw/current/scripts/stategraphs/SGchester.lua#L489-L578)、[判定](../../../raw/current/scripts/stategraphs/SGchester.lua#L499-L500)、[哈奇置位](../../../raw/current/scripts/prefabs/hutch.lua#L451) |
| API／数据：SetVariation(rand, scale) | 主模拟写 `_rand` 与量化 `_scale`（0~7 共 8 档）；客户端换算实际缩放 `_scale/7×(max−min)+min` | 每壳一次；rand≤0 时客户端直接不出 FX | [SetVariation](../../../raw/current/scripts/prefabs/lavae_move_fx.lua#L43-L47)、[换算](../../../raw/current/scripts/prefabs/lavae_move_fx.lua#L39)、[守卫](../../../raw/current/scripts/prefabs/lavae_move_fx.lua#L33-L41) |
| 声明：依赖清单 | lavae.lua／lavae_pet.lua 依赖 `lavae_move_fx`，hutch.lua 依赖 `hutch_move_fx` | 加载期注册，不是生成证据 | [lavae](../../../raw/current/scripts/prefabs/lavae.lua#L11)、[lavae_pet](../../../raw/current/scripts/prefabs/lavae_pet.lua#L14)、[hutch](../../../raw/current/scripts/prefabs/hutch.lua#L26) |
| 宿主接线 | lavae 与 lavae_pet 都 `SetStateGraph("SGlavae")`；hutch 与 chester 都 `SetStateGraph("SGchester")` | SGchester 的出痕分支只对哈奇生效 | [hutch](../../../raw/current/scripts/prefabs/hutch.lua#L453)、[chester](../../../raw/current/scripts/prefabs/chester.lua#L696)、[lavae_pet](../../../raw/current/scripts/prefabs/lavae_pet.lua#L141) |

## 典型接入

### 复用壳模式做移动拖尾

- 前提与选择：主模拟 SpawnPrefab 后调 `SetVariation(随机变体, 目标缩放)` 再摆位；非专服自动在下一帧出 FX，宿主 SG 负责节奏与变体去重。
- 生效与撤销：壳 1 秒自删兜底；本地 FX 播完（animover）自删；变体去重表维护在宿主 `sg.mem.recentfx`。
- 原版依据：SGlavae walk 态即完整用例（[周期生成](../../../raw/current/scripts/stategraphs/SGlavae.lua#L246)）。

```lua
local fx = SpawnPrefab("lavae_move_fx")
fx:SetVariation(math.random(7), 1.0)
fx.Transform:SetPosition(inst.Transform:GetWorldPosition())
```

- 调整与验证：缩放参数先量化到 8 档，过细的缩放差异会被取整；变体传 0 客户端不出 FX（[守卫](../../../raw/current/scripts/prefabs/lavae_move_fx.lua#L33-L41)）。

## 常见陷阱与限制

- **误区：切斯特也会出痕**：SGchester 的全部生成点都在 `leave_slime` 分支内，只有哈奇置该标记；切斯特共用同一 SG 却永不出痕。[判定](../../../raw/current/scripts/stategraphs/SGchester.lua#L499-L500)
- **边界：动画未显式传循环参数**：本地 FX 单次播放、animover 即 Remove，不存在重播机制——每个壳只出一次 FX。[PlayFX](../../../raw/current/scripts/prefabs/lavae_move_fx.lua#L6-L31)
- **边界：客户端与主模拟值有半档误差**：缩放经 0~7 量化传递，客户端换算值与主模拟传入值不完全相等；乘色／位置随 netvar 与代理坐标走，各端一致性未逐端核验。[量化](../../../raw/current/scripts/prefabs/lavae_move_fx.lua#L46)、[换算](../../../raw/current/scripts/prefabs/lavae_move_fx.lua#L39)
- **边界：专服静默**：`IsDedicated` 不注册 randdirty 监听，专服只有空壳等 1 秒自删。[分支](../../../raw/current/scripts/prefabs/lavae_move_fx.lua#L60-L64)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；97 行工厂全文（壳、本地 FX、量化与守卫）、SGlavae SpawnMoveFx 与 walk 态、SGchester walktimeline 与 leave_slime 链、lavae／lavae_pet／hutch／chester 接线、译名 po 行（[LAVAE L343999-344002](../../../raw/current/scripts/languages/chinese_s.po#L343999-L344002)、[LAVAE_PET L344019-344022](../../../raw/current/scripts/languages/chinese_s.po#L344019-L344022)、[HUTCH L343424-343427](../../../raw/current/scripts/languages/chinese_s.po#L343424-L343427)、[CHESTER L340944-340947](../../../raw/current/scripts/languages/chinese_s.po#L340944-L340947)）已核验；anim 资源内容、宠物岩浆虫实际行走表现、晚加入时序未覆盖。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
