---
title: "读书施法特效族（fx_book_moon 等 18 注册名）"
created: "2026-11-01"
updated: "2026-11-08"
type: prefab
summary: "books.lua 原版书目施法成功的法术演出特效 18 注册名：通用表 FX 一批、步行/骑乘双 anim，两条生成路径（Book:OnRead 读者落点 fx/fxmount 字段链、书本 fn 内按受效对象直接生成），无 fxmount 的书骑乘回落基名并强制六向。"
aliases: ["fx_book_moon", "fx_book_moon_mount", "fx_book_research_station", "fx_book_research_station_mount", "fx_book_temperature", "fx_book_temperature_mount", "fx_book_bees", "fx_book_fire", "fx_book_fire_mount", "fx_book_light", "fx_book_light_upgraded", "fx_book_birds", "fx_book_birds_mount", "fx_book_sleep", "fx_book_sleep_mount", "fx_book_rain", "fx_book_rain_mount", "fx_book_fish", "读书特效", "读书法术特效", "施法特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26474873"
sources: ["fx.lua", "prefabs/fx.lua", "prefabs/books.lua", "components/book.lua", "components/reader.lua", "actions.lua", "stategraphs/SGwilson.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: complex
---

# 读书施法特效族（fx_book_moon 等 18 注册名）

## 快速判断

- 职责与适用：**FX 页面**。`fx.lua` 通用表中相邻连续的 18 注册名（[L2926-L3085](../../../raw/current/scripts/fx.lua#L2926-L3085)），是 `books.lua` 原版书目正式阅读施法成功后的法术演出特效：落点式（读者或受效对象位置，不父挂），全部纯表现——书本的精神、伤害、世界效果归书本与各机制页。18 名同属一条骑乘选名链与两条生成路径，需整体说明，故单页 complex 档整体保留。
- 优先使用：查"读书时脚下／目标身上的法术圈演出"选本页；工厂协议（服务端代理＋各端本地实体）只在 [generic-fx-factory#关键流程](../mechanisms/generic-fx-factory.md#关键流程) 维护，本页不复制。
- 重要副作用：每名带 `sound` 键 `wickerbottom_rework/book_spells/*`，由本地实体 SoundEmitter 播放；专用服务器不创建本地实体，故专服无此音效。[声音分支](../../../raw/current/scripts/prefabs/fx.lua#L43-L49)
- 修改边界：表内差异仅 anim／sixfaced／sound／fn（仅 fish 有 fn）；选名逻辑全部在消费者（`books.lua` def 字段与 [components/book.lua](../../../raw/current/scripts/components/book.lua#L61-L80)），改表不改选名。

## 注册与构造

全部 18 名走[通用工厂](../mechanisms/generic-fx-factory.md#关键流程)：每名一个 Prefab，主模拟负责隐形网络代理及其 1 秒回收；各非专服端（含非专服主机）另建本地动画实体，专服不建本地表现。本地实体按代理数据一帧后出现、`animover` 自删。基名与骑乘变体共享同一 bank/build（即基名），差异只有 anim、朝向与 fn。

| 注册名（基名／骑乘变体） | anim／朝向 | sound 键 | 专属 fn 与证据 |
|---|---|---|---|
| `fx_book_moon`／`fx_book_moon_mount` | `play_fx`／`play_fx_mount`（mount 六向） | `book_spells/moon` | 无 [条目](../../../raw/current/scripts/fx.lua#L2926-L2939) |
| `fx_book_research_station`／`fx_book_research_station_mount` | 同上 | `book_spells/researchstation` | 无 [条目](../../../raw/current/scripts/fx.lua#L2943-L2956) |
| `fx_book_temperature`／`fx_book_temperature_mount` | 同上 | `book_spells/temp` | 无 [条目](../../../raw/current/scripts/fx.lua#L2960-L2973) |
| `fx_book_bees`（无 mount） | `play_fx`，四向 | `book_spells/bees` | 无 [条目](../../../raw/current/scripts/fx.lua#L2977-L2981) |
| `fx_book_fire`／`fx_book_fire_mount` | `play_fx`／`play_fx_mount`（mount 六向） | `book_spells/fire` | 无 [条目](../../../raw/current/scripts/fx.lua#L2985-L2998) |
| `fx_book_light`／`fx_book_light_upgraded`（无 mount） | 均 `play_fx`，四向 | `book_spells/light`／`book_spells/light_upgrade` | 无 [条目](../../../raw/current/scripts/fx.lua#L3002-L3014) |
| `fx_book_birds`／`fx_book_birds_mount` | `play_fx`／`play_fx_mount`（mount 六向） | `book_spells/birds` | 无 [条目](../../../raw/current/scripts/fx.lua#L3018-L3031) |
| `fx_book_sleep`／`fx_book_sleep_mount` | `play_fx`／`play_fx_mount`（mount 六向） | `book_spells/sleep` | 无 [条目](../../../raw/current/scripts/fx.lua#L3035-L3048) |
| `fx_book_rain`／`fx_book_rain_mount` | `play_fx`／`play_fx_mount`（mount 六向） | `book_spells/rain` | 无 [条目](../../../raw/current/scripts/fx.lua#L3052-L3065) |
| `fx_book_fish`（无 mount） | `play_fx`，四向 | `book_spells/fish` | 贴地背景层＋MultColour 渐隐（见陷阱） [条目](../../../raw/current/scripts/fx.lua#L3069-L3085) |

六向由条目 `sixfaced = true` 在代理上 `Transform:SetSixFaced()`（[工厂分支](../../../raw/current/scripts/prefabs/fx.lua#L138-L146)），本地实体经 `SetFromProxy` 继承；基名未设 faced 标记走默认 `SetFourFaced`。18 名均无 NAMES 显示名键（`languages/chinese_s.po` 与 `strings.lua` 检索无命中，见验证记录），中文标题为已核验效果描述，非官方译名。

## 生成路径与骑乘选名

两条路径共用同一批注册名，选名依据不同：

**路径 A——`Book:OnRead` 在读者位置生成。** 施法结算成功后由 book 组件执行（[OnRead](../../../raw/current/scripts/components/book.lua#L61-L80)）：`ismount = reader.components.rider ~= nil and rider:IsRiding()`，`fx = ismount and self.fxmount or self.fx`，SpawnPrefab 后 `SetPosition(读者位置)`、`SetRotation(读者朝向)`。def 的 `fx`/`fxmount` 字段在 [MakeBook](../../../raw/current/scripts/prefabs/books.lua#L971-L977) 只作依赖声明，真正消费是 [SetFx 挂载](../../../raw/current/scripts/prefabs/books.lua#L1030)→[选名](../../../raw/current/scripts/components/book.lua#L64-L67)。def 未提供 `fxmount` 时骑乘**不自动拼 `_mount`**，而是回落基名并 `Transform:SetSixFaced()`（[回落分支](../../../raw/current/scripts/components/book.lua#L68-L71)）。

**路径 B——书本 `fn` 内按受效对象直接生成。** 与组件无关，是 def `fn` 的普通 SpawnPrefab：按各受效对象自身骑乘状态选名，位置/朝向取该对象；bees 用环形生成位、固定基名。均在各自成功判定通过之后执行。

| 书本 def（books.lua） | 路径 A：读者位置 | 路径 B：受效对象处 | 证据 |
|---|---|---|---|
| `book_moon` | moon／moon_mount | 无 | [def](../../../raw/current/scripts/prefabs/books.lua#L845-L873) |
| `book_research_station` | 无 `fx` 字段，不生成 | 范围内每名玩家：temperature 同款按玩家骑乘选名 | [deps](../../../raw/current/scripts/prefabs/books.lua#L938)、[生成](../../../raw/current/scripts/prefabs/books.lua#L946-L948) |
| `book_temperature` | 无 `fx` 字段，不生成 | 范围内每名玩家：按该玩家骑乘选名，落点/朝向=该玩家 | [deps](../../../raw/current/scripts/prefabs/books.lua#L716)、[生成](../../../raw/current/scripts/prefabs/books.lua#L725-L727) |
| `book_bees` | bees（骑乘回落基名六向） | 每个蜂卫生成位：固定 `fx_book_bees`，无骑乘变体 | [fx](../../../raw/current/scripts/prefabs/books.lua#L881)、[生成](../../../raw/current/scripts/prefabs/books.lua#L906-L908) |
| `book_fire` | fire／fire_mount | 无 | [def](../../../raw/current/scripts/prefabs/books.lua#L618-L619) |
| `book_light` | light（骑乘回落基名六向） | 无 | [fx](../../../raw/current/scripts/prefabs/books.lua#L761) |
| `book_light_upgraded` | light_upgraded（骑乘回落基名六向） | 无 | [fx](../../../raw/current/scripts/prefabs/books.lua#L787) |
| `book_birds` | birds／birds_mount | 无（鸟由 birdspawner 生成） | [def](../../../raw/current/scripts/prefabs/books.lua#L277-L278) |
| `book_rain` | rain／rain_mount | 无（下雨由天气系统结算） | [fx](../../../raw/current/scripts/prefabs/books.lua#L813-L814) |
| `book_sleep` | sleep／sleep_mount | 每个受效目标：按**目标**骑乘选名，落点/朝向=目标 | [fx](../../../raw/current/scripts/prefabs/books.lua#L387-L388)、[生成](../../../raw/current/scripts/prefabs/books.lua#L422-L424) |
| `book_fish` | fish（骑乘回落基名六向） | 无（鱼群由 schoolspawner 生成） | [fx](../../../raw/current/scripts/prefabs/books.lua#L562) |

## 关联机制导航

| 类型／对象 | 本 Prefab 如何被使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议：`prefabs/fx.lua` | 18 条目逐项 `MakeFx` 注册；代理＋本地实体分工、animover 自删、代理 1 秒回收、`fx_spawned` 事件 | 主模拟建代理并回收；各非专服端（含主机）建本地实体；专服不建本地表现；本地实体无 Network、`persists=false` | [generic-fx-factory#关键流程](../mechanisms/generic-fx-factory.md#关键流程)、[加载器](../../../raw/current/scripts/prefabs/fx.lua#L165-L172) |
| 组件：book | `SetFx(fx, fxmount)` 存名，`OnRead` 成功后选名生成（路径 A） | 主模拟；`reader.components.sanity` 存在才走默认结算 | [[reference/components/book#关键接口]]、[SetFx](../../../raw/current/scripts/components/book.lua#L21-L24) |
| 组件／动作：reader 与 ACTIONS.READ | READ 动作 → `Reader:Read` → `Book:OnRead` | `READ = Action({mount_valid=true})`，fn 在施法结算（路径窗口）内 | [[reference/components/reader]]、[动作](../../../raw/current/scripts/actions.lua#L360)、[fn](../../../raw/current/scripts/actions.lua#L1328-L1333) |
| SG：SGwilson `book`／`book_repeatcast`／`book2` | READ 的 ActionHandler 只选 `book`（cookbook／翻阅另选）；`book_repeatcast` 另有 CASTAOE 施法入口；book2 第 44 帧只准备 `fn50`，普通正式 READ 在第 50 帧才 `PerformBufferedAction()` 结算，`repeatcast` 则在 44 帧立即执行 → 上两条路径都发生在这之后 | 主模拟状态；骑乘由 `mount_valid` 与组件内 `IsRiding` 判定 | [READ 选状态](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1046-L1051)、[repeatcast 入口](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1316-L1320)、[44 帧准备](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10267-L10306)、[50 帧结算](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10307-L10311)；手持书动画链细节见 [[reference/prefabs/book_fx]] |
| 生成者数据：books.lua def | `fx`/`fxmount`/`deps` 字段与 `fn`；`MakeBook` 把字段并入 prefabs 数组并 `SetFx` | 仅依赖声明不生成；生成点见上节两路径 | [MakeBook](../../../raw/current/scripts/prefabs/books.lua#L963-L977)、[挂载](../../../raw/current/scripts/prefabs/books.lua#L1030) |
| 关联 Prefab：`book_fx` 族 | 读书动作中挂在读者身上的**手持书本动画**，与本法术演出分层并存 | SGwilson 按书本 `shadowmagic` 标签与骑乘选名，`SetParent(读者)` 父挂 | [[reference/prefabs/book_fx]] |
| 关联 Prefab：`fx_<名>_over_book`／`_under_book` 族 | books.lua `MakeFX` 独立工厂（bank `fx_books`），book2 按 def `fx_over`/`fx_under` 生成的前后景层；本批 18 名不属于该族（fish 书另有 `fx_under = "fish"`） | 骑乘追加 `_mount` | [MakeFX](../../../raw/current/scripts/prefabs/books.lua#L1054-L1078)、[fish fx_under](../../../raw/current/scripts/prefabs/books.lua#L563)；该族 Wiki 待建 |
| 资源 | 每名 `Asset("ANIM", "anim/<基名>.zip")` 由工厂按 build 声明 | 声明已核验，zip 包内容未解包 | [资产生成](../../../raw/current/scripts/prefabs/fx.lua#L7-L16) |

## 典型接入

### 让自定义书复用本族特效

- 前提与选择：自定义书走原版读书动作（READ → book 组件）时，只需配 book 组件的回调与 FX 名，**不要自己 SpawnPrefab 读者落点特效**——成功后组件自动生成。
- 生效与撤销：`SetFx(fx, fxmount)` 两参均可省略 mount；省略时骑乘回落基名并强制六向。施法失败（回调返回 false）不生成，也不扣次数。
- 原版依据：原书即按 [def → SetFx](../../../raw/current/scripts/prefabs/books.lua#L1030) 配置；[回落分支](../../../raw/current/scripts/components/book.lua#L68-L71)。

```lua
-- 自定义书 prefab 的 master 分支（TheWorld.ismastersim 之后）：
inst:AddComponent("book")
inst.components.book:SetOnRead(function(inst, reader)
    -- 施法效果与成功判定写在这里；返回 false 则本族 FX 不生成
    return true
end)
inst.components.book:SetFx("fx_book_fire", "fx_book_fire_mount")
-- 成功后读者位置生成 fx_book_fire；骑乘时换 fx_book_fire_mount（六向）
```

- 调整与验证：替换点是 `SetFx` 两个字符串参数（须为已注册 FX 名）；读者落点式生成由组件完成，模组无需处理定位。验证入口为 components/book.lua `OnRead` 与本文注册表。

## 常见陷阱与限制

- **误区：`deps`／prefabs 数组等于生成**：`book_temperature`、`book_research_station` 的 def 只有 `deps` 声明、没有 `fx` 字段，读者位置**不生成**特效；实际生成点全在 `fn` 内按受效玩家逐个执行。[deps vs fx](../../../raw/current/scripts/prefabs/books.lua#L971-L977)
- **边界：无 `fxmount` 的书骑乘不拼名**：bees／fish／light／light_upgraded 四名没有 mount 注册，骑乘时回落基名并 `SetSixFaced()`；不存在 `fx_book_bees_mount` 等 prefab。[回落分支](../../../raw/current/scripts/components/book.lua#L68-L71)
- **边界：失败窗口不生成**：路径 A 只在 `Interact` 成功后执行；路径 B 的直接生成位于各自判定（睡眠目标、蜂群上限、玩家范围）通过之后——判定失败返回 false 的窗口内两条路径都不出特效。[OnRead](../../../raw/current/scripts/components/book.lua#L61-L80)
- **边界：fish 渐隐不是实体移除计时**：`fx_book_fish` 的专属 fn 用 `SetMultColour` 把 alpha 从 1 每模拟帧递减 1/30（`DoPeriodicTask(0, …, 0.75)` 延迟 0.75 秒后起步），并设 `GroundOrientation`＋`OCEAN_BOAT_BUMPERS` 排序；实体清理仍按工厂协议 `animover` 自删。[fn](../../../raw/current/scripts/fx.lua#L3075-L3085)
- **边界：音效归属本地 fx 实体**：`wickerbottom_rework/book_spells/*` 由 fx 实体自己的 SoundEmitter 播放；专用服务器没有本地实体，不播也不计失败反馈。失败音效 `book_spells/fail` 走 [[reference/prefabs/book_fx]] 族的读者侧路径，与本族无关。[声音分支](../../../raw/current/scripts/prefabs/fx.lua#L43-L49)
- **辨析：三族同名前缀不同归属**：本页 18 名是 fx.lua 通用表法术演出；`book_fx` 族（book_fx.lua 工厂）是手持书动画；`fx_<名>_over/under_book` 族（books.lua MakeFX）是 book2 前后景层。三者同现于一次读书动作，互不替代。

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。已核验：18 条目逐名（fx.lua L2926-L3085：bank/build/anim/sound/sixfaced/fn 逐项）；工厂协议（prefabs/fx.lua 代理/本地实体/animover/sixfaced/资产分支）；11 本 def 的 fx/fxmount/deps/fn 逐项（books.lua，含 book_rain 路径 A 专属）；components/book.lua SetFx/OnRead 选名与回落；components/reader.lua `Read` 分派；actions.lua READ（L360、fn L1328-L1333）；SGwilson READ ActionHandler 只选 `book`（L1046-L1051）、`book_repeatcast` 的 CASTAOE 入口（L1316-L1320）与 book2 第 44 帧准备／第 50 帧 `PerformBufferedAction`（L10267-L10311）。第 6 轮复核明确普通正式 READ 的结算在第 50 帧，44 帧只是准备回调。全域反查范围：raw/current/scripts 根（含 prefabs/、components/、stategraphs/ 与根级文件）`rg "fx_book_"`（除 fx.lua 定义外命中仅 books.lua、book_fx.lua 的 `fx_book_waxwell` build 名、book_web_ground.lua 的 `fx_book_web`，后两者非本族注册名）；`"fx_" ..` 等动态拼名检索无本族名产生；`languages/chinese_s.po` 与 `strings.lua` 检索无 NAMES 显示名键。未覆盖：动画 zip 包内容、各 anim 实际帧数/时长、客户端预测期表现。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-character-index|角色技能、施法与读书]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 4 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-10）。
