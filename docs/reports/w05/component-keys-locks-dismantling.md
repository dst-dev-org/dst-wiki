# 钥匙、目标锁与怀表拆解组件检视

2026-10-20 完成第 62～66 批五页，依据 Steam buildid 25643519。使用 dst-wiki Skill；源码准备确认同 build，Skip，未更换快照。游戏源码与输入模组只读，文本与临时 Lua 通过 apply_patch 写入和清理，复用现有环境／二进制。

## 交付范围

| 批次／页面 | 实际行数／UTF-8 字节 | 完整 modmain 案例 |
|---|---|---|
| 62：[key](../../../wiki/reference/components/key.md) | 68／6,158 | 原探测杖使用回调给底座发模组通知，保留原回调与后续钥匙处理 |
| 63：[lock](../../../wiki/reference/components/lock.md) | 80／8,053 | 服务端事件校验原探测杖、玩家持有者、类型与卡住状态，再进入原底座解锁 |
| 64：[klaussackkey](../../../wiki/reference/components/klaussackkey.md) | 65／6,942 | 普通一号鹿角增加原暴食公园门的 gate_key 扩展值，不转真钥匙 |
| 65：[klaussacklock](../../../wiki/reference/components/klaussacklock.md) | 77／6,982 | 原赃物袋要求有效非幽灵玩家，透传原回调的成功／原因／消费三值 |
| 66：[pocketwatch_dismantler](../../../wiki/reference/components/pocketwatch-dismantler.md) | 71／6,916 | 精密工具执行资格拒绝拆原回溯表，保留其他怀表冷却／角色门禁 |

合计 35,051 字节，均采用五主节且未超大小限制。未新增类型、必填章节、空分类或完整新造物工程；例子沿用实际原宿主，明确可见性、事件／动作边界及停用后不回滚已发生的业务。

## 关键结论与证据范围

- 旧式 key／lock 的类型标签、动作候选和执行分开；Unlock 不重检 keytype、不返回成功，UNLOCK 动作本身可在未解锁时仍 true。钥匙 OnUsed 调用漏传 doer，不据此构造玩家身份；使用回调发生在拆堆／隐藏之前，不允许示例删除待处理钥匙。
- lock 先改状态／调用宿主，再留存钥匙实体；SetLocked 不处理钥匙且回调看到旧状态，OnLoad 也可重放回调。原底座 ready 解除卡住，解锁回调又设置 isstuck，现动作的重新上锁分支是注释。本例不承诺旧传送装置在普通世界可用，也不绕过就绪或自动归还。
- klaussackkey 的 truekey 默认 nil，原真假由 Prefab 工厂决定；SetTrueKey 通知世界恢复关联，false 不发撤销事件，组件没有自动保存真假／keytype。门使用 keytype 是宿主扩展；普通鹿角例不转真、不接管世界登记，原赃物袋仍视其为错误钥匙。
- klaussacklock 将三个回调结果分别处理，先业务后消费，false,WRONGKEY,true 仍可召唤／掉骨片并扣一件；组件没有内建锁状态、通用匹配或保存。原袋子与门的业务各在宿主，本例保留原消费责任方，不自行销毁钥匙。
- pocketwatch_dismantler 的 CanDismantle 只处理冷却／clockmaker，原候选与工厂提供其他前提；Dismantle 直接调用会绕过资格。正常路径完整展开配方、排除 nightmarefuel，拟态另走 TurnEvil；实际发材料在 target:Remove 之前，与该行注释相反。未依据注释承诺库存先腾空。

核验完整五组件、必要动作／候选、原探测杖／底座、鹿角真假注册、袋子回调与钥匙登记、世界恢复消费者、门匹配／解锁保存、精密工具／怀表工厂和配方展开。持有者规则按同版本 inventoryitem 页复用，不重新展开无关库存实现。

未全面审校旧传送装置、暴食模式加载、Klaus 行为／掉落表、全部门网络／保存、拟态转化、所有怀表配方／SG、库存溢出／跨分片或模组包装冲突。页面保留具体未覆盖范围，不升级为 runtime-tested。

## 检查与受控读取

五页与三张受影响导航的元数据、来源文件、本地链接／锚点、大小通过；全库导航为 171 篇知识页、12 篇索引、22 条 Top 30，唯一登记与父入口无问题。源码 strict 合并检查 122 处引用、77 个唯一目标、18 份源文件，0 error／0 warning；truncated=false、omitted=0、by_page_omitted=0。

五份完整示例各以 apply_patch 临时另存，与页面 Lua 围栏逐字核对，已有 Full Moon 二进制执行 Lua 5.1 syntax-only 通过后清理。它只证明语法，不证明名称绑定、游戏交互或业务结果。初查修正了探测杖引用越界、空行／分支起点，以及底座配置引用落点；修后适用检查通过。

| 页面 | “典型接入”JSON stdout 实际 UTF-8 字节 |
|---|---:|
| key | 4,953 |
| lock | 5,423 |
| klaussackkey | 5,172 |
| klaussacklock | 5,033 |
| pocketwatch_dismantler | 5,012 |

章节读取均保留元数据、快速判断、完整示例／条件与验证记录；complete_selected_ranges=true、source_version=match、truncated=false、omitted=0，Lua 往返一致。字节仅作规模代理，不换算 token，不称独立模型自主评测。五批共享准备并合并检查，未逐批独立测时，不由本轮外推全库工期。

## 登记与后续

四张钥匙／锁页唯一登记于交互技能主题，拆表页登记于物品主题；人物 16／物品 31／战斗 13／世界 66／交互技能 37，共 163 篇组件。加 3 篇 modinfo 和 5 篇 cookbook，合计 171 篇知识页。全部 163 个已建 CSV 记录与实际页面集合一致、来源 SHA-256 与原快照字节匹配。

798 个组件候选中 163 已建、635 未建；19 replica／3 helper／1 empty 继续单列。旧式 key／lock 来自原扩展池，克劳斯两项及拆表工具来自专项池；剩扩展 407／专项 225／职责待确认 3，历史路线不重写。组件入口 25 行／1,921 字节，交互技能主题 70 行／7,094 字节，物品主题 94 行／6,027 字节，均未超限且正文地址稳定。

同步覆盖 CSV、三张导航、PLAN、README、设计与 docs 入口，日志追加一次。按请求到第 66 批收尾；下次第 67 批候选 machine／circuitnode，先核验宿主、门禁与开关／回调，不以名称推断职责；Prefab、配方 API 与 cookbook 原队列保留。

临时 Lua 已清理；未安装依赖、编译解析器、运行游戏或独立模型评测、重建缓存、提交或推送。源码、Skill 包、工具箱和历史报告未修改。

收尾复核：五份工程 Markdown 的 139 处本地链接／锚点无错误，git diff --check 通过；日志为 240 个物理行／35,790 字节，未触发归档；第 62～66 批临时 Lua 为零。
