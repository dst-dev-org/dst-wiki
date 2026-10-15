# 养分显示组件检视

2026-10-19；Steam buildid 25643519。按 dst-wiki Skill 完成第二十四批一页；复用同会话已核对快照，未再次解压。文本及临时 Lua 均用 apply_patch 写入／清理。

## 交付与边界

[nutrients_visual_manager](../../../wiki/reference/components/nutrients-visual-manager.md) 为 72 行／9,782 个实际 UTF-8 字节；包含非专服世界宿主、样式／登记接口、显示协作、完整 modmain 排序包装、陷阱与验证范围。

- 原世界工厂在 ismastersim 返回分支之前、非专服条件内安装该组件；客户端和有画面主机都可有，专服构造有断言。示例不加服务端限定，也不手工挂载组件。
- farming_manager 管真实土壤数据及父覆盖层创建／移除；nutrients_overlay 将三养分按阈值压成网络等级，非专服再建立本地 nutrients_overlay_visual。本管理器只处理本地标记样式／登记，不负责精确数值或水分动画。
- nutrientsvision 事件在世界对象上监听。装备标签经 playervision 的本地 ThePlayer 检查发送；ForceNutrientVision 为单个强制布尔值，不能误写成复数方法或多来源堆栈。HUD 遮罩另监听同一事件，不由土壤标记组件统一控制。
- 登记不初始化样式、注销不删除对象。原视觉子对象先应用样式再登记，在 onremove 注销；有画面主模拟的父覆盖层睡眠时删除子对象、唤醒重建。关闭视野改变乘色／加色和图层，不是 Hide 或销毁。
- 完整示例包装 UpdateVisualAnimState，先走原逻辑再将排序号 1 换成 2，保留颜色／图层和模式变化。该例只说明样式入口，不承诺跨图层遮挡优先级、远程分发、自定义美术或强制视野效果；停用重新加载恢复，不改土壤存档。

仅核验养分相关消费者和必要生命周期；未审校完整 playervision、全部眼镜行为、网络引擎、自定义资源或视野能力迁移。未生成空的 playervision／Prefab 页面，也不把本页扩成完整农作显示教程。

## 校验与读取

三个 Wiki 文件（新页、世界主题、组件分类）通过元数据、来源、标签、大小、本地链接及标题检查；全库导航 120 篇知识页、12 篇导航、22 条 Top 30，无遗漏／重复登记或父入口问题。五个主节及按需三级分工说明由人工审阅，未改模板或类型。

新页 32 处源码行引用、30 个唯一目标、9 份来源文件 strict 检查为 0 error／0 warning；truncated=false、omitted=0、by_page_omitted=0。组件原字节 SHA-256 与覆盖 CSV 一致；798 候选现为 112 已建／686 未建，19 replica／3 helper／1 empty 继续单列。

完整 Lua 临时文件与页面代码逐字一致，通过已有 Full Moon Lua 5.1 syntax-only；byte／position 检查按该模式跳过。没有执行示例、安装依赖、构建解析器或重建缓存；临时文件已删除。

需求“调整养分标记绘制排序，保留眼镜切换”按已知页面读取“典型接入”，返回元数据、快速判断、完整例与条件及验证记录：JSON stdout 为 5,203 个实际 UTF-8 字节（换行按 LF），complete_selected_ranges=true、truncated=false、omitted=0、source_version=match。

世界主题新增一条唯一登记，现为 113 行／8,579 字节；组件分类 25 行／1,501 字节，主题范围增“养分显示”。五主题分别为交互技能 15、人物 15、战斗 13、物品 28、世界 41，共 112 篇组件；登记与覆盖 CSV 逐项匹配，无重复／遗漏。分类以主题索引为维护来源，不新增第二份分类清单。

以上为作者受控读取与格式／引用位置／语法检查，不是独立模型自主查询或 token 收益测量；没有运行游戏，不证明显示与联网效果已验证。

## 收尾与后续

同步覆盖 CSV、PLAN、README、design、docs 入口与追加日志；本批五份工程 Markdown 本地引用／锚点及 git diff --check 通过，历史报告及既有日志未改写。日志未超归档阈值，无 batch24 临时 Lua 残留。

下一批拟核验 childspawner／homeseeker 的实际巢穴／子体宿主、生成与归巢关联、数量／时序及保存／清理分工；不把生成器局部接口当完整新生物骨架。heatrock Prefab、配方注册 API 与 cookbook 队列保留。

未改 Skill、工具箱、游戏源码或输入模组，未提交／推送。页面保持 source-reviewed；远端客户端／有画面主机、眼镜穿脱／强制视野、主机睡醒、量化变化、分片及其他模组包装叠加待游戏实测。
