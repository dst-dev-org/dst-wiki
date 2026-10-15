---
title: "weighable"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "维护鱼或巨型作物的重量、范围比例与捕获者记录，衔接称重、奖杯比较和形态转存。"
aliases: ["物品重量", "鱼重量", "重量比例", "捕获者记录", "weighable"]
tags: [component, item, data, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/weighable.lua", "prefabs/pondfish.lua", "prefabs/oceanfish.lua", "components/oceanfishingrod.lua", "components/trophyscale.lua", "actions.lua", "stategraphs/SGwilson.lua", "mainfunctions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# weighable

## 快速判断

- 职责与适用：保存实体重量、范围比例和捕获者信息；不改变物理质量、食物收益或角色搬运能力。便携称重工具用 [[reference/components/itemweigher]]。[完整实现](../../../raw/current/scripts/components/weighable.lua#L1-L96)
- 优先使用：先 `Initialize(min, max)` 再 `SetWeight(value)`；用 `GetWeight()`／`GetWeightPercent()` 读取。`type` 决定动作标签，重量本身不决定能否称量。[类型与比例](../../../raw/current/scripts/components/weighable.lua#L1-L31) [接口](../../../raw/current/scripts/components/weighable.lua#L45-L60)
- 重要副作用：SetWeight 向下截到两位小数，重算并夹紧**比例**；不会把实际重量夹到上下限。捕获者由独立接口设置，不等同当前物品持有者。[设重量](../../../raw/current/scripts/components/weighable.lua#L53-L71)
- 修改边界：原版鱼在服务端初始化；水中海鱼可只保存数据而不设称重类型，库存形态才设 fish 类型。不要把每个有组件的实体都当可在背包称量的物品。[水中形态](../../../raw/current/scripts/prefabs/oceanfish.lua#L371-L376) [库存形态](../../../raw/current/scripts/prefabs/oceanfish.lua#L522-L525)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `type` | 默认 nil；类别字符串 | 变更维护 `weighable_<type>` 标签；移组件清当前类型标签。与工具／奖杯台的类型匹配分开配置。[标签](../../../raw/current/scripts/components/weighable.lua#L1-L7) [清理](../../../raw/current/scripts/components/weighable.lua#L33-L37) |
| `Initialize(min_weight, max_weight)` | 数值上下限 | 只赋上下限，不重算已有比例；调用方应提供有效且严格递增的范围，代码不校验。[实现](../../../raw/current/scripts/components/weighable.lua#L53-L56) [比例计算](../../../raw/current/scripts/components/weighable.lua#L10-L12) |
| `SetWeight(weight)` | 数值；无返回 | 写入 `floor(weight*100)/100`；weight 属性回调以范围映射比例并夹到 0～1。无完整上下限时比例为 0.5；构造中的初始比例仍是 nil。[构造](../../../raw/current/scripts/components/weighable.lua#L14-L31) [setter](../../../raw/current/scripts/components/weighable.lua#L58-L60) |
| `GetWeight()`／`GetWeightPercent()` | 当前重量／比例，可为 nil | 单纯读字段；不现场补生成、随机化或重算。玩家称重 SG 使用两者，完整初始化后再开放交互。[getter](../../../raw/current/scripts/components/weighable.lua#L45-L51) [消费](../../../raw/current/scripts/stategraphs/SGwilson.lua#L9984-L9995) |
| `SetPlayerAsOwner(owner)` | 玩家或 nil；无返回 | 先清 `prefab_override_owner`，再复制 userid／name，或清两者；没有归属锁定或持有权限校验。[实现](../../../raw/current/scripts/components/weighable.lua#L62-L72) |
| `CopyWeighable(src)` | 源组件，可为 nil | 用源 OnSave 数据调用目标 OnLoad，复制重量及三种归属字段；不复制 type、上下限、回调或实体本身。[复制](../../../raw/current/scripts/components/weighable.lua#L39-L43) [存读字段](../../../raw/current/scripts/components/weighable.lua#L74-L90) |

存档保存 weight、owner_userid、owner_name、prefab_override_owner；加载直接赋保存重量并触发比例计算，**不经过 SetWeight 的截位**。type 和上下限要由目标工厂先恢复；不能把 CopyWeighable 当作完整组件配置迁移。[存读](../../../raw/current/scripts/components/weighable.lua#L74-L90)

## 典型接入

### 把新生成的池塘鱼初始重量设为上限

完整目标模组 `modmain.lua`，只改 `pondfish`，沿用原鱼资源、类型和范围。原版先初始化范围，再随机赋重量；此入口在构造之后将其改为已配置的上限，不改鳗鱼或海鱼。[池塘鱼初始化](../../../raw/current/scripts/prefabs/pondfish.lua#L144-L148) [数据与工厂](../../../raw/current/scripts/prefabs/pondfish.lua#L161-L175) [注册名](../../../raw/current/scripts/prefabs/pondfish.lua#L195-L196)

```lua
AddPrefabPostInit("pondfish", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local weighable = inst.components.weighable
    if weighable ~= nil and weighable.max_weight ~= nil then
        weighable:SetWeight(weighable.max_weight)
    end
end)
```

数值沿用原版重量标度，未建立物理质量单位换算。只改变**初始值**：读档先生成实体、后恢复组件数据，因此旧鱼的保存重量仍会覆盖这个初始化值；新生成后已保存的鱼则恢复其保存值。移除代码不追溯改写现存／已保存重量。[生成时 PostInit](../../../raw/current/scripts/mainfunctions.lua#L387-L404) [存档恢复时序](../../../raw/current/scripts/mainfunctions.lua#L464-L466) [恢复数据](../../../raw/current/scripts/mainfunctions.lua#L503) [组件加载](../../../raw/current/scripts/components/weighable.lua#L83-L90)

### 海鱼形态转换与实际捕获者

原版海钓成功后对投射形态调用 `SetPlayerAsOwner(fisher)`；落在陆地时生成对应 `_inv` 物品并 `CopyWeighable`，然后清旧形态归属并移除它。水中与库存工厂各自初始化类型／范围，复制只负责记录字段。[海钓归属](../../../raw/current/scripts/components/oceanfishingrod.lua#L337-L349) [陆地转换](../../../raw/current/scripts/prefabs/oceanfish.lua#L102-L116) [两种工厂注册](../../../raw/current/scripts/prefabs/oceanfish.lua#L597-L604)

奖杯台比较也读取重量和归属，但使用独立 `trophyscale:Compare`，成功接纳会记录数据并消耗输入的一件／整实体；不能把比较理解为便携秤的无损播报。本页仅定位这条消费链，不覆盖完整奖杯台机制。[比较与消费](../../../raw/current/scripts/components/trophyscale.lua#L57-L86) [动作分支](../../../raw/current/scripts/actions.lua#L4666-L4676)

## 常见陷阱与限制

- **先设重量、后改范围，比例仍旧：** Initialize 不触发 weight 属性计算。工厂按 Initialize→SetWeight 初始化，不只改 min_weight／max_weight 就假设消费者已刷新。[实现](../../../raw/current/scripts/components/weighable.lua#L10-L12) [初始化接口](../../../raw/current/scripts/components/weighable.lua#L53-L60)
- **把截位当四舍五入／合法范围保护：** floor 是向下截取；只有比例夹到 0～1，重量可以超出范围。自定义来源须校验数值及范围，不指望 setter 代做。[计算](../../../raw/current/scripts/components/weighable.lua#L10-L12) [赋值](../../../raw/current/scripts/components/weighable.lua#L58-L60)
- **把捕获者等同物品主人：** 这是可保存的名字／userid 记录，不是 inventoryitem.owner。生物掉落可用 prefab_override_owner 标注来源；SetPlayerAsOwner 会清这个覆盖字段。[归属接口](../../../raw/current/scripts/components/weighable.lua#L62-L72) [生物掉落记录](../../../raw/current/scripts/prefabs/pondfish.lua#L54-L58)
- **复制时丢掉类型或沿用错误比例：** Copy 不复制目标范围和 type；先按新形态配置再复制。完整对象替换、资源与物品交付另由宿主负责，详见上文海鱼转换。[复制字段](../../../raw/current/scripts/components/weighable.lua#L39-L43) [存读](../../../raw/current/scripts/components/weighable.lua#L74-L90)
- **把普通字段当客户端重量副本：** 组件没有 netvar；原版客户端依靠类型标签发现动作，重量播报由服务端 SG 发起。新需求若需本地实时显示，须另定同步方案。[组件](../../../raw/current/scripts/components/weighable.lua#L14-L31) [原版共同标签／服务端门禁](../../../raw/current/scripts/prefabs/pondfish.lua#L95-L104) [服务端播报](../../../raw/current/scripts/stategraphs/SGwilson.lua#L9984-L9995)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的变更文件（actions.lua、SGwilson.lua、mainfunctions.lua）对照 build-26148036 更新报告条目 R-01／R-04 逐条核验，引用区域内容均未变化，锚点重定位 6 处，无结论修订；未覆盖：运行验证。

- 源码审阅：2026-10-23，Steam buildid 25643519；覆盖完整组件、池塘鱼注册／范围初始化、海鱼两形态工厂及海钓归属／转换、玩家称重消费、奖杯台比较入口与生成／读档时序；未全量审校鱼种、巨型作物、奖杯台或所有重量消费者。
- 游戏运行：未运行；新鱼重量、旧鱼读档、联机播报和跨模组改范围待实测。
- 结构检查：本页格式／链接、源码锚点严格检查及完整 Lua 5.1 示例语法检查通过；不代替游戏验证。
