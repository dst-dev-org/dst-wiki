---
title: "bottler"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "为物品接入装瓶动作，区分目标资格、回调返回与宿主负责的消耗和产物。"
aliases: ["装瓶工具", "空瓶捕获黏液", "装瓶回调", "bottler"]
tags: [component, item, callback, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/bottler.lua", "componentactions.lua", "actions.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "prefabs/messagebottle.lua", "prefabs/gelblob.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# bottler

## 快速判断

- 职责与适用：挂在装瓶工具上，将有效且带 `canbebottled` 标签的目标交给宿主回调；不负责捕获、产物或资源消耗。[bottler.lua:1](../../../raw/current/scripts/components/bottler.lua#L1 [Bottle](../../../raw/current/scripts/components/bottler.lua#L10-L15)
- 优先使用：用 `SetOnBottleFn(fn)` 配置 `(tool, target, doer)`；原版空瓶 `messagebottleempty` 是可复用的实际参照。[回调设置](../../../raw/current/scripts/components/bottler.lua#L6-L8) [宿主挂载](../../../raw/current/scripts/prefabs/messagebottle.lua#L255-L262)
- 重要副作用：组件直接转发回调返回值；删除目标、拆分瓶堆和生成装满的瓶子均发生在原版回调里。[原版 OnBottle](../../../raw/current/scripts/prefabs/messagebottle.lua#L212-L228)
- 修改边界：原版只在服务端挂组件；目标标签用于动作发现，不能代替回调中对具体目标和资源条件的核验。[端边界](../../../raw/current/scripts/prefabs/messagebottle.lua#L249-L259) [动作发现](../../../raw/current/scripts/componentactions.lua#L1076-L1080)

## 关键接口

| 接口／入口 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetOnBottleFn(fn)` | 回调或 nil | 覆盖单个 `onbottlefn` 槽；不叠加、不保存旧回调。[bottler.lua:6-8](../../../raw/current/scripts/components/bottler.lua#L6-L8) |
| `Bottle(target, doer)` | 回调的原始返回值；门禁不通过为 false | 只检查回调存在、目标存在且有效、目标标签；不检查 doer、距离、数量或冷却。[bottler.lua:10-15](../../../raw/current/scripts/components/bottler.lua#L10-L15) |
| `ACTIONS.BOTTLE` | `act.invobject` 是工具 | 动作调用物品组件，再原样返回结果；玩家服务端与客户端均选 `dolongaction`。[actions.lua:6054-6058](../../../raw/current/scripts/actions.lua#L6054-L6058) [服务端 SG](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1699) [客户端 SG](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L944) |
| `canbebottled` | 目标标签 | 工具的动作采集据此提供 BOTTLE；原版 `gelblob_small_fx` 在 `SetPristine` 前添加，特定释放流程会移除。[componentactions.lua:1076-1080](../../../raw/current/scripts/componentactions.lua#L1076-L1080) [添加](../../../raw/current/scripts/prefabs/gelblob.lua#L854) [移除](../../../raw/current/scripts/prefabs/gelblob.lua#L815-L819) |

## 典型接入

### 空瓶装入小黏液的原版组装路径

需要新建类似工具时，以此链条确定职责；本节是原版接入说明，不是可直接复制的新 Prefab 工程：

1. `messagebottle.lua` 将 `messagebottleempty` 注册到 `emptybottlefn`，声明产物依赖 `gelblob_bottle`；沿用瓶子动画、物品物理、漂浮与物品栏组件，服务端挂 `bottler` 并设置 `OnBottle`。[依赖](../../../raw/current/scripts/prefabs/messagebottle.lua#L25-L28) [工厂](../../../raw/current/scripts/prefabs/messagebottle.lua#L232-L271) [注册](../../../raw/current/scripts/prefabs/messagebottle.lua#L484-L487)
2. 目标是 `gelblob.lua` 中 `smallfn` 注册出的 `gelblob_small_fx`，其 `canbebottled` 标签提供动作资格；给别的目标加同名标签并不会让原版回调自动识别它。[目标构造](../../../raw/current/scripts/prefabs/gelblob.lua#L835-L862) [目标注册](../../../raw/current/scripts/prefabs/gelblob.lua#L966-L970)
3. 原版回调只接受该 Prefab：先记位置，拆出并删除一个空瓶、删除目标，再生成 `gelblob_bottle`；有 doer 物品栏则交付，否则放到工具原位置。其他 Prefab 返回 false。[完整回调](../../../raw/current/scripts/prefabs/messagebottle.lua#L212-L228)

新工具自行定义支持目标、消耗／产物和失败返回；若包装既有回调，应保留原返回与消耗边界。拆堆与交付细节查 [[reference/components/stackable]]、[[reference/components/inventory]]，可拾取骨架查 [[reference/components/inventoryitem]]。

## 常见陷阱与限制

- **有装瓶动作但执行失败：** 动作采集只看目标标签，原版回调另要求 `target.prefab == "gelblob_small_fx"`；新增目标必须同步对应工具回调。[动作资格](../../../raw/current/scripts/componentactions.lua#L1076-L1080) [目标判断](../../../raw/current/scripts/prefabs/messagebottle.lua#L212-L227)
- **回调不返回结果：** 组件不会补 true，动作收到回调实际返回；自定义回调应明确返回成功或失败。[转发](../../../raw/current/scripts/components/bottler.lua#L10-L15) [动作消费者](../../../raw/current/scripts/actions.lua#L6054-L6058)
- **把装瓶视为事务：** 原版先消耗／删除再生成，并未在该回调中做产物生成失败后的回滚。修改产物前核验注册与依赖，不能宣称失败必然保留原物。[OnBottle 顺序](../../../raw/current/scripts/prefabs/messagebottle.lua#L212-L228)
- **期待组件维护库存或生命周期：** 本组件没有任务、事件、存读或移除钩子，仅存回调；宿主额外添加的状态／监听需自行维护。[完整组件](../../../raw/current/scripts/components/bottler.lua#L1-L17)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 锚点重定位 7 处（actions.lua BOTTLE fn、双端 SG 动作处理器、gelblob.lua 目标标签与注册）；结论修订：无，仅锚点迁移。依据：枢纽 diff 审阅＋变更清单命中 actions.lua、componentactions.lua、SGwilson.lua、SGwilson_client.lua、prefabs/gelblob.lua（命中区域逐行核对均未漂移）。
- 源码审阅：2026-10-23，`steam-buildid:25643519`；覆盖完整组件、BOTTLE 动作发现与执行、双端 SG、空瓶注册／服务端挂载及小黏液目标标签。已核验原版消耗／生成顺序，不将任意带标签目标视为受支持。
- 游戏运行：未运行；自定义目标、产物失败与其他模组组合未实测。
- 结构检查：页面格式／链接与源码锚点严格检查通过；本页为原版接入说明，无新增完整 Lua 示例。
