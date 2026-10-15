---
title: "读书施法书本特效（book_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "读书动作中挂在读者身上的书本特效：薇克巴顿/麦克斯韦两套、步行/骑乘双版本共六个注册名，联网 FX 由 SGwilson book 状态按书本标签与骑乘自动选型，施法结算失败播失败动画与音效。"
aliases: ["book_fx", "book_fx_mount", "waxwell_book_fx", "waxwell_book_fx_mount", "waxwell_shadow_book_fx", "waxwell_shadow_book_fx_mount", "读书特效", "施法书本特效"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/book_fx.lua", "stategraphs/SGwilson.lua", "prefabs/waxwelljournal.lua", "prefabs/books.lua", "prefabs/wickerbottom.lua", "prefabs/waxwell.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 读书施法书本特效（book_fx）

## 快速判断

- 职责与适用：读书动作（SGwilson `book` 状态）期间挂在读者身上的**手持书本动画实体**：联网 FX（AddNetwork），FinalOffset 3 前景层，各注册名带不同 MultColour 着色；六个注册名覆盖薇克巴顿/麦克斯韦两套书、步行/骑乘双版本。[工厂](../../../raw/current/scripts/prefabs/book_fx.lua#L14-L54)
- 优先使用：查"读书时手里的书／施法失败动画"选本页；生成与选型入口在 SGwilson `book` 状态，模组自定义书按标签自动获得。[选型](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10061-L10066)
- 重要副作用：失败路径——`book_fx`／`book_fx_mount` 监听 `fail_fx` 事件，播失败动画并由**读者**播放 `wickerbottom_rework/book_spells/fail` 音效；waxwell 系无 failanim 不监听。[OnFail](../../../raw/current/scripts/prefabs/book_fx.lua#L7-L12)
- 修改边界：动画工厂一次性单参播放（未显式传循环参数），尾部垫空帧（源码注释）撑过 book→book2 两状态后 animover 自删；被打断由 SG onexit 手动 Remove。[自删](../../../raw/current/scripts/prefabs/book_fx.lua#L50-L51)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `book_fx` | MakeBookFX 共同工厂：CreateEntity → Transform＋AnimState＋Network → FX 标签 → 四向（ismount=false）→ bank/build/anim → SetFinalOffset(3) → SetMultColour(tint) → SetPristine → 主模拟侧 persists=false、监听 fail_fx（有 failanim 时）＋animover Remove | bank/build `book_fx_wicker`、anim `book_fx_wicker`、failanim `book_fx_fail_wicker`、tint {1,1,1,.4}；薇克巴顿书默认特效。[注册](../../../raw/current/scripts/prefabs/book_fx.lua#L59-L60) |
| `book_fx_mount` | 同工厂 ismount=true → 六向 | anim `book_fx_wicker_mount`、failanim `book_fx_fail_wicker_mount`，其余同上；骑乘版本。[注册](../../../raw/current/scripts/prefabs/book_fx.lua#L60) |
| `waxwell_book_fx` | 同工厂 | bank/build `book_fx`、anim `book_fx`、无 failanim（不监听 fail_fx）、tint {0,0,0,1} 纯黑；麦克斯韦暗影魔典。[注册](../../../raw/current/scripts/prefabs/book_fx.lua#L61) |
| `waxwell_book_fx_mount` | 同工厂 ismount=true → 六向 | anim `book_fx_mount`，其余同上。[注册](../../../raw/current/scripts/prefabs/book_fx.lua#L62) |
| `waxwell_shadow_book_fx` | 同工厂 | bank/build `fx_book_waxwell`、anim `shadowmagic`、tint {1,1,1,.6}；book2 状态按 shadowmagic 标签额外生成的前景层。[注册](../../../raw/current/scripts/prefabs/book_fx.lua#L63)、[生成](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10163-L10167) |
| `waxwell_shadow_book_fx_mount` | 同工厂 ismount=true → 六向 | anim `shadowmagic_mount`。[注册](../../../raw/current/scripts/prefabs/book_fx.lua#L64) |

六个注册名均无 NAMES 条目；标题中文为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：SGwilson `book` 状态 | 按书本 `shadowmagic` 标签选 waxwell/book 系，`rider:IsRiding()` 追加 `_mount` 后缀，SpawnPrefab 后 `SetParent(读者)`；repeatcast 时 `SetFrame(总帧数+6)` 对齐续读 | 此处为主模拟 SG；FX 通过 Network 同步，不能把 SGwilson 当成预测客户端状态图 | [选型与生成](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10061-L10070)、[续读入口](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10031-L10036) |
| 标签来源：shadowmagic | `waxwelljournal` 实体挂该标签 → 读书换 waxwell 系特效；薇克巴顿书目无此标签走 book_fx | 书本实体标签，生成时读取 | [waxwelljournal](../../../raw/current/scripts/prefabs/waxwelljournal.lua#L634) |
| 生命周期：book → book2 | book 结束时把 `statemem.book_fx` 经 data 传入 book2 继续使用；book2 按帧跳对齐 | 打断时 book onexit Remove；book2 onexit 兜底 Remove | [移交](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10080-L10090)、[book2 兜底](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10354-L10356) |
| 失败路径：fail_fx 事件 | book2 第 44/50 帧把 fx 解绑、定位并按读者转向，`PerformBufferedAction` 失败时 `PushEvent("fail_fx", 读者)`；OnFail 播 failanim＋读者播失败音效 | 仅注册了 failanim 的 book_fx/book_fx_mount 生效；事件 doer 参数即读者 | [解绑与结算](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10276-L10298)、[OnFail](../../../raw/current/scripts/prefabs/book_fx.lua#L7-L12) |
| 关联 Prefab：`fx_<名>_over_book` 族 | 每本书的法术专属前后层（book2 从 `book.def.fx_over_prefab/fx_under_prefab` 生成）是**另一工厂族**（books.lua 生成），不在本页六个注册名内 | 骑乘时同样追加 `_mount` | [def 生成](../../../raw/current/scripts/prefabs/books.lua#L981-L989)、[生成点](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10132-L10144) |
| 资源 | anim/book_fx_wicker.zip、anim/book_fx.zip、anim/fx_book_waxwell.zip | 声明与使用已核验，包内容未解包 | [资产声明](../../../raw/current/scripts/prefabs/book_fx.lua#L59-L64)、[依赖声明](../../../raw/current/scripts/prefabs/wickerbottom.lua#L15-L16)、[依赖声明](../../../raw/current/scripts/prefabs/waxwell.lua#L17-L22) |

## 典型接入

### 让自定义书在读书时换用麦克斯韦特效

- 前提与选择：模组书走原版读书动作（SGwilson `book`）时**无需自己生成任何 fx**——状态按标签自动选型；想换成暗影书外观最直接的杠杆是给书本实体加 `shadowmagic` 标签。
- 生效与撤销：标签加在书本 prefab 的主模拟分支；移除标签即回到薇克巴顿系外观。注意该标签同时会让 book2 生成 `waxwell_shadow_book_fx` 前景层（[生成](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10163-L10167)）。
- 原版依据：waxwelljournal 即靠这一标签区分（[标签](../../../raw/current/scripts/prefabs/waxwelljournal.lua#L634)、[选型](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10061)）。

```lua
-- 自定义书 prefab 的 master 分支里：
inst:AddTag("shadowmagic") -- 读书特效换用 waxwell_book_fx 系（黑色书）＋前景暗影层
```

- 调整与验证：替换点只有该标签；步/骑版本由读者骑乘状态自动切换，模组无需干预。验证入口为 SGwilson 选型段与 book2 生成段。

## 常见陷阱与限制

- **误区：失败音效是 fx 播的**：OnFail 把 `wickerbottom_rework/book_spells/fail` 播在**读者**（事件 doer）的 SoundEmitter 上，fx 只负责失败动画。[OnFail](../../../raw/current/scripts/prefabs/book_fx.lua#L7-L12)
- **边界：waxwell 系未实现本工厂的失败表现**：四个 waxwell 注册名 failanim 为 nil，工厂不注册 fail_fx 监听；不能据此推断宿主其他逻辑也没有失败反馈。[注册](../../../raw/current/scripts/prefabs/book_fx.lua#L61-L64)
- **边界：正常路径的寿命由动画决定**：动画尾部垫空帧（源码注释 [L50](../../../raw/current/scripts/prefabs/book_fx.lua#L50-L51)）撑过 book→book2，animover 自删；SG 只在打断时手动 Remove（[book onexit](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10093-L10102)），不要依赖 SG 清理正常路径。
- **边界：步/骑是两套注册名**：骑乘换 `_mount` 后缀（六向），动画名与朝向都不同；生成侧不可混用。[选型](../../../raw/current/scripts/stategraphs/SGwilson.lua#L10062-L10064)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；65 行工厂全文（六注册名、OnFail、生命周期监听）、SGwilson book/book_repeatcast/book2 状态（选型、移交、失败路径、onexit 清理）、waxwelljournal 标签、books.lua fx_over/fx_under 生成区分、wickerbottom/waxwell 依赖声明、NAMES 检索（无命中）已核验；动画包内容、fail_fx 在客户端预测分支的实际表现、晚加入时序未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-1）。
