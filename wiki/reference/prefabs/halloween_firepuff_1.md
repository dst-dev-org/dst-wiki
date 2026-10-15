---
title: "万圣夜火焰绒与月绒烟（halloween_firepuff_1）"
created: "2026-11-01"
updated: "2026-11-05"
type: prefab
summary: "fx.lua 通用表 7 注册（火焰绒 3＋冷焰绒 3＋月绒 1）：万圣夜药水被营火族火焰取作燃料时的一次性起焰烟，与月之药水使用、洞穴满月前后月玻璃再生时的烟团；纯表现，燃料与变异结算在调用方。"
aliases: ["halloween_firepuff_1", "halloween_firepuff_2", "halloween_firepuff_3", "halloween_firepuff_cold_1", "halloween_firepuff_cold_2", "halloween_firepuff_cold_3", "halloween_moonpuff", "万圣夜火焰绒", "冷焰绒", "月绒烟", "药水燃料烟", "火绒特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/fx.lua", "prefabs/halloweenpotion_common.lua", "prefabs/halloweenpotion_embers.lua", "prefabs/halloweenpotion_buffs.lua", "prefabs/halloweenpotion_moon.lua", "prefabs/livingtree_root.lua", "components/fuel.lua", "components/fueled.lua", "components/burnable.lua", "components/halloweenpotionmoon.lua", "components/halloweenmoonmutable.lua", "actions.lua", "componentactions.lua", "prefabs/campfire.lua", "prefabs/coldfire.lua", "prefabs/coldfirepit.lua", "prefabs/firepit.lua", "prefabs/portablefirepit.lua", "prefabs/cotl_tabernacle.lua", "prefabs/yotr_decor.lua", "prefabs/grotto_pool_moonglass.lua", "prefabs/grotto_waterfall_small.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 万圣夜火焰绒与月绒烟（halloween_firepuff_1）

## 快速判断

- 职责与适用：通用表 FX 7 注册，两组资源：火焰绒 6 名（halloween_embers／halloween_embers_cold）是万圣夜药水被火取作燃料时的一次性起焰烟，按火焰冷热选组；月绒 1 名（moon_tea_fx）是月之药水使用与月玻璃再生的落点烟。均无玩法结算，协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[表项](../../../raw/current/scripts/fx.lua#L1753-L1814)
- 优先使用：查"药水投火冒烟／月药水烟／月玻璃再生烟"选本页；给自定义燃料物品复刻同款烟见[典型接入](#给物品复刻投火起焰烟)；查工厂公共行为（代理、父挂、回收）读 [[reference/mechanisms/generic-fx-factory]]。
- 重要副作用：全族条目仅 `bloom=true`（泛光）、`fn=FinalOffset3`（`SetFinalOffset(3)`）、音效 `dontstarve/common/fireAddFuel`；燃料量、火堆 debuff、突变等玩法作用全部在生成者。[fn](../../../raw/current/scripts/fx.lua#L9-L11)、[bloom 消费](../../../raw/current/scripts/prefabs/fx.lua#L94-L96)
- 修改边界：生成点全部在主模拟（服务端 `SpawnPrefab`）；表现体动画结束自删，生成者无需清理。条目无 `update_while_paused`／`animqueue` 等特殊分支；动画时长由资源决定未核验。[fn 消费](../../../raw/current/scripts/prefabs/fx.lua#L104-L114)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `halloween_firepuff_1`／`_2`／`_3` | 表项 → `MakeFx` 网络代理＋一帧后本地实体；本地实体 `SetBloomEffectHandle("shaders/anim.ksh")`、`fn(inst, proxy)` 执行 `SetFinalOffset(3)`、延迟 0 播声 | bank／build＝`halloween_embers`，anim 分别 `puff_1`／`puff_2`／`puff_3`，其余字段逐项相同。[表项](../../../raw/current/scripts/fx.lua#L1753-L1778) |
| `halloween_firepuff_cold_1`／`_2`／`_3` | 同上 | 资源换 `halloween_embers_cold`，anim 同 `puff_1`／`puff_2`／`puff_3`；与火版差异仅 bank／build。[表项](../../../raw/current/scripts/fx.lua#L1780-L1805) |
| `halloween_moonpuff` | 同上 | bank＝`fx_moon_tea`、build＝`moon_tea_fx`、anim＝`puff`（单枚）；同表 `wormwood_lunar_transformation_finish` 复用同资源但 fn／音效不同，是另一注册名。[表项](../../../raw/current/scripts/fx.lua#L1807-L1814)、[同资源条目](../../../raw/current/scripts/fx.lua#L3178-L3185) |

定向检索简中 `languages/chinese_s.po` 未发现这 7 个注册名的显示名称键；标题中文为已核验效果描述，非官方译名。资源 zip 由加载器按 build 自动声明（`anim/<build>.zip`）；药水文件另显式声明了 halloween_embers 两 zip。[资产声明](../../../raw/current/scripts/prefabs/fx.lua#L13-L15)、[药水 assets](../../../raw/current/scripts/prefabs/halloweenpotion_common.lua#L7-L11)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：药水作燃料（火焰绒 6 名） | `halloweenpotion_common.SpawnPuffFx`：按 `target:HasTag("blueflame")` 选冷／热组、`math.random` 三选一 `SpawnPrefab`，`AttachToTarget` 把代理父挂火焰、本地位置 (0,0,0)，火焰有 `burnable.fxdata[1].follow` 时给代理装 Follower 跟随该符号 | 服务端；前提是火焰的 `fueled:TakeFuelItem` 取走药水触发 `fuel:Taken`→`ontaken`，且火焰带 `campfire` 标签 | [SpawnPuffFx](../../../raw/current/scripts/prefabs/halloweenpotion_common.lua#L26-L31)、[fuel 链](../../../raw/current/scripts/components/fuel.lua#L31-L36)、[[reference/components/fuel|fuel 组件]] |
| 生成者分支：四类燃料入口 | embers／sparks 药水 `ontaken` 先给火加常驻 debuff 再放烟；health／sanity／bravery 药水、月之药水与种植用活树根 `livingtree_root` 的 `ontaken` 只放烟（按物品计为三类九种药水＋活树根） | 都要求 `HasTag("campfire")`；常驻 buff（sparks／embers 循环动画）是独立 debuff 实体，不走通用表注册 | [embers 入口](../../../raw/current/scripts/prefabs/halloweenpotion_embers.lua#L26-L31)、[buffs 入口](../../../raw/current/scripts/prefabs/halloweenpotion_buffs.lua#L69-L73)、[moon 入口](../../../raw/current/scripts/prefabs/halloweenpotion_moon.lua#L28-L32)、[活树根](../../../raw/current/scripts/prefabs/livingtree_root.lua#L72-L76) |
| 生成者：月之药水使用（月绒） | `onusefn` 落点 `SetPosition`（容器内目标→容器、变异产物→新实体、否则目标），无父挂 | 服务端；HALLOWEENMOONMUTATE 动作链：候选仅排除 `DECOR`，动作层再拒绝燃烧／阴燃／冻结目标，target 为空时动作直接失败、不会调 `Use`；`Use` 无条件调 `onusefn`——变异成败只影响失败事件与落点取变异产物，不影响出烟 | [onusefn](../../../raw/current/scripts/prefabs/halloweenpotion_moon.lua#L13-L26)、[动作](../../../raw/current/scripts/actions.lua#L4601-L4614)、[[reference/components/halloweenpotionmoon|月之药水组件·典型接入]] |
| 生成者：月玻璃再生（月绒） | `grotto_pool_moonglass`／`grotto_waterfall_small` 被采空后由 `set_mined`／`set_fully_mined` 监听 `iscavefullmoon`；回调 `OnCaveFullMoon` 不检查传入的 `fullmoon` 值，概率命中才调 `set_full` 并取消监听 | 服务端；两名月玻璃瀑布／池面共用此链；`iscavefullmoon` 变为 false 时（若监听仍在）同样会触发判定 | [池面](../../../raw/current/scripts/prefabs/grotto_pool_moonglass.lua#L32-L48)、[小瀑布](../../../raw/current/scripts/prefabs/grotto_waterfall_small.lua#L31-L47) |
| 冷／热组条件：`blueflame` 标签 | `puff_fx_cold` 仅当目标有该标签；全库 `AddTag("blueflame")` 仅 coldfire 与 coldfirepit（冷火、冷火坑，均同时带 `campfire` 标签） | 其余营火族走热组 | [coldfire](../../../raw/current/scripts/prefabs/coldfire.lua#L116-L117)、[coldfirepit](../../../raw/current/scripts/prefabs/coldfirepit.lua#L121-L124) |
| 火焰符号：`burnable.fxdata[1].follow` | 营火族用 `AddBurnFX(..., "firefx", true)` 配常驻火苗，故 `fxdata[1].follow＝"firefx"`（偏移取各火源 fxdata，如火坑 y=25）；campfire／firepit／portablefirepit／coldfire／coldfirepit／cotl_tabernacle 六处同式，yotr_decor 带 `campfire` 标签但无 AddBurnFX（Follower 分支静默跳过） | `AddBurnFX` 第 3 参写入 `fxdata.follow`；药水烟的符号跟随只在此条件成立时安装 | [fxdata 结构](../../../raw/current/scripts/components/burnable.lua#L134-L136)、[campfire](../../../raw/current/scripts/prefabs/campfire.lua#L165)、[coldfire](../../../raw/current/scripts/prefabs/coldfire.lua#L135) |
| 依赖声明 | halloweenpotion_common 把 6 名并入各药水 `prefabs`；月药水与两个 grotto 文件在 `prefabs` 数组声明 `halloween_moonpuff` | 仅声明；上列生成点均为实际 `SpawnPrefab`，声明与生成一一对应 | [药水声明](../../../raw/current/scripts/prefabs/halloweenpotion_common.lua#L6)、[月药水声明](../../../raw/current/scripts/prefabs/halloweenpotion_moon.lua#L8-L11) |

本页只存生成关系与差异；代理／本地表现分工、父挂与回收的公共协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。月药水的变异本体在 [[reference/components/halloweenmoonmutable|月光可变组件]]（烟只是伴随表现）。

## 典型接入

### 给物品复刻投火起焰烟

- 前提与选择：目标物品需 `fuel` 组件且会被火接收；`ontaken` 在火焰取走燃料（`fueled:TakeFuelItem`→`fuel:Taken`）时回调，实参 `(fuel_inst, target)`，target 是火不是投放者。复用共享模块 `SpawnPuffFx` 可同时得到冷热选组与父挂跟随。
- 生效与撤销：服务端生成；烟经代理父挂到火焰，动画结束自删，无需清理代码。
- 原版依据：四种燃料物品同一写法（[embers](../../../raw/current/scripts/prefabs/halloweenpotion_embers.lua#L26-L31)、[buffs](../../../raw/current/scripts/prefabs/halloweenpotion_buffs.lua#L69-L73)、[moon](../../../raw/current/scripts/prefabs/halloweenpotion_moon.lua#L28-L32)、[livingtree_root](../../../raw/current/scripts/prefabs/livingtree_root.lua#L72-L76)）；模块导出见 [halloweenpotion_common 返回](../../../raw/current/scripts/prefabs/halloweenpotion_common.lua#L34-L39)。

```lua
local PotionCommon = require("prefabs/halloweenpotion_common")

-- 物品 fn 内、TheWorld.ismastersim 分支：
inst:AddComponent("fuel")
inst.components.fuel.fuelvalue = TUNING.MED_FUEL
inst.components.fuel.ontaken = function(inst, target)  -- target＝取走燃料的火
    if target:HasTag("campfire") then                  -- 原版只对营火族放烟
        PotionCommon.SpawnPuffFx(inst, target)         -- 内部按 blueflame 冷热三选一并父挂
    end
end
```

- 调整与验证：想固定某档直接 `SpawnPrefab("halloween_firepuff_2")` 后自行父挂；`SpawnPuffFx` 需要目标可能有 `burnable`（内部已判空，可安全用于无火苗的 `campfire` 标签实体）；资源由加载器按 build 自动声明，无需随 Prefab 重复声明。

### 复用月绒烟做变异／恢复表现

- 前提与选择：任何主模拟代码可落点生成；原版两处都配"状态恢复／突变完成"的时机（月药水 Use、月玻璃 set_full）。
- 生效与撤销：落点式（不父挂），位置即代理位置；动画结束自删。
- 原版依据：[月药水落点](../../../raw/current/scripts/prefabs/halloweenpotion_moon.lua#L16-L19)、[月玻璃落点](../../../raw/current/scripts/prefabs/grotto_pool_moonglass.lua#L24-L25)。

```lua
local fx = SpawnPrefab("halloween_moonpuff")
fx.Transform:SetPosition(x, y, z)
```

## 常见陷阱与限制

- **误区：月绒只在变异成功时出现**：`onusefn` 在 `Use` 内无条件执行，只要找得到落点对象就生成月绒；`success=false` 只额外推失败事件与零伤害 `attacked`。失败时烟落在目标（或目标所在容器）上；落点链末位的 doer 兜底仅在完全找不到落点对象时触达，而原版动作链 target 为空直接失败、不会进 `Use`，该情形不可达。[Use](../../../raw/current/scripts/components/halloweenpotionmoon.lua#L11-L30)
- **误区：这是营火日常加燃料的烟**：普通燃料（木头等）无 `ontaken` 回调；原版只有四类燃料入口（三类万圣夜药水共 9 种物品＋livingtree_root）把 `SpawnPuffFx` 接入 `ontaken`，烟只在投这些物品时出现。月之药水投火走的也是火焰绒（`ontaken` 路径），不是月绒。[入口表](../../../raw/current/scripts/prefabs/halloweenpotion_buffs.lua#L69-L73)
- **边界：Follower 装在无 AnimState 的代理上**：`AttachToTarget` 对代理调用 `FollowSymbol("firefx", ...)`；通用表本地表现体无 Follower、位置只在创建时复制一次（见工厂页"父挂"行），持续符号跟随不应作为保证依赖，火焰不移动时无可见差异。[AttachToTarget](../../../raw/current/scripts/prefabs/halloweenpotion_common.lua#L13-L24)
- **边界：`AttachToTarget` 的 build／cold_build 形参未被使用**：调用方（含 embers 的 buff 装配）传入也不生效，不要据此推断存在换 build／换肤逻辑。[函数体](../../../raw/current/scripts/prefabs/halloweenpotion_common.lua#L13-L24)
- **边界：投火冒烟与常驻 buff 是两条链**：embers／sparks 药水的火堆 buff（sparks／embers 循环动画 debuff 实体）是独立工厂产物，与一次性火焰绒同文件不同协议。[buff 实体](../../../raw/current/scripts/prefabs/halloweenpotion_embers.lua#L151-L187)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。7 表项逐项核对（bank／build／anim／sound／bloom／fn）及 FinalOffset3 定义；halloweenpotion_common／embers／buffs／moon／livingtree_root 五文件全读，fuel:Taken←fueled:TakeFuelItem 触发链、coldfire／coldfirepit 的 blueflame、全域 `AddTag("campfire")` 七处与营火族 AddBurnFX 符号逐处核对（campfire／firepit／portablefirepit／coldfire／coldfirepit／cotl_tabernacle 六处同式带 firefx，yotr_decor 无 AddBurnFX）；月药水动作链（componentactions 候选→actions 校验→Use→onusefn）与 halloweenmoonmutable:Mutate 返回语义、grotto 两文件再生链全读。生成点检索两轮：全域 `rg "halloween_firepuff|halloween_moonpuff"`（覆盖动态拼名）命中 fx.lua 与 6 个消费者文件；再按共享模块 `SpawnPuffFx|halloweenpotion_common` 反查补入 livingtree_root（字面名检索无命中、经模块复用生成），无其他生成点。简中 .po 定向检索无这 7 名的显示名键。未覆盖：puff 动画资源内容与时长、各火源能否实际接收燃料的逐 fueled 核验、月玻璃再生概率的调参值。
- 游戏运行：未运行；烟的冷热分档表现与符号跟随实际观感未实测。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 通过；lua 示例通过 parser_probe 语法检查。

返回 [[reference/prefabs/fx-table-world-index|通用表 FX·环境与物件]]、[[reference/prefabs/index|Prefab 索引]]。
