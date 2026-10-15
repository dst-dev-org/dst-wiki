# 新物品与机制修改首轮任务检视

日期：2026-10-11；源码为 Steam buildid 25643519。已生成两份隔离模组并核对源码、语法及显式结构，W05 仍在进行。此次记录读取路径与返回量，未启动游戏，也未取得准确模型 token。

## 任务与产物

两项任务均从已存在的 `modinfo.lua`／`modmain.lua` 起步，没有本地物品工厂。测试入口取自已交付起步骨架，作为各路径的共同输入；正常新增任务不重复读取起步页。

| 任务 | 需求与实现 | 实际生成结果 |
|---|---|---|
| 新增物品 | 新建 `dstwiki_stacktoken`，复用燧石外观／图标，可检查、拾取／收纳；堆叠采用原版 SMALLITEM 档位，不要求配方或额外效果 | 保留既有入口与全局代理，追加 PrefabFiles／名称／检查语；工厂在服务端添加 inspectable、inventoryitem、stackable；当前上限为 40 |
| 修改机制 | 普通 `axe` 总次数为 200，保留初始化时的比例、CHOP 消耗与原版耗尽删除；不迁移旧存档的比例 | 仅新增服务端 Prefab postinit，调用 GetPercent → SetMaxUses → SetUses；没有新 Prefab／资源，没有覆盖 onfinished／消耗映射 |

本地产物分别位于 `scripts/output/w05-task-pilot-20261011/mods/dstwiki_stack_pilot/` 与 `mods/dstwiki_axe_pilot/`，均被 Git 忽略，未安装到游戏或修改用户的三个模组。它们是工程验收产物；共享骨架与机制代码仍由 [new-item](../../../wiki/cookbook/new-item.md)／[finiteuses](../../../wiki/reference/components/finiteuses.md) 维护，不新增重复 Wiki 案例页。

## 测量方法

同一作者在同一会话中读过相关内容后，使用固定任务和当前文件执行五条路径的**受控读取重放**。只生成一套共同结果，没有让独立模型分别盲测产物；源码路径也使用人工选定的必要区间，包含先验定位优势。此次可核对返回文本与组装缺口，不能证明模型自主路由成功率、每条路径独立生成的正确率或普遍成本收益。

累计 UTF-8 字节包含完整检索／章节 JSON、状态、重复元数据、源码行号和导航正文；截断后的补读计入。字符按返回字符串计，字节不是 token。单行 JSON 的物理行数不能表示内容规模；机器记录另保留所选正文的行数／字符和来源哈希。

各路径共同的规则／源码准备、目标入口检查、创建 Wiki／索引、编码输出和验收不计入表内；命令文本、外层工具封装、延迟及已有对话上下文也未计入，因此表是**内容返回量**，不是整个会话成本。读取操作数指记录的搜索／读取批次；批次可读取多个文件或章节，不等于外层工具 API 调用数。

| 任务／路径 | 读取操作 | 返回 UTF-8 字节 | 返回字符 | 直接读源码行数 |
|---|---:|---:|---:|---:|
| 新增／定向源码 | 3 | 20,665 | 20,665 | 541 |
| 新增／机制页＋源码补组装 | 4 | 34,916 | 28,894 | 281 |
| 新增／cookbook＋机制 | 2 | 16,569 | 11,511 | 0 |
| 修改／定向源码 | 2 | 12,182 | 12,182 | 297 |
| 修改／机制页 | 2 | 10,332 | 7,646 | 0 |

## 实际读取路径

新增的 cookbook 路径直接读短场景导航，选择 `new-item#最小-demo`，再合并读取 stackable 的关键接口、SMALLITEM／TINYITEM 示例与陷阱。没有读取根导航、起步页或无关装备／容器扩展；两个知识页的完整章节、条件与验证边界共返回 15,269 字节，另加导航 1,300 字节。本次知识已覆盖组装所需内容，路径内没有源码回退；独立验收时仍核对来源。

机制路径先搜索“可拾取物品”“物品堆叠”，候选数上限省略 12 项且有显式提示。一次读取 inventoryitem、inspectable、stackable 三份关键接口时超出 16,000 字节，最后一页整项省略；随后单独读取 stackable 及所需限制，未把省略当作已经读取。组件页不能代替完整 Prefab 注册、名称与资源／网络骨架，因此追加组装源码区间，保留已取得的机制上下文。这条路径的补读和回退使返回量增加。

修改任务搜索“修改已有斧头”，同时返回 finiteuses 与 tool；核对前者修改次数、后者修改工作效率后，只读 finiteuses 的案例、陷阱及生命周期，合并重复前提。没有读 cookbook 或 `new-item`，修改局部机制可以直接进入 reference。返回量略低于定向源码，本次不调整规则或宣称固定节省比例。

以下为实际 Wiki 重放命令，从知识库根执行；Python 路径按本机环境调整。这些工程命令不成为普通 Markdown 使用的前提。

```powershell
& .\.venv\Scripts\python.exe -X utf8 scripts/tools/wiki/query_wiki.py --build steam-buildid:25643519 search "可拾取物品" "物品堆叠" --path reference/components/
& .\.venv\Scripts\python.exe -X utf8 scripts/tools/wiki/query_wiki.py --build steam-buildid:25643519 read "reference/components/inventoryitem#关键接口" "reference/components/inspectable#关键接口" "reference/components/stackable#关键接口"
& .\.venv\Scripts\python.exe -X utf8 scripts/tools/wiki/query_wiki.py --build steam-buildid:25643519 read "reference/components/stackable#关键接口" "reference/components/stackable#将燧石上限提高到原版微小物品堆叠档位" "reference/components/stackable#常见陷阱与限制"
& .\.venv\Scripts\python.exe -X utf8 scripts/tools/wiki/query_wiki.py --build steam-buildid:25643519 read "cookbook/new-item#最小-demo" "reference/components/stackable#关键接口" "reference/components/stackable#将燧石上限提高到原版微小物品堆叠档位" "reference/components/stackable#常见陷阱与限制"
& .\.venv\Scripts\python.exe -X utf8 scripts/tools/wiki/query_wiki.py --build steam-buildid:25643519 search "修改已有斧头"
& .\.venv\Scripts\python.exe -X utf8 scripts/tools/wiki/query_wiki.py --build steam-buildid:25643519 read "reference/components/finiteuses#典型接入" "reference/components/finiteuses#常见陷阱与限制" "reference/components/finiteuses#生命周期与原版模式"
```

源码读取清单如下，路径相对 `raw/current/scripts/`，数字为一基行号、包含两端。重放按文件／区间添加行号而非读取整个文件；版本或页面变化后应重新测量，不沿用表内数字。

| 批次 | 已读取区间 |
|---|---|
| 新增组装；机制缺口也读此批次 | mods.lua 297–337／677–720；modindex.lua 144–159；mainfunctions.lua 145–179／347–372；prefabs.lua 1–17；prefabs/flint.lua 1–61；standardcomponents.lua 371–387；entityscript.lua 814–818；stringutil.lua 396–414 |
| 新增机制 | components/stackable.lua 1–35／72–96／99–119／159–201；components/stackable_replica.lua 1–10／47–58／98–111；tuning.lua 81–85；components/inventoryitem.lua 58–80／463–466；components/inventoryitem_replica.lua 131–168；simutil.lua 670–699 |
| 修改定义与调用 | components/finiteuses.lua 1–21／44–87／97–125；prefabs/axe.lua 38–68／86–141／291–294；modutil.lua 592–599；mainfunctions.lua 347–381／438–480；entityscript.lua 1963–1988 |

新增定位搜索 PrefabFiles／LoadPrefabFile／MakeInventoryPhysics／SetMaxSize／STACK_SIZE_SMALLITEM／ChangeImageName，限定上述 mods、mainfunctions、standardcomponents、stackable_replica、tuning、inventoryitem 文件；修改定位搜索 SetMaxUses／SetUses／OnLoad／SetOnFinished／AddPrefabPostInit／modfns／MakeAxe，限定 finiteuses、modutil、mainfunctions、axe。具体命令、所有原返回文本、区间与文件哈希由本地 `measure_tasks.py`／`measurements.json` 保存；它们被忽略，不随克隆分发，可依据本报告清单重新量取。

## 正确性与验证边界

| 核对项 | 依据与结果 |
|---|---|
| 保留既有入口，新增 Prefab 正确登记 | 工厂返回注册名与 PrefabFiles／文件名／字符串键一致；modmain 追加数组条目。[mods.lua:693](../../../raw/current/scripts/mods.lua#L693)、[prefabs.lua:6](../../../raw/current/scripts/prefabs.lua#L6) |
| 网络分界与资源 | 实体与动画沿原版燧石设置，服务端才添加组件；全端加载新工厂。图标为 flint，不查询不存在的新图元；资源可用性沿用骨架实际安装检查。[flint.lua:15](../../../raw/current/scripts/prefabs/flint.lua#L15)、[inventoryitem](../../../wiki/reference/components/inventoryitem.md) |
| 堆叠上限与状态 | SMALLITEM 当前为 40，并在 replica 支持的固定表内；默认每个新实例为 1，不直接塞入超限初始数量。合并／拆分／保存由原版组件处理。[tuning.lua:83](../../../raw/current/scripts/tuning.lua#L83)、[stackable_replica.lua:1](../../../raw/current/scripts/components/stackable_replica.lua#L1)、[stackable.lua:72](../../../raw/current/scripts/components/stackable.lua#L72) |
| 斧头初始化与原行为 | 普通 axe 工厂初始化 finiteuses、CHOP 消耗与 onfinished；postinit 后置，只改 total/current。[axe.lua:112](../../../raw/current/scripts/prefabs/axe.lua#L112)、[mainfunctions.lua:368](../../../raw/current/scripts/mainfunctions.lua#L368) |
| 比例与耗尽 | 原版新斧头 100/100 → 本例 200/200；若先前初始化为 50/100，则 100/200；耗尽仍走现有回调。这是实现推演，未执行 Lua 行为测试。[finiteuses.lua:60](../../../raw/current/scripts/components/finiteuses.lua#L60)、[:116](../../../raw/current/scripts/components/finiteuses.lua#L116)、[:64](../../../raw/current/scripts/components/finiteuses.lua#L64) |
| 旧存档恢复 | postinit 后宿主再 OnLoad，保存的绝对次数恢复；保存 50 次的旧物品将为 50/200，本例不声称保持旧存档百分比。[mainfunctions.lua:438](../../../raw/current/scripts/mainfunctions.lua#L438)、[entityscript.lua:1980](../../../raw/current/scripts/entityscript.lua#L1980)、[finiteuses.lua:54](../../../raw/current/scripts/components/finiteuses.lua#L54) |

五个生成 Lua 文件通过 Full Moon 3.0.0 的 Lua 5.1 语法与原字节往返；三个运行时代码文件通过现有提取器，显式选择范围为 complete、无截断。核对新工厂组件均处于客户端提前返回之后、注册目标解析到本地 fn、上限常量与数组／名称一致。AST 和这些结构检查不证明运行行为；外部游戏调用保留未解析边界，没有另造全量 Lua 模拟器。

两份模组均未启动 DST：生成、检查／拾取、堆叠拆分／转移／存档、实际砍树扣次、耗尽删除、多人图标／网络及兼容均未实测。后续游戏验收应覆盖这些预期，斧头另测旧存档恢复和有限组件缺失时跳过；未实际执行前不升级 Wiki 的 validation。

本轮没有改变知识页、检索工具、fork、游戏源码或用户模组；更新工程状态并保存报告，不把验收文档加入知识候选。已核验源文件及 Git 暂存区字节保持，日志只追加。

## 后续

保留当前 cookbook／reference 路由和输出预算，本次不引入数据库或 codegraph。先按首批覆盖路线补装备／配方／建筑的必要骨架，再扩展实际任务样本、部分缺口回退与暖石组合；独立模型路由、6～10 项任务、案例维护成本、Top 30 比较和游戏运行仍留 [PLAN](../../../PLAN.md#w05真实方案与检索闭环)。
