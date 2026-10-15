---
title: "梦魇鞍具的跟随特效与暗影脚印（saddle_shadow_fx、saddle_shadow_footprint）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "梦魇鞍具装备后的网络代理 FX：服务端同步归属与颜色，各端用 3 枚本地帧跟随 swap_saddle；骑乘时按 3.5 距离门限生成并池化 4 脚印地面贴片，卸下／换主／移除各有清理。"
aliases: ["saddle_shadow_fx", "saddle_shadow_footprint", "梦魇鞍具特效", "影鞍特效", "暗影脚印", "鞍具脚印", "外围 FX 族"]
tags: [prefab, fx, server-client]
size_class: complex
dst_build: "steam-buildid:26335394"
sources: ["prefabs/saddle.lua", "prefabs/saddle_shadow_footprints.lua", "components/rideable.lua", "components/rider.lua", "components/colouradder.lua", "components/colouraddersync.lua", "components/updatelooper.lua", "components/highlightchild.lua", "mathutil.lua", "netvars.lua", "prefablist.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 梦魇鞍具的跟随特效与暗影脚印（saddle_shadow_fx、saddle_shadow_footprint）

## 快速判断

- 职责与适用：梦魇鞍具（`saddle_shadow`）的两个纯表现注册。`saddle_shadow_fx` 是网络代理壳：服务端只维护父挂、颜色子级与 `ownerevent`，可见的 3 枚帧由各端本地创建并跟随 `swap_saddle`；`saddle_shadow_footprint` 是骑乘时按 3.5 距离门限落在世界坐标的 4 脚印贴片，`animover` 后回收到 `owner.footprint_pool` 复用。与 [[reference/prefabs/gestalt_trail|gestalt_trail]] 同为宿主侧生成／跟随／自清理，本页多“网络壳＋本地帧”与池化两层。
- 优先使用：三段改主（装备→坐骑、上鞍→骑手、下鞍→坐骑）看 `CreateFxFollowFrame`／`SpawnFxForOwner`；按移动距离落贴片并复用看 `RestartFX`／`OnAnimOver`；调用现成效果按“生成点与宿主”直达。
- 重要副作用：换主会重排帧并清空脚印池；卸下／重装 `Remove` 旧 FX；帧登记进宿主 `highlightchildren`；颜色由宿主 `colouradder` → `colouraddersync` → 各端帧 `SetAddColour`。
- 修改边界：只覆盖两个注册的工厂、端、生成点、父挂／跟随与清理；鞍的配方、加成、耐久、熔炉修复与上下鞍状态机不在本页；跨端时序与动画包未运行核验。

## 注册与构造

两个注册分居两个文件：`saddle.lua` 的返回同时注册 5 种鞍物品与 `saddle_shadow_fx`（[L406-L412](../../../raw/current/scripts/prefabs/saddle.lua#L406-L412)），脚印文件单独返回 `Prefab`（[L101](../../../raw/current/scripts/prefabs/saddle_shadow_footprints.lua#L101)）。`prefablist.lua` 的 `saddle_shadow_footprints` 是文件名条目（`PREFABFILES`），不是注册名（[L1077-L1078](../../../raw/current/scripts/prefablist.lua#L1077-L1078)）。

| 注册名／宿主文件 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `saddle_shadow_fx`（`prefabs/saddle.lua`） | `ShadowSaddleFx`：CreateEntity → Transform／Network → `FX` → `colouraddersync` → `net_event("saddle_shadow_fx.ownerevent")` → `SetPristine`；非主模拟只监听 `ownerevent` 后返回，主模拟导出 `SetOwner`、`persists=false`（[工厂 L214-L238](../../../raw/current/scripts/prefabs/saddle.lua#L214-L238)、[端分界 L228-L232](../../../raw/current/scripts/prefabs/saddle.lua#L228-L232)） | 壳无 AnimState；`SetOwner` 尝试解旧颜色子级（`DetachChild` 实参是帧表而非壳，实际不移除旧宿主登记，见陷阱）、`SetParent(owner.entity)`、挂新宿主 `colouradder` 子级、`ownerevent:push()`，非专用端再 `SpawnFxForOwner`（[L181-L204](../../../raw/current/scripts/prefabs/saddle.lua#L181-L204)）。本地帧：非网络实体，bank／build `saddle_shadow`、`swap_loop_1..3` 循环、`SetSymbolLightOverride("red", .5)`、`SetLightOverride(.1)`、`AddOverrideBuild("hat_voidcloth")`、`highlightchild`（[L57-L80](../../../raw/current/scripts/prefabs/saddle.lua#L57-L80)） |
| `saddle_shadow_footprint`（`prefabs/saddle_shadow_footprints.lua`） | `fn`：CreateEntity → `FX`＋`NOCLICK` → `SetCanSleep(false)`、`persists=false` → Transform／AnimState → bank／build `saddle_shadow_tracks`、`ground` → `OnGround`＋`LAYER_BACKGROUND`＋`SetSortOrder(2)` → 隐藏 4×3 offset 符号 → `updatelooper`＋`animover`（[L63-L99](../../../raw/current/scripts/prefabs/saddle_shadow_footprints.lua#L63-L99)） | 非网络贴片；`RestartFX` 重播 `ground`、随机每枚脚印的 offset（3 选 1）与 art（`swap_footprint1..4`）、重新登记逐帧淡出（[L43-L59](../../../raw/current/scripts/prefabs/saddle_shadow_footprints.lua#L43-L59)）；`OnAnimOver` 优先回池、无池则 `Remove`（[L27-L37](../../../raw/current/scripts/prefabs/saddle_shadow_footprints.lua#L27-L37)） |

名称：两个注册名在 `languages/chinese_s.po` 均无 `STRINGS.NAMES.*` 键；标题中的“跟随特效”“暗影脚印”是效果描述，不是官方译名。宿主物品 `saddle_shadow` 的简中名是 `STRINGS.NAMES.SADDLE_SHADOW`＝梦魇鞍具（[L348894-L348897](../../../raw/current/scripts/languages/chinese_s.po#L348894-L348897)），只用于定位宿主。`saddle_shadow_fx` 复用物品的 `saddle_shadow` 动画包，工厂自身无 assets 声明（[注册](../../../raw/current/scripts/prefabs/saddle.lua#L412)）。

## 关联机制导航

### 生成点与宿主

| 生成点 | 涉及注册 | 端、阶段与条件 | 证据 |
|---|---|---|---|
| `saddle_shadow` 的 `equipped` 事件 | `saddle_shadow_fx` | 主模拟；`Rideable:SetSaddle` 以 `{ owner = self.inst }` 推给鞍，接收方先删旧 `inst.fx`，再 `SpawnPrefab`＋`SetOwner(data.owner)`（owner 是坐骑） | [接收 L39-L46](../../../raw/current/scripts/prefabs/saddle.lua#L39-L46)、[推事件 L144](../../../raw/current/scripts/components/rideable.lua#L144) |
| `rider:Mount` | `saddle_shadow_fx`（改主） | 主模拟；鞍的 `saddler` 提供 `swap_saddle` 覆盖后，若 `rideable.saddle.fx` 存在则 `SetOwner(player)` | [L98-L108](../../../raw/current/scripts/components/rider.lua#L98-L108) |
| `rider:ActualDismount` | `saddle_shadow_fx`（改回） | 主模拟；`SetOwner(self.mount)`，回到坐骑 | [L199-L202](../../../raw/current/scripts/components/rider.lua#L199-L202) |
| `Rideable:OnLoadDomesticatable` | `saddle_shadow_fx`（重建） | 读档用 `SetSaddle(nil, SpawnSaveRecord(...))` 再走 `equipped`；FX 因 `persists=false` 不随存档保存 | [L226-L233](../../../raw/current/scripts/components/rideable.lua#L226-L233) |
| `ShadowSaddle_Fx_OnUpdate` | `saddle_shadow_footprint` | 本地表现侧；owner 带 `moving` 且距上一枚脚印 ≥3.5 世界单位时生成；坐骑为 owner 时循环已被移除 | [L104-L132](../../../raw/current/scripts/prefabs/saddle.lua#L104-L132) |
| `saddle_shadow` 的 `unequipped` 事件 | `saddle_shadow_fx`（移除） | 主模拟；`inst.fx:Remove()` 并置 `nil` | [L48-L53](../../../raw/current/scripts/prefabs/saddle.lua#L48-L53) |

### 挂接、跟随与清理

| 对象 | 父挂／跟随 | 清理路径 | 证据 |
|---|---|---|---|
| FX 壳 | 服务端 `SetParent(owner.entity)`；客户端按复制父挂；宿主有 `colouradder` 时作为其子级接收颜色同步 | 卸下／重装前宿主显式 `Remove`；`OnRemoveEntity` 删 3 帧与池；`persists=false`；专用服务器不建本地帧 | [父挂与颜色 L186-L196](../../../raw/current/scripts/prefabs/saddle.lua#L186-L196)、[专用分支 L200-L203](../../../raw/current/scripts/prefabs/saddle.lua#L200-L203)、[移除 L82-L94](../../../raw/current/scripts/prefabs/saddle.lua#L82-L94) |
| 3 枚本地帧 | `SetParent(owner.entity)`＋`FollowSymbol(owner.GUID, "swap_saddle", nil, nil, nil, true, nil, i - 1)`；非网络实体，不随存档 | 由壳的 `OnRemoveEntity` 删除；换主时复用有效帧并重挂；客户端换父后旧帧可能失效，故每次检查 `IsValid` | [重建 L134-L179](../../../raw/current/scripts/prefabs/saddle.lua#L134-L179) |
| 脚印贴片 | 无父挂；由 FX 写世界坐标与朝向；`owner` 指向 FX 壳 | `animover` 回收到 `owner.footprint_pool`（`RemoveFromScene`）；池被置 nil／删除或 owner 无池时才 `Remove`（空表仍可回收）；`persists=false` | [生成 L120-L130](../../../raw/current/scripts/prefabs/saddle.lua#L120-L130)、[回池 L27-L37](../../../raw/current/scripts/prefabs/saddle_shadow_footprints.lua#L27-L37) |

### 组件、资源与互链

| 类型／对象 | 本注册如何使用 | 条件与边界 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`colouraddersync`／`colouradder` | FX 壳挂 `colouraddersync`；宿主 `colouradder:AttachChild(inst)`，颜色经 net_uint 同步到客户端，再由回调写到 3 帧 `SetAddColour` | 宿主无 `colouradder` 时跳过；换主解绑见“常见陷阱” | [[reference/components/colouraddersync]]、[AttachChild L24-L39](../../../raw/current/scripts/components/colouradder.lua#L24-L39)、[SyncColour L36-L49](../../../raw/current/scripts/components/colouraddersync.lua#L36-L49)、[回调 L96-L100](../../../raw/current/scripts/prefabs/saddle.lua#L96-L100) |
| 组件：`updatelooper` | FX 壳在 owner 为玩家时挂上并注册 `ShadowSaddle_Fx_OnUpdate`；脚印挂同一组件登记逐帧淡出 | 坐骑为 owner 时移除；`RemoveOnUpdateFn` 延迟到下一帧 | [[reference/components/updatelooper]]、[启用 L161-L167](../../../raw/current/scripts/prefabs/saddle.lua#L161-L167)、[AddOnUpdateFn L24-L29](../../../raw/current/scripts/components/updatelooper.lua#L24-L29)、[脚印更新 L14-L25](../../../raw/current/scripts/prefabs/saddle_shadow_footprints.lua#L14-L25) |
| 组件：`highlightchild` | 每枚帧 `SetOwner(owner)`，登记到宿主 `highlightchildren`；换主自动改登记 | 专用服务器跳过 | [[reference/components/highlightchild]]、[登记 L155](../../../raw/current/scripts/prefabs/saddle.lua#L155)、[切换 L35-L55](../../../raw/current/scripts/components/highlightchild.lua#L35-L55) |
| `net_event` | `saddle_shadow_fx.ownerevent` 只做“归属已变”的一次性通知；客户端按 `GetParent()` 重建帧 | 主机分支不注册监听；晚加入是否收到首事件未运行核验 | [定义 L85-L99](../../../raw/current/scripts/netvars.lua#L85-L99)、[客户端回调 L206-L212](../../../raw/current/scripts/prefabs/saddle.lua#L206-L212) |
| Follower（引擎） | 本地帧用 `FollowSymbol` 贴合宿主 `swap_saddle`；末位 `i-1` 区分同符号的三份跟随者 | 符号由鞍的 `saddler:SetSwaps` 与骑手态覆盖提供 | [[reference/components/follower]]、[帧跟随 L153](../../../raw/current/scripts/prefabs/saddle.lua#L153) |
| 宿主组件 | `saddler` 提供 `swapbuild`／`swapsymbol`；`rideable` 推 equipped／unequipped；`rider` 在上／下鞍时改主 | 本页只覆盖 FX 侧调用 | [[reference/components/saddler]]、[[reference/components/rideable]]、[[reference/components/rider]] |
| 关联物品：`saddle_shadow` | FX 的生成宿主与资源来源；`data.shadow.prefabs` 声明两个 FX 依赖 | 依赖声明不生成；物品页未建（Wiki 待建） | [数据 L364-L403](../../../raw/current/scripts/prefabs/saddle.lua#L364-L403) |
| 资源 | 帧复用 `anim/saddle_shadow.zip`（物品资源）并 `AddOverrideBuild("hat_voidcloth")`；脚印用 `anim/saddle_shadow_tracks.zip` | 快照只含 `scripts/`，动画包实际内容／帧长未核验 | [物品资源 L257-L259](../../../raw/current/scripts/prefabs/saddle.lua#L257-L259)、[脚印资源 L1-L4](../../../raw/current/scripts/prefabs/saddle_shadow_footprints.lua#L1-L4) |

## 典型接入

### 复用梦魇鞍具的跟随层与改主时机

- 前提与选择：需要“装备后挂在坐骑上、骑上后跟随骑手、下鞍后回到坐骑”的三段切换时，复用 `saddle_shadow_fx` 的 `SetOwner`；原版由 `saddle_shadow` 的 equipped／unequipped 与 `rider` 的 `Mount`／`ActualDismount` 四处调用。
- 生效与撤销：`SetOwner` 只在主模拟暴露；同 owner 直接返回；换主会先解旧宿主颜色子级、重挂父实体、推 `ownerevent`，非专用端重建本地帧。卸下时宿主 `Remove` 并把字段置 `nil`。
- 原版依据：[装备接收](../../../raw/current/scripts/prefabs/saddle.lua#L39-L46)、[SetOwner](../../../raw/current/scripts/prefabs/saddle.lua#L181-L204)、[上鞍改主](../../../raw/current/scripts/components/rider.lua#L105-L107)、[下鞍改回](../../../raw/current/scripts/components/rider.lua#L200-L202)

```lua
local function attach_owner_fx(item, owner)
    if not TheWorld.ismastersim then
        return
    end
    if item.fx ~= nil then
        item.fx:Remove()
    end
    local fx = SpawnPrefab("saddle_shadow_fx")
    fx:SetOwner(owner)
    item.fx = fx
end
```

- 调整与验证：`owner` 传目标实体（坐骑或骑手），同一 owner 重复调用直接返回；换主会触发帧重建，不要每帧调用。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：FX 壳本身不可见。** 网络实体只有 Transform／Network 与颜色同步，没有 AnimState；可见的是 3 枚本地帧。专用服务器不建这些帧，客户端由 `ownerevent` 触发，主机在 `SetOwner` 内直接创建。[工厂](../../../raw/current/scripts/prefabs/saddle.lua#L214-L238)、[专用分支](../../../raw/current/scripts/prefabs/saddle.lua#L200-L203)
- **边界：脚印只在 owner 是玩家时存在。** 装备时 owner 是坐骑，`SetOwner` 走 `else` 分支移除 `updatelooper`、删池；只有上鞍改主到玩家后才建池。下鞍即清池，已显示的贴片要等各自 `animover` 才消失。[分支](../../../raw/current/scripts/prefabs/saddle.lua#L161-L178)
- **陷阱：解绑旧宿主的实参是帧表。** `SetOwner` 用 `DetachChild(inst.fx)`，而挂接时是 `AttachChild(inst)`；`colouradder` 的 children 以实体为键，传帧表查不到，旧宿主的登记不会被删（未运行核验可见影响）。[解绑](../../../raw/current/scripts/prefabs/saddle.lua#L186-L188)、[children](../../../raw/current/scripts/components/colouradder.lua#L24-L49)
- **边界：脚印池挂在 FX 壳上。** 回收条件是 `inst.owner ~= nil and inst.owner.footprint_pool ~= nil`；壳被移除或改主时池被删，未播完的贴片自行 `Remove`。池化实例复用时不重设 owner。[回池](../../../raw/current/scripts/prefabs/saddle_shadow_footprints.lua#L27-L37)、[清池](../../../raw/current/scripts/prefabs/saddle.lua#L169-L177)
- **边界：逐帧淡出按动画帧号写 alpha。** `OnUpdate` 在 2–36 帧直接返回，之后用 `Remap` 给 4 个 `footprint_N` 符号错峰降 alpha；`Remap` 不夹取，超出窗口的取值由引擎处理（未运行核验）。[淡出](../../../raw/current/scripts/prefabs/saddle_shadow_footprints.lua#L14-L25)、[Remap](../../../raw/current/scripts/mathutil.lua#L19-L21)
- **未核验：晚加入客户端。** 工厂没有 `OnEntityReplicated` 之类的首次复制回调，客户端只监听 `ownerevent`；晚加入时是否收到一次事件、从而创建首批本地帧，源码未见，需运行验证。[客户端分支](../../../raw/current/scripts/prefabs/saddle.lua#L228-L232)、[回调](../../../raw/current/scripts/prefabs/saddle.lua#L206-L212)

## 验证记录

- 源码审阅：2026-11-04，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。逐行核对两个工厂的注册名、bank／build／动画、标签、端分支、`persists` 与清理；生成点追到 `saddle_shadow` 的 `OnEquipped`／`OnUnequipped`、`Rideable:SetSaddle`／`OnLoadDomesticatable`、`rider:Mount`／`ActualDismount`；并核 `colouradder`／`colouraddersync`／`updatelooper`／`highlightchild`／`net_event` 的接线。全 `raw/current/scripts` 用 `grep -rn --include='*.lua'` 反查两个完整注册名：`saddle_shadow_fx` 只命中自身工厂、`ownerevent`、依赖表与注册；`saddle_shadow_footprint` 只命中 `SpawnPrefab`、依赖表与注册，另 `prefablist.lua` 的 `saddle_shadow_footprints` 是文件名条目；未穷举运行时拼接名。
- 名称核验：`languages/chinese_s.po` 定向检索两个注册名无 `STRINGS.NAMES.*` 键；`SADDLE_SHADOW` 的物品名键见“注册与构造”，标题使用效果描述并注明不是官方译名。
- 分档：覆盖 2 个注册、2 个工厂文件、1 条装备→上鞍→下鞍→读档链、2 种端模型与双端本地帧协议，完整证据链超出 standard 预算（实测 17,621 字节，约 17.2 KiB），按 `complex` 记录；在 400 行／32 KiB 内。
- 游戏运行：未运行；未验证动画包实际帧数／循环标记、跨端可见时序、晚加入与专用服务器表现。
- 结构检查：`validate_page.py`、`validate_source_links.py --strict` 与 Lua 语法探针的实际结果见交付报告。

## 相关链接

- [[reference/prefabs/gestalt_trail|宿主附着与跟随的外围 FX 族]]（同型：宿主侧生成、跟随符号、清理；对照网络壳＋本地帧）
- [[reference/prefabs/armor_lunarplant_glow_fx|装备与帽饰跟随光效]]（同型：网络控制器＋本地子帧、颜色同步）
- [[reference/components/rideable|rideable]]、[[reference/components/rider|rider]]、[[reference/components/saddler|saddler]]
- [[reference/components/colouraddersync|colouraddersync]]、[[reference/components/updatelooper|updatelooper]]、[[reference/components/highlightchild|highlightchild]]、[[reference/components/follower|follower]]
- [[reference/prefabs/fx-world-index|特效 Prefab 索引·生物与环境]]

返回 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]。
