# 世界土壤水分与养分组件检视

2026-10-18；Steam buildid 25643519。按 dst-wiki Skill 完成第十八批 farming_manager 局部接口页；复用同会话已核对的快照／安装结果（Skip），未再次解压。文本与临时 Lua 均用 apply_patch 写入／清理。

## 交付与范围

[farming_manager](../../../wiki/reference/components/farming-manager.md) 为 94 行／11,507 个实际 UTF-8 字节，按文件命名规则使用 farming-manager.md，标识符 farming_manager 保留在标题、别名与叶级登记中。覆盖世界宿主／初始化、三养分与水分接口、坐标／资格／返回、阶段消费及必要地形／存读；不宣称管理器全量审校或完整农作系统覆盖。

- Tile 三养分接口使用地块二维坐标，AtPoint 使用世界三维坐标；原部署 helper 先查农田资格再转换坐标。AddTileNutrients 限 0～100 并返回 true，但不验证地皮；SetTileNutrients 直接按位编码，不自动按同样范围限幅。
- 水分增量只处理已有农田覆盖层，不生成农田、没有成功返回值；下限受世界 wetness 约束，上限本快照为 100。IsSoilMoistAtPoint 只回布尔且包含世界 wetness，不是水量 getter。周期、饮水率及压力统计继续复用已建页面。
- CycleNutrientsAtPoint 返回 depleted，而非成功；缺覆盖层直接 true。预览不扣养分，正式调用不足时仍消费实际可得部分，并按实际消耗总数向 restore 类别返还；空 restore 存在除数／随机数边界。作物查看与阶段成长分别传入对应模式，补肥不回减旧压力点。
- 完整 modmain 示例包装普通 farm_hoe 的 Till，原成功后为所在整格补各 5 养分和 10 水分，保留原返回／土堆与动作消费；可重复合法锄地叠加，不包含每格一次或额外扣费。停用不回滚世界网格，金锄／热泉锄不在目标范围。
- 原版参照核验肥料部署和洒水壶配置／wateryprotection 土壤调用；实体效果半径不表示多格浇水。世界存读恢复网格并重建覆盖层，饮水者重新登记；杂草、果蝇、旧版本迁移和完整同步不在本批范围。

## 校验与读取

页面与世界主题两个 Wiki 文件的元数据、来源、标签、大小、本地链接／标题检查通过；全库导航 105 篇知识页、12 篇导航、22 条 Top 30，无遗漏、重复登记或父入口问题。章节职责人工审阅，沿用五个主节，复杂接口用三级小节，不增加模板或目录层级。

42 处源码行引用、38 个唯一目标、12 份来源文件 strict 检查为 0 error／0 warning；truncated=false、omitted=0、by_page_omitted=0。初查一处原数据构造起点落在空行，已改到实际 for 循环并复查。组件 SHA-256 与覆盖清单匹配；798 候选现为 97 已建／701 未建，另 19 replica／3 helper／1 empty 单列。

临时 Lua 与页面完整示例逐字一致，通过已有 Full Moon Lua 5.1 syntax-only；byte／position 检查按该模式跳过。只核验语法，不证明 API 绑定、世界初始化时机或实际补给效果；临时文件已删除，无安装、编译或缓存重建。

两项作者受控查询分别选择“锄地补水施肥”与“阶段养分预览是否消费”；均返回快速判断、目标完整章节与验证记录，complete_selected_ranges=true、truncated=false、omitted=0：

| 章节 | JSON stdout 实际 UTF-8 字节（换行按 LF） |
|---|---:|
| 典型接入 | 6,211 |
| 阶段养分消费与压力 | 5,151 |

世界主题新增一条，现为 83 行／6,111 字节；组件分类仍为 25 行／1,378 字节，未知需求只进入相关主题，已知名称可通过标识符定位新页。此为作者选定路径／章节的受控读取，不是独立模型自主发现或 token 收益评测。

## 收尾与后续

同步覆盖 CSV、PLAN、README、design、docs 入口及追加日志；五份本批工程 Markdown 的 99 处本地引用／锚点和 git diff --check 通过。既有历史报告和日志保持原文，当前日志 126 行／18,201 字节，未超归档阈值；本批临时 Lua 已清理。

下一批拟补 wateryprotection／fillable／watersource，核验洒水、补水来源及物品消耗的协作；这些候选仍待执行前核验。heatrock Prefab、配方注册 API 与后续 cookbook 队列保留，不等待全组件完成。

未改 Skill、工具箱、游戏源码或输入模组，未提交／推送。游戏未运行，页面保持 source-reviewed；重复锄地、上下限、同格多作物、天气、存读／地形变换、分片和其他模组包装顺序待实测。
