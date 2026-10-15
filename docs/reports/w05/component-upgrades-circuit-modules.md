# 通用升级与电路模块组件检视

2026-10-21 完成第 77～81 批五页，依据 Steam buildid 25643519。沿用 dst-wiki Skill；准备脚本确认同 build，Skip，未更换快照。源码和输入模组只读，文本与临时 Lua 均通过 apply_patch 写入／清理，复用已有环境和解析器。

## 交付范围

| 批次／页面 | 实际行数／UTF-8 字节 | 完整 modmain 案例 |
|---|---|---|
| 77：[upgrader](../../../wiki/reference/components/upgrader.md) | 64／5,915 | 蜘蛛丝每件提供两点升级值，保留原类型／角色门禁与目标消费 |
| 78：[upgradeable](../../../wiki/reference/components/upgradeable.md) | 71／7,381 | 原蜘蛛巢每阶段累计门槛设三，不接管成长或清现有累计 |
| 79：[upgrademodule](../../../wiki/reference/components/upgrademodule.md) | 79／7,650 | 原健康电路激活时加 1.1 移动倍率贡献，失活成对撤销，保留原效果 |
| 80：[upgrademoduleowner](../../../wiki/reference/components/upgrademoduleowner.md) | 79／9,825 | 原 WX-78 服务端奖励事件补一格实际电量，保留夹限和激活重算 |
| 81：[upgrademoduleremover](../../../wiki/reference/components/upgrademoduleremover.md) | 77／8,021 | 原工具请求结束持有者对自己的检视，沿人物停止入口进入 SG 退出 |

五页合计 38,792 字节，采用五主节，复杂协作归接口下三级节；未增加类型、必填章节、空分类或完整新模块工程。示例注明原资源、文件／加载、作用对象、单位和撤销／保存边界。

## 核验结论与边界

- upgrader 的 CanUpgrade 只检查类型和执行者标签，不读取 silk 上的 canupgradefn 赋值。材料 upgradevalue 按每件消费给目标累计，不能按整堆乘算或宣称跨级；原动作还要检查目标 CanUpgrade。
- upgradeable 先累加、消费一件，再调 onupgradefn，然后至多 AdvanceStage 一次，清累计且不结转溢出。回调中的 obj 可能已被 Remove，回调返回不撤销消费；SetStage 和 OnLoad 不替代宿主阶段外观／成长，原蜘蛛巢另走 growable 和自身 SetStage。
- upgrademodule 的 target、activated 与实际安装分开；TryActivate／Deactivate 应由 owner 调度，移出前先撤能力，保留 target 给回调。原模块的静态 _slots／_type／_netid 与服务端字段分别维护，仅修改 setter 不证明 UI／网络一致。额外移动贡献复用同版本 locomotor 页面，不重新铺开其实现。
- upgrademoduleowner 按每栏电量前缀决定激活，PushModule 不重检容量，原 WX 容量回调、返还／拆卸电量／模块耐久与 classified 通知分别核验。关自动激活不保证撤掉已有能力；PopOneModule 返回零是旧兼容值，实际扣电在宿主回调，不能重复扣费或误称免费。
- upgrademoduleremover 是空标记。现代检视／手持动作、模块选择请求、人物 SG 与 owner 拔出有分工，REMOVEMODULE 不直接拔模块。旧复数 REMOVEMODULES 使用已废弃 NumModules()==0 的链，不能作为当前教程。停止例复用已绑定到 WX 的 StopInspectingModules，经当前 SG 接收事件退出，未强改标签或误关其他检视者。

核验完整五组件、UPGRADE／APPLYMODULE 及拆卸候选／动作、丝／Webber／蜘蛛巢宿主、原 WX／模块工厂／健康电路定义、classified 检视回调、UnplugModule 请求处理和 SG 消费／实际拔出入口。模块 owner 的保存重建与 WX 受模块影响的三维恢复分开记录。

未全面审校所有升级材料／目标、全部电路技能／网络注册／UI、防滥用请求链、手柄预测／中断、备份身体／影子插槽、人物换身或跨分片。页面均保留 source-reviewed，未运行游戏。

## 检查与受控读取

五页与四张受影响导航的元数据、来源文件、本地链接／锚点及大小通过。全库导航为 186 篇知识页、12 篇索引、22 条 Top 30，唯一登记／父入口无问题。strict 源码引用合并检查 134 处引用、85 个唯一目标、18 份来源文件，0 error／0 warning；truncated=false、omitted=0、by_page_omitted=0。

五份完整 Lua 示例以 apply_patch 临时另存，与页面围栏逐字核对，使用已有 Full Moon 二进制检查 Lua 5.1 syntax-only 后清理；未执行代码。完整“典型接入”读取保留元数据、快速判断、条件／示例和验证记录，complete_selected_ranges=true、source_version=match、truncated=false、omitted=0，Lua 往返一致。

| 页面 | “典型接入”JSON stdout 实际 UTF-8 字节 |
|---|---:|
| upgrader | 4,334 |
| upgradeable | 4,461 |
| upgrademodule | 5,453 |
| upgrademoduleowner | 4,999 |
| upgrademoduleremover | 5,452 |

字节仅作读取规模代理，不换算 token，不称独立模型自主评测或运行行为验收。共享准备／合并检查，未逐批独立测时，不据此估算全库工期。

## 登记与后续

upgrader／upgrademodule／upgrademoduleremover 唯一登记于物品，upgradeable 登记于交互技能，upgrademoduleowner 登记于人物；人物 18／物品 36／战斗 13／世界 66／交互技能 45，共 178 篇组件，加 3 篇 modinfo 和 5 篇 cookbook，共 186 篇知识页。全部 178 个已建 CSV 记录的页面集合与实际文件一致，来源 SHA-256 与原快照匹配。

798 个组件候选中 178 已建、620 未建，另 19 replica／3 helper／1 empty 单列。五项来自原扩展池，余扩展 392／专项 225／职责待确认 3，历史路线不改写。组件根入口 25 行／2,122 字节，人物主题 66 行／4,004 字节、物品主题 109 行／7,110 字节、交互技能主题 98 行／8,862 字节，均未超限，正文地址稳定。

同步覆盖 CSV、四张导航、PLAN、README、设计与 docs 入口，日志追加一次。按请求在第 81 批收尾；下次第 82 批候选 setbonus／damagetyperesist，先核验套装匹配、伤害类别消费者与生效／撤销。Prefab、配方 API 和 cookbook 原队列保留。

本轮临时 Lua 已清理；未安装依赖、编译解析器、运行游戏或独立模型评测、重建缓存、提交或推送。源码、Skill 包、工具箱和历史报告未修改。

收尾复核：五份工程 Markdown 的 145 处本地链接／锚点无错误，git diff --check 通过；日志为 258 个物理行／38,744 字节，未触发归档；第 77～81 批临时 Lua 为零。
