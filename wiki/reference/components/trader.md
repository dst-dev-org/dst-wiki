---
title: "trader"
created: "2026-10-11"
updated: "2026-10-28"
type: "component"
summary: "让物品或实体接收投喂材料，处理拒收、堆叠数量、修复及被接受物品的生命周期。"
aliases: ["给物品投喂材料", "接收道具", "材料修复装备", "交易接收", "trader"]
tags: ["component", "item", "callback", "lifecycle"]
dst_build: "steam-buildid:26230496"
sources: ["actions.lua", "componentactions.lua", "components/armor.lua", "components/trader.lua", "modutil.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# trader

## 快速判断

- 职责与适用：接收方组件；被送出的材料数据见 [[reference/components/tradable]]。不局限 NPC，可用于装备投料升级／修复。[components/trader.lua:25](../../../raw/current/scripts/components/trader.lua#L25
- 优先使用：SetAcceptTest 决定是否想收，SetOnAccept 执行结果；只有确需替换身体状态检查才用 SetAbleToAcceptTest。[components/trader.lua:104](../../../raw/current/scripts/components/trader.lua#L104 [components/trader.lua:125](../../../raw/current/scripts/components/trader.lua#L125
- 重要副作用：默认先从原 owner 移出／拆堆，再删除材料，最后调用 onaccept；此时材料不保证有效。[components/trader.lua:129](../../../raw/current/scripts/components/trader.lua#L129
- 修改边界：服务端负责接受；客户端动作还受 trader／alltrader 等条件控制，新工厂应在公共阶段配置所需标签。[components/trader.lua:36](../../../raw/current/scripts/components/trader.lua#L36 [componentactions.lua:1431](../../../raw/current/scripts/componentactions.lua#L1431

## 关键接口

| 接口／字段 | 约定 | 作用与证据 |
|---|---|---|
| `SetAcceptTest(fn)` | `(inst, item, giver, count?) -> truthy` | 愿意接受；拒收走 onrefuse。[components/trader.lua:81](../../../raw/current/scripts/components/trader.lua#L81 [components/trader.lua:125](../../../raw/current/scripts/components/trader.lua#L125 |
| `SetAbleToAcceptTest(fn)` | 同一形参顺序 | 在 enabled／mimic 后提前返回，取代后面的死亡／睡眠／busy 检查；不应只为物品类型过滤使用。[components/trader.lua:104](../../../raw/current/scripts/components/trader.lua#L104 |
| `SetOnAccept(fn)` | `(inst, giver, item, count)` | 与 test 的 item／giver 顺序不同；默认材料已删除，不能异步依赖该实体。[components/trader.lua:151](../../../raw/current/scripts/components/trader.lua#L151 |
| `acceptnontradable`／`deleteitemonaccept` | 默认 false／true | 前者改变 alltrader 标签；后者 false 也不自动给 container，仅可给接收方 inventory。[components/trader.lua:15](../../../raw/current/scripts/components/trader.lua#L15 [components/trader.lua:143](../../../raw/current/scripts/components/trader.lua#L143 |
| `SetAcceptStacks()`／`AcceptGift(giver, item, count?)` | 开启接堆／默认 count=1 | 正常 GIVE 决定可接收数量；组件调用者须验证正整数和不超过实存数量。[components/trader.lua:99](../../../raw/current/scripts/components/trader.lua#L99 [actions.lua:2303](../../../raw/current/scripts/actions.lua#L2303 |

## 典型接入

### 给物品投喂材料

以下放目标模组 `modmain.lua`，令已损坏木甲接受一根木头修复 100 耐久，拒收满耐久木甲。本例不叠加接收整堆，沿用 log 的 tradable 与 armorwood 资源。trader 接受链见 [components/trader.lua:129](../../../raw/current/scripts/components/trader.lua#L129)，修复入口 [components/armor.lua:166](../../../raw/current/scripts/components/armor.lua#L166)；postinit 见 [modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593)。

```lua
AddPrefabPostInit("armorwood", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.armor == nil then return end
    if inst.components.trader == nil then inst:AddComponent("trader") end
    local trader = inst.components.trader
    trader:SetAcceptTest(function(armor_inst, item)
        return item.prefab == "log" and armor_inst.components.armor:IsDamaged()
    end)
    trader:SetOnAccept(function(armor_inst, giver, item, count)
        armor_inst.components.armor:Repair(100 * count)
    end)
end)
```

每次默认一件，修复不超过最大耐久；材料消费由 trader 完成，不再 Remove。作用于模组期间木甲实例，停用后新实例无此接收能力，已修复值不回滚。若目标被其他模组配置 trader，本例会替换其 test／onaccept，必须按目标改为串联／专用工厂；未验证所有持有状态下的 GIVE 交互。

（26148036 核验：armor:SetCondition 由 math.min 改为 math.clamp 到 [0, maxcondition]，正常入参下 Repair 仍不超过最大耐久，本页示例行为不变；GIVE 数量选择锚点随 actions.lua 新增内容平移到 :2293／:2296。）

### 整堆投料与自定义数量

本地片段参照（未随库分发，未测游戏／外部 Hook；[版本与文件指纹](../../../docs/reports/w05/local-mod-references-20261017.md)）：DSTKisaki 的 kisaki_tool.lua:323～359 对材料白名单／剩余耐久计算补充与返还；MCW 的 mcw_magic_armor.lua:32～42 通过模组自定义 cantrader 配数量并按材料堆叠改比例。这些都只是模组选题线索：cantrader 不在原版 Trader／GIVE 的上述消费链中，必须检查模组外部 Hook；模组回调使用被接受的 item.components 也不能照搬为生命周期保证。

默认一次一件优先使用 onaccept 的 count；开启 SetAcceptStacks 后，原版 GIVE 会参考接收方 inventory 容量或材料整堆数。避免把可接收最大量误作本次已消费量；超过需要数量的退还还须保留皮肤、湿度、自定义状态及创建失败策略。不要仅新建同名 Prefab 就假设与原材料等价。[actions.lua:2306](../../../raw/current/scripts/actions.lua#L2306 [components/trader.lua:135](../../../raw/current/scripts/components/trader.lua#L135 [[reference/components/stackable#拆分与自定义状态]]

## 常见陷阱与限制

- **onaccept 再次删除／延迟读材料组件**：默认实体已 Remove；立即读取不保证组件仍可用。需要前置状态时在接受检查／自己的交易事务捕获，或设计保留材料路径。[components/trader.lua:143](../../../raw/current/scripts/components/trader.lua#L143
- **接收测试的形参套到结果回调**：test 为 item,giver，onaccept 为 giver,item，错用会把人物当物品。[components/trader.lua:125](../../../raw/current/scripts/components/trader.lua#L125 [components/trader.lua:152](../../../raw/current/scripts/components/trader.lua#L152
- **deleteitemonaccept=false 自动存入箱子**：仅接收方 inventory 被调用；container 存放、失败返还与卸主必须单独处理。[components/trader.lua:145](../../../raw/current/scripts/components/trader.lua#L145

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；相关调用与示例仍适用，本轮仅重定位证据（锚点 2 处），未运行游戏。

- 源码依据复核：2026-10-17，Steam buildid 25643519；核对范围：GIVE 数量选择、接受测试／结果参数、默认删除材料后回调与木甲修复示例，依据见上文对应锚点；未核验模组外部交易 Hook 或全部持有状态下的 GIVE。
- 历次结构／语法检查：元数据、源码路径／行号、链接／章节、标签、大小和导航登记检查通过；本页 Lua 代码经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查。
- 游戏运行：未运行；案例的多人交互、存档重载和其他模组兼容仍需游戏验证。

相关：[[reference/components/tradable]]；[[reference/components/armor]]；[[reference/components/stackable]]；[[reference/components/leader#人物未来喂食忠诚收益乘二|喂食与招募]]。
