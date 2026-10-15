---
title: 知识库结构与建设顺序
created: 2026-10-10
updated: 2026-10-13
type: decision
summary: 最小常驻入口，Skill 内使用规则与模板，仓库 references 放工程参考，docs 留设计与记录；类型核验在 fork 维护
tags: [decision]
sources:
  - .claude/skills/dst-wiki/references/query.md
  - .claude/skills/dst-wiki/references/maintain/maintain-wiki.md
  - .claude/skills/dst-wiki/references/maintain/write-format.md
  - .claude/skills/dst-wiki/references/maintain/maintain-index.md
  - docs/design.md
  - .claude/references/project-maintenance.md
  - docs/decisions/layered-index-and-retrieval.md
  - .claude/skills/dst-wiki/references/maintain/verify-interfaces.md
  - .claude/skills/dst-wiki/references/prepare-source.md
  - scripts/tools/setup/setup_raw.ps1
  - PLAN.md
  - .claude/references/output-format.md
  - scripts/vendor/lsp-dst.json
  - scripts/README.md
  - https://github.com/YingSP/lsp-dst
  - https://github.com/cclank/lanshu-wiki-skill/tree/f235a8d734c4744ef63b763a2681c57c02297d87
---

# 知识库结构与建设顺序

2026-10-13 当前分发职责与安装方式以 [四仓决策](repository-split.md) 为准；下面保留此前取舍的演进依据，日常工作流由已安装 Skill提供。

这是项目建设决策，不是 DST 行为或接口核验记录。跨模块架构与数据流见 [项目总体设计](../design.md)，本页保留结构选择的理由和重议条件。

## 目标

服务于新增物品、装备、建筑以及修改已有机制。让 AI 按需求找到可复用方案，再按必要证据缺口查源码。

## 选择与理由

| 决策 | 理由 |
|---|---|
| 保留源码、知识、工具、入口与 fork 引用边界 | 正式核验输入在 fork，主项目仅 pin；生成物与源码快照分开维护 |
| cookbook 保存有限的通用任务骨架，机制案例与陷阱留具体页 | 造新东西可直接取得必要组装上下文，再按需读机制；完整机制只维护一份，项目专属骨架归模组 Wiki |
| Wiki 分 cookbook 任务骨架与 reference 机制参考，API 多页主题共用目录 | 根边界清楚，modinfo 等相关页集中维护；来源与验证不按目录推断，名称／章节直达不增加加载步骤 |
| CLAUDE 仅引入 AGENTS，AGENTS 只触发 Skill，SKILL 统一操作路由 | 常驻入口不重复维护查询／知识／工程分流；无关任务不读取 DST 规范，完整读取链的成本另测 |
| Skill references 保存使用规则，仓库 .claude/references 保存工程规则与技术参考，docs 保存总体设计、决策与验收记录 | 使用、建设与记录各有职责；工程参考不混入发布 Skill，普通查询不加载项目文档；技术约定只维护一份 |
| 知识维护与工程建设分路，查询／知识维护不读 README／PLAN，根仅含知识分类与有限高频快捷入口 | 分类限定检索避免项目历史进入上下文；Top 30 不变成全量列表或必读候选 |
| 查询与模组编码共用源码准备，按会话检查、按 build 跳过解压 | Wiki 无命中时有源码兜底；临时解压通过后才切换版本并保留旧快照 |
| 模板归 Skill 的 assets/templates，工具和测试保持 scripts | 模板随写作入口归位，各规则只维护一份；不为现有工具增加空包装或重复实现 |
| 模板采用五节基本骨架，细项按真实需要补充 | 快速判断、关键接口、典型接入、陷阱与验证先满足查询；不为填表复制类型、证据或创建无内容章节 |
| scripts 作为工具箱，fullmoon-dst 只承载 Rust 模块 | 源码准备、fork 维护与索引查询各有职责，工具按用途分类；Schema、测试数据与测试程序分开，克隆配置在克隆外 |
| 根分类＋Top 30 简短链接；主题／叶级索引与知识页按大小预算及实际需求拆分，已知名称直查 | 常用任务少走导航，同时限制根额外成本和单页规模；实际大小只在页面格式维护 |
| W01 契约 → W02 模板与基准 → W03 最小提取 → W04 双产物 → W05 真实任务 | 已核验源码特征先约束数据格式，再以两类任务检查结果；按收益扩展 |
| 独立维护 YingSP/lsp-dst，主项目固定 commit | 使用者直接克隆即可复用；类型与语义证据随 fork 保存，后续 Wiki 共用输入 |
| 查询不写知识页；源码准备是共用前置，会话与日志不作为默认知识候选 | 区分源码可用性与知识维护，只在实际准备新快照时记录日志 |

## 参考与适配

参考 [LanShu Wiki Skill](https://github.com/cclank/lanshu-wiki-skill) 的轻量路由、Schema 统一规则、先读后写、去重、源码锚点、增量更新及冲突记录。
审阅版本为 `f235a8d734c4744ef63b763a2681c57c02297d87`，具体参见其 [SKILL.md](https://github.com/cclank/lanshu-wiki-skill/blob/f235a8d734c4744ef63b763a2681c57c02297d87/SKILL.md) 和 [代码仓库 Schema](https://github.com/cclank/lanshu-wiki-skill/blob/f235a8d734c4744ef63b763a2681c57c02297d87/schema/wiki-code-repo-SCHEMA.md)。

本项目以 DST 编码为用途，不扩展个人知识库或多受众导览。查询不自动写知识页，源码准备共用；历史记录按维护需要读取；陷阱是否归档根据证据与替代内容，而非未引用时长。
W01–W04 已完成，W03 已交付 4,030 文件的 facts／索引及完整 116 项基准，W04 两份声明／组件页与 176 条 reviews 已发布。W05 已交付检索／章节读取、起步 cookbook 与三份 modinfo 页，当前范围见 [总体设计](../design.md) 和 [PLAN](../../PLAN.md)；完整任务收益与增量维护仍待验证。目录与检索规则是设计选择，token 收益仍需真实任务测量。

2026-10-10 在 W04 前将原本分散在根目录、docs 和提取器目录的使用规则与模板归入 Skill，工程说明按 contracts／tools／reports 分类，更新全部当前入口；历史日志保留原路径。此前“模板与规则放仓库公共目录”的布局造成 Skill 已有子目录空置，因此调整物理位置；工具与知识库仍由主项目提供，不声称 Skill 已能独立安装。

随后按读者与职责调整：工程执行规则移到仓库级 `.claude/references/`，项目设计决策归 `docs/decisions/`；进一步将 AI 开发工具时直接使用的输出契约、环境、基准和索引说明归入仓库 references，docs 保留总体设计、决策与验收报告，并链接同一份工程参考。是否曾被 AI 读取不作为归位标准；设计理由与历史报告可按需查阅，人类也可阅读工程参考。机器 JSON Schema 与测试输入继续在 scripts，游戏知识继续在 wiki，各处只维护一份。

fork 依赖配置归 `scripts/vendor/lsp-dst.json`，与被忽略的独立克隆 `scripts/vendor/lsp-dst/` 并列；主项目维护说明归 `.claude/references/lsp-dst.md`，不再单设 annotations 目录。配置纳入主仓库，克隆可重建；避免配置被克隆目录的忽略规则排除或混入 fork 自身的文件。迁移保留配置内容和固定 commit，不改变正式核验输入。

此前按主题收紧分类，将方案、案例与教程归具体页“典型接入”，问题归“常见陷阱与限制”，移除了仅有空索引／占位的独立分类和模板。现将该限制收窄：允许 cookbook 通用造物骨架，机制方案与问题仍在主要具体页维护一份并互链，需求措辞保留在摘要／别名／标题中。cookbook 首份已核验内容在 W05 落地时再建目录与导航；不恢复任意方案、独立陷阱或会话分类。详细归属与分批建设见 [cookbook 决策](shared-and-mod-wiki.md#cookbook-层的范围与建设)。

工具箱进一步按职责归位：Rust 核心与可执行 parser_probe 保留在 fullmoon-dst，Python／PowerShell 命令归 tools 的 setup／source／verify／maintenance，正式机器契约归 schemas，基准、参考 bundle 与独立 Lua 输入归 tests/fixtures，Python 测试归 tests/python。`scripts/requirements.txt` 统一 Python 依赖；[工具箱 README](../../../dst-toolbox/scripts/README.md) 保存用途与运行入口，详细格式和版本依据仍链接工程参考，不增加普通 Skill 的默认读取。fork 配置、克隆和现有生成物保持原位，迁移不改基准预期、Lua 原字节或数据格式。

2026-10-10 进一步收紧按需加载：日常维护只保留证据与增修步骤，页面格式／标签和索引组织拆成各自的条件文件；仅机器 facts、正式 reviews／类型补充、LuaCATS 或 fork 数据维护触发接口核验。组件模板只在新建或重组组件页时读，普通案例与场景问题增修不触发。工程入口移除目录职责表，统一由总体设计保存；PLAN 先列未完成工作，已交付项只保留摘要与依据。基础查询与常用经验增修无需 Python，工具箱独立仓库作为后续分发计划，当前未拆分。

2026-10-12 统一 Skill 入口：AGENTS 仅触发条件与 SKILL 链接，四种操作区分查询编码、开发后骨架整理、知识写入与检查，工程维护保留条件入口。references 根保存查询及准备／检索用法与 maintain-skeleton，maintain 子目录保存机制维护、格式、索引和接口核验；只调整路径与入口说明，证据、标签和大小规则保持。当前跨项目整仓克隆后引用 Skill，安装时推荐准备本地源码，日常仍按版本检查；不声称已有独立安装包，也不以入口变短证明 token 收益。

## 任务与机制两层布局

2026-10-11 将 cookbook 与 reference 作为 Wiki 的两个知识入口：前者保存从零组装的任务骨架，后者下分 components／prefabs／stategraphs／api，保存核验过的定义、行为、接入与限制。按用途分层；cookbook 的经验也核验接口，reference 的案例也可来自开发，不以目录代替来源或可信度。

API 按公共接入点组织模组声明、注册／Hook、公共函数和引擎接口；组件自身方法留组件页。多页主题单独成目录，当前 modinfo 保存 fields／configuration／game-modes 与一个短索引，不预建无内容的 global／network／modding 目录。根只列两个入口与少量快捷页，各层只维护导航；已知名称与 cookbook 扩展直达目标章节，不沿目录逐层加载。

迁移同步 Wiki／Markdown 链接、相对源码与类型链接、Skill 查询范围、工具分类与类型目录检查及缓存；保留 Lua 示例、源码相对 sources、版本／验证状态和章节锚点。知识页只登记一个所属叶级；跨分类关系正文互链。历史报告仅调整仍需可点击的文件链接，原结论与历史命令不改，日志保留原字节。

## 何时重新考虑

- 单一分类出现难以匹配的真实需求时，调整摘要、别名或分类。
- 多个方案重复维护同一实现时，抽取可复用部分。
- W02 / W03 的试点暴露无法表达的源码特征时，再迭代契约与最小索引；不直接扩展完整图分析。
- W05 检索测量出现大索引读取或候选召回不足时，调整主题拆分、摘要、别名和排序；简单文件索引不足再评估其他存储与检索方式。
- 整仓显式引用不足以满足跨项目需求时，再评估自动发现与独立安装包；fork 的直接克隆复用已可用，正式 release 另按需求建设。

## 相关页面

- [分层索引与按需检索](layered-index-and-retrieval.md)：详细设计、工具边界及 W03／W05 分工。
- [通用与模组 Wiki 的经验归属](shared-and-mod-wiki.md)：cookbook 通用任务骨架与项目专属实现的边界。
- [知识根导航](../../wiki/index.md)：从具体分类进入页面及案例章节。
