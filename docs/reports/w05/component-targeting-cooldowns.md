# 定向技能与冷却组件检视

2026-10-17；Steam buildid 25643519。按 dst-wiki Skill 完成第十一批六页，先实施已排定的组件主题导航拆分。源码准备返回 Skip，未解压；所有文本改动和临时 Lua 清理均使用 apply_patch。

## 页面与核验范围

| 页面 | 已核验重点 | 物理行／实际 UTF-8 字节 |
|---|---|---:|
| [aoecharging](../../../wiki/reference/components/aoecharging.md) | 双端网络计数、松手事件、SG 攻击时点、骑乘与清理 | 71／8,029 |
| [aoespell](../../../wiki/reference/components/aoespell.md) | CanCast／CastSpell 分离、返回语义、事件、地图与距离门禁 | 75／6,510 |
| [aoetargeting](../../../wiki/reference/components/aoetargeting.md) | 双端构造、enabled、配置浅复制、骑乘／净空与 range 边界 | 68／7,431 |
| [cooldown](../../../wiki/reference/components/cooldown.md) | 初始无任务、首充／默认时长、重排、完成与保存恢复 | 67／6,831 |
| [reticule](../../../wiki/reference/components/reticule.md) | 本地鼠标／手柄、颜色／位置函数与监听清理 | 68／7,574 |
| [spellbookcooldowns](../../../wiki/reference/components/spellbookcooldowns.md) | 人物技能键冷却、classified 同步、剩余比例、主动消费与不存档 | 72／7,934 |

核验完整六组件及必要 Map、动作、controller、弹弓／骨甲／冷却实体工厂、player_common、幽灵命令和人物 SG 消费点，不以字段名称推定行为。aoetargeting／aoecharging 需要共同端构造；aoespell 回调和冷却启动属于服务端操作；本地合法颜色不代替服务端授权。蓄力组件只发松手事件，SG 决定攻击时点；pingprefab 当前仅作开关，生成名实际硬编码。人物多技能冷却不自动阻止效果或保存至存档。

六个案例分别复用原版物品／人物宿主，写明 modmain 位置、双端／服务端、生效与停用边界、资源和未覆盖内容；不把冷却条目创建称为完整新技能。页面保持 source-reviewed，未运行 DST。

## 导航拆分

- 保留所有 `reference/components/<名称>.md` 正文地址；组件分类入口由 156 行／12,271 字节降为 25 行／1,252 字节，只列五个主题。
- 人物、物品、战斗、世界、交互技能主题索引分别为 42／50／36／46／43 行；当前各为 2,896／3,316／2,446／2,963／2,958 字节。原 66 条登记逐行原样迁入，新增六条后共 72 条唯一登记。
- 已搜索旧组件索引的章节入链，未发现需迁移项。主题入口与叶级登记继续用表格，返回父级用正文链接，既有查询／导航校验支持，不修改 Skill 或工具箱。
- 名称已知的 reticule 直接按“典型接入”读取，跳过全部导航；未知名称的“按住蓄力并松手攻击”受控按分类入口→交互技能主题→aoecharging 章节定位。该路径只读两张导航，共 4,210 文件字节，再读目标章节；不展开其余四主题或全部组件正文。

这是作者受控路径检查，不是独立模型自主发现评测；导航读取量仅记录文件规模，不换算 token 或宣称完整任务收益。未重建查询缓存，缓存失效时仍按既有规则回退当前 Markdown。

## 检查与读取

六页按“典型接入”读取，均返回快速判断、完整代码／接入条件与验证记录，无截断／省略；调用显式注明核验 build。

| 页面 | JSON stdout 实际 UTF-8 字节 |
|---|---:|
| aoecharging | 4,375 |
| aoespell | 4,540 |
| aoetargeting | 4,358 |
| cooldown | 4,426 |
| reticule | 4,277 |
| spellbookcooldowns | 5,069 |

- 六页与六张受影响导航通过格式、来源文件存在性、标签、大小、本地链接／锚点检查；全库登记检查为 80 篇知识页、12 篇导航、22 条 Top 30，无遗漏／重复／父入口问题。登记检查不是全库页面语义审校。
- 六页 177 处源码行引用、107 个唯一目标、16 份来源文件 strict 检查为 0 error／0 warning，报告未截断。初查发现五处空／结构行引用，按实际变量读取、骑乘判断与硬编码生成分支修正后通过；脚本不判断结论是否得到源码支持。
- 六段完整示例与页面逐项一致，已有 Full Moon 3.0.0／Lua 5.1 探针 syntax-only 通过，byte/position 明确跳过；不宣称类型、字节往返或游戏行为验证。六份临时 Lua 创建于 output，检查后删除。
- 覆盖 CSV 只更新本批六行，保留并复核源码 SHA-256；798 个候选现为 72 已建／726 未建，另 19 replica／3 helper／1 empty 单列。同步计划、README、总体设计、报告入口及追加日志。
- 未运行游戏、独立模型评测或全库解析；未写游戏／输入模组、Skill、工具箱，未提交／推送，保留原有暂存与工作区变更。

第十一批和导航拆分完成；下一批为 craftingstation／deployhelper／placer／prototyper 的科技与建造接入。heatrock Prefab、配方注册 API 与 cookbook 队列保留，真实任务依赖可提前插入。多人延迟、预测／骑乘切换、手柄、byte 边界、读档与跨模组包装仍需游戏验证。
