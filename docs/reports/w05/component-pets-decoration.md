# 宠物、行为记录与巢穴装饰组件检视

2026-10-19；Steam buildid 25643519。按 dst-wiki Skill 连续完成第 39～43 批，共六篇组件页。复用已确认快照和已有 Wiki，游戏源码／模组输入只读；文本和临时 Lua 使用 apply_patch 写入及清理。

## 批次与测时

工具实际 UTC 时钟转为 Asia/Shanghai。每批从相关实现审阅开始，到新页／主题登记／覆盖清单、适用检查和临时 Lua 清理完成；相关宿主可提前定位、跨批复用，不是六个独立冷启动样本。统一工程文档、总报告和日志收尾另计。

| 批次 | 页面 | 开始—结束（北京时间） | 批内墙钟 | 源码引用数 |
|---|---|---|---|---:|
| 39 | petleash | 23:22:16—23:24:42 | 2 分 26 秒 | 28 |
| 40 | pethealthbar | 23:24:42—23:26:49 | 2 分 07 秒 | 26 |
| 41 | beefalometrics | 23:26:49—23:28:35 | 1 分 46 秒 | 22 |
| 42 | crittertraits | 23:28:35—23:30:50 | 2 分 15 秒 | 26 |
| 43 | bedazzler、bedazzlement | 23:30:50—23:33:48 | 2 分 58 秒 | 42 |

五批核心审阅／生成／检查共 11 分 32 秒。该样本含共享上下文和已建机制支撑，不能据此外推复杂世界／网络组件全部工时；墙钟、页面数和输出字节不证明 token 收益或游戏行为正确性。

统一工程文档、总报告／日志及最终检查于 23:33:48—23:36:34 完成，另计 2 分 46 秒；截至该检查完成总墙钟 14 分 18 秒。其后的测时补记和最终回复不计入此观察截止值，不称整个会话的精确终止时间。

## 交付与边界

| 页面 | 实际行数／UTF-8 字节 | 完整接入案例 |
|---|---|---|
| [petleash](../../../wiki/reference/components/petleash.md) | 69／7,352 | 威尔逊普通宠物容量改为二，保留专用限额和原回调 |
| [pethealthbar](../../../wiki/reference/components/pethealthbar.md) | 70／7,835 | 原温蒂服务端事件刷新当前宠物显示上限，不改实际生命 |
| [beefalometrics](../../../wiki/reference/components/beefalometrics.md) | 73／7,124 | 为原牛规范最近驯养参与者的 userid 只读查询 |
| [crittertraits](../../../wiki/reference/components/crittertraits.md) | 75／8,039 | 原领养小猫正向 PLAYFUL 倍率翻倍，保留动画、偏置和上限 |
| [bedazzler](../../../wiki/reference/components/bedazzler.md) | 63／5,311 | 原巢装饰工具直接耗用减半，不重设耐久 |
| [bedazzlement](../../../wiki/reference/components/bedazzlement.md) | 72／7,370 | 三种原巢服务端解除装饰事件，保留原成长／Buff 退出协议 |

六页共 43,031 字节，全部在页面上限内，均采用五节结构，无新增类型或空导航。

- petleash 的普通计数与按 Prefab 限额分开，后者包含预留；GetNumPets 不是全体数量，GetPetsWithPrefab 也不能当所有普通宠物的通用筛选。AttachPet 不走 onspawnfn，DetachPet 不撤销跟随／恢复 persists，读档不重新检查容量。
- pethealthbar 装在玩家且双端创建 netvar，HUD 另按宿主条件显示；SetPet(nil) 不是解绑，相同目标重绑也不完全初始化。显示上限、实际生命与附加条分开；箭头是有限状态提示，不是净回血率。
- beefalometrics 只是行为统计和最近参与者记录，getter 可返回实体／userid／nil；不用作所有权／权限依据。刷毛处理器更新记录用 doer，但统计参数实际读 feeder，按实现记录差异。Stats 入口构建上下文并编码发送，不将调用当平台接受或 gameplay 成功。
- crittertraits 的抚摸先进入 SG，animover 且 AnimDone 才发计分事件；不是 OnPet 本身直接计分。分数、周期选主导、GOODIES 锁定和标签 setter 各有边界；StartTracking 的主人监听不自动重绑，不用重复调用冒充安全转交。
- bedazzler 的状态测试不验证全部目标／角色前提，直接 Bedazzle 不再测试且直接扣 finiteuses；正常候选与动作各自负责门禁。不要靠额外 SetConsumption 企图覆盖原直接 Use。
- bedazzlement 强依赖原巢资源和宿主字段，暂停成长、周期续 Buff；Stop 不逐一清除已有 Buff。Start 的零延迟网半径任务未被 Stop 保存／撤销，同帧开关不承诺一致；单独 PacifySpiders 也不检查装饰状态。

未全面审校领养 UI／角色迁移、ghostlybond／HUD 全生命周期、平台统计、所有小宠物 Brain／SG、转交后监听、蜘蛛完整战斗和平台成长政策。未修改 Skill、工具箱、游戏源码、正式类型数据或历史报告。

## 校验与受控读取

逐批页面格式／来源文件／本地链接／锚点／大小与全库登记检查通过；六例和临时 Lua 逐字比对，经已有 Full Moon Lua 5.1 syntax-only，通过后清理。源码 SHA-256 与覆盖 CSV 对应；统一收尾核对全部 140 个已建记录的来源指纹及页面集合一致。

六页合并 strict 源码检查：144 处引用、114 个唯一目标、24 份来源文件，0 error／0 warning；truncated=false、omitted=0、by_page_omitted=0。首轮 HUD 引用终点空行已按实际条件分支修正；语义复核修正 Stats 入口描述和抚摸动画完成时机，不将结构工具通过视为这些技术结论的证明。

| 页面 | “典型接入”JSON stdout 实际 UTF-8 字节 |
|---|---:|
| petleash | 4,340 |
| pethealthbar | 4,841 |
| beefalometrics | 5,047 |
| crittertraits | 5,098 |
| bedazzler | 4,078 |
| bedazzlement | 5,252 |

六次按已知页读取所需章节，均保留元数据、快速判断、完整示例／条件与验证记录；complete_selected_ranges=true、source_version=match、truncated=false、omitted=0。字节按 LF stdout 记录，不换算模型 token，不称独立模型自主评测；未重建缓存。

## 收尾与后续

新页在世界主题唯一登记五条、物品主题一条；当前组件 140 篇，人物 16／物品 30／战斗 13／世界 66／交互技能 15。另有 3 篇 modinfo、5 篇 cookbook，合计 148 篇知识页、12 篇导航、22 条 Top 30；实际主题表仍是分类维护来源。

798 组件候选中 140 已建／658 未建，另 19 replica／3 helper／1 empty 单列。本轮三项来自扩展池、crittertraits／bedazzlement／bedazzler 三项来自原专项池，现剩扩展 423／专项 232／职责待确认 3，历史路线快照不改写。

世界主题 143 行／13,057 字节，物品主题 88 行／5,758 字节，组件入口 25 行／1,708 字节；均在索引上限内，未移动正文或新增导航层。

同步 PLAN、README、设计、docs 入口、组件入口、覆盖 CSV 与追加日志。下一批拟核验 activatable／useabletargeteditem，实际职责与案例执行前再确认；heatrock Prefab、配方注册 API 和 cookbook 原队列保留。

最终九份 Wiki 文件的格式／本地链接／来源／大小及全库导航登记通过；五份工程 Markdown 的 132 处本地链接／锚点通过，git diff --check 通过。日志为 206 行／30,748 字节，未触发归档。

六份临时 Lua 已清理，未安装依赖、构建解析器、运行游戏或独立模型评测，未提交／推送。新页均保持 source-reviewed；格式、来源位置、语法与受控读取通过不升级为 runtime-tested。
