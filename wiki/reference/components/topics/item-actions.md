---
title: 物品交互与施法组件索引
created: 2026-10-23
updated: 2026-10-29
type: index
tags: [overview]
---

# 物品交互与施法

用于交易部署、使用切档、施法、阅读演奏及传送工具；原“物品交互与技能”锚点保留交易／部署入口，其余按下列分组定位。只读命中分组；适用版本与验证范围见具体页，已知名称可直读正文。

## 物品交互与技能

本节保留交易／部署；其他原入口分别见 [[reference/components/topics/item-actions#使用切档与重物交互|使用与切档]]、[[reference/components/topics/item-actions#施法与持续交互|施法]]、[[reference/components/topics/item-actions#阅读题字与演奏|阅读演奏]]、[[reference/components/topics/item-actions#传送与魔术工具|传送工具]]。

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/trader]] | 接收材料、测试／回调顺序与投料修复 | 给物品投喂材料、材料修复装备 / active |
| [[reference/components/tradable]] | 材料端猪王换金／蚁狮贡品数据 | 可交易物品、猪王换金 / active |
| [[reference/components/shop]] | 商店字段与按Prefab列表生成落物，不提供价格、支付或购买流程 | 商店交货生成、DeliverItems / active；接入边界说明 |
| [[reference/components/thief]] | 从背包／容器偷落物品、回调与返回值及拆堆边界 | 青蛙打落物品、nosteal、防盗 / active |
| [[reference/components/poppable]] | 单次戳破回调锁与气球宿主效果，区分伤害／删除和存档 | 气球戳破、Pop、popped / active |
| [[reference/components/repellent]] | 随从解除与附近战斗目标清除两分支，标签／例外和次数消耗 | 蜘蛛驱散、只解散随从 / active |
| [[reference/components/deployable]] | 放置模式、双端位置验证与部署物消费 | 部署物品、放置墙体 / active |

## 使用、切档与重物交互

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/useableitem]] | 已装备物品开始／停止使用与 inuse 清理 | 物品主动使用、使用状态 / active |
| [[reference/components/useableequippeditem]] | 装备使用／停止动作、使用者回调、启动拒绝与动态技能挂载 | 服务端停止WX78无人机遥控、已装备物品持续使用 / active |
| [[reference/components/reloaditem]] | 空装填标记接入CHANGE_TACKLE，物品转入手持容器由动作处理 | 犬牙装填、手持容器、reloaditem_ammo / active |
| [[reference/components/toggleableitem]] | 双态翻转／回调、无存读和动作 nil 返回的局部适配 | 剃刀模组标记开关、物品双态翻转 / active |
| [[reference/components/cyclable]] | 多档循环／回调、动作开关与直接调用边界、档位存读及宿主刷新 | 循环切换模式、前后切档、Cycle / active |
| [[reference/components/activatable]] | 实体资格／回调、inactive 与动作 nil 返回、重复使用和候选分工 | 栅栏门拒绝非玩家激活、激活回调返回nil / active |
| [[reference/components/heavyobstacleusetarget]] | 目标接收当前 BODY 重物的动作回调，资格过滤与物品转移由宿主负责 | 搬运重物放入目标、健身房装载 / active |
| [[reference/components/pushable]] | 被推物前向运动与间距补偿、推者 SG 协作，雪人与雪球目标替换 | 推动物体、滚雪球、START_PUSHING / active |
| [[reference/components/useabletargeteditem]] | 对实体使用／停止、目标筛选与执行校验、状态和宿主清理 | 命名牌服务端校验目标、对实体使用物品 / active |
| [[reference/components/sweeper]] | 工具端空标记，右键候选与动作按钮识别扫帚，耐久协议走 finiteuses | 自制扫帚、清扫工具标记 / active |
| [[reference/components/sweepable]] | 目标端标签门禁与清扫回调结算，构造挂标签、SetEnabled 摘挂，enabled 不存档 | 可清扫目标、清扫回调 / active |
| [[reference/components/itemmimicconfig]] | 拟态物品动作级强制揭示登记与最近动作记录，配合 ShouldItemMimicBeRevealedFor 判定链 | 拟态强制揭示、宝箱怪现形 / active |


## 节日物品与装饰

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/vasedecoration]] | 物品对 vase 目标生成 DECORATEVASE，装饰由 vase 组件接收 | 花瓶装饰材料 / active |
| [[reference/components/moonrelic]] | 供品标记：对 moontrader 生成 GIVE，CanAcceptFn 按 moonportalkey 标签决定 | 月坛供品、月亮传送门供品 / active |
| [[reference/components/carnivalgameitem]] | 喂鸡游戏食物标记：目标硬编码 feedchicks_nest 生成 CARNIVALGAME_FEED | 嘉年华游戏食物 / active |
| [[reference/components/pumpkincarver]] | 雕刻刀标记：对 pumpkincarvable 生成 CARVEPUMPKIN，图形按 prefab 名映射 | 南瓜雕刻刀 / active |
| [[reference/components/snowmandecor]] | 雪人装饰材料标记：对 heavy+snowmandecoratable 雪人生成 DECORATESNOWMAN | 雪人装饰材料 / active |
| [[reference/components/halloweenpotionmoon]] | 月之药水使用执行，变异走 halloweenmoonmutable 且无条件消耗 | 月之药水 / active |
| [[reference/components/pumpkincarvable]] | 巨型蔬菜雕刻会话：校验压缩存储 net 同步重建 FX | 南瓜雕刻 / active |
| [[reference/components/vase]] | 花瓶接收端：Decorate 插花、蔫化/灯光任务、花样式回调 | 花瓶、插花容器 / active |

## 药剂、沐浴与灵药

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/bathbomb]] | 浴弹标记：双标签候选、执行成功即 OnBathBombed 且浴弹销毁 | 温泉浴弹 / active |
| [[reference/components/ghostlyelixir]] | 幽灵灵药 Apply：elixir_drinker 转持有者＋GetApplyToTarget 解析＋回调消耗 | 幽灵灵药、阿比盖尔药水 / active |
| [[reference/components/bathbombable]] | 浴弹命中目标的状态与回调，can_be_bathbombed 同步标签 | 可浴弹目标 / active |

## 捕捉、交易与物品交互

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/talkable]] | TALKTO 成功前提数据组件，候选靠 maxwellnottalking 标签；原版无挂载者 | TALKTO 对话目标 / active |
| [[reference/components/moonstormstaticcatcher]] | 捕捉物品端 DIVEGRAB 抓取判定与瞄准通知 | 静电捕捉器 / active |
| [[reference/components/gestaltcage]] | 织影牢笼瞄准与捕获，成功换生成填充牢笼 | 织影牢笼 / active |
| [[reference/components/tributable]] | 贡品积累器（孤儿组件）：累加/衰减/发奖计数 | 献贡组件 / active |
| [[reference/components/nabbag]] | 抓取袋：PICKUP/NET 成功复制到前方锥形范围，限数量耐久 | 抓取袋 / active |
| [[reference/components/trophyscale]] | 奖杯称重比较台 | 奖杯秤 / active |
| [[reference/components/itemmimic]] | 拟态物品现形：事件映射到 TurnEvil 替换与惊吓，自动倒计时与暂停源 | 物品拟态 / active |

## 地图、传送与动作选择

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/mapspotrevealer]] | READ 动作揭示地图点，classified 推图钉＋RevealArea | 地图揭示 / active |
| [[reference/components/vaultorbteleporter]] | 宝库球两段式地图传送：缓冲选点动作→Activate 校验目的地组件后回调 | 宝库球传送器 / active |
| [[reference/components/playeractionpicker]] | 点击输入→优先级排序 BufferedAction 候选与过滤栈 | 动作拾取 / active |

## 武器改装与投掷

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/slingshotmodder]] | 弹弓改装工具入口，校验归属后转发 slingshotmods | 弹弓改装工具 / active |
| [[reference/components/oceanthrowable]] | 掷入海洋标记，补挂 complexprojectile 并回调配置 | 掷入海洋、OCEAN_TOSS / active |

## 施法与持续交互

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/spellcaster]] | 自身／目标／地点施法与最终冷却校验 | 右键施法、对目标施法 / active |
| [[reference/components/spell]] | 持续效果载体的目标／周期回调、启动与恢复及存档边界，区别施法入口 | 效果实体计时、StartSpell、ResumeSpell / active |
| [[reference/components/channelable]] | 建筑准入／参与者、SG 与回调停止、多人模式及宿主清理 | 服务端中止沙漠传送阵调谐、建筑持续交互 / active |
| [[reference/components/channelcastable]] | 物品持续施法握手／回调、动态技能挂载与卸装清理 | 服务端停止打火机持续吸火、持续施法物品 / active |
| [[reference/components/channelcaster]] | 人物持续状态／事件、物品和无物品模式、移动及派生效果撤销 | 持续施法时每分钟增加三点精神贡献、人物持续施法状态 / active |
| [[reference/components/spellbook]] | 轮盘／条目选择、客户端请求与服务端效果 | 技能轮盘、物品技能菜单 / active |

## 阅读、题字与演奏

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/book]] | 技能书正式／翻阅回调、成功消费与精神／FX 结算 | 睡前故事阅读精神消耗减半、技能书阅读效果 / active |
| [[reference/components/reader]] | 人物正式／翻阅分派、精神倍率与成功后回调、双端路由 | 麦斯威尔阅读精神倍率减半、人物读书能力 / active |
| [[reference/components/simplebook]] | 无 reader 的可见性检查／阅读回调、原弹窗与动作返回边界 | 烹饪指南阅读后发送模组事件、普通角色阅读物品 / active |
| [[reference/components/writeable]] | 单写入者会话、文本提交／保存、布局与保持距离 | 木牌编辑保持五单位距离、牌子题字 / active |
| [[reference/components/instrument]] | 演奏回调顺序、听众范围／筛选、SG 声音与动作消费 | 排箫听众搜索半径扩大百分之五十、乐器演奏范围 / active |

## 传送与魔术工具

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/blinkstaff]] | 瞬移资格、延迟落地／复查与即时消费、SG 恢复 | 懒人魔杖关闭瞬移音效、橙色法杖瞬移 / active |
| [[reference/components/recallmark]] | 回溯坐标／分片、标记复制／保存与宿主显示刷新 | 服务端重设回溯表标记、回溯点与分片 / active |
| [[reference/components/remoteteleporter]] | 目的台距离选择、附近物品集合、技能／耗能及 SG 恢复 | 传送伞周边物品半径改为三、薇诺娜传送台选择 / active |
| [[reference/components/magician]] | 人物魔术工具暂存／停止、归还与读档宿主协作 | 服务端结束麦斯威尔魔术工具会话、人物魔术工具暂存 / active |
| [[reference/components/magiciantool]] | 工具使用者握手、动态转换／回调与容器收尾 | 从魔术高礼帽结束当前会话、魔术工具回调 / active |

计时、瞄准与其他交互见[[reference/components/topics/abilities|交互技能与持续效果]]；通用移动／传送协议见[[reference/components/topics/movement|移动索引]]。

返回 [[reference/components/index|组件主题导航]]。
