---
title: "upgrademodule"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "WX-78 模块侧的槽位／类型、宿主绑定与激活回调；容量、实体暂存和电量排序归模块宿主。"
aliases: ["健康电路激活时额外加速", "WX78模块激活与撤销", "upgrademodule"]
tags: [component, item, callback, lifecycle, energy, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/upgrademodule.lua", "components/upgrademoduleowner.lua", "prefabs/wx78_modules.lua", "wx78_moduledefs.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# upgrademodule

## 快速判断

- 职责与适用：模块物品保存 slots／type／target／activated，通知添加、激活、撤销和移出；原工厂按 wx78_moduledefs 注册 wx78module_<name>。材料式消耗升级另见 [[reference/components/upgrader]]。[组件](../../../raw/current/scripts/components/upgrademodule.lua#L1-L25) [原工厂](../../../raw/current/scripts/prefabs/wx78_modules.lua#L72-L97)
- 优先使用：扩展原模块效果包 onactivatedfn／ondeactivatedfn，成对增删并保留 isloading；安装和电量资格交 [[reference/components/upgrademoduleowner]]，不要直接改 activated 当作施加效果。[激活与撤销](../../../raw/current/scripts/components/upgrademodule.lua#L38-L64) [宿主排序](../../../raw/current/scripts/components/upgrademoduleowner.lua#L181-L190)
- 重要副作用：TryActivate 先置 activated=true 再调用效果；TryDeactivate 先置 false 再调用撤销，重复同状态不重放。SetTarget(nil) 不自动撤销，RemoveFromOwner 也不是完整宿主拆卸链。[模块方法](../../../raw/current/scripts/components/upgrademodule.lua#L22-L64)
- 修改边界：服务端；源码注明激活／撤销应由 owner 调用。模块存在、已安装、能量足够与效果激活是不同条件；原宿主按槽位顺序决定激活。[约定](../../../raw/current/scripts/components/upgrademodule.lua#L37-L54) [消费电量前缀](../../../raw/current/scripts/components/upgrademoduleowner.lua#L181-L190)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `SetRequiredSlots(n)`／`GetSlots()` | 默认一格，setter 只赋值；原工厂同时在客户端静态 _slots 保存定义值。服务端单改 slots 不会同步改 UI／候选数据。[组件](../../../raw/current/scripts/components/upgrademodule.lua#L1-L19) [工厂静态数据](../../../raw/current/scripts/prefabs/wx78_modules.lua#L54-L74) |
| `SetType(bar)`／`GetType()` | 默认 CIRCUIT_BARS.ALPHA；要匹配 owner.module_bars。原 _type／_netid 和注册数据也在工厂，不由 setter 注册新网络类型。[方法](../../../raw/current/scripts/components/upgrademodule.lua#L29-L34) [工厂](../../../raw/current/scripts/prefabs/wx78_modules.lua#L54-L76) |
| `SetTarget(target,isloading?)` | 先存 target；非 nil 时调 onaddedtoownerfn(module,target,isloading)，不检测是否已添加，也不处理库存／父子关系。[实现](../../../raw/current/scripts/components/upgrademodule.lua#L22-L25) |
| `TryActivate(isloading?)`／`TryDeactivate()` | 状态改变才调 onactivatedfn(module,target,isloading) 或 ondeactivatedfn(module,target)；不做能量／目标有效性验证，也不以回调 false 回滚。[实现](../../../raw/current/scripts/components/upgrademodule.lua#L38-L55) |
| `RemoveFromOwner()` | 保存旧 target、SetTarget(nil)，再 onremovedfromownerfn(module,oldtarget)。正常 owner 在此之前已 TryDeactivate，故效果回调仍可访问旧 target。[模块](../../../raw/current/scripts/components/upgrademodule.lua#L59-L64) [宿主顺序](../../../raw/current/scripts/components/upgrademoduleowner.lua#L227-L252) |

原模块工厂移出回调扣 finiteuses，交换模块时可免除；所有移出与能量不足并不等价，停电的 TryDeactivate 不应额外扣拆卸次数。组件本身不保存 target／activated，owner 保存实体记录并在加载时 PushModule(...,true) 恢复。[耐久](../../../raw/current/scripts/prefabs/wx78_modules.lua#L10-L18) [保存恢复](../../../raw/current/scripts/components/upgrademoduleowner.lua#L403-L451)

## 典型接入

### 原健康电路激活时额外提供 1.1 倍移动贡献，失活成对撤销

目标模组 `modmain.lua`，只改原 wx78module_maxhealth，保留原增血和技能回调；不新增模块注册／资源／槽位。该注册由 name=maxhealth 的定义和通用工厂拼接确认。[定义](../../../raw/current/scripts/wx78_moduledefs.lua#L199-L218) [注册](../../../raw/current/scripts/prefabs/wx78_modules.lua#L97)

```lua
AddPrefabPostInit("wx78module_maxhealth", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local module = inst.components.upgrademodule
    if module == nil then return end
    local activate = module.onactivatedfn
    local deactivate = module.ondeactivatedfn
    module.onactivatedfn = function(item, owner, isloading)
        if activate ~= nil then activate(item, owner, isloading) end
        if owner ~= nil and owner.components.locomotor ~= nil then
            owner.components.locomotor:SetExternalSpeedMultiplier(item, "my_mod_circuit", 1.1)
        end
    end
    module.ondeactivatedfn = function(item, owner)
        if owner ~= nil and owner.components.locomotor ~= nil then
            owner.components.locomotor:RemoveExternalSpeedMultiplier(item, "my_mod_circuit")
        end
        if deactivate ~= nil then deactivate(item, owner) end
    end
end)
```

复用 [[reference/components/locomotor#关键接口]] 的来源实体／键协议：同一模块重复设置会替换，多枚来源按该机制相乘；1.1 是移动倍率贡献，不保证最终速度恰好快 10%。加载时也应重施自定义贡献，原回调仍收到 isloading，不能无条件改写原生命恢复逻辑。

能量不足／正常拔出由 owner 撤销；停用并重载后不再安装自定义回调，原模块效果和耐久保留。仅手动清 target、移除组件或跳过 owner 的拆卸不属于本例支持的清理路径。删除来源的移动贡献清理按 locomotor 页，不视作整个模块系统已恢复。

## 常见陷阱与限制

- **SetTarget／TryActivate 就完成安装**：库存移出、父子隐藏、排序和通知都在 PushModule；模块方法不能替代完整安装资格。[宿主](../../../raw/current/scripts/components/upgrademoduleowner.lua#L207-L223)
- **SetTarget(nil) 会撤销能力**：它不调用 TryDeactivate；正常 PopModule 先撤能力、再清 target，不能倒序。[模块](../../../raw/current/scripts/components/upgrademodule.lua#L22-L25) [正常顺序](../../../raw/current/scripts/components/upgrademoduleowner.lua#L227-L252)
- **仅修改 slots／type 就有一致 UI 和新模块**：原静态 _slots／_type／_netid、定义与候选另有责任；本页不提供新电路注册方案。[工厂](../../../raw/current/scripts/prefabs/wx78_modules.lua#L54-L76)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 0 处（唯一命中变更清单的 wx78_moduledefs.lua 引用区间未在本版改动）；结论修订：无，仅登记版本。
- 依据：枢纽 diff 审阅（actions.lua／componentactions.lua hunk 地图）＋变更清单命中 wx78_moduledefs.lua（改动位于 L648／L1598，页引 L199-L218 未变）。

- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖完整模块、owner 的绑定／激活／移出／保存恢复、原工厂／耐久和健康电路定义／加载参数。移动来源协议复用同版本 locomotor 页；未完整审校技能树、UI／网络类型注册、所有模块效果或异常拆卸。
- 结构检查：本批格式、来源、链接／锚点、大小与登记；完整成对加速例经已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；装入未激活、充放电、多个模块叠加、拔出／交换、死亡／读档和预测移动待实测。
