---
title: "sittable"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "维护座椅占用与可坐标签，衔接 SITON、玩家坐姿状态和弹离通知。"
aliases: ["可坐家具", "座椅占用", "让玩家离开椅子", "sittable"]
tags: [component, building, actions, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/sittable.lua", "componentactions.lua", "actions.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "prefabs/furniture_chairs.lua", "components/wisecracker.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# sittable

## 快速判断

- 职责与适用：挂在座椅上，保存一个 `occupier`、维护 `cansit` 及占用事件；人物位置、坐姿动画和上下椅子由 SG 协作。[组件](../../../raw/current/scripts/components/sittable.lua#L1-L69) [玩家入座](../../../raw/current/scripts/stategraphs/SGwilson.lua#L23597-L23628)
- 优先使用：判断座位用 `IsOccupied()`／`IsOccupiedBy(sitter)`，要求原版坐者离开用 `EjectOccupier()`；正常入座经 `ACTIONS.SITON`，不要只写占用引用。[接口](../../../raw/current/scripts/components/sittable.lua#L46-L58) [动作执行](../../../raw/current/scripts/actions.lua#L6009-L6020)
- 重要副作用：占用／释放改变 `cansit`，向**椅子**发送 `becomeunsittable`／`becomesittable`；被占用椅子着火时向**坐者**发送 `sittableonfire`，数据为椅子。[属性与构造](../../../raw/current/scripts/components/sittable.lua#L1-L21) [设置占用](../../../raw/current/scripts/components/sittable.lua#L23-L44)
- 修改边界：原版家具在服务端门禁后挂载；动作采集按标签，客户端 SG 预览，实际占位由服务端动作／SG 完成。组件不提供坐姿资源或占位存档。[挂载](../../../raw/current/scripts/prefabs/furniture_chairs.lua#L287-L309) [采集](../../../raw/current/scripts/componentactions.lua#L788-L792) [客户端预览](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L6913-L6938)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `occupier` | 实体引用；初始未赋值 | 属性回调按非 nil／nil 移除／添加 `cansit`；直接赋值只触发标签回调，不执行 setter 的监听／事件协作。[定义](../../../raw/current/scripts/components/sittable.lua#L1-L21) |
| `SetOccupier(occupier)` | 实体或 nil；无返回 | 引用变化才执行：拆旧坐者 onremove 与椅子 onignite 监听，更新引用，按需重新监听并发事件；不检查新坐者合法性、距离或其他占用冲突。[实现](../../../raw/current/scripts/components/sittable.lua#L27-L44) |
| `IsOccupied()`／`IsOccupiedBy(occupier)` | 布尔值 | 分别判断引用非 nil、与非 nil 参数相同；不检查实体有效性。[查询](../../../raw/current/scripts/components/sittable.lua#L46-L52) |
| `EjectOccupier()` | 无参数、无返回 | 有坐者时只对椅子发 `becomeunsittable`，不直接清占用；消费者需退出并清引用。[通知](../../../raw/current/scripts/components/sittable.lua#L54-L58) [SG 收尾](../../../raw/current/scripts/stategraphs/SGwilson.lua#L23997-L24018) |
| `OnRemoveFromEntity()` | 移组件钩子 | 拆占用相关监听、删除 `cansit`、向椅子发不可坐事件；不主动改坐者 SG，也不把引用设 nil。[清理](../../../raw/current/scripts/components/sittable.lua#L60-L67) |

## 典型接入

### 新家具复用原版座椅的组装与坐姿协作

以下为原版构造参照，不是只加一个组件即可运行的新家具工程。`furniture_chairs.lua` 的 `AddChair` 实际注册 `wood_chair`／`wood_stool`、石制座椅及摇椅，名称不是源文件名。[工厂与注册](../../../raw/current/scripts/prefabs/furniture_chairs.lua#L341-L359)

| 层次 | 需要保留的协作 | 证据入口 |
|---|---|---|
| 家具实体 | 网络构造、朝向、碰撞、动画与 `faced_chair` 或摇椅标签；服务端添加 sittable，锤击回调弹离坐者 | [共同构造](../../../raw/current/scripts/prefabs/furniture_chairs.lua#L249-L309) [workable 配置](../../../raw/current/scripts/prefabs/furniture_chairs.lua#L314-L332) [锤击通知](../../../raw/current/scripts/prefabs/furniture_chairs.lua#L27-L33) |
| 交互入口 | 无 `fire` 且有 `cansit` 才收集 SITON；动作仅在 doer SG 为 `start_sitting` 且目标空位时占座 | [采集](../../../raw/current/scripts/componentactions.lua#L788-L792) [执行](../../../raw/current/scripts/actions.lua#L6009-L6020) |
| 玩家服务端 SG | 执行动作后校验占座，播放坐下；持续状态监听椅子移除／不可坐，退出时按自身占用关系释放 | [入座](../../../raw/current/scripts/stategraphs/SGwilson.lua#L23597-L23654) [占位及监听](../../../raw/current/scripts/stategraphs/SGwilson.lua#L23773-L23807) [退出](../../../raw/current/scripts/stategraphs/SGwilson.lua#L23997-L24018) |
| 客户端 SG | 原版 SITON 绑定 `start_sitting`，预览动作并等待服务端；不在客户端自行调用 SetOccupier | [路由](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L932) [预览／等待](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L6913-L6949) |

修改已有椅子的强制离座触发点时，在原有服务端宿主回调调用 `EjectOccupier()` 即可复用上述通知链；若替换 `workable` 回调，保留原回调，防止丢失锤击动画与拆除行为。新家具还需按实际坐姿 SG 核验自身资源、朝向和物理尺寸，不把任意物品加 `cansit` 当作完成接入。[原版 OnHit／OnHammered](../../../raw/current/scripts/prefabs/furniture_chairs.lua#L27-L43)

### 占用、着火与移除分别由谁收尾

- `SetOccupier` 会监听当前坐者的 `onremove`，坐者被移除时调用 `SetOccupier(nil)`；更换坐者先拆旧监听。[设置占用](../../../raw/current/scripts/components/sittable.lua#L27-L44)
- 椅子有 burnable 且正被占用时才监听 `onignite`；该事件只转成坐者的 `sittableonfire`。已核验的 wisecracker 消费者延迟检查后说台词，不应据事件名称推断自动跳下。[转发](../../../raw/current/scripts/components/sittable.lua#L23-L38) [台词消费者](../../../raw/current/scripts/components/wisecracker.lua#L321-L333)
- 原版家具烧毁时移除 sittable；其钩子通知玩家 SG，SG 校验组件是否仍在后清理自身坐姿。家具存档只记烧毁状态，本组件无 OnSave／OnLoad，占用引用不由它持久化。[烧毁／存读](../../../raw/current/scripts/prefabs/furniture_chairs.lua#L140-L171) [组件清理](../../../raw/current/scripts/components/sittable.lua#L60-L69)

## 常见陷阱与限制

- **调用 Eject 后仍读到坐者：** 它只是通知；自定义坐者若没有原版 SG 的监听与退出释放逻辑，占用不会自动清空。自己设计的坐者必须补齐收尾，不把调用等同同步清座。[通知](../../../raw/current/scripts/components/sittable.lua#L54-L58) [监听](../../../raw/current/scripts/stategraphs/SGwilson.lua#L23792-L23801)
- **裸 SetOccupier 覆盖他人或只占座不坐下：** 方法不防冲突，也不移动／播放动画；原版 ACTIONS 和 SG 分别做空位及占有者校验，正常玩家操作保留这条链。[setter](../../../raw/current/scripts/components/sittable.lua#L27-L44) [动作](../../../raw/current/scripts/actions.lua#L6009-L6020) [SG 检查](../../../raw/current/scripts/stategraphs/SGwilson.lua#L23773-L23782)
- **把 becomesittable 当成添加组件后必能捕获的通知：** 构造当场发事件；原版摇椅监听是在挂载之后添加，监听不到构造这一次。初始化逻辑需自行执行，不能只等首次事件。[构造](../../../raw/current/scripts/components/sittable.lua#L9-L21) [宿主监听顺序](../../../raw/current/scripts/prefabs/furniture_chairs.lua#L309-L325)
- **移组件后手动补回 cansit：** 清理故意删标签并通知不可坐；只补标签不能恢复组件、动作执行和坐姿收尾。[移除钩子](../../../raw/current/scripts/components/sittable.lua#L60-L67) [动作执行](../../../raw/current/scripts/actions.lua#L6009-L6020)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 15 处（actions／componentactions／SGwilson／SGwilson_client／wisecracker 行号迁移，区间文本逐段比对一致）；结论修订：无，仅锚点迁移。
- 依据：枢纽 diff 审阅（actions.lua／componentactions.lua hunk 地图）＋变更清单命中 actions.lua、componentactions.lua、stategraphs/SGwilson.lua、stategraphs/SGwilson_client.lua、components/wisecracker.lua。

- 源码审阅：2026-10-23，`steam-buildid:25643519`；覆盖完整 sittable、SITON 采集／执行、玩家双端入座与关键占位退出、家具工厂实际注册／服务端挂载／锤击／烧毁／存读，以及着火台词消费者。未审校寄居蟹全部坐姿状态、各椅子动画资源或所有中断状态。
- 游戏运行：未运行；自定义家具与自定义坐者未实测。
- 结构检查：本页格式、链接与源码锚点严格检查通过；仅原版组装参照，无新增完整 Lua 示例。
