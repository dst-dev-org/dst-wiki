---
title: "simplemagicgrower：连续推进到指定成长阶段"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "调用 growable 的自然成长路径并短延迟重试，达到配置阶段后恢复自然计时并清魔法成长标签。"
aliases: [simplemagicgrower, 连续魔法催熟, 树木魔法成长, magicgrowth 标签残留]
tags: [component, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/simplemagicgrower.lua, components/growable.lua, prefabs/evergreens.lua, prefabs/books.lua]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# simplemagicgrower：连续推进到指定成长阶段

## 快速判断

- 职责与适用：借 growable:DoGrowth 逐步推进到 last_stage；尚未到达时排0–1秒随机延迟重试，到达或超过时调用 growable:StartGrowing 并清 `magicgrowth`。不直接 SetStage，也不提供独立的成熟算法。[simplemagicgrower.lua:9-27](../../../raw/current/scripts/components/simplemagicgrower.lua#L9-L27)
- 优先使用：原版树木已配置并运行 growable、希望沿自然阶段回调连续推进时参考本组件；普通阶段时长／暂停控制见 [[reference/components/growable]]，不是只加 magicgrowable 标记就自动获得这条链。[evergreens.lua:893-902](../../../raw/current/scripts/prefabs/evergreens.lua#L893-L902)、[books.lua:66-80](../../../raw/current/scripts/prefabs/books.lua#L66-L80)
- 重要副作用：StartGrowing 先加 magicgrowth，再立即 Grow；Grow 缺依赖时直接返回而不清标签，忽略 DoGrowth 返回值，并且没有任务句柄、重复启动保护或取消接口。[simplemagicgrower.lua:1-29](../../../raw/current/scripts/components/simplemagicgrower.lua#L1-L29)
- 修改边界：原版常绿树在服务端添加并设末阶段；组件没有自己的端断言、存读或移除／休眠钩子。last_stage 必须与宿主阶段和推进条件匹配，不能把调用成功等同于已成熟。[evergreens.lua:866-868](../../../raw/current/scripts/prefabs/evergreens.lua#L866-L868)、[evergreens.lua:893-902](../../../raw/current/scripts/prefabs/evergreens.lua#L893-L902)

## 关键接口

| 接口 | 参数、返回 | 条件、副作用与证据 |
|---|---|---|
| `SetLastStage(last_stage)` | 目标阶段数；无显式返回 | 仅赋值，构造未赋默认值，不做可达性或上下限校验。[simplemagicgrower.lua:1-7](../../../raw/current/scripts/components/simplemagicgrower.lua#L1-L7) |
| `StartGrowing()` | 无参数／无显式返回 | 添加 magicgrowth 标签后同步调用 Grow，不校验自己是否已运行，不负责先启动自然 growable。[simplemagicgrower.lua:24-27](../../../raw/current/scripts/components/simplemagicgrower.lua#L24-L27) |
| `Grow()` | 无参数／无显式返回 | 缺 growable 或 last_stage 就返回；低于目标则 DoGrowth 并排下一轮，否则重启自然成长计时、清标签。[simplemagicgrower.lua:9-22](../../../raw/current/scripts/components/simplemagicgrower.lua#L9-L22) |

## 关键流程

### 推进前提与停止条件

growable:DoGrowth 在 targettime 和 pausedremaining 都为 nil（完全停止）时返回 false；运行或暂停过的成长才进入推进。它按下一阶段调用 pregrowfn，未设 growonly 时 SetStage，再在实体有效时调用 growfn 并重排／停止自然计时，最后返回 true。simplemagicgrower 不传 skipgrownfn，因此不是跳过自然阶段效果的纯状态赋值。[growable.lua:121-151](../../../raw/current/scripts/components/growable.lua#L121-L151)

DoGrowth 的 true 不保证 stage 增加：growonly 可跳过 SetStage，末阶段／循环由 GetNextStage 决定。本组件不检查返回值、进度或失败次数；完全停止或阶段不可达时可持续排任务却不完成。这是源码推导，未游戏复现。[growable.lua:93-101](../../../raw/current/scripts/components/growable.lua#L93-L101)、[growable.lua:121-151](../../../raw/current/scripts/components/growable.lua#L121-L151)、[simplemagicgrower.lua:15-20](../../../raw/current/scripts/components/simplemagicgrower.lua#L15-L20)

达到目标分支会调用自然 StartGrowing，尝试按当前阶段重新计时，再移除标签；不是让植物永久停在目标阶段，也不检查是否真的排出任务，后者可能因阶段表／时间条件返回。自然暂停及存读语义见 [[reference/components/growable#生命周期]]。[simplemagicgrower.lua:19-20](../../../raw/current/scripts/components/simplemagicgrower.lua#L19-L20)、[growable.lua:42-79](../../../raw/current/scripts/components/growable.lua#L42-L79)

## 典型接入

### 常绿树配置与林业书籍调用

常绿树服务端工厂先配置 growable.stages、初始阶段、循环／春季成长及 magicgrowable=true，再自然 StartGrowing；随后添加 simplemagicgrower，将 last_stage 设为 `#stages - 1`。其他植物应按自身阶段意义选择目标，不复制树的表达式。[evergreens.lua:893-902](../../../raw/current/scripts/prefabs/evergreens.lua#L893-L902)

`book_silviculture` 在读者30范围内按标签筛候选，其中 magicgrowth 被排除；随后分散调用 trygrowth。trygrowth 先拒绝失效、limbo 或枯萎对象，再在 growable 允许魔法成长或非树桩树类分支中优先调用本组件的 StartGrowing，并立即返回 true；否则才考虑 domagicgrowthfn 或单次 DoGrowth。书籍采用该返回值不代表魔法推进已完成，也不验证本组件 last_stage 是否配置。[books.lua:19-23](../../../raw/current/scripts/prefabs/books.lua#L19-L23)、[books.lua:49-80](../../../raw/current/scripts/prefabs/books.lua#L49-L80)、[books.lua:515-547](../../../raw/current/scripts/prefabs/books.lua#L515-L547)

## 常见陷阱与限制

- **缺组件／配置会自动退出干净**：StartGrowing 已加标签，Grow 的早退分支没有清理；若中途移走 growable，已排回调同样可早退并遗留标签。它又被林业／园艺书籍的候选过滤排除，修复依赖时需同时处理该状态，不能只把标签当成功标记。[simplemagicgrower.lua:9-27](../../../raw/current/scripts/components/simplemagicgrower.lua#L9-L27)、[books.lua:19-23](../../../raw/current/scripts/prefabs/books.lua#L19-L23)
- **标签是组件重入锁**：StartGrowing 不检查 magicgrowth 或已有任务；重复调用可各自立即推进并排出独立任务链。书籍候选标签过滤不替代组件自己的启动保护，尤其不能保证先前已排调用不会重入。[simplemagicgrower.lua:15-27](../../../raw/current/scripts/components/simplemagicgrower.lua#L15-L27)
- **停自然成长或删组件等于取消魔法任务**：本组件不保存 DoTaskInTime 句柄，也没有停止／移除钩子；停 growable 不取消这些回调，反而可能令后续 DoGrowth 一直返回 false。没有已核验的组件内取消协议，不能以移除它作为可靠撤销方式；组件也不保存 last_stage、任务或魔法过程，重建由宿主重新配置。[simplemagicgrower.lua:1-29](../../../raw/current/scripts/components/simplemagicgrower.lua#L1-L29)、[growable.lua:81-89](../../../raw/current/scripts/components/growable.lua#L81-L89)、[growable.lua:121-125](../../../raw/current/scripts/components/growable.lua#L121-L125)

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖组件全文、growable 的 DoGrowth 门禁／返回／回调和自然 StartGrowing、常绿树服务端配置、林业书籍候选与 trygrowth 分派；自然成长持久化复用同版本 growable 页。
- 游戏运行：未运行；未实测无进展重试、重入、多本书延迟调用、标签残留、暂停／存读或阶段回调改型；未审阅其他植物全部阶段和所有书籍效果；无新增 Lua 示例。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；未运行语法探针。

- 版本沿用：2026-10-28 批量沿用-70（哈希等价升版），见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/growable]]、[[reference/components/book]]。
