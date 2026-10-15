---
title: "feasts"
created: "2026-10-24"
updated: "2026-10-28"
type: component
summary: "在服务端世界中管理冬季盛宴餐桌连通组与进餐者，周期扣桌上食物并追加盛宴Buff，实际动作与效果由宿主分工。"
aliases: ["feasts", "冬季盛宴聚餐", "盛宴餐桌分组", "多人聚餐Buff", "feasterstarted"]
tags: [component, building, food, event, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/feasts.lua", "components/wintersfeasttable.lua", "prefabs/forest.lua", "prefabs/cave.lua", "prefabs/table_winters_feast.lua", "prefabs/wintersfeastbuff.lua", "stategraphs/SGwilson.lua", "actions.lua", "componentactions.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# feasts

## 快速判断

- 职责与适用：服务端世界的餐桌分组与参与者管理器；forest／cave 已安装。餐桌本体用 wintersfeasttable，人物用 SG，不能把世界组件直接挂给食物或人物。[主模拟断言](../../../raw/current/scripts/components/feasts.lua#L5-L7) [地面世界](../../../raw/current/scripts/prefabs/forest.lua#L691) [洞穴世界](../../../raw/current/scripts/prefabs/cave.lua#L541)
- 优先使用：沿原版餐桌的 RegisterTable／UnregisterTable 生命周期与人物进餐事件接入；只查询时用 GetTableGroup、GetFeasters／GetTableGroups，不直接改返回的内部表。[查询接口](../../../raw/current/scripts/components/feasts.lua#L190-L200) [餐桌注册](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L536-L540)
- 重要副作用：有参与者时每 1/3 秒先调用同组各桌的 DepleteFood，再为每人追加 wintersfeastbuff，并按节奏播放台词；不是每次右键立即分发普通 edible 收益。[周期](../../../raw/current/scripts/components/feasts.lua#L80-L100) [Buff委派](../../../raw/current/scripts/components/feasts.lua#L57-L77)
- 修改边界：事件没有完整目标／重复登记验证；正常入口要求桌子已分组、库存和桌组件可用，人物走成对 SG 进出。没有网络变量或组件存档，不能当完整新餐桌／多人交互骨架。[事件接线](../../../raw/current/scripts/components/feasts.lua#L134-L184) [人物退出](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22643-L22653)

## 关键接口

| 接口／事件 | 约定与副作用 | 证据 |
|---|---|---|
| `RegisterTable(table)` | 按周围带 wintersfeasttable 标签的已登记桌合并分组，并给邻桌排 ruffle。非幂等接口；不会为实体添加库存／食物／桌组件 | [分组与合并](../../../raw/current/scripts/components/feasts.lua#L218-L256) [排表现任务](../../../raw/current/scripts/components/feasts.lua#L258-L267) |
| `UnregisterTable(table)` | 移出目标后按距离重建原组的连通分组；没有通用成功返回或自动清除玩家进餐状态的承诺 | [实现](../../../raw/current/scripts/components/feasts.lua#L270-L307) |
| `GetTableGroup(table)` | 返回当前数字组索引或 nil；合并／删除会改变索引，不是可保存的稳定组 ID | [查找](../../../raw/current/scripts/components/feasts.lua#L47-L55) [接口](../../../raw/current/scripts/components/feasts.lua#L190-L192) |
| `GetTableGroups()`／`GetFeasters()` | 返回内部活表，分别含餐桌数组和 `{player,target}` 记录；查询不复制，改表会影响管理器 | [定义与返回](../../../raw/current/scripts/components/feasts.lua#L20-L21) [接口](../../../raw/current/scripts/components/feasts.lua#L194-L200) |
| 世界 `feasterstarted` | 载荷 `{player, target}`；直接追加记录、监听 player.onremove，首人创建 world.feastingtask；不去重、不验证目标已分组 | [进入](../../../raw/current/scripts/components/feasts.lua#L134-L145) [监听对象](../../../raw/current/scripts/components/feasts.lua#L183-L184) |
| 世界 `feasterfinished` | 载荷 `{player,target,is_in_dark?}`；按 player 删首个记录，最后一人取消任务；不是按传入 target 匹配。另取消旧结束台词任务，非黑暗离场再延迟播报 | [退出](../../../raw/current/scripts/components/feasts.lua#L148-L175) |
| `GetFeasterGroup(player)` | 当前实现循环变量为 chardata，却读取未在函数中定义的 `data.target`，属于源码疑点；不作为推荐查询入口 | [实际实现](../../../raw/current/scripts/components/feasts.lua#L202-L215) |

### 一轮聚餐如何处理

1. 先把参与者按其 target 当前所在餐桌组归并。每组每张桌调用一次 `DepleteFood(#feasters)`；但桌组件实际忽略该参数，只对第 1 槽有限次数物品 `Use()` 一次。不能写成“人数翻倍就每轮扣两倍份数”。[世界调度](../../../raw/current/scripts/components/feasts.lua#L80-L97) [实际扣量](../../../raw/current/scripts/components/wintersfeasttable.lua#L65-L79)
2. 随后再次读同组各桌的 inventory 第 1 槽，只有带 wintersfeastcookedfood 的物品计入：总份数是有食物的桌数，不是堆叠数量；不同食物种类按 prefab 去重。再给每位参与者 AddDebuff 并向效果传人数／种类数／总份数。[统计与施加](../../../raw/current/scripts/components/feasts.lua#L57-L77)
3. 效果的 attached／detached 管理饥饿耗速、精神持续贡献，另有周期回血／补饥饿与到期停止；人数和食物统计主要转为追加 buffover 时间，并非直接改变组件内固定营养字段。持续效果基础分工复用 [[reference/components/debuffable]]／[[reference/components/debuff]]。[效果接入与撤销](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L37-L74) [时长追加](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L78-L97) [周期收益](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L9-L22)

## 典型接入

### 参考原版餐桌的注册、进餐与中断

这是现有冬季盛宴系统的协作指引，不是可独立加载的新餐桌 demo；自制物品／建筑仍需资源、库存、交互和 SG 方案。

1. **桌子先就绪：** 原版工厂安装 wintersfeasttable、inventory 和 shelf，绑定耗用／结束回调；零延时任务仅在未 burnt 时向世界注册。库存放食物、readyforfeast 标签及显示由桌子宿主控制，注册本身不使空桌可吃。[桌组件与库存](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L490-L499) [延迟注册](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L536-L540)
2. **动作与开始：** 右键候选要求 readyforfeast 且未 fire／burnt；动作函数自身只 return true，真正进入由服务端 ActionHandler 向世界推 feasterstarted，再进 winters_feast_eat。人物状态另在目标的 current_feasters 表登记自己。[候选](../../../raw/current/scripts/componentactions.lua#L977-L980) [空动作函数](../../../raw/current/scripts/actions.lua#L5217-L5219) [开始事件](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1559-L1565) [桌上登记](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22584-L22602)
3. **续吃与退出：** SG 动画结束还可继续入同状态，用 keep_eating 避免每轮重新加入世界名单；真正退出才删桌上记录、停声音、发 feasterfinished。看不见桌子也会退出并标记黑暗原因。不要只播放动画或单发开始事件模拟完整聚餐。[续轮与退出](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22623-L22653)
4. **关停与移除：** canfeast=false 会撤 readyforfeast 并对当前进餐者发 feastinterrupted。原桌起火／烧毁／删除沿这些路径处理；删除先 CancelFeasting，再 UnregisterTable，避免世界名单指向已移除桌子。[桌组件中断](../../../raw/current/scripts/components/wintersfeasttable.lua#L10-L18) [玩家中断接收](../../../raw/current/scripts/stategraphs/SGwilson.lua#L2423-L2428) [宿主清理](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L268-L304)

### 分组、保存与效果到期的边界

桌距使用 TABLE_RANGE，当前为 3.5；连通链可把不直接相邻的桌放进同组。RegisterTable 在新桌有平台时才要求已登记邻桌同平台，新桌不在平台时不施加此条件；UnregisterTable 重连只按距离，不复查平台。不要把“同船／同岸隔离”作为所有路径保证。[注册筛选](../../../raw/current/scripts/components/feasts.lua#L221-L237) [重连](../../../raw/current/scripts/components/feasts.lua#L286-L300) [范围](../../../raw/current/scripts/tuning.lua#L5169-L5174)

世界组件没有 OnSave／OnLoad；餐桌重建后重新注册，人物当前进餐状态也不由此组件保存。离场只停止参与追加和安排台词，**不会立即撤销已获得的 Buff**；效果自己的计时器到期才 Stop，基本持续效果和存读责任不能由餐桌组索引替代。[退出函数](../../../raw/current/scripts/components/feasts.lua#L148-L175) [效果到期](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L72-L76) [效果装配](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L123-L132)

## 常见陷阱与限制

- **手动重复发开始事件或重复 RegisterTable：** 两处都无去重门禁，可能重复计人数／餐桌、消费或施加效果；退出只删首个参与记录，不能把它当整人清空。使用既有 SG／工厂接线，不在每帧补发注册。[参与者追加／删除](../../../raw/current/scripts/components/feasts.lua#L134-L160) [桌登记](../../../raw/current/scripts/components/feasts.lua#L218-L248)
- **未注册目标也可安全发事件：** 周期查询依赖 target 能找到组；无组会产生 nil 组键，后续访问不具备防御性降级。新入口应先确认桌组件、库存与分组，不能把世界事件当服务端授权 API。[组查询与写入](../../../raw/current/scripts/components/feasts.lua#L80-L93)
- **按名字推测 GetFeasterGroup 正常工作：** 读 `data.target` 而非 chardata.target 的疑点在命中参与者分支才发生，未运行复现；查询时可按原桌 OnDepleteFood 的 GetFeasters→GetTableGroup(set.target) 模式，不悄悄修正游戏代码。[疑点](../../../raw/current/scripts/components/feasts.lua#L202-L215) [原版查询](../../../raw/current/scripts/prefabs/table_winters_feast.lua#L149-L160)
- **人数越多仅提高直接恢复值或有硬时长封顶：** 人数、种类和总桌食物数传给 Buff 的加时公式；AddEffectBonus 直接把 bonus 加到剩余时间，没有按 MAXDURATION 钳制。MAXDURATION 在本效果的强度计算中另有用途，不把常量名当上限保证。[加时](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L78-L97) [强度](../../../raw/current/scripts/prefabs/wintersfeastbuff.lua#L25-L27)
- **移世界组件就取消所有任务与关系：** 该定义没有移除清理钩子；世界周期任务、参与者移除监听和个人结束台词任务由上述流程管理。不能只 RemoveComponent 后期待它们全部撤销，自定义替换须先停止实际进餐并设计注册迁移。[任务和监听](../../../raw/current/scripts/components/feasts.lua#L130-L175)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的变更文件（actions.lua、componentactions.lua、tuning.lua、stategraphs/SGwilson.lua、prefabs/forest.lua、prefabs/cave.lua）对照更新报告 R-01/R-04、SB-17 与 hunk 地图逐条核验，进餐候选、SG 进出事件、feasts 挂载与 TABLE_RANGE 均无语义变化（SB-17 为 Rifts 8 新增内容，winters_feast_eat／feastinterrupted 区段经旧新逐行比对一致；cave 变更为 vaultroommanager→rockybosspawner 替换，不触及本页锚点），锚点重定位 9 处、结论修订 0 处；未覆盖：运行验证。

- 源码审阅：2026-10-24，Steam buildid 25643519；完整 feasts，核验 forest／cave 挂载、餐桌注册／退出／食物扣量、人物动作／SG 成对事件、Buff 统计／加时和效果分工。GetFeasterGroup 自由变量疑点按源码记录；未全面审校餐桌投料与所有食物、Buff存读、客户端预测和跨船移动。
- 结构检查：本页格式、来源、本地链接和源码锚点严格检查通过；无完整 Lua 示例，导航登记由本批统一检查。
- 游戏运行：未运行；多人登记、连通桌链、动态移桌／跨平台、食物最后一份时序、台词和Buff实际表现未实测。
