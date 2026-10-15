# 科技与建造接入组件检视

2026-10-17；Steam buildid 25643519。按 dst-wiki Skill 完成第十二批四页；源码准备返回 Skip，未解压。文本与临时 Lua 均通过 apply_patch 写入／清理。

## 交付与关键边界

| 页面 | 核验与案例 | 物理行／实际 UTF-8 字节 |
|---|---|---:|
| [prototyper](../../../wiki/reference/components/prototyper.md) | 科技树、多人回调、选站与激活；科学机器提供二级 SCIENCE | 71／7,312 |
| [craftingstation](../../../wiki/reference/components/craftingstation.md) | 成对条目、nounlock 消费、限额／保存；雕塑台默认战车图纸与读档补齐 | 82／7,835 |
| [placer](../../../wiki/reference/components/placer.md) | 本地工厂／controller／测试／吸附／清理；手柄虚影前方三单位 | 74／7,617 |
| [deployhelper](../../../wiki/reference/components/deployhelper.md) | 本地登记／过滤／续期／关闭；盛宴桌辅助仅白天显示 | 73／7,216 |

- builder 选择一个当前 prototyper，普通扫描与显式 override 的检查不同；CanUsePrototyper 不能当所有路径的统一门禁。TurnOn／Activate 本身也不完成资格校验或扣料。
- 科学机器工厂直接引用共享 TUNING 科技表，案例先深拷贝实例再改 SCIENCE，保留动画／激活；不注册新科技或抹除已学配方。
- craftingstation 不注册配方，item／recipe 两数组按位置对应，按 item 去重；限额更新与菜单同步依赖实际消费者。雕塑台案例保留宿主 OnLoad 并在组件恢复后补齐；停用不自动撤销已保存条目，拆台可能掉落授予图纸。
- 钓具图纸工厂已使用 teacher，旧 tacklesketch 组件不能单独证明当前图纸接入；本批采用仍在使用的雕塑台路径。未发布新的钓具图纸页面或额外注册教程。
- placer 是本地独立预览，controller 会再赋 testfn，网格也会重复测试；实际制作仍由服务器重验地图／缓冲／配方。LinkEntity 不包办父级与删除。
- deployhelper 的两种过滤是 OR，delay=2 按 wall-update 次数递减，不是两秒。重复请求只续期，onenablehelper 不保证每次获得最新 placer；辅助效果不代替部署验证。

## 检查与按需读取

六个 Wiki 文件（四页、世界主题索引与组件分类入口）通过格式、来源文件、标签、大小、本地链接／标题锚点检查。全库导航登记检查为 84 篇知识页、12 篇导航、22 条 Top 30，无遗漏、重复或父入口问题；不等于全库技术语义检查。

四页 113 处源码行引用、82 个唯一目标、15 份来源文件 strict 检查为 0 error／0 warning，输出未截断。四项覆盖记录保留并复核源码 SHA-256；798 个组件候选现为 76 已建／722 未建，19 replica／3 helper／1 empty 仍单列。

四段完整 modmain 示例与页面代码逐项一致，通过已有 Full Moon 3.0.0／Lua 5.1 syntax-only 探针；byte/position 明确跳过，不声称类型、字节往返或游戏验证。四份临时 Lua 已删除，未新增依赖或构建工具。

作者受控按“典型接入”读取，均包含快速判断、完整示例／前提和验证记录，无截断／省略：

| 页面／需求 | JSON stdout 实际 UTF-8 字节 |
|---|---:|
| craftingstation／雕塑台默认图纸 | 4,725 |
| deployhelper／仅白天辅助 | 4,138 |
| placer／手柄前方三单位 | 4,218 |
| prototyper／科学机器二级科技 | 4,252 |

名称已知直接按页／章节定位；未知名称的科技与放置需求可从组件分类入口进入世界主题的“科技与建造接入”四行，再读所需页面。世界主题当前 55 行／3,713 字节；保持正文地址，不新增必读导航层。以上为受控读取，不是独立模型发现评测，字节不换算 token 收益。

同步 PLAN、README、总体设计、文档入口、覆盖清单及追加日志；保留历史报告和已有暂存／工作区变更。未改游戏源码／输入模组、Skill、工具箱，未重建缓存、提交或推送。

## 后续与限制

第十二批完成，下一批为 cookable／cooker／dryable／dryingrack／stewer。配方注册 API 仍待按实际需求补齐，再推进配方／建筑 cookbook；不把四组件页称为完整建造骨架。heatrock Prefab 与其他既有队列保留。

全部页面保持 source-reviewed，未运行游戏。多人站点切换、菜单刷新／网络编码、存档／烧毁与拆台掉落、手柄／鼠标／网格／平台、昼夜转换和跨模组回调仍需运行验证。
