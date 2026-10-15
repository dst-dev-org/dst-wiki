---
title: "麋鹿鹅巢电弧特效（moose_nest_fx_idle）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "麋鹿鹅蛋（mooseegg 巢体）的电弧动画：网络壳＋非专服本地实体的代理模式，idle 随机循环触发、hit 受击触发；无玩法组件。"
aliases: ["moose_nest_fx_idle", "moose_nest_fx_hit", "moose_nest_fx", "麋鹿鹅巢特效", "麋鹿鹅蛋电弧", "nest electric fx"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/moose_nest_fx.lua", "stategraphs/SGmooseegg.lua", "prefabs/mooseegg.lua", "languages/chinese_s.po"]
size_class: simple
reliability: verified
validation: source-reviewed
status: active
---

# 麋鹿鹅巢电弧特效（moose_nest_fx_idle）

## 快速判断

- 职责与适用：麋鹿鹅巢（mooseegg，官方译名"麋鹿鹅蛋"／"麋鹿鹅巢"，[MOOSEEGG1](../../../raw/current/scripts/languages/chinese_s.po#L345275-L345277)、[MOOSENEST1](../../../raw/current/scripts/languages/chinese_s.po#L345285-L345287)）巢体上的电弧动画：`idle` 有蛋时随机循环触发、`hit` 巢受击时触发一次。纯表现：无玩法组件，声音与动画都在本地实体上。[注册](../../../raw/current/scripts/prefabs/moose_nest_fx.lua#L63-L64)、[触发](../../../raw/current/scripts/stategraphs/SGmooseegg.lua#L63-L74)
- 优先使用：查"麋鹿鹅巢电击动画／代理模式本地特效"选本页；宿主巢体状态见 SGmooseegg。
- 重要副作用：仅表现；专用服务器不生成本地动画（源码注释写明），仅存在 1 秒的网络壳。[专服分支](../../../raw/current/scripts/prefabs/moose_nest_fx.lua#L40-L47)
- 修改边界：本地动画 `idle`／`hit` 均未显式传循环参数，播完 animover 即 Remove；网络壳 persists=false、1 秒后自删，不入存档。[壳生命周期](../../../raw/current/scripts/prefabs/moose_nest_fx.lua#L55-L56)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `moose_nest_fx_idle` | Make("idle") → fn：Transform＋Network → FX 标签 → 非专服延迟 1 帧起 PlayFX 本地实体 → SetPristine → 客户端壳早退 → 主模拟 persists=false、1 秒 Remove | 本地实体非网络、不睡眠：SetFromProxy 定位、bank/build `goosemoose_nest_fx` 播 `idle`、FinalOffset 3、`egg_electric` 音效、bloom。[工厂](../../../raw/current/scripts/prefabs/moose_nest_fx.lua#L31-L61)、[本地实体](../../../raw/current/scripts/prefabs/moose_nest_fx.lua#L6-L29) |
| `moose_nest_fx_hit` | 同工厂 Make("hit")，仅传给本地实体的动画名不同 | 与 idle 共用同一动画 zip 与音效。[注册](../../../raw/current/scripts/prefabs/moose_nest_fx.lua#L64) |

两注册名均无 NAMES 中文条目；标题为已核验效果描述（宿主"麋鹿鹅蛋／麋鹿鹅巢"为官方译名）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：SGmooseegg idle_full | doeffect 落 `moose_nest_fx_idle` 于巢位 (x, 0.1, z)，随后 `DoTaskInTime(math.random()*10, doeffect)` 自续；状态退出取消任务 | 主模拟起调度；本地动画在非专服各自生成 | [idle_full](../../../raw/current/scripts/stategraphs/SGmooseegg.lua#L59-L72)、[退出清理](../../../raw/current/scripts/stategraphs/SGmooseegg.lua#L84-L88) |
| 生成者：SGmooseegg hit | 巢受击进入 hit 态时落一次 `moose_nest_fx_hit`，animover 回 idle_full | 主模拟；每次受击一次 | [hit 态](../../../raw/current/scripts/stategraphs/SGmooseegg.lua#L104-L112) |
| 代理模式 | 网络壳只带 Transform＋Network；本地实体用 `Transform:SetFromProxy(proxy.GUID)` 对齐位置，动画／声音全在本地 | 专服零表现；客户端与主机各自渲染 | [专服分支](../../../raw/current/scripts/prefabs/moose_nest_fx.lua#L40-L47)、[本地实体](../../../raw/current/scripts/prefabs/moose_nest_fx.lua#L6-L29) |
| 依赖声明 | mooseegg prefab 依赖表声明两注册名 | 声明保证资源加载，不代表即时生成 | [依赖表](../../../raw/current/scripts/prefabs/mooseegg.lua#L8-L10) |

## 典型接入

仿"网络壳＋非专服本地实体"可在不增加网络实体渲染负担的前提下做定点循环特效：壳上只存动画名等参数，`DoTaskInTime(0)` 延迟 1 帧等定位稳定后再起本地表现。[延迟原因注释](../../../raw/current/scripts/prefabs/moose_nest_fx.lua#L42-L43)

## 常见陷阱与限制

- **边界：网络壳与可见动画不是同一实体**：服务端可以引用壳，但可见电弧是非联网本地实体，专服没有该视觉实体；不要把壳当成带 AnimState 的电弧。[壳生命周期](../../../raw/current/scripts/prefabs/moose_nest_fx.lua#L55-L56)
- **边界：idle 是自续任务不是动画循环**：`math.random()*10` 秒随机间隔由 SGmooseegg 的 fx_task 重新调度，动画本身只播一次；离开 idle_full 态即停止。[自续](../../../raw/current/scripts/stategraphs/SGmooseegg.lua#L69-L71)
- **边界：与蛋状态绑定**：idle 电弧只在 idle_full（有蛋）态出现，空巢 idle_empty 态不触发。[状态](../../../raw/current/scripts/stategraphs/SGmooseegg.lua#L59-L88)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；64 行工厂全文（壳／本地实体／两注册）、SGmooseegg idle_full 与 hit 态、mooseegg 依赖表、译名检索已核验；动画实际帧数、`egg_electric` 在专服是否可闻未核验（专服不生成本地实体，预期不可闻）。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
