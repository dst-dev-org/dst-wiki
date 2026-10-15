---
title: "书本挂载特效族（fx_tentacles_under_book 等 12 注册名）"
created: "2026-11-04"
updated: "2026-11-04"
type: prefab
summary: "books.lua 的 MakeFX 按书目 def 的 fx_over／fx_under 生成的 12 个书本上／下层 FX：6 个效果键各注册步行／骑乘两版，SGwilson book2 把实例挂到读者并跟随书本符号，动画结束自删或状态退出兜底，不判定读书成败。"
aliases: ["fx_tentacles_under_book", "fx_tentacles_under_book_mount", "fx_lightning_over_book", "fx_lightning_over_book_mount", "fx_plants_small_under_book", "fx_plants_small_under_book_mount", "fx_plants_big_under_book", "fx_plants_big_under_book_mount", "fx_roots_under_book", "fx_roots_under_book_mount", "fx_fish_under_book", "fx_fish_under_book_mount", "fx_book_mount", "书页挂载特效", "书本前后层特效", "书本挂载特效"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/books.lua", "stategraphs/SGwilson.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 书本挂载特效族（fx_tentacles_under_book 等 12 注册名）

## 快速判断

- 职责与适用：`prefabs/books.lua` 的 `MakeFX` 工厂生成的 12 个书本上／下层 FX（[工厂](../../../raw/current/scripts/prefabs/books.lua#L1053-L1092)）。效果键 6 个：`tentacles`、`lightning`、`plants_small`、`plants_big`、`roots`、`fish`；每个效果同时注册步行与骑乘两版（[注册](../../../raw/current/scripts/prefabs/books.lua#L1094-L1109)）。12 名只承载读书时的书本上下层动画，不含玩法结算。
- 优先使用：查"读书时书本上下层动画从哪来、为什么自定义书没有、为何读书失败也出现"用本页；手持书动画见 [[reference/prefabs/book_fx|book_fx]]；读书成功后的 18 名法术演出见 [[reference/prefabs/fx_book_moon|fx_book_moon]]；读书光实体见 [[reference/prefabs/cavelight_small|cavelight_small]]。
- 重要副作用：主模拟进入 SGwilson `book2` 时生成，挂到读者实体并跟随 `swap_book_fx_over`／`swap_book_fx_under` 符号；生成发生在读书结算之前，所以书页层不等待读书成功；FX 自身无音效／Light，不存档；动画结束自删，状态退出兜底 Remove。
- 修改边界：12 名的字面前缀都是 `fx_`，没有 `fx_book_*` 形式，也没有 `fx_book_mount` 这个注册名；名字由 `fx_`..效果键..`_over_book`／`_under_book` 拼出，`_mount` 同时改注册名与动画名（[命名与挂载](../../../raw/current/scripts/prefabs/books.lua#L1053-L1057)）。只有 `book.def` 带 `fx_over_prefab`／`fx_under_prefab` 的书才触发；这是 `books.lua` 与 SGwilson 之间的内部字段，不是公开接口。

## 注册与构造

`MakeFX(name, anim, ismount)` 是唯一工厂：

- 资产只有 `anim/fx_books.zip`（[assets_fx](../../../raw/current/scripts/prefabs/books.lua#L7-L10)）；实体建 Transform／AnimState／Follower／Network，加游戏标签 `FX`，步行设四向、骑乘设六向（[基础构造](../../../raw/current/scripts/prefabs/books.lua#L1062-L1073)）。
- 全部 `bank`／`build` 为 `fx_books`，播放请求的动画名；`SetPristine` 后非主模拟直接返回（[动画与端分界](../../../raw/current/scripts/prefabs/books.lua#L1075-L1083)）。
- 主模拟监听 `animover` 自删，`persists = false`；不 `AddComponent`、不设 SG／Brain、无 Light／SoundEmitter（[清理与主模拟](../../../raw/current/scripts/prefabs/books.lua#L1085-L1091)）。
- `ismount=true` 时给注册名与动画名同时追加 `_mount`（[L1054-L1057](../../../raw/current/scripts/prefabs/books.lua#L1054-L1057)）。

注册循环遍历 `book_defs`：有 `fx_over` 的书拼出 `fx_<键>_over_book`，有 `fx_under` 的书拼出 `fx_<键>_under_book`，把字段写回 def（`v.fx_over_prefab`／`v.fx_under_prefab`），再对每个字段分别调用 `MakeFX(..., false)` 与 `MakeFX(..., true)`（[L1097-L1105](../../../raw/current/scripts/prefabs/books.lua#L1097-L1105)）。6 个效果键因此产生 12 个注册。

`MakeBook` 也把每个 `fx_over`／`fx_under` 的基名与 `_mount` 名插入自己 `Prefab` 的 `prefabs`／deps 列表（[L979-L990](../../../raw/current/scripts/prefabs/books.lua#L979-L990)）。这仅是依赖声明，不生成实体；唯一生成点是下节的 SGwilson `book2`。

| 书目 def（fx 字段） | 注册名（步行／骑乘） | 动画名（步行／骑乘） |
|---|---|---|
| `book_tentacles`（`fx_under = "tentacles"`，[L206-L210](../../../raw/current/scripts/prefabs/books.lua#L206-L210)） | `fx_tentacles_under_book`／`fx_tentacles_under_book_mount` | `tentacles`／`tentacles_mount` |
| `book_brimstone`（`fx_over = "lightning"`，[L349-L353](../../../raw/current/scripts/prefabs/books.lua#L349-L353)） | `fx_lightning_over_book`／`fx_lightning_over_book_mount` | `lightning`／`lightning_mount` |
| `book_horticulture`（`fx_under = "plants_small"`，[L472-L476](../../../raw/current/scripts/prefabs/books.lua#L472-L476)） | `fx_plants_small_under_book`／`fx_plants_small_under_book_mount` | `plants_small`／`plants_small_mount` |
| `book_horticulture_upgraded`（`fx_under = "plants_big"`，[L493-L497](../../../raw/current/scripts/prefabs/books.lua#L493-L497)） | `fx_plants_big_under_book`／`fx_plants_big_under_book_mount` | `plants_big`／`plants_big_mount` |
| `book_silviculture`（`fx_under = "roots"`，[L515-L519](../../../raw/current/scripts/prefabs/books.lua#L515-L519)） | `fx_roots_under_book`／`fx_roots_under_book_mount` | `roots`／`roots_mount` |
| `book_fish`（`fx_under = "fish"`，[L558-L563](../../../raw/current/scripts/prefabs/books.lua#L558-L563)） | `fx_fish_under_book`／`fx_fish_under_book_mount` | `fish`／`fish_mount` |

6 行 × 2 = 12 注册。under 组统一由 [L1103-L1105](../../../raw/current/scripts/prefabs/books.lua#L1103-L1105) 注册，over 组由 [L1098-L1100](../../../raw/current/scripts/prefabs/books.lua#L1098-L1100) 注册。原版 6 本各只设 `fx_over` 或 `fx_under` 之一，所以一次读书最多出现一层；`book_fish` 另有 `fx = "fx_book_fish"` 的法术演出（属 18 名那一族，不在此表，[L558-L563](../../../raw/current/scripts/prefabs/books.lua#L558-L563)）。

## 生成、父挂与清理

1. **状态入口。** `book2` 的 `onenter` 从 `bufferedaction` 取书本实体；书本在上一状态已 `ReturnActiveActionItem` 回背包，因此这里的父挂对象不是书本物品（[取书](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10046-L10048)、[book2 入口](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10105-L10132)）。
2. **选名。** `suffix = rider:IsRiding() and "_mount" or ""`；`book.def` 存在时按 `fx_over_prefab`／`fx_under_prefab` 分别 `SpawnPrefab(名..suffix)`，结果写入 `statemem.fx_over`／`fx_under`（[L10130-L10143](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10130-L10143)）。两个字段互不依赖，原版不会同时有值。
3. **父挂。** 生成后立即 `entity:SetParent(inst.entity)` 挂到读者；`Follower:FollowSymbol` 跟随读者读书动画的 `swap_book_fx_over`／`swap_book_fx_under` 符号，而不是书本实体（[父挂与符号](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10135-L10142)）。朝向按步行／骑乘设四／六向。
4. **帧对齐。** `frameskip` 默认 0；`repeatcast` 时 `book2` 以 6 帧跳过开头，书页层也 `SetFrame(6)`（[帧跳过](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10117-L10125)、[over](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10137)、[under](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10143)）。首次读书从第 0 帧开始。
5. **端。** `inst.def` 只在书本 prefab 的主模拟分支赋值（[books.lua L1013-L1019](../../../raw/current/scripts/prefabs/books.lua#L1013-L1019)），客户端构造时 `book.def` 为 nil，因此生成段只在主模拟执行。FX 实体带 Network；客户端通过复制获得实体，工厂在 `SetPristine` 后对非主模拟直接返回，不注册 `animover` 监听（[端分界](../../../raw/current/scripts/prefabs/books.lua#L1079-L1086)），客户端移除依赖服务端复制。
6. **清理。** 主模拟在 `animover` 时 `inst.Remove` 自删，并 `persists = false` 不存档（[L1085-L1086](../../../raw/current/scripts/prefabs/books.lua#L1085-L1086)）；`book2` 的 `onexit` 对两者做 `IsValid` 检查后 Remove，覆盖打断与动画未先结束（[L10360-L10365](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10360-L10365)）。
7. **与结算的先后。** 书本结算 `PerformBufferedAction` 在 `book2` 时间线的后段（第 44／50 帧；`earlycast` 类更早），书页层在进入状态时已生成（[结算时间线](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10267-L10312)、[早期施法](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10197-L10223)）。书本 `fn` 返回失败时书页层仍会出现并持续到动画结束或状态退出；成功后才生成的法术演出属于另一族。

## 关联机制导航

本族不 `AddComponent`、不设 SG／Brain、无 Light／SoundEmitter；仅引擎侧 `Follower` 供符号挂接，唯一状态协作是 SGwilson `book2` 的生成。

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂：`MakeFX` | 12 个注册共用；四／六向、`FX` 标签、不存档、`animover` 自删 | 主模拟创建并清理；客户端由 Network 复制 | [工厂](../../../raw/current/scripts/prefabs/books.lua#L1053-L1092) |
| 组件：`Follower` | `AddFollower` 后由 SG 调 `FollowSymbol` | `FollowSymbol` 在主模拟 | [L1064](../../../raw/current/scripts/prefabs/books.lua#L1064)、[L10136](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10136) |
| 数据：`book_defs`／`MakeBook` | `fx_over`／`fx_under` 字段；注册循环写回 `fx_*_prefab`；`MakeBook` 把基名与 `_mount` 名并入 deps | deps 只声明不生成；字段只在原版书目上自动出现 | [deps](../../../raw/current/scripts/prefabs/books.lua#L979-L990)、[注册循环](../../../raw/current/scripts/prefabs/books.lua#L1097-L1105) |
| 消费者：SGwilson `book2` | 进入状态时按 def 字段生成、`SetParent`、`FollowSymbol`、可选的 6 帧跳过；`onexit` 兜底 Remove | 主模拟状态；骑乘追加 `_mount`；打断由 `onexit` 清理 | [生成](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10128-L10144)、[清理](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10349-L10375) |
| 宿主：书本 prefab 的 `def` | `inst.def = def` 只在主模拟；SG 读取 `fx_over_prefab`／`fx_under_prefab` | 客户端 `book.def` 为 nil；无公开设置接口 | [books.lua L1013-L1019](../../../raw/current/scripts/prefabs/books.lua#L1013-L1019) |
| 关联 Prefab：`book_fx` 族 | 读书动作中挂在读者身上的手持书本动画，由 `book` 状态生成 | 独立工厂与注册；与本族分层并存 | [[reference/prefabs/book_fx]]、[生成](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10061-L10070) |
| 关联 Prefab：`fx_book_*` 18 名 | 读书成功结算后生成的法术演出（含 `fx_book_light` 等） | 独立工厂与生成路径，详见该页 | [[reference/prefabs/fx_book_moon]] |
| 关联：`booklight` | `book_light`／`book_light_upgraded` 的读书光实体；两本没有 `fx_over`／`fx_under`，只有 `fx` 与 `deps = { "booklight" }` | 光源实体；不是本族 12 名 | [[reference/prefabs/cavelight_small]]、[book_light](../../../raw/current/scripts/prefabs/books.lua#L757-L762)、[upgraded](../../../raw/current/scripts/prefabs/books.lua#L783-L788) |
| 资源：`anim/fx_books.zip` | 12 名共用的资产声明与 bank／build／动画来源 | 包内容未解包；动画是否存在、帧长未验证 | [资产](../../../raw/current/scripts/prefabs/books.lua#L7-L10)、[使用](../../../raw/current/scripts/prefabs/books.lua#L1075-L1077) |
| 数据：`layer_sound` | `book2` 在读者 SoundEmitter 上播放的书本层音效；不属于本族 FX | 仅部分书目有；帧位置各自不同 | [消费](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10146-L10160)、[tentacles](../../../raw/current/scripts/prefabs/books.lua#L211)、[upgraded](../../../raw/current/scripts/prefabs/books.lua#L498)、[silviculture](../../../raw/current/scripts/prefabs/books.lua#L520) |

## 常见陷阱与限制

- **误区：12 名是 `fx_book_*`**。实际名字是 `fx_<效果键>_over_book`／`_under_book`（骑乘再加 `_mount`）；页面文件名 `fx_book_mount` 是主题名，不是注册名。[命名](../../../raw/current/scripts/prefabs/books.lua#L1098-L1105)
- **误区：`MakeBook` 的 `prefabs`／deps 会生成书页层**。它只把名字加入依赖声明，唯一 `SpawnPrefab` 在 SGwilson `book2`；不要把 deps 数当生成数。[deps](../../../raw/current/scripts/prefabs/books.lua#L979-L990)、[生成](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10133-L10143)
- **边界：书页层不等待读书成功**。`book2` 进入时已生成，结算在更晚帧；`fn` 失败时仍会出现。成功后的 `fx_book_*` 走另一条路径，不能用本族的时序推断它。[时间线](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10267-L10312)
- **边界：只有主模拟能生成**。`book.def` 在客户端为 nil，客户端只收到复制实体；不要把它当客户端预测 FX 的范例。[端分界](../../../raw/current/scripts/prefabs/books.lua#L1013-L1019)
- **边界：原版一次最多一层**。6 本各只有一个 `fx_over`／`fx_under`；只有模组同时设两个字段才会 over＋under 同现（SG 两段都支持）。[字段](../../../raw/current/scripts/prefabs/books.lua#L206-L210)
- **边界：动画与资源强耦合**。12 名共用 bank／build `fx_books`，动画名等于效果键，`_mount` 同时改两名；不能套用 `fx_book_*` 的 bank／build 或任意动画名。zip 内容未解包验证。[资源](../../../raw/current/scripts/prefabs/books.lua#L1075-L1077)
- **辨析：读书动作里的四类表现别混**。`book_fx` 是手持书动画，`fx_book_*` 是成功后的法术演出，`booklight` 是读书光，本页 12 名是书本上下层；四者工厂、宿主与页面不同，见上表互链。
- **清理边界：动画先结束则提前消失**。`animover` 自删可能早于 `book2` 结束；不要假设书页层一定撑满整段读书。[自删](../../../raw/current/scripts/prefabs/books.lua#L1085-L1086)

## 验证记录

- 源码审阅：2026-11-04，Steam buildid 26335394。已核验：`books.lua` 的 `MakeFX` 全文（L1053-L1092）、`assets_fx`（L7-L10）、6 个带 `fx_over`／`fx_under` 的书目条目（L206-L210、L349-L353、L472-L476、L493-L497、L515-L519、L558-L563）、注册循环（L1094-L1109）、`MakeBook` 依赖并入（L979-L990）与 `inst.def` 主模拟赋值（L1013-L1019）；SGwilson 的 `book`／`book2` 生成、父挂、符号、帧跳过、端边界与 `onexit` 清理（L10046-L10048、L10105-L10144、L10267-L10312、L10349-L10375）。全 `raw/current/scripts` 反查：`fx_over`／`fx_under` 仅命中 `prefabs/books.lua` 与 `stategraphs/SGwilson.lua`；12 个最终名没有静态字面量，均来自动态拼接；`"fx_"..` 动态拼名在本族只出现在 `books.lua` 四处；`languages/*.po` 与 `strings.lua` 对 12 名无命中，故标题为已核验效果描述，不是官方译名。未覆盖：`anim/fx_books.zip` 包内动画是否存在及帧长、`swap_book_fx_over`／`swap_book_fx_under` 在玩家动画资源中的定义、客户端复制时序与晚加入表现、读书失败时实际可见时长。
- 游戏运行：未运行。
- 结构检查：`validate_page.py` 通过（93 行／16,291 字节，standard 档 240 行／24 KiB 内）；`validate_source_links.py --strict` 通过（errors 0／warnings 0）；本页无完整 Lua 示例，另对两份 sources 跑 `parser_probe --syntax-only` 通过（只证语法）。分档：standard（同一工厂与生成点，无多状态）。

返回 [[reference/prefabs/fx-world-index|特效 Prefab 索引·环境与世界]]。
