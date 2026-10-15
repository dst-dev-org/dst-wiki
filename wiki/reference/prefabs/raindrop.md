---
title: "落点水花与呼气粒子（raindrop／frostbreath）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "两个宿主自管生命周期的本地外围 FX：raindrop 由 rain／caverain／lunarhail 池化取用并在 animover 回池；frostbreath 是单发射器 VFXEffect 呼气粒子，由 frostybreather 与切斯特挂接、手动 Emit 与移除。"
aliases: ["raindrop", "frostbreath", "雨滴落点", "落点水花", "呼气粒子", "寒气吐息", "白色吐息", "外围 FX 族"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/raindrop.lua", "prefabs/frostbreath.lua", "prefabs/rain.lua", "prefabs/caverain.lua", "prefabs/lunarhail.lua", "components/frostybreather.lua", "prefabs/chester.lua", "emitters.lua", "prefablist.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 落点水花与呼气粒子（raindrop／frostbreath）

## 快速判断

- 职责与适用：两个手写本地 FX，不走 `fx.lua` 通用表。`raindrop` 是单动画落点水花，只由 `rain`／`caverain`／`lunarhail` 的落点池取用；`frostbreath` 是单发射器 VFXEffect 呼气粒子，只由 `frostybreather` 与切斯特挂接。两者都只有 `FX` 标签、无 `AddNetwork()`、`persists = false`，不结算伤害／湿度／温度。[构造](../../../raw/current/scripts/prefabs/raindrop.lua#L19-L39)、[构造](../../../raw/current/scripts/prefabs/frostbreath.lua#L69-L104)
- 优先使用：雨滴落地水花复用 `raindrop` 池约定（父级持池 → `GetPooledFx` → `animover` 回池）；口部白气 `SpawnPrefab("frostbreath")` 后由宿主 `AddChild`／`SetParent` 并手动 `Emit()`。[池](../../../raw/current/scripts/prefabs/rain.lua#L53-L78)、[挂接](../../../raw/current/scripts/components/frostybreather.lua#L47-L67)
- 重要副作用：`raindrop` 动画结束回池或自删；`frostbreath` 无自删，必须宿主移除；专用服上它只有 Transform 与空 `Emit`。[回池](../../../raw/current/scripts/prefabs/raindrop.lua#L6-L13)、[专服](../../../raw/current/scripts/prefabs/frostbreath.lua#L78-L85)
- 修改边界：本页只写两个注册的工厂、端、生成点、父挂与清理；取点门禁见 [[reference/mechanisms/rain-fx-gating]]，`frostybreather` 的温度／开关／偏移见 [[reference/components/frostybreather]]，切斯特状态玩法归其宿主页（Wiki 待建）。

## 注册与构造

两者都是 `Prefab(注册名, fn, assets)` 三参直接返回，无包装工厂、自身不带 `prefabs` 依赖表、无变体，`fn` 不接收外部参数。差异在生命周期载体：`raindrop` 用 AnimState 和 `animover`；`frostbreath` 用 VFXEffect 和手动 `Emit`。

| 注册名／变体 | 工厂与初始化分界 | 类别、端与差异证据 |
|---|---|---|
| `raindrop` | `fn`（[L19-L39](../../../raw/current/scripts/prefabs/raindrop.lua#L19-L39)）：`CreateEntity()` → `FX` 标签 → `SetCanSleep(false)`／`persists = false` → Transform＋AnimState → bank／build `raindrop`、播 `anim` → 监听 `animover` → 暴露 `RestartFx`。注册（[L41](../../../raw/current/scripts/prefabs/raindrop.lua#L41)）；assets 为 `anim/raindrop.zip`（[L1-L4](../../../raw/current/scripts/prefabs/raindrop.lua#L1-L4)）。 | 纯动画落点：无 Network／SoundEmitter／物理；`RestartFx` 只重播 `anim`，不重置池字段（[L15-L17](../../../raw/current/scripts/prefabs/raindrop.lua#L15-L17)）。 |
| `frostbreath` | `fn`（[L69-L104](../../../raw/current/scripts/prefabs/frostbreath.lua#L69-L104)）：Transform → `FX` 标签 → `persists = false` → 专用服只赋空 `Emit` 并提前返回（[L78-L85](../../../raw/current/scripts/prefabs/frostbreath.lua#L78-L85)）→ 非专服注册包络（[L20-L42](../../../raw/current/scripts/prefabs/frostbreath.lua#L20-L42)）＋VFXEffect 单发射器（[L87-L96](../../../raw/current/scripts/prefabs/frostbreath.lua#L87-L96)）＋`CreateSphereEmitter(.05)`（[L98](../../../raw/current/scripts/prefabs/frostbreath.lua#L98)）＋`Emit`（[L48-L64](../../../raw/current/scripts/prefabs/frostbreath.lua#L48-L64)）＋`EmitterManager:AddEmitter(inst, nil, empty_func)`（[L101](../../../raw/current/scripts/prefabs/frostbreath.lua#L101)）。注册（[L106](../../../raw/current/scripts/prefabs/frostbreath.lua#L106)）；assets 为 `fx/frostbreath.tex`＋`shaders/vfx_particle.ksh`（[L1-L12](../../../raw/current/scripts/prefabs/frostbreath.lua#L1-L12)）。 | 纯粒子：白色、寿命 2.0～2.5 秒、缩放 .4→3、最多 64 粒、预乘混合、UV 1×1；专服无 VFXEffect，`Emit` 为空函数。 |

名称核验：`chinese_s.po` 全文件无 `STRINGS.NAMES.RAINDROP`／`FROSTBREATH` 名称键；仅两处台词含 “raindrop”（[L41016](../../../raw/current/scripts/languages/chinese_s.po#L41016)、[L159186](../../../raw/current/scripts/languages/chinese_s.po#L159186)），`STRINGS.NAMES.RAIN` 为“短路”（[L348469-L348472](../../../raw/current/scripts/languages/chinese_s.po#L348469-L348472)），对应 `rain` 而非本页。中文标题为效果描述，非官方译名。

### 生成点与反查

| 注册名 | 生成点（文件／函数） | 调用形态与条件 | 证据 |
|---|---|---|---|
| `raindrop` | `rain.SpawnRaindropAtXZ` | `GetPooledFx("raindrop", inst.raindrop_pool)`；溅落预算 > 0 时才取用；取点需可通行＋`CanPointHaveRain`，雨棚边缘另有补偿滴 | [池](../../../raw/current/scripts/prefabs/rain.lua#L116)、[取用](../../../raw/current/scripts/prefabs/rain.lua#L72-L78) |
| `raindrop` | `caverain.SpawnRaindropAtXZ` | 与 `rain` 同构，池独立 | [池](../../../raw/current/scripts/prefabs/caverain.lua#L117)、[取用](../../../raw/current/scripts/prefabs/caverain.lua#L73-L79) |
| `raindrop` | `lunarhail.SpawnLunarHailDropAtXZ` | 仅 `Map:IsOceanAtPoint` 为真时取 `raindrop`，陆地取 `lunarhaildrop`；喷溅只排雨棚 | [海陆选择](../../../raw/current/scripts/prefabs/lunarhail.lua#L76-L95)、[池](../../../raw/current/scripts/prefabs/lunarhail.lua#L136-L137) |
| `frostbreath` | `frostybreather.StartBreath` | `SpawnPrefab("frostbreath")` → `AddChild` → `SetPosition(GetOffset())`；`EmitOnce` 先排除朝上三向再重设位置并 `Emit()`；`StopBreath` 解父并 `Remove()` | [创建](../../../raw/current/scripts/components/frostybreather.lua#L47-L56)、[发射](../../../raw/current/scripts/components/frostybreather.lua#L93-L103) |
| `frostbreath` | `chester.EnableFrostBreath` | 非专服生成两只，`SetParent(inst.entity)`＋`AddFollower` 跟随 `breath_left`／`breath_right`；`EmitFrost` 每 4～5 帧随机偏移后 `Emit()`；关闭时移除并取消任务 | [启用](../../../raw/current/scripts/prefabs/chester.lua#L237-L283)、[发射](../../../raw/current/scripts/prefabs/chester.lua#L171-L197) |

反查：`raw/current/scripts` 全树 `.lua` 检索两个注册名、`GetPooledFx(`、`SpawnPrefab(` 与 `Prefab(`。`raindrop` 池取用仅上表三处，字面 `SpawnPrefab("raindrop")` 零命中（由 `GetPooledFx` 形参传入 `SpawnPrefab(prefab)`）；`frostbreath` 的 `SpawnPrefab` 仅 `frostybreather` 与切斯特（两只）。其余命中是依赖声明（[rain L14-L17](../../../raw/current/scripts/prefabs/rain.lua#L14-L17)、[caverain L14-L17](../../../raw/current/scripts/prefabs/caverain.lua#L14-L17)、[lunarhail L14-L18](../../../raw/current/scripts/prefabs/lunarhail.lua#L14-L18)、[chester L33](../../../raw/current/scripts/prefabs/chester.lua#L33)）与 `prefablist`（[L1030](../../../raw/current/scripts/prefablist.lua#L1030)、[L473](../../../raw/current/scripts/prefablist.lua#L473)），以及 `debugcommands` 的注释行。`worldgen/`、`map/`、房间与场景零命中；四个 `GetPooledFx` 均为文件内 `local`，无全局 helper。

### 池化、挂接与端

- `raindrop` 不父挂：`SpawnRaindropAtXZ` 只 `Transform:SetPosition(x, 0, z)`；池由每个父 FX 各持一份（rain L116、caverain L117、lunarhail L137），不共享。[取用](../../../raw/current/scripts/prefabs/rain.lua#L72-L78)
- 取用是后进先出：`table.remove(pool.ents)` 取最后一个；复用先 `ReturnToScene()` 再 `RestartFx()`，只有新生成才写 `fx.pool = pool`。[GetPooledFx](../../../raw/current/scripts/prefabs/rain.lua#L53-L63)
- `frostbreath` 有两种挂法：`frostybreather` 用 `AddChild`＋本地坐标；切斯特用 `SetParent`＋`AddFollower` 跟嘴部符号并按朝向改 `SetSortOrder`。切斯特专服分支只置哨兵、由 `animover` 推网络事件；`frostybreather` 不按端裁剪，专用服上得到只有 Transform 的空壳。[组件挂接](../../../raw/current/scripts/components/frostybreather.lua#L47-L56)、[切斯特挂接](../../../raw/current/scripts/prefabs/chester.lua#L245-L255)

### 清理

- `raindrop` 的结束由动画决定：`animover` → 池有效则 `RemoveFromScene()`＋`table.insert(pool.ents)`，池不存在或已失效则 `Remove()`。[OnAnimOver](../../../raw/current/scripts/prefabs/raindrop.lua#L6-L13)
- 父 FX 移除时 `OnRemoveEntity` 清空池内实体并置 `pool.valid = false`；此后才播完、仍在场景里的水滴走 `Remove()`。喷溅预算归零时每帧 `ClearPoolEnts`。[父级清理](../../../raw/current/scripts/prefabs/rain.lua#L80-L83)、[停喷清理](../../../raw/current/scripts/prefabs/rain.lua#L249-L253)
- `frostbreath` 无 `animover`、定时器或自删：`frostybreather:StopBreath()` 先 `RemoveChild` 再 `Remove()`，组件移除时也会调用；切斯特关闭时移除两只并取消任务。`EmitterManager` 注册由实体 `onremove` 注销。[组件清理](../../../raw/current/scripts/components/frostybreather.lua#L15-L18)、[管理器注销](../../../raw/current/scripts/emitters.lua#L30)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| `组件：frostybreather` | 创建／持有单个 `frostbreath` 子实体，按朝向重设位置后 `Emit()`；移除时清子实体。 | 双方构造；专用服只建空壳；温度与开关条件归组件页。 | [[reference/components/frostybreather]]；[创建](../../../raw/current/scripts/components/frostybreather.lua#L47-L56)、[发射](../../../raw/current/scripts/components/frostybreather.lua#L93-L103) |
| `关联 Prefab：chester` | 非专服生成两只 `frostbreath` 跟随嘴部符号，按朝向设 sort order 并周期 `Emit()`；专服走哨兵＋网络事件。 | 状态切到雪分支时启用，关闭时移除。 | 切斯特（Wiki 待建）；[启用](../../../raw/current/scripts/prefabs/chester.lua#L237-L283) |
| `关联 Prefab：rain` | 持有 `raindrop_pool`，溅落预算过门禁后 `GetPooledFx("raindrop", pool)`；`PostInit()` 预热按当前预算带 fastforward 生成一批。 | 非专服；四类降水共用一个组件。 | [[reference/prefabs/rain]]；[池](../../../raw/current/scripts/prefabs/rain.lua#L116)、[预热](../../../raw/current/scripts/prefabs/rain.lua#L258-L266) |
| `关联 Prefab：caverain／lunarhail` | 同构池取用；`lunarhail` 只在海面取 `raindrop`，陆地取 `lunarhaildrop`。 | 洞穴雨／月雹；都非专服。 | [[reference/prefabs/rain]]；[月雹选择](../../../raw/current/scripts/prefabs/lunarhail.lua#L76-L88) |
| `机制：雨滴门禁` | 取点前判雨棚、可通行与 `CanPointHaveRain`；本页只写池消费，不重复门控。 | 地表／洞穴共用。 | [[reference/mechanisms/rain-fx-gating]] |
| `机制：通用 FX 工厂` | 对照：这两个注册是手写 Prefab，不是 `fx.lua` 表条目，没有网络代理与 `fx_spawned`。 | — | [[reference/mechanisms/generic-fx-factory]] |

## 典型接入

### 给宿主挂一个本地呼气粒子

- 前提与选择：只在非专服需要可见粒子；`frostbreath` 是本地 FX，不联网，不能作为玩法效果。宿主已有 Transform 即可，挂点用本地坐标。
- 生效与撤销：生成后立即 `Emit()` 一次，之后每次呼气再 `Emit()`；撤销必须 `RemoveChild`／`Remove()`，实体不会自己消失。
- 原版依据：`frostybreather:StartBreath()` 的 `SpawnPrefab` → `AddChild` → `SetPosition` 与 `StopBreath()` 的 `RemoveChild` → `Remove`；切斯特的 `SetParent`＋`AddFollower` 是跟随符号的变体。[创建](../../../raw/current/scripts/components/frostybreather.lua#L47-L56)、[清理](../../../raw/current/scripts/components/frostybreather.lua#L58-L67)

```lua
-- modmain.lua：给宿主挂一个本地呼气粒子，并在撤销时移除
local function AttachBreath(host)
    if GLOBAL.TheNet:IsDedicated() then
        return nil
    end
    local fx = GLOBAL.SpawnPrefab("frostbreath")
    host:AddChild(fx)
    fx.Transform:SetPosition(0, 1.5, 0)
    fx:Emit()
    return fx
end

local function DetachBreath(host, fx)
    if fx ~= nil and fx:IsValid() then
        host:RemoveChild(fx)
        fx:Remove()
    end
end
```

- 调整与验证：偏移按宿主模型改 `SetPosition`；需要跟动画符号时改 `entity:SetParent`＋`AddFollower`＋`FollowSymbol`。该片段按原版调用序列改写，未做游戏运行验证；粒子参数在工厂内固定，不从外部接收。

## 常见陷阱与限制

- **边界：`SpawnPrefab("frostbreath")` 不会自己冒气。** 工厂只建发射器和包络，`EmitterManager` 注册的 updateFunc 是空函数；必须外部调用 `Emit()`。[工厂注册](../../../raw/current/scripts/prefabs/frostbreath.lua#L98-L101)
- **边界：`frostbreath` 不能套用 `raindrop` 的池。** 它没有 `RestartFx`、没有 AnimState，`GetPooledFx` 的复用分支会直接报错。[GetPooledFx](../../../raw/current/scripts/prefabs/rain.lua#L53-L63)
- **边界：专用服上 `frostbreath` 只有 Transform 与空 `Emit`，没有 `VFXEffect`。** 无条件访问 `fx.VFXEffect` 会报错；切斯特正是用专服分支避免生成。[专服分支](../../../raw/current/scripts/prefabs/frostbreath.lua#L78-L85)
- **边界：`frostbreath` 无自删，漏掉 Remove 会长期留场；`raindrop` 则在动画结束时自行回池或移除。** 两者都不入存档，`persists = false` 不负责清理。
- **边界：`raindrop` 的池是父级私有且不父挂。** 直接 `SpawnPrefab("raindrop")` 没有 `pool` 字段，`animover` 会直接 `Remove()`，无法复用。[回池条件](../../../raw/current/scripts/prefabs/raindrop.lua#L6-L13)

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394（`raw/current/BUILD.txt`）。逐行核验两个工厂与注册；核验三个池消费者的取用、池构造、停喷与移除清理，`frostybreather` 的创建／发射／移除，切斯特的生成／跟随／sort order／移除与专服哨兵，`emitters.lua` 的球面发射器与 `EmitterManager` 注册／注销，`prefablist` 与 `chinese_s.po` 名称键。全 scripts 反查两注册名、`GetPooledFx(`、`SpawnPrefab(` 与 `Prefab(`，`worldgen/`、`map/`、房间与场景零命中。未覆盖：动画帧长与实际外观、粒子参数在引擎内的表现、资源包实际可用性、宿主玩法与存档恢复。
- 游戏运行：未运行。
- 结构检查：`validate_page.py` 通过；`validate_source_links.py --strict` 通过；Lua 示例 `parser_probe --syntax-only` 通过。分档未填 `size_class`，按默认 standard。

## 相关链接

- [[reference/prefabs/rain]]、[[reference/components/frostybreather]]、[[reference/mechanisms/rain-fx-gating]]、[[reference/prefabs/acidraindrop]]
- 返回 [[reference/prefabs/fx-world-index|天气、地形与生态痕迹]]
