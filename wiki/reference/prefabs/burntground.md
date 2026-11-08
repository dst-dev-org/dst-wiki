---
title: "焦地渐隐与蜂蜜拖尾减速地面 FX（burntground、honey_trail）"
created: "2026-11-05"
updated: "2026-11-08"
type: prefab
summary: "两个共享工厂的 4 个联网地面/贴底 FX：burntground.lua 的焦地两档（net_smallbyte 64 tick×5 天渐隐、可存读，月亮虹吸器破碎／流星雨落地／岩地房间世界生成三处生成）与 honey_trail.lua 的蜂蜜拖尾两档（每帧 PushTempGroundSpeedMultiplier 0.4 减速的混合载体，蜂王／蜂蜜弹弓弹／暗影憎恶三处生成，含 registered tags 检索变体）。"
aliases: ["burntground", "burntground_faded", "honey_trail", "honey_trail_shrouden_fx", "焦地", "焦痕渐隐", "蜂蜜拖尾", "蜂蜜减速", "蜂王蜂蜜", "蜂蜜弹弓", "暗影憎恶凝胶拖尾"]
tags: [prefab, fx, server-client]
size_class: standard
dst_build: "steam-buildid:26474873"
sources: ["prefabs/burntground.lua", "prefabs/honey_trail.lua", "prefabs/beequeen.lua", "prefabs/slingshotammo.lua", "prefabs/shrouden.lua", "prefabs/moon_device.lua", "prefabs/shadowmeteor.lua", "components/meteorshower.lua", "map/rooms/forest/terrain_rocky.lua", "components/locomotor.lua", "worldtiledefs.lua", "prefabs/ghostly_elixirs.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 焦地渐隐与蜂蜜拖尾减速地面 FX（burntground、honey_trail）

## 快速判断

- 职责与适用：两个共享工厂、互不相干宿主的 **4 个联网地面 FX**。①`burntground.lua` 的焦地两档：`burntground`（全新焦痕，α≈0.985 起）与 `burntground_faded`（已褪 20 tick，α≈0.677 起）——`net_smallbyte` 驱动的 64 tick×5 天渐隐，**可存读**；②`honey_trail.lua` 的蜂蜜拖尾两档：`honey_trail`（蜂王／蜂蜜弹弓弹的黏蜜地面）与 `honey_trail_shrouden_fx`（暗影憎恶的凝胶拖尾变体，`light_override 1`、registered tags 检索）——**每帧给半径内 locomotor 实体推 0.4 倍速乘数**的混合载体（[burntground 注册](../../../raw/current/scripts/prefabs/burntground.lua#L93-L94)、[honey 注册](../../../raw/current/scripts/prefabs/honey_trail.lua#L132-L142)）。
- 重要副作用：焦地自身零玩法（纯渐隐贴片）；蜂蜜拖尾**是玩法载体**——减速由 FX 实体的周期任务直接调 `locomotor:PushTempGroundSpeedMultiplier(0.4, WORLD_TILES.MUD)` 在服务端结算，客户端另有本机预测同步（[OnUpdateServer](../../../raw/current/scripts/prefabs/honey_trail.lua#L5-L16)、[OnUpdateClient](../../../raw/current/scripts/prefabs/honey_trail.lua#L21-L29)）。
- 修改边界：只覆盖 4 注册的工厂、生成点、渐隐／减速链、端模型与清理；蜂王战斗、弹弓弹药、暗影憎恶凝胶系统、月亮虹吸器与流星雨事件只辨析生成入口不展开。同类 anim 复用者（独眼鹿鹿激光／天体守卫激光／融合暗影弹的焦痕，均用 `burntground` 动画包但不注册本页实体）见辨析表。
- 名称：`BURNTGROUND`／`BURNTGROUND_FADED`／`HONEY_TRAIL`／`HONEY_TRAIL_SHROUDEN_FX` 在简中 `chinese_s.po` 均无 `STRINGS.NAMES.*` 键（0 命中），标题中文为已核验效果描述非官方译名；链上宿主官方名 `MOON_DEVICE`＝"月亮虹吸器"、`SHROUDEN`＝"暗影憎恶"、`BEEQUEEN`＝"蜂王"（锚点见[验证记录](#验证记录)）。

## 注册与构造

### 焦地（burntground.lua 2 注册）

| 注册名 | 初值 fade | 差异与证据 |
|---|---|---|
| `burntground` | 0 | 全新焦痕：α=(64-0)/65≈0.985 起步，5 天内渐隐至消失。[注册](../../../raw/current/scripts/prefabs/burntground.lua#L93) |
| `burntground_faded` | 20 | 已褪焦痕：α=(64-20)/65≈0.677 起步，剩余约 44 tick（≈3.4 天）后消失——世界生成岩地房间专用档。[注册](../../../raw/current/scripts/prefabs/burntground.lua#L94) |

工厂共性（`makeburntground(name, initial_fade)`，[工厂](../../../raw/current/scripts/prefabs/burntground.lua#L45-L91)）：Transform／AnimState／Network（无 SoundEmitter）→ bank=build `burntground` 播 `idle` → `ANIM_ORIENTATION.OnGround`＋`LAYER_GROUND`＋SortOrder 3 → 标签 `NOCLICK`／`FX` → `_fade` net_smallbyte（事件 `fadedirty`，[L63](../../../raw/current/scripts/prefabs/burntground.lua#L63)）→ **两变体都 `SetPrefabName("burntground")`**（[L65](../../../raw/current/scripts/prefabs/burntground.lua#L65)）→ SetPristine → 客户端：`DoPeriodicTask(FADE_INTERVAL, UpdateFade, math.random())`＋`fadedirty` 监听＋`set_local` 初值后早退（[L69-L76](../../../raw/current/scripts/prefabs/burntground.lua#L69-L76)）→ 主模拟：同周期任务（初始延迟 `FADE_INTERVAL - random()`）、`_fade:set` 写初值、随机朝向 0–360°、挂 OnSave/OnLoad；周期推进在两端都用 `set_local`（[L78-L87](../../../raw/current/scripts/prefabs/burntground.lua#L78-L87)）。

渐隐数学：`FADE_INTERVAL = TUNING.TOTAL_DAY_TIME * 5 / 64`＝37.5 秒（一天 480 秒），`UpdateFade` 每跳 `_fade` +1 至 63 后主模拟 Remove／客户端置 α0（[UpdateFade](../../../raw/current/scripts/prefabs/burntground.lua#L13-L22)）；α=(64-fade)/65（[OnFadeDirty](../../../raw/current/scripts/prefabs/burntground.lua#L8-L11)）。0→63→移除共 64 跳×37.5 秒＝**5 游戏日**（源码注释 "64 ticks for smallbyte"）。`persists` 未关闭：fade／rotation／scale 都入存档（[OnSave](../../../raw/current/scripts/prefabs/burntground.lua#L24-L28)、[OnLoad](../../../raw/current/scripts/prefabs/burntground.lua#L30-L43)）。

### 蜂蜜拖尾（honey_trail.lua 2 注册）

| 注册名 | build | 排除标签 | 差异与证据 |
|---|---|---|---|
| `honey_trail` | `honey_trail` | 服务端＋客户端：`honey_ammo_afflicted`、`vigorbuff` | 蜂王黏蜜与蜂蜜弹弓弹拖尾；7 种 `trail1..7` 形变，Recent-4 去重由生成方维护。[注册](../../../raw/current/scripts/prefabs/honey_trail.lua#L132-L136) |
| `honey_trail_shrouden_fx` | `honey_trail_shrouden`（额外 anim 包） | 服务端＋客户端：`gelblobbed` | 暗影憎恶变体：`light_override 1`（[L141](../../../raw/current/scripts/prefabs/honey_trail.lua#L141)）、`OverrideSearchParams(REGISTERED_GEL_TAGS, filterfn)` 走引擎注册检索（[调用](../../../raw/current/scripts/prefabs/shrouden.lua#L164-L167)）、拖尾持续 24–32 秒（普通档 1.2–4.5 秒） |

工厂共性（`MakeFx(name, data)`，[工厂](../../../raw/current/scripts/prefabs/honey_trail.lua#L68-L130)）：Transform／AnimState／SoundEmitter／Network＋标签 `FX` → bank `honey_trail`／build 按档 → `LAYER_BACKGROUND`＋SortOrder 3 → `_isfading` net_bool（事件 `isfadingdirty`，[L109](../../../raw/current/scripts/prefabs/honey_trail.lua#L109)）→ SetPristine → 客户端：监听 `isfadingdirty`（淡出即停本机减速）＋`DoPeriodicTask(0, OnInit, nil, 客户端排除表)`（[L113-L117](../../../raw/current/scripts/prefabs/honey_trail.lua#L113-L117)）→ 主模拟：`persists=false`＋默认 `DoTaskInTime(0, Remove)` 保底，暴露 `SetVariation`／`OverrideSearchParams`（[L120-L126](../../../raw/current/scripts/prefabs/honey_trail.lua#L120-L126)）。

生命周期：生成方 `SetVariation(rand, scale, duration)`（[L77-L89](../../../raw/current/scripts/prefabs/honey_trail.lua#L77-L89)，仅首次生效）——缩放既是视觉尺寸**也是减速半径**；播 `trail<rand>_pre`＋`honey_drip` 声 → `OnInit` 取消保底删除任务，取一次当前坐标与 scale 作为周期任务的固定参数并启动 0 周期任务（[L52-L61](../../../raw/current/scripts/prefabs/honey_trail.lua#L52-L61)）→ `animover` 进循环段并排定 `duration` 后 `OnStartFade`（播 `_pst`、`_isfading:set(true)`、停减速）→ `_pst` 播完 `animover` 自删（[OnAnimOver](../../../raw/current/scripts/prefabs/honey_trail.lua#L43-L50)）。`SetVariation` 未被调用则 0 tick 即自删。

## 关联机制导航

### 生成点与宿主调用

| 注册 | 生成语句（file:line） | 消费文件与条件 |
|---|---|---|
| `burntground` | [moon_device.lua#L284](../../../raw/current/scripts/prefabs/moon_device.lua#L284)（spawnscorchmark）；[shadowmeteor.lua#L68](../../../raw/current/scripts/prefabs/shadowmeteor.lua#L68) | ①月亮虹吸器破碎召唤天体守卫者：本体破碎时**中央 1 枚（scale 1.6）＋环形 3 枚（scale 1.1–1.5）**（[stage1_break](../../../raw/current/scripts/prefabs/moon_device.lua#L289-L304)）；②流星雨（meteorspawner 经 [meteorshower.lua#L166](../../../raw/current/scripts/components/meteorshower.lua#L166) 生成 `shadowmeteor`）落地有效地面 1 枚，scale=size×1.3（0.91／1.3／1.69） |
| `burntground_faded` | [terrain_rocky.lua#L49](../../../raw/current/scripts/map/rooms/forest/terrain_rocky.lua#L49) | **世界生成**：Rocky 岩地房间 `countprefabs` 随机 3–5 枚；运行时无生成点 |
| `honey_trail` | [beequeen.lua#L87](../../../raw/current/scripts/prefabs/beequeen.lua#L87)；[slingshotammo.lua#L207](../../../raw/current/scripts/prefabs/slingshotammo.lua#L207) | ①蜂王 `DoHoneyTrail` 每 0.2 秒按移动档落蜜：非移动（threshold 8）scale 0.5–0.8、duration 基数 1.2 秒；moving 且 `walkspeed ≤ BEEQUEEN_SPEED`（threshold 2）0.5–1.1、2 秒；更快（threshold 1）1–1.3、4 秒；生成时三档都再加 `[0,0.5)` 随机，落海改水花（[HONEY_LEVELS](../../../raw/current/scripts/prefabs/beequeen.lua#L54-L68)、[TrySpawnHoney](../../../raw/current/scripts/prefabs/beequeen.lua#L85-L89)、[StartHoney](../../../raw/current/scripts/prefabs/beequeen.lua#L119-L125)）；②蜂蜜弹弓弹（`slingshotammo_honey`）命中：目标挂 `honey_ammo_afflicted`＋直接 `SetExternalSpeedMultiplier(0.4)`，此后 8 秒每秒在目标处落蜜——`k=(1−elapsed/8)^2*0.6+0.3`，`min_scale=k`、`max_scale=k+0.2`，`duration=2+[0,0.5)` 随机（[OnUpdate_Honey](../../../raw/current/scripts/prefabs/slingshotammo.lua#L215-L220)、[TrySpawnHoney](../../../raw/current/scripts/prefabs/slingshotammo.lua#L204-L209)） |
| `honey_trail_shrouden_fx` | [shrouden.lua#L164](../../../raw/current/scripts/prefabs/shrouden.lua#L164) | 暗影憎恶凝胶系统 `StartGel` 的拖尾分支：0.2 秒周期；`speed<1` 不落蜜，`1≤speed<4` 取 `threshold=6`，`speed≥4` 取 `threshold=3`；scale 1.05–1.35、`duration=24+8*rand`；同时 `OverrideSearchParams(REGISTERED_GEL_TAGS, HoneyFilterFn)`（registered tags 检索＋过滤 flying/gelblobbed，[注册](../../../raw/current/scripts/prefabs/shrouden.lua#L234-L235)） |

### 减速结算归属（honey_trail 两档）

- **结算位置在 FX 实体的周期任务上**：`OnUpdateServer` 每帧对半径内带 `locomotor` 的实体调 `locomotor:PushTempGroundSpeedMultiplier(TUNING.BEEQUEEN_HONEYTRAIL_SPEED_PENALTY = 0.4, WORLD_TILES.MUD)`——0.4 乘数**替换**地面速度乘数，0.034 秒窗口靠每帧重推维持（[locomotor.lua](../../../raw/current/scripts/components/locomotor.lua#L638-L653)、[消费](../../../raw/current/scripts/components/locomotor.lua#L204)）；判定用的 `x/y/z/rad` 是 `OnInit` 时取一次的固定参数，不是每 tick 重读 FX 位置——模组后续移动／缩放 FX 不会自动更新该闭包。`WORLD_TILES.MUD` 第二参被脚步声系统读取——被减速实体脚步按泥地播放（[worldtiledefs.lua](../../../raw/current/scripts/worldtiledefs.lua#L102-L103)）。
- **双端分工**：服务端结算全部实体（权威），客户端 `OnUpdateClient` 只对本机玩家做预测同步（排除表各档不同，[L18-L29](../../../raw/current/scripts/prefabs/honey_trail.lua#L18-L29)）。半径＝SetVariation 的 scale 参数。
- **豁免语义**：普通档排除 `honey_ammo_afflicted`（已被蜂蜜弹直接减速的目标，避免叠加）与 `vigorbuff`（阿比盖尔"活力"精油的免疫标记，[ghostly_elixirs.lua](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L212)）；shrouden 档排除 `gelblobbed`（已凝胶目标）——它的检索集合是 `TheSim:RegisterFindTags({"locomotor"}, {...暗影系排除...})`，把暗影生物/Boss 一次性排除。
- 弹弓弹对命中的第一目标**另有**直接减速（`SetExternalSpeedMultiplier`，走 [[reference/components/locomotor|locomotor 组件]]的外部乘数表），与拖尾的区域减速是两套机制（[OnHit_Honey](../../../raw/current/scripts/prefabs/slingshotammo.lua#L232-L254)）。

### 焦地的存读与 faded 渐隐链

- 焦地是**可存档实体**：OnSave 记 fade（>0 时）、rotation、scale；OnLoad 恢复 rotation／scale（缺省 fallback 1/2/3）/fade（cap 63）并立即刷 α（[OnLoad](../../../raw/current/scripts/prefabs/burntground.lua#L30-L43)）。主模拟首个周期延迟略短于间隔（`FADE_INTERVAL - random()`），客户端初始延迟纯随机——两端 tick 相位不同步；周期推进由两端各自 `_fade:set_local` 完成，`set` 只在构造／加载时写权威值，`fadedirty` 只接收这些赋值，不是每 tick 的服务端广播。
- `burntground_faded` 的"已褪 20 tick"是构造参数：世界生成直接按该档落地；运行时三条生成语句都用基础档 `burntground`。两档共享同一 bank/build/动画，仅 fade 初值不同。
- 与 [[reference/prefabs/scorchedground|地面焦痕、跟随换装与附着状态 FX 族]] 的辨析：`scorchedground` 是龙蝇竞技场**静态布局**贴片（10 种随机 idle、persists、无渐隐、无网络变量）；本页焦地是**运行时／房间散布**的渐隐贴片（单一 idle、netvar 同步、5 天消失）。同为"焦痕"语义但注册、存读与视觉周期都不同。

### 同 anim 复用者（不属本页覆盖）

| 实体 | 复用方式 | 已有页 |
|---|---|---|
| 独眼鹿鹿激光焦痕 | `deerclops_laser.lua` 用 `burntground` bank/build 造独立激光灼痕注册 | [[reference/prefabs/deerclops_laserscorch|独眼鹿鹿激光灼痕]] |
| 天体守卫激光焦痕 | `alterguardian_laser.lua` 同上 | [[reference/prefabs/alterguardian_laserscorch|天体守卫激光灼痕]] |
| 融合暗影弹焦痕 | `fused_shadeling_bomb.lua` 同上 | [[reference/prefabs/crabking_claw_shadow|地面贴片与投影阴影 FX 族]] |

## 典型接入

### 生成一块按天渐隐的焦痕贴片

- 前提与选择：主模拟生成；要"起始已褪"观感直接用 `burntground_faded`，或对 `burntground` 手动 `_fade:set(n)` 跳进度。贴地姿态与层级已内建，勿再调 Orientation。
- 生效与撤销：实体 5 天自然消失；要立即清除就 `Remove`（无专用 KillFX）。fade 初值在构造／加载时经 netvar 同步，周期推进由各端本地 `set_local` 完成，不是每 tick 同步。

```lua
local function SpawnFadingScorch(x, z, scale)
    local scorch = SpawnPrefab("burntground")
    scorch.Transform:SetPosition(x, 0, z)
    scorch.Transform:SetScale(scale, scale, scale)
    -- 可选：scorch._fade:set(20) 起始半褪（等价 burntground_faded 的初值）
    return scorch
end
```

- 调整与验证：原版对应 [spawnscorchmark](../../../raw/current/scripts/prefabs/moon_device.lua#L283-L287)；scale 同时是视觉缩放（焦地无减速语义）。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：蜂蜜拖尾只是视觉。** 两档都在服务端逐帧结算 0.4 倍速乘数并改脚步声瓦片；蜂王拖尾、蜂蜜弹、暗影憎恶凝胶是真实的走位控制手段。[结算段](#减速结算归属honeytrail-两档)
- **边界：拖尾的减速半径＝视觉缩放。** `SetVariation` 的 scale 一参两用；生成方各自传不同区间（蜂王 0.5–0.8／0.5–1.1／1–1.3、弹弓 0.3–1.1（min=k、max=k+0.2，k 从 0.9 降到 0.3）、憎恶 1.05–1.35），复用时注意半径随视觉大小变化。[SetVariation](../../../raw/current/scripts/prefabs/honey_trail.lua#L77-L89)
- **边界：`SetVariation` 是一次性初始化且失败即自删。** 忘调它会 0 tick 被 `DoTaskInTime(0, Remove)` 收走；重复调用被 `trailname == nil` 守卫忽略。位置必须先设——原版两处调用都注释了 "must be before SetVariation"（[beequeen.lua#L86](../../../raw/current/scripts/prefabs/beequeen.lua#L86)）。
- **边界：`burntground` 两档都可存档，别当成纯 FX。** `persists` 未关闭且带 OnSave/OnLoad——模组批量生成焦地会写进存档；临时演出贴片建议用后自删（原版靠 5 天计时自然消失）。[存读](../../../raw/current/scripts/prefabs/burntground.lua#L24-L43)
- **边界：`burntground_faded` 只在世界生成出现。** 运行时无生成点；想要"半褪"观感须自己 set fade 或换基础档。[反查记录](#验证记录)
- **边界：焦地与 scorchedground 不互通。** `burntground` 的 fade 是 netvar 同步＋定时器推进，读档恢复进度；`scorchedground` 的 anim/rotation 是构造随机＋OnSave 覆盖，无渐隐。混用两者语义会得到错误的存读预期。[[reference/prefabs/scorchedground|对照页]]
- **边界：`honey_trail_shrouden_fx` 的检索走 registered tags。** `OverrideSearchParams` 换成 `FindEntities_Registered` 后排除表由 `RegisterFindTags` 一次性注册（含 shadow 系），直接照抄普通档的 cant 标签会漏掉这些豁免；`filterfn` 另排 flying/gelblobbed。[注册](../../../raw/current/scripts/prefabs/shrouden.lua#L234-L235)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/burntground.lua`、`prefabs/honey_trail.lua`；按需读取 `beequeen.lua` HONEY_LEVELS/TrySpawnHoney/StartHoney 段、`slingshotammo.lua` 蜂蜜弹 OnHit_Honey/OnUpdate_Honey/TrySpawnHoney 段、`shrouden.lua` StartGel/DoHoneyTrail/TrySpawnHoney 段、`moon_device.lua` spawnscorchmark/breaksequence 段、`shadowmeteor.lua` onexplode 与 sizes 表、`components/meteorshower.lua` 生成段、`map/rooms/forest/terrain_rocky.lua` Rocky 房间、`components/locomotor.lua` 速度乘数三函数、`worldtiledefs.lua` 脚步瓦片段、`ghostly_elixirs.lua` vigorbuff 标记、`tuning.lua` 调值（TOTAL_DAY_TIME=480、BEEQUEEN_HONEYTRAIL_SPEED_PENALTY=0.4、SLINGSHOT_AMMO_HONEY_DURATION=8）。全 `raw/current/scripts` 反查：`burntground` 命中注册（burntground.lua:93）、生成（moon_device.lua:284、shadowmeteor.lua:68）、prefablist.lua:189；`burntground_faded` 命中注册（L94）与 terrain_rocky.lua:49——两档共 **3 处生成语句，台账批 10 结论核实属实**；另命中 deerclops_laser/alterguardian_laser/fused_shadeling_bomb 三处 bank/build 复用（非本页注册，已列辨析表）与 customize.lua 图标引用。`honey_trail` 命中注册（honey_trail.lua:132）、prefablist.lua:565、beequeen.lua 依赖（L11）与生成（L87）、slingshotammo.lua 生成（L207）；`honey_trail_shrouden_fx` 命中注册（L137）、shrouden.lua 依赖（L19）与生成（L164）——生成点如上表、无其它。未运行游戏。
- 名称核验：`languages/chinese_s.po` 定向检索——`BURNTGROUND`／`HONEY_TRAIL`／`PLANT_DUG` 等覆盖注册名全 0 命中；链上宿主 `MOON_DEVICE`＝"月亮虹吸器"（[L345829-L345832](../../../raw/current/scripts/languages/chinese_s.po#L345829-L345832)）、`SHROUDEN`＝"暗影憎恶"（[L349359-L349362](../../../raw/current/scripts/languages/chinese_s.po#L349359-L349362)）、`BEEQUEEN`＝"蜂王"（[L339339-L339342](../../../raw/current/scripts/languages/chinese_s.po#L339339-L339342)）。标题中文使用已核验效果描述。
- 分档说明：`size_class: standard`——4 注册分 2 文件、各一条主链（渐隐 netvar 链／减速 locomotor 链），生成点表＋归属小节可完整承载；未选 complex，因两族均无多状态机，蜂蜜档的生命周期是单程 pre→loop→pst。
- 未覆盖：蜂王战斗与 HONEY_LEVELS 的阈值微调语义、暗影憎恶凝胶接触系统全貌（OnUpdateGel）、蜂蜜弹的仇恨与聚怪玩法、月亮虹吸器装配流程、流星雨刷怪规则、`burntground` OnLoad scale 缺省 1/2/3 的历史成因（按代码如实记录）、动画包帧长与观感；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

- 第 15 轮补正（2026-11-07）：蜂蜜减速用初始化坐标与半径、焦地两端 `set_local`、蜂王三档尺寸与弹弓 0.3～1.1；同步维护日期。

## 相关链接

- [[reference/prefabs/scorchedground|地面焦痕、跟随换装与附着状态 FX 族]]（世界生成静态焦痕对照；无渐隐、无 netvar）
- [[reference/prefabs/deerclops_laserscorch|独眼鹿鹿激光灼痕]]／[[reference/prefabs/alterguardian_laserscorch|天体守卫激光灼痕]]（同 `burntground` 动画包的独立注册）
- [[reference/prefabs/moon_device_pillar|月装置支柱、顶盖、一阶段破碎与裂纹层]]（月亮虹吸器破碎演出的同链实体）
- [[reference/prefabs/eye_charge|有表现的玩法载体（眼弹、诱鱼区、黏块、速爆弹）]]（`gelblobbed` 凝胶语义的宿主家族）
- [[reference/components/locomotor|locomotor 组件]]（`PushTempGroundSpeedMultiplier` 与外部速度乘数两套减速的宿主协议）
- 返回 [[reference/prefabs/fx-world-index|天气、地形与生态痕迹]]
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-06）。
