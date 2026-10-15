# 补水与浇水工具组件检视

2026-10-18；Steam buildid 25643519。按 dst-wiki Skill 完成第十九批三页；复用同会话已核对的快照／安装结果，未再次解压。文本与临时 Lua 均通过 apply_patch 写入／清理。

## 交付与边界

| 页面 | 核验与完整 modmain 示例 | 物理行／实际 UTF-8 字节 |
|---|---|---:|
| [wateryprotection](../../../wiki/reference/components/wateryprotection.md) | 实体效果、单格土壤、范围回调及原壶消费；普通壶 addwetness 翻倍 | 70／7,731 |
| [fillable](../../../wiki/reference/components/fillable.md) | 原地补次数／换物、先消费、海水门禁与返回；允许普通壶装海水 | 68／7,219 |
| [watersource](../../../wiki/reference/components/watersource.md) | available 标签、使用委托、份量与容量；一块冰给原版壶补 2 次水 | 66／7,260 |

- wateryprotection 对实体范围处理后只更新中心单格土壤，再调用收尾回调；扩大半径不会自动浇多格。原壶收尾回调负责 Use，直接 ApplyProtectionToEntity 不执行该回调或浇土壤。改 addwetness 也影响实体湿度及烧毁释水消费者，不能只称右键浇水量变化。
- fillable 先调用来源 Use，再委托 overrideonfillfn 或生成 filledprefab；失败无自动退款。showoceanaction 只是候选显示，acceptsoceanwater 是动作门禁，Fill 自身不检查它。海水动作资格通过后忽略 Fill 返回而直接 true，本例保留原壶成功回调。
- watersource 的 available 维护标签，不是 Use 的内部拒绝开关；Use 只调用 onusefn，不传播回调结果或自动扣物品。原冰／雪团回调消费一份，原壶解释 override_fill_uses 并限到容量；血囊默认替换不读取该份量，池塘另由宿主冻结切换资格。
- 三例分别保留原壶消费、原 OnFill、原冰一份消费；无新资源，不声称支持任意嵌套容器、按容量退款、净水或完整多格浇水。停用只恢复后续初始化的配置，既有水分／次数不回滚。

## 校验与读取

六个 Wiki 文件（三页、物品主题、世界主题和组件分类）通过格式、来源、标签、大小、本地链接及标题检查；全库导航 108 篇知识页、12 篇导航、22 条 Top 30，无遗漏／重复登记或父入口问题。章节职责人工审阅，沿用五个主节，无模板或类型变更。

三页 87 处源码行引用、57 个唯一目标、13 份来源文件 strict 检查为 0 error／0 warning；truncated=false、omitted=0、by_page_omitted=0。三条组件 SHA-256 与覆盖 CSV 匹配；798 候选现为 100 已建／698 未建，另 19 replica／3 helper／1 empty 单列。

三段临时 Lua 与页面示例逐字一致，通过已有 Full Moon Lua 5.1 syntax-only，byte／position 检查按该模式跳过；不证明 API、资源、路径或游戏运行。临时文件已删除，未安装依赖、编译或重建缓存。

作者受控读取三页“典型接入”，均包含快速判断、完整代码／条件和验证记录，complete_selected_ranges=true、truncated=false、omitted=0：

| 页面 | JSON stdout 实际 UTF-8 字节（换行按 LF） |
|---|---:|
| wateryprotection | 4,572 |
| fillable | 4,669 |
| watersource | 5,024 |

物品主题新增“补水与浇水工具”三条唯一登记，为 76 行／5,259 字节；世界主题只补跨主题导航，为 85 行／6,212 字节；组件分类 25 行／1,396 字节。未知名称先筛相关主题，已知名称直达，不增导航层级。此为作者选定路径／章节的读取，不是独立模型发现或 token 收益评测。

## 收尾与后续

同步覆盖 CSV、PLAN、README、design、docs 入口和追加日志；五份本批工程 Markdown 的 103 处本地引用／锚点及 git diff --check 通过。历史报告和既有日志保留原文，当前日志 132 行／19,167 字节，未达归档阈值；本批三份临时 Lua 已清理。

下一批拟核验 witherable／pollinator／treegrowthsolution，按枯萎保护、授粉、树木促进的实际宿主／消费者分别生成，不预设统一植物效果；heatrock Prefab、配方注册 API 与后续 cookbook 队列保留。

本批未改 Skill、工具箱、游戏源码或输入模组，未提交／推送。游戏未运行，页面保持 source-reviewed；空壶／满壶、岸边／手柄、范围与干湿、最后一份材料、冻结、存读及其他模组回调顺序待实测。
