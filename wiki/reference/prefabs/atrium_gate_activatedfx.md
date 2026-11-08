---
title: "远古大门充能光层与距离化警示声（atrium_gate_activatedfx 等 4 注册）"
created: "2026-11-05"
updated: "2026-11-08"
type: prefab
summary: "远古大门（atrium_gate）的激活态光效层与三个同工厂警示声注册：activatedfx 随大门状态在 idle/overload 间切换动画与标签（SetFX/KillFX 协议）；pulsesfx/explodesfx/grotto_war_sfx 由同一 makesfx 工厂生成的联网声效代理——各端在焦点附近建本地声源、1 秒自毁，按玩家所在区域决定可闻性。"
aliases: ["atrium_gate_activatedfx", "atrium_gate_pulsesfx", "atrium_gate_explodesfx", "grotto_war_sfx", "远古大门充能光效", "大门过载特效", "地穴警示声", "档案开战音", "atrium gate fx", "梦魇区脉冲声"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/atrium_gate_activatedfx.lua", "prefabs/atrium_gate_pulsesfx.lua", "prefabs/atrium_gate.lua", "components/grottowarmanager.lua", "prefabs/archive_props.lua", "components/highlight.lua", "components/playercontroller.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 远古大门充能光层与距离化警示声（atrium_gate_activatedfx 等 4 注册）

## 快速判断

- 职责与适用：远古大门（`atrium_gate`，官方名"远古大门"，[po](../../../raw/current/scripts/languages/chinese_s.po#L338919-L338922)）体系里的 **4 个表现注册**：`atrium_gate_activatedfx` 是父挂大门的**激活/过载光效层**（泛光动画＋netvar 式状态切换），`atrium_gate_pulsesfx`/`atrium_gate_explodesfx` 是大门不稳定期的**周期脉冲/爆炸警示声代理**，`grotto_war_sfx` 是地穴战线开战音代理——后三者出自同一 `makesfx` 工厂，结构完全一致。注册行：[activatedfx](../../../raw/current/scripts/prefabs/atrium_gate_activatedfx.lua#L99)、[pulsesfx](../../../raw/current/scripts/prefabs/atrium_gate_pulsesfx.lua#L72)、[explodesfx](../../../raw/current/scripts/prefabs/atrium_gate_pulsesfx.lua#L73)、[grotto_war_sfx](../../../raw/current/scripts/prefabs/atrium_gate_pulsesfx.lua#L74)
- 三兄弟辨析：makesfx 生成的代理实体只有 Transform＋Network＋`FX` 标签，**声音不在代理上播**——非专服各端 0 秒后在 `TheFocalPoint` 处建非联网本地声源、按"门到焦点的距离"公式定位、播一次性声后立刻自删；代理自身主模拟 1 秒自毁（[工厂](../../../raw/current/scripts/prefabs/atrium_gate_pulsesfx.lua#L43-L70)）。三者唯一差异是声音路径与地区门控：pulsesfx/explodesfx 只在玩家处于 `Nightmare` 标签区域时可闻，grotto_war_sfx 反向——除地穴（vault）与查理 Boss 竞技场外随处可闻。
- 重要副作用：`atrium_gate_activatedfx` 自带 Light 配置但**从未被启用**（工厂 `Enable(false)` 后无人打开，光效实际由大门本体的 Light 承担）；SetFX 在非首次切换时同步交换标签——`idle` 态带 `FX`、`overload` 态换成 `DECOR`＋`NOCLICK`（首次 `SetFX("idle")` 不触发该分支，标签来自工厂；[标签交换](../../../raw/current/scripts/prefabs/atrium_gate_activatedfx.lua#L18-L26)）。
- 修改边界：只覆盖 4 注册的工厂、生成点、父子与清理；大门本体（钥匙镶嵌、不稳定计时、石精守护战、召唤仪式、RITUAL_STATES 状态机）只写与 FX 相关的触发点，玩法全貌不入本页。
- 名称：4 个覆盖注册（`ATRIUM_GATE_ACTIVATEDFX`、`ATRIUM_GATE_PULSESFX`、`ATRIUM_GATE_EXPLODESFX`、`GROTTO_WAR_SFX`）在简中 `chinese_s.po` 均无 `STRINGS.NAMES` 键，标题与正文中文为已核验效果描述，非官方译名。

## 注册与构造

### atrium_gate_activatedfx（状态切换光效层）

| 项 | 内容 | 证据 |
|---|---|---|
| 壳构造 | Transform／AnimState／SoundEmitter／Light／Network；bank/build `atrium_gate_overload_fx`、开场 `idle_pre`、泛光 `shaders/anim.ksh`、FinalOffset 1、标签 `FX`；Light（半径 8、强度 0.65、falloff 0.9、色 200/140/140）`Enable(false)` | [工厂](../../../raw/current/scripts/prefabs/atrium_gate_activatedfx.lua#L54-L97) |
| 端分界 | `SetPristine` → 客户端 `OnEntityReplicated`：父级是 `atrium_gate` 则把自己登记进父级 `highlightchildren`；主模拟：`anim=nil`、补推 `idle_loop` 循环、暴露 `SetFX`/`KillFX`、`persists=false` | [客户端登记](../../../raw/current/scripts/prefabs/atrium_gate_activatedfx.lua#L40-L45)、[服务端段](../../../raw/current/scripts/prefabs/atrium_gate_activatedfx.lua#L88-L94) |
| `SetFX(inst, anim)` | `killed` 后静默；目标态与当前态不同才动：已有状态则推 `旧态_pst`→`新态_pre`→`新态_loop`(循环)，无状态则直接 `新态_pre`→`新态_loop`；同时换标签（`idle`→`FX`，其余→`DECOR`＋`NOCLICK`） | [L6-L29](../../../raw/current/scripts/prefabs/atrium_gate_activatedfx.lua#L6-L29) |
| `KillFX(inst)` | `killed` 置位后推 `当前态_pst`，`animqueueover` 自删＋4 秒定时兜底；重复调用静默 | [L31-L38](../../../raw/current/scripts/prefabs/atrium_gate_activatedfx.lua#L31-L38) |

实际只用两个状态：`"idle"`（大门修复完毕待战）与 `"overload"`（不稳定期）。首次生成即 `SetFX("idle")` 时，工厂已播 `idle_pre`/`idle_loop` 并带了 `FX` 标签；`(nil or "idle") ~= "idle"` 为假，SetFX 只把 `inst.anim` 记为 `idle`，既不补标签也不改动画队列；后续切到 `overload` 才走标签交换。

### makesfx 工厂三兄弟（pulsesfx、explodesfx、grotto_war_sfx）

| 注册名 | 声音路径 | 地区门控（本地声源是否创建） | 生成点 |
|---|---|---|---|
| `atrium_gate_pulsesfx` | `dontstarve/common/together/atrium_gate/shadow_pulse` | 仅当 ThePlayer 处于 areaaware 标签 `Nightmare` 区域 | [atrium_gate.lua L311](../../../raw/current/scripts/prefabs/atrium_gate.lua#L288-L315) |
| `atrium_gate_explodesfx` | `dontstarve/common/together/atrium_gate/explode` | 同上（Nightmare 区域） | [atrium_gate.lua L401](../../../raw/current/scripts/prefabs/atrium_gate.lua#L397-L402) |
| `grotto_war_sfx` | `grotto/common/archive_switch/shadow_war` | 反向：ThePlayer 在任意 vault 或查理 Boss 竞技场内则不播，其余播放 | [grottowarmanager.lua L162](../../../raw/current/scripts/components/grottowarmanager.lua#L133-L172)、[L266](../../../raw/current/scripts/components/grottowarmanager.lua#L255-L266) |

三者的工厂体逐字节同构（[makesfx](../../../raw/current/scripts/prefabs/atrium_gate_pulsesfx.lua#L43-L70)）：代理建完 `SetPristine` 前，非专服先挂 0 秒任务调 `PlayWarningSound`；主模拟段 `SetCanSleep(false)`＋`persists=false`＋1 秒自毁。本地声源端（[PlayWarningSound](../../../raw/current/scripts/prefabs/atrium_gate_pulsesfx.lua#L3-L41)）：Transform＋SoundEmitter、`SetParent(TheFocalPoint.entity)`，按 `30×(1−((0.6)/((距离/15)²+1)+0.4))` 把声源沿"门→焦点"方向摆在焦点旁 0～18 单位处（越远越偏离屏幕中心），`PlaySound` 后立即 `Remove()`。

## 关联机制导航

### activatedfx 状态切换链（谁调 SetFX/KillFX）

| 大门状态 | 调用 | FX 动作 | 证据 |
|---|---|---|---|
| 修复完毕、石精守护战开始 | `OnTrackStalker` → `ShowFx(inst, "idle")` | 首次生成＋idle 态 | [L482](../../../raw/current/scripts/prefabs/atrium_gate.lua#L475-L498) |
| 读档时处于不稳定延迟期 | `OnLoadPostPass` → `OnQueueDestabilize(inst, true)` → `ShowFx(inst, "idle")` | 读档补建 idle 态 | [L354](../../../raw/current/scripts/prefabs/atrium_gate.lua#L352-L371)、[L608-L609](../../../raw/current/scripts/prefabs/atrium_gate.lua#L583-L612) |
| 延迟期结束进入不稳定 | `StartDestabilizing` → `ShowFx(inst, "overload")`；本体 Light 打开、过载循环声起 | idle→overload 切换：推 `idle_pst`→`overload_pre`→`overload_loop` | [L334](../../../raw/current/scripts/prefabs/atrium_gate.lua#L317-L350) |
| 不稳定期每 16.5 秒脉冲 | `OnDestabilizingPulse`（首拍 2 秒后）→ 大门本体播 `overload_pulse`＋生成 pulsesfx；周期/首拍取自调值（配置值 16.5/2 秒） | FX 层不变，仅本体动画 | [L288-L315](../../../raw/current/scripts/prefabs/atrium_gate.lua#L288-L315)、[DoPeriodicTask L349](../../../raw/current/scripts/prefabs/atrium_gate.lua#L349)、[调值](../../../raw/current/scripts/tuning.lua#L3419-L3423) |
| 爆炸收尾 | `OnDestabilizeExplode` → `HideFx`（KillFX）→ 1.75 秒后关本体 Light、47 帧后弹回钥匙 | 推 `overload_pst` 后 `animqueueover` 或 4 秒自删 | [L397-L410](../../../raw/current/scripts/prefabs/atrium_gate.lua#L397-L423) |
| 取回钥匙复位 | `OnKeyTaken` → `HideFx` | 同上（此时多为 idle 态） | [L234](../../../raw/current/scripts/prefabs/atrium_gate.lua#L220-L240) |

生成与登记（[ShowFx L160-L168](../../../raw/current/scripts/prefabs/atrium_gate.lua#L160-L168)）：唯一生成点 `atrium_gate.lua:162`，`SpawnPrefab` 后 `SetParent(大门)` 并插入服务端 `highlightchildren`；客户端副本经 `OnEntityReplicated` 自行登记同一张表，`OnRemoveEntity` 注销（[L47-L52](../../../raw/current/scripts/prefabs/atrium_gate_activatedfx.lua#L47-L52)）。该表由 highlight 组件的 `ApplyColour` 遍历（[highlight.lua L81-L92](../../../raw/current/scripts/components/highlight.lua#L81-L92)）；highlight 并非大门工厂静态装配，而是玩家控制器选中有效实体后按需 `AddComponent("highlight")`（[playercontroller L2850-L2869](../../../raw/current/scripts/components/playercontroller.lua#L2850-L2869)），所以这是“选中后”的高亮传播通道，不是预留接口。大门工厂本身未装 highlight（同表还登记 shrouden 部件，[L1241](../../../raw/current/scripts/prefabs/atrium_gate.lua#L1239-L1245)），未选中时不会自动高亮；具体哪些状态下可见未运行核验。

### grotto_war_sfx 的档案开战链

华丽基座（archive_switch）通电后 `startshadowwar` 起 7 秒任务推 `ms_archivesbreached`（[archive_props.lua L692-L699](../../../raw/current/scripts/prefabs/archive_props.lua#L692-L699)）→ grottowarmanager 监听该事件开战（[L347](../../../raw/current/scripts/components/grottowarmanager.lua#L347)）→ 铺设战线（梦魇裂缝与梦魇生长器交错布防）时在战线中点生成 `grotto_war_sfx`——改造世界路径 [L162](../../../raw/current/scripts/components/grottowarmanager.lua#L133-L172)，常规路径 [L266](../../../raw/current/scripts/components/grottowarmanager.lua#L255-L266)。战争管理器玩法详见 [[reference/components/grottowarmanager|地穴战争管理器]]。

### 清理路径

| 注册 | 清理方式 | 边界与证据 |
|---|---|---|
| `atrium_gate_activatedfx` | `KillFX`：`killed` 门控 → 推退场动画 → `animqueueover` 自删＋4 秒兜底；父级大门被直接删除时脚本层没有显式清理，原生 `SetParent` 是否带走未核验，以 KillFX 为确定收尾路径 | FX 层 `persists=false`，读档由大门 ShowFx 重建 |
| 三兄弟代理 | 主模拟 1 秒 `DoTaskInTime(1, Remove)` 自毁 | `SetCanSleep(false)` 防止 1 秒内休眠跳过任务（[L63-L66](../../../raw/current/scripts/prefabs/atrium_gate_pulsesfx.lua#L63-L66)） |
| 本地声源 | `PlaySound` 后同帧 `Remove()` | 声音后续由引擎音频层接管，脚本层无生命周期维护 |

## 典型接入

### 复刻"状态切换式"跟随 FX 的 Set/Kill 协议

- 前提与选择：宿主有多个长驻形态时，FX 工厂暴露 `SetFX(state)`/`KillFX()`，把动画推进（旧态 pst→新态 pre→新态 loop）与标签交换收进工厂内部；宿主只记一个引用、按状态机转发。适合大门充能、护盾层、形态光环这类"同一实体、多阶段演出"。
- 生效与撤销：首次 SetFX 前工厂需自播好初始态（本例 `idle_pre`/`idle_loop`）；撤销走 KillFX 的 `killed` 门控＋退场动画＋双通道自删（animqueueover＋定时兜底），宿主侧引用置空。

```lua
-- 服务端：宿主侧的生成与切换（对应 atrium_gate.ShowFx/HideFx 的模式）
if inst._gatefx == nil then
    inst._gatefx = SpawnPrefab("atrium_gate_activatedfx")
    inst._gatefx.entity:SetParent(inst.entity)
end
inst._gatefx:SetFX("overload")   -- 状态切换；"idle"/"overload" 之外的值也按同一链推进
-- 收尾：
if inst._gatefx ~= nil then
    inst._gatefx:KillFX()
    inst._gatefx = nil
end
```

- 调整与验证：替换点是状态名与动画名（约定 `<state>_pre/_loop/_pst` 三件套）；`SetFX` 对相同状态静默、`KillFX` 后再 SetFX 全静默（`killed` 门控）。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：`atrium_gate_activatedfx` 的 Light 是死配置。** 工厂配置了半径 8 的红光但 `Enable(false)`（[L63-L67](../../../raw/current/scripts/prefabs/atrium_gate_activatedfx.lua#L63-L67)），atrium_gate.lua 全文无人打开它；不稳定期的实际光效来自大门本体自己的 Light（[本体同参数 Light](../../../raw/current/scripts/prefabs/atrium_gate.lua#L343-L347)）。复用时别指望 FX 层发光。
- **边界：`SetFX("idle")` 首次调用不播动画。** `(inst.anim or "idle") ~= "idle"` 在初始态为假，工厂预播的 `idle_pre`/`idle_loop` 就是展示内容；SetFX 只负责标签交换与后续切换（[L9](../../../raw/current/scripts/prefabs/atrium_gate_activatedfx.lua#L6-L29)）。
- **边界：`killed` 之后一切静默。** KillFX 置位后 SetFX 直接 return、KillFX 自身也只走一次；4 秒兜底保证就算 `animqueueover` 不触发（动画睡眠）也能退场（[L31-L38](../../../raw/current/scripts/prefabs/atrium_gate_activatedfx.lua#L31-L38)）。
- **误区：三兄弟的声音在代理实体上。** 代理只是 1 秒命的联网锚点，真正的声音来自各端焦点旁的本地非联网声源；专服完全不建声源（`IsDedicated` 门控）。把代理当 SoundEmitter 实体用是错的（[工厂](../../../raw/current/scripts/prefabs/atrium_gate_pulsesfx.lua#L43-L70)）。
- **边界：警示声是"局部可闻"而非全图广播。** pulsesfx/explodesfx 仅当玩家在 `Nightmare` 标签区域（`areaaware:CurrentlyInTag`）才创建声源；grotto_war_sfx 在地穴与查理竞技场内被静音（源码注释原话 "anywhere except vault =)"，[L3-L21](../../../raw/current/scripts/prefabs/atrium_gate_pulsesfx.lua#L3-L21)）。测试时站错区域会误判"没播"。
- **边界：`grotto_war_sfx` 归属在档案侧、注册在门侧。** 它的名字与声音都属于地穴战争（`archive_switch/shadow_war`），但注册行在 atrium_gate_pulsesfx.lua——按注册处主要归属归本页；其生成点全部在 grottowarmanager（[L162](../../../raw/current/scripts/components/grottowarmanager.lua#L162)、[L266](../../../raw/current/scripts/components/grottowarmanager.lua#L266)），`archive_props.lua:47` 只是依赖表登记、无生成。
- **边界：大门本体是重度玩法载体。** trader/pickable/worldsettingstimer/charliecutscene 全在本体上，FX 只是演出层；破坏大门循环、仪式召唤等请从本体页或组件页入手，本页不展开。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/atrium_gate_activatedfx.lua`、`prefabs/atrium_gate_pulsesfx.lua`、`prefabs/atrium_gate.lua`，按需读取 `components/grottowarmanager.lua` 战线生成与开战监听段、`prefabs/archive_props.lua` startshadowwar 段、`components/highlight.lua` ApplyColour 段、`tuning.lua` ATRIUM_GATE 调值段。对 4 个注册名以完整名在 `raw/current/scripts` 全域反查：`atrium_gate_activatedfx` 命中 prefablist.lua:70、atrium_gate.lua:21 依赖、:162 生成、注册行——生成点唯一（台账批 2 结论"atrium_gate.lua:162 生成"复核属实，ShowFx 函数体 L160-L168）；`atrium_gate_pulsesfx` 命中 prefablist.lua:71、atrium_gate.lua:22 依赖、:311 生成、注册行——生成点唯一；`atrium_gate_explodesfx` 命中 atrium_gate.lua:23 依赖、:401 生成、注册行——生成点唯一；`grotto_war_sfx` 命中 grottowarmanager.lua:162/:266 生成、atrium_gate_pulsesfx.lua:74 注册、archive_props.lua:47 依赖表——生成点两处且均在战线铺设内。`highlightchildren` 的读者为 components/highlight.lua（L81-L92、L135-L136）与 prefabskin/mast 的皮肤/船桅自用表；highlight 组件本身由 playercontroller.lua L2850-L2869 在选中有效实体时按需安装，故大门未静态装配 highlight 不影响该通道成立，只是“选中后”才高亮。
- 名称核验：`languages/chinese_s.po` 定向检索——`ATRIUM_GATE`＝"远古大门"（锚点见快速判断）；`ATRIUM_GATE_ACTIVATEDFX`、`ATRIUM_GATE_PULSESFX`、`ATRIUM_GATE_EXPLODESFX`、`GROTTO_WAR_SFX` 的 NAMES 键全文 0 命中，标题使用效果描述。
- 分档说明：`standard`（字段省略）——activatedfx 为双状态单链、三兄弟为同构工厂，均无多变体或多端子实体系统；消费链集中在一个宿主文件加一个组件文件，standard 预算内可完整承载。
- 未覆盖：大门本体的钥匙交易/仪式/RITUAL_STATES 全链（仅记 FX 触发点）、石精守护战与查理 Boss 竞技场玩法、动画包内容与观感、`PlayWarningSound` 距离公式对应的实际听感衰减（仅记公式与取值范围 0～18）、引擎父挂级联在宿主移除时的实测行为；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录。
- 第 16 轮补正（2026-11-07）：首次 `SetFX("idle")` 只记状态、不改标签队列；`highlightchildren` 改述为玩家控制器按需装 highlight 后的选中高亮通道（补 `components/playercontroller.lua` 来源）；宿主直接删除的原生结果保留未核验，清理以 KillFX 为准。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/archive_ambient_sfx|远古档案机关声画实体]]（同为档案区设施声画；`grotto_war_sfx` 的开战事件源自华丽基座）
- [[reference/components/grottowarmanager|地穴战争管理器]]（`grotto_war_sfx` 生成所在的战线铺设组件页）
- 返回 [[reference/prefabs/fx-structure-index|建筑、装置与遗迹演出]]
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-06）。
