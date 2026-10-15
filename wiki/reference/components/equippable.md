---
title: equippable
created: "2026-10-11"
updated: "2026-10-28"
type: component
summary: "管理装备状态和回调，配置装备槽、移动倍率、精神速率及穿戴限制。"
aliases: ["装备回调", "穿戴与卸下", "入袋回调", "装备加速", "装备回精神", "equippable"]
tags: [component, item, equipment, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/equippable.lua", "components/equippable_replica.lua", "components/inventoryitem_replica.lua", "components/inventory.lua", "components/linkeditem.lua", "components/locomotor.lua", "components/sanity.lua", "class.lua", "entityscript.lua", "constants.lua", "equipslotutil.lua", "tuning.lua", "prefabs/cane.lua", "prefabs/torch.lua", "prefabs/hats.lua", "modutil.lua", "mainfunctions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# equippable

## 快速判断

- 职责与适用：管理物品的装备标志、回调和属性；**装备才生效**的加速／精神贡献可用本组件，放物品栏就生效的需求须另查持有者机制。[cane.lua:67](../../../raw/current/scripts/prefabs/cane.lua#L67)、[sanity.lua:510](../../../raw/current/scripts/components/sanity.lua#L510
- 优先使用：Prefab 初始化配置字段及 `SetOnEquip`／`SetOnUnequip`；实际穿脱由持有者 `inventory:Equip`／`Unequip` 管理。[inventory.lua:1219](../../../raw/current/scripts/components/inventory.lua#L1219)、[:1266](../../../raw/current/scripts/components/inventory.lua#L1266)
- 重要副作用：装备／卸装回调后在**物品**发 `equipped`／`unequipped`；人物另发 `equip`／`unequip`。速度、槽位与限制字段写入经 Class 属性回调同步 replica。[equippable.lua:1](../../../raw/current/scripts/components/equippable.lua#L1)、[:92](../../../raw/current/scripts/components/equippable.lua#L92)、[inventory.lua:1241](../../../raw/current/scripts/components/inventory.lua#L1241)、[:1377](../../../raw/current/scripts/components/inventory.lua#L1377)
- 修改边界：服务端组件配置；客户端查相应 replica。回调 owner 的约定因入口不同：装备为实体，原版入袋经常为 nil。组件 `Equip` 不替代人物 inventory 的槽位／物品转移流程。

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `equipslot` | `string`；默认 `EQUIPSLOTS.HANDS` | 值为槽名，网络 ID 由工具转换；[constants.lua:654](../../../raw/current/scripts/constants.lua#L654)、[equipslotutil.lua:6](../../../raw/current/scripts/equipslotutil.lua#L6 |
| `SetOnEquip(fn?)`／`SetOnEquipToModel(fn?)` | 回调 `(inst, owner, from_ground)`；第三参可 nil | nil 清除；假人回调在普通回调与物品事件之后；[equippable.lua:68](../../../raw/current/scripts/components/equippable.lua#L68)、[:92](../../../raw/current/scripts/components/equippable.lua#L92) |
| `SetOnUnequip(fn?)`／`SetOnPocket(fn?)` | `(inst, owner)`；入袋 owner 可 nil | nil 清除；ToPocket 仅调回调，不改装备标志；[equippable.lua:72](../../../raw/current/scripts/components/equippable.lua#L72)、[:109](../../../raw/current/scripts/components/equippable.lua#L109) |
| `walkspeedmult`／`GetWalkSpeedMult()` | `number\|nil`；默认 nil，读取按 1 | 已装备／有 owner 才应用 vigorbuff 与持有者修正；网络范围见下文；[equippable.lua:126](../../../raw/current/scripts/components/equippable.lua#L126 |
| `dapperness`／`SetDappernessFn(fn?)` | 默认 0；回调 `(inst, owner?) -> number` | 动态回调覆盖基础值，随后仍有湿物品惩罚；[equippable.lua:196](../../../raw/current/scripts/components/equippable.lua#L196 |
| `GetDapperness(owner?, ignore_wetness?)` | 返回数值速率贡献 | owner 可省略；自定义回调若访问 owner 须处理 nil。只读接口不能证明人物净增长；[equippable.lua:196](../../../raw/current/scripts/components/equippable.lua#L196)、[sanity.lua:510](../../../raw/current/scripts/components/sanity.lua#L510 |
| `restrictedtag`／`IsRestricted(target)` | `string\|nil`；判定返回 boolean | true 为受限，nil／空标签不限制；仅 player／possessedbody 应用绑定和标签限制；[equippable.lua:147](../../../raw/current/scripts/components/equippable.lua#L147 |
| `IsRestricted_FromLoad(target)` | 实体参数，返回 boolean | 只放宽匹配人物的技能树标签，其他转正常判定；[equippable.lua:164](../../../raw/current/scripts/components/equippable.lua#L164 |
| `SetPreventUnequipping(flag?)` | 布尔约定；nil／省略解除 | 开启时监听 onremove；`ShouldPreventUnequipping()` 严格比较 true；[equippable.lua:176](../../../raw/current/scripts/components/equippable.lua#L176 |
| `IsInsulated()`／`GetEquippedMoisture()` | boolean／`{moisture:number,max:number}` | 电气绝缘不是温度保温；湿度返回单个表；[equippable.lua:64](../../../raw/current/scripts/components/equippable.lua#L64)、[:214](../../../raw/current/scripts/components/equippable.lua#L214) |

## 典型接入

### 修改已有手杖：装备加速与精神基础贡献

- 前提与选择：放目标模组已加载的 `modmain.lua`；沿用原版 cane 资源、槽位、装备和卸装回调，不新增资源。原版服务端先配置 equippable，再由 Prefab postinit 修改。[cane.lua:54](../../../raw/current/scripts/prefabs/cane.lua#L54)、[:67](../../../raw/current/scripts/prefabs/cane.lua#L67)、[modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593)、[mainfunctions.lua:388](../../../raw/current/scripts/mainfunctions.lua#L388
- 生效与撤销：此模组下手杖实例基础速度倍率改为 1.30，装备精神基础贡献为 `3 / 60`／秒；物品栏携带不生效。消费者从装备槽读取，卸下后不再消费该项，无须自建周期任务。[locomotor.lua:188](../../../raw/current/scripts/components/locomotor.lua#L188)、[sanity.lua:510](../../../raw/current/scripts/components/sanity.lua#L510

```lua
AddPrefabPostInit("cane", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local equippable = inst.components.equippable
    if equippable == nil then
        return
    end
    equippable.walkspeedmult = 1.30
    equippable.dapperness = 3 / 60
end)
```

- 调整与验证：`1.30` 为基础倍率；精神贡献还受角色规则、潮湿、其他来源及 rate_modifier 影响，**不保证净增 3／分钟**。原版 cane 无动态 dapperfn，本例未覆盖其他模组的动态回调／冲突；已源码审阅，未游戏运行。停用后新实例沿原版配置。[sanity.lua:523](../../../raw/current/scripts/components/sanity.lua#L523)、[:590](../../../raw/current/scripts/components/sanity.lua#L590)
- 原版不同模式：手杖为属性＋手持显示；[torch.lua:130](../../../raw/current/scripts/prefabs/torch.lua#L130)、[:166](../../../raw/current/scripts/prefabs/torch.lua#L166)、[:188](../../../raw/current/scripts/prefabs/torch.lua#L188) 展示装备、卸装与假人额外收尾；[hats.lua:4684](../../../raw/current/scripts/prefabs/hats.lua#L4684 展示按时段返回动态精神贡献。只读当前需求所需模式。

## 常见陷阱与限制

- **把入袋 owner 当人物**：Inventory 的 ToPocket 多处不传参，直接访问回调 owner 会报错；先判 nil。ToPocket 不代表所有物品持有者变化，不能靠它实现完整随身效果。[inventory.lua:1023](../../../raw/current/scripts/components/inventory.lua#L1023)、[:1155](../../../raw/current/scripts/components/inventory.lua#L1155)、[:1351](../../../raw/current/scripts/components/inventory.lua#L1351)
- **自己直接调组件 Equip**：只改标志／回调／事件，没有登记人物槽位。原版 inventory 先 OnPutInInventory，再组件 Equip，随后登记槽位和发人物 equip；物品 equipped 回调里查人物槽位仍可能是旧值。[inventory.lua:1364](../../../raw/current/scripts/components/inventory.lua#L1364
- **网络速度填任意小数**：SetWalkSpeedMult 对序列化数值 `x` 要求 0～255，并断言 `abs(mult*100-x)<.01`；按 0.01 步进配置 0～2.55，不能只看服务端赋值成功。与 saddler 共用网络字段，原版明确禁止两组件同驻。[inventoryitem_replica.lua:33](../../../raw/current/scripts/components/inventoryitem_replica.lua#L33)、[:391](../../../raw/current/scripts/components/inventoryitem_replica.lua#L391)
- **精神单位或适用范围混淆**：不是统一 `/54`；sanity 用贡献汇总的 rate 乘 dt。dapperfn 覆盖基础／鱼人翻转值，湿惩罚仍在后面；角色可替换消费规则，携带不等于装备。[equippable.lua:196](../../../raw/current/scripts/components/equippable.lua#L196)、[sanity.lua:510](../../../raw/current/scripts/components/sanity.lua#L510)、[:609](../../../raw/current/scripts/components/sanity.lua#L609)
- **限制标签拼错／反义判断**：用 `restrictedtag`，true 表示不能装备；构造器里的 `retrictedtag` 只是未执行注释，正确属性在 Class 映射中存在。[equippable.lua:36](../../../raw/current/scripts/components/equippable.lua#L36)、[:51](../../../raw/current/scripts/components/equippable.lua#L51)、[:147](../../../raw/current/scripts/components/equippable.lua#L147)
- **移组件等于卸装**：OnRemoveFromEntity 解除锁、重置 replica 速度和限制，却不调用户卸装回调或清人物装备槽；自建灯光／监听／修饰须有匹配清理。实体删除走 onremove 与 OnRemoveEntity，不自动调此钩子。[equippable.lua:55](../../../raw/current/scripts/components/equippable.lua#L55)、[entityscript.lua:670](../../../raw/current/scripts/entityscript.lua#L670)、[:1730](../../../raw/current/scripts/entityscript.lua#L1730)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 4 处（constants EQUIPSLOTS +2、mainfunctions SpawnPrefabFromSim +26、hats.lua scrap_monocle 段 +7、entityscript onremove 段 +3）；无结论修订，仅锚点迁移——equippable.lua 及各 replica 未变；torch.lua 本版改动在 324 行之后，130/166/188 引用未变；locomotor.lua 改动在 845 行之后，188 引用未变。依据：全局枢纽 diff 审阅＋变更清单命中 constants.lua、tuning.lua、torch.lua、hats.lua、locomotor.lua、entityscript.lua、mainfunctions.lua。
- 源码审阅：2026-10-11，Steam buildid 25643519；覆盖此页接口、属性／调用顺序／入袋 nil 与手杖接入。正式 reviews 记录 17 个字段、18 个真实方法的原版约定；依赖 `ent` 仍为兼容编辑器模型，未证明全部引擎／模组类型。
- 类型与结构：[固定声明](https://github.com/YingSP/lsp-dst/blob/1db05561ea6ea271093ea780b60d49e7499baf40/scripts/components/members/equippable.lua) 由同一 reviews 导出并发布，LuaLS 3.19.1／Lua 5.1 声明和正例无诊断，六处故意错误命中；页面元数据、链接／大小检查通过。普通查询不要求克隆类型库。
- 游戏运行：未运行；上述代码仅源码审阅。客户端预测、多人显示、骑乘、角色精神差异、存档重载与模组兼容仍需实际游戏验证。

相关：[[reference/components/sanity#给人物增加每分钟三点精神贡献|独立精神来源]]；[[reference/components/health#装备手杖每十秒回复一点生命|装备再生与撤销]]；[[reference/components/inventoryitem#生命周期|持有者变化]]；[[reference/components/inventory#携带范围的选择|装备与主栏范围]]；[[reference/components/finiteuses|按次消耗]]；[[reference/components/waterproofer|防雨]]；[[reference/components/insulator|温度保温]]；[[reference/components/armor|护甲]]；[[reference/components/locomotor#关键接口|独立移动倍率]]。
