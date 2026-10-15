---
title: "舞台剧木偶出场与退场特效（marionette_appear_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "舞台剧（playbill 剧目）演出时盖在演员身上的提线木偶线影：appear 正放＋loop 显式循环＋fader 渐隐，disappear 反向；时长由剧本 SetTime 控制。"
aliases: ["marionette_appear_fx", "marionette_disappear_fx", "marionette_fx", "木偶特效", "舞台剧木偶", "提线木偶特效", "marionette"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/marionette_fx.lua", "play_commonfn.lua", "play_the_doll.lua", "play_the_vault.lua", "prefabs/playbill.lua", "languages/chinese_s.po"]
size_class: simple
reliability: verified
validation: source-reviewed
status: active
---

# 舞台剧木偶出场与退场特效（marionette_appear_fx）

## 快速判断

- 职责与适用：舞台剧演出中盖在每位出场演员（cast member）身上的提线木偶线影：`marionette_appear_fx` 出场（appear→loop 循环→渐隐），`marionette_disappear_fx` 退场（loop 循环→dissappear 收尾）。纯表现：无玩法组件，时长由剧本行数据控制。[注册](../../../raw/current/scripts/prefabs/marionette_fx.lua#L106-L107)、[剧本调用](../../../raw/current/scripts/play_commonfn.lua#L84-L104)
- 优先使用：查"舞台剧木偶线影／演员变形演出效果"选本页；剧目清单见 playbill 与 play_the_* 剧本。
- 重要副作用：仅表现；appear 音效在公共段与主模拟各播一次（是否重叠未在游戏内核验）。[公共段](../../../raw/current/scripts/prefabs/marionette_fx.lua#L37)、[主模拟](../../../raw/current/scripts/prefabs/marionette_fx.lua#L46)
- 修改边界：`loop` 段**显式传循环参数 true**，会一直重复直到 fader 渐隐或切 disappear；实体 `persists=false` 不入存档。[appear 循环](../../../raw/current/scripts/prefabs/marionette_fx.lua#L36-L38)、[disappear 循环](../../../raw/current/scripts/prefabs/marionette_fx.lua#L87)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `marionette_appear_fx` | marionette_appear_fn：Transform＋AnimState＋SoundEmitter＋Network → FX 标签 → bank/build `marionette_fx` 播 `appear` 后 Push `loop`（显式循环）、alpha 0.8、appear 音 → SetPristine → 客户端早退 → 主模拟再播 appear 音、fader 组件、1 帧后默认渐隐（0.8→0，15 帧）、暴露 SetTime | SetTime 取消并重排渐隐起点；渐隐结束 Remove。[工厂](../../../raw/current/scripts/prefabs/marionette_fx.lua#L24-L56) |
| `marionette_disappear_fx` | marionette_disappear_fn：同框架，直接播 `loop`（显式循环）、alpha 0 → 主模拟 fader 淡入（0→0.8，15 帧）、1 帧后默认退场：播 `dissappear`（源码拼写）＋disappear 音、animover Remove、暴露 SetTime | SetTime 重排退场起点。[工厂](../../../raw/current/scripts/prefabs/marionette_fx.lua#L75-L104) |

两注册名均无 NAMES 中文条目；剧目名"附魔玩偶"为官方译名（[PLAYBILL_THE_DOLL](../../../raw/current/scripts/languages/chinese_s.po#L346385-L346387)），标题中"木偶"为已核验效果描述。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 剧本指令：marionetteon／marionetteoff | play_commonfn 对除鸟与旁白外的每个 cast member 延迟 0.1+0.4×rand 秒生成 FX 并 `SetTime(duration)`（时长取剧本行 time 或 duration） | 落在演员当前位置；SetTime 传的是残存时长而非绝对时间 | [marionetteon](../../../raw/current/scripts/play_commonfn.lua#L84-L93)、[marionetteoff](../../../raw/current/scripts/play_commonfn.lua#L95-L104)、[落点与 SetTime](../../../raw/current/scripts/play_commonfn.lua#L75-L82) |
| 使用剧目 | play_the_doll（附魔玩偶）、play_the_princess_yoth、play_the_vault 的多行剧本调用 marionetteon／off | play_the_vault 用 `time = MARIONETTE_TIME 1.1` 秒 | [剧本引用](../../../raw/current/scripts/play_the_vault.lua#L117-L129)、[依赖声明](../../../raw/current/scripts/prefabs/playbill.lua#L7-L9) |
| [[reference/components/fader#关键接口]] | 渐隐／淡入 alpha，经 fade_step 写 AnimState MultColour | 仅主模拟挂载；客户端看到的是网络实体复制后的颜色变化 | [渐隐](../../../raw/current/scripts/prefabs/marionette_fx.lua#L8-L22)、[淡入](../../../raw/current/scripts/prefabs/marionette_fx.lua#L97-L98) |
| 依赖声明 | playbill（节目单物品）按剧目声明两注册名 | 四个剧目各有依赖列表 | [依赖表](../../../raw/current/scripts/prefabs/playbill.lua#L19-L45) |

## 典型接入

仿剧本调用可为任意"临时变身"表现计时：生成 FX 后 `fx:SetTime(秒数)`，出场 FX 到点自动渐隐、退场 FX 到点自动播收尾并删除；不调 SetTime 时默认 1 帧后就开始渐隐／退场（几乎立即）。[默认任务](../../../raw/current/scripts/prefabs/marionette_fx.lua#L52)、[重排](../../../raw/current/scripts/prefabs/marionette_fx.lua#L16-L22)

## 常见陷阱与限制

- **误区：SetTime 传绝对时刻**：SetTime 是 `DoTaskInTime(time)`，即"再过多久开始渐隐／退场"；剧本传的 duration 同义。[SetAppearFadeTime](../../../raw/current/scripts/prefabs/marionette_fx.lua#L16-L22)
- **边界：loop 是显式循环**：与同批其他地面 FX 的单次动画不同，这里 `PushAnimation("loop", true)` 显式循环，FX 会持续存在到 fader 结束；复用时要自己保证收尾路径。[循环](../../../raw/current/scripts/prefabs/marionette_fx.lua#L38)
- **边界：退场拼写是 dissappear**：源码动画名即如此拼写（连续两个 s），引用动画时按源码拼写。[拼写](../../../raw/current/scripts/prefabs/marionette_fx.lua#L61)
- **边界：声音双份触发**：appear 音在公共段（双端）与主模拟 postinit 各调一次；实际听感是否重叠未核验。[公共段](../../../raw/current/scripts/prefabs/marionette_fx.lua#L37)、[主模拟](../../../raw/current/scripts/prefabs/marionette_fx.lua#L46)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；107 行工厂全文（两工厂全生命周期）、play_commonfn 剧本调用链、play_the_vault 时长常量、playbill 依赖表、译名检索已核验；三个使用剧目全剧本流程、音效重叠听感、演出中断时 FX 的清理路径未核验。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-event-index|节庆、小游戏与事件模式]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
