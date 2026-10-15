# 知识仓项目文档

用于人类和按需工程建设；不是日常查询或知识增修的前置。游戏知识在 wiki，安装 Skill 中保存执行规则，工具／类型文档随工具箱。

| 问题 | 入口 |
|---|---|
| 定位、安装数据与使用边界 | [README](../README.md) |
| 建设进度、暂停状态与批次 | [PLAN](../PLAN.md) |
| 当前 98 项范围及完成后索引重整提醒 | [阶段检查点](reports/w05/component-priority-checkpoint.md) |
| 四仓协作、数据流与当前能力 | [总体设计](design.md) |
| 自包含、路径与生成物取舍 | [四仓决策](decisions/repository-split.md)／[决策索引](decisions/index.md) |
| 分仓实际检查与限制 | [迁移验收](reports/repository-split.md) |
| 新物品／修改任务的受控检视 | [任务检视](reports/w05/tasks.md) |
| 已有骨架观察与验证 | [手部](reports/w05/hand-equipment.md)／[身体](reports/w05/body-armor.md)／[头部](reports/w05/head-equipment.md) |
| 已交付组件批次 | [核心](reports/w05/component-core.md)／[环境](reports/w05/component-environment.md)／[制作](reports/w05/component-crafting.md)／[成长](reports/w05/component-growth-following.md)／[计时](reports/w05/component-timed-effects.md)／[潮湿与冰冻](reports/w05/component-moisture-freezing.md)／[工作量与使用消耗](reports/w05/component-work-efficiency.md)／[睡眠与困倦](reports/w05/component-sleep-drowsiness.md)／[治疗与修复](reports/w05/component-healing-repair.md)／[投射物与爆炸](reports/w05/component-projectiles-explosion.md)／[定向技能与冷却（含导航拆分）](reports/w05/component-targeting-cooldowns.md)／[科技与建造接入](reports/w05/component-building-tech.md)／[烹饪与干燥](reports/w05/component-cooking-drying.md)／[打包与拆包](reports/w05/component-bundling.md)／[传送与距离触发](reports/w05/component-teleport-proximity.md)／[农作与施肥基础](reports/w05/component-farming-basics.md)／[播种、饮水与压力](reports/w05/component-farming-growth.md)／[世界土壤管理](reports/w05/component-farming-manager.md)／[补水与浇水工具](reports/w05/component-watering-tools.md)／[枯萎保护、授粉与树木促进](reports/w05/component-plant-care.md)／[植物与肥料研究](reports/w05/component-plant-research.md)／[旧农场种植路径](reports/w05/component-legacy-farming.md)／[产物累积与周边再生](reports/w05/component-production-regrowth.md)／[养分显示](reports/w05/component-nutrients-visuals.md)／[巢穴与子体](reports/w05/component-nests-home.md)／[位置记忆与生成模式](reports/w05/component-location-spawning.md)／[群体协作](reports/w05/component-herds.md)／[发情机制](reports/w05/component-mood.md)／[驯化、骑乘与拴牛（含五批测时）](reports/w05/component-riding-hitching.md)／[装扮、刷剃与毛发生长（含五批测时）](reports/w05/component-grooming-hair.md)／[宠物、行为记录与巢穴装饰（含五批测时）](reports/w05/component-pets-decoration.md)／[激活、定向使用与建筑持续交互（含三批测时）](reports/w05/component-activation-channeling.md)／[持续施法与物品使用协议（含五批检查点）](reports/w05/component-channelcast-controls.md)／[阅读、题字与演奏（含五批检查点）](reports/w05/component-reading-writing-music.md)／[传送与魔术工具（含五批检查点）](reports/w05/component-teleport-magic-tools.md)／[钥匙、目标锁与怀表拆解](reports/w05/component-keys-locks-dismantling.md)／[机器、节点连接与标记代理](reports/w05/component-machines-circuits-marks.md)／[陷阱、诱饵与诅咒物品](reports/w05/component-traps-bait-curses.md)／[通用升级与电路模块](reports/w05/component-upgrades-circuit-modules.md)／[套装与伤害抗性](reports/w05/component-setbonus-damage-resistance.md)／[格挡、控制与伤害记录](reports/w05/component-parry-control-damage-tracking.md)／[通用基础：作祟、说话与物品环境](reports/w05/component-common-foundations.md) |
| 通用组件 C04～C06 | [保鲜材料、堆肥、容器代理／安装物与充电](reports/w05/component-preservation-containers-batteries.md) |
| 通用组件 C07～C09 | [物品存储、食物记忆／供料与地皮工具](reports/w05/component-storage-food-terrain.md) |
| 本轮通用组件 C10～C12 | [鸟笼占用、火情检测／扇风与鉴评／宰杀](reports/w05/component-occupation-fire-appraisal.md) |
| 本地模组观察的版本与文件指纹 | [来源复核](reports/w05/local-mod-references-20261017.md)，仅追溯输入时读取，未分发模组源码 |
| 剩余候选与页面覆盖 | [路线盘点](reports/w05/component-roadmap.md)／[覆盖清单](reports/w05/component-coverage.csv)，不进入知识检索 |

本仓工程规则见 [维护入口](../references/project-maintenance.md)。原 W03／W04和检索工具报告已迁到工具箱 docs/reports，历史正文／命令／路径保持，当前使用按各仓 README。报告与原 wiki/log 旧路径通过分仓决策解释，不能当当前可执行路径；历史原文不因迁移改成新版本结论。

## 工程文档归属

本仓 README／PLAN 只描述知识资产、覆盖与内容复查；各 Skill 的安装／流程／包设计和工具箱的实现／契约／类型协作由各自 README、PLAN、docs/design.md 维护。总体设计只保留跨仓关系。拆分前的混合计划见 [历史档案](history/README.md)，不进入知识检索或当前任务必读链。

## 2026-10-14 — Skill 规则目录调整

知识库 Skill 的执行文档现按 setup／query／maintain 归类；以下链接用于当前开发 checkout 的工程导航，安装后的 Skill 仍从自己的 SKILL.md 直达流程，不依赖相邻仓库。旧决策、报告、历史计划与日志保留原文，其中旧路径按记录时的布局理解，当前入口见下表。

| 历史 Skill 路径 | 当前入口 |
|---|---|
| references/paths.md、first-use.md、prepare-source.md | [路径](../../dst-wiki-skill/references/setup/paths.md)／[首次使用](../../dst-wiki-skill/references/setup/first-use.md)／[源码准备](../../dst-wiki-skill/references/setup/prepare-source.md) |
| references/query.md、query-source.md、search-wiki.md | [查询](../../dst-wiki-skill/references/query/query.md)／[源码索引查询](../../dst-wiki-skill/references/query/query-source.md)／[Wiki 工具](../../dst-wiki-skill/references/query/search-wiki.md) |
| references/maintain-skeleton.md | [模组经验提炼](../../dst-wiki-skill/references/maintain/extract-mod.md)／[骨架维护](../../dst-wiki-skill/references/maintain/maintain-skeleton.md) |
| references/maintain-version.md | [版本复查](../../dst-wiki-skill/references/maintain/maintain-version.md) |

知识页和导航的路径保持原样，格式唯一来源仍是 references/maintain/write-format.md；本次仅调整规则存放与加载分工，不升级知识版本或既有验收结果。
