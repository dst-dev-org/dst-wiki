---
title: "黏液飞溅与贴地溅落（goosplash 等 9 注册）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "同一 MakeSplash 工厂的 9 个纯 FX 注册：8 个定号变体与 1 个随机基名；网络壳只同步 flip 并在 1 秒后清理，非专服延迟创建溅花与贴地溅落两层本地动画。"
aliases: ["goosplash", "goosplash1", "goosplash2", "goosplash3", "goosplash4", "goosplash5", "goosplash6", "goosplash7", "goosplash8", "黏液飞溅", "黏液溅落", "黏液落点"]
tags: [prefab, fx, network-sync, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/goosplash.lua", "prefabs/lavaarena_peghook.lua", "prefablist.lua", "prefabs.lua", "mainfunctions.lua", "entityscript.lua", "netvars.lua", "gamelogic.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 黏液飞溅与贴地溅落（goosplash 等 9 注册）

## 快速判断

- 职责与适用：`prefabs/goosplash.lua` 用一个 `MakeSplash` 工厂注册 9 个纯 FX：8 个定号变体 `goosplash1`～`goosplash8` 与 1 个随机基名 `goosplash`。每个注册是“不带动画的联网壳＋两层非联网本地动画”：`gooball_fx` 的溅花层与 `lavaarena_hits_splash` 的贴地溅落基底；不挂组件、不结算伤害、无音效。[工厂与注册](../../../raw/current/scripts/prefabs/goosplash.lua#L76-L129)
- 优先使用：需要命中点的两层黏液溅落时按外观选变体（1–4 用溅花 `player_hit_5`、5–8 用 `player_hit_6`，基底依次 `player_hit_1`～`player_hit_4`），或生成基名让工厂随机八选一；只取单层落点贴片可对照 [[reference/prefabs/impact|impact（命中与落点表现）]]。它不负责命中判定、伤害或持续跟随。[映射](../../../raw/current/scripts/prefabs/goosplash.lua#L65-L74)
- 重要副作用：联网壳只有 Transform／Network 与 `FX` 标签，没有 AnimState、音效、物理或 `NOCLICK`；服务器写 3-bit `flip`（1–4），1 秒后 `Remove`。可见动画只在非专服生成：基底在 0.15–0.2 秒后出现，溅花再晚 0.1 秒且抬高 0.7；专用服务器没有任何动画。[壳](../../../raw/current/scripts/prefabs/goosplash.lua#L78-L112)、[netvar](../../../raw/current/scripts/netvars.lua#L7)
- 修改边界：本页覆盖 9 个注册、端分界、两层动画、父挂与清理。当前 `raw/current/scripts` 全量反查没有 `SpawnPrefab("goosplash…")`；唯一外部依赖引用是 `lavaarena_peghook.lua` 的依赖声明（`prefablist.lua` 只收录文件名），依赖的消费代码可能位于快照之外的事件数据里，见[生成缺口与依赖](#生成缺口与依赖)。动画帧长、资源包内容与客户端观感未运行核验。

## 注册与构造

`MakeSplash(name, num, prefabs)` 返回 `Prefab(name, fn, assets, prefabs)`：`assets` 固定两个 ANIM，第四参作为 `deps` 交给 `Prefab` 构造器并由 `RegisterPrefabsImpl` 转交引擎；8 个变体传固定 `num`，基名传 `nil` 与 8 个变体名组成的 `prefs`。注册循环生成变体后追加基名；`num` 到两层动画的换算见 `PlaySplashAnim`。[工厂](../../../raw/current/scripts/prefabs/goosplash.lua#L76-L129)、[循环](../../../raw/current/scripts/prefabs/goosplash.lua#L118-L124)、[映射](../../../raw/current/scripts/prefabs/goosplash.lua#L65-L74)、[Prefab 构造器](../../../raw/current/scripts/prefabs.lua#L6-L12)、[注册转交](../../../raw/current/scripts/mainfunctions.lua#L129-L143)

| 注册名 | `num` 与两层动画 |
|---|---|
| `goosplash1` | `n=1`；溅花 `player_hit_5`＋基底 `player_hit_1` |
| `goosplash2` | `n=2`；溅花 5＋基底 2 |
| `goosplash3` | `n=3`；溅花 5＋基底 3 |
| `goosplash4` | `n=4`；溅花 5＋基底 4 |
| `goosplash5` | `n=5`；溅花 `player_hit_6`＋基底 1 |
| `goosplash6` | `n=6`；溅花 6＋基底 2 |
| `goosplash7` | `n=7`；溅花 6＋基底 3 |
| `goosplash8` | `n=8`；溅花 6＋基底 4 |
| `goosplash` | `nil` → `math.random(8)`；外观随 `n`；工厂把实体 prefab 名改成 `goosplash<n>`，再把显示名 override 回 `goosplash`[L96-L99](../../../raw/current/scripts/prefabs/goosplash.lua#L96-L99) |

名称核验：`languages/chinese_s.po` 以 `GOOSPLASH` 与 8 个变体名定向检索均无 `STRINGS.NAMES` 键；标题与表内中文是按资源名和动画核验的效果描述，不是官方译名。`SetPrefabNameOverride("goosplash")` 只影响显示名解析。

### 共同工厂与端分界

- 网络壳：Transform＋Network＋`FX` → 取 `n` → 非专服安排 `PlaySplashAnim` → 声明 `flip` netvar → 基名改名 → `SetPrefabNameOverride` → `SetPristine` → 客户端早退 → 主模拟写 `flip`、`persists=false`、1 秒后 `Remove`。[L78-L112](../../../raw/current/scripts/prefabs/goosplash.lua#L78-L112)
- 端：主机既执行服务器段，也在 0.15–0.2 秒后建本地两层；客户端只保留壳并建本地两层；专用服务器只保留壳，不建动画。[非专服分支](../../../raw/current/scripts/prefabs/goosplash.lua#L88-L92)、[客户端早退](../../../raw/current/scripts/prefabs/goosplash.lua#L103-L105)
- 名字：`SetPrefabName` 同时写 `self.prefab`、引擎 prefab 名与 `self.name`；`SetPrefabNameOverride` 只设 `nameoverride` 供显示名解析。基名生成后实体 prefab 可能已变成某个变体号。[名称函数](../../../raw/current/scripts/entityscript.lua#L840-L850)
- 依赖：基名第四参是 8 个变体名，变体自身没有 deps；`deps` 只参与 `TheSim:RegisterPrefab` 的加载依赖，不等于生成或运行时选名。[构造器](../../../raw/current/scripts/prefabs.lua#L6-L12)、[注册转交](../../../raw/current/scripts/mainfunctions.lua#L142)

### 两层本地动画

`PlaySplashAnim(proxy, num)` 先算 `variation = math.ceil(num / 4)`，再建溅花层并把世界坐标抬高 0.7，最后建贴地基底层；两层都只用 `SetFromProxy(proxy.GUID)` 取壳的创建时变换，没有 `SetParent`，也不随壳后续移动（复制语义见 [[reference/mechanisms/generic-fx-factory#常见陷阱与限制|通用 FX 工厂]]）。[L65-L74](../../../raw/current/scripts/prefabs/goosplash.lua#L65-L74)

- 溅花层：无 Network、`SetCanSleep(false)`、`persists=false`；bank／build `gooball_fx`，染色 (.2, 1, 0, 1)，`SetFinalOffset(3)`；延迟 0.1 秒播 `player_hit_5/6`；`animover` 自删；翻转时 `SetScale(-1, 1)`。[L15-L38](../../../raw/current/scripts/prefabs/goosplash.lua#L15-L38)
- 基底层：同样无 Network、不可睡眠、不持久；bank／build `lavaarena_hits_splash`，播 `player_hit_1..4`、同色；`OnGround`＋`LAYER_BACKGROUND`＋`SetSortOrder(3)`；水平缩放 ±1/3、垂直 1/3；`animover` 自删。[L40-L63](../../../raw/current/scripts/prefabs/goosplash.lua#L40-L63)

`flip` 的镜像不对称：溅花只在 2／4 翻转，基底只在大于 2（3／4）时翻转。[L68](../../../raw/current/scripts/prefabs/goosplash.lua#L68)、[L73](../../../raw/current/scripts/prefabs/goosplash.lua#L73)

| `flip` | 溅花镜像 | 基底层镜像 |
|---:|---|---|
| 1 | 否 | 否 |
| 2 | 是 | 否 |
| 3 | 否 | 是 |
| 4 | 是 | 是 |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 网络壳＋同步 | 壳仅 Transform／Network／`FX`；`goosplash.flip` 用 `net_tinybyte`（3-bit，0–7），服务器写 `math.random(4)` | 各端构造；服务器关闭持久化并 1 秒后自删；客户端只读 netvar，不等待 dirty | [L94](../../../raw/current/scripts/prefabs/goosplash.lua#L94)、[L107-L110](../../../raw/current/scripts/prefabs/goosplash.lua#L107-L110)、[netvars](../../../raw/current/scripts/netvars.lua#L7) |
| 本地两层 | 溅花 `gooball_fx`＋基底 `lavaarena_hits_splash`，各自 `animover`，无父挂 | 仅非专服；创建时取壳变换；细节见[两层本地动画](#两层本地动画) | [L15-L74](../../../raw/current/scripts/prefabs/goosplash.lua#L15-L74) |
| 资源 | `anim/lavaarena_hits_splash.zip`、`anim/gooball_fx.zip`，均在 assets 声明 | 声明即核验范围；包内容与帧长未解包 | [L1-L5](../../../raw/current/scripts/prefabs/goosplash.lua#L1-L5) |
| 关联 Prefab：`peghook_splashfx` | 同文件的独立注册，也用 `lavaarena_hits_splash`，但播 `aoe_hit`、缩放 .54，工厂与本族不同 | 本快照无直接 `SpawnPrefab`，生成可能同样落在事件数据 | [注册](../../../raw/current/scripts/prefabs/lavaarena_peghook.lua#L313-L318)、[工厂](../../../raw/current/scripts/prefabs/lavaarena_peghook.lua#L249-L279) |
| 关联 Prefab：`peghook_hitfx` | 第四参依赖表含 `goosplash` 与 `peghook_splashfx`；主模拟收尾调 `event_server_data("lavaarena", ...).hitfx_postinit` | 依赖不是生成；postinit 文件不在 `scripts` 快照 | [依赖表](../../../raw/current/scripts/prefabs/lavaarena_peghook.lua#L32-L36)、[收尾](../../../raw/current/scripts/prefabs/lavaarena_peghook.lua#L242) |
| 加载：`PREFABFILES` | `prefablist.lua` 收录 `goosplash`；`gamelogic.lua` 分批 `LoadPrefabFile("prefabs/" .. file)` | 只证明文件被读取，不证明被生成 | [清单](../../../raw/current/scripts/prefablist.lua#L517)、[加载](../../../raw/current/scripts/gamelogic.lua#L203-L204) |

### 生成缺口与依赖

- 检索范围：`raw/current/scripts` 全量 `grep -rn "goosplash"` 仅 7 行、3 文件——本文件 5 行、`prefablist.lua:517`、`lavaarena_peghook.lua:35`。
- 结论：当前快照的脚本层没有任何 `SpawnPrefab("goosplash")`、按变体名的生成或 `"goosplash"..` 动态拼名；唯一外部依赖引用是 `peghook_hitfx` 的依赖表，随 `Prefab("peghook_hitfx", ...)` 注册，是加载依赖而非生成语句。
- 未覆盖：`peghook_hitfx` 的主模拟收尾在 `event_server_data("lavaarena", "prefabs/lavaarena_peghook").hitfx_postinit(inst)`，事件数据位于 `lavaarena_event_server/...`，不在 `raw/current/scripts` 快照内；不能排除该 postinit 生成 `goosplash`。`prefablist.lua` 的收录与 `PREFABFILES` 循环只说明文件会加载。按诚实缺口记录：脚本层无生成点、事件数据未覆盖，不判死代码、不判 blocked。

## 典型接入

### 在自定义命中点手动播放一次黏液溅落（原版未接线）

- 前提与选择：需要在自定义黏液命中点复刻两层溅落，且能接受“服务器生成壳、各客户端本地演出”的端模型；定号变体固定外观，基名随机。原版当前快照没有调用点，下面是按工厂契约手写的接入，不是原版现有案例。
- 生效与撤销：只在主模拟 `SpawnPrefab`；调用方要在 0.15–0.2 秒的本地动画创建前写好位置，否则会落在原点。壳 1 秒后自删，两层本地动画各自 `animover` 回收，不需要手动 `Remove`。伤害、命中与音效由调用方另做。
- 原版依据：[共同工厂](../../../raw/current/scripts/prefabs/goosplash.lua#L78-L112)、[两层动画](../../../raw/current/scripts/prefabs/goosplash.lua#L65-L74)。

```lua
local function PlayGooSplash(x, y, z)
    local fx = SpawnPrefab("goosplash5")
    if fx ~= nil then
        fx.Transform:SetPosition(x, y, z)
    end
end
```

- 调整与验证：把 `goosplash5` 换成 1–8 中任一变体即可固定外观，换成 `goosplash` 则随机；`flip` 由服务器随机写入，不要由调用方设置。示例只做语法检查，未在游戏内执行；专用服务器不会出现动画。

## 常见陷阱与限制

- **误区：`goosplash` 在当前脚本里有生成点**：全量反查没有 `SpawnPrefab("goosplash…")`，唯一引用是 `peghook_hitfx` 的依赖声明；依赖消费代码可能在快照外的事件数据里。不要把“被依赖”写成“被生成”。[缺口](#生成缺口与依赖)
- **边界：可见动画只在非专服**：专用服务器只保留无 AnimState 的联网壳，1 秒后删除；本地两层只靠 `animover` 回收，没有定时兜底，若动画缺失、被打断或长于 1 秒，壳删除不会清理本地层。动画包内容与长动画表现未运行核验。[非专服](../../../raw/current/scripts/prefabs/goosplash.lua#L88-L92)、[壳清理](../../../raw/current/scripts/prefabs/goosplash.lua#L109-L110)
- **边界：没有 `NOCLICK` 或物理体**：9 个注册只挂 `FX`，壳没有 Physics、AnimState、SoundEmitter；不要当可点击目标、阻挡物或声音源，也不要把“没有 NOCLICK”反推为可交互。[壳标签](../../../raw/current/scripts/prefabs/goosplash.lua#L80-L83)
- **边界：`flip` 是不对称镜像且客户端不等待同步**：溅花只在 2／4 翻转，基底只在 3／4 翻转；客户端在 0.15–0.2 秒后直接读 `value()`，工厂没有监听 dirty。晚同步时的实际取值与视觉效果未运行核验。[镜像](../../../raw/current/scripts/prefabs/goosplash.lua#L68-L73)、[读取](../../../raw/current/scripts/prefabs/goosplash.lua#L88-L94)
- **边界：基名会改实体 prefab 名**：基名工厂先 `SetPrefabName("goosplash"..n)`，再 `SetPrefabNameOverride("goosplash")`；前者改 `inst.prefab`／`inst.name`，后者只改显示名解析。生成基名后不要依赖 `inst.prefab` 仍等于 `goosplash`。[改名](../../../raw/current/scripts/prefabs/goosplash.lua#L96-L99)、[名称函数](../../../raw/current/scripts/entityscript.lua#L840-L850)
- **边界：动画名与 build 的对应未核验**：溅花层用 `gooball_fx` 播 `player_hit_5/6`，基底用 `lavaarena_hits_splash` 播 `player_hit_1..4`；`player_hit_5/6` 只在本文件出现，快照不含 anim 包，不能据名字断定资源内一定有该动画。[溅花](../../../raw/current/scripts/prefabs/goosplash.lua#L26-L35)、[基底](../../../raw/current/scripts/prefabs/goosplash.lua#L51-L53)

## 验证记录

- 源码审阅：2026-11-04，steam-buildid:26335394（`raw/current/BUILD.txt`）。逐项核验 9 个注册的工厂、闭包 `num`、端分界、资源、动画名、父挂与清理；核验 `Prefab` 第四参到 `deps` 的传递、`SetPrefabName`／`SetPrefabNameOverride` 的职责、`net_tinybyte` 的 3-bit 范围，以及 `prefablist.lua`／`gamelogic.lua` 的加载链。全 `raw/current/scripts` 反查 `goosplash` 仅 7 行、3 文件，无任何 `SpawnPrefab("goosplash…")`；`lavaarena_peghook.lua` 的依赖声明与 `event_server_data` 收尾已区分。`languages/chinese_s.po` 定向检索无 `GOOSPLASH`／8 个变体名的 `STRINGS.NAMES` 键，未使用官方译名。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。
- 结构检查：`validate_page.py` 通过（0 issue；118 行、standard 档内）；`validate_source_links.py --strict` 通过（0 error／0 warning）；Lua 片段 `parser_probe.exe --syntax-only` 通过（只查语法、未执行）。
- 未覆盖与分档：动画包实际内容与帧长、客户端实际演出、晚加入／专服观感、`flip` 晚同步、动画缺失或被打断时的本地层清理、熔炉事件数据中的生成点、其他模组调用。9 个注册共用单一工厂，无组件、状态切换或多阶段生命周期，按默认 `standard` 档成页，未升 `complex`。

返回 [[reference/prefabs/fx-combat-index|攻击、弹道与落点]]；落点类对照 [[reference/prefabs/impact|impact（命中与落点表现）]]。
