# 机器、节点连接与标记代理组件检视

2026-10-20 完成第 67～71 批五页，依据 Steam buildid 25643519。沿用 dst-wiki Skill；准备脚本确认同 build，Skip，未更换源码。游戏源码和输入模组只读，文本及临时 Lua 均通过 apply_patch 写入／清理，复用已有环境与二进制。

## 交付范围

| 批次／页面 | 实际行数／UTF-8 字节 | 完整 modmain 案例 |
|---|---|---|
| 67：[machine](../../../wiki/reference/components/machine.md) | 68／6,976 | 原灭火器后续开关冷却设一秒，不接管探测／耗燃料 |
| 68：[circuitnode](../../../wiki/reference/components/circuitnode.md) | 73／7,822 | 服务端事件断开原聚光灯当前节点，保留宿主回调／后续重连 |
| 69：[electricconnector](../../../wiki/reference/components/electricconnector.md) | 73／8,641 | 原围栏 finder 半径最多四单位，不宣称约束直连／已有连线／重建 |
| 70：[markable](../../../wiki/reference/components/markable.md) | 73／7,321 | 原评选柱每柱最多四个当前标记，保留参与者条件和已有票取消 |
| 71：[markable_proxy](../../../wiki/reference/components/markable-proxy.md) | 79／7,712 | 原牛代理 Mark 执行前复查代理／目标开关与有效目标，不改投票记录 |

五页合计 38,472 字节，采用五主节；宿主清理、原版协作归接口下三级节，没有新增类型、必填章节、空分类或新造物工程。示例明确目标文件、资源、生效／撤销和未覆盖边界。

## 核验结论与边界

- machine 每次直接开关都重放回调／冷却／事件，不检查 enabled／冷却／groundonly；动作只检查当前开关状态。CanInteract 不读取 caninteractfn，原灭火器赋值不能当作该字段生效证据；加载会重放开关，冷却和 enabled 不作为剩余状态保存。
- circuitnode 是启用集合和直接邻居协议，不提供电量；连接回调先在本端发生再递归对端，范围／平台过滤只在发现入口。原电源 tick 调 AddBatteryPower，聚光灯加载只先启用，再由电源 postpass 发现；清理抑制本端断开回调，不能只靠它停止所有宿主任务。
- electricconnector 的 finder、CanLinkTo 和 ConnectTo 检查范围不同，不能把 finder 半径／对端满额标签当所有入口的硬约束。StartLinking 只发事件，SG 后续找对端；双方登记共享电场实体，最后断边才触发本端 onunlinkedfn；保存重建绕过 linking 状态但保留平台／重复限制。
- markable 先取消已有 doer 记录，再处理新资格；编号池八个不自动拒绝第九位，回调返回不决定记票，SetMarkable 仅改候选标签。原柱子承担彩带／跨柱转投／参与者条件，stage 管启停与 Unmarkall；空存读不支持持久投票。
- markable_proxy 转交同一目标，不复制 marks；SetMarkable 会同时修改目标，不是局部代理开关。原赛程先 false 再清 proxy，否则本方法不会清旧代理标签；目标值变化不自动同步所有代理，Mark 本身不复查候选开关。示例只补牛侧执行分支，不宣称修复所有赛程入口。

核验完整五组件及必要候选／动作、灭火器宿主、低级电源／聚光灯的连接与加载协作、电围栏宿主／SG／电场入口、牛／柱子／stage 的标记绑定与清理。全局 math 已由同版本起步 Wiki 的模组环境证据确认，无需为标准库增加 GLOBAL 代理前置。

未完整审校自动灭火、所有工程设备／功率分摊、灯光余量、电场分段碰撞／AI、完整比赛评分、NPC 调度或所有断线／平台／跨分片。每页保留具体验证缺口，均 source-reviewed，未运行游戏。

## 检查与受控读取

五页与两张受影响导航的元数据、来源、本地链接／锚点、大小通过。全库导航为 176 篇知识页、12 篇索引、22 条 Top 30，唯一登记／父入口无问题。strict 源码引用合并检查 147 处引用、112 个唯一目标、17 份来源文件，0 error／0 warning；truncated=false、omitted=0、by_page_omitted=0。

五份完整 Lua 示例通过 apply_patch 临时另存，先与页面逐字核对，再用已有 Full Moon 二进制检查 Lua 5.1 syntax-only，完成后删除临时文件；未执行示例。完整章节读取同时检查元数据、快速判断、示例／前提及验证记录，complete_selected_ranges=true、source_version=match、truncated=false、omitted=0，代码往返一致。

| 页面 | “典型接入”JSON stdout 实际 UTF-8 字节 |
|---|---:|
| machine | 4,422 |
| circuitnode | 4,746 |
| electricconnector | 5,027 |
| markable | 4,871 |
| markable_proxy | 5,214 |

字节只作读取规模代理，不换算 token 或称独立模型评测；共享准备、合并检查，未逐批独立测时，不据此外推全库工期或游戏正确性。

## 登记与后续

五页均唯一登记在交互技能主题：人物 16／物品 31／战斗 13／世界 66／交互技能 42，共 168 篇组件；加 3 篇 modinfo／5 篇 cookbook，共 176 篇知识页。全部 168 个已建 CSV 记录的页面集合与实际文件一致，来源 SHA-256 与原快照匹配。

798 个组件候选中 168 已建、630 未建，19 replica／3 helper／1 empty 单列。本轮五项来自原扩展池，现余扩展 402／专项 225／职责待确认 3；历史路线不重写。组件入口 25 行／1,963 字节，交互技能索引 85 行／8,178 字节，均未超限，正文地址稳定。

同步覆盖 CSV、两张导航、PLAN、README、设计与 docs 入口，日志追加一次。按请求在第 71 批收尾；下次第 72 批候选 mine／trap，先核验安放、触发、捕获／爆破和宿主清理；Prefab、配方 API 与 cookbook 队列保留。

本轮临时 Lua 已清理；未安装依赖、编译解析器、运行游戏或独立模型评测、重建缓存、提交或推送。未修改源码、Skill 包、工具箱或历史报告。

收尾复核：五份工程 Markdown 的 141 处本地链接／锚点无错误，git diff --check 通过；日志为 246 个物理行／36,741 字节，未触发归档；第 67～71 批临时 Lua 为零。
