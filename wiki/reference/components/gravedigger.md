---
title: gravedigger
created: 2026-10-22
updated: "2026-10-28"
type: component
summary: 为整墓迁移工具提供使用回调，理解骨灰坛先保存墓碑、再由目标拆除的顺序。
aliases: [gravedigger, 骨灰坛迁墓, 整墓搬迁工具, GRAVEDIG]
tags: [component, item, callback, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/gravedigger.lua, components/gravediggable.lua, actions.lua, componentactions.lua, prefabs/graveurn.lua, prefabs/gravestone.lua, prefabs/skilltree_wendy.lua, stategraphs/SGwilson.lua, stategraphs/SGwilson_client.lua]
reliability: verified
validation: source-reviewed
status: active
---

# gravedigger

## 快速判断

- 职责与适用：挂在迁墓工具上，仅把使用者与目标转交 `onused`；原版宿主是 `graveurn`。目标侧用 [[reference/components/gravediggable]]，普通挖掘／工作量另见 [[reference/components/workable]]。[组件](../../../raw/current/scripts/components/gravedigger.lua#L1-L13)、[宿主安装](../../../raw/current/scripts/prefabs/graveurn.lua#L193-L194)
- 优先使用：需要骨灰坛的存墓与重新部署时沿用完整宿主；仅添加此组件不会自动获得记录、资源、技能资格或部署能力。[原版工具回调](../../../raw/current/scripts/prefabs/graveurn.lua#L119)
- 重要副作用：`GRAVEDIG` 先执行工具 `OnUsed`，再执行目标 `DigUp`；目标失败不能回滚工具已产生的变化。[动作顺序](../../../raw/current/scripts/actions.lua#L6231-L6245)
- 修改边界：原骨灰坛在主模拟添加组件；动作提示还要求目标 `gravediggable` 标签、操作者 `gravedigger_user` 标签。实际动作函数不重复这些标签检查。[工厂](../../../raw/current/scripts/prefabs/graveurn.lua#L187-L194)、[动作候选](../../../raw/current/scripts/componentactions.lua#L1334-L1338)

## 关键接口

| 接口／字段 | 参数或约定 | 行为与证据 |
|---|---|---|
| `onused` | 可选 `fn(tool, user, target)` | 直接赋值，没有 setter；组件构造未初始化该字段。[实现](../../../raw/current/scripts/components/gravedigger.lua#L1-L11) |
| `OnUsed(user, target)` | 无显式返回 | 有回调就调用，不验证资格、不查目标组件，也不消费工具；回调返回值不参与动作判定。[分发](../../../raw/current/scripts/components/gravedigger.lua#L7-L11)、[消费者](../../../raw/current/scripts/actions.lua#L6231-L6245) |

组件没有任务、存读或移除清理钩子。工具的持久数据与转换由 prefab 负责；不能用移除此组件代替清除已存墓碑。[组件全文](../../../raw/current/scripts/components/gravedigger.lua#L1-L13)

## 典型接入

### 原版骨灰坛：空坛收墓，存记录后转为可部署物

这是原版接入说明，不强造缺少工厂上下文的新工具 demo：

1. `graveurn` 公共构造创建预览 netvar；服务端安装 gravedigger、inventoryitem 和宿主存读。空坛用 `graveurn_empty` 图标，注册名仍是 `graveurn`。[公共与服务端构造](../../../raw/current/scripts/prefabs/graveurn.lua#L164-L205)、[注册](../../../raw/current/scripts/prefabs/graveurn.lua#L281)
2. 温蒂 `wendy_gravestone_1` 技能增删操作者 `gravedigger_user` 标签；这与能否制作工具是不同条件。物品对目标及装备右键两种候选路径均检查该标签。[技能启停](../../../raw/current/scripts/prefabs/skilltree_wendy.lua#L217)、[停用](../../../raw/current/scripts/prefabs/skilltree_wendy.lua#L222)、[物品路径](../../../raw/current/scripts/componentactions.lua#L1334-L1338)、[装备路径](../../../raw/current/scripts/componentactions.lua#L2360-L2364)
3. 玩家服务端和客户端将动作映射到 `graveurn_in`；服务端在动画第 58 帧才执行缓冲动作。[服务端映射](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1724)、[执行帧](../../../raw/current/scripts/stategraphs/SGwilson.lua#L8445-L8446)、[客户端映射](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L971)
4. 工具回调先把装饰阶段重置为 1，按原随机逻辑处理花瓣，再取目标 `GetSaveRecord()`。`SetGraveSaveData` 保存记录、换图标、移除 gravedigger、添加 deployable 并刷新预览数据；随后目标才走自己的拆除回调。[取记录前处理](../../../raw/current/scripts/prefabs/graveurn.lua#L119)、[工具转换](../../../raw/current/scripts/prefabs/graveurn.lua#L98)、[目标拆除](../../../raw/current/scripts/prefabs/gravestone.lua#L67-L84)
5. 部署时从 `_grave_record` 经 `SpawnSaveRecord` 恢复墓碑、设坐标和动画，再移除骨灰坛；没有记录时回退生成 gravestone。宿主保存 `grave_record`，加载时重新执行工具转换配置。[部署](../../../raw/current/scripts/prefabs/graveurn.lua#L60)、[保存与恢复](../../../raw/current/scripts/prefabs/graveurn.lua#L150-L160)

扩展这条链时保留原回调顺序、存档与资源；需要拒绝迁墓的条件应在工具回调执行前检查，具体最小接入见 [[reference/components/gravediggable#典型接入]]。

## 常见陷阱与限制

- **在工具 `onused` 返回 false 拒绝**：动作忽略其返回值，仍调用目标 DigUp；应在完整动作开始前处理拒绝。[动作实现](../../../raw/current/scripts/actions.lua#L6231-L6245)
- **目标返回失败就没有搬迁副作用**：原工具可能已降装饰阶段、保存记录并移除了自身 gravedigger，无法仅凭最终 false 判断状态未变；不要简单重试或另行再复制一次记录。[工具转换](../../../raw/current/scripts/prefabs/graveurn.lua#L98)、[原回调](../../../raw/current/scripts/prefabs/graveurn.lua#L119)
- **把普通铲子挂组件即可完整迁墓**：本组件不生成存档记录或可部署物，也不添加操作者标签；GRAVEDIG 与普通 DIG／workable 是不同入口，实际效果必须由工具和目标协作。

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 漂移文件 actions.lua、componentactions.lua、stategraphs/SGwilson.lua、stategraphs/SGwilson_client.lua 对照 build-26148036 更新报告（R-01、R-04、SB-17）逐条核验，锚点重定位 7 处、结论修订 0 处（GRAVEDIG 执行链、双路候选与 graveurn_in SG 映射／执行帧未变，均仅整体位移；SGwilson 漂移为 Rifts 8 新增内容不触及本页）；未覆盖：运行验证。

- 源码审阅：2026-10-22，Steam buildid 25643519；覆盖完整组件、动作先后／返回、双方候选、技能标签、玩家 SG 映射及执行帧、骨灰坛注册／保存转换／部署与墓碑拆除。未完整审校装饰掉落概率、皮肤权限或整个技能树。
- 结构检查：本批定向检查元数据、大小、链接和源码锚点；本页只有原版接入说明，无新增完整 Lua。
- 游戏运行：未运行；移墓中断、旧档恢复、联机和其他模组包装顺序待实测。
