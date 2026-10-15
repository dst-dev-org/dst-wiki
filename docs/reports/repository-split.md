# 四仓迁移验收

日期：2026-10-13。职责、路径、自包含与生成物策略见 [决策](../decisions/repository-split.md)。本轮暂停新知识生成，完成本地迁移与文件／命令验证。

## 已交付

四仓为 dst-wiki、dst-wiki-skill、dst-mod-dev-skill、dst-toolbox。原知识仓保留 Git 历史与 origin；另外三仓已有独立本地 Git。原 dst-mod-wiki-skill改名，不产生第五个仓。本轮未暂存、提交、创建远程或推送。

两个 Skill 根是可安装包，内部引用均留包内，知识／源码按用户 knowledge_root定位。开发 Skill 提供开发／调试、最小知识查询和本地 Wiki流程；知识库 Skill保留共享页面／骨架／索引维护。helper有唯一实现与显式副本同步／哈希清单，不要求两个 Skill 同时安装。

工具保留原 scripts布局，明确源码／Wiki／输出／格式参数。原 scripts/output迁到知识仓 output并忽略，AST默认不落盘；Schema、fixtures、锁版本和正式 fork pin继续版本控制，完整 fork克隆忽略。个人配置未代用户创建／修改。

## 检查结果

- 两个 Skill frontmatter／基础结构检查通过；搬到与开发目录无关的含空格安装目录后，内部链接有效。
- 独立 PowerShell 定位通过：配置指向另一含空格数据目录、显式 KnowledgeRoot优先且不读取损坏配置、配置缺失报错、源码和工具箱缺失仍返回明确可用状态。各安装副本执行 7 项模拟准备／跳过／更新归档／失败保留检查，共 14 项通过。
- Cargo --locked debug构建通过，Rust核心原代码未修改。完整 Python回归 134 项通过，含新增 5 项外部路径／优先级检查；页面与导航测试使用独立合成格式输入，生产校验明确读取唯一 write-format。
- 全部 56 篇知识页和 9 篇导航通过生产页面／source路径／标签／链接／登记／Top 30／大小检查；独立知识缓存重建通过，人工页面未改。
- 从与工具仓不同工作目录执行 Wiki search／read成功，缓存 current，完整选定章节输出，未截断；SetMaxUses查询输出 1,792 UTF-8字节，finiteuses接入章节 4,802字节，均非 token测量。
- 从异地工作目录显式提取真实 health.lua成功，facts complete，源码索引构建与 DoDelta精确查询通过；本地 output保留此次缓存，没有重跑全量源码或生成新 Wiki。
- 原基准 5个独立输入／8个试点的来源字节检查通过；契约 bundle核对通过，外部历史注解字节未在本轮重查。固定 fork的 176 条正式记录／源码字节检查通过；未修改注解、reviews或pin。
- 106 份受保护文件（知识页／导航、覆盖／历史报告、Schema、fixtures、Rust与版本文件等）原字节一致；wiki/log.md原 121,085字节前缀一致。现代文档本地链接与标题锚点检查通过，两个 Skill文件链接不越出安装包。

临时脚本、迁移前备份与详细检查结果在主工作目录 migration-local/2026-10-13，不属于四仓。可重建的 health／知识缓存在本仓忽略的 output/repository-split。

## 历史与后续

历史工具报告正文／命令原字节保留。旧 .claude/skills/dst-wiki对应知识庫 Skill根，旧 .claude/references的工具说明对应工具箱 references；scripts对应工具仓 scripts，setup随 Skill，scripts/output对应知识仓 output，raw／wiki仍归知识仓。旧相对链接描述当时布局，当前命令以各仓README为准。

真实 Claude Code／Codex等 harness自主任务路由与外部目录权限仍待实际验收；文件／脚本可移植不等于所有客户端行为已验证。普通 Markdown查询／增修规则已解除工程依赖，但本轮没有独立模型完整开发／维护任务评测、LuaLS复跑或游戏运行；准确 token收益未测。远程仓库创建、Git历史分发策略与 release版本固定留明确发布任务，组件生成保持暂停。
