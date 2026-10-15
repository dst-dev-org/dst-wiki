# 套装与伤害抗性组件检视

2026-10-21 完成第 82～86 批五页，依据 Steam buildid 25643519。沿用 dst-wiki Skill；准备脚本确认同 build，Skip，未更换快照。源码只读，文本与临时 Lua 均通过 apply_patch 写入／清理，复用已有环境和解析器。

## 交付范围

| 批次／页面 | 实际行数／UTF-8 字节 | 完整 modmain 案例 |
|---|---|---|
| 82：[setbonus](../../../wiki/reference/components/setbonus.md) | 76／6,457 | 原月亮植物甲套装启用时加 0.9 月亮类别抗性贡献，禁用成对撤销 |
| 83：[damagetypebonus](../../../wiki/reference/components/damagetypebonus.md) | 71／6,789 | 原玻璃刀对 shadow_aligned 目标额外乘 1.25，保留原 shadowsubmissive 加成 |
| 84：[damagetyperesist](../../../wiki/reference/components/damagetyperesist.md) | 69／6,615 | 原月亮植物甲增加 shadow_aligned 来源的 0.8 剩余伤害倍率贡献 |
| 85：[resistance](../../../wiki/reference/components/resistance.md) | 72／6,847 | 原骨甲仅黄昏／夜晚允许原抵挡，保留原资格、燃料和冷却 |
| 86：[explosiveresist](../../../wiki/reference/components/explosiveresist.md) | 72／6,412 | 原熊獾爆炸抗性衰减尺度改 20 秒，不改累积／延迟或设初始抗性 |

五页合计 33,120 字节，均为五主节；来源协作及爆炸结算归主节下三级节。无新类型、必填章节、空分类或完整新装备工程，示例标明文件、原资源、服务端、作用条件、倍率单位及撤销／保存边界。

## 核验结论与边界

- setbonus 的基础门禁固定 HEAD／BODY 同名，成立后遍历装备，启用同名且尚未启用的组件；卸下清槽后还禁用自身。先置 enabled 再回调，组件不生成任何数值效果。SetSetName 的大写检查不改变实际存值，新增多栏／任意组合不自动成立。
- damagetypebonus 查目标标签，不查攻击者标签。同 tag 多来源／键、不同匹配 tag 和动态回调相乘；nil 目标返回 1。武器与人物贡献可以叠加，武器消费同时处理普通／特殊伤害，爆炸源消费还影响工作量。原玻璃刀和原月亮植物剑用的是不同标签，不能当作别名。
- damagetyperesist 查攻击者 OR 武器，同 tag 命中一次，不同 tag 相乘；attacker=nil 时不检查武器。pct 为剩余倍率。装备在库存内作用于普通／特殊伤害，自身 combat 是另一消费位置；当前条件受伤分支与后续普通伤害行均含 damagetypemult，不笼统承诺所有自定义宿主恰好乘一次。
- resistance 是标签集合及资格／通知，没有来源计数。ResistDamage 自身只回调与发事件，实际库存通过后 return 0,nil；不能称仅挡普通伤害或手动通知等于完成挡伤。骨甲回调会临时置 nil 并由宿主重设，因此例子只包 shouldresistfn，保留冷却／燃料／穿卸清理；注册名为 armorskeleton。
- explosiveresist 在 GetAttacked 前累积，输入是已乘本次爆炸抗性的普通 dmg，不是护甲后的生命损失；同处特殊伤害不乘该抗性。设置当前值会启停更新，延迟跨帧不精确拆 dt；保存向下取整百分数且不保存剩余延迟，未提供自身 LongUpdate。

核验完整五组件、SourceModifierList、库存套装穿卸／防御入口、weapon:GetDamage、combat 所引倍率分支、explosive 普通／特殊与工作量调用、原月亮植物甲／骨甲宿主，以及玻璃刀／月亮植物剑／熊獾必要配置和套装／调参常量。跨页复用来源键协议，不在抗性页复制完整实现。

未全面审校全部套装／武器／目标标签、多栏模组、骑乘、自定义伤害／条件倍率、特殊伤害系统、熊獾变体、离线补时或 UI／网络链。五页保持 source-reviewed，未运行游戏。

## 检查与受控读取

五页与两张受影响导航的元数据、来源文件、本地链接／锚点及大小通过；全库导航为 191 篇知识页、12 篇索引、22 条 Top 30，唯一登记／父入口无问题。strict 源码引用合并检查 108 处引用、79 个唯一目标、17 份来源文件，0 error／0 warning；truncated=false、omitted=0、by_page_omitted=0。

五份完整 Lua 通过 apply_patch 临时另存，与页面围栏逐字核对，使用已有 Full Moon 二进制检查 Lua 5.1 syntax-only 后清理，未执行代码。五次“典型接入”读取保留元数据、快速判断、完整案例和验证记录；complete_selected_ranges=true、source_version=match、truncated=false、omitted=0，读取结果与原 Lua 围栏逐字一致。

| 页面 | “典型接入”JSON stdout 实际 UTF-8 字节 |
|---|---:|
| setbonus | 4,666 |
| damagetypebonus | 4,336 |
| damagetyperesist | 4,408 |
| resistance | 4,796 |
| explosiveresist | 4,488 |

字节仅作读取规模代理，不换算 token，不称独立模型自主评测或游戏验收。共享准备／合并检查，未逐批独立测时，不据此估算全库工期。

## 登记与后续

五页唯一登记于战斗主题，补“套装与伤害类别”小节，爆炸抗性归原投射物／爆炸小节。主题分布为人物 18／物品 36／战斗 18／世界 66／交互技能 45，共 183 篇组件，加 3 篇 modinfo 和 5 篇 cookbook，共 191 篇知识页。全部 183 个已建 CSV 记录与实际页面集合一致，来源 SHA-256 匹配当前快照。

798 个组件候选中 183 已建、615 未建，另 19 replica／3 helper／1 empty 单列。五项来自原扩展池，剩扩展 387／专项 225／职责待确认 3，历史路线不改写。组件根入口 25 行／2,176 字节，战斗主题 46 行／3,333 字节，其他主题保持 66～143 行；正文地址稳定，无页面或导航超限。

同步覆盖 CSV、两张导航、PLAN、README、设计与 docs 入口，日志追加一次。按请求在第 86 批收尾；下次第 87 批候选 parryweapon／pinnable，先核验格挡／控制消费者与宿主恢复。Prefab、配方 API 和 cookbook 原队列保留。

本轮临时 Lua 已清理；未安装依赖、编译解析器、运行游戏或独立模型评测、重建缓存、提交或推送。源码、Skill 包、工具箱和历史报告未修改。

收尾复核：五份工程 Markdown 的 147 处本地链接／锚点无错误，git diff --check 通过；按原路线重算剩余扩展池为 387。日志为 264 个物理行／39,701 字节，未触发归档；第 82～86 批临时 Lua 为零。
