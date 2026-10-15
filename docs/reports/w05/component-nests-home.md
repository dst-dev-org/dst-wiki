# 巢穴与子体组件检视

2026-10-19；Steam buildid 25643519。按 dst-wiki Skill 完成第二十五批两页；复用同会话已确认快照，未再次解压。文本及临时 Lua 用 apply_patch 写入／清理。

## 交付与边界

- [childspawner](../../../wiki/reference/components/childspawner.md)：86 行／12,343 个实际 UTF-8 字节；覆盖普通／应急计数、容量与补员、生成／接管／接收时序、外部计时、保存重连与必要清理，附普通蜂箱容量加 2 的完整 modmain 示例。
- [homeseeker](../../../wiki/reference/components/homeseeker.md)：88 行／9,662 字节；覆盖家园引用／监听、动作提交、直线时间估计、工蜂 Brain 与接收资格／存档分工，附自定义服务端召回事件接收端。不包含按键、范围选蜂或 RPC 发送端，不冒充完整召回系统。

关键结论以组件实现、蜂箱／野蜂巢、工蜂 Brain／beecommon、GOHOME 动作与 worldsettingsutil 桥接为据：

- SetMaxChildren 会按外出量重算巢内储备；SpawnChild 成功才扣 inside 并接管，onspawned 早于接管；TakeOwnership 本身不扣 inside，也不自动转移旧巢登记，不能重复调用充当归巢。
- 巢内主要保存数量；回家接收先发巢／子体事件，再去外出记录、Remove 实体并恢复 inside，不保证保存生命／任意模组字段。在外子体用 GUID 引用重连；max 只在显式开启保存时入档，布尔 or 恢复不能保证 false 覆盖默认 true。
- 出巢与补员独立；内置 5～10 秒更新、LongUpdate 单次推进、离屏排队与唤醒分别说明。蜂箱／野蜂巢接外部 worldsettingstimer；不把 SetPeriod 写组件字段误称同步重置外部剩余时间，也不审校未读的计时器内部。
- homeseeker 的 SetHome 不登记巢穴、不更新 knownlocations、不自动寻路；家园移除默认卸掉组件。工蜂 Brain 使用 beecommon 直接创建动作，包装 homeseeker:GoHome 不覆盖全部回巢路径。
- 通用 GOHOME 优先 spawner，再 childspawner、hideout；没有接收组件时可能直接删除生物。召回事件动态检查有效家园、燃烧、普通外出登记并排除优先分发的 spawner，仍不承诺战斗／路径竞争下成功。
- 蜂箱加容量示例包装原 setter，保留季节和世界禁用配置；初始化重算储备、存档恢复、季节补满以及停用后超额存量均有边界，不称可逆存档迁移。原蜂蜜生产复用 harvestable 页面。

未全面审校所有巢穴、外来品种概率、spawner／hideout 内部、worldsettingstimer、完整 Brain／SG 或客户端／分片行为；没有新造空的相关机制页。

## 校验与读取

四个 Wiki 文件（两新页、世界主题与组件分类）通过元数据、来源、标签、大小、本地链接／标题检查；全库导航 122 篇知识页、12 篇导航、22 条 Top 30，无重复／遗漏登记或父入口问题。两页五个主节及按需三级说明由人工审阅。

两页 83 处源码行引用、63 个唯一目标、10 份来源文件经 strict 检查为 0 error／0 warning；truncated=false、omitted=0、by_page_omitted=0。首轮检出蜂箱季节引用起点落在上一函数 end，核验后改为 SeasonalSpawnChanges 定义起点并复查通过；不把位置通过当语义证明。

两个完整 modmain 示例与临时 Lua 内容逐字一致，通过已有 Full Moon Lua 5.1 syntax-only；未执行示例，byte／position 检查按该模式跳过。没有安装依赖、构建解析器或重建缓存；临时文件已删除。

受控需求“普通蜂箱多两只蜜蜂”和“为工蜂提供服务端召回事件”均直达相应页“典型接入”，返回元数据、快速判断、完整案例／前提和验证记录。JSON stdout 分别为 5,480／5,503 个实际 UTF-8 字节（换行按 LF），均 complete_selected_ranges=true、truncated=false、omitted=0、source_version=match；不是独立模型自主检索或 token 收益测量。

两个组件源码 SHA-256 与覆盖 CSV 一致；798 候选现为 114 已建／684 未建，19 replica／3 helper／1 empty 另列。五主题分别为交互技能 15、人物 15、战斗 13、物品 28、世界 43，共 114；每页恰在一个主题登记，覆盖 CSV 与实际页面一致。世界主题为 120 行／9,022 字节，组件分类为 25 行／1,528 字节，保持正文路径与名称直查。

## 收尾与后续

同步 PLAN、README、design、docs 入口和覆盖 CSV；追加日志，未重写旧报告或日志。五份工程 Markdown 的 114 处本地链接／锚点及 git diff --check 通过；日志现为 168 行／24,839 字节，未超归档阈值，两个 batch25 临时 Lua 已清理。

下一批拟核验 knownlocations／spawner／periodicspawner 的实际宿主及保存／清理，先区分位置记忆和其他生成模式；只做名称排队，尚未核验其职责，不建占位页。heatrock Prefab、配方注册 API 与 cookbook 原队列保留。

未修改 Skill、工具箱、游戏源码或输入模组，未提交／推送。没有运行游戏，页面保持 source-reviewed；换季／存读／世界禁用、超额数量、归巢与产蜜、烧毁／捕获／死亡、行动竞争和多模组包装待实测。
