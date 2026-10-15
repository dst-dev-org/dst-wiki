# 位置记忆与生成模式组件检视

2026-10-19；Steam buildid 25643519。按 dst-wiki Skill 完成第二十六批三页；复用同会话已确认快照，未再次解压。文本与临时 Lua 均通过 apply_patch 写入／清理。

## 交付与边界

| 页面 | 实际行数／UTF-8 字节 | 本批覆盖 |
|---|---|---|
| [knownlocations](../../../wiki/reference/components/knownlocations.md) | 70／7,308 | 命名坐标引用／覆盖／删除、序列化合并、宿主存读与家园消费者；猪人延迟记录自定义位置键 |
| [spawner](../../../wiki/reference/components/spawner.md) | 78／10,888 | 单住户接管／进出屋、失去补员、回调／排队／离屏、内外保存与外部计时；普通猪人房未来补员等待减半 |
| [periodicspawner](../../../wiki/reference/components/periodicspawner.md) | 78／10,502 | 随机增量、重排／保留、密度／间距、重试／回调返回、海上分派、单次追赶和存读；格罗姆后续间隔减半 |

选型与限制来自实际实现及必要宿主，不将相似名称等同机制：

- knownlocations 直接保存／返回位置对象引用；没有家园实体追踪或自动移动。DeserializeLocations 合并且覆盖同名项，空数据不保存。自定义键避免原 home，由零延迟任务在同步放置／存读之后补记，dont_overwrite 保留已恢复值；不承诺记住绝对历史出生点或适配任意更晚异步放置。
- spawner 在屋时保留真实子实体、挂载并 RemoveFromScene，出屋恢复；不同于 childspawner 的数量储备。宿主可改变状态，猪人房 onvacate 会重置变身触发并恢复生命，不能把保持同体说成冻结所有属性／组件任务。
- spawner 的 Configure 会立即排首轮，不是纯配置。OnChildKilled 汇合捕获／死亡／detachchild／移除；delay 用于未来补员，改字段不重排已保存／进行中的计时。外部桥接传入实际秒数，世界禁用仍影响计时；本例不改变 maxtime 或迁移保存比例。
- spawner 的 IsOccupied 只看父子关系，不表示无在外住户。排队须有 retryperiod；离屏开关约束定时／睡眠路径，直接 ReleaseChild 并不检查。保存包括在屋记录、在外 GUID，以及显式启用的非持久子体按记录保存分支，重载不保证临时字段／GUID 恒等。
- periodicspawner 用 base+random×variance，而非对称方差；SetRandomTimes 默认重排，no_reset=true 才保留当前任务。TrySpawn 返回 boolean 和可选重试秒数，不返回实体；末尾 true 也可能对应 nil 产物，ForceNextSpawn 不绕过门禁且会重启调度。
- 密度搜索中心在宿主，过滤函数与同 prefab 条件做 OR；单设 spacing 而无正 density 会拒绝。自定义远端落点不能假设候选范围覆盖完整。一般资格失败用 base×0.25，缺落点用 15 秒，密度失败回正常间隔。
- 周期生成只保存未来剩余秒数，不单独保存 Stop 状态；LongUpdate 到期只尝试一次，不循环补产。格罗姆例同时减半 base／randtime 并保留当前任务／读档剩余，原 OnSpawnFuel 与 goo 状态仍负责产物隐藏／显示。小牛为实际密度例；grassgator 中相似挂载在块注释内，没有认定为活跃宿主。

未全面审校 worldsettingstimer 存档比例／版本迁移、flotsamgenerator 内部、所有生成器宿主、完整 Brain／SG、自定义异步放置或分片迁移；没有为缺页创建空导航。

## 校验与读取

五个 Wiki 文件（三新页、世界主题和组件分类）通过格式、元数据、来源存在性、标签、大小、本地链接／标题检查；全库导航为 125 篇知识页、12 篇导航、22 条 Top 30，登记／父入口无问题。三页均采用五个主节，补充内容归三级说明，由人工审阅职责与层级。

三页 109 处源码行引用、79 个唯一目标、16 份来源文件 strict 检查为 0 error／0 warning；truncated=false、omitted=0、by_page_omitted=0。源码位置检查不证明引用能支撑技术结论，语义仍以上述实际审阅为依据。

三个完整 modmain 示例与临时 Lua 逐字一致，通过已有 Full Moon Lua 5.1 syntax-only；byte／position 检查按该模式跳过。没有执行示例、安装环境、构建解析器或重建查询缓存，临时文件已清理。

受控需求分别为“记录猪人首次延迟初始化位置并保留读档值”“普通猪人房未来补员等待减半”“格罗姆后续间隔减半并保留当前计时”。按已知页面直读“典型接入”，均返回元数据、快速判断、完整代码／条件和验证记录；JSON stdout 分别为 5,058／5,530／5,760 个实际 UTF-8 字节（换行按 LF），complete_selected_ranges=true、truncated=false、omitted=0、source_version=match。补充 spawner 接口／保存边界后，所选案例范围未变化。

三份源码 SHA-256 与覆盖 CSV 一致；798 个候选现为 117 已建／681 未建，另列 19 replica／3 helper／1 empty。主题分别为交互技能 15、人物 15、战斗 13、物品 28、世界 46，组件页与主题唯一登记、覆盖清单一致。世界主题 123 行／9,574 字节，组件入口 25 行／1,558 字节，名称直查及正文地址保持。

以上是作者受控读取、源码／结构／位置／语法检查，不是独立模型自主任务或准确 token 收益测量，没有游戏验证。

## 收尾与后续

同步覆盖 CSV、PLAN、README、design 和 docs 入口，按规则追加日志；五份工程 Markdown 的 117 处本地链接／锚点及 git diff --check 通过。日志为 174 行／25,804 字节，未超归档阈值。三份 batch26 临时 Lua 已清理，历史报告与旧日志保留原文。

下一批拟核验 herd／herdmember 的真实群体宿主、成员登记／退出、中心消费者与保存／清理；仅按名称排队，不称已核验职责。heatrock Prefab、配方注册 API 与 cookbook 队列继续保留，不等待全部组件完成。

未修改 Skill、工具箱、游戏源码或输入模组，未提交／推送。页面保持 source-reviewed；新建／读档、补员／出门、禁用／排队／销毁、密度／离屏、海上／平台、产物 nil／动画与多模组叠加待游戏实测。
