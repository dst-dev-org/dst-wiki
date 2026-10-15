# 农作与施肥基础组件检视

2026-10-17；Steam buildid 25643519。按 dst-wiki Skill 完成第十六批四页；源码准备返回 Skip，未解压。文本和临时示例通过 apply_patch 写入／清理。

## 交付与边界

| 页面 | 核验与完整 modmain 示例 | 物理行／实际 UTF-8 字节 |
|---|---|---:|
| [fertilizer](../../../wiki/reference/components/fertilizer.md) | 养分／旧字段、部署与消耗；便便养分翻倍且不污染共享表 | 72／7,206 |
| [fertilizable](../../../wiki/reference/components/fertilizable.md) | 自肥委托、回调拼写与成功结算；沃姆伍德成功自肥加 3 精神 | 69／5,720 |
| [farmtiller](../../../wiki/reference/components/farmtiller.md) | 农田资格、土堆与动作消耗；普通园艺锄基础消耗改 0.5 | 64／5,912 |
| [farmplanttendable](../../../wiki/reference/components/farmplanttendable.md) | 开关／标签、回调与阶段重开；照料种植胡萝卜加 1 精神 | 70／6,526 |

- 现代土壤施肥走 deployable helper，使用三养分和地块坐标，各养分限制到 0～100；fertilizervalue／soil_cycles／withered_cycles 由旧 crop／grower／pickable 路径分别消费。改便便养分也影响人物自肥，不能宣称只改农田。
- fertilizable 是薄回调，实际字段拼写为 onfertlizedfn；普通目标施肥不调此组件，已核验原版用途为执行者自肥。保留原沃姆伍德回调与返回，成功后的肥料消费交由动作，示例不重复 OnApplied。
- farmtiller 只负责合法农田锄地／生成土堆／tilling 事件；成功动作后的物品通知和 finiteuses 才负责耗损。示例基础消耗 0.5 仍可能叠加 efficientuser 或其他消费改写。
- farmplanttendable 只在回调真值时关闭开关；原植物分阶段重新开放，魔法照料可不传 doer。示例奖励真实照料者，不能称为每株终生一次或消除全部压力。

## 检查与读取

六个 Wiki 文件（四页、世界主题与组件分类）通过元数据、来源文件、标签、大小、本地链接及标题检查；全库导航为 101 篇知识页、12 篇导航、22 条 Top 30，无遗漏、重复登记或父入口问题，不代表全库语义复核。

四页 91 处源码行引用、72 个唯一目标、24 份来源文件 strict 检查为 0 error／0 warning，无截断／省略。首次检查发现一处阶段表起点落在结构行，已按实际内容移至 name 字段并复查。四条源码 SHA-256 均与覆盖清单匹配；798 候选现为 93 已建／705 未建，另 19 replica／3 helper／1 empty 单列。

四段临时 Lua 与各页完整示例逐项一致，通过已有 Full Moon Lua 5.1 syntax-only；byte／position 检查按该模式跳过，不声称运行、类型或 API 绑定通过。临时文件已清理，未安装依赖或构建解析器。

受控读取各页“典型接入”，均带快速判断、完整代码／条件与验证记录，complete_selected_ranges=true、truncated=false、omitted=0：

| 页面 | JSON stdout 实际 UTF-8 字节（换行按 LF） |
|---|---:|
| fertilizer | 4,573 |
| fertilizable | 4,421 |
| farmtiller | 4,326 |
| farmplanttendable | 4,670 |

未知名称的施肥／照料需求从组件分类进入世界主题“农作与施肥基础”，筛四条后直读所需页；已知名称仍直达。世界主题 74 行／5,311 字节，分类 25 行／1,351 字节，无新增层级。此为作者受控读取，不是独立模型自主发现或 token 收益评测，未重建缓存。

## 后续与限制

同步覆盖 CSV、PLAN、README、design、docs 入口与追加日志；五份本批工程 Markdown 的 98 处本地引用／锚点检查通过，git diff --check 通过（仅行尾转换提示）。当前日志 114 行／16,338 字节，无需归档；四段临时示例已删除，未留 batch16 临时文件。

下一批拟补 farmplantable／farmsoildrinker／farmplantstress，先核验种子／土堆、饮水统计及压力阶段协作；farming_manager 后续独立小批。首篇 Prefab heatrock、配方注册 API 与 cookbook 队列保留。

本批未改 Skill、工具箱、游戏源码或输入模组，未提交／推送。游戏未运行，页面保持 source-reviewed；多人、手柄、耗尽／存档、阶段重开、魔法照料、角色技能／状态与其他模组包装顺序待实测，不把四页当完整新植物／现代农作骨架。
