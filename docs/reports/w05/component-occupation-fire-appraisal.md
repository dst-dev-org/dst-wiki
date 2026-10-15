# 通用组件：鸟笼占用、火情检测与物品交互

2026-10-22，Steam buildid 25643519。承接通用 C07～C09，本轮三个子代理分别完成 C10～C12，主代理审阅并统一登记。准备脚本返回同 build Skip，未更换快照；开始时 258 个已建主源码指纹匹配，所选六个主题均未建。

## 本轮交付

| 组 | 页面 | 物理行数／UTF-8 字节 | 接入场景 |
|---|---|---|---|
| C10 | [occupiable](../../../wiki/reference/components/occupiable.md) | 80／8,481 | 鸟笼只允许新装金丝雀，覆盖组件重建并保留旧档其他鸟 |
| C10 | [occupier](../../../wiki/reference/components/occupier.md) | 80／7,970 | 金丝雀每次入笼含读档至少半寿命，保留原洞穴毒气回调 |
| C11 | [firedetector](../../../wiki/reference/components/firedetector.md) | 82／9,256 | 原灭火器仅主动选火焰／阴燃，保留布防和实际范围效果 |
| C11 | [fan](../../../wiki/reference/components/fan.md) | 86／8,792 | 羽毛扇保留降温灭火，额外降低动作目标 5 点潮湿 |
| C12 | [appraisable](../../../wiki/reference/components/appraisable.md) | 57／6,985 | 牛玩偶向评委亭请求鉴评的原版接入说明 |
| C12 | [murderable](../../../wiki/reference/components/murderable.md) | 61／7,684 | 池鱼／池鳗的动作入口、事件、掉落和移除说明 |

六页共 49,168 字节，均在页面上限内；四个完整 modmain 示例，另两页采用原版接入说明。短标记组件不强造独立工程或测试打印案例。未修改 Skill、模板、工具箱或其他 AI 的页面。

## 核验与复核

- occupiable 的直接 Occupy 绕过 CanOccupy；存档恢复也直接占用并重放回调。门禁例使用 ComponentPostInit，兼容鸟笼取走死亡产物后的组件重建；正常 STORE 拒绝不代表客户端预先隐藏提示。
- occupier 仅保存 owner；Harvest 不调用 onoccupied(nil)，移组件不能替代释放。金丝雀例明确操作 perishable 比例而非 hunger，且旧档每次低于一半时可再次补足，原毒气回调保留。
- firedetector 负责选靶位置，实际保护分别经醒着的 SG／雪球和睡眠宿主路径。布防态 Deactivate 保留任务，切模式保留两秒抑制，退出紧急回调收到清零后等级。主代理收窄保护判定表述：保护仅排除枯萎／复壮评分分支，燃烧或阴燃仍可被选中。
- fan 仅分发使用／吟唱回调；普通效果、吟唱龙卷风与动作耗次分开。减潮湿例只改动作目标，未将其写成范围干燥或库存物品除湿。
- appraisable 的检查和执行独立，执行回调返回值被忽略；APPRAISE 只显式转发 NOTNOW。核验九种玩偶、评委亭状态图与类别数据消费，未把标签当作完整目标契约。
- murderable 不提供 Kill／Murder 方法，动作不走普通 health:Kill。主代理修正快速判断中的顺序：先发施作者事件，再 GenerateLoot／GiveItem，最后 Remove；消耗数量取移出后实体的堆数，保留容器超堆、health 门禁和焚化事件差异。

各组核验实现、必要动作与消费者、宿主注册／构造和恢复清理；主代理通读六页并复核关键顺序和条件。既有同 build 组件知识按需复用，没有全量重审鸟种、评选比赛或紧急调级接口。

## 检查结果

- 六页及四张修改导航的元数据、来源存在性、链接与大小通过；全库 272 篇知识页／13 篇索引唯一登记和父入口通过，Top 30 保持 22 条。
- 六页 191 处源码行号引用，151 个唯一目标、36 份来源文件；strict 检查 0 error／0 warning，未截断。
- 四个最终完整 Lua 示例经各代理使用现有 Full Moon Lua 5.1 syntax-only 检查通过；临时 Lua 均用 apply_patch 删除。另两页没有新增 Lua，不纳入语法样本。
- 六次“典型接入”读取与原章节逐字一致，source_version=match、complete_selected_ranges=true、truncated=false、omitted=0；保留前提、快速判断、完整示例或原版接入说明及验证记录。
- 覆盖清单 264 个 built 记录与实际组件页集合一致，264 个主源码 SHA-256 全部匹配当前快照；原指纹和历史记录路径风格保留。

| 页面 | 章节读取 JSON stdout UTF-8 字节 |
|---|---:|
| occupiable | 4,986 |
| occupier | 5,411 |
| firedetector | 4,085 |
| fan | 5,007 |
| appraisable | 5,541 |
| murderable | 4,564 |

字节只表示读取规模，不是 token 测量。所有页面保持 source-reviewed；未运行 DST，未验证多人、旧档、动态重建、应急表现和跨模组回调组合。

## 登记与后续

occupiable／occupier／firedetector 登记 world，fan／murderable 登记 items，appraisable 登记 abilities；只增加有实际页面的浏览分组，保留原章节锚点与正文地址。同步 [覆盖清单](component-coverage.csv)、[PLAN](../../../PLAN.md)、README、设计与工程报告入口，日志只追加一次。

累计 264 组件已建、534 候选未建；另有 3 篇 modinfo、5 篇 cookbook，共 272 篇知识页与 13 篇导航。按历史路线重算剩余扩展 306、专项 225、职责待确认 3，归组不是使用频率或职责核验。

world 主题现为 159 行／15,285 字节，仍在上限内；再次向该主题登记前先按真实内容拆分，并检查旧锚点、唯一登记及父入口。下一轮候选包括精神／温度扩展、法杖精神／恐吓、挖墓两侧，须重新核对覆盖和认领。未安装、构建、重建缓存、运行独立模型评测、提交或推送；游戏源码和历史报告未改。

收尾核对：五份工程 Markdown 的 170 处本地引用／锚点有效，git diff --check 通过。覆盖 CSV 无重复 source 或多余列，C10／C11／C12 各登记两页；日志 28 行／4,048 字节，无需归档。
