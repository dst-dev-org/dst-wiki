# 产物累积与周边再生组件检视

2026-10-19；Steam buildid 25643519。按 dst-wiki Skill 完成第二十三批两页；复用本会话已检查的快照，未再次解压。文本与临时 Lua 均用 apply_patch 写入／清理。

## 交付与边界

| 页面 | 核验与完整 modmain 示例 | 物理行／实际 UTF-8 字节 |
|---|---|---:|
| [harvestable](../../../wiki/reference/components/harvestable.md) | 逐次累积、整批收获、额外门禁／回调与存读；蘑菇农场后续种植的生长间隔减半 | 81／9,962 |
| [plantregrowth](../../../wiki/reference/components/plantregrowth.md) | 种类时钟／桶调度、原始区域密度、落点／计数与注销；常青树周边再生基础等待减半 | 75／9,913 |

- harvestable 的 SetProduct 会清库存，SetGrowTime 不重排当前任务；Disable 不能阻止宿主直接 Grow。Harvest 先清计数、执行宿主回调，再按原数量生成／发放并尝试重启；基本可收获不等于额外检查已通过，生成失败不回滚。
- 蜂箱既有授粉归巢直接 Grow，也有睡眠期间按环境条件启动的计时；蘑菇农场投料配置产物和分次时间，并先 Grow 一次。示例仅包蘑菇农场的 SetGrowTime，保留 nil、回调及技能倍率，不将所有 harvestable 宿主改成统一定时产物。
- 蘑菇农场会直接存读 product／maxproduce／growtime，组件另存库存／剩余时间；示例不追溯重排旧批次，也不称已有存档加载即加速。组件卸载钩子只清标签，动态移除前停任务，与原烧毁处理一致。
- plantregrowth 的 SetRegrowthRate 会登记共享桶且不去重，重复调用可能遗留重复项；示例在已有工厂登记后改字段并 ResetGrowthTime。种类时钟有季节／月相和世界倍率，任务每 31 秒只轮一个桶，不承诺所有母株逐 31 秒检查或现实等待固定减半。
- GetFiveRadius 依赖母株所在原始 topology／generated.densities；缺密度会注销，落点／计数失败也已重置下一轮。SetSkipCanPlantCheck 仅跳过一项土壤检查，换 product 不改变半径所用母株类型；只挂组件不构成新植物繁殖工程。
- 常青树变体先规范为 evergreen，SpawnPrefabFromSim 在工厂后按规范名执行 Hook；示例覆盖该规范名下实际带组件的对象。存档恢复晚于 Hook，会覆盖本轮到期值，后续重置才用新 rate。未把相同工厂内的 growable 阶段成长与周边繁殖混用。

仅审阅必要实现／宿主／消费者；未全面审校全部树种、harvestable 宿主、自定义母株／世界密度注册、客户端外观或存档迁移。页面按五个主节组织，协作说明放相关主节内，不修改 Skill 模板或工具。

## 校验与读取

四个 Wiki 文件（两页、世界主题与组件分类）通过元数据、来源、标签、大小、本地链接及标题检查；全库导航 119 篇知识页、12 篇导航、22 条 Top 30，无遗漏／重复登记或父入口问题。章节职责人工审阅。

两页 75 处源码行引用、64 个唯一目标、12 份来源文件 strict 检查为 0 error／0 warning；truncated=false、omitted=0、by_page_omitted=0。两条组件 SHA-256 与覆盖 CSV 匹配；798 候选现为 111 已建／687 未建，另 19 replica／3 helper／1 empty 单列。

两份临时 Lua 与页面完整示例逐字一致，通过已有 Full Moon Lua 5.1 syntax-only；byte／position 检查按该模式跳过。未执行示例、安装依赖、编译解析器或重建缓存，临时文件已清理。

作者按明确需求受控读取各页“典型接入”，返回元数据、快速判断、完整示例／条件与验证记录，complete_selected_ranges=true、truncated=false、omitted=0：

| 需求 → 页面 | JSON stdout 实际 UTF-8 字节（换行按 LF） |
|---|---:|
| 蘑菇农场后续种植间隔减半 → harvestable | 5,893 |
| 常青树周边再生加快 → plantregrowth | 5,660 |

世界主题为 112 行／8,336 字节，组件分类为 25 行／1,486 字节；五主题 111 篇组件逐项唯一登记。主题索引是分类维护来源，覆盖 CSV 只记文件建设状态／核验范围及指纹，不新增第二份分类清单。已知名称仍直达页面，未知名称按“采收与成长”分组筛选，不增加索引层级。

这只是作者选择路径／章节的完整读取检查，不是独立模型自主发现或 token 收益测量；真实游戏未运行，不把结构、源码行位置和 Lua 解析通过当作产物／繁殖效果已测。

## 收尾与后续

同步覆盖 CSV、PLAN、README、design、docs 入口并追加日志；本批工程 Markdown 本地链接／锚点与 git diff --check 检查通过，历史报告及既有日志未改写。日志未超归档阈值，未留 batch23 临时 Lua。

下一批独立核验 nutrients_visual_manager 的实际宿主、双端边界、显示启停／清理及与 farming_manager 分工，按证据限定交付。heatrock Prefab、配方注册 API 与 cookbook 队列继续保留。

未改 Skill、工具箱、游戏源码或输入模组，未提交／推送。页面保持 source-reviewed；积雪／催长／蜂箱睡醒、季节／世界再生关闭／密度与落点、已有存档及其他模组叠加待游戏实测。
