---
title: "海芽插穗（waterplant_planter）"
created: "2026-11-10"
updated: "2026-11-10"
type: prefab
summary: "upgrader 材料型物品：用 UPGRADETYPES.WATERPLANT 升级海蚀柱与水草岩、生成海芽并以 2 天重生计时开始产藤壶，以及它的堆叠、燃料与唯一来源。"
aliases: ["waterplant_planter", "海芽插穗", "海芽", "海草插穗", "海蚀柱种植", "在石柱上种植海草", "barnacle_root"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/waterplant_planter.lua", "prefabs/seastack.lua", "prefabs/waterplant_rock.lua", "prefabs/waterplant.lua", "prefabs/waterplant_baby.lua", "prefabs/player_common.lua", "stategraphs/SGwaterplant.lua", "skins_defs_data.lua", "components/upgrader.lua", "components/upgradeable.lua", "components/harvestable.lua", "componentactions.lua", "actions.lua", "constants.lua", "tuning.lua", "strings.lua", "standardcomponents.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 海芽插穗（waterplant_planter）

## 快速判断

- 职责与适用：`waterplant_planter` 是唯一注册，作用是“把海芽种到海蚀柱上”。它**不是** `deployable` 物品，而是 `upgrader` 材料：对 `waterplant_upgradeable` 的目标使用升级动作，把目标替换成海芽（`waterplant_baby`）。本页覆盖这一条链路、物品侧数值与来源；海草本体的战斗、花粉与鱼群行为留源码入口。[构造](../../../raw/current/scripts/prefabs/waterplant_planter.lua#L29-L46)、[组件](../../../raw/current/scripts/prefabs/waterplant_planter.lua#L39-L40)
- 优先使用：改“能种在哪、种出什么”看[升级链](#升级链)；改堆叠、燃料与图鉴文案看[物品侧数值](#物品侧数值)；改获得方式看[来源](#来源)。组件语义见 [[reference/components/upgrader#关键接口]] 与 [[reference/components/upgradeable#关键接口]]。
- 重要副作用：一次升级消费 1 张插穗并**删除目标实体**（普通海蚀柱或已退回的水草岩），在原位生成海芽并播 `waterplant_destroy` 效果；海芽先进入 `REBIRTH_TIME`＝2 天（960 秒）的重生等待，期间不产藤壶。成长是**预约制**：满月状态发生变化时，只有已满产且没有待执行任务（`_grow_task == nil`）才预约；满月时长满也可预约，同样受这条防重限制；执行与读档重排都不查月相。[产物](../../../raw/current/scripts/prefabs/seastack.lua#L61-L80)、[等待](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L171-L183)、[长成](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L40-L55)
- 修改边界：目标侧组件（`upgradeable` 的阶段与消费）归目标页；海草死亡后留下的 `waterplant_rock` 只是本页的目标之一，它的开采与掉落不在本页；`waterplant_destroy` 的动画与生成时机见 [[reference/prefabs/waterplant_destroy#关联机制导航]]。

## 注册与构造

`fn()` 走标准物品路径：`MakeInventoryPhysics` → bank／build 都是 `barnacle_root`、播 `idle` → `MakeInventoryFloatable(inst, "med", 0.2, 0.85)` → `scrapbook_specialinfo = "PLANTABLE_SEASTACK"` → `SetPristine()` → 主模拟段。[工厂](../../../raw/current/scripts/prefabs/waterplant_planter.lua#L7-L27)、[图鉴文案](../../../raw/current/scripts/strings.lua#L18970)

| 条件 | 组件／字段 | 取值与作用 | 证据 |
|---|---|---|---|
| 主模拟 | `stackable.maxsize` | `TUNING.STACK_SIZE_LARGEITEM`＝10 | [堆叠](../../../raw/current/scripts/prefabs/waterplant_planter.lua#L29-L30)、[常量](../../../raw/current/scripts/tuning.lua#L81-L82) |
| 主模拟 | `upgrader.upgradetype` | `UPGRADETYPES.WATERPLANT`；赋值时给自身加 `waterplant_upgrader` 标签 | [赋值](../../../raw/current/scripts/prefabs/waterplant_planter.lua#L39-L40)、[标签](../../../raw/current/scripts/components/upgrader.lua#L1-L8) |
| 主模拟 | `fuel.fuelvalue` | `TUNING.LARGE_FUEL`＝180 秒 | [燃料](../../../raw/current/scripts/prefabs/waterplant_planter.lua#L36-L37)、[常量](../../../raw/current/scripts/tuning.lua#L1677) |
| 主模拟 | `burnable` | `MakeMediumBurnable(inst, TUNING.LARGE_BURNTIME)`，燃尽 30 秒 | [可燃](../../../raw/current/scripts/prefabs/waterplant_planter.lua#L42)、[常量](../../../raw/current/scripts/tuning.lua#L1683) |
| 主模拟 | `propagator`／`hauntable` | `MakeSmallPropagator`；`MakeHauntableLaunchAndIgnite`（点火分支在 `#HAUNTFIX` 下停用，只弹飞） | [接线](../../../raw/current/scripts/prefabs/waterplant_planter.lua#L43-L44)、[实现](../../../raw/current/scripts/standardcomponents.lua#L1191-L1217) |
| 未挂 | `deployable`／`deployedplant` | 没有部署组件，也没有部署标签；放置不走 `DEPLOY` 动作 | [全文](../../../raw/current/scripts/prefabs/waterplant_planter.lua#L1-L49) |

## 升级链

`UPGRADE` 要分两段看：**候选**只决定这条动作会不会出现在可用动作里，**执行**另有一套检查；候选通过不是执行前提，执行也不重新验证“菜单里有没有”。

1. **候选（菜单）**：`componentactions` 遍历 `UPGRADETYPES`，材料带 `<type>_upgrader`、使用者带 `<type>_upgradeuser`、目标带 `<type>_upgradeable` 三者齐全，就把 `ACTIONS.UPGRADE` 插进可用动作列表。[候选](../../../raw/current/scripts/componentactions.lua#L1889-L1897)
2. **执行资格（服务端）**：`ACTIONS.UPGRADE.fn` 检查对象与组件（`target.components.upgradeable`、`invobject.components.upgrader`），再调 `Upgrader:CanUpgrade(target, doer)`——后者只比较 `upgradetype` 相等与 `doer:HasTag("waterplant_upgradeuser")`；所有经 `MakePlayerCharacter` 构造的角色都带这个标签。它不检查这条动作是否来自菜单。[执行](../../../raw/current/scripts/actions.lua#L4056-L4068)、[方法](../../../raw/current/scripts/components/upgrader.lua#L21-L29)、[角色标签](../../../raw/current/scripts/prefabs/player_common.lua#L2456)
3. **目标侧**：`Upgradeable:CanUpgrade()` 要求 `stage < numstages`；两个原版目标都没改这两项，默认 `1 < 3` 恒成立。[判定](../../../raw/current/scripts/components/upgradeable.lua#L60-L73)
4. **结算**：通过上面的检查后 `ACTIONS.UPGRADE.fn` 调 `Upgradeable:Upgrade(invobject, doer)`。后者按固定顺序做四件事：先累加 `numupgrades += upgradevalue`（默认 1），再**消费 1 件**，然后调目标注册的 `onupgradefn`，最后才判 `numupgrades >= upgradesperstage` 并可能 `AdvanceStage`——即累加发生在回调之前，而回调把目标删掉后，这条判定与可能的阶段推进仍发生在已移除的实体上。另外 `Upgradeable:Upgrade` **丢弃** `onupgradefn` 的返回值并固定返回 `true`，所以海蚀柱回调里 `return baby` 只在直接调用该回调时有用，走动作拿不到这个对象。[执行](../../../raw/current/scripts/actions.lua#L4056-L4068)、[顺序与返回](../../../raw/current/scripts/components/upgradeable.lua#L75-L93)、[回调返回](../../../raw/current/scripts/prefabs/seastack.lua#L79)
5. **文案**：`ACTIONS.UPGRADE.strfn` 在目标带 `waterplant_upgradeable` 时返回专用串 `"WATERPLANT"`，所以动作名不是通用“升级”。[文案分支](../../../raw/current/scripts/actions.lua#L4071-L4074)

| 目标 | 目标的 `upgradeable` 配置 | 升级回调做的事 | 证据 |
|---|---|---|---|
| `seastack`（海蚀柱） | `upgradetype = WATERPLANT`、`onupgradefn = OnUpgraded` | 在 `(x, 0, z)` 生成 `waterplant_baby` 并 `WaitForRebirth()`；若 `doer` 非空则向世界推 `itemplanted`；生成 `waterplant_destroy`；删除海蚀柱；把海芽返回给调用方（动作层会丢弃该返回值，只在直接调回调时有意义） | [配置](../../../raw/current/scripts/prefabs/seastack.lua#L224-L226)、[回调](../../../raw/current/scripts/prefabs/seastack.lua#L61-L80) |
| `waterplant_rock`（海草死亡后留下的岩） | 同上 | 在自身坐标生成海芽并 `WaitForRebirth()`、生成 `waterplant_destroy`、（`doer` 非空时）推 `itemplanted`、删除岩石 | [配置](../../../raw/current/scripts/prefabs/waterplant_rock.lua#L135-L137)、[回调](../../../raw/current/scripts/prefabs/waterplant_rock.lua#L36-L53) |

- 两个目标都只是外观不同的“石柱”：`waterplant_rock` 用 `SetPrefabNameOverride("seastack")` 显示为“海蚀柱”，所以玩家看到的两种石柱同名。[覆盖](../../../raw/current/scripts/prefabs/waterplant_rock.lua#L87-L88)、[名称](../../../raw/current/scripts/languages/chinese_s.po#L349065-L349067)
- 阶段阈值是默认 `numstages = 3`、`upgradesperstage = 5`；原版一件只加 1，且回调随即替换目标，所以普通路径一次到不了 5、也不会推进阶段。只有自定义目标不删自身、或把 `upgradevalue` 调到 ≥ 5 时，才可能在随后（或同一实体上）走到 `AdvanceStage`——对本页两个目标没有可观察产出。[默认值](../../../raw/current/scripts/components/upgradeable.lua#L17-L26)、[累计与推进](../../../raw/current/scripts/components/upgradeable.lua#L88-L92)

## 与 waterplant 的关系

```text
waterplant（海草） --死亡掉落 1 张--> waterplant_planter --UPGRADE--> waterplant_baby（海芽）
waterplant（海草） --死亡／烧毁--> waterplant_rock（水草岩） <--UPGRADE-- waterplant_planter
waterplant_baby --满月状态变化时满产且无待执行任务，或满月时长满且无待执行任务 → 预约任务--> waterplant（海草）
```

- **向上的产物**：`waterplant_baby` 的 `WaitForRebirth` 会隐藏藤壶、停掉 `harvestable` 生长并把产量清零，再等 `REBIRTH_TIME`（2 天）播 `rebirth` 后重新开始产藤壶；也就是说插穗种下后不是立刻可采。[等待](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L171-L183)、[常量](../../../raw/current/scripts/tuning.lua#L5534-L5537)
- **长成海草分三层，月相只在预约层**：①**预约入口 A**——`inst:WatchWorldState("isfullmoon", on_full_moon)`，世界状态**只要变化就回调**（`true→false` 的月落也算），回调只检查 `produce == maxproduce` 与 `_grow_task == nil`，不读新值也不查当前月相；②**预约入口 B**——`on_grow` 长满那一刻要求 `TheWorld.state.isfullmoon` 为真，同样受 `_grow_task == nil` 防重。两处都延迟随机 2–4 秒执行 `try_full_grow`；③**执行**——到点只复核 `harvestable` 与 `produce >= TUNING.WATERPLANT.MAX_BARNACLES`，不足就取消、停在原形态，通过才 `do_full_grow` 播 `growth1`、16 帧后 `finish_full_grow` 生成 `waterplant` 并把未满的产量带过去。读档把 `_grow_task` 存成 `grow_on_load`，`on_load` 直接按随机 2–4 秒重排，同样不查月相。三种输入核对：满产且无任务→预约；未满产→不预约；已有 `_grow_task`→不重复预约（月落时同样如此）。[监听回调](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L73-L80)、[注册监听](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L251-L253)、[状态广播](../../../raw/current/scripts/components/worldstate.lua#L24-L35)、[长满分支](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L102-L107)、[复检](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L64-L71)、[生成](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L40-L55)、[读档重排](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L141-L156)
- **海草的死亡链**：`SGwaterplant` 的死亡态在 `animover` 时先按剩余藤壶喷生／熟藤壶，再固定喷 1 张 `waterplant_planter`，最后 `RevertToRock()` 生成 `waterplant_rock`。[死亡态](../../../raw/current/scripts/stategraphs/SGwaterplant.lua#L122-L134)、[回到岩石](../../../raw/current/scripts/prefabs/waterplant.lua#L128-L134)
- **燃尽链不产插穗，但死亡态不看燃烧状态**：`on_burnt` 只把剩余藤壶按个数变成 `barnacle_cooked` 并生成 `waterplant_rock`，随后由默认燃尽处理删除本体，这条路径不经过死亡态，所以不产插穗；而 `SGwaterplant` 的死亡态 `animover` 是无条件发插穗的——即使此刻 `burnable` 正在燃烧，只要实体走到死亡动画结束仍会发 1 张。[烧毁](../../../raw/current/scripts/prefabs/waterplant.lua#L248-L259)、[默认燃尽](../../../raw/current/scripts/standardcomponents.lua#L18-L44)、[死亡态发件](../../../raw/current/scripts/stategraphs/SGwaterplant.lua#L122-L135)

## 物品侧数值

| 项目 | 取值 | 证据 |
|---|---|---|
| 堆叠 | 10（`STACK_SIZE_LARGEITEM`） | [堆叠](../../../raw/current/scripts/prefabs/waterplant_planter.lua#L29-L30) |
| 燃料 | 180 秒（`LARGE_FUEL`） | [燃料](../../../raw/current/scripts/prefabs/waterplant_planter.lua#L36-L37)、[常量](../../../raw/current/scripts/tuning.lua#L1677) |
| 可燃 | 燃尽 30 秒（`LARGE_BURNTIME`） | [可燃](../../../raw/current/scripts/prefabs/waterplant_planter.lua#L42)、[常量](../../../raw/current/scripts/tuning.lua#L1683) |
| 浮水 | `"med"`、垂直偏移 0.2、缩放 0.85 | [浮水](../../../raw/current/scripts/prefabs/waterplant_planter.lua#L20) |
| 图鉴 | `PLANTABLE_SEASTACK`："Can be planted on Sea Stacks." | [字段](../../../raw/current/scripts/prefabs/waterplant_planter.lua#L22)、[文案](../../../raw/current/scripts/strings.lua#L18970) |
| 升级值 | `upgradevalue` 默认 1，对这两个目标无实际差异 | [默认](../../../raw/current/scripts/components/upgrader.lua#L10-L19) |

## 来源

- **唯一生成点**：本版脚本里只有一处 `SpawnPrefab("waterplant_planter")`，即海草死亡态的 `animover`；没有配方、没有掉落表条目、也没有世界生成引用。[生成](../../../raw/current/scripts/stategraphs/SGwaterplant.lua#L130)
- **图鉴与依赖**：海草与海蚀柱的图鉴依赖都列出本物品，说明“插穗→海芽→海草”这条链在界面层也是这么串的。[图鉴数据](../../../raw/current/scripts/screens/redux/scrapbookdata.lua#L1305)
- **反向用途**：它同时是燃料与可燃物，误扔进火里就没了；10 个一叠也意味着一次失误最多损失整叠。[组件段](../../../raw/current/scripts/prefabs/waterplant_planter.lua#L36-L44)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：upgrader | 自身是材料侧：存 `upgradetype` 与默认 `upgradevalue = 1` | 主模拟；赋值即加 `waterplant_upgrader` 标签 | [[reference/components/upgrader#关键接口]]；[赋值](../../../raw/current/scripts/prefabs/waterplant_planter.lua#L39-L40) |
| 组件：upgradeable | 目标侧负责累计、消费与 `onupgradefn` | `seastack` 与 `waterplant_rock` 挂载 | [[reference/components/upgradeable#关键接口]]；[消费](../../../raw/current/scripts/components/upgradeable.lua#L75-L93) |
| 组件：stackable | 上限 10，升级时取 1 件 | 主模拟 | [[reference/components/stackable#关键接口]]；[堆叠](../../../raw/current/scripts/prefabs/waterplant_planter.lua#L29-L30) |
| 组件：fuel／burnable／propagator／hauntable | 燃料 180 秒、燃尽 30 秒、蔓延、作祟只弹飞 | 主模拟 | [[reference/components/fuel#关键接口]]；[组件段](../../../raw/current/scripts/prefabs/waterplant_planter.lua#L36-L44) |
| 组件：harvestable | 海芽与海草的藤壶产量与生长；插穗本身不挂 | 目标产物侧 | [[reference/components/harvestable#关键接口]]；[接线](../../../raw/current/scripts/prefabs/waterplant.lua#L441-L449) |
| SG：`SGwaterplant` | 海草的死亡态是插穗唯一来源 | 海草本体 | Wiki 待建；[死亡态](../../../raw/current/scripts/stategraphs/SGwaterplant.lua#L122-L134) |
| API／数据：`UPGRADETYPES.WATERPLANT` | 取值为 `"waterplant"`，拼出三组标签 | 主模拟与客户端候选 | [枚举](../../../raw/current/scripts/constants.lua#L1969-L1978)、[候选](../../../raw/current/scripts/componentactions.lua#L1889-L1897) |
| API／数据：`ACTIONS.UPGRADE` | 执行与专用文案分支 | 服务端执行 | [执行](../../../raw/current/scripts/actions.lua#L4056-L4068)、[文案](../../../raw/current/scripts/actions.lua#L4071-L4074) |
| 关联 Prefab：`seastack` | 主要目标，普通海蚀柱 | 世界态，海上 | Wiki 待建；[回调](../../../raw/current/scripts/prefabs/seastack.lua#L61-L80) |
| 关联 Prefab：`waterplant_rock` | 海草死亡／烧毁后的岩石，也是合法目标 | 世界态，海上 | Wiki 待建；[配置](../../../raw/current/scripts/prefabs/waterplant_rock.lua#L135-L137) |
| 关联 Prefab：`waterplant_baby`、`waterplant` | 产物与最终形态 | 世界态 | Wiki 待建；[长成](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L40-L55) |
| 关联 Prefab：`waterplant_destroy` | 升级时在原位播放的爆裂效果 | 通用 FX 表 | [[reference/prefabs/waterplant_destroy#关联机制导航]] |
| 资源：anim 包 | `anim/barnacle_root.zip` 与 `INV_IMAGE waterplant_planter` | 客户端段加载 | [资产](../../../raw/current/scripts/prefabs/waterplant_planter.lua#L1-L5) |

## 典型接入

### 把插穗的堆叠上限提到 20，保持升级仍然一次只扣 1 张

- 前提与选择：服务端 `modmain.lua`。消费逻辑在目标的 `Upgradeable:Upgrade` 里，与材料堆叠上限无关，所以只改 `stackable.maxsize` 就能一次带更多，而不会一次全被吃掉。[消费](../../../raw/current/scripts/components/upgradeable.lua#L75-L85)
- 生效与撤销：`AddPrefabPostInit` 在每次构造后执行，含读档重建；停用模组后新造出的插穗恢复 10 的上限，已经存在堆叠里的数量不回退。
- 原版依据：上限来自 `TUNING.STACK_SIZE_LARGEITEM`，消费点用 `stackable:Get(1):Remove()`，两者互不影响。[常量](../../../raw/current/scripts/tuning.lua#L81-L82)、[取 1 件](../../../raw/current/scripts/components/upgradeable.lua#L78-L82)

```lua
-- modmain.lua：海芽插穗一叠 20 张，升级仍然一次只消耗 1 张
AddPrefabPostInit("waterplant_planter", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local stackable = inst.components.stackable
    if stackable ~= nil then
        stackable.maxsize = 20
    end
end)
```

- 调整与验证：单位是“张”；替换点是 `maxsize`，不是 `upgradevalue`（后者只影响目标侧累计，对海蚀柱与水草岩没有实际产出差异）。若要让插穗不被点燃烧掉，要改的是 `burnable`（或直接 `RemoveComponent("burnable")`）；`fuel` 是独立组件，移除 `burnable` 不影响把它当燃料投进火里，反过来 `fuel` 值也不参与被点燃后的燃尽。

## 常见陷阱与限制

- **误区：“插穗是部署物品，走放置预览”**：它没有 `deployable`，也没有 `deployedplant` 标签，判定与执行都在 `UPGRADE` 动作里；所以海蚀柱上不会出现绿色／红色放置预览，取而代之的是升级动作名 `"WATERPLANT"`。[组件](../../../raw/current/scripts/prefabs/waterplant_planter.lua#L29-L46)、[文案](../../../raw/current/scripts/actions.lua#L4071-L4074)
- **误区：“烧掉海草也能拿到插穗”**：燃尽走 `on_burnt` → 熟藤壶＋`waterplant_rock`，不产插穗；但“被打死时正在燃烧”是另一条路——死亡态 `animover` 不检查燃烧，照样发 1 张。别把这两条都写成“烧死不产”。[烧毁](../../../raw/current/scripts/prefabs/waterplant.lua#L248-L259)、[死亡态](../../../raw/current/scripts/stategraphs/SGwaterplant.lua#L122-L135)
- **边界：升级会吃掉整块岩石**：对 `waterplant_rock` 用插穗会删掉那块岩，只在原位留下海芽；想留着岩石开采就别拿它当目标。[回调](../../../raw/current/scripts/prefabs/waterplant_rock.lua#L36-L53)
- **边界：种下后 2 天内没有收成**：`WaitForRebirth` 期间 `harvestable` 被停掉且产量清零，`REBIRTH_TIME`＝2 天，之后才恢复生长。[等待](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L171-L183)
- **边界：海芽不保证长成海草**：月落时只要满产且没有待执行任务也能预约（`on_full_moon` 不查月相），满月时长满同样只是预约且受 `_grow_task` 防重；任务触发时产量掉下去就取消，所以停在芽态可能是任一条件没满足，而不是单纯“满月就会长”。[监听](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L73-L80)、[触发复核](../../../raw/current/scripts/prefabs/waterplant_baby.lua#L64-L71)
- **未核验**：海草本体（战斗、花粉、鱼群）与海芽的完整行为没有对应 Wiki 页；本页只覆盖与插穗相连的那一段。

## 验证记录

- 源码审阅：2026-11-10，Steam buildid 26474873。逐行读完 `waterplant_planter.lua`（全部 49 行）、`seastack.lua` 与 `waterplant_rock.lua` 的升级回调段、`waterplant.lua` 的死亡与烧毁段、`waterplant_baby.lua` 的重生与长成段、`SGwaterplant.lua` 的死亡态；核验 `upgrader`／`upgradeable` 的判定与消费顺序、`componentactions` 的候选条件、`ACTIONS.UPGRADE` 的执行与文案、`player_common` 的角色标签、`DefaultBurntFn` 的删除路径与 tuning 常量换算。
- 名称核验（`inspect_prefab_names.py`，只认 `languages/chinese_s.po` 的 msgstr）：`WATERPLANT_PLANTER`＝海芽插穗（Sea Sprout Starter）、`WATERPLANT_BABY`＝海芽、`WATERPLANT`＝海草、`SEASTACK`＝海蚀柱；`WATERPLANT_ROCK` 无独立键，靠 `SetPrefabNameOverride("seastack")` 显示为“海蚀柱”。[PLANTER](../../../raw/current/scripts/languages/chinese_s.po#L351850-L351852)、[BABY](../../../raw/current/scripts/languages/chinese_s.po#L351840-L351842)、[WATERPLANT](../../../raw/current/scripts/languages/chinese_s.po#L351835-L351837)、[SEASTACK](../../../raw/current/scripts/languages/chinese_s.po#L349065-L349067)
- 检索范围：在 `raw/current/scripts` 全域检索 `waterplant_planter`，命中只有本文件、`waterplant.lua` 的依赖声明、`prefablist.lua`／`debugcommands.lua`／`scrapbook_prefabs.lua`、图鉴数据、`skins_defs_data.lua` 的一行图鉴编号解锁注释（`AddScrapbook13Key`），以及 `SGwaterplant.lua` 的一处生成点（功能性入口见[来源](#来源)）；未发现配方、掉落表或世界生成引用。
- 游戏运行：未运行。2 天＝960 秒、`LARGE_FUEL`＝180 秒、`LARGE_BURNTIME`＝30 秒均由 tuning（`seg_time = 30`、`total_day_time = 480`）推导；重生与长成是代码路径推导。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本批报告。

## 相关链接

- 关联对象：[[reference/prefabs/waterplant_destroy]]、[[reference/prefabs/waterplant_pollen_fx]]
- 组件：[[reference/components/upgrader]]、[[reference/components/upgradeable]]、[[reference/components/stackable]]、[[reference/components/harvestable]]

返回 [[reference/prefabs/index|Prefab 索引]]。
