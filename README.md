# DST Wiki 知识资产

为 DST 模组开发提供共享 cookbook 骨架、原版机制／接口知识和源码证据。截至本轮合并，337 篇已登记组件、三篇 modinfo、五篇 cookbook，共 345 篇已登记知识页及 16 篇导航；本轮补齐料理记录／节庆大餐、飞鸟吸引／生成、幸运物品／幸运来源六页，第 92～110 组 43 页（含接收）已由另一 AI 复核登记；源码审阅与结构检查不等于游戏运行，准确 token 收益仍待真实任务测量。

## 使用与安装

本仓是数据，Skill 可安装在任意 harness 的 Skill 目录，无需与本仓相邻。普通开发安装 dst-mod-dev-skill；独立查询／维护共享知识安装 dst-wiki-skill。两者不互为前置，工程提取／批量校验才按需取得 dst-toolbox。

把本仓放到任意可访问位置，在用户主目录 `.dst-wiki/config.json` 填实际绝对路径，例如：

```json
{
  "knowledge_root": "D:/Code/dst-wiki-develop/dst-wiki"
}
```

可选 toolbox_root 指向独立工具箱。本次明确路径优先于用户配置；第一版不设项目覆盖配置。配置需使用者确认，不写入已安装 SKILL.md，更新 Skill 不覆盖个人设置。两个 Skill 内置同一定位约定，基础查询／案例维护无需 Python、Rust、LuaLS 或 fork。

首次使用可通过所用 Skill 的 scripts/setup_raw.ps1 -KnowledgeRoot <实际本仓绝对路径> 准备本地 Steam 源码；脚本不依赖 Python／工具箱，检查版本未变则跳过，更新先校验再保留旧快照。其他平台按 Skill 的快照约定人工准备；没有源码仍读 Wiki并明确无法核验范围。游戏源码只读，不提供下载或远程分发。

## 知识入口

- [知识根导航](wiki/index.md) 保留分类与有限 Top 30；已知名称可直达章节。
- [cookbook](wiki/cookbook/index.md) 提供通用任务骨架，扩展直达必要机制。
- [reference](wiki/reference/index.md) 当前提供 components 和 modinfo API；Prefab／状态机／内部机制随首篇有效内容再建分类，案例与陷阱留具体页。

开发先确认目标模组约定和实现；新增未知组装筛骨架，再按缺口查机制与源码，局部机制问题可直接定位。模组自身 Wiki 随模组项目，通用经验经明确整理进入本仓，不自动双写。

```text
dst-wiki/
├── wiki/             # 提交：知识与导航；updates/ 存版本更新总结；log 只追加；source-baseline.json 为源码哈希基线
├── raw/current/      # 本地：scripts 与 BUILD.txt；游戏源码不提交
├── raw/archive/      # 本地：保留旧快照
├── output/           # 本地：facts、索引和验收缓存
├── docs/             # 总体设计、决策、知识建设检视
├── references/       # 本仓工程维护规则
└── PLAN.md           # 知识建设进度与后续工作
```

## 工程建设

总体边界见 [设计](docs/design.md)，分仓与自包含依据见 [决策](docs/decisions/repository-split.md)，进度见 [PLAN](PLAN.md)。工具／类型验收和机器契约随独立工具箱；知识格式和模板随知识库 Skill。普通查询和知识增修不读这些工程入口。

主要宿主的查询／局部维护样本、前一百一十批及通用 C01～C48 已登记，存量修整和主题导航已完成。当前按当轮授权分批建设，记录实际来源、验证范围和缺口；覆盖与认领见 PLAN。历史报告保留，工具和 Skill 的工程计划由各自仓库维护，日常查询无需读取工程记录。

Full Moon AST 仅用于内部解析，默认不落盘；精简 facts／源码索引输出本地 output，可按源码／生产者／契约版本重建，Git 忽略。远程只保存值得维护的知识、规则、工具、版本配置和必要测试样本，不提交全量 AST／facts、游戏源码或本地产物。

不同 harness 的目录访问与真实自主路由仍待实际验收；不因文件分仓就宣称全面兼容。历史日志保留旧路径，历史报告按当时版本解释；当前命令以各包说明为准。

## 许可证

原创 Wiki、文档和示例采用 [CC BY-SA 4.0](LICENSE)：可商用，须署名并以同一许可共享改作。DST 游戏源码／资源、引用的游戏或第三方片段及依赖保留原授权，不随本仓授权转授。独立 lsp-dst fork 保留上游 MIT-0，正式 reviews 和声明仍在那里维护。外部贡献按 [贡献约定](CONTRIBUTING.md) 接收。
