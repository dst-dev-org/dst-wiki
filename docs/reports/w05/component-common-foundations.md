# 通用基础组件：作祟、说话与物品环境

2026-10-21，依据 Steam buildid 25643519。按用户分工，本对话优先通用、高复用组件，其余由另一 AI 并行建设。通用优先级来自开发需求，不是使用频率统计；本次准备脚本确认同 build，Skip，未更换快照。

## 本轮交付

C01 作祟／说话、C02 物品环境、C03 偏好／添料，各由一个子代理核验并写两页，主代理审阅、合并和检查公共文件；全部文本及临时 Lua 使用 apply_patch。

| 页面 | 物理行数／UTF-8 字节 | 完整 modmain 接入 |
|---|---|---|
| [hauntable](../../../wiki/reference/components/hauntable.md) | 87／10,751 | 燧石作祟冷却改6秒，保留原发射回调 |
| [talker](../../../wiki/reference/components/talker.md) | 90／11,821 | 玩家初始化后尝试提示一句，保留门禁／原说话回调 |
| [inventoryitemmoisture](../../../wiki/reference/components/inventoryitemmoisture.md) | 90／11,045 | 燧石暂停自然吸湿／干燥，保留显式赋湿和堆叠 |
| [inventoryitemtemperature](../../../wiki/reference/components/inventoryitemtemperature.md) | 93／11,491 | 原吸湿器自然目标增加10度偏移，不承诺恒温或人物供热 |
| [foodaffinity](../../../wiki/reference/components/foodaffinity.md) | 80／9,352 | 威尔逊培根煎蛋偏好倍率改1.5，保留饮食门禁 |
| [fuelmaster](../../../wiki/reference/components/fuelmaster.md) | 87／8,949 | 薇洛原添料加成再乘2，明确也影响其他接收目标 |

六页合计 63,409 字节，均低于页面上限；保留五主节，接口协作采用三级小节。未新增类型、目录分类或模板。物品环境两页合并重复说明，从初稿约12／14KB降至各约11KB，保留必要代码、同步与清理证据；不把这项体积变化换算为 token 收益。

## 关键核验

- hauntable：动作资格与直接 DoHaunt 分开，失败也可置 haunted／冷却；部分 helper 实效分支已注释，名称不能证明当前可用。组合 helper 保留前段结果与执行顺序，冷却不自动撤销所有自定义效果。
- talker：Say 与 Chatter 的消息路径不同，获准的新 Say 会取消旧台词。服务器和客户端均需正确宿主；普通消息不传全部本地动画／颜色参数，ShutUp 不保证全网立即取消。Wonkey 乱码按真实标签条件说明，未凭训练知识补造分支。
- inventoryitemmoisture：必须经 inventoryitem 绑定 replica；物品值与角色湿度不同，湿润标志也独立于数值。外部控制冻结自然目标，不拦显式写入、读档或堆叠混合；waterproofer 可能移除／重建扩展。
- inventoryitemtemperature：默认可选且未启用，物品自然温度、目标偏移与绝热分别处理。冰箱／口袋容器／漂浮分支有实际差异；运行时启用不保证立即开始换热。该 build 客户端 replica getter 可返回编码字节，classified 事件才解码，不能把原码直接与服务端温度阈值比较。
- foodaffinity：匹配值取最大，不相乘；调味食品按实际映射还原基础 prefab。prefab 偏好有负生命／精神基础值的额外影响，但不保证所有负效果消失，也不放宽可食门禁。
- fuelmaster：挂在添料者，不是燃料或目标；只影响补充公式，不改变持续消耗。零倍率不拒绝动作／消费；回调实参与目标范围、原配置恢复由页面明确。

各组完整审阅所属组件，定向核验必要消费者、原宿主／注册、动作／SG、初始化／保存／来源与清理；已有同版本 eater、fueled、fuel、inventoryitem、temperature、heater 等证据按需复用。主代理复核关键真值／倍率、作祟组合、消息参数、容器／更新和编码区别，修正过宽表述并合并重复维护内容。

六页保持 source-reviewed。未运行游戏；未穷举所有宿主、容器、食品、fueler、联机、引擎网络路由、多语言、编码边界或跨模组覆盖。

## 检查结果

- 六页及四张修改导航的元数据、sources 存在性、Wiki／本地链接和大小通过；六个新页均唯一叶级登记。
- 合并 strict 源码引用：217 处引用、187 个唯一目标、37 份来源文件，0 error／0 warning，未截断。
- 六个完整 Lua 示例由所属代理逐字另存，通过已有 Full Moon Lua 5.1 syntax-only 后用 apply_patch 清理；未执行代码，主代理复核案例完整性。
- 六次“典型接入”读取均保留元数据、快速判断、所选完整节／Lua及验证记录；complete_selected_ranges=true、source_version=match、truncated=false、omitted=0。
- 合并检查时 203 个已登记组件的主源码 SHA-256 匹配当前快照，已登记页面均存在；其他 AI 在建草稿单列，不因文件出现自动记为 built。

| 页面 | 所选章节 JSON stdout UTF-8 字节 |
|---|---:|
| hauntable | 4,159 |
| talker | 4,941 |
| inventoryitemmoisture | 4,726 |
| inventoryitemtemperature | 4,683 |
| foodaffinity | 5,244 |
| fuelmaster | 5,244 |

**全库导航尚未收尾：** 该次 validate_page 加 --check-indexes 返回 1，原因是他方正在编写的 boatphysics、fishingrod、oceanfishingrod 尚无叶级登记；六页及四张导航本身 passed。检查后文件观察还出现他方 sheltered 草稿，显示并行建设范围仍在变化。本轮不覆盖或代其登记，不能把所选页通过称为全库导航通过。报告的全库页数是扫描时点值，不作为最终完成数。

上述字节只是读取规模代理，不是准确 token 测量；并行数量不证明总成本降低，源码／结构／语法通过不代表运行通过。

## 协作与后续

本轮 foodaffinity 登记人物，inventoryitemmoisture／inventoryitemtemperature／fuelmaster 登记物品，hauntable／talker 登记交互技能。合并时主题登记为人物20／物品39／战斗30／世界67／交互技能47，共203组件；另3篇modinfo与5篇cookbook，共211篇已登记知识页。12篇导航、22条Top30保持，其他在建草稿另计。

798 个组件候选中，合并时203已建、595未完成；按历史路线重算剩扩展367／专项225／职责待确认3。与上一轮195相比的增量包含本轮六页及另一AI已登记的两篇血条页，未将后者算成本轮新写页面或代其验收。

PLAN 已记录分工和本轮 C01～C03，保留另一 AI 的第92批 CSV／索引增量。另一 AI 已先开始的 sheltered／钓鱼／船体页由其继续收尾；本对话后续优先 rainimmunity／lightningblocker 等未认领通用组件，先查实际页面与占用。正文地址稳定，根入口仅补范围，不扩增 Top30 或生成空页。

同步覆盖 CSV、四张导航、PLAN、README、设计与 docs 入口；本轮日志只追加一次。历史报告和来源保持原样，不修改 Skill、工具箱、其他 AI 页面或输入源码。未安装／构建、运行游戏或独立模型评测、重建缓存、提交或推送；本轮临时 Lua 为零。

收尾复核：五份工程 Markdown 的153处本地链接／锚点无错误，git diff --check通过；日志276个物理行／41,409字节，未触发归档。C01～C03临时Lua为零。全库导航的他方在建缺口仍按上文保留。
