# 潮湿与冰冻组件检视

2026-10-16；Steam buildid 25643519。源码准备返回 Skip，本轮未解压或改变快照。第六批 moisture 首页已在独立安装 Skill 的临时副本试建、经人工源码审阅修正后发布，经过与边界见当日独立使用记录；本次在当前主会话按知识 Skill 规则补齐 moistureimmunity／freezable，不新增独立模型评测或游戏运行。

## 页面与范围

| 页面 | 本次交付 | 物理行／UTF-8 字节 |
|---|---|---:|
| [moistureimmunity](../../../wiki/reference/components/moistureimmunity.md) | 目标多来源免湿、标签与 ForceDry、生命周期、原版限时食物 Buff 与 carrot 完整接入 | 89／9,913 |
| [freezable](../../../wiki/reference/components/freezable.md) | 积累／抗性、两段计时、受击、SG／Brain、保存与移除；猪人与冰杖两份独立接入 | 108／13,546 |
| [moisture](../../../wiki/reference/components/moisture.md) | 保留已交付正文，仅将免湿待建说明改为实际页面互链 | 101／10,924 |

组件索引新增两条唯一登记，现 126 行／9,617 字节；正文地址保持稳定，导航拆分留下一工作项。51 组件、3 modinfo、5 cookbook，共 59 知识页／9 导航；798 个组件候选中 747 未建，另 23 个非组件页候选分类不变。覆盖 CSV 本次仅改 freezable／moistureimmunity 两行，其余行与输入哈希核对保留。

模板沿用已有五节；freezable 需整体说明计时／SG 协作，使用已允许的“关键流程”。未新增必填字段、通用模板章节、独立组合方案或 cookbook，也未改变 Skill 使用规则、脚本、工具箱、Schema、正式 reviews／LuaCATS 或 fork。

## 审阅时明确的边界

- moistureimmunity 添加组件后仍须 AddSource；组件挂在目标，来源用独立实体，同来源重复不是引用计数。最后一个来源撤销后组件自动删除，再次使用必须重新检查是否存在；普通卸装／失效时显式撤销，实体删除才用 onremove 自动清理。
- ForceDry 清零／停更新与标签湿润查询分别说明；永久 wet 标签优先，原版 foodbuffs 的附着函数也跳过这类目标。Buff 存在或 AddDebuff 返回 true 不能单独证明已生效。后添加 moisture 不会自动重放已有来源。
- 原版 buff_moistureimmunity 已由 foodbuffs 注册，carrot 接入复用其时长、同键刷新、死亡／到期与脱附清理，不另造效果工厂。来源集合本身没有存档接口，恢复由具体效果／装备系统承担，不把 debuffable 的恢复推广到所有来源。
- freezable 的 coldness 是积累量，不是摄氏温度。负数 AddColdness 不切状态；nofreeze 的判断只在有 SG、未冻结且达阈值的分支，不是通用开关。Freeze 的可见／存活门禁、重定向与重复冻结保留。
- 冻结 task 到期先 Thaw，第二段才 Unfreeze；freezetime 与 GetTimeToWearOff 不是完整控制总时长。extraresist 参与阈值和时长，Thaw 的时长实际经历两次 ResolveWearOffTime，不改源码或臆测设计意图。
- IsFrozen 包含 THAWING；受击打碎依据 attacked 与累计绝对伤害、默认阈值 0。Reset／RemoveComponent 都不完成 Unfreeze 的 Brain／事件恢复；仅保存正 extraresist，未保存冻结进度、task 剩余或基础配置。
- 猪人的 helper／SG 注册及公共冻结状态处理、玩家工厂覆盖参数、冰杖 blue → blue_common → onattack_blue 消费链分别核验。冰杖示例只设工厂已有字段，保留原攻击回调，不声称任意新生物仅挂组件即可完整冻结。

## 需求与实际读取

这是已读源码作者完成页面后的内容检视。没有同条件直接源码对照、准确模型 token 或新会话自主路由验收；下表是工具 stdout 的 UTF-8 字节，不是模型上下文或 token。

| 需求／调用 | 输出字节 | 命中与读取结果 |
|---|---:|---|
| search：食物免湿＋防水Buff，限 components | 7,644 | 新页两个章节排在前列，也返回 waterproofer／debuff／debuffable；五候选上限，omitted=4、truncated=true，保留省略标记 |
| read：carrot 接入＋免湿陷阱 | 7,388 | 同页两目标合并，含快速判断、完整代码／文件环境、验证记录，完整返回；ranges.text 5,938 字节 |
| search：冰杖冻结＋冻结抗性，限 components | 4,885 | 前两候选为 freezable，另召回 planardefense，未省略；条件需读页判定 |
| read：冰杖接入＋积累／两段解冻 | 6,534 | 同页两目标合并，完整返回；ranges.text 5,016 字节 |
| read：普通猪人抗性／阶段时长 | 4,959 | 独立示例连同前提与验证边界完整返回；ranges.text 3,524 字节 |

食物任务读取链为：需求检索 → 新页原版 Buff 接入／限制 → 必要时链接 debuffable 的保存边界。冻结任务读取链为：需求检索 → 新页冰杖字段／流程或猪人配置 → 新宿主差异才回查 SG／工厂。完成页面后的这些检查没有新增源码回查，但作者已有核验上下文，不能据此推断独立模型也无需源码。

章节读取暴露冰杖示例原文用“同样追加”依赖前一案例的环境说明，已改为本节独立注明目标 modmain、资源、双端安装与全局代理前提；没有把本页个例升级为新的模板规则。最后检查均使用修正后的页面。

## 检查与限制

- 四个改动 Wiki 文件通过元数据、来源、大小、标签和链接检查，整库 59 知识／9 导航通过既有构建器的全库结构／导航检查；可重建缓存更新到本次内容，未重写人工导航。
- 两新页 73 个源码行锚点检查存在且非空，并保存行内容与文件哈希；游戏语义另按上文范围审阅，不由链接检查替代。17 个主要来源文件及 BUILD.txt 哈希保持。
- 三段发布 Lua 通过既有 Full Moon 3.0.0／Lua 5.1 探针的语法、原 token 字节和精确往返检查；未运行 LuaLS、全量提取或工具箱回归。
- 未运行游戏：免湿食用／叠加／撤销、永久湿润、死亡与重载，冻结连续／间隔命中、受击／重冻、变身／读档与客户端动画仍待实测。页面保持 source-reviewed，不将静态检查写成运行通过。
- 原始校验、源码哈希／锚点、查询输出和 Lua 代码／探针结果在 `output/component-batch-6-20261016/`，Git 忽略。清理该目录下三个临时 Lua 文件及 syntax 目录被自动审批以 `blocked by policy` 拦截，未给具体原因，暂留；没有以其他方式绕过，也未尝试删除先前评测目录。

本批完成；下一步按 PLAN 拆组件主题导航，再推进第七批效率／消耗。独立模组经验提炼、版本增量任务、其他 harness／平台与游戏运行继续各自待办，不被本批内容检视替代。
