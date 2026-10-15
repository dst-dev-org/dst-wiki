---
title: "attunable"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "为场景实体建立与玩家的调谐关系，处理同类替换、专属网络代理和离线恢复。"
aliases: ["attunable", "物品调谐", "建筑绑定玩家", "复活雕像绑定", "调谐成本", "离线绑定恢复"]
tags: [component, building, network-sync, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/attunable.lua", "components/attuner.lua", "prefabs/attunable_classified.lua", "prefabs/resurrectionstatue.lua", "prefabs/wendy_resurrectiongrave.lua", "prefabs/player_common.lua", "componentactions.lua", "actions.lua", "constants.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# attunable

## 快速判断

- 职责与适用：挂在被绑定的实体上，保存多个玩家的在线／离线调谐关系；玩家侧查询与代理索引由 [[reference/components/attuner]] 承担。[构造](../../../raw/current/scripts/components/attunable.lua#L1-L18)
- 优先使用：服务端 `LinkToPlayer(player)`／`UnlinkFromPlayer(player)`；同类互斥用 `SetAttunableTag(tag)`，业务效果由宿主回调实现。[链接与解绑](../../../raw/current/scripts/components/attunable.lua#L93-L135)
- 重要副作用：链接创建仅面向该玩家的 `attunable_classified`，推送玩家 `attuned`，旧同类来源收到后解绑；新旧关系短暂共存。[链接顺序](../../../raw/current/scripts/components/attunable.lua#L105-L118) [旧源监听](../../../raw/current/scripts/components/attunable.lua#L20-L30)
- 修改边界：要求非空 `userid` 和玩家 `attuner`；原版被调谐宿主在服务端添加本组件。它提供绑定，不负责复活、传送或通用所有权权限。[门禁](../../../raw/current/scripts/components/attunable.lua#L86-L91) [宿主端边界](../../../raw/current/scripts/prefabs/resurrectionstatue.lua#L141-L172) [复活动作消费者](../../../raw/current/scripts/actions.lua#L4117-L4129)

## 关键接口

| 接口／字段 | 参数、返回与条件 | 作用与证据 |
|---|---|---|
| `CanAttune(player)` | 布尔；玩家非空 userid、有 attuner、尚未在线链接本源 | 不调用成本函数，不等同最终支付成功。[实现](../../../raw/current/scripts/components/attunable.lua#L86-L91) |
| `LinkToPlayer(player, isloading?)` | 成功 `true`；门禁失败 `false`；成本失败 `false, reason` | 非加载时先执行成本，再创建代理、发 attuned、挂监听、调 onlink。成本回调须返回真值才继续。[实现](../../../raw/current/scripts/components/attunable.lua#L93-L118) |
| `UnlinkFromPlayer(player, isloading?)` | 无返回；非在线链接直接退出 | 删除代理及在线项，移除该玩家的监听，再调 onunlink；不删除离线 userid 项。[实现](../../../raw/current/scripts/components/attunable.lua#L121-L135) |
| `SetAttunableTag(tag)`／`GetAttunableTag()` | 默认 nil | tag 在**新建代理时**复制；setter 仅改字段，不能同步修改已有代理。[设置](../../../raw/current/scripts/components/attunable.lua#L62-L68) [复制](../../../raw/current/scripts/components/attunable.lua#L105-L109) |
| `SetOnAttuneCostFn(fn)` | `fn(source, player) → success, reason?` | `isloading` 为真时跳过；适合做一次性门禁／付费，不用于恢复后的持续状态重建。[调用](../../../raw/current/scripts/components/attunable.lua#L98-L103) |
| `SetOnLinkFn(fn)`／`SetOnUnlinkFn(fn)` | `fn(source, player, isloading)` | 仍会在加载／转移时调用，应区分状态重建与音效／收费等一次性效果。[设置](../../../raw/current/scripts/components/attunable.lua#L74-L80) [调用](../../../raw/current/scripts/components/attunable.lua#L115-L134) |
| `IsAttuned(player)` | 在线项是否存在 | 不查 `attuned_userids`，不能拿离线玩家 userid 字符串代替实体调用。[实现](../../../raw/current/scripts/components/attunable.lua#L82-L84) |

### 同类互斥的准确范围

旧源在玩家 `attuned` 事件中按以下条件解绑：新旧 **prefab 相同**，或双方 tag 非空且相同，或新 tag 等于 `EQUIVALENT_ATTUNABLE_TAGS[旧 tag]`。不同 prefab 且无匹配 tag 可以共存；并非任意实体标签都全局唯一。当前等价表双向关联 `remoteresurrector` 与 `gravestoneresurrector`。[比较逻辑](../../../raw/current/scripts/components/attunable.lua#L20-L30) [等价表](../../../raw/current/scripts/constants.lua#L2954-L2960)

### 离线、读档与移除

- 玩家 `onremove` 只把 userid 转入离线表并清在线映射，**不等同调用 Unlink／onunlink**。之后世界 `ms_playerjoined` 命中 userid，移出离线表并 `LinkToPlayer(player, true)`。[回调与世界监听](../../../raw/current/scripts/components/attunable.lua#L33-L45)
- `OnSave` 将在线玩家 userid 与离线 userid 写入 `links`；`OnLoad` 对已在 AllPlayers 的玩家免成本重连，其余暂存离线表。玩家公共激活流程先推世界 joined，再注册复活调谐表现监听，避免恢复时重复播放效果。[存读](../../../raw/current/scripts/components/attunable.lua#L137-L164) [玩家顺序](../../../raw/current/scripts/prefabs/player_common.lua#L852-L866)
- `OnRemoveEntity`／`OnRemoveFromEntity` 共用清理：移除 GUID 识别 tag、世界监听，对**在线项**逐个解绑；没有遍历离线 userid 调用 onunlink。[清理](../../../raw/current/scripts/components/attunable.lua#L48-L60)

## 典型接入

### 以原版复活建筑为调谐接入参照

用于新建筑设计前判断职责或修改已有调谐机制。以下是原版接入路径，无需新增美术文件；不是独立复活工程或可直接复制的 Prefab 骨架。

1. `resurrectionstatue.lua` 实际注册 `resurrectionstatue`。工厂服务端添加 attunable，tag 为 `remoteresurrector`，装配成本／链接／解绑回调；烧毁时移除组件，复活消费事件由宿主处理。[注册](../../../raw/current/scripts/prefabs/resurrectionstatue.lua#L181-L182) [装配与消费](../../../raw/current/scripts/prefabs/resurrectionstatue.lua#L165-L176) [烧毁](../../../raw/current/scripts/prefabs/resurrectionstatue.lua#L37-L40)
2. 玩家对未绑定目标收集 `ATTUNE`，服务端动作调用 LinkToPlayer；玩家双端 SG 将动作接入 dolongaction。UI 有动作不代表成本足够，最终由服务端回调判断。[动作收集](../../../raw/current/scripts/componentactions.lua#L168-L173) [动作执行](../../../raw/current/scripts/actions.lua#L4101-L4107) [服务端 SG](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1439) [客户端 SG](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L721)
3. 原版成本检查 health 并扣生命；普通链接只驱动动画／音效。新宿主应明确自己的成本与业务，不把原版健康值逻辑当组件内置规则。[成本](../../../raw/current/scripts/prefabs/resurrectionstatue.lua#L61-L73) [链接与解绑表现](../../../raw/current/scripts/prefabs/resurrectionstatue.lua#L75-L89)
4. `wendy_resurrectiongrave` 用 `gravestoneresurrector`，回调还重建火焰显示、渐变灯光和玩家名称；加载时仍需要这些持续表现，仅略过一次性动画／音效。[另一宿主回调](../../../raw/current/scripts/prefabs/wendy_resurrectiongrave.lua#L65-L85) [组件装配](../../../raw/current/scripts/prefabs/wendy_resurrectiongrave.lua#L174-L178) [注册](../../../raw/current/scripts/prefabs/wendy_resurrectiongrave.lua#L251-L253)

需要读玩家当前来源或维护自定义状态，直接见 [[reference/components/attuner#典型接入]]；生命扣除接口见 [[reference/components/health]]。调谐之后的远程复活由独立动作向玩家推 `respawnfromghost`，不可仅添加 tag 就宣称完成复活实现。[消费入口](../../../raw/current/scripts/actions.lua#L4117-L4129)

## 常见陷阱与限制

- **PostInit 修改回调被建造覆盖：** 两个原版建筑的 `onbuilt` 为免费自动绑定，临时清三个回调，完成后恢复原始函数。修改这些回调要同时核验建造入口和读档入口；不能只在 PrefabPostInit 改一次就保证永久生效。[雕像建造](../../../raw/current/scripts/prefabs/resurrectionstatue.lua#L97-L114) [墓碑建造](../../../raw/current/scripts/prefabs/wendy_resurrectiongrave.lua#L93-L116)
- **滥用 isloading 绕过付费：** `LinkToPlayer(player, true)` 会跳过成本；保留给恢复／转移，不把它作为普通交互默认参数。回调是否播放表现也由该参数控制。[门禁与成本](../../../raw/current/scripts/components/attunable.lua#L93-L103) [宿主加载判断](../../../raw/current/scripts/prefabs/wendy_resurrectiongrave.lua#L65-L85)
- **把改 tag 当迁移：** 既有代理不随 setter 改 tag，旧源匹配却读取当前字段；应在首次绑定前配置类别，需要改存量关系时另行设计解绑／重连和离线处理。[setter](../../../raw/current/scripts/components/attunable.lua#L66-L68) [代理标签](../../../raw/current/scripts/components/attunable.lua#L105-L109)
- **把目标实体当客户端数据源：** 远处建筑可能不可见，本地玩家仍有 classified；客户端类别判断应走玩家 attuner／proxy，不在客户端调用 LinkToPlayer。[代理约定](../../../raw/current/scripts/components/attuner.lua#L58-L65) [专属网络目标](../../../raw/current/scripts/prefabs/attunable_classified.lua#L17-L22)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：actions.lua（R-01）、componentactions.lua（R-04）、constants.lua（R-06）、prefabs/player_common.lua（P-83）、stategraphs/SGwilson.lua（SB-17）与 SGwilson_client.lua 对照 build-26148036 更新报告逐条核验，锚点重定位 6 处、结论修订 0 处：ATTUNE 候选（componentactions.lua:168-173）与执行、双端 SG `dolongaction` 接入、EQUIVALENT_ATTUNABLE_TAGS、ms_playerjoined 注册顺序（player_common.lua:852-866）均逐行无变化（P-83 仅删 worm_boss 消化音效 netvar，与本页无关）；未覆盖：运行验证。

- 源码审阅：2026-10-23，`steam-buildid:25643519`；核验本组件完整实现、玩家查询与 classified、两种宿主的注册／构造／成本／建造覆盖／加载表现、ATTUNE 动作与双端 SG，以及玩家加入顺序。证据见上文；未审阅完整复活状态机或跨分片存档协议。
- 游戏运行：未运行；未验证同类建筑并发调谐、断线重连和复活效果。
- 结构检查：本次定向页面格式／链接与源码引用 strict 检查通过；本页无独立 Lua demo。
