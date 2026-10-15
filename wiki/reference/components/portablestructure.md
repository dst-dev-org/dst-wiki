---
title: "portablestructure"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "给可搬走的建筑接入拆解回调，区分右键资格、服务端门禁与宿主打包／状态转移。"
aliases: ["建筑打包", "收起便携建筑", "拆解便携帐篷", "便携锅收起", "portablestructure"]
tags: [component, building, callback, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/portablestructure.lua", "componentactions.lua", "actions.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua", "prefabs/portablecookpot.lua", "prefabs/portabletent.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# portablestructure

## 快速判断

- 职责与适用：挂在可拆解的建筑态实体上，将 `Dismantle(doer)` 转给宿主；组件不生成打包物品、不转移内容或耐久，也不删除建筑。[完整组件](../../../raw/current/scripts/components/portablestructure.lua#L1-L16)
- 优先使用：`SetOnDismantleFn(fn)` 配置实际转换；额外玩家资格用实体的 `candismantle`，同时服务于动作发现与执行。不要靠回调返回 false 拒绝动作。[设置与调用](../../../raw/current/scripts/components/portablestructure.lua#L6-L14) [发现](../../../raw/current/scripts/componentactions.lua#L668-L684) [执行](../../../raw/current/scripts/actions.lua#L4569-L4591)
- 重要副作用：正常 DISMANTLE 先拒绝容器占用／非空／烹饪、睡袋占用等，再调用组件并直接返回 true；直接调用组件会绕开这些门禁。[服务端门禁](../../../raw/current/scripts/actions.lua#L4569-L4591)
- 修改边界：原版宿主在服务端挂组件；两端动作发现还看角色／目标标签及容器 replica，和服务端检查并非同一清单。打包后的形态、动画及保存责任留给宿主。[帐篷端边界](../../../raw/current/scripts/prefabs/portabletent.lua#L193-L200) [发现条件](../../../raw/current/scripts/componentactions.lua#L668-L684)

## 关键接口

| 接口／字段 | 签名或默认 | 条件、作用与证据 |
|---|---|---|
| `SetOnDismantleFn(fn)`／`ondismantlefn` | 默认 nil；回调 `(inst, doer)` | 只赋值，不添加资格标签或网络值；nil 时调用不执行效果。[组件](../../../raw/current/scripts/components/portablestructure.lua#L1-L14) |
| `Dismantle(doer)` | 无返回值 | 有回调即调用，不查燃烧、容器、操作者或重复执行；不转发回调结果。[执行](../../../raw/current/scripts/components/portablestructure.lua#L10-L14) |
| `inst.candismantle` | 可选实体函数 `(inst, doer) → 真值／假值` | 不属于组件 setter；动作发现、服务端执行均用方法调用检查。需要客户端菜单一致时两端设置；不能依赖仅服务端可见的组件字段。[发现](../../../raw/current/scripts/componentactions.lua#L678-L681) [执行](../../../raw/current/scripts/actions.lua#L4586-L4591) |
| `ACTIONS.DISMANTLE` 门禁 | 失败可返回 `INUSE`／`NOTEMPTY`／`COOKING` | 非篝火的燃烧目标拒绝；有 container 时优先查容器，只有没有 container 才查 sleepingbag。通过后不检查组件回调的结果。[动作](../../../raw/current/scripts/actions.lua#L4569-L4591) |

右键发现另要求：便携篝火对应使用者标签、厨具对应 `masterchef`、工程建筑对应 `portableengineer`；容器须能开启且不是当前操作者已开启。服务端动作不逐项重复这组标签条件，自定义调用者应明确自己的资格边界。[发现实现](../../../raw/current/scripts/componentactions.lua#L668-L684)

## 典型接入

### 仅允许 Walter 收起便携帐篷，保留原版打包与耐久

以下完整代码放入现有模组 `modmain.lua`，无需新增资源。`portabletent` 是地面帐篷，`portabletent_item` 是背包中的部署物；只改前者的拆解资格。刻意两端设置 `candismantle`，让右键菜单和服务端执行一致，不在客户端读取 sleepingbag／finiteuses。

```lua
AddPrefabPostInit("portabletent", function(inst)
    local oldcandismantle = inst.candismantle
    inst.candismantle = function(target, doer)
        if doer == nil or doer.prefab ~= "walter" then
            return false
        end
        return oldcandismantle == nil or oldcandismantle(target, doer)
    end
end)
```

原版真实注册将 `portabletent` 绑定 `fn`，服务端配置 `OnDismantle`；回调开始收起动画并移除 sleepingbag／portablestructure／workable，动画结束才 `ReplacePrefab` 为物品，转移剩余次数。[注册](../../../raw/current/scripts/prefabs/portabletent.lua#L310-L312) [挂载](../../../raw/current/scripts/prefabs/portabletent.lua#L193-L200) [回调](../../../raw/current/scripts/prefabs/portabletent.lua#L81-L83) [转换全过程](../../../raw/current/scripts/prefabs/portabletent.lua#L19-L39)

本例不直接调用 `Dismantle`，因此仍保留使用中／燃烧等原动作限制；其他角色不会获得该拆解菜单，直接执行原动作也被资格拒绝。资格只影响 DISMANTLE，不禁止锤击、燃烧或其他转换入口；锤击仍可能进入 `ChangeToItem`。不新增监听、任务或存档字段；关闭模组后新加载实体恢复原构造，现存实体不会自动撤销本包装。[动作门禁](../../../raw/current/scripts/actions.lua#L4569-L4591) [锤击入口](../../../raw/current/scripts/prefabs/portabletent.lua#L41-L54)

### 按宿主选择转换方式

| 模式 | 宿主实际职责 | 证据 |
|---|---|---|
| 便携锅：立即生成物品并删除旧建筑 | `ChangeToItem` 处理成品／容器内容，再生成 `portablecookpot_item`；`OnDismantle` 随后删除锅。正常动作会先拦住非空／已做好的锅，不能据转换函数反推菜单允许它们 | [转换](../../../raw/current/scripts/prefabs/portablecookpot.lua#L37-L49) [拆解回调](../../../raw/current/scripts/prefabs/portablecookpot.lua#L211-L214) [挂载](../../../raw/current/scripts/prefabs/portablecookpot.lua#L271-L278) [注册](../../../raw/current/scripts/prefabs/portablecookpot.lua#L382-L384) |
| 便携帐篷：动画结束替换并转移耐久 | 先关闭交互和睡眠组件；动画结束读取次数、替换物品、设置次数。不是组件自动保存状态 | [转换](../../../raw/current/scripts/prefabs/portabletent.lua#L19-L39) [次数配置](../../../raw/current/scripts/prefabs/portabletent.lua#L222-L225) |

玩家双端 SG 均将 DISMANTLE 按建造速度标签路由到对应动作状态；这里只改资格，不另造动作或 SG。[服务端路由](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1512-L1517) [客户端路由](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L782-L787)

## 常见陷阱与限制

- **仅加组件就以为能变成背包物品：** 默认无回调，即使动作返回 true 也可能什么都没发生；完整建筑／物品双形态、部署路径仍由 Prefab 实现。部署机制见 [[reference/components/deployable]]。[空回调执行](../../../raw/current/scripts/components/portablestructure.lua#L1-L14) [锅的部署](../../../raw/current/scripts/prefabs/portablecookpot.lua#L324-L335)
- **回调返回 false 阻止拆解：** 组件不接收结果，动作仍返回 true；将资格放在 `candismantle`，或为自定义流程另设明确的执行契约。[组件](../../../raw/current/scripts/components/portablestructure.lua#L10-L14) [动作](../../../raw/current/scripts/actions.lua#L4586-L4591)
- **脚本直接 Dismantle 导致占用中收起：** 组件不负责容器或睡袋检查；不能将调用组件等同执行完整玩家动作。相关占用语义分别见 [[reference/components/container]]、[[reference/components/sleepingbag]]。[门禁](../../../raw/current/scripts/actions.lua#L4574-L4584)
- **重复调用／转换中存档：** 组件没有一次性锁、存读或移除清理钩子；帐篷靠移组件与宿主转换流程隔离后续交互。不要把动画等待或剩余次数转移归给组件，也不要推断它能恢复任意中间态。[组件全文](../../../raw/current/scripts/components/portablestructure.lua#L1-L16) [帐篷转换](../../../raw/current/scripts/prefabs/portabletent.lua#L19-L39) [帐篷保存](../../../raw/current/scripts/prefabs/portabletent.lua#L69-L79)

## 验证记录

- 版本沿用：2026-10-25，`steam-buildid:26148036`；原结论复用、锚点迁移 5 处（actions.lua DISMANTLE 三区间、SGwilson.lua／SGwilson_client.lua 双端路由）；依据：actions.lua／componentactions.lua 枢纽 diff 审阅＋变更清单命中 actions.lua、componentactions.lua、SGwilson.lua、SGwilson_client.lua，占用／非空／烹饪门禁、candismantle 检查与建造速度路由同旧版。未运行游戏。
- 源码审阅：2026-10-23，`steam-buildid:25643519`；覆盖完整组件、动作发现／服务端门禁、玩家双端 SG 路由、便携锅和帐篷的真实注册／端边界／拆解转换及帐篷耐久交接。示例复用原宿主转换，不代表全部便携建筑或完整睡眠／烹饪机制已审校。
- 游戏运行：未运行；需实测 Walter／其他角色菜单、占用拒绝、动画结束物品和次数、保存读档及其他模组包装冲突。
- 结构检查：页面格式／本地链接、源码引用严格检查及完整 modmain 的 Lua 5.1 语法检查通过。
