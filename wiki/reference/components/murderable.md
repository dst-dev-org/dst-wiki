---
title: "murderable"
created: "2026-10-22"
updated: "2026-10-28"
type: component
summary: "让物品接入 MURDER 动作，理解音效、堆叠消费、施作者事件与直接生成掉落的边界。"
aliases: ["物品杀死动作", "宰杀鱼", "MURDER", "murderable"]
tags: [component, actions, loot, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/murderable.lua", "componentactions.lua", "actions.lua", "prefabs/pondfish.lua", "components/inventoryitem.lua", "components/inventory.lua", "components/container.lua", "components/health_replica.lua", "components/incinerator.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# murderable

## 快速判断

- 职责与适用：组件只添加 `murderable` 标签、保存可选音效字段，并在卸载时移除标签；没有 `Murder`、`Kill` 或掉落方法。[完整实现](../../../raw/current/scripts/components/murderable.lua#L1-L13)
- 优先使用：让没有 `health` 的活物物品参与物品栏 `MURDER` 动作；原版池鱼将它与 `inventoryitem`、`lootdropper` 组合。[物品栏动作](../../../raw/current/scripts/componentactions.lua#L2819-L2821) [池鱼组合](../../../raw/current/scripts/prefabs/pondfish.lua#L119-L129)
- 重要副作用：真正执行发生在 `ACTIONS.MURDER.fn`：取出物品，先向施作者发事件，再生成并给予掉落，最后直接移除受害物品。[动作实现](../../../raw/current/scripts/actions.lua#L3166-L3205)
- 修改边界：原版池鱼在主模拟端安装；仅添加标签不能代替组件和物品组件。动作直接访问受害者 `inventoryitem`、施作者 `Transform`／`SoundEmitter`，有掉落时还需施作者 `inventory`。[端门禁](../../../raw/current/scripts/prefabs/pondfish.lua#L100-L129) [执行依赖](../../../raw/current/scripts/actions.lua#L3166-L3196)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `murdersound` | 默认 `nil`；音效值或 `(victim, doer) → 音效值` | MURDER 用 `FunctionOrValue` 求值，在施作者上播放；若 `health.murdersound` 非 nil 则优先使用后者。[默认字段](../../../raw/current/scripts/components/murderable.lua#L1-L7) [音效消费](../../../raw/current/scripts/actions.lua#L3173-L3177) |
| `OnRemoveFromEntity()` | 无参数、无显式返回 | 移除 `murderable` 标签，不负责存档、死亡或恢复已移除物品。[卸载](../../../raw/current/scripts/components/murderable.lua#L9-L11) |
| `ACTIONS.MURDER.fn(act)` | `invobject` 优先，否则 `target` | 接受带 `health` 或 `murderable` 组件的对象；执行结束返回 true。组件本身不提供可杀性回调。[动作边界](../../../raw/current/scripts/actions.lua#L3166-L3171) [结束](../../../raw/current/scripts/actions.lua#L3199-L3205) |

### 动作中的消费和掉落顺序

1. 调 `inventoryitem:RemoveFromOwner(true, true)`，采用返回实体或原实体，放到施作者位置；随后播放音效并读取该实体的 `StackSize()`，无堆叠组件则为 1。[取出与计数](../../../raw/current/scripts/actions.lua#L3166-L3179)
2. 先向 **施作者** 发送 `murdered`、`killed`，数据均为 `{ victim = murdered, stackmult = stacksize }`，此时尚未生成动作掉落。[事件次序](../../../raw/current/scripts/actions.lua#L3181-L3183)
3. 有 `lootdropper` 时设置 `causeofdeath = doer`，按上述数量逐次 `GenerateLoot()`；每个名字经 `SpawnPrefab` 成功后交给施作者 `inventory:GiveItem`。这条路径不调用 `DropLoot()`，不要套用其散落与后处理约定。[掉落消费](../../../raw/current/scripts/actions.lua#L3185-L3197) 生成名单的规则复用 [[reference/components/lootdropper#关键接口]]。
4. 若受害物品有 `inventory` 且带 `drop_inventory_onmurder`，再转移其库存；最后 `Remove()`。这里没有调用 `health:Kill()`，不能把它等同于普通战斗死亡流程。[结束分支](../../../raw/current/scripts/actions.lua#L3199-L3205)

## 典型接入

### 原版池鱼作为可宰杀物品

`pondfish.lua` 的 `pondfishfn → commonfn` 注册 `pondfish`，另以同一工厂注册 `pondeel`。工厂在主模拟端组合 `inventoryitem + murderable + lootdropper`，从各自数据配置掉落：池鱼为 `fishmeat_small`，池鳗为 `eel`。这是物品转换为掉落的原版参照，不要求额外生命组件或模组示例。[构造组合](../../../raw/current/scripts/prefabs/pondfish.lua#L100-L129) [池鱼数据与工厂](../../../raw/current/scripts/prefabs/pondfish.lua#L161-L176) [池鳗数据与注册](../../../raw/current/scripts/prefabs/pondfish.lua#L178-L196)

玩家与客户端状态图均把 `MURDER` 映射为长动作；`quagmire_fasthands` 改用中等动作。组件只接通物品栏动作入口，执行仍由原动作完成；自定义物品应在服务端构造中组合相应物品与掉落组件，并接受最终移除的生命周期。移除组件仅撤销它提供的标记；动作执行后的物品不能靠重新添加组件恢复。[服务端映射](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1065-L1068) [客户端映射](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L389-L392) [组件清理](../../../raw/current/scripts/components/murderable.lua#L9-L11)

## 常见陷阱与限制

- **只限制 health 侧的可杀门禁就一定禁止**：health 的动作分支经 `replica.health:CanMurder()` 检查，但 murderable 分支无条件添加动作；同时存在时不能只依赖 health 的 `cannotmurder` 标签。[health 分支](../../../raw/current/scripts/componentactions.lua#L2742-L2746) [murderable 分支](../../../raw/current/scripts/componentactions.lua#L2819-L2821) [health 标签判定](../../../raw/current/scripts/components/health_replica.lua#L204-L214)
- **每次一定只消耗一件，或一定消费原始整堆**：动作传 `wholestack=true`，普通库存不拆一件；但容器的 `keepoverstacked=true` 遇超堆可拆出 `originalmaxsize`。掉落倍数以取出后实体的堆数为准。[移出转发](../../../raw/current/scripts/components/inventoryitem.lua#L475-L483) [普通库存分支](../../../raw/current/scripts/components/inventory.lua#L1390-L1413) [容器超堆分支](../../../raw/current/scripts/components/container.lua#L1019-L1037)
- **加组件必定产出掉落**：无 `lootdropper` 仍会发事件并移除对象；有组件也需正确配置名单，且生成失败的 Prefab 会跳过。[动作掉落门禁](../../../raw/current/scripts/actions.lua#L3181-L3205)
- **`murdered` 总表示执行 MURDER 动作**：焚化器也将带 health 或 murderable 的物品视为活物，向施作者发同名事件并附 `incinerated=true`；其 `killed` 还另要求受害者有 combat。消费者需区分来源，不能重复发放动作掉落。[焚化回调](../../../raw/current/scripts/components/incinerator.lua#L16-L38)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；漂移文件 actions.lua、componentactions.lua、components/inventoryitem.lua、stategraphs/SGwilson.lua、SGwilson_client.lua 对照 R-01/R-04/C-19 hunk 地图逐条核验：MURDER 候选（health／murderable 分支）、ACTIONS.MURDER.fn 全链与双侧 SG 映射逐行未变，SGwilson 位移源于头部两段新增（+12/+26）；C-19 的 GetGrandOwner 重写、无雨区吸湿与 OnPickupStopSmoldering 抽出均不触及本页结论（RemoveFromOwner 引用段仅位移）；锚点重定位 15 处、结论无修订；未覆盖：运行验证。

- 源码审阅：2026-10-22，Steam buildid 25643519；覆盖完整标记组件、物品栏动作入口、双方动作映射、池鱼／池鳗工厂与注册、MURDER 音效／事件／名单生成／移除及容器超堆边界，并核验焚化事件差异。未全量审校所有宿主、事件监听者或容器类型。
- 结构检查：本页元数据、大小、链接与源码行号经定向校验；无新增 Lua 示例。
- 游戏运行：未运行；堆叠数量、背包溢出、动作中断及模组交互待实测。

相关：[[reference/components/inventoryitem#关键接口]]；[[reference/components/health#常见陷阱与限制]]。
