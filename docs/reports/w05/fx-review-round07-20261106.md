# FX 独立检视第 7 轮（合并原生成批 8、9）

初检日期：2026-11-06。初检 P1×0、P2×1，另有 P3×1 非阻断措辞建议。**同日修后独立复验：R01、R02 均已覆盖，未发现新的 P1/P2，第 7 轮 9 页可收口。** 下文初检保留历史，以文末复验为当前结论。本轮按用户建议合并相邻小批次，只读知识页与源码，写报告并追加日志；不修知识页、不 record、不改索引、PLAN、台账、Skill 或工具，不提交推送。未运行游戏。

## 范围与工作量

- 实读 `raw/current/BUILD.txt`：`26335394`；源码根 `D:/Code/dst-wiki-develop/dst-wiki/raw/current/scripts`。K HEAD：`97afb4e9e6ca05390558769926f1f9c8598d0f70`。
- 采用 [覆盖台账原批 8、9](fx-coverage-ledger-20261101.md#L1021-L1022)：F21～F25、F27～F29、F31，共 9 页、44 个通用表注册。台账既有 accepted／复审自报只作线索。
- 页面的 `moon_altar_link_fx_spawner` 别名用于生成器定位，不另抵扣本批通用表注册数量；正文已区分生成器与 FX。
- 核对 Git 状态，保留上一轮修订及其他 Agent 的装备扩库；S／T 状态干净。本轮不替第 5 轮 R10 改变独立复验状态。

| 页面 | 注册数 | 物理行数／UTF-8 字节 | 本轮结果 |
|---|---:|---:|---|
| [messagebottle_break_fx](../../../wiki/reference/prefabs/messagebottle_break_fx.md) | 4 | 51／4,014 | 未发现 P1/P2 |
| [waterplant_destroy](../../../wiki/reference/prefabs/waterplant_destroy.md) | 5 | 55／5,028 | 未发现 P1/P2 |
| [shadow_teleport_in](../../../wiki/reference/prefabs/shadow_teleport_in.md) | 4 | 55／5,084 | P2：R01；P3：R02 |
| [moon_altar_link_fx](../../../wiki/reference/prefabs/moon_altar_link_fx.md) | 1 | 47／3,035 | 未发现 P1/P2 |
| [farm_plant_happy](../../../wiki/reference/prefabs/farm_plant_happy.md) | 4 | 54／5,492 | 未发现 P1/P2 |
| [alterguardian_spintrail_fx](../../../wiki/reference/prefabs/alterguardian_spintrail_fx.md) | 7 | 59／7,219 | 未发现 P1/P2 |
| [moonstorm_glass_fx](../../../wiki/reference/prefabs/moonstorm_glass_fx.md) | 4 | 52／3,755 | 未发现 P1/P2 |
| [icespike_fx_1](../../../wiki/reference/prefabs/icespike_fx_1.md) | 9 | 58／9,164 | 未发现 P1/P2 |
| [dr_warm_loop_1](../../../wiki/reference/prefabs/dr_warm_loop_1.md) | 6 | 57／6,082 | 未发现 P1/P2 |

合计 488 个物理行、48,873 字节。moon_altar_link_fx 为 simple，其余默认 standard，均在当前预算内。

## 待修 P2

### R01｜交换搜索的 radius 上限不能表述为实际搜索半径

- 页面：[shadow_teleport_in L47](../../../wiki/reference/prefabs/shadow_teleport_in.md#L47) 写“搜索到达半径 30 时均不生成对应 in”。这容易使查询者把 30 理解为世界坐标中的搜索半径。
- 实现：[陆到海 L198-L210](../../../raw/current/scripts/prefabs/shadowcreature.lua#L198-L210)、[海到陆 L291-L303](../../../raw/current/scripts/prefabs/oceanshadowcreature.lua#L291-L303) 的确检查 `radius >= 30`，但每轮先 `radius = radius + 2`，再把这轮 offset 累加到已经移动过的 `sx/sy/sz`，不是每次从原始坐标加一个半径。沿同一方向的累计位移因此可以超过 30。
- 例如按代码运算，步长依次为 2、4、6、8、10、12 时，累计位移为 42，而循环变量仅为 12；若此时抵达所需地形，不会因 `radius >= 30` 被拒绝。这是源码算术示例，不是游戏运行结果。
- 建议只改这一句为“目标缺失、目标地形不符，或搜索循环变量 `radius` 达到 30 时，不生成对应 `in`”；若继续解释距离，加一句“步长逐轮累加到当前位置，30 不是实际搜索半径”。保留既有目标门禁、交换失败仍移除旧实体和 `out` 不保证成功的结论。无需扩写搜索算法或另建机制页。

## 非阻断 P3

### R02｜宝盒延时从开始播放 activation 计算

- 页面：[shadow_teleport_in L43](../../../wiki/reference/prefabs/shadow_teleport_in.md#L43) 的“激活动画后延迟 174/30 秒调用 teach”可以理解为动画结束后再计时。
- [archive_lockbox L102-L108](../../../raw/current/scripts/prefabs/archive_lockbox.lua#L102-L108) 在 `PlayAnimation("activation")` 后立即排 `DoTaskInTime(174/30, ...)`，没有等待 animover。
- 建议顺手改成“开始播放 `activation` 时排入 `174/30` 秒后的 `teach`”。现有数值和 builder／recipe 门禁无误，不把这处措辞单独作为收口门槛。

## 已核对的关键事实

- **瓶与水植**：瓶子水中播自身 bob／bob_gelblob，不等于生成 bob FX；歌贝在主模拟按平台 wood 标签选名。水植升级、弹体命中、避雷针充能与随机续排均能对上实际调用，伤害／保护归生成者，依赖声明未被误当生成。
- **暗影与宝盒**：陆上状态入场及第 40 帧各生成一次 out，海上第 42 帧生成 out；in 由交换成功分支生成。蘑菇地精记录攻击入场位置，第 23 帧出咳出 FX、第 25 帧起出孢子；宝盒 AddChild 和是否已知蓝图的分支符合正文。
- **月坛与月脉冲**：生成器每 0.2 秒步进 3，距目标不大于 2 时移除；月脉冲在非专服创建的本地控制器生成光点。光点距离为 4 加随机 0～32，小／大波次数为 8／17，续排间隔为 0.04 加随机 0～0.07；没有把主模拟与非专服混为一谈。
- **作物与活动**：happiness 类别回调不等于总压力；杂草照料延时生成 happy；枕头战仅合法参赛者越界时生成彩纸。yotb_confetti 的依赖声明未被当成生成点。
- **天体英雄与月装置**：尖刺先执行 DoAttack，再按返回值选择碎裂 FX 或尖刺实体；旋转轨迹开启 animqueue、循环 idle，60 帧后启动 60 帧 ErodeAway。装置 breaksequence 延迟 9 帧破裂、再延迟 1 帧出 Boss；陷阱移除回调负责末尾 FX 和掉落。
- **月暴玻璃**：crack 动画结束后同点出两层 FX，音效与攻击由玻璃负责；定时和 defused 存档加载可进 explode，work 用完的路径不同。
- **采掘与雕像**：PlayMiningFX 选名优先级、Transform 门禁、独立声音门禁与正文一致。全树查得 16 处 PlayMiningFX 文本命中，扣除定义后为正文所列的 15 处调用；战争处 `_2` 是注释，未误记为生成。
- **探测杖与 WX**：四档阈值及 EFFECTS 映射、无目标不选 FX、代理 Follower 与卸装移除均核对；音乐模块周期／技能切换／已有 leaderrollcall 的不同分支符合正文。现行 HeatSteam helper 直接 CreateEntity 播 steam_1／steam_2，不是旧 wx78_heat_steam 注册的生成证据。

## 检索与机械校验

| 检查 | 本轮实际结果 |
|---|---|
| validate_page --check-indexes | 所选 9／9 页通过；全库导航通过：1122 知识页、34 索引、0 问题 |
| validate_source_links --strict | 181 处行号引用、174 个唯一目标、61 个源码文件；0 error、0 warning，未截断、无跳过 |
| validate_source_hashes check --fresh-hours 0 | 9 页 78 组比较均 unchanged；changed／sources_changed／invalid／unrecorded 为 0；未 record |
| sources 列用审计 | 9 页正文直接引用的源码均已声明；通过不意味着所有隐含依赖已自动证明完整 |
| Markdown 章节 read | 9 页各读标题原文“关联机制导航”，9 次退出 0；不依赖 search 缓存 |
| 注册名主要归属 | 本批 44 名和额外 spawner 定位别名在 Prefab aliases 中均只有一个主要页 |
| 简中名称键 | 对上述 45 个名字逐一精确查 STRINGS.NAMES，无命中；中文为效果描述，未冒称注册名官方译名 |

检索覆盖 `raw/current/scripts` 根级与子目录的 4095 个 Lua 文件。完整名／关联 spawner 共 199 行字面候选全部读取；生成、表项、依赖、注释分开判断。另复查 `messagebottle`、`yotb`、`wx78`、`icespike`／`ice.?spike` 的拼接及生成候选，并追 EFFECTS、music_spawn_fx、现行 HeatSteam helper 和 PlayMiningFX。未发现足以推翻页面内 bob、牛年彩纸、旧冰刺、旧蒸汽保守缺口的生成证据；这不是不存在或不可用的证明。未运行 DST，未审动画素材，不宣称 runtime-tested；9 页没有 Lua 围栏示例，不另跑 Lua 语法。

## 后续轮次

固定 158 页范围内，本轮新增检视 9 页，累计 **49／158 页，余 109 页**；“已检视”包含待修页，不等于全部独立收口。按每轮 8～12 页，余量约 10～14 轮新检视，建议以约 10 页为目标，按调用链复杂度微调；修后复验另计，不按问题少就省略源码核对。下一轮可合并原生成批 10、11（11 页、69 注册），再按页面复杂度决定是否拆开。

## 2026-11-06 修后独立复验与收口

**R01、R02 均已覆盖，第 7 轮可收口，未发现新的 P1/P2。** 本次只定向复验 shadow_teleport_in 的修订及相邻上下文；其余 8 页沿用本轮初检结论，已比对文件哈希确认未变，不冒称重新全文检视。

| 项目 | 当前修订与复验 |
|---|---|
| R01（P2） | L47 明确 `radius` 为循环变量，先加 2 再把 offset 累加到当前位置，30 不是实际搜索半径；与两个交换函数一致。目标门禁、失败仍删除旧实体及 out 不保证成功的边界保留。 |
| R02（P3） | L43 明确开始播放 activation 后即排入 174/30 秒后的 teach，不等 animover；与 OnTeach 一致。 |
| 同步面 | L52 验证记录已补两项修订；摘要、快速判断、索引登记未含原错误半径或计时表述，无需为本次改动重写。sources 未变，无须 record。 |
| 页面与导航 | 1／1 页通过；全库导航通过：1124 知识页、34 索引、0 问题。知识页数量增加来自并行扩库。 |
| 源码引用与基线 | strict 19 处引用，0 error、0 warning；8 组来源比较全部 unchanged，无 sources_changed／invalid／unrecorded。 |
| 独立章节读取 | 标题原文“关联机制导航”“常见陷阱与限制”“验证记录”三次 read 均退出 0。 |

当前修订页仍为 55 个物理行、5,419 字节。非阻断维护提醒：updated 仍为 2026-11-01，后续可同步修订日期，不据此重开技术问题。本次不修改知识页或基线，保留他方并行增量；未运行游戏、未提交推送。复验不增加页数，累计仍为 **49／158 页，余 109 页**；本次不替第 5 轮 R10 收口。
