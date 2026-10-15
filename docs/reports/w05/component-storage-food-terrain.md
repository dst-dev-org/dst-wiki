# 通用组件：物品存储、食物记忆与地皮工具

2026-10-22，Steam buildid 25643519。承接通用 C04～C06，本轮按用户要求继续剩余主题，三个子代理分别完成 C07～C09，主代理审阅并统一登记。准备脚本同 build 返回 Skip，未更换快照。

## 本轮交付

开始时 252 个组件页与覆盖清单一致，已登记主源码 SHA-256 全部匹配；六个所选主题均未建，未覆盖其他 AI 的页面或认领。

| 组 | 页面 | 物理行数／UTF-8 字节 | 接入场景 |
|---|---|---|---|
| C07 | [inventoryitemholder](../../../wiki/reference/components/inventoryitemholder.md) | 85／9,533 | gelblob_storage 只收新交入肉类，保留旧档恢复与原宿主回调 |
| C07 | [itemstore](../../../wiki/reference/components/itemstore.md) | 81／9,504 | 原漂浮物出口每次打捞最多恢复一条记录，沿用分发与动画 |
| C08 | [foodmemory](../../../wiki/reference/components/foodmemory.md) | 75／8,526 | 沃利重复食物记忆窗口改为一个游戏日，不重排既有任务 |
| C08 | [fueler](../../../wiki/reference/components/fueler.md) | 79／9,545 | 原鸦年华小乌鸦添火基础量翻倍，保留 Brain／SG 与时机 |
| C09 | [terraformer](../../../wiki/reference/components/terraformer.md) | 80／9,448 | 普通草叉铲地后固定露出 DIRT，不恢复农田原下层 |
| C09 | [terraformerremoveable](../../../wiki/reference/components/terraformerremoveable.md) | 67／9,030 | 高尔夫球场原版接入说明：比赛禁拆、恢复、返还与附属清理 |

六页合计 55,586 字节，均在页面上限内，保持五个主节；最后一页采用原版接入说明，没有为专用目标强造完整 Lua demo。未改模板、Skill 或工具箱。

## 关键核验与复核

- inventoryitemholder 持有真实物品或同类堆叠，交入后回调可能收到 nil，取回合并后物品可能失效。食品暂停腐败、展示和销毁归还需宿主补足。示例保留以宿主为 giver 的恢复调用，避免缩窄新过滤时拒收旧档内容。
- itemstore 存记录而非活体，AddItem 删除原实体；GetFirstItems 重建后出队，但无完整引用恢复或异常回滚。迁移重试属于事务层；示例只限原出口的批量，不改记录和交付链。
- foodmemory 在本次进食前被读取、末尾才记忆；再次进食重排该键任务，到期整项遗忘。调味食品合并基础键；改 duration 不重排已有任务，旧存档按保存剩余秒数恢复。
- fueler 有小乌鸦真实宿主和 Brain／SG 消费链。TakeFuelItem(nil, doer) 用执行者供料，不删除执行者；非 nil item 仍会删除，且 fuel 优先于 fueler。它与 fuelmaster 的倍率职责分开。
- terraformer 先改地图再完成回调；世界实体与工具／执行者的同名事件载荷不同。耐久归成功动作链，地皮部署与目标移除是其他入口。主代理明确示例作用所有可铲地块，未将其误写为仅修改农田。
- terraformerremoveable 的回调返回值被忽略，无回调也返回成功；删除、返还与清理由宿主承担。保留原拼写，核验球场动态移除／重加组件的禁拆流程，不以回调返回 false 代替门禁。

各组核验实现、必要消费者与真实宿主／注册、保存清理；已覆盖同版本 inventoryitem、container、finiteuses、eater、fuelmaster 等知识按需复用。主代理通读六页并复核上述关键边界；未将来源存在或解析通过当作游戏行为验证。

## 检查结果

- 六页及四张改动导航的元数据、来源存在性、链接、大小通过；全库 266 篇知识页／13 篇索引唯一登记与父入口通过，Top 30 仍为 22 条。
- 六页 196 处源码行号引用，161 个唯一目标、36 份来源文件；strict 检查 0 error／0 warning，无截断。
- 三个子代理合计检查五个最终完整 modmain 示例，现有 Full Moon Lua 5.1 syntax-only 全部通过；各自临时 Lua 已用 apply_patch 删除，未执行示例。
- 六次“典型接入”读取均匹配原节全文，source_version=match、complete_selected_ranges=true、truncated=false、omitted=0；保留快速判断、完整接入及验证记录。
- 258 个 built 记录与实际组件页集合一致，258 个主源码 SHA-256 匹配当前快照；覆盖 CSV 保留原来源指纹和旧记录路径风格。

| 页面 | 章节读取 JSON stdout UTF-8 字节 |
|---|---:|
| inventoryitemholder | 6,077 |
| itemstore | 6,217 |
| foodmemory | 4,730 |
| fueler | 6,069 |
| terraformer | 5,761 |
| terraformerremoveable | 6,105 |

字节是读取规模代理，不是模型 token。所有页面保留 source-reviewed；未运行 DST，未验证多人、恢复异常、旧档迁移、技能或跨模组组合效果。

## 登记与后续

inventoryitemholder／itemstore／fueler 登记 items，foodmemory 登记 character，terraformer／terraformerremoveable 登记 world；组件分类入口补食物记忆范围，正文地址不迁移。同步 [覆盖清单](component-coverage.csv)、[PLAN](../../../PLAN.md)、README、设计与工程报告入口，日志只追加一次。

累计 258 个组件已建、540 个候选未建；另有 3 篇 modinfo、5 篇 cookbook，共 266 篇知识页和 13 篇导航。按历史路线重算剩余扩展 312、专项 225、职责待确认 3，不把名称归组当作职责或使用频率核验。

world 主题现为 156 行／14,801 字节；后续新登记先量取余量，按真实内容增长再拆。下一轮从 occupiable／occupier、firedetector／fan 等候选重新核对覆盖与认领，不预建空页。未修改游戏源码、其他 AI 页面、历史报告或日志条目；未安装、构建、重建缓存、运行独立模型评测、提交或推送。

收尾核对：五份工程 Markdown 的 167 处本地引用／锚点有效，git diff --check 通过。覆盖 CSV 无重复 source 或多余列，C07／C08／C09 各登记两页；日志 22 行／3,066 字节，无需归档。
