---
title: "container_transform"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "在 RUMMAGE 打开前按回调条件把被翻找的实体替换成另一个容器实体。"
aliases: ["翻找前替换容器", "打开时变身容器", "RUMMAGE 转换实体", "container_transform"]
tags: [component, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/container_transform.lua", "actions.lua", "componentactions.lua", "entityscript.lua", "prefabs/wx78_possessedbody.lua", "prefabs/wx78_backupbody.lua", "brains/wx78_possessedbodybrain.lua", "components/playercontroller.lua", "util.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# container_transform

## 快速判断

- 职责与适用：给服务端实体加一个“翻找前转换”钩子。玩家对其执行 RUMMAGE 时，动作先问 `CanTransform`，通过后调 `TryTransformToContainer`，把动作目标换成回调返回的实体，再继续走普通容器打开流程。[入口](../../../raw/current/scripts/actions.lua#L1076-L1085)
- 优先使用：需要“同一个交互入口，打开时换成另一个容器/实体”（原版是机器人本体变回底盘）时使用；两个回调必须成对设置。[设置](../../../raw/current/scripts/components/container_transform.lua#L7-L13)
- 重要副作用：转换在动作执行中同步发生，原实体可被回调直接删除；返回的实体被当作动作目标继续打开容器，组件不校验它是否带 container。[后续](../../../raw/current/scripts/actions.lua#L1085-L1105)
- 修改边界：只有服务端实例（宿主在 master 段添加）；回调是普通函数，不存档、不复制，读档或重建实体后必须重设。[pristine 与端](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L567-L569) [添加](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L693-L695)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetCanTransform(fn)`／`cantransform` | `fn(inst, doer)` → `success, reason`；传 `nil` 清空 | `CanTransform(doer)` 经 `FunctionOrValue` 调用，返回值原样作为动作成败；未设置时返回 nil。[L7-L9](../../../raw/current/scripts/components/container_transform.lua#L7-L9) [L29-L32](../../../raw/current/scripts/components/container_transform.lua#L29-L32) [FunctionOrValue](../../../raw/current/scripts/util.lua#L1718-L1724) |
| `SetOnTransform(fn)`／`ontransform` | `fn(inst)` → 实体或 nil；传 `nil` 清空 | `TryTransformToContainer()` 返回 `ontransform ~= nil and ontransform(inst)`，未设置时得到 `false` 而不是 nil。[L11-L13](../../../raw/current/scripts/components/container_transform.lua#L11-L13) [L34-L36](../../../raw/current/scripts/components/container_transform.lua#L34-L36) |
| `IsTryingToOpenMe(doer)` | boolean | 先看 doer 的当前缓冲动作；没有再看 `playercontroller:GetRemoteInteraction()`。两路都要求目标就是宿主且动作是 `ACTIONS.RUMMAGE`。参数是“想打开的人”，不是宿主。[L15-L25](../../../raw/current/scripts/components/container_transform.lua#L15-L25) |
| 实例字段 | `inst`／`cantransform`／`ontransform` | 构造只设这三个字段；无 netvar、无存档、无移除钩子。[L1-L5](../../../raw/current/scripts/components/container_transform.lua#L1-L5) |

### 谁写、谁读

- 写：源码中只有宿主 prefab 在 master postinit 调两个 Set，没有其他写入点。[wx78_possessedbody](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L693-L695)
- 读：服务端 `ACTIONS.RUMMAGE.fn` 每次翻找读一次；`wx78_possessedbodybrain` 用 `IsTryingToOpenMe` 让本体停下并面向想打开它的玩家。[brain](../../../raw/current/scripts/brains/wx78_possessedbodybrain.lua#L31-L43)
- 客户端动作：`componentactions` 只按 tag/replica 判断，不调用组件方法；动作可见性来自服务端 `AddComponent` 时登记的组件 id，经 `actioncomponents` net 数组复制，所以客户端没有组件实例也能看到 RUMMAGE。[收集器](../../../raw/current/scripts/componentactions.lua#L314-L320) [登记](../../../raw/current/scripts/componentactions.lua#L3182-L3204) [复制](../../../raw/current/scripts/entityscript.lua#L92-L99)

## 典型接入

### 原版参照：wx78_possessedbody 被翻找时变回底盘容器

- 目标文件与加载：宿主 prefab 为 `prefabs/wx78_possessedbody.lua`，注册名 `wx78_possessedbody`；组件在 `SetPristine` 与服务端分支之后添加，客户端没有该组件。[注册](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L731) [添加](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L693-L695)
- 端与资源：判定与生成都在服务端；转换生成另一个已注册 prefab `wx78_backupbody`，其容器以 `wx78_backupbody` 作为 widget。[生成](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L181-L213) [容器](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L945-L948)
- 生效条件：`CanTransformToContainer` 要求 doer 是机器人（`wx78_classified`）、宿主属于该玩家（linkeditem 的 owner userid 一致）且不处于 busy；失败分别返回 `NOTAROBOT`／`NOTMYBACKUP`，busy 时只返回 false、没有原因。[L396-L412](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L396-L412)
- 转换结果：`TransformToContainer` 调 `TryToReplaceWithBackupBody(true)`，后者生成 backupbody、搬移属性、删除本体；随后把 backupbody 置为“被翻找的容器态”并返回它，RUMMAGE 接着打开它的容器。[L414-L424](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L414-L424) [容器态](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L813-L826)
- 单位与撤销：组件不涉及数量或时长，RUMMAGE 动作本身也没有 distance 字段，实际可达距离由动作系统默认值决定。撤销是关闭容器：backupbody 的 `onclosefn` 在关机动画结束后调用 `RevertToPossessedBody` 重新生成本体，本组件由本体 prefab 重新挂上。[关闭回调](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L602-L615) [恢复](../../../raw/current/scripts/prefabs/wx78_backupbody.lua#L581-L583)

## 常见陷阱与限制

- **只设 `OnTransform` 会被静默拒绝**：`CanTransform` 默认返回 nil，RUMMAGE 在 `if not success` 处直接 `return false`（reason 也是 nil，没有失败台词）；`SetCanTransform` 必须显式设为 `true` 或函数。[L29-L32](../../../raw/current/scripts/components/container_transform.lua#L29-L32) [消费](../../../raw/current/scripts/actions.lua#L1076-L1079)
- **`OnTransform` 返回 `false` 会在动作里报错**：未设置时 `TryTransformToContainer` 返回 `false`，RUMMAGE 用 `targ ~= nil` 判真后访问 `targ.components`，对 boolean 取字段会抛 Lua 错误；取消请返回 `nil`，成功必须返回带 container 的实体。[L34-L36](../../../raw/current/scripts/components/container_transform.lua#L34-L36) [L1082](../../../raw/current/scripts/actions.lua#L1085)
- **先转换再打开**：RUMMAGE 在检查宿主自身容器之前就做转换，所以同时带 `container` 和本组件的实体会走转换分支，打不开自己的容器。[顺序](../../../raw/current/scripts/actions.lua#L1076-L1085)
- **不是存档状态**：两个回调是闭包，`OnSave`／`OnLoad` 不管它们；重建实体或读档后不重设就退化为不可打开。[字段](../../../raw/current/scripts/components/container_transform.lua#L1-L5)
- **`IsTryingToOpenMe` 的远程兜底有前提**：`GetRemoteInteraction` 只在客户端上报交互目标后才有值；RUMMAGE 只有在目标是 `locomotor` 实体或操作者带 `wx78_classified` 时才上报，所以没有 locomotor 的宿主只对 WX-78 玩家有兜底，其他玩家只剩缓冲动作那一路。[上报条件](../../../raw/current/scripts/components/playercontroller.lua#L5885-L5900) [上报与读取](../../../raw/current/scripts/components/playercontroller.lua#L5860-L5871)
- **没有清理钩子**：组件不监听 onremove，也没有 `OnRemoveFromEntity`；需要随实体销毁清理的外部状态由宿主自己做。

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的漂移文件（actions.lua、componentactions.lua、entityscript.lua、wx78_possessedbody.lua、wx78_possessedbodybrain.lua、playercontroller.lua、util.lua）对照 build-26148036 更新报告（R-01/R-04/R-08/P-150/SB-31）逐条核验：RUMMAGE fn、动作收集器与登记、GetRemoteInteraction 上报条件及 wx78 宿主/brain 相关行均逐行未变（P-150 仅 :599 处 SetDiet 等价替换、SB-31 仅 :362 电击排除表加 shadowboss），actions.lua／componentactions.lua 的施法与拾取类改动不触及本页结论；锚点重定位 6 处、结论修订 0 处；未覆盖：运行验证。

- 源码审阅：2026-10-21，Steam buildid 25643519；通读 `components/container_transform.lua`（38 行），核验 RUMMAGE 消费路径、客户端动作收集、唯一宿主与唯一 brain 消费者；除组件自身外，源码中 9 处引用分布在 actions.lua、componentactions.lua、wx78_possessedbody.lua、wx78_possessedbodybrain.lua 四个文件。
- 游戏运行：未运行；机器人转换、关闭容器恢复以及多人同时翻找的实际时序仍需游戏验证。
- 结构检查：`validate_page.py` 通过（0 issue；本页 63 行／8989 字节）；`validate_source_links.py --strict` 共 34 条源码引用、0 error／0 warning；本页无 Lua 示例，未生成临时 `.lua`。
