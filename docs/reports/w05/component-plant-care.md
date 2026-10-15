# 枯萎保护、授粉与树木促进组件检视

2026-10-18；Steam buildid 25643519。按 dst-wiki Skill 完成第二十批三页；复用同会话已核对的快照／安装结果，未再次解压。文本与临时 Lua 使用 apply_patch 写入／清理。

## 交付与边界

| 页面 | 核验与完整 modmain 示例 | 物理行／实际 UTF-8 字节 |
|---|---|---:|
| [witherable](../../../wiki/reference/components/witherable.md) | 温度／雨水、自然／强制枯萎、保护与睡醒存读；采草后保护一个游戏日 | 76／8,909 |
| [pollinator](../../../wiki/reference/components/pollinator.md) | 花实体去重／门槛、寻花回巢与无家造花；工蜂访问 3 个不同花实体达到收集阈值 | 68／6,954 |
| [treegrowthsolution](../../../wiki/reference/components/treegrowthsolution.md) | 动作／组件筛选、成长入口／结果／消费；拒绝无成长入口的目标而不扣材料 | 73／7,560 |

- witherable 的 Protect 延长截止时间并尝试复壮，不是永久免疫；ForceWither 可绕过 enabled／保护，自然检查、crop 复壮资格和普通贫瘠分别判断。示例保留原 grass 采收回调，不改冬季／肥力／产物，已有存档保护不会因停用立即消失。
- pollinator 使用不同花实体引用去重，HasCollectedEnough 为严格 >，默认 5 需要访问 6 个。FindFlower／Brain／SG 分别承担寻花、回家和无家落地造花；CreateFlower 自身不查密度，实际密度搜索也未要求 flower 标签。示例 collectcount=2 不承诺立即回家、花或蜂蜜增产。
- treegrowthsolution 先 FX、再优先目标回调／普通 DoGrowth、最后消费一份；忽略成长返回且无入口也可消费并 true。示例只补入口存在性，不承诺停止的 growable 或 false 回调不消费。oceantree 的阶段／高树进度与定时消费者单列，不把每份效果等同普通树加一阶段。
- 三者分别作用于植物保护、授粉者及材料，未合成“通用增产”接口。只核验必要宿主与消费者，未全面审校旧 crop、其他授粉生物、蜂箱、所有树种或榕树生命周期。

## 校验与读取

五个 Wiki 文件（三页、世界主题和组件分类）通过元数据、来源、标签、大小、本地链接及标题检查；全库导航 111 篇知识页、12 篇导航、22 条 Top 30，无遗漏、重复登记或父入口问题。章节职责人工审阅，维持五个主节，无模板或类型变更。

三页 86 处源码行引用、70 个唯一目标、15 份来源文件 strict 检查为 0 error／0 warning，truncated=false、omitted=0、by_page_omitted=0。三条组件 SHA-256 与覆盖 CSV 匹配；798 候选现为 103 已建／695 未建，另 19 replica／3 helper／1 empty 单列。

三份临时 Lua 与页面示例逐字一致，通过已有 Full Moon Lua 5.1 syntax-only；byte／position 检查按该模式跳过。不执行 Lua，不证明 API、资源或游戏行为；临时文件已删除，无依赖安装、编译或缓存重建。

作者受控读取三页“典型接入”，均包含快速判断、完整示例／条件和验证记录，complete_selected_ranges=true、truncated=false、omitted=0：

| 页面 | JSON stdout 实际 UTF-8 字节（换行按 LF） |
|---|---:|
| witherable | 5,094 |
| pollinator | 4,437 |
| treegrowthsolution | 5,423 |

世界主题新增三条唯一登记，为 93 行／6,821 字节；组件分类 25 行／1,414 字节。未知名称按需求进入本批小节，已知名称仍直达，无新增导航层级。此为作者选定页面／章节的受控读取，不是独立模型自主发现或 token 收益评测。

## 收尾与后续

同步覆盖 CSV、PLAN、README、design、docs 入口与追加日志；五份本批工程 Markdown 的 105 处本地引用／锚点及 git diff --check 通过。既有报告和日志保留原文，当前日志 138 行／20,159 字节，未超归档阈值；三份 batch20 临时 Lua 已清理。

下一批拟核验 plantresearchable／fertilizerresearchable／plantregistryupdater，按实际实现与宿主界定研究／登记协作。heatrock Prefab、配方注册 API 与后续 cookbook 队列保留，不等待所有组件完成。

未改 Skill、工具箱、游戏源码或输入模组，未提交／推送。页面保持 source-reviewed，游戏未运行；热浪／雨水／存读、采收循环、昼夜战斗与回巢、花密度、树阶段／高树进度及其他模组包装顺序待实测。
