---
title: "化石碎裂与化石化特效（fossilized_break_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "化石主题四注册特效族：fossilized_break_fx 碎裂一次后侵蚀消散，fossilizing_fx／_1／_2 代理＋本地动画的『石化进行中』尘雾（与蚁狮塌陷共用 sinkhole_spawn_fx 资源），1 秒后自删。"
aliases: ["fossilized_break_fx", "fossilizing_fx", "fossilizing_fx_1", "fossilizing_fx_2", "化石碎裂特效", "化石碎片特效", "fossilizing fx"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/fossilized_fx.lua", "prefabs/lavaarena_beetletaur.lua", "prefabs/lavaarena_rhinodrill.lua", "prefabs/lavaarena_boarrior.lua", "prefabs/eyeofterror_sinkhole.lua", "languages/chinese_s.po"]
size_class: simple
reliability: verified
validation: source-reviewed
status: active
---

# 化石碎裂与化石化特效（fossilized_break_fx）

## 快速判断

- 职责与适用：化石主题的四个 FX 注册：`fossilized_break_fx` 是联网碎裂动画（播一次即侵蚀消散）；`fossilizing_fx`／`fossilizing_fx_1`／`fossilizing_fx_2` 是"代理＋本地动画"尘雾（通用入口首次选择 idle1/idle2，与蚁狮塌陷尘雾共用 `sinkhole_spawn_fx` 资源）。全部无玩法组件。[四注册](../../../raw/current/scripts/prefabs/fossilized_fx.lua#L113-L116)
- 优先使用：查"石化／化石碎裂尘雾"选本页；注意本快照 scripts 内**没有**对这四个注册名的生成调用，复用需自行 Spawn（见陷阱）。
- 重要副作用：纯表现；`fossilizing_fx` 主模拟只存活 1 秒，视觉本体是非专服 0 帧延迟创建的本地实体（专服无视觉）。[代理工厂](../../../raw/current/scripts/prefabs/fossilized_fx.lua#L75-L109)
- 修改边界：代理实体仅 Transform＋Network；本地尘雾 `SetCanSleep(false)`、`persists=false`，跟随代理父级生成。[本地动画](../../../raw/current/scripts/prefabs/fossilized_fx.lua#L45-L73)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `fossilized_break_fx` | breakfn：Transform＋AnimState＋Network → FX 标签 → 仅 SetBuild `fossilized`（**未调 SetBank**）播 `fossilized_break_fx` → SetPristine → 客户端早退 → 主模拟 `persists=false`＋animover ErodeAway | 单次碎裂，无声音组件；ErodeAway 走全局侵蚀消散。[工厂](../../../raw/current/scripts/prefabs/fossilized_fx.lua#L19-L41) |
| `fossilizing_fx` | MakeFossilizingFX(name,nil)：Transform＋Network → FX 标签 → 非专服 0 帧后 PlayFossilizingAnim：anim 为 nil 时随机 1/2、`SetPrefabName("fossilizing_fx_"..n)`、播 `idle{n}`；SetPristine → 客户端早退 → 主模拟 1 秒后 Remove | 通用入口首次在 anim=nil 时随机二选一，随后会改写工厂闭包变量 anim；不能承诺每次生成重新随机；依赖声明 fossilizing_fx_1/_2。[工厂](../../../raw/current/scripts/prefabs/fossilized_fx.lua#L75-L109)、[随机](../../../raw/current/scripts/prefabs/fossilized_fx.lua#L88-L93) |
| `fossilizing_fx_1`／`fossilizing_fx_2` | 同工厂，anim 固定 `idle1`／`idle2` | 与通用入口同资源同生命周期；无随机。[注册](../../../raw/current/scripts/prefabs/fossilized_fx.lua#L113-L116) |

本地尘雾：bank/build `sinkhole_spawn_fx`、MultColour(0.75,0.5,0.5,1) 红调、FinalOffset 2、缩放按代理父级物理半径（缺省 0.5）+1、animover 自删。[本地实体](../../../raw/current/scripts/prefabs/fossilized_fx.lua#L45-L73)

本族均无 NAMES 中文条目；"化石碎片"（[NAMES.FOSSIL_PIECE](../../../raw/current/scripts/languages/chinese_s.po#L342384-L342387)）是 fossil_piece 物品的官方译名，仅作主题参照——本快照未见本族 FX 与该物品的直接调用关系。标题中文为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 依赖声明：熔炉（The Forge）系列 | lavaarena 六个战斗单位把 `fossilizing_fx`、`fossilized_break_fx` 列入 prefabs 依赖表 | 仅声明资源加载；本快照 scripts 内无 Spawn 调用 | [boarrior 依赖](../../../raw/current/scripts/prefabs/lavaarena_boarrior.lua#L9-L10) |
| 同名动画：熔炉专属变体 | beetletaur／rhinodrill 各自注册 `beetletaur_fossilized_break_fx_*` 等**独立 Prefab**，复用 `fossilized_break_fx` 动画名但非本文件注册 | 熔炉事件的石化态专属碎裂，与本页四注册并存不冲突 | [beetletaur 注册](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L307-L310)、[rhinodrill 注册](../../../raw/current/scripts/prefabs/lavaarena_rhinodrill.lua#L227) |
| 共用资源：sinkhole_spawn_fx bank | 本族 fossilizing 尘雾与蚁狮塌陷 `sinkhole_spawn_fx_1/2/3`、恐怖之眼下潜尘雾同 bank/build | 资源共用不代表行为关联 | [蚁狮塌陷](../../../raw/current/scripts/prefabs/eyeofterror_sinkhole.lua#L31-L35) |

## 典型接入

仿代理工厂复用"双端安全的短命尘雾"：主模拟 Spawn `fossilizing_fx` 后立刻设位置／父级，非专服 0 帧后自动长出本地动画，1 秒后主模拟删除；本地动画 animover 自删兜底。直接 Spawn `fossilizing_fx_1`／`_2` 可固定动画变体。[工厂](../../../raw/current/scripts/prefabs/fossilized_fx.lua#L75-L109)

```lua
local fx = SpawnPrefab("fossilizing_fx")
fx.entity:SetParent(host.entity)   -- 可选：跟随宿主
fx.Transform:SetPosition(x, y, z)
```

- 调整与验证：尘雾颜色／缩放在 PlayFossilizingAnim（[L63-L70](../../../raw/current/scripts/prefabs/fossilized_fx.lua#L63-L70)）；碎裂动画播完后走 ErodeAway，改"立即消失"需替换 animover 回调。

## 常见陷阱与限制

- **边界：本快照无生成调用**：全 scripts 树检索 `SpawnPrefab` 未命中这四个注册名；熔炉实际生成逻辑在事件外部数据（`event_server_data("lavaarena",...)`，不在 scripts 快照内），复用前自行确认适用场景。[依赖声明即证据上限](../../../raw/current/scripts/prefabs/lavaarena_boarrior.lua#L9-L10)
- **边界：fossilized_break_fx 未调 SetBank**：仅 SetBuild 后直接 PlayAnimation，能否命中 bank 取决于动画资源本身的 bank 组织；复制该写法前先在游戏内确认渲染正常。[工厂](../../../raw/current/scripts/prefabs/fossilized_fx.lua#L28-L29)
- **边界：视觉本体专服不存在**：fossilizing 族的本地尘雾在 `TheNet:IsDedicated()` 为真时跳过，专服只有 1 秒寿命的空代理实体。[分支](../../../raw/current/scripts/prefabs/fossilized_fx.lua#L84-L94)
- **误区：fossilizing_fx 是 fossilizing_fx_1 的父实体**：三者是并列注册；通用入口在本地动画创建时才随机挑 `idle1`/`idle2`，与两个固定版之间无生成关系；anim 为闭包变量，首次选定后后续调用不再满足 nil 门禁。[随机逻辑](../../../raw/current/scripts/prefabs/fossilized_fx.lua#L88-L92)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；117 行工厂全文（四注册、代理＋本地双层、生命周期）、熔炉依赖表与专属变体注册、sinkhole 资源共用、译名检索已核验；全 scripts 树 Spawn 检索为空、熔炉外部事件数据不可见、动画实际帧数未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
