# 群体发情组件检视

2026-10-19；Steam buildid 25643519。按 dst-wiki Skill 完成第二十八批一页，复用同会话已确认快照，未再次解压。文本及临时 Lua 用 apply_patch 写入／清理。

## 交付与边界

[mood](../../../wiki/reference/components/mood.md) 为 80 行／12,304 个实际 UTF-8 字节，覆盖天数循环、季节校准、状态与开关、宿主回调、存档归一化及牛群／企鹅消费者；附普通牛群后续 wait 加倍的完整 modmain 示例。[herd](../../../wiki/reference/components/herd.md) 只追加新页关联指针，现为 79 行／10,867 字节，未升级原页验证范围。

- 牛的 mood 位于群体节点；成员从 herdmember 指向的群体读取，进入／退出事件另处理 scarytoprey 和外观。群体回调启停 periodicspawner 并广播状态，新增成员也同步当前状态；不把单独给 beefalo 挂组件当完整接入。
- cycles 每次回调仅减 1，检查的是 worldsettingsenabled 和有效计数，不检查 enabled。Enable 会请求退出状态，Enable(true) 也不是无副作用的恢复；禁用不等于暂停剩余天数，forcemood 不绕过两个进入门禁。
- SetMoodTimeInDays 会直接重设 wait／计数和 isinmood=false，不走退出回调。SetMoodSeason 追加列表／首次监听，不立即应用季节；ValidateMood 可重复整季进入并覆盖计数，ResetMood 只处理 seasonmood。整季进入取当前季节总长度，而非剩余天数。
- CheckForMoodChange 仅在计数到期尝试切换；牛群退出回调还调用它，wait=0 时可在同一回调链重新进入。重复设置强制状态或每帧校准会影响回调／计数，不作为安全轮询方式推荐。
- 保存用当前状态的 in/out multiplier 归一化 daysleft，v2 加载再乘当前倍率并向上取整，旧无版本分支另用 math.min。这些倍率不决定每日扣天速度；开关、length／wait 等仍由工厂设置。加载尝试切换会触发回调，牛群零延迟 ValidateMood 还可再次校准，未承诺读档季节计数原样保持。
- 参数例只将 moodtimeindays.wait 乘 2，不再次调用重配方法、不直接改当前计数／状态或保存倍率；默认 wait=20 时后续等待使用 40，当前阶段仍走原恢复／季节逻辑。保留 length、forcemood、世界禁用、繁殖资格与所有原回调，不称整体繁殖周期或产仔间隔都翻倍。
- 企鹅群配置 wait=0，没有登记季节，成员事件设置 nesting；工厂未实际挂载 periodicspawner。回调有该组件的判空分支不代表存在定时产蛋调度，本批未审校企鹅产蛋系统。

只核验本主题必要实现及消费者，未全面审校世界 cycles 跳变、全部旧版／损坏存档、卸组件后的清理、战斗／骑乘／驯化、所有外观或世界生成。未修改 Skill、工具箱、游戏源码或输入模组。

## 校验与读取

四个 Wiki 文件（新页、herd 关联、世界主题和组件入口）通过格式、元数据、来源存在性、标签、大小、本地链接及标题检查；全库导航 128 篇知识页、12 篇导航、22 条 Top 30，无遗漏／重复登记或父入口问题。新页五个主节及按需三级说明由人工审阅。

mood 与受影响 herd 两页共 90 处源码行引用、63 个唯一目标、13 份来源文件 strict 检查为 0 error／0 warning；truncated=false、omitted=0、by_page_omitted=0。引用位置检查不替代上述语义核验，也不证明时序／产量效果。

完整 modmain 示例与临时 Lua 逐字一致，通过已有 Full Moon Lua 5.1 syntax-only，byte／position 检查按该模式跳过。未执行示例、安装依赖、构建解析器或重建缓存；临时文件已删除。

需求“延长牛群后续非发情等待，不直接重置当前计数”直读 mood 的“典型接入”，返回元数据、快速判断、完整代码／条件与验证记录；JSON stdout 为 6,284 个实际 UTF-8 字节（换行按 LF），complete_selected_ranges=true、truncated=false、omitted=0、source_version=match。属于作者受控读取，不是独立模型自主检索或准确 token 收益测量。

组件原字节 SHA-256 与覆盖 CSV 一致；798 候选现为 120 已建／678 未建，19 replica／3 helper／1 empty 另列。主题分别为交互技能 15、人物 15、战斗 13、物品 28、世界 49；实际组件页、唯一叶级登记与覆盖清单一致。世界主题 126 行／10,110 字节，组件入口 25 行／1,576 字节，保持正文地址和名称直查。

## 收尾与后续

同步覆盖 CSV、PLAN、README、design、docs 入口并追加日志；五份工程 Markdown 的 120 处本地链接／锚点及 git diff --check 通过。日志为 186 行／27,511 字节，未超归档阈值。batch28 临时 Lua 已清理，历史报告／日志保持原文。

下一批拟独立核验 domesticatable 的牛宿主、驯化／服从消费者、回调及保存／清理；骑乘按 rider／rideable／saddler 等实际依赖继续拆批，名称排队不表示职责已审校。heatrock Prefab、配方注册 API 与 cookbook 既有队列保留。

没有运行游戏、独立模型评测、缓存重建、提交或推送。页面保持 source-reviewed；春季开局／换季、普通循环／强制发情、禁用／重启、零等待回调、存档倍率变化／零延迟校准、成员表现与多模组叠加待实测。
