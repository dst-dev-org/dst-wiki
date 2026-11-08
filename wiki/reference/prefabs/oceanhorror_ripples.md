---
title: "海上暗影生物的水面涟漪与船只挂点（oceanhorror_ripples、oceanhorror_attachpivot）"
created: "2026-11-05"
updated: "2026-11-08"
type: prefab
summary: "海上暗影生物 oceanhorror 的 2 个附层登记：脚下的网络化涟漪子实体（宿主 SG 驱动缩放、宿主透明组件驱动 alpha）与上船用的不可见挂点实体（生物成为其子体、经引擎平台跟随随船移动）；逐项记工厂、生成点、父挂与清理。"
aliases: ["oceanhorror_ripples", "oceanhorror_attachpivot", "海上暗影生物涟漪", "上船挂点", "恐怖利爪附层", "水面涟漪层"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/oceanshadowcreature.lua", "stategraphs/SGoceanshadowcreature.lua", "components/shadowcreaturespawner.lua", "prefabs/shadowcreature.lua", "components/walkableplatform.lua", "components/walkableplatformmanager.lua", "entityscript.lua", "componentutil.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 海上暗影生物的水面涟漪与船只挂点（oceanhorror_ripples、oceanhorror_attachpivot）

## 快速判断

- 职责与适用：本页收拢 `oceanshadowcreature.lua` 里 **2 个附层注册**，都只服务海上暗影生物 `oceanhorror` 的表现与位移：`oceanhorror_ripples` 是贴在宿主脚下的水面涟漪层（网络化子实体，动画、缩放与透明度全由宿主驱动）；`oceanhorror_attachpivot` 是没有 AnimState 的不可见挂点实体（生物上船时成为它的子体，挂点再以引擎平台跟随随船移动）。[涟漪注册](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L510)、[挂点注册](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L511)
- 本体辨析：`oceanhorror` 本体（注册于 [oceanshadowcreature.lua](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L509)）是 sanity 触发的玩法生物（combat／health／lootdropper／brain，官方名"恐怖利爪"），**不是 FX**，不在本页覆盖；其战斗、传送与刷怪规则不展开。
- 重要副作用：涟漪的 alpha 不走 netvar——宿主的 `transparentonsanity` 在每台非专用端本地计算 alpha 后写进涟漪，专用服务器上该组件不存在；挂点只在上船时生成，离船即被 `Remove`，`persists=false` 不跨存档。
- 修改边界：只覆盖 2 个附层的工厂、生成点、父挂／平台跟随与清理；宿主玩法（刷怪条件、战斗、船只传送状态机全貌）与 `shadow_oceanhorror.zip` 动画包内容不展开，引擎层平台跟随与父子回收未运行验证。
- 名称：宿主有官方名 `STRINGS.NAMES.OCEANHORROR`＝"恐怖利爪"（[po](../../../raw/current/scripts/languages/chinese_s.po#L346559-L346562)）；2 个附层注册名在简中 `chinese_s.po` 均无 `STRINGS.NAMES.*` 键，标题中的"水面涟漪""上船挂点"是已核验的效果描述，**非官方译名**。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `oceanhorror_ripples` | `ripplesfn`：Transform／AnimState／Network → bank＝build＝`oceanhorror`／`shadow_oceanhorror`、`water` 循环、`SetMultColour(1,1,1,.5)` → `NOCLICK`＋`FX` 标签 → Pristine → 客户端挂 `OnEntityReplicated`；主模拟只设 `persists=false` | 与宿主共用同一动画资源 `anim/shadow_oceanhorror.zip`；自身无透明逻辑，注释写明"transparency set from oceanhorror parent object"。[工厂](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L452-L478)、[资源](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L1-L4) |
| `oceanhorror_attachpivot` | `attachpivotfn`：Transform＋Network（**无 AnimState，不可见**）→ `NOCLICK`＋`ignorewalkableplatformdrowning` 标签 → Pristine → 主模拟 `persists=false`、听自身 `onsink` | 无资源、无动画；职责只是"生物的父锚点＋随船移动＋沉船信号中继"。[工厂](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L486-L507)、[onsink 处理](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L480-L484) |

宿主 `oceanhorror` 的关键差异（本页仅辨析用）：主模拟在 Pristine 之后无条件生成涟漪子实体并写 `inst._ripples`；`transparentonsanity` 只在非专用端、Pristine 之前装配。[生成涟漪](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L378-L386)、[装配透明组件](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L365-L368)

## 关联机制导航

### 生成点、端与父挂

| 生成点 | 涉及注册 | 端、阶段与条件 | 父挂／跟随 | 证据 |
|---|---|---|---|---|
| `oceanhorror` 工厂主模拟分支 | `oceanhorror_ripples` | 主模拟；宿主组装时无条件生成，全库唯一生成点 | `SetParent(host.entity)`；无 Follower，涟漪随宿主位移（引擎父子） | [生成](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L378-L379)、[注册](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L510) |
| `AttachToBoat`（宿主上船） | `oceanhorror_attachpivot` | 主模拟；`SGoceanshadowcreature` 的 `boatteleport`／`update` 任务调 `_attach_to_boat_fn`，且宿主当前无父时才生成；全库唯一生成点 | 生物 `SetParent(pivot.entity)` 成为子体；挂点定位到船心，生物再写船缘局部坐标并缩小碰撞胶囊 | [SpawnPrefab 挂点](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L94-L114)、[SG 调用](../../../raw/current/scripts/stategraphs/SGoceanshadowcreature.lua#L203-L214) |
| 挂点的随船移动（非生成） | `oceanhorror_attachpivot` | 主模拟；`walkableplatformmanager` 每帧对每个平台调 `SetEntitiesOnPlatform`，挂点无排除标签、无父，被登记为平台跟随者（引擎 `SetPlatform`），整条"挂点→生物→涟漪"链随船移动 | 引擎平台跟随，不是 Lua 组件 | [平台遍历](../../../raw/current/scripts/components/walkableplatformmanager.lua#L56-L62)、[登记条件](../../../raw/current/scripts/components/walkableplatform.lua#L154-L184)、[SetPlatform](../../../raw/current/scripts/entityscript.lua#L953-L971) |
| 宿主本体的刷怪点（辨析，不展开） | — | 主模拟；玩家在船上且 sanity<0.1 时 `shadowcreaturespawner` 在船外海面刷 `oceanhorror`；陆上暗影生物目标在海上时经 `ExchangeWithOceanTerror` 换成 `oceanhorror` | 生物在开阔海面无挂点，直接顶层存在 | [海面刷怪](../../../raw/current/scripts/components/shadowcreaturespawner.lua#L96-L111)、[换体](../../../raw/current/scripts/prefabs/shadowcreature.lua#L193-L212) |

### 属性驱动、同步与资源

| 对象 | 谁写、怎么写 | 条件与边界 | 证据 |
|---|---|---|---|
| 涟漪 alpha | 宿主 `transparentonsanity.onalphachangedfn = OnAlphaChanged`，把宿主 alpha 经 `OverrideMultColour(1,1,1,alpha)` 同步给涟漪；主模拟在生成后登记，客户端由涟漪复制回调按 `parent.prefab == "oceanhorror"` 重新登记 | 组件只在非专用端存在：专用服务器两侧（无组件）都不写 alpha，涟漪保持初始 0.5；alpha 是各端本地值，不是 netvar | [回调](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L269-L273)、[主模拟登记](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L381-L384)、[复制回调](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L443-L450) |
| 涟漪缩放 | 宿主 SG `SetRippleScale`：`boatteleport` 进态 1、随动画 1→0、退出 0；`appear` 进态 0、随动画 0→1、退出 1；下限钳到 0.01（注释：防止持海面装备时鼠标一直悬停宿主） | 主模拟调用，AnimState 随联网实体由引擎同步到客户端 | [SetRippleScale](../../../raw/current/scripts/stategraphs/SGoceanshadowcreature.lua#L76-L79)、[boatteleport](../../../raw/current/scripts/stategraphs/SGoceanshadowcreature.lua#L157-L176)、[appear](../../../raw/current/scripts/stategraphs/SGoceanshadowcreature.lua#L253-L271) |
| netvar | 两个附层都**没有自定义 netvar**；宿主、涟漪联网只为同步宿主写入的动画状态，挂点联网是因为生物要成为它的引擎子体 | 对照：`circlingbuzzard` 家族才有自管 netvar | [涟漪工厂](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L452-L478)、[挂点工厂](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L486-L507) |
| `ignorewalkableplatformdrowning` 标签 | 挂点带此标签：平台沉没清理（`DestroyObjectsOnPlatform`）与换地块沉没检索都跳过它，自身只听 `onsink` 做解绑 | 生物本体带的是 `ignorewalkableplatforms`（两份排除表都跳过），涟漪带 `FX` 同样被跳过 | [挂点标签](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L492-L494)、[排除表](../../../raw/current/scripts/components/walkableplatform.lua#L77-L78)、[换地块排除](../../../raw/current/scripts/componentutil.lua#L191-L198) |
| 资源 | `anim/shadow_oceanhorror.zip`（宿主与涟漪共用 bank `oceanhorror`） | 只核验声明名与工厂传入的 bank／build 字符串；包内帧数与视觉未解包 | [资源表](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L1-L4) |

### 清理路径

| 注册 | 清理方式 | 边界与证据 |
|---|---|---|
| `oceanhorror_ripples` | 无自删任务、无移除钩子；生命期完全挂在宿主上（宿主死亡态播完 `Remove`，涟漪的引擎级联回收未核验） | `persists=false` 只保证不存档；父体删除时是否连带回收子体未运行验证。[工厂尾部](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L470-L478)、[宿主死亡态](../../../raw/current/scripts/stategraphs/SGoceanshadowcreature.lua#L283-L296) |
| `oceanhorror_attachpivot`（离船） | `DetachFromBoat`：生物把世界坐标写回 Transform、解除父挂、`pivot:Remove()`、恢复海上碰撞胶囊 | 生物回到开阔海面后挂点必然不存在。[解绑](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L121-L139) |
| `oceanhorror_attachpivot`（生物被移除） | 生物的 `onremove` 处理器 `OnRemove`：若仍有父（挂点）则 `parent:Remove()` | 保证不留孤儿挂点。[OnRemove](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L279-L284) |
| `oceanhorror_attachpivot`（沉船） | 平台移除时对平台上实体推 `onsink`，挂点命中 `attachpivot_onsink` → 调生物 `_detach_from_boat_fn` 解绑 | 挂点在沉船时先解绑生物，再被后续流程处理；生物与挂点都因排除标签不会被平台清理逻辑直接销毁。[平台沉没推送](../../../raw/current/scripts/components/walkableplatform.lua#L24-L41)、[onsink 处理](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L480-L484) |

## 典型接入

### 复刻"宿主生成、父挂、宿主全权驱动"的贴体 FX

- 前提与选择：需要一个"随宿主出现／消失、宿主改其动画属性、客户端各自补登记"的联网子实体时，复用 `oceanhorror_ripples` 的组装：主模拟 `SpawnPrefab`＋`SetParent`，客户端在 `OnEntityReplicated` 里按父体 prefab 名回写引用。与 `nightmarelightfx` 的差别是本家族把 alpha 也留给宿主组件驱动。[生成](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L378-L379)、[复制回调](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L443-L450)
- 生效与撤销：原版依赖父挂与非持久化，宿主死亡后的引擎级联未核验；若自定义宿主想让 FX 淡出，应写宿主自身的透明逻辑并把值转给 FX，而不是直接改 FX（客户端会在复制回调里覆盖回调引用）。复用时应自管引用，并在撤销时显式 `Remove`，不能一边保留缺口一边保证“宿主死亡即消失”。
- 原版依据：工厂、标签与两端登记路径见"注册与构造"。

```lua
local function attach_ripples(host)
    if not TheWorld.ismastersim then
        return nil
    end
    local fx = SpawnPrefab("oceanhorror_ripples")
    fx.entity:SetParent(host.entity)
    host._ripples = fx
    return fx
end
```

- 调整与验证：替换点是 `SpawnPrefab` 注册名与宿主的 alpha／缩放写入方；客户端回写需自配 `OnEntityReplicated`。示例只做语法检查，未在游戏内执行。

### 用"挂点＋平台跟随"让子体随船移动

- 前提与选择：`oceanhorror_attachpivot` 展示了第三种随船方式——不装平台组件、不设 netvar，只做一个顶层锚点实体让平台管理器把它登记为引擎平台跟随者，再把生物挂到锚点下。适合"实体本身要保持自由移动（脑、物理），但整体要跟着平台走"的场景。
- 生效与撤销：锚点须放在平台半径内且无父、不带 `ignorewalkableplatforms` 等排除标签才会被登记；撤销即解绑子体并 `Remove` 锚点。沉船时平台会向锚点推 `onsink`，可借此解绑。[登记](../../../raw/current/scripts/components/walkableplatform.lua#L154-L184)、[沉没推送](../../../raw/current/scripts/components/walkableplatform.lua#L24-L41)
- 原版依据：[AttachToBoat](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L80-L119)

## 常见陷阱与限制

- **边界：涟漪 alpha 在专用服务器上恒为 0.5。** `transparentonsanity` 只在非专用端装配，`OnAlphaChanged` 的两端登记都以该组件存在为前提；专用服务器上宿主与涟漪都不淡出，淡出是各客户端本地视觉。[装配](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L365-L368)、[登记条件](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L381-L384)
- **边界：客户端回写依赖父体注册名精确匹配。** 复制回调判 `parent.prefab == "oceanhorror"`；若自定义宿主换名复用涟漪，客户端不会登记回调，宿主淡出时涟漪不同步。[复制回调](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L443-L450)
- **误区：挂点是船的子体。** 挂点从不 `SetParent` 到船上，也不装 Lua 组件；它靠平台管理器每帧的 `SetEntitiesOnPlatform` 把自己登记为引擎平台跟随者。若锚点在登记前就被移走或带排除标签，随船移动不会生效。[登记条件](../../../raw/current/scripts/components/walkableplatform.lua#L154-L184)
- **边界：`ignorewalkableplatformdrowning` 不等于 `ignorewalkableplatforms`。** 前者只出现在"沉没时不要销毁"的两份排除表里（`DestroyObjectsOnPlatform`、换地块沉没检索），平台跟随登记的排除表只认后者；挂点带前者所以既会随船、又不会被沉船清理销毁。[两份排除表](../../../raw/current/scripts/components/walkableplatform.lua#L77-L78)、[换地块排除](../../../raw/current/scripts/componentutil.lua#L191-L198)
- **边界：两个附层都无自身清理代码。** 涟漪没有任务与移除钩子，挂点只有 `onsink` 解绑；所有 `Remove` 都发生在宿主／解绑路径上。父体删除后的引擎级回收未运行核验，不要据此假设"无 Lua 清理＝无清理"。[涟漪工厂尾部](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L470-L478)、[挂点工厂尾部](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L498-L507)
- **边界：`DetachFromBoat` 的坐标回写依赖引擎父子语义。** 生物在仍处于父挂下时先 `SetPosition(GetWorldPosition())` 再解除父挂，这一写法在世界坐标与局部坐标之间的最终效果属引擎行为，本页未运行验证；自定复刻时应在真实环境确认落点。[解绑](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L121-L130)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/oceanshadowcreature.lua`（三个工厂全文）与 `stategraphs/SGoceanshadowcreature.lua` 的传送／出现／死亡状态，并按需读取 `shadowcreaturespawner`、`shadowcreature` 换体、`walkableplatform`／`walkableplatformmanager`／`entityscript` 平台跟随链与 `componentutil` 沉没排除表。对 2 个完整注册名以 `grep -rn "注册名" --include=*.lua` 在 `raw/current/scripts` 全域反查：`oceanhorror_ripples` 命中 3 处（工厂注册、宿主依赖表、宿主生成点），`oceanhorror_attachpivot` 命中 3 处（工厂注册、宿主依赖表、`AttachToBoat` 生成点）——两注册名均无其他生成点。宿主 `oceanhorror` 另命中 `shadowcreaturespawner` 刷怪、`shadowcreature` 换体、`forest.lua` 依赖表与 scrapbook／模块定义（非生成）。`prefablist.lua` 的文件级清单含整份 `oceanshadowcreature.lua`，同文件注册随文件加载。未穷举运行时拼接名。
- 名称核验：`languages/chinese_s.po` 定向检索——宿主 `STRINGS.NAMES.OCEANHORROR`＝"恐怖利爪"（[po](../../../raw/current/scripts/languages/chinese_s.po#L346559-L346562)）；`OCEANHORROR_RIPPLES`／`OCEANHORROR_ATTACHPIVOT` 无 `STRINGS.NAMES.*` 键，标题使用效果描述。
- 分档说明：`size_class: standard`（2 注册、3 类端角色、两条生成链与一套平台跟随机制，单页可完整承载），实测行数与字节见结构检查。
- 未覆盖：`shadow_oceanhorror.zip` 动画包实际帧数与涟漪观感；宿主刷怪节奏（`TUNING.SANITYMONSTERS_*` 与目标人口）与战斗数值表；`SGoceanshadowcreature` 其余状态；引擎层平台跟随的插值时序与父体删除时的子体回收。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录（0 error 0 warning 后入库）。

## 相关链接

- [[reference/prefabs/nightmarelightfx|暗影光源、幻灵头与盘旋秃鹫 FX 族]]（同为宿主附层；对照"宿主写 highlightchildren"与"宿主写 alpha"两种驱动）
- [[reference/prefabs/ice_puddle|宿主附属的贴地与贴水 FX 族]]（含 `oceanfishinghook_ripple`，同为水面涟漪表现）
- [[reference/components/walkableplatform|walkableplatform]]（平台跟随登记与沉没 `onsink` 推送的机制页）
- [[reference/components/shadowcreaturespawner|shadowcreaturespawner]]（宿主 `oceanhorror` 的刷怪管理器）
- 返回 [[reference/prefabs/fx-shadow-index|暗影生物、裂隙生物与附着]]
- 2026-11-08 build 26474873：本页随版本迁移升版（机械沿用；引用区间未受本次源码行号位移影响），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-07）。
