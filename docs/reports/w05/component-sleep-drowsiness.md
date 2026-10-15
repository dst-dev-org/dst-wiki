# 睡眠与困倦组件检视

2026-10-17；Steam buildid 25643519。源码准备返回 Skip，同 build 未解压。使用当前 dst-wiki Skill，在主会话审阅实现、必要消费者／宿主后发布第八批四页；没有独立模型评测或游戏运行。组件主题导航拆分仍为单独待办，本批只向现有索引登记。

## 交付范围

| 页面 | 覆盖内容 | 物理行／实际 UTF-8 字节 |
|---|---|---:|
| [grogginess](../../../wiki/reference/components/grogginess.md) | 困倦、抗性／免疫来源、更新单位、昏睡 SG 与死亡／复活重建 | 76／10,042 |
| [sleeper](../../../wiki/reference/components/sleeper.md) | 生物睡醒测试、催眠门槛、Brain／SG、离屏／清理与兔子日程 | 79／10,018 |
| [sleepingbag](../../../wiki/reference/components/sleepingbag.md) | 床具参数与占用、动作／SG、回调、唤醒与帐篷治疗量 | 79／10,837 |
| [sleepingbaguser](../../../wiki/reference/components/sleepingbaguser.md) | 人物倍率、ShouldSleep 消费差异、tick 顺序与退出 | 81／9,770 |

当前 58 篇组件、3 篇 modinfo、5 篇 cookbook，共 66 篇知识页与 7 篇导航；覆盖 CSV 仅改本批四行，798 个组件候选中 740 未建。组件索引 138 行／10,865 字节，页面地址与原索引锚点保持不变；无空分类、新 cookbook 或模板扩展。

## 核验得到的边界

- grogginess 的 decayrate 每次更新直接扣量，没有乘 dt；昏睡与恢复时间才累加 dt。SetPercent 的负差量被 AddGrogginess 拒绝，不能拿它清空困倦。禁用减速不等于免疫，IsKnockedOut 查 SG 标签。
- 人物死亡移除 grogginess、复活重建，因此抗性示例用 ComponentPostInit 加独立来源，保留工厂随后设置的基础门槛；不把普通 PlayerPostInit 当全部动态组件恢复入口。门槛、免疫和 SG 接受／苏醒条件各自独立。
- sleeper 的恰好门槛分支延迟 resistance 秒请求睡眠；任务句柄不在 StopTesting／WakeUp 内撤销。GoToSleep(sleeptime) 的时长是首次唤醒测试延迟，还受倍率、测试／hibernate 和 SG 影响，不保证准确时长。
- 生物睡眠会停 Brain、清战斗目标、发 gotosleep；状态图处理动画。离屏 OnEntitySleep 只是停止测试。移 sleeper 组件不主动唤醒，需先处理 Brain／SG 状态；只保存额外抗性，不保存完整睡眠。
- sleepingbag 是床具侧，sleepingbaguser 是人物侧；床具建立和清除双向关联，人物任务结算恢复。直接调用任一内部方法不能替代 SLEEPIN 动作、SG 的危险／饥饿／时段判断。
- ShouldSleep 原样返回 success／reason，但当前 bedroll 消费的是 reason；false, nil 不能在该入口拒绝，tent 没有相同调用。未提供“统一禁睡”的误导代码。
- tick 先扣饥饿，再回精神，未挨饿且 healthsleep 开启才治疗；温度回调后才因挨饿请求醒来。干燥率与 ambient_temp 由 moisture／temperature 更新，非 SleepTick 的一部分。
- DoWakeUp(true) 只跳使用者 SG 切换，仍执行 onwake；帐篷依然扣次数。使用中移人物组件没有自带退出钩子，应从床具清理，不手写关系字段。

## 需求与章节读取

通过标题／aliases 的定向搜索定位四个目标，再使用 query_wiki 的 read 读取真实章节。以下是作者受控内容检视，不是独立 AI 路由评测；仅记录输出字节，不换算 token 或宣称节省比例。四项均未截断／省略，目标章节、快速判断和验证记录完整返回。

| 需求 | 对应章节 | JSON 输出 UTF-8 字节 |
|---|---|---:|
| 人物抗性提高且复活后仍生效 | grogginess／角色困倦抗性增加两点并随组件重建恢复 | 4,708 |
| 兔子改为夜行 | sleeper／让原版兔子白天睡、夜间活动 | 4,527 |
| 睡床回血翻倍、饥饿扣量减半 | sleepingbaguser／人物睡床回血翻倍且睡眠饥饿扣量减半 | 4,483 |
| 自定义可睡回调能否统一禁睡 | sleepingbaguser／只限制睡袋入口时如何接可睡回调 | 3,842 |

## 检查与未覆盖范围

- 五个 Wiki 文件（四新页＋原组件索引）的元数据、来源、标签、大小、本地链接／锚点及全库唯一登记／父入口／Top 30 检查通过；未改其他知识页技术结论。
- 本批 172 处源码行引用 strict 检查为 0 error／0 warning，无截断；路径／行号通过不证明技术语义，结论由上述实现／消费者审阅支撑。
- 四段独立 modmain 示例经已有 Full Moon 3.0.0 探针 syntax-only 解析通过；本次未检查 token 位置／原字节往返、类型或游戏效果。临时四份 Lua 及本轮临时目录均已清理。
- 未执行全量提取、工具回归或缓存重建，未修改 Skill／工具箱／游戏源码或输入模组，未提交／推送。未审校所有角色／Boss、床具变体或跨分片协议；全部新页保留 source-reviewed。
- 游戏仍需验证昼夜、催眠边界、死亡／复活、三维临界值、起身／销毁／断线、多人显示和跨模组覆盖。

第八批完成；后续组件批次从治疗与修复继续，组件主题导航、heatrock Prefab 与配方队列保留，不将它们作为继续组件写作的硬前置。
