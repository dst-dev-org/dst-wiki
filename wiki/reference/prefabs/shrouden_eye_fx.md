---
title: "暗影憎恶死亡水洼之眼（shrouden_eye_fx）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "暗影憎恶死亡演出中浮在死亡水洼里的可检视眼睛：联网动画壳＋每端本地 goop 层，用 net_event 同步动画，inspectable 状态 PUDDLE 走专属检视文本；随死亡水洼状态或 Boss 移除而清理。"
aliases: ["shrouden_eye_fx", "暗影憎恶之眼", "死亡水洼之眼", "水洼眼睛", "暗影憎恶眼睛"]
tags: [prefab, fx, server-client]
size_class: standard
dst_build: "steam-buildid:26335394"
sources: ["prefabs/shrouden.lua", "stategraphs/SGshrouden.lua", "components/inspectable.lua", "stringutil.lua", "entityscript.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 暗影憎恶死亡水洼之眼（shrouden_eye_fx）

## 快速判断

- 职责与适用：`prefabs/shrouden.lua` 末尾与 Boss 本体一起返回的**第二个注册** `shrouden_eye_fx`，是暗影憎恶（`shrouden`，官方名"暗影憎恶"）进入死亡演出后浮在死亡水洼里的眼睛。它不是纯装饰：带 `inspectable`，检视状态返回 `"PUDDLE"`，会把检查文本切到专属的"戳它眼睛"台词。[注册](../../../raw/current/scripts/prefabs/shrouden.lua#L1620-L1621)、[检视状态](../../../raw/current/scripts/prefabs/shrouden.lua#L1561-L1563)
- 优先使用：给自定义 Boss 做"死亡后水洼里留一只可检视眼睛"的演出时，可复用这只眼睛的"网络动画壳＋本地 goop 层"结构；驱动入口是主模拟上的 `SyncAnim(anim, loop, frame)`。[同步](../../../raw/current/scripts/prefabs/shrouden.lua#L1550-L1559)
- 重要副作用：`SetPrefabNameOverride("shrouden")` 让它按 Boss 的名字显示与检视；`displaynamefn` 还会按玩家身份切换"藏／异者／暗域"等称呼。眼睛本身**没有 `FX` 标签、没有伤害、没有自删**，`persists=false`。[名称覆写](../../../raw/current/scripts/prefabs/shrouden.lua#L1590-L1597)、[显示名](../../../raw/current/scripts/prefabs/shrouden.lua#L1209-L1221)
- 修改边界：只覆盖这 1 个注册的工厂、本地 goop 层、动画同步协议、死亡水洼各状态的生成／清理与名称／检视链；暗影憎恶本体战斗、`shrouden_optic_blast_fx` 光束（已由既有页承载）不展开。
- 名称：`STRINGS.NAMES.SHROUDEN`＝"暗影憎恶"（[po](../../../raw/current/scripts/languages/chinese_s.po#L349359-L349362)）；`shrouden_eye_fx` 在简中 `.po` 无 `STRINGS.NAMES` 键，标题中的"死亡水洼之眼"是已核验效果描述。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `shrouden_eye_fx` | `eyefn()`：Transform／AnimState／Network → 六向 → bank `shrouden`、build `shrouden_voidcloth`（**故意用错 build 让符号默认空白**）、`Hide("sh_eye")`、5 个 `OverrideSymbol`（`sh_eye_pupil`／`RED_fx_eye_ball`／`RED_fx_eye_flame`／`fx_bubble`／`RED_swirl_01`，均取自 `shrouden_build`）、播 `death`、光覆写 1 → `net_event("shrouden_eye_fx.animsync")` → 高亮覆写 → `SetPrefabNameOverride("shrouden")` → 非专服建本地 goop 层并父挂 → `displaynamefn` → `SetPristine`。客户端加 `updatelooper` 并监听 `animsync`；主模拟加 `inspectable`（`getstatus`＝`eye_GetStatus`）、`persists=false`、导出 `SyncAnim`。 | 单一注册；**工厂阶段不加 `FX` 标签**，它不是按 FX 标签筛选的实体；无 health／combat／physics。goop 层只在非专服创建，是本地实体。[工厂](../../../raw/current/scripts/prefabs/shrouden.lua#L1565-L1616) |

共享资产来自 Boss 本体：`shrouden_build`／`shrouden_basic`／`shrouden_actions`／`shrouden_death`／`shrouden_voidcloth` 五个 zip（[assets](../../../raw/current/scripts/prefabs/shrouden.lua#L1-L8)）。眼睛的 `prefabs` 依赖也在 Boss 文件的依赖表里（[依赖](../../../raw/current/scripts/prefabs/shrouden.lua#L10-L30)）。

### 本地 goop 层

`CreateEyeGoopLayer()` 造一个**非联网**的贴地水洼层（[实现](../../../raw/current/scripts/prefabs/shrouden.lua#L1490-L1512)）：

- `DECOR`＋`NOCLICK`、`persists=false`、六向；
- 同样用 `shrouden_voidcloth` 这个"错 build"让符号空白，只覆盖 `puddle_eye_lip`；
- 初始播 `death`，`SetFinalOffset(1)`、光覆写 1；
- 客户端在眼睛构造时创建并 `SetParent` 到眼睛实体；父挂关系已确认，随父移动，删除级联未核验（不据此断言 goop 必然残留或必然随删）。

眼睛本体负责"眼睛＋眼白／火焰／气泡／旋涡符号"，goop 层负责水洼边缘；两者用同一套动画时间对齐。[goop 创建](../../../raw/current/scripts/prefabs/shrouden.lua#L1592-L1595)

## 动画同步协议

客户端没有 SG；除工厂初始的 `death` 外，后续动画只由 `animsync` 驱动。它监听网络事件，然后用 `updatelooper` 的后更新把 goop 层对到眼睛当前帧（[客户端](../../../raw/current/scripts/prefabs/shrouden.lua#L1601-L1606)、[同步实现](../../../raw/current/scripts/prefabs/shrouden.lua#L1514-L1548)）：

| 步骤 | 行为 | 证据 |
|---|---|---|
| 主模拟 `SyncAnim(anim, loop, frame)` | `PlayAnimation(anim, loop)`；`frame` 非空时 `SetFrame(frame)`；`animsync:push()` | [L1550-L1555](../../../raw/current/scripts/prefabs/shrouden.lua#L1550-L1555) |
| 客户端收到事件 | `eye_OnAnimSync` 置 `_animsyncing` 并挂 `eye_DoAnimSync` 后更新 | [L1543-L1548](../../../raw/current/scripts/prefabs/shrouden.lua#L1543-L1548) |
| 后更新对帧 | 按眼睛当前动画名把 goop 切到同名动画并 `SetTime` 对齐；`final_blow` 时给眼睛加 `NOCLICK`，其余动画移除 `NOCLICK` | [L1514-L1541](../../../raw/current/scripts/prefabs/shrouden.lua#L1514-L1541) |
| 主模拟本地 | 若本机也有 goop（非专服主客机），`SyncAnim` 直接调一次 `eye_DoAnimSync`，不依赖网络回环 | [L1556-L1558](../../../raw/current/scripts/prefabs/shrouden.lua#L1556-L1558) |

`final_blow` 是唯一给眼睛加 `NOCLICK` 的动画：最终一击演出期间眼睛不可点；`death`／`death_puddle_loop`／`death_puddle_loop2`／`death_puddle_hit` 都会移除 `NOCLICK`，恢复可检视。[分支](../../../raw/current/scripts/prefabs/shrouden.lua#L1519-L1540)

## 死亡水洼中的出现与清理

眼睛由 `SGshrouden` 的 `ShowPuddleEye` 按需生成（[实现](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L237-L248)）：

```lua
local eye = SpawnPrefab("shrouden_eye_fx")
local x, _, z = inst.Transform:GetWorldPosition()
eye.Transform:SetPosition(x, 0, z)
eye.Transform:SetRotation(inst.Transform:GetRotation())
eye:ListenForEvent("onremove", function(inst) eye:Remove() end, inst)
inst.sg.mem.eye = eye
eye:SyncAnim(anim, loop, frame)
```

- 位置取 Boss 当前 x/z，**y 强制 0**（贴地），朝向继承 Boss；
- `sg.mem.eye` 是单槽缓存：已有眼睛时只 `SyncAnim`，不会重复生成；
- 监听 Boss 的 `onremove`，Boss 被删时眼睛跟着删；
- 真正的清理入口是 `HidePuddleEye`：`sg.mem.eye:Remove()` 并清空槽位与转向节流字段（[清理](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L250-L256)）。

眼睛在各死亡状态里的动画与清理（[状态表](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L880-L1279)）：

| 状态 | 眼睛动画 | 生成／保留 | 清理 |
|---|---|---|---|
| `death` | `death` | `ShowPuddleEye` | `onexit` 在非 `statemem.death` 时 `HidePuddleEye` |
| `death_fr94` | `death`，`SetFrame(94)` | 复用同一只眼睛 | 同上 |
| `death_loop` | `death_puddle_loop`（循环） | 复用 | `onexit`（非 `statemem.death` 时 `HidePuddleEye`） |
| `death_blink` | `death_puddle_loop2`（循环） | 复用 | `onexit`（非 `statemem.death` 时 `HidePuddleEye`） |
| `death_hit` | `death_puddle_hit` | 复用 | `onexit`（非 `statemem.death` 时 `HidePuddleEye`） |
| `death_final` | `final_blow` | 复用；加 `NOCLICK` | 超时 `inst:Remove()`，`onexit` 再 `HidePuddleEye` |

上表各状态（`death_final` 除外）的 `onexit` 清理都受 `not inst.sg.statemem.death` 门禁；正常链内切换会先置 `statemem.death = true`，因此 `onexit` 不会在转换时把眼睛删掉；直到 `death_final` 超时移除 Boss，眼睛才随 `onremove`／`onexit` 一起消失。[中间转换](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L880-L1015)、[最终清理](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L1201-L1279)

### 水洼期的朝向

`OnUpdatePuddle` 在死亡水洼状态（`dt > 0` 且未禁用 stomp 时）找最近的**移动中**合法玩家，若眼睛存在且距上次转向超过 0.2 秒，就把眼睛转向该玩家（`math.atan2(mindz, -mindx)`）——这是眼睛"盯着玩家"的来源，逻辑在 SG，不在眼睛工厂。[转向](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L288-L294)

## 名称与检视

- `SetPrefabNameOverride("shrouden")` 只写 `inst.nameoverride`，不改 `prefab`；`GetBasicDisplayName` 先看 `displaynamefn`，再回退到 `nameoverride` 的 `STRINGS.NAMES`。[覆写](../../../raw/current/scripts/entityscript.lua#L846-L850)、[取名](../../../raw/current/scripts/entityscript.lua#L687-L691)
- `displaynamefn = DisplayNameFn`：玩家带 `shadowthrall_parasite_mask` 时显示"暗域"（`SHROUDEN_VOIDMASQUE`），装备 `ancient_reader` 时显示"异者"（`SHROUDEN_OTHER`），暗影阵营玩家显示"藏"（`SHROUDEN_ALLEGIANCE`），否则返回 nil 并落到"暗影憎恶"。[实现](../../../raw/current/scripts/prefabs/shrouden.lua#L1209-L1221)、[po 四名](../../../raw/current/scripts/languages/chinese_s.po#L349359-L349377)
- 检视：`inspectable.getstatus = eye_GetStatus` 恒返回 `"PUDDLE"`；`Inspectable:GetStatus` 用它，`GetDescription` 以 `item.nameoverride`（即 `"shrouden"`）拼出 `DESCRIBE.SHROUDEN.PUDDLE`，于是检查文本变成"我想我该……戳它眼睛！"而不是 Boss 的通用台词。[状态](../../../raw/current/scripts/prefabs/shrouden.lua#L1561-L1563)、[inspectable](../../../raw/current/scripts/components/inspectable.lua#L27-L47)、[描述键](../../../raw/current/scripts/stringutil.lua#L364-L380)、[po 台词](../../../raw/current/scripts/languages/chinese_s.po#L19489-L19492)

## 关联机制导航

| 类型／对象 | 本注册如何使用 | 变体、端与条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `Prefab：shrouden` | 宿主 Boss；眼睛由它的 SG 生成，Boss 删除时连带删除 | 只有死亡演出会生成；本体战斗不生成 | [注册](../../../raw/current/scripts/prefabs/shrouden.lua#L1620-L1620) |
| `SG：SGshrouden` | `ShowPuddleEye`／`HidePuddleEye`／`OnUpdatePuddle` 驱动 | 六个死亡相关状态 | [生成](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L237-L248)、[清理](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L250-L256) |
| `组件：inspectable` | `getstatus` 返回 `PUDDLE`，驱动专属检视文本 | 只在主模拟挂载 | [挂载](../../../raw/current/scripts/prefabs/shrouden.lua#L1608-L1609) |
| `网络：net_event` | `shrouden_eye_fx.animsync` 把主模拟动画切到客户端 | 客户端只监听，不主动切 | [事件](../../../raw/current/scripts/prefabs/shrouden.lua#L1585-L1585)、[监听](../../../raw/current/scripts/prefabs/shrouden.lua#L1603-L1603) |
| `关联 Prefab：shrouden_optic_blast_fx` | 同属暗影憎恶的表现，但那是独立注册的位面光束，已有页承载 | 与本页无父子／生成关系 | [[reference/prefabs/shadow_despawn|暗影消散、舞台影手与暗影憎恶光束]] |
| `本地实体：goop 层` | 非联网水洼边缘，只在客户端与主客机创建 | 专服无 goop；随眼睛父挂 | [创建](../../../raw/current/scripts/prefabs/shrouden.lua#L1490-L1512) |

## 典型接入

### 在自定义死亡水洼里放一只可检视眼睛

- 前提与选择：宿主必须在主模拟；眼睛的 `SyncAnim` 只在主模拟存在。仅需视觉时可直接生成并让它停在 `death`；需要循环水洼动画时由服务端调 `SyncAnim`。
- 生效与撤销：眼睛不自删，演出结束或宿主移除时必须显式 `Remove()`；`persists=false` 不入存档。
- 原版依据：[ShowPuddleEye](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L237-L248)、[SyncAnim](../../../raw/current/scripts/prefabs/shrouden.lua#L1550-L1559)。

```lua
local function SpawnPuddleEye(x, z)
    local eye = SpawnPrefab("shrouden_eye_fx")
    eye.Transform:SetPosition(x, 0, z)
    eye:SyncAnim("death_puddle_loop", true) -- 主模拟调用；客户端经 animsync 跟随
    return eye -- 结束或宿主移除时 eye:Remove()
end
```

- 调整与验证：替换点是落点、初始动画与是否循环；`SetFrame` 可用于对齐某一帧（原版 `death_fr94` 用 94）。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：它不是 `FX` 标签实体。** 工厂只加 Transform／AnimState／Network，没有 `FX`／`NOCLICK`；按 FX 标签做清场或统计时不会命中它。`NOCLICK` 只在 `final_blow` 期间临时加。[工厂](../../../raw/current/scripts/prefabs/shrouden.lua#L1565-L1599)、[NOCLICK 分支](../../../raw/current/scripts/prefabs/shrouden.lua#L1519-L1527)
- **边界：错 build 是故意的。** 眼睛与 goop 都用 `shrouden_voidcloth` 让符号默认空白，再逐个 `OverrideSymbol`；改成 `shrouden_build` 会把整张 Boss 贴图带出来，不是"修正"。[注释](../../../raw/current/scripts/prefabs/shrouden.lua#L1574-L1581)
- **边界：goop 层不联网。** 它只在非专服创建并 `SetParent` 到眼睛；专服没有这层，客户端各自建。不要把它当成会被网络复制的子实体。[创建条件](../../../raw/current/scripts/prefabs/shrouden.lua#L1592-L1595)
- **边界：眼睛单槽复用。** `ShowPuddleEye` 发现 `sg.mem.eye` 非空就只 `SyncAnim`，不会生成第二只；手动生成多只会叠在同一位置。[单槽](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L237-L248)
- **边界：没有自删。** 眼睛没有 `animover`／定时器清理，`persists=false` 只表示不入存档；SG 的 `HidePuddleEye` 与 Boss `onremove` 才是实际退场路径。[清理](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L250-L256)
- **误区：它是"Boss 死后永久留下的尸体眼"。** 它只存在于 `death`～`death_final` 的死亡水洼演出；`death_final` 超时会移除 Boss，眼睛随之消失。把它当永久世界物会漏掉清理。[最终状态](../../../raw/current/scripts/stategraphs/SGshrouden.lua#L1201-L1279)
- **误区：`shrouden_eye_fx` 有官方中文名。** 简中 `.po` 没有它的 `STRINGS.NAMES` 键；显示的是 `SetPrefabNameOverride("shrouden")` 指向的 Boss 名（或 `displaynamefn` 的身份称呼），不是眼睛自己的名字。[po](../../../raw/current/scripts/languages/chinese_s.po#L349359-L349377)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/shrouden.lua` 的眼睛段（资产 L1-L8、依赖 L10-L30、`DisplayNameFn` L1209-L1221、goop L1490-L1512、同步 L1514-L1559、`eyefn` L1565-L1621）；按需读取 `stategraphs/SGshrouden.lua`（`ShowPuddleEye`／`HidePuddleEye` L237-L256、`OnUpdatePuddle` 转向 L286-L294、六个死亡状态 L880-L1279）、`components/inspectable.lua`（`GetStatus`／`GetDescription` L27-L79）、`stringutil.lua`（`GetDescription` 的 `itemname` 与 modifier 解析 L364-L380）、`entityscript.lua`（`GetBasicDisplayName` L687-L691、`SetPrefabNameOverride` L846-L850）。全域反查（`grep -rn "shrouden_eye_fx" --include=*.lua`）：仅 `shrouden.lua` 的依赖 L12、`net_event` 名 L1585、注册 L1621、`SGshrouden.lua:240` 生成；无其它生成点。台账分诊结论（1 注册、生成 SGshrouden.lua:240、AnimState＋persists=false＋inspectable("PUDDLE")）复核属实；补充实核：眼睛无 `FX` 标签、goop 仅客户端、六个死亡状态与 `final_blow` 的 `NOCLICK` 切换、名称覆写与 `PUDDLE` 描述键链。
- 名称核验：`languages/chinese_s.po` 定向检索——`STRINGS.NAMES.SHROUDEN`＝"暗影憎恶"（msgid "Umbral Abomination"，[L349359-L349362](../../../raw/current/scripts/languages/chinese_s.po#L349359-L349362)）；`SHROUDEN_ALLEGIANCE`＝"藏"、`SHROUDEN_OTHER`＝"异者"、`SHROUDEN_VOIDMASQUE`＝"暗域"（[L349364-L349377](../../../raw/current/scripts/languages/chinese_s.po#L349364-L349377)）；`shrouden_eye_fx` 无 `STRINGS.NAMES` 键。
- 分档说明：`size_class: standard`——单注册，但需承载工厂符号覆写、本地 goop 与网络同步、六个死亡状态与单槽清理、名称／检视两条链；无多形态生命周期，未升 complex。
- 未覆盖：暗影憎恶本体战斗与 `death_final` 触发条件全貌、`shrouden_optic_blast_fx` 光束细节、goop 动画包实际帧长、专服／晚加入玩家看到眼睛的时序；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/shadow_despawn|暗影消散、舞台影手与暗影憎恶光束]]（同为暗影憎恶表现的外围注册与光束页）
- 返回 [[reference/prefabs/fx-shadow-index|暗影生物、裂隙生物与附着]]
