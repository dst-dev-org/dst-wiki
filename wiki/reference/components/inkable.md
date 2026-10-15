---
title: "inkable"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "复用鱿鱼墨汁的玩家染墨状态、视线门禁、网络屏幕提示和附着特效。"
aliases: ["鱿鱼墨汁", "玩家染墨", "墨汁遮屏", "inkable"]
tags: [component, event, network, ui, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/inkable.lua", "prefabs/player_common.lua", "prefabs/player_classified.lua", "prefabs/waterprojectiles.lua", "prefabs/squid.lua", "stategraphs/SGsquid.lua", "simutil.lua", "entityscript.lua", "screens/playerhud.lua", "widgets/inkover.lua", "widgets/inkover_splat.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# inkable

## 快速判断

- 职责与适用：为原版玩家记录临时染墨并接通 classified／debuff／HUD 链；同时影响 `CanEntitySeePoint` 视线判定，不只是屏幕装饰。[组件](../../../raw/current/scripts/components/inkable.lua#L1-L36) [视线消费者](../../../raw/current/scripts/simutil.lua#L541-L557)
- 优先使用：已有玩家需要受墨汁影响时在服务端调用 `player.components.inkable:Ink()`，保留网络脉冲与附着效果；原版墨弹在落点半径 1 内对带 inkable 的实体调用它。[原版调用](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L77-L93)
- 重要副作用：每次将剩余时间**重设为 2 秒**，置 `inked=true`、推 classified 事件并添加 `squid_ink_player_fx`；到期清标记、发 `deinked`、移效果并停止更新，无成功返回。[施加／结束](../../../raw/current/scripts/components/inkable.lua#L7-L26)
- 修改边界：`player_common` 在公共构造阶段双端挂载 inkable，但服务端才创建 classified 和处理原版墨弹命中；不能因客户端也有组件就在那里调用 Ink。组件假设有 `player_classified.inked`，不适合直接挂到任意物品。[挂载与门禁](../../../raw/current/scripts/prefabs/player_common.lua#L2511-L2516) [服务端 classified](../../../raw/current/scripts/prefabs/player_common.lua#L2611-L2646) [墨弹门禁](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L312-L327)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `inked`／`inktime` | 初始 nil／0；剩余时间单位秒 | Ink 后为 true／2，OnUpdate 按 dt 递减；这些普通字段没有自动网络复制。[定义与更新](../../../raw/current/scripts/components/inkable.lua#L1-L26) |
| `Ink()` | 无参数、无返回 | 刷新计时而非累加；每次都推 classified `inked` 事件并调用 AddDebuff，不以当前 inked 去重。[实现](../../../raw/current/scripts/components/inkable.lua#L7-L15) |
| `OnUpdate(dt)` | dt 秒；无返回 | 剩余时间 <=0 时清 inked，向玩家发 deinked 并移除效果；单独写 `inked=nil` 不完成这些步骤。[实现](../../../raw/current/scripts/components/inkable.lua#L17-L26) |
| `TransferComponent(newinst)` | 目标须有 inkable | 旧组件 inked 为真时，先调用新组件 Ink 再复制旧剩余时间；会重新推网络脉冲／添加效果，不清旧组件。[实现](../../../raw/current/scripts/components/inkable.lua#L28-L34) |
| `CanEntitySeePoint`／`CanEntitySeeTarget` | 视线检查函数 | 前者拒绝 inked 实体，后者转发目标坐标；不能把组件状态解释成所有客户端绘制都被禁用。[消费者](../../../raw/current/scripts/simutil.lua#L541-L557) |

## 典型接入

### 将玩家服务端染墨状态延长为 5 秒

放在目标模组 `modmain.lua`；这是对已有玩家的完整入口示例，不是新角色工程。无额外美术资源，保留原版 Ink 的 classified、更新与 debuff；`5` 的单位为秒，改的是**服务端染墨／视线门禁及附着效果剩余时间**，不保证屏幕墨迹动画同步变长。[原始施加](../../../raw/current/scripts/components/inkable.lua#L7-L15) [到期清理](../../../raw/current/scripts/components/inkable.lua#L17-L26)

```lua
AddPlayerPostInit(function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local inkable = inst.components.inkable
    if inkable == nil then
        return
    end
    local old_ink = inkable.Ink
    inkable.Ink = function(self)
        old_ink(self)
        self.inktime = 5
    end
end)
```

每次喷墨重新计 5 秒，结束仍走原版清理。移除此模组并重启后恢复构造时的方法；已有角色实例的热卸载不由本例处理。宿主角色替换时，通用 `SwapAllCharacteristics` 遍历有 TransferComponent 的组件；inkable 的转移会在新宿主 Ink 后覆盖成旧剩余时间，保留进度而非保证满 5 秒。[迁移分发](../../../raw/current/scripts/prefabs/player_common.lua#L2329-L2333) [剩余时间迁移](../../../raw/current/scripts/components/inkable.lua#L28-L34)

### 原版从墨弹到屏幕与头部效果的分工

1. `SGsquid` 发射调用宿主 `LaunchProjectile`，宿主生成 `inksplat`；实际注册在 `waterprojectiles.lua`，服务端 complexprojectile 的 OnHit 指向 OnHitInk。命中按落点范围筛 inkable，不只对传入 target 生效。[SG 发射](../../../raw/current/scripts/stategraphs/SGsquid.lua#L335-L348) [宿主发射](../../../raw/current/scripts/prefabs/squid.lua#L52-L57) [命中](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L77-L93) [配置](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L312-L327) [注册](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L401-L405)
2. Ink 推 `player_classified.inked` 这个 net_event；其监听只在焦点属于 parent 时向该玩家发 `inked`。HUD 的 InkOver 收到事件后播放声音并选一层墨迹动画，widget 自己没有读取组件 inktime。[网络字段](../../../raw/current/scripts/prefabs/player_classified.lua#L1471-L1472) [监听登记](../../../raw/current/scripts/prefabs/player_classified.lua#L1236-L1248) [玩家转发](../../../raw/current/scripts/prefabs/player_classified.lua#L777-L781) [HUD 创建](../../../raw/current/scripts/screens/playerhud.lua#L199-L206) [遮罩消费者](../../../raw/current/scripts/widgets/inkover.lua#L5-L49) [动画播放](../../../raw/current/scripts/widgets/inkover_splat.lua#L34-L45)
3. `squid_ink_player_fx` 由 squid 文件单独注册，服务端 debuff 的 attached 把实体跟随到玩家 headbase，detached 播结束动画后移除。debuff 添加条件与通用管理见 [[reference/components/debuffable]]；AddDebuff 可能因状态门禁不施加，而 Ink 不检查其返回值，不能用 inked=true 证明附着成功。[附着／脱离](../../../raw/current/scripts/prefabs/squid.lua#L330-L343) [效果构造与注册](../../../raw/current/scripts/prefabs/squid.lua#L347-L379) [AddDebuff 门禁](../../../raw/current/scripts/entityscript.lua#L2140-L2153)

## 常见陷阱与限制

- **把服务端字段当网络状态：** classified 传的是事件脉冲，不是 inked／inktime 副本；普通客户端也有 inkable，但原版这条接收链只触发本地 HUD 事件，不更新其字段。[双端挂载](../../../raw/current/scripts/prefabs/player_common.lua#L2511-L2516) [转发](../../../raw/current/scripts/prefabs/player_classified.lua#L777-L781) [组件施加](../../../raw/current/scripts/components/inkable.lua#L7-L15)
- **把墨汁当作纯遮屏效果：** inked 会使 `CanEntitySeePoint` 返回假；延长状态也延长这项门禁，不能只根据 UI 外观验收。[视线判断](../../../raw/current/scripts/simutil.lua#L541-L549)
- **直接反复 Ink 累加时长／主动到期：** Ink 每次重置为 2，既不累加也不提供去墨接口；设 inktime=0 需等待后续 OnUpdate 才完成清理，单改 inked 会留下其他协作。[施加与更新](../../../raw/current/scripts/components/inkable.lua#L7-L26)
- **移组件以为会撤销全部状态：** 本组件无 OnRemoveFromEntity／OnSave／OnLoad；动态移除不能依赖其自动发 deinked 或 RemoveDebuff，也不提供读档续时。需要主动清理时核对 debuff 与事件消费者，保留原版到期路径更稳妥。[完整组件](../../../raw/current/scripts/components/inkable.lua#L1-L36)
- **给普通生物挂 inkable 就可被墨弹安全影响：** OnHitInk 只判组件存在，Ink 随即访问 player_classified.inked；不具备玩家网络宿主契约会报错，不应只靠组件名扩散给所有实体。[命中门禁](../../../raw/current/scripts/prefabs/waterprojectiles.lua#L85-L90) [Ink 前提](../../../raw/current/scripts/components/inkable.lua#L7-L15)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 9 处（player_common 挂载／classified／迁移分发、player_classified 监听／网络字段、playerhud HUD 创建、squid 附着与注册、simutil 视线两段、entityscript AddDebuff 门禁）；无结论修订，仅锚点迁移——依据：全局枢纽 diff 审阅＋变更清单命中 player_common.lua、player_classified.lua、squid.lua、simutil.lua、playerhud.lua、entityscript.lua，simutil 本版改动在 Charlie 定位／小地图段，CanEntitySeePoint／CanEntitySeeTarget 判定逐行同旧版；inkable.lua、waterprojectiles.lua、SGsquid.lua 未变。
- 源码审阅：2026-10-23，`steam-buildid:25643519`；覆盖完整 inkable、玩家公共挂载／服务端 classified 与通用迁移、鱿鱼 SG／墨弹注册和命中、视线消费者、classified 到 HUD 的脉冲链、附着效果构造及 AddDebuff 门禁。未审校所有视线函数调用者、完整角色转换或特殊模式 HUD。
- 游戏运行：未运行；5 秒示例的网络表现、死亡／变身与客户端动画未实测。
- 结构检查：本页格式、源码锚点严格检查和完整 modmain 的 Lua 5.1 语法检查通过；语法检查不验证游戏效果。
