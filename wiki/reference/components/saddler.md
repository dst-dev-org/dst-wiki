---
title: "saddler"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "为鞍物品提供骑乘速度、攻击、防御及外观参数，区分人物装备和鞍丢弃耐久。"
aliases: ["普通牛鞍速度倍率", "鞍附加伤害", "鞍减伤", "saddler"]
tags: [component, equipment, network, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/saddler.lua", "prefabs/saddle.lua", "components/locomotor.lua", "components/combat.lua", "components/rideable.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# saddler

## 快速判断

- 职责与适用：鞍物品侧存加速／攻击／防御与替换符号；上鞍／坐骑保存交给 [[reference/components/rideable]]，人物装备交给 [[reference/components/equippable]]。[配置](../../../raw/current/scripts/components/saddler.lua#L13-L59)
- 优先使用：SetBonusSpeedMult、SetBonusDamage、SetAbsorption 分别配置；默认读取速度 1、伤害 0、吸收 0，不需要给鞍补 equippable。[取值](../../../raw/current/scripts/components/saddler.lua#L45-L55)
- 重要副作用：速度属性通过 inventoryitem replica 的 WalkSpeedMult 通道同步；源码明确同一 Prefab 不得同时有 saddler 与 equippable，两者共享此通道。[同步约束](../../../raw/current/scripts/components/saddler.lua#L5-L10)
- 修改边界：加速由实际骑乘移动消费者读取，普通收纳不生效；防御处理不自动扣鞍耐久，丢弃回调由宿主配置。[移动消费](../../../raw/current/scripts/components/locomotor.lua#L178-L185) [原鞍丢弃](../../../raw/current/scripts/prefabs/saddle.lua#L2-L8)

## 关键接口

| 接口 | 条件、作用与证据 |
|---|---|
| `SetSwaps(build, symbol, skin_guid)` | 记录符号／皮肤参数，上鞍与人物骑乘消费；不创建资源或替代原鞍工厂。[设置](../../../raw/current/scripts/components/saddler.lua#L27-L31) [坐骑显示](../../../raw/current/scripts/components/rideable.lua#L136-L153) |
| `SetBonusSpeedMult(mult)`／`GetBonusSpeedMult()` | 完整乘数，1.4 表示乘 1.4，不是额外加 1.4；属性回调同步库存 replica，getter 缺省 1。[方法](../../../raw/current/scripts/components/saddler.lua#L37-L50) [移动消费](../../../raw/current/scripts/components/locomotor.lua#L178-L185) |
| `SetBonusDamage(damage)`／`GetBonusDamage(target)` | getter 返回常数或 0，当前实现不使用 target；骑乘战斗消费将其加至 basedamage，不承诺所有远程／特殊伤害分支等价。[方法](../../../raw/current/scripts/components/saddler.lua#L33-L46) [消费者](../../../raw/current/scripts/components/combat.lua#L922-L928) |
| `SetAbsorption(percent)`／`ApplyDamage(damage, attacker, weapon, spdamage)` | setter 不限幅，正常配置用 0～1；先按鞍 damagetyperesist 调整普通伤害，再按吸收计算剩余。特殊伤害逐类型扣防御并原地改输入表，返回剩余普通／特殊伤害；不是直接扣生命，也不自动 Use 耐久。[实现](../../../raw/current/scripts/components/saddler.lua#L61-L89) [坐骑受击消费](../../../raw/current/scripts/components/combat.lua#L635-L642) |
| `SetDiscardedCallback(cb)` | cb(saddle)；rideable 在无 doer 丢弃旧鞍时监听落地触发，原鞍回调扣 finiteuses。不同于所有卸鞍、受击或每秒骑乘都扣耐久。[设置](../../../raw/current/scripts/components/saddler.lua#L57-L59) [触发](../../../raw/current/scripts/components/rideable.lua#L44-L49) [监听条件](../../../raw/current/scripts/components/rideable.lua#L113-L133) |

原鞍 SetupSaddler 用实例 _data 设置参数；影鞍破损会移除组件、修复后重新 SetupSaddler，初始化一次的覆盖可能因此丢失。本页示例只针对普通鞍，不外推修复变体。组件本身无普通 OnSave，参数随工厂配置；耐久、修复与鞍记录另由宿主和 rideable／domesticatable 保存。[配置](../../../raw/current/scripts/prefabs/saddle.lua#L242-L254) [破损／修复](../../../raw/current/scripts/prefabs/saddle.lua#L17-L35)

（26148036 核验：combat 受击链在攻击者"穿在受害者身上"（inventory IsItemEquipped，slurper／bat_boss 类）时跳过鞍防御 ApplyDamage；骑乘加速消费位置未变，locomotor 新增 `do_not_locomote` 支持并移除 busy 期间转向限制，均不影响本页结论。）

## 典型接入

### 普通牛鞍骑乘速度倍率设为 1.4

目标模组 `modmain.lua`，复用 saddle_basic 原资源与耐久，只改该实例组件，不修改共享 _data／TUNING 表。[工厂](../../../raw/current/scripts/prefabs/saddle.lua#L290-L308) [注册](../../../raw/current/scripts/prefabs/saddle.lua#L407-L411)

```lua
AddPrefabPostInit("saddle_basic", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local saddler = inst.components.saddler
    if saddler ~= nil then
        saddler:SetBonusSpeedMult(1.4)
    end
end)
```

1.4 是鞍这一项的乘数，最终仍与其他移动贡献合并，不等于固定移动速度。新建／重载重新应用，停用重载恢复原配置；未给在用的其他鞍变体改数据，也不重写存档耐久。上鞍、鞍跟踪与上下骑见 [[reference/components/rideable]]／[[reference/components/rider]]。

## 常见陷阱与限制

- **把鞍当人物普通装备并再加 equippable**：共享同步字段会冲突，遵守源码约束，功能放 saddler，坐骑接入放 rideable。[约束](../../../raw/current/scripts/components/saddler.lua#L5-L10)
- **普通吸收百分比同样减免全部特殊伤害**：特殊伤害走单独防御类型，输入表还可能被修改；不能照搬 armor 的完整受击结论。[实际处理](../../../raw/current/scripts/components/saddler.lua#L61-L89)
- **修复后的组件仍保留 PostInit 一次赋值**：宿主可以卸载并重新配置组件；变体应核验实际重建入口，本例不修改影鞍。[修复](../../../raw/current/scripts/prefabs/saddle.lua#L17-L35)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件对照 build-26148036 更新报告逐条核验，锚点重定位 2 处、结论修订 0 处（combat 穿身豁免核验，见正文"26148036 核验"标注）；未覆盖：运行验证。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖参数／同步约束、实际移动和战斗消费者、丢弃耐久、普通鞍工厂及影鞍重建边界。未全面审校所有战斗分支、特殊防御实现、皮肤和修复变体。
- 结构检查：本批格式、来源、链接／行号、大小与登记通过；完整例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；实际速度／预测、上下鞍、受击／丢弃、存读与其他模组参数覆盖待实测。
