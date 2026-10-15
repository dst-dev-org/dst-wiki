# 播种、土壤饮水与植物压力组件检视

2026-10-18；Steam buildid 25643519。按 dst-wiki Skill 完成第十七批三页；源码准备返回 Skip，未解压。文本与临时示例使用 apply_patch 写入／清理。

## 交付与边界

| 页面 | 核验与完整 modmain 示例 | 物理行／实际 UTF-8 字节 |
|---|---|---:|
| [farmplantable](../../../wiki/reference/components/farmplantable.md) | 土堆／种子消费、事件、动作单颗拆分与地面路径；普通种子土堆播种固定胡萝卜 | 66／6,738 |
| [farmsoildrinker](../../../wiki/reference/components/farmsoildrinker.md) | 负率耗水、干湿统计、世界登记与阶段存读；种植胡萝卜原耗水率减半 | 70／8,150 |
| [farmplantstress](../../../wiki/reference/components/farmplantstress.md) | 注册／状态、阶段累计、查看与成长检测及巨型消费者；胡萝卜结算豁免季节压力 | 71／8,567 |

- farmplantable 直接删除传入种子实体；人物动作先 RemoveItem 拆出一颗，失败尝试还回。本例只改土堆播种产物，plantkin 直接地面部署仍由原 OnDeploy 生成随机种子植物，不把三条种植路径混为一谈。无效产物可能已删除土堆，不宣称事务回滚。
- farmsoildrinker 的速率由世界乘 dt 相加，原植物为负值；包装回调保留生长暂停时的 0，不修改共享 plant_def。CalcPercentTimeHydrated 会补计时，Reset 后尚无湿润累计时返回 0；耗水降低不保证免水分压力。原魔法催生同样读取该速率，不能称只影响定时刷新。
- farmplantstress 的注册 testfn 被描述／调试消费；原植物成长结算直接调用本地检测函数，不能只替换 testfn 就声称改了结算。示例在 MakeCheckpoint 前清 season，只免后续点数，不改变季节生长速度或查看描述，也不保证巨型。最终等级阈值是固定累计值，不按类别数缩放。

## 检查与读取

五个 Wiki 文件（三页、世界主题与组件分类）通过格式、来源、标签、大小、本地链接与标题检查；全库导航 104 篇知识页、12 篇导航、22 条 Top 30，无遗漏／重复登记或父入口问题。章节职责人工审阅，五个标准二级章节保持轻量，不新增模板／类型／目录。

三页 96 处源码行引用、75 个唯一目标、13 份来源文件 strict 检查为 0 error／0 warning，truncated=false、omitted=0、by_page_omitted=0。首次发现成熟阶段表三处起点落在 `{`，已改到实际 name 字段并复查；未机械移动必要区间终点。三条源码 SHA-256 与覆盖 CSV 匹配；798 候选现为 96 已建／702 未建，另 19 replica／3 helper／1 empty 单列。

三段临时 Lua 与页面代码逐项一致，通过已有 Full Moon Lua 5.1 syntax-only，byte／position 检查按该模式跳过；不声称类型、绑定或运行通过。完成后删除本批临时文件，不安装依赖、不编译解析器、不重建查询缓存。

受控读取三页“典型接入”，均返回快速判断、完整代码／条件及验证记录，complete_selected_ranges=true、truncated=false、omitted=0：

| 页面 | JSON stdout 实际 UTF-8 字节（换行按 LF） |
|---|---:|
| farmplantable | 4,875 |
| farmsoildrinker | 4,898 |
| farmplantstress | 5,426 |

未知名称的播种／饮水／压力需求由组件分类进入世界主题的本批三条登记，已知名称仍直达；世界主题 82 行／5,913 字节、组件分类 25 行／1,378 字节，未新增导航层级。此为作者受控读取，不是独立模型发现或 token 收益评测。

## 收尾与后续

同步覆盖 CSV、PLAN、README、design、docs 入口和追加日志；五份本批工程 Markdown 的 98 处本地引用／锚点与 git diff --check 通过。既有历史报告／日志不改写，累计日志 120 行／17,315 字节，未达归档阈值；三份 batch17 临时 Lua 已清理。

下一批拟独立核验 farming_manager，限定土壤水分／养分接口、坐标单位、世界宿主与必要消费者；不将本批或下一批当完整农作骨架／杂草／果蝇系统教程。heatrock Prefab、配方注册 API 与后续 cookbook 队列保留。

本批未改 Skill、工具箱、游戏源码或输入模组，未提交／推送。游戏未运行，保持 source-reviewed；多人播种、主栏／背包拆分、干湿切换、魔法成长、存读与其他模组包装顺序仍需实测。
