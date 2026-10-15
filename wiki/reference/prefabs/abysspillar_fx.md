---
title: "深渊石柱坍塌与重建特效（abysspillar_fx）"
created: "2026-11-02"
updated: "2026-11-02"
type: prefab
summary: "深渊石柱坍塌动画与试炼重建动画共享的联网实体；区分纯坍塌和可恢复实体石柱的形成状态。"
aliases: ["abysspillar_fx", "深渊石柱特效", "深渊柱坍塌", "深渊柱重建"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/abysspillar.lua", "prefabs/abysspillar_trial.lua", "prefabs/lightsout_trial.lua", "components/abysspillargroup.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 深渊石柱坍塌与重建特效（abysspillar_fx）

## 快速判断

- `abysspillar_fx` 是专属工厂注册的联网动画实体，不是 `fx.lua` 通用表条目。默认坍塌表现会回收；试炼调用 `StartForming` 后，它还能在结束时生成真正的 `abysspillar` 并移交组追踪，因此不能一概当作纯视觉对象。[工厂](../../../raw/current/scripts/prefabs/abysspillar.lua#L804-L844)、[结束处理](../../../raw/current/scripts/prefabs/abysspillar.lua#L749-L775)
- 坍塌时的平台销毁与实体坠落事件在原石柱上处理；重建时实体石柱由 FX 的服务端结束处理创建。组接口另见 [[reference/components/abysspillargroup#关键接口]]。[坍塌](../../../raw/current/scripts/prefabs/abysspillar.lua#L279-L315)
- 标题中文为根据源码用途核验的效果描述，非官方译名；`languages/chinese_s.po` 定向检索 `STRINGS.NAMES.ABYSSPILLAR_FX` 未命中。动画资源内部外观和时长未运行核验。

## 注册与构造

| 注册名 | 工厂及分界 | 字段与证据 |
|---|---|---|
| `abysspillar_fx` | `Prefab` → `fxfn`；主模拟端之后挂回调与状态 | Transform、AnimState、SoundEmitter、Network；bank/build `abyss_pillar`，初播 `collapse`，地下层，`NOCLICK`／`FX`；服务端初始 `persists=false`、状态 COLLAPSE。资源声明 `anim/abyss_pillar.zip`。[注册](../../../raw/current/scripts/prefabs/abysspillar.lua#L843-L844)、[资源](../../../raw/current/scripts/prefabs/abysspillar.lua#L1-L4)、[构造](../../../raw/current/scripts/prefabs/abysspillar.lua#L804-L839) |

## 关联机制导航

| 路径 | 条件、位置及行为 | 证据 |
|---|---|---|
| 石柱坍塌 | 原柱进入 COLLAPSE 时，未休眠、非 POPULATING 且可见才在原柱完整世界坐标生成 FX，播 fall 声并继承翻转；原柱随后推坠落事件、销毁平台对象并移除。 | [状态](../../../raw/current/scripts/prefabs/abysspillar.lua#L279-L315) |
| 两种试炼生成 | `abysspillar_trial` 与 `lightsout_trial` 的生成函数在非 instant 时先于 `(x,0,z)` 生成 FX；FX 未休眠则随机翻转并 `StartForming`，否则移除它并直接生成石柱；instant 也直接生成石柱。两者由各自组组件注册为生成回调。 | [柱试炼](../../../raw/current/scripts/prefabs/abysspillar_trial.lua#L73-L103)、[绑定](../../../raw/current/scripts/prefabs/abysspillar_trial.lua#L513-L515)、[熄灯试炼](../../../raw/current/scripts/prefabs/lightsout_trial.lua#L319-L340)、[绑定](../../../raw/current/scripts/prefabs/lightsout_trial.lua#L512-L514) |
| 延迟与上下层 | `StartForming` 的 delay 非空时先隐藏并启用持久化，延时转 FORMING；FORMING 播 `place_a`／`place_b`、分帧调整堆叠声，同时生成同名子实体，父挂并设 FORMING_ABOVE 世界层。状态变化时移除旧 abovefx。 | [启动](../../../raw/current/scripts/prefabs/abysspillar.lua#L739-L746)、[状态与声音](../../../raw/current/scripts/prefabs/abysspillar.lua#L668-L736) |
| 完成与组追踪 | `animover` 或实体休眠调用 `fx_Finish`；POPULATING 时直接返回；COLLAPSE／FORMING_ABOVE 移除自身；FORMING 或休眠的 FORMING_DELAY 会生成实体石柱、复制位置和翻转，有有效 group 时把追踪从 FX 转到新柱，最后移除 FX。 | [结束](../../../raw/current/scripts/prefabs/abysspillar.lua#L749-L775)、[组追踪](../../../raw/current/scripts/components/abysspillargroup.lua#L51-L80) |
| 存档恢复 | 形成及延迟状态设置 `persists=true`；保存 group GUID。加载时若 GUID 在 `ents` 有记录，或存档未指定 group，就生成实体柱；若指定 GUID 却无记录则不生成。有记录时直接访问组组件，未检查实体有效性或组件存在；复原柱翻转与 `idle_b` 各自重新随机，最后移除 FX。 | [持久化切换](../../../raw/current/scripts/prefabs/abysspillar.lua#L702-L735)、[存取](../../../raw/current/scripts/prefabs/abysspillar.lua#L778-L802) |

## 常见陷阱与限制

- `abysspillar_fx` 是联网实体而非通用表的代理＋本地动画工厂；不能把其结束条件套成通用工厂的一秒代理回收。[构造](../../../raw/current/scripts/prefabs/abysspillar.lua#L804-L839)
- FORMING 在生成子 FX 的同时仍持有生成实体柱和组追踪转换的职责；跳过动画或直接移除它可能跳过这些处理。试炼的 instant／休眠分支则直接建石柱，不需要等待 FX。[结束](../../../raw/current/scripts/prefabs/abysspillar.lua#L749-L775)、[试炼生成](../../../raw/current/scripts/prefabs/lightsout_trial.lua#L319-L340)

## 验证记录

- 源码审阅：2026-11-02，Steam buildid 26335394；核验注册、工厂、状态／端／声音／存档及两个试炼的生成回调；对整个 `raw/current/scripts` 检索完整名 `abysspillar_fx` 和 `StartForming(`，区分依赖声明与实际生成，并追到组组件的生成与追踪。未核验动画包内部、模组动态调用和游戏运行。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-world-index|特效 Prefab 索引·环境与世界]]。
