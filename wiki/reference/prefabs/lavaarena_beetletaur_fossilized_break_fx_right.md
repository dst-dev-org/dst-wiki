---
title: "熔炉独眼巨猪与犀牛兄弟化石碎裂特效（beetletaur_fossilized_break_fx_right 等）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "熔炉（The Forge）独眼巨猪（beetletaur）与犀牛兄弟（rhinodrill／rhinodrill2）专属的化石碎裂表现：两文件各内联 MakeFossilizedBreakFX，7 个联网 FX 注册按 side 隐藏单侧岩块、非中断巨猪版覆盖宿主 rock 符号，animover 后走 ErodeAway 侵蚀消散；实际生成点在缺失的 fossilizable 组件与熔炉事件包。"
aliases: ["beetletaur_fossilized_break_fx_right", "beetletaur_fossilized_break_fx_left", "beetletaur_fossilized_break_fx_left_alt", "beetletaur_fossilized_break_fx_alt", "rhinodrill_fossilized_break_fx_right", "rhinodrill_fossilized_break_fx_left", "rhinodrill_fossilized_break_fx", "lavaarena_beetletaur_fossilized_break_fx_right", "熔炉化石碎裂特效", "熔炉化石化碎裂特效"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/lavaarena_beetletaur.lua", "prefabs/lavaarena_rhinodrill.lua", "prefabs/fossilized_fx.lua", "stategraphs/commonstates.lua", "simutil.lua", "entityscript.lua", "languages/chinese_s.po"]
size_class: standard
reliability: verified
validation: source-reviewed
status: active
---

# 熔炉独眼巨猪与犀牛兄弟化石碎裂特效（beetletaur_fossilized_break_fx_right 等）

## 快速判断

- 职责与适用：熔炉（The Forge）两只生物——独眼巨猪 `beetletaur` 与犀牛兄弟 `rhinodrill`／`rhinodrill2`——的化石化碎裂**表现层**：两文件各自内联 `MakeFossilizedBreakFX`，共 7 个联网 FX 注册（巨猪 4 个、犀牛 3 个），播放 `fossilized` build 的 `fossilized_break_fx` 动画；侧向变体隐藏单侧岩块，只有巨猪有 `interrupted`（`_alt`）版本。[巨猪工厂](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L265-L303)、[巨猪注册](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L306-L310)、[犀牛工厂](../../../raw/current/scripts/prefabs/lavaarena_rhinodrill.lua#L194-L228)
- 优先使用：普通生物与熔炉其他单位（boaron／boarrior 等）的通用碎裂与石化尘雾用 [[reference/prefabs/fossilized_break_fx|化石碎裂与化石化特效（fossilized_break_fx）]]；本页只覆盖熔炉这两只生物的专属 7 注册，按生物＋侧向（right／left／空）＋是否中断选名，完整名单见下表。
- 重要副作用：联网实体，仅 `FX` 标签，无玩法组件、无 `SoundEmitter`、未设缩放；主模拟 `persists=false`，`animover` → `ErodeAway` 默认 1 秒侵蚀后 `Remove`；客户端在 `SetPristine` 后早退，不注册清理监听。[清理](../../../raw/current/scripts/simutil.lua#L575-L597)、[端分界](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L291-L298)
- 修改边界：**脚本树内没有本族 7 名的 `SpawnPrefab` 调用**；触发链落在通用石化状态的 `fossilizable:OnSpawnFX()`，但组件实现与熔炉事件包均不在快照内。事件包缺口只影响两个本体 prefab 的 `master_postinit`／SG 接线，不影响上述独立 FX 工厂的构造与清理。[巨猪接线](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L258)、[犀牛接线](../../../raw/current/scripts/prefabs/lavaarena_rhinodrill.lua#L186)、[加载器](../../../raw/current/scripts/entityscript.lua#L77-L87)

## 注册与构造

两个文件各自定义局部 `MakeFossilizedBreakFX`（互不复用），都以 `Prefab(name, fn, <宿主资产表>)` 返回。共同构造链：`CreateEntity` → `AddTransform`／`AddAnimState`／`AddNetwork` → 仅 `FX` 标签 → 朝向数（巨猪四向、犀牛六向，与本体一致）→ **`SetBank` 被注释、不进入 pristine** → `SetBuild("fossilized")`＋`PlayAnimation("fossilized_break_fx")` → 可选符号覆盖／隐藏 → `SetPristine` → 客户端早退 → 主模拟 `persists=false`＋`animover` → `ErodeAway`。[巨猪工厂](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L265-L303)、[犀牛工厂](../../../raw/current/scripts/prefabs/lavaarena_rhinodrill.lua#L194-L228)

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `beetletaur_fossilized_break_fx_right` | `MakeFossilizedBreakFX("fossilized_break_fx", "right", false)` → 四向＋共同链 | 非中断：把 `fossilized` build 的 `rock`／`rock2` 覆盖为 `lavaarena_beetletaur` 同名符号；隐藏 `fx_lavarock_L`。[实参](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L307)、[覆盖](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L282-L285)、[隐藏](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L287-L289) |
| `beetletaur_fossilized_break_fx_left` | 同工厂，`("fossilized_break_fx", "left", false)` | 非中断、覆盖宿主岩块；隐藏 `fx_lavarock_R`。[实参](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L308) |
| `beetletaur_fossilized_break_fx_left_alt` | 同工厂，`("fossilized_break_fx", "left", true)` | `interrupted=true`：**跳过** `rock`／`rock2` 覆盖；隐藏 `fx_lavarock_R`；名字追加 `_alt`。[实参](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L309) |
| `beetletaur_fossilized_break_fx_alt` | 同工厂，`("fossilized_break_fx", "", true)` | 中断＋无侧向：跳过覆盖、也不隐藏。[实参](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L310) |
| `rhinodrill_fossilized_break_fx_right` | `MakeFossilizedBreakFX("right")` → 六向＋共同链 | 无 `interrupted`／无符号覆盖；隐藏 `fx_lavarock_L`。[实参](../../../raw/current/scripts/prefabs/lavaarena_rhinodrill.lua#L232)、[隐藏](../../../raw/current/scripts/prefabs/lavaarena_rhinodrill.lua#L211-L213) |
| `rhinodrill_fossilized_break_fx_left` | 同工厂，`("left")` | 隐藏 `fx_lavarock_R`。[实参](../../../raw/current/scripts/prefabs/lavaarena_rhinodrill.lua#L233) |
| `rhinodrill_fossilized_break_fx` | 同工厂，`("")` | 无侧向：两侧岩块都保留。[实参](../../../raw/current/scripts/prefabs/lavaarena_rhinodrill.lua#L234)、[命名](../../../raw/current/scripts/prefabs/lavaarena_rhinodrill.lua#L227) |

命名规则：巨猪为 `"beetletaur_"..anim..(side:len() > 0 and ("_"..side) or "")..(interrupted and "_alt" or "")`（[L303](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L303)）；犀牛为 `side:len() > 0 and ("rhinodrill_fossilized_break_fx_"..side) or "rhinodrill_fossilized_break_fx"`（[L227](../../../raw/current/scripts/prefabs/lavaarena_rhinodrill.lua#L227)）。**本页文件名带来源前缀 `lavaarena_`，注册名没有**：`SpawnPrefab("lavaarena_beetletaur_fossilized_break_fx_right")` 会失败，实际名是 `beetletaur_fossilized_break_fx_right`。

资源与参数：两工厂都用宿主整张 `assets` 表，不只是 `anim/fossilized.zip`——巨猪表含 `anim/lavaarena_beetletaur.zip`（`rock` 符号覆盖来源）与 `anim/fossilized.zip` 等（[L1-L10](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L1-L10)）；犀牛表用 `assets`、不含 `assets_alt`，因此 `rhinodrill2` 的衣服 build 不会被本族 FX 加载（[L1-L18](../../../raw/current/scripts/prefabs/lavaarena_rhinodrill.lua#L1-L18)）。工厂不设 `SetScale`（本体为 1.05／1.15，[L212-L213](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L212-L213)、[L139-L140](../../../raw/current/scripts/prefabs/lavaarena_rhinodrill.lua#L139-L140)）、不设 `SetFinalOffset`／bloom、无声音组件；只有 `FX` 标签。

七个注册名都没有 `STRINGS.NAMES` 条目（全语言 .po／`strings.pot` 定向检索无命中）；`NAMES.BEETLETAUR`＝“地狱独眼巨猪”、`NAMES.RHINODRILL`／`RHINODRILL2`＝“后扣帽犀牛兄弟”／“平檐帽犀牛兄弟”只是本体名（[BEETLETAUR](../../../raw/current/scripts/languages/chinese_s.po#L339369-L339372)、[RHINODRILL](../../../raw/current/scripts/languages/chinese_s.po#L348634-L348637)、[RHINODRILL2](../../../raw/current/scripts/languages/chinese_s.po#L348639-L348642)）。标题中文为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 触发链：石化状态与 `fossilizable:OnSpawnFX` | 通用模板在成功解石进入 `unfossilized` 时调 `OnSpawnFX`（`fossilized_pst` 起播）；`fossilized`／`unfossilizing` 被中断的 `onexit` 也调；当前状态带 `nofreeze` 标签时在收到 `fossilize` 事件后直接调 | 与 `_alt` 的 `interrupted` 参数命名吻合，但两只熔炉生物是否注册该模板在缺失 SG 中，`OnSpawnFX` 的选名（side／`_alt`）在缺失组件中 | [fossilized onexit](../../../raw/current/scripts/stategraphs/commonstates.lua#L1849-L1858)、[unfossilizing onexit](../../../raw/current/scripts/stategraphs/commonstates.lua#L1884-L1892)、[unfossilized](../../../raw/current/scripts/stategraphs/commonstates.lua#L1899-L1901)、[nofreeze](../../../raw/current/scripts/stategraphs/commonstates.lua#L75-L82) |
| 宿主本体：`beetletaur`／`rhinodrill`／`rhinodrill2` | 本体提供朝向与资产表；`AddOverrideBuild("fossilized")` 为石化状态预挂覆盖 build，并带 `fossilizable` 标签供组件识别 | 本体玩法、SG／Brain 与 `master_postinit` 不属本页；`master_postinit` 在缺失事件包 | [巨猪本体](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L222-L231)、[犀牛本体](../../../raw/current/scripts/prefabs/lavaarena_rhinodrill.lua#L153-L161) |
| 依赖声明：`prefabs` 表 | 巨猪依赖 4 个本族名、犀牛依赖 3 个，保证 FX prefab 随本体文件预载 | 只是加载依赖，不是生成调用；实际 `SpawnPrefab` 未在树内出现 | [巨猪依赖](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L13-L21)、[犀牛依赖](../../../raw/current/scripts/prefabs/lavaarena_rhinodrill.lua#L20-L29) |
| 通用对照：`fossilized_break_fx` | 同 build／anim、同 `animover`→`ErodeAway`；通用版无 side、无四向／六向设置、无符号覆盖、无 `_alt` | 本族是熔炉专属变体；通用四注册与石化尘雾见既有页 | [[reference/prefabs/fossilized_break_fx|化石碎裂与化石化特效]]、[通用工厂](../../../raw/current/scripts/prefabs/fossilized_fx.lua#L19-L41) |
| 缺失依赖：`fossilizable` 组件与熔炉事件包 | 全树只有对 `components.fossilizable` 的调用与 `fossilizable` 标签注释，没有组件定义文件；`lavaarena_event_server/` 目录也不在快照内，故断裂点之前的选名／生成逻辑不可见 | 事件包缺口影响本体 `master_postinit`／SG 接线，不影响独立 FX 工厂的构造与清理 | [加载器](../../../raw/current/scripts/entityscript.lua#L77-L87)、[组件引用](../../../raw/current/scripts/stategraphs/commonstates.lua#L75-L86) |
| 资源 | `anim/fossilized.zip`（build `fossilized`、anim `fossilized_break_fx`）＋宿主资产；无音效 | 包内容未解包；“不 SetBank 能否命中 bank”未运行核验 | [build/anim](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L279-L280) |

## 典型接入

### 在主模拟生成熔炉化石碎裂表现

- 前提与选择：只在主模拟 `SpawnPrefab` 正确注册名后摆位；FX 自带网络同步，客户端会得到动画副本。按宿主（巨猪／犀牛）、侧向（right／left／空）与是否需要中断版（仅巨猪有 `_alt`）选择。
- 生效与撤销：主模拟壳 `persists=false`，动画播完后经 1 秒侵蚀自删；客户端不注册清理，删除由服务端同步。不要只在客户端生成，否则该端实体不会走 `ErodeAway`。
- 原版依据：工厂行为面已核验；原版 `SpawnPrefab` 调用点在缺失事件包／组件内，以下按注册名可见性与工厂清理编写，不代表原版触发时机。

```lua
local fx = SpawnPrefab("beetletaur_fossilized_break_fx_right")
if fx ~= nil then
    fx.Transform:SetPosition(x, y, z)
end
```

- 调整与验证：换 `right`／`left`／空串与 `_alt` 改侧向／中断样式；需要贴合宿主朝向时由生成端设置 Transform 旋转（树内无调用可对照，未核验）。缩放与音效需自行补。

## 常见陷阱与限制

- **边界：页名不是注册名**：本页路径的 `lavaarena_` 前缀来自源文件名，7 个注册名都不带；按页名拼 `SpawnPrefab` 会失败。[命名](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L303)
- **边界：树内无生成点**：4095 个 Lua 文件检索（2 个非 UTF-8 文件跳过），7 名只出现在两个文件的 `prefabs` 依赖表与犀牛命名构造行；`SpawnPrefab` 无命中，`fossilizable` 组件与 `lavaarena_event_server/` 均缺失。不要据依赖声明推断生成时机。[依赖](../../../raw/current/scripts/prefabs/lavaarena_rhinodrill.lua#L20-L29)
- **边界：`SetBank` 被注释**：两工厂只 `SetBuild("fossilized")` 后直接播 `fossilized_break_fx`，注释说明是故意不放进 pristine；能否渲染依赖动画资源自身的 bank 组织，复制前先在游戏内确认。[L277-L280](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L277-L280)、[L206-L209](../../../raw/current/scripts/prefabs/lavaarena_rhinodrill.lua#L206-L209)
- **边界：客户端不清理**：`SetPristine` 后客户端分支直接 `return`，只有主模拟注册 `animover`→`ErodeAway`；只在客户端生成会留下不自删的实体。[端分界](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L291-L298)
- **边界：side 隐藏方向相反**：`right` 隐藏 `fx_lavarock_L`、`left` 隐藏 `fx_lavarock_R`，空串不隐藏；`interrupted` 只影响巨猪的 `rock`／`rock2` 覆盖，犀牛没有 `_alt` 注册。[隐藏](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L287-L289)、[犀牛注册](../../../raw/current/scripts/prefabs/lavaarena_rhinodrill.lua#L227)
- **边界：无缩放／无声音／无 NOCLICK**：本体缩放 1.05／1.15，FX 为默认缩放；只有 `FX` 标签，不复用本体声音或点击屏蔽。`CreateBreakFX`（bank `beetletaur_break`、build `lavaarena_beetletaur_break`）是同文件的 buff 碎裂本地特效，不是本页注册。[CreateBreakFX](../../../raw/current/scripts/prefabs/lavaarena_beetletaur.lua#L25-L47)

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394；两文件工厂全文、7 个注册实参与命名、`side` 隐藏／`interrupted` 覆盖分支、四向／六向、资产表与端分界、`animover`→`ErodeAway`、通用页分工、`AddFossilizedStates` 触发链、全语言 .po 名称检索、4095 个 Lua 文件名称／`SpawnPrefab` 检索（2 个非 UTF-8 文件跳过）已核验；`fossilizable` 组件与熔炉事件包缺失、7 个注册名的实际生成点与选名逻辑、动画包内容、客户端侵蚀可见性未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、parser_probe --syntax-only 均通过（2026-11-04 实际执行）。

返回 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]（上级 [[reference/prefabs/index|Prefab 索引]]）。
