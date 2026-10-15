# 烹饪与干燥组件检视

2026-10-17；Steam buildid 25643519。按 dst-wiki Skill 完成第十三批五页；源码准备返回 Skip，未解压。文本与临时 Lua 均使用 apply_patch 写入／清理。

## 交付与关键边界

| 页面 | 核验与完整 modmain 示例 | 物理行／实际 UTF-8 字节 |
|---|---|---:|
| [cookable](../../../wiki/reference/components/cookable.md) | 食材产物／烤前回调／新鲜度；大肉直接烤出肉干 | 60／4,911 |
| [cooker](../../../wiki/reference/components/cooker.md) | 炉具门禁／结果回调／原料删除；石头营火产物恢复全新鲜 | 70／5,618 |
| [dryable](../../../wiki/reference/components/dryable.md) | 原料产物／秒数／显示 build；大肉配置两分钟晾晒 | 61／5,598 |
| [dryingrack](../../../wiki/reference/components/dryingrack.md) | 多格晾晒／天气／保鲜／转移恢复；普通晾晒架自来源免雨 | 70／7,534 |
| [stewer](../../../wiki/reference/components/stewer.md) | 锅中配方／耗料／变质／收获／恢复；普通厨锅新开锅时长减半 | 65／7,385 |

- cookable 不删除原料或发放产物；oncooked 在产物生成前调用，不能作为成功通知。cooker 则在通过门禁后删除整个输入实体，即使未生成结果；正常动作负责取件／拆堆与发放。
- cooker.CanCook 不包办所有权／距离，dangerouscooker 分支还依赖有效 chef。两种完成回调参数不同；产物保鲜示例使用能访问结果的 oncookfn，并保留旧回调。
- dryable 是配置组件；改秒数不自动修改架上任务。rack 完成只生成一个产物并删除输入，原版容器不收堆叠；不套用直接烤制的新鲜度公式。
- 当前普通 meatrack 注册于 meatrack2.lua，使用三格容器；先装 container 再装 dryingrack。Enable／Disable 管理 preserver；实时暂停不保证 LongUpdate 停止推进。免雨示例不建立第二套任务，说明自来源键与其他免雨消费者的边界。
- stewer 的 CanCook 只查满格，StartCooking 不作全部状态验证；按锅 Prefab 的食谱表算结果，立刻消耗材料。onspoil 需由宿主实际改产物，OnRemoveFromEntity 不负责完整停机。
- cooktimemult 是新建任务倍率，已保存剩余时间不会重算；keepspoilage 保留原料平均新鲜比例，不等于禁止锅内腐败。制作配方与料理注册 API 都仍是独立缺口，不发布新锅或注册骨架。

## 检查与按需读取

七个 Wiki 文件（五页、物品主题与组件分类入口）通过格式、来源文件存在性、标签、大小、本地链接与标题锚点检查。全库导航登记为 89 篇知识页、12 篇导航、22 条 Top 30，无遗漏／重复／父入口问题；不等于全库语义核验。

五页 131 处源码行引用、105 个唯一目标、16 份来源文件 strict 检查为 0 error／0 warning，输出未截断。人工复核字段语义时另将 stewer 默认倍率锚点指到实际赋值行，而非可读但不支持该结论的字段；脚本通过不证明证据语义。

五段示例与页面 fenced Lua 完全一致，通过已有 Full Moon 3.0.0／Lua 5.1 syntax-only 探针；byte／position 明确跳过，不声称类型、往返或运行验证。五份临时 Lua 已删除，不安装依赖、不构建解析器。

五次作者受控的“典型接入”章节读取均保留快速判断、前提／完整代码与验证记录，truncated=false、omitted=0；输出 UTF-8 已与原页摘要核对：

| 页面／需求 | JSON stdout 实际 UTF-8 字节 |
|---|---:|
| cookable／直接烤出肉干 | 3,632 |
| cooker／营火恢复新鲜 | 4,213 |
| dryable／大肉两分钟 | 3,950 |
| dryingrack／雨天继续晾晒 | 4,513 |
| stewer／厨锅时长减半 | 4,276 |

已知组件名直达章节；未知名的“雨天继续晾晒”或“厨锅时间减半”从组件分类入口进入物品主题的“烹饪与干燥”，只筛五条登记再读所需页面。物品主题 60 行／4,146 字节，分类入口 25 行／1,288 字节；不增加必读导航层、不重建查询缓存。这是作者受控读取，不是独立模型自主发现评测，字节不换算 token 收益。

五条覆盖记录保留并核对源码 SHA-256；798 个组件候选现为 81 已建／717 未建，另 19 replica／3 helper／1 empty 单列。同步 PLAN、README、总体设计、文档入口与追加日志；历史报告及已有暂存／工作区变更保留。

## 后续与限制

第十三批完成；下一批 bundlemaker／bundler／unwrappable。第十四至十五批合计剩 8 个候选；heatrock Prefab、注册 API 与场景骨架队列继续保留。

本批未改 Skill、工具箱、游戏源码或输入模组，未提交／推送。全部页面保持 source-reviewed；未运行 DST。多人烹饪／占用、取件与失败消费、天气／酸雨、免雨来源、保鲜显示、离屏／保存恢复、烧毁及跨模组回调和倍率仍需实际游戏验证。
