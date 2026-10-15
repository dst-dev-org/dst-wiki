# 治疗与修复组件检视

2026-10-17；Steam buildid 25643519。使用当前 dst-wiki Skill 核验实现、动作候选／执行与必要消费者，再发布第九批五页。源码准备返回 Skip，同 build 未解压；未启动独立模型评测或游戏。

## 交付范围

| 页面 | 重点 | 物理行／实际 UTF-8 字节 |
|---|---|---:|
| [healer](../../../wiki/reference/components/healer.md) | 当前生命、执行者倍率、接受／完成回调与消费 | 71／7,504 |
| [maxhealer](../../../wiki/reference/components/maxhealer.md) | 有符号惩罚比例、强心针、HEAL 优先级与无效果消费 | 70／6,984 |
| [repairable](../../../wiki/reference/components/repairable.md) | 四分支顺序、初始化标签、测试／消费顺序与斧头修复 | 102／11,979 |
| [repairer](../../../wiki/reference/components/repairer.md) | 材料单位／标签、全匹配目标影响与噩梦燃料 | 77／8,864 |
| [sewing](../../../wiki/reference/components/sewing.md) | USAGE 衣物、燃料量、工具次数／堆叠与回调 | 80／8,665 |

当前 63 篇组件、3 篇 modinfo、5 篇 cookbook，共 71 篇知识页与 7 篇导航；798 个组件候选中 735 未建。覆盖 CSV 仅登记本批五行，源码哈希与现有记录相符；组件索引 148 行／11,748 字节。既有正文及锚点不迁移，主题导航拆分保留待办。

## 核验所得边界

- healer:Heal 对有效 doer 的 HEAL 倍率取值；canheal=false 时不回血但仍可回调／消费，onheal 不是净恢复成功事件。HEAL 动作的死亡／幽灵检查不在组件方法中。
- maxhealer:SetHealthAmount 的实际值直接加到 penalty，负数恢复、正数加重；旧注释不能把数值解释为复活次数。恢复可用上限不等于补当前生命，零惩罚仍消费，且 HEAL 优先选 healer。
- repairable 的生命→工作量→保鲜→次数分支独占；已命中分支拒绝后不回落。生命固定量与比例走两次 DoDelta，成功不证明净恢复；onrepaired 在材料消费之后。
- 构造 repairable 不初始化 finiteuses 缺损标签，案例主动同步；后续由 finiteuses 属性回调更新。NeedsRepairs 使用 95% 提示阈值，不是实际 Repair 的最终门禁。
- repairer 的五项数值维护候选标签，目标决定真正结算；自定义字符串不自动进入 MATERIALS 枚举。修改材料影响所有匹配目标，材料组件自身没有 Repair。
- sewing 直接恢复 fueled 单位，不是百分比；有限次数工具优先 Use(1)，否则才取 stackable 一件，没有两者不自动消费。ONSEWN 回调在消费后，原版工具还发人物 repair 事件；船补能力来自另一个组件。

## 按章节读取检视

作者受控读取，目标为本页实际 demo；每项都返回快速判断、完整代码与前提／生效边界、验证记录，未截断或省略。不是独立模型自主发现评测，也不将输出字节换算成 token 收益。

| 需求 | 目标页 | JSON stdout 实际 UTF-8 字节 |
|---|---|---:|
| 普通药膏基础回复四十点 | healer | 4,489 |
| 强心针减少五十个百分点惩罚 | maxhealer | 4,454 |
| 普通斧头用石头修复二十次 | repairable | 6,063 |
| 噩梦燃料恢复二十五次耐久 | repairer | 5,018 |
| 针线包修补量翻倍、保留五次 | sewing | 4,804 |

查询使用工具生成的规范锚点；sewing 标题的中文逗号在锚点中移除。首次传原文逗号未命中，改用实际锚点后通过，不修改工具或声称首次读取成功。

## 检查与未覆盖范围

- 六个 Wiki 文件（五新页＋组件索引）格式、来源文件、标签、大小、链接／锚点检查通过；全库唯一登记／父入口／Top 30 检查为 71 篇知识页、7 篇导航、22 条快捷入口。
- 本批 170 处源码引用、110 个唯一目标、28 份来源文件 strict 检查 0 error／0 warning、未截断；修正初查发现的 sewing 一处空行引用。锚点检查不证明结论语义。
- 五段独立 modmain 示例经已有 Full Moon 3.0.0／Lua 5.1 探针 syntax-only 通过；输出明确跳过 byte/position，不宣称字节往返、类型、资源或游戏行为通过。
- 工程进度／设计／文档入口及本报告的 85 处本地链接／锚点人工脚本核对通过；git diff --check 通过。
- 本轮五份临时 Lua 与其临时目录已自动清理；未写游戏源码／模组输入／Skill／工具箱，未全量提取、重建缓存、提交或推送。既有暂存和工作区修改保留。
- 所有新页保持 source-reviewed。未验证多人 UI、最终游戏结算、特殊角色／模式、读档与跨模组覆盖；这些限制分别写入页面，不因结构和语法通过升级。

第九批完成。后续内容为第十批投射物与爆炸；组件主题导航、heatrock Prefab、配方与骨架队列仍保留。
