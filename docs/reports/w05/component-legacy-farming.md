# 旧农场种植路径组件检视

2026-10-19；Steam buildid 25643519。按 dst-wiki Skill 完成第二十二批三页；本次核对本机安装，源码准备返回 Skip，未解压。文本与临时 Lua 均用 apply_patch 写入／清理。

## 交付与边界

| 页面 | 核验与完整 modmain 示例 | 物理行／实际 UTF-8 字节 |
|---|---|---:|
| [crop](../../../wiki/reference/components/crop.md) | 连续进度、自然门禁、成熟／枯萎、收获与存读；plant_normal 新启动时长减半 | 72／8,658 |
| [grower](../../../wiki/reference/components/grower.md) | 设施种植点／子作物／肥力轮次、时长乘数与 Reset；已有旧高级农场后续种植时长减半 | 70／8,417 |
| [plantable](../../../wiki/reference/components/plantable.md) | 旧种子两字段及消费者解释；胡萝卜种子旧路径产南瓜，现代播种仍保留胡萝卜 | 64／6,469 |

- 当前 prefablist 仍列 farmplot，工厂返回 slow_farmplot／fast_farmplot；recipes 中其名称命中的是旧结构的拆解记录，不据此认定普通制作菜单仍能建。示例适用于已有或明确恢复旧农场的项目，不提供旧设施／配方恢复工程。
- plantable 只有 product／growtime，grower 固定生成 plant_normal，product 是最终产物；当前种子仍保留这组数据。veggies 现行 OnDeploy 已使用 farmplantable，plant_normal_ground／crop 部署只在注释块内；不能以文本命中或资源依赖当实际调用。
- grower.growrate 乘在秒数上，调小才快；PlantItem 不查空位／肥力而先 Reset，直接传整堆会整实体删除。默认空种植点仍可能消费并进入非空状态，不能只挂组件就称完整农场。肥力扣减、收获与存读由设施和子作物协作。
- crop 的 DoGrow 受温度／夜间／光照等条件影响；Mature 并非完整进度 setter，MakeWithered 也不自加 withered 标签。收获有发放／移除副作用，回调在清理前。存档保存 rate，不自动随新配置重算；三种时长修改入口可叠乘，页面提示按需求选一个。
- 本批仅核验必要实现／宿主／消费者，不把旧机制推荐为现代农田方案，也不凭 deprecated 注释判整个链路无实现。未全面审校旧恢复模组、制作入口、所有产物／资源、火烧重置或存档迁移。

## 校验与读取

五个 Wiki 文件（三页、世界主题与组件分类）通过元数据、来源、标签、大小、本地链接与标题检查；全库导航 117 篇知识页、12 篇导航、22 条 Top 30，无遗漏／重复登记或父入口问题。章节职责人工审阅，沿用五个主节，无模板或类型变更。

三页 87 处源码行引用、62 个唯一目标、12 份来源文件 strict 检查为 0 error／0 warning；truncated=false、omitted=0、by_page_omitted=0。三条组件 SHA-256 与覆盖 CSV 匹配；798 候选现为 109 已建／689 未建，另 19 replica／3 helper／1 empty 单列。

三份临时 Lua 与页面示例逐字一致，通过已有 Full Moon Lua 5.1 syntax-only，byte／position 检查按该模式跳过。不执行代码、不证明旧对象取得、资源或游戏运行；临时文件已删除，无安装依赖、编译或缓存重建。

作者受控读取各页“典型接入”，均返回快速判断、完整代码／条件与验证记录，complete_selected_ranges=true、truncated=false、omitted=0：

| 页面 | JSON stdout 实际 UTF-8 字节（换行按 LF） |
|---|---:|
| crop | 4,666 |
| grower | 4,684 |
| plantable | 4,824 |

世界主题新增“旧农场种植路径”三条唯一登记，为 110 行／7,964 字节；组件分类 25 行／1,456 字节。未知名称按相关分组筛选，已知名称直达；五主题仍不增层级。此为作者选定路径／章节的受控读取，不是独立模型自主发现或 token 收益评测。

## 收尾与后续

同步覆盖 CSV、PLAN、README、design、docs 入口与追加日志；五份本批工程 Markdown 的 109 处本地引用／锚点及 git diff --check 通过。五主题 109 篇组件逐项核对无遗漏或重复；当前日志 150 行／22,159 字节，未超归档阈值。历史报告和既有日志不改写，三份 batch22 临时 Lua 已清理。

下一批拟核验 harvestable／plantregrowth，按真实宿主限定产物累积与再生协作，复杂世界机制不强行写成普通组件调用例；nutrients_visual_manager 后续独立。heatrock Prefab、配方注册 API 与 cookbook 队列保留。

未改 Skill、工具箱、游戏源码或输入模组，未提交／推送。页面保持 source-reviewed，游戏未运行；旧设施取得、黑暗／低温／枯萎／燃烧收获、肥力上下限、多点、存读及其他模组倍率／产物覆盖待实测。
