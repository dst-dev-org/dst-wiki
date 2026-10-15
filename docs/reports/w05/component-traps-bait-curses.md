# 陷阱、诱饵与诅咒物品组件检视

2026-10-21 完成第 72～76 批五页，依据 Steam buildid 25643519。沿用 dst-wiki Skill；准备脚本确认同 build，Skip，未更新快照。游戏源码与输入模组只读，文本和临时 Lua 通过 apply_patch 写入／清理，复用已有环境和解析器。

## 交付范围

| 批次／页面 | 实际行数／UTF-8 字节 | 完整 modmain 案例 |
|---|---|---|
| 72：[mine](../../../wiki/reference/components/mine.md) | 69／7,470 | 普通犬牙陷阱搜索半径改二，保留阵营、伤害和次数责任方 |
| 73：[trap](../../../wiki/reference/components/trap.md) | 73／8,678 | 普通捕捉陷阱范围改二，不重置现有捕获或改变 SG／收获 |
| 74：[bait](../../../wiki/reference/components/bait.md) | 74／7,047 | 服务端解除熟胡萝卜与 Trap 的合法双向关系，保留诱饵实体 |
| 75：[cursable](../../../wiki/reference/components/cursable.md) | 78／8,424 | 服务端净化事件核对库存与计数后消费一枚代币、减一层猴子诅咒 |
| 76：[curseditem](../../../wiki/reference/components/curseditem.md) | 76／8,432 | 临时抑制重复拾取 Given 的逃不掉诅咒台词，保留首次施咒和原追踪 |

合计 40,051 字节，采用五主节，复杂宿主协作归接口下三级小节；没有新增类型、必填章节、空分类或完整造物工程。示例注明目标文件、原资源、条件、效果／撤销和局部范围。

## 核验结论与边界

- mine 的搜索与 Explode 分开，直接 Explode 不检半径／阵营／当前状态，实际伤害和耐久在犬牙宿主回调。Spring 只还原触发状态／显示，加载不会因此再攻击；自定义搜索标签分支不能默认为仍有默认 combat／存活测试。重置候选的不可复用标签不阻止直接 Reset。
- trap 经 springtrap／SG／DoSpring 捕获，再记录有限 Prefab／lootdata 并删除原目标；Harvest 新建产物、恢复扩展数据并推进腐败。Reset 会清已有产物，不能用于无损刷新；原收获回调扣最后一次可先删除陷阱，源码 WARNING 已归档到限制和接口。鸟类宿主另有离屏捕获，组件范围并不覆盖所有概率路径。
- bait 保存 Trap 组件引用，目标 SetBait／RemoveBait 管双向关系／层序；被吃和被偷转发执行者。RemoveBait 不销毁物品、移除 bait 组件也不等于正常脱钩；撤下后附近生物或后续陷阱结算仍可能消费饵物。保存关联由 trap GUID／postpass 负责。
- cursable 施咒按整堆数增加，ApplyCurse 没有自带资格／去重，且 nil item 会在末尾出错。RemoveCurse 按请求数量减计数，即使库存实际不足；净化例先确认计数／同类库存，避免盲减。强制收纳可挤物，死亡调用与猴化恢复由实际 SG／工具协作，不是直接清字段。
- curseditem 的 target 与 cursed_target 分别指待追玩家和绑定主人；active 在此组件中只改标签，不阻止 OnUpdate／Given。原寻人只选择最近候选，空间不足不在同次搜索中遍历第二名；拆堆仅复制部分引用／标签，不视为完整保存协议。静默例仅包 Given，不影响首次猴化工具的提示或已有台词任务。

核验完整五组件、必要动作／候选、普通犬牙与变体、捕捉／鸟类宿主与 SGtrap、蔬菜 cooked 工厂、原猴子代币、玩家公共挂载、猴化工具、SGwilson 死亡调用和猴王净化入口，及拆堆诅咒字段复制。未把注释掉的 curses 保存／加载代码当现行恢复实现。

未完整审校全部陷阱／生物 Brain／SG、鸟生成器、魂分发、完整变身／换人保存、库存或堆叠并发、跨分片与坏主人引用。具体缺口留各页；均 source-reviewed，未运行游戏。

## 检查与受控读取

五页与四张受影响导航的元数据、来源文件、本地链接／锚点及大小通过。全库导航为 181 篇知识页、12 篇索引、22 条 Top 30，唯一登记／父入口无问题。strict 源码引用合并检查 152 处引用、101 个唯一目标、18 份来源文件，0 error／0 warning；truncated=false、omitted=0、by_page_omitted=0。

五份完整 Lua 示例临时另存，与页面围栏逐字核对，使用已有 Full Moon 二进制检查 Lua 5.1 syntax-only 后以 apply_patch 清理；没有执行示例。完整“典型接入”读取保留元数据、快速判断、前提、示例与验证记录，complete_selected_ranges=true、source_version=match、truncated=false、omitted=0，代码往返一致。

| 页面 | “典型接入”JSON stdout 实际 UTF-8 字节 |
|---|---:|
| mine | 4,385 |
| trap | 4,632 |
| bait | 4,769 |
| cursable | 5,535 |
| curseditem | 5,051 |

字节仅作读取规模代理，不换算 token、冒充游戏运行或独立模型自主评测。共享准备／合并检查，未逐批独立测时，不据此估算全库工期。

## 登记与后续

mine／trap 登记于交互技能，bait／curseditem 登记于物品，cursable 登记于人物，各唯一登记；人物 17／物品 33／战斗 13／世界 66／交互技能 44，共 173 篇组件，加 3 篇 modinfo 和 5 篇 cookbook，共 181 篇知识页。全部 173 个已建 CSV 记录与实际页面集合一致，来源 SHA-256 与原快照字节匹配。

798 个组件候选中 173 已建、625 未建，另 19 replica／3 helper／1 empty 单列。五项均来自原扩展池，余扩展 397／专项 225／职责待确认 3，历史路线不改写。组件根入口 25 行／2,041 字节；人物主题 60 行／3,722 字节、物品主题 101 行／6,450 字节、交互技能主题 92 行／8,595 字节，均未超限，正文地址稳定。

同步覆盖 CSV、四张导航、PLAN、README、设计与 docs 入口，日志追加一次。按请求在第 76 批收尾；下次第 77 批候选 upgrader／upgradeable，先核验材料／目标匹配、阶段／消费及宿主恢复；Prefab、配方 API 和 cookbook 原队列保留。

本轮临时 Lua 已清理；未安装依赖、编译解析器、运行游戏或独立模型评测、重建缓存、提交或推送。未修改源码、Skill 包、工具箱或历史报告。

收尾复核：五份工程 Markdown 的 143 处本地链接／锚点无错误，git diff --check 通过；日志为 252 个物理行／37,739 字节，未触发归档；第 72～76 批临时 Lua 为零。
