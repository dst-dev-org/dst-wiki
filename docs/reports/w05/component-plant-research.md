# 植物与肥料研究组件检视

2026-10-18；Steam buildid 25643519。按 dst-wiki Skill 完成第二十一批三页；复用同会话已核对的快照／安装结果，未再次解压。文本与临时 Lua 均用 apply_patch 写入／清理。

## 交付与边界

| 页面 | 核验与完整 modmain 示例 | 物理行／实际 UTF-8 字节 |
|---|---|---:|
| [plantresearchable](../../../wiki/reference/components/plantresearchable.md) | 查询回调／随机判定、研究阶段／键、人物事件与客户端候选；研究胡萝卜附带种子阶段 | 73／7,549 |
| [fertilizerresearchable](../../../wiki/reference/components/fertilizerresearchable.md) | 肥料键与研究事件、双端 Getter、施肥与研究分工；成功施用便便后自动研究 | 74／6,960 |
| [plantregistryupdater](../../../wiki/reference/components/plantregistryupdater.md) | 人物事件、双端登记／RPC、个人本地档案与有限去重；当前登记中新阶段奖励 1 精神 | 78／8,976 |

- 两种 researchable 的回调字段实际拼为 reasearchinfofn，使用 setter；回调只查询。植物随机判定只看第一个返回是否 nil，研究阶段不等同 growable.stage；Learn 方法发事件、无新增布尔返回，动作 true 也不证明新增。
- 原图鉴候选依赖客户端可读 Getter、相关标签、plantinspector 装备与本地图鉴；仅加服务端组件／返回新键不等于完成 UI、数据和网络注册。胡萝卜例保留原当前阶段，再提交已核验的原种子阶段 1；便便例主动赋予使用即学习，不宣称这是原版默认行为。
- updater 在玩家公共工厂的客户端返回之前安装；原学习事件监听位于服务端，客户端 RPC 接收则直接调用本人 updater。playeractivated 时本地玩家切换到已加载 ThePlantRegistry 并启用保存；专服／远端人物的独立记录不知道完整个人历史，不作为永久成就或重登防刷依据。
- 奖励例只在服务端包装 LearnPlantStage，通过当前登记前后查询判断新增，保留原通知；重登／实体重建可能再次奖励，肥料／照片不在目标内。照片数据层还排除模组角色，远端路径不能按本地最高纪录去重，不套用阶段研究语义。
- 持久化由数据层在 save_enabled／更新条件下写本地 plantregistry，非 modded 定义还有平台交互；本组件无角色存档钩子。未全面审校本地加载／平台／跨平台、照片皮肤或新条目 UI／注册全链。

## 校验与读取

六个 Wiki 文件（三页、世界主题、人物主题和组件分类）通过元数据、来源、标签、大小、本地链接及标题检查；全库导航 114 篇知识页、12 篇导航、22 条 Top 30，无遗漏、重复登记或父入口问题。章节职责人工审阅，维持五个主节，无模板或类型变更。

三页 77 处源码行引用、60 个唯一目标、17 份来源文件 strict 检查为 0 error／0 warning；truncated=false、omitted=0、by_page_omitted=0。三条组件 SHA-256 与覆盖 CSV 匹配；798 候选现为 106 已建／692 未建，另 19 replica／3 helper／1 empty 单列。

三份临时 Lua 与各页完整示例逐字一致，通过已有 Full Moon Lua 5.1 syntax-only；byte／position 检查按该模式跳过。不执行示例、不证明 API、网络或游戏效果；临时文件已删除，无依赖安装、编译或查询缓存重建。

作者受控读取三页“典型接入”，均包含快速判断、完整示例／条件及验证记录，complete_selected_ranges=true、truncated=false、omitted=0：

| 页面 | JSON stdout 实际 UTF-8 字节（换行按 LF） |
|---|---:|
| plantresearchable | 4,966 |
| fertilizerresearchable | 5,252 |
| plantregistryupdater | 5,608 |

两个研究源页唯一登记在世界主题，人物更新页唯一登记在人物主题，并由跨主题链接连接。世界主题 102 行／7,359 字节、人物主题 48 行／3,194 字节，组件分类 25 行／1,444 字节；五主题共登记 106 篇，不增层级。此为作者选定路径／章节的受控读取，不是独立模型自主发现或 token 收益评测。

## 收尾与后续

同步覆盖 CSV、PLAN、README、design、docs 入口及追加日志；五份本批工程 Markdown 的 107 处本地引用／锚点及 git diff --check 通过。五主题 106 篇组件逐项核对无遗漏或重复，当前日志 144 行／21,182 字节，未超归档阈值；既有报告和日志保留原文，本批三份临时 Lua 已清理。

下一批拟核验 crop／grower／plantable，先确认旧农场实际消费者、适用范围及与现代农作的区别，不凭名称／deprecated 注释推断全部弃用。heatrock Prefab、配方注册 API 与后续 cookbook 队列保留。

未改 Skill、工具箱、游戏源码或输入模组，未提交／推送。页面保持 source-reviewed，游戏未运行；眼镜候选、原阶段／巨型／腐烂、施肥最后一份、主机／专服／远端、重登／角色重建及其他模组包装顺序待实测。
