---
title: 活动与节日组件索引
created: 2026-10-28
updated: 2026-10-29
type: index
tags: [overview]
---

# 活动与节日

万圣夜、嘉年华、生肖年与冬日盛宴等限时活动的专属对象与管理组件；活动期判定用 `SPECIAL_EVENTS`／`WORLD_SPECIAL_EVENT`。只读命中分组；适用版本与验证范围见具体页，已知名称可直读正文。

## 嘉年华游戏

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/carnivalgamefeedable]] | 巢可喂开关：enabled 赋值同步 carnivalgame_canfeed 标签，OnFeed/DoFeed 承载喂食 | 喂鸡巢可喂状态 / active |
| [[reference/components/carnivalhostsummon]] | 构造加 carnivalhostsummon 标签＋SetCanSummon；当前脚本无 AddComponent 与标签读取者 | 嘉年华主持人召唤标记 / active |

## 讲故事与演出

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/storytellingprop]] | 营火等物件标记为 TELLSTORY 目标 | 讲故事道具、TELL / active |
| [[reference/components/playbill]] | 戏单数据件：scripts/costumes 幕次，供讲台换幕与演出注册 | 戏单、演出剧本数据 / active |


## 节日装饰与冬日活动

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/wintertreegiftable]] | 冬季树收礼资格：距上次赠礼≥4 天给好礼，活动激活才挂载 | 冬季树收礼、赠礼冷却 / active |
| [[reference/components/carnivaldecor]] | 嘉年华装饰登记到附近 ranker 计分器并携带装饰值 | 嘉年华装饰 / active |
| [[reference/components/carnivaldecorranker]] | 统计范围内装饰价值换算广场等级并回调 | 装饰评分、广场等级 / active |
| [[reference/components/wintersfeasttable]] | 盛宴餐桌开餐开关：canfeast 同步字段驱动标签，按 finiteuses 耗食回调换菜 | 盛宴餐桌 / active |
| [[reference/components/gingerbreadhunter]] | 冬日盛宴姜饼猎捕：每 3 天远处放姜饼猪，多轮后结算 | 姜饼猎捕 / active |
| [[reference/components/wintersurprisespawner]] | 冬日盛宴惊喜树：冬季内在鹿群刷新点旁放满礼物的 winter_tree | 惊喜树生成器 / active |
| [[reference/components/snowmandecoratable]] | 雪球/雪人堆叠、帽子与装饰串 netvar，主端校验消耗背包物品 | 堆雪人 / active |

## 卡拉特赛跑与龙舟

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/yotc_racestart]] | 卡拉特赛跑起点开关，race_on tag 表达状态 | 赛跑起点组件 / active |
| [[reference/components/yotc_racestats]] | 卡拉特四维 0-5 属性：加点/退化/基线快照，纯数据无同步 | 赛跑鼠属性 / active |
| [[reference/components/yotc_raceprizemanager]] | 卡拉特赛裁判：起点线分桶、检查点、每日一次幸运金块 | 赛跑奖品 / active |
| [[reference/components/yotc_racecompetitor]] | 卡拉特赛跑选手：检查点链寻路、体力遗忘特质、冲线领奖 | 赛跑选手 / active |
| [[reference/components/yotd_raceprizemanager]] | 龙舟赛奖品额度＋检查点注册表，跨天恢复、活动关闭归零 | 龙舟赛奖品 / active |

## 小游戏、捉迷藏与高尔夫

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/minigame_spectator]] | 观众标记：登记观看的 minigame，结束自动自拆 | 小游戏观众 / active |
| [[reference/components/minigame_participator]] | 小游戏参与者登记：标签供 AI 避让，3s 无心跳自退 | 小游戏参与者 / active |
| [[reference/components/minigame]] | 小游戏主机：0.75s 脉冲收编观众/参与者，热度窗口 | 小游戏主机 / active |
| [[reference/components/hideandseeker]] | 捉迷藏寻找者每秒校验游戏与距离，失效自移除 | 捉迷藏寻找者 / active |
| [[reference/components/hideandseekhidingspot]] | 捉迷藏藏身点：运行时动态挂载，藏 kitcoon 出假身，互动即翻出自拆 | 捉迷藏藏身点 / active |
| [[reference/components/hideandseekgame]] | 捉迷藏小游戏裁判：藏匿点登记、吸找方、结算 | 捉迷藏游戏 / active |
| [[reference/components/carnivalgameshooter]] | 射击小游戏角度摆动瞄准与弧上抛射物发射 | 射击小游戏瞄准 / active |
| [[reference/components/golfpropitem]] | 高尔夫道具出界 0.5s 轮询回传标定位，掉落减速 | 高尔夫道具约束 / active |
| [[reference/components/golfclub]] | 球杆瞄准占用＋蓄力倍率＋命中事件链 | 高尔夫球杆 / active |
| [[reference/components/golfable]] | 高尔夫可击打目标：占用标签＋命中冲量＋外力接口 | 可击打高尔夫球 / active |
| [[reference/components/golfballspinner]] | 嘉年华旋盘：对 golfable 实体施加周期切向推力 | 高尔夫旋盘 / active |
| [[reference/components/golfclub_reticule]] | 高尔夫杆网络瞄准：target/chargeticks netvar、客户端动态 reticule | 高尔夫瞄准环 / active |

## 舞台比赛与剧本

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/playbill_lecturn]] | 剧本架换本：GIVE 换 playbill、驱动 writeable 文本与戏台 AddPlay | 剧本架 / active |
| [[reference/components/stageactor]] | 舞台演员：实活面仅标签＋SetStage/GetStage；performplay 无原版调用 | 舞台演员 / active |
| [[reference/components/stageactingprop]] | 查理舞台演出：戏服选角、剧本匹配、台词线程与中断 | 查理舞台 / active |
| [[reference/components/yotb_stagemanager]] | 牛年舞台赛协调：舞台登记、单槽比赛态、逐日开赛 | 牛年舞台管理 / active |
| [[reference/components/yotb_stager]] | 牛年舞台单机比赛组件：开赛检查、NPC 补牛、猜评判定与奖品投掷 | 牛年舞台比赛 / active |

## 其他活动对象与设施

| 页面入口 | 需求摘要 | 别名 / 状态 |
|---|---|---|
| [[reference/components/moontrader]] | 单向献祭交易：校验后收走祭品触发回调 | 月亮献祭交易 / active |
| [[reference/components/deckcontainer]] | 牌组容器：增删/按位/合并拆分/洗牌，空堆自动移除实体 | 牌组容器 / active |
| [[reference/components/playingcardsmanager]] | 森林世界扑克牌 ID 池：发牌/登记/移除回收 | 发牌、牌 ID 回收 / active |
| [[reference/components/furnituredecortaker]] | 家具承物端：收装饰物、锁定自身并与 furnituredecor 联动 | 家具放物 / active |
| [[reference/components/gym]] | 卡墩训练循环：timer 计时、trainfn 注入效果、夜休 | 卡墩健身房 / active |
| [[reference/components/bathingpool]] | 泡澡池占用协议：EnterPool/LeavePool 定位避让，SG 双向契约 | 泡澡池 / active |
| [[reference/components/vaulttorchgrid]] | 火炬网格布置/邻接遍历/状态位打包存档 | 火炬网格 / active |
| [[reference/components/mightygym]] | 健身器械：双槽配重、入/出会话状态交换、收益换算 | 健身器 / active |
| [[reference/components/quagmire_recipeprices]] | 熔炉菜品估价广播队列：主模拟 3 秒/条写 netvar | 暴食估价 / active |
| [[reference/components/quagmire_recipebook]] | 熔炉菜谱发现广播队列（与估价组件同实体协同） | 暴食菜谱 / active |

返回 [[reference/components/index|组件主题导航]]。
