# 格挡、控制与伤害记录组件检视

2026-10-21 完成第 87～91 组，共 12 篇组件页，依据 Steam buildid 25643519。按用户明确要求启动 5 个独立子代理，当前环境包含主代理最多 4 个并发，因此分 3＋2 两波调度。准备脚本确认同 build，Skip，未更换快照；源码只读。

## 分工与交付

| 组 | 子代理范围 | 页面数量 |
|---|---|---:|
| 87：格挡与闪避 | parryweapon、attackdodger | 2 |
| 88：定身与眩晕 | pinnable、rooted、stunnable | 3 |
| 89：电击与反伤 | electricattacks、damagereflect | 2 |
| 90：阈值与伤害记录 | healthtrigger、damagetracker、dpstracker | 3 |
| 91：位面与范围副效果 | planarentity、aoediminishingreturns | 2 |

每个代理使用独立精简任务，按 Skill 读取必要规则与源码，只写所属组件页及临时 Lua，执行所选页格式／源码引用／适用示例语法检查。主代理复核全部成页及关键语义，统一修改覆盖 CSV、索引、计划、报告和日志，避免并发写公共文件。没有新建用户聊天、工作树或额外仓库。

| 页面 | 实际行数／UTF-8 字节 | 原宿主接入或参照 |
|---|---|---|
| [parryweapon](../../../wiki/reference/components/parryweapon.md) | 75／8,993 | 原女武神盾牌实际招架角度改 120 度 |
| [attackdodger](../../../wiki/reference/components/attackdodger.md) | 78／8,970 | 伍迪原动态闪避组件的后续冷却配置乘 2 |
| [pinnable](../../../wiki/reference/components/pinnable.md) | 76／9,722 | 已有玩家黏液自然磨损时长改 6 秒 |
| [rooted](../../../wiki/reference/components/rooted.md) | 79／10,293 | 原藤蔓效果的初始 timer 改 3 秒，保留来源清理 |
| [stunnable](../../../wiki/reference/components/stunnable.md) | 79／9,764 | 原龙蝇倒地结束事件延时／冷却改 6／45 秒 |
| [electricattacks](../../../wiki/reference/components/electricattacks.md) | 75／9,055 | 原电羊添加独立持续电击来源，保留原充放电 |
| [damagereflect](../../../wiki/reference/components/damagereflect.md) | 76／10,008 | 原亮茄盔甲替换为固定普通反伤 20，明确移除原位面反伤 |
| [healthtrigger](../../../wiki/reference/components/healthtrigger.md) | 71／7,494 | 原龙蝇 80% 血量节点迁至 75%，保留原回调 |
| [damagetracker](../../../wiki/reference/components/damagetracker.md) | 70／7,796 | 原龙蝇倒地掉鳞累计门槛改 1000，保留启停与累计 |
| [dpstracker](../../../wiki/reference/components/dpstracker.md) | 73／8,888 | 原第四阶段月亮守护者威胁回调输入乘 1.25 |
| [planarentity](../../../wiki/reference/components/planarentity.md) | 65／8,190 | 原手部影怪组装与蜈蚣控制器 FX 参照；无完整新生物 demo |
| [aoediminishingreturns](../../../wiki/reference/components/aoediminishingreturns.md) | 87／10,074 | 原旋转模块激活时给副效果额外 0.8 贡献，失活成对撤销 |

12 页合计 109,247 字节，保持五主节；独立协作归主节下三级小节。11 份完整 modmain 示例标明原资源、加载、端、单位与生效／撤销。planarentity 已有宿主仅需解释实际组装和消费，没有数值 setter，不强造完整生物工程或重复添加代码。未新增模板、页面类型、空分类或规则。

## 语义核验与重要边界

- 招架由武器判定与玩家 SG 重定向协作，EnterParryState 依赖同步监听窗口；字段改变不自动改变客户端瞄准弧。attackdodger 的最终真值按条件结果或可用冷却放行，不是严格 AND，直接 Dodge 不检查资格。
- pinnable 管理黏液受困，玩家 SG 处理控制／动画；Stick 不自行执行恢复宽限查询。rooted 构造即施加物理／零移动来源，正常按来源撤销，空来源不自动到期；保存重施由实际效果宿主负责。stunnable 累计生命事件绝对量，治疗也入账、同时间键覆盖；只发事件，倒地由龙蝇 SG 完成，单独删组件不替它注销匿名监听或取消未保存句柄的任务。
- electricattacks 的真实攻击消费者检查组件存在性。来源实体自动移除只清来源表，不能保证移除组件，效果结束需显式 RemoveSource；电击倍率在 DoAttack 计算，直接传 electric 到 GetAttacked 不会自动补湿度倍率。
- damagereflect 先于目标防御预计算、攻击后返还；不按目标实际掉血决定是否成立，返还事件也不保证攻击者最终掉血。回调 nil／false 回退默认值，0 可明确无普通反伤。源码标注 unused 的 GetDamageReflect 仍调用组件不存在的旧方法，不能推荐该入口。
- healthtrigger 是双向跨线，不是持续低血检测或登记时补触发。damagetracker 是绝对事件量累计，启停不清零、治疗也计入、删组件可能残留监听。dpstracker 采样净生命差，首样本为零、回血可负、静默时缓存不自动归零；威胁降级来自宿主独立任务，不是组件计时能力。
- planarentity 对到达该步骤的普通伤害作非线性变换，特殊表原样返回；位面输出／位面数值防御归其他组件，FX 不证明具体伤害值。aoediminishingreturns 只是来源倍率容器，原旋转调用链用于战斧回血和工具热量，未自动缩放 combat 伤害；理智代价没有随吸血倍率缩放。

完整组件均已由所属代理审阅；关联文件只核验必要消费者、实际注册／构造、动作／SG、事件、来源、保存与清理。主代理复核组件真值／来源清理、攻击与反伤消费顺序、控制协作以及统计区别；已有同版本机制可复用，不重铺全部调用图。

未运行游戏，全部页面保持 source-reviewed。未全面审校全部伤害入口、联机预测、多栏模组、其他宿主、全部位面实体／旋转动作、完整热量系统或其他模组兼容；未知运行结果不因并行或静态通过而升级。

## 集成检查与读取

- 子代理局部检查：每组指定页面的格式／来源／本地链接与严格源码引用通过；11 份完整 Lua 逐字另存并通过已有 Full Moon Lua 5.1 syntax-only，均通过 apply_patch 清理临时文件，未执行示例。
- 主代理集成：12 页与两张受影响导航的格式、sources 存在性、链接／锚点和大小通过。导航为 203 篇知识页、12 篇索引、22 条 Top 30，唯一登记／父入口无问题。
- 合并源码引用：429 处引用、318 个唯一目标、57 份源码文件，strict 结果 0 error／0 warning；truncated=false、omitted=0、by_page_omitted=0。脚本只验证可读位置，不替代上文语义核验。
- 12 次“典型接入”读取均保留元数据、快速判断、完整所选节及验证记录，complete_selected_ranges=true、source_version=match、truncated=false、omitted=0。11 份 Lua 在读取结果中保持原文。
- 全部 195 个已建覆盖记录的页面集合与实际文件一致，主源码 SHA-256 匹配当前快照。逐页来源／技术结论的覆盖与缺口仍以页面验证记录为准。

| 页面 | “典型接入”JSON stdout 实际 UTF-8 字节 |
|---|---:|
| parryweapon | 4,812 |
| attackdodger | 5,857 |
| pinnable | 5,238 |
| rooted | 6,154 |
| stunnable | 4,902 |
| electricattacks | 5,505 |
| damagereflect | 5,530 |
| healthtrigger | 5,093 |
| damagetracker | 4,924 |
| dpstracker | 5,486 |
| planarentity | 5,421 |
| aoediminishingreturns | 6,124 |

以上字节仅为读取规模代理，不换算 token。未记录各代理的准确 token 或独立冷启动耗时，不能据此声称并行降低总消耗、整体查询收益或预测全库工期。仍需真实开发任务与游戏验证。

## 登记、同步与后续

12 页全部唯一登记于战斗主题，增“格挡与控制”“生命阈值与伤害记录”两个小节，其他条目归原有分组；稳定正文地址不变。主题分布为人物 18／物品 36／战斗 30／世界 66／交互技能 45，共 195 篇组件，加 3 篇 modinfo 与 5 篇 cookbook，共 203 篇知识页。

798 个组件候选中 195 已建、603 未建，另 19 replica／3 helper／1 empty 单列。12 项均来自原扩展池，按旧路线重算剩扩展 375／专项 225／职责待确认 3；历史盘点不改写。组件根入口 25 行／2,287 字节，战斗主题 68 行／5,301 字节；所有主题 66～143 行，均未超限，根 Top 30 未扩增。

同步覆盖 CSV、两张导航、PLAN、README、设计和 docs 入口；日志仅由主代理追加一次。按请求在第 91 组收尾，下次第 92 批候选 healthbar／healthsyncer，先核验宿主与显示／健康同步消费，不从名称承诺通用客户端血条。Prefab、配方 API 和 cookbook 队列保留。

本轮各组临时 Lua 均为零；源码、Skill 包、工具箱、既有知识页和历史报告未修改。未安装依赖、构建工具、运行游戏或独立模型评测、重建缓存、提交或推送。

收尾复核：五份工程 Markdown 的 156 处本地链接／锚点无错误，git diff --check 通过；按历史路线重算剩余扩展池为 375。日志 270 个物理行／40,593 字节，未触发归档；第 87～91 组临时 Lua 为零。
