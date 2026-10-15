---
title: "upgrademoduleowner"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "WX-78 多栏模块实体、按电量前缀激活、拆卸与嵌套保存；容量、归还及同步依赖原宿主。"
aliases: ["服务端给WX78补一格电", "WX78模块宿主", "模块安装与电量", "upgrademoduleowner"]
tags: [component, lifecycle, callback, energy, network, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/upgrademoduleowner.lua", "components/upgrademodule.lua", "prefabs/wx78.lua", "prefabs/wx78_modules.lua", "actions.lua", "constants.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# upgrademoduleowner

## 快速判断

- 职责与适用：宿主保存按 CIRCUIT_BARS 分组的模块实体、实际电量和检视状态，按顺序激活／撤销 [[reference/components/upgrademodule]]；原 WX-78 另配容量测试、归还、拆卸成本和 classified 同步。[组件](../../../raw/current/scripts/components/upgrademoduleowner.lua#L43) [原挂载](../../../raw/current/scripts/prefabs/wx78.lua#L767-L774)
- 优先使用：充放电用 SetChargeLevel／DoDeltaCharge；正常安装走原 CanUpgrade→PushModule，正常按件拔出走 FindAndPopModule／PopOneModule，不直接改 module_bars。[电量](../../../raw/current/scripts/components/upgrademoduleowner.lua#L335-L345) [动作](../../../raw/current/scripts/actions.lua#L5792-L5804)
- 重要副作用：PushModule 移出库存、绑定主人、AddChild／RemoveFromScene 并重算激活；PopModule 恢复场景、撤销能力、宿主返还、清绑定／扣模块耐久，再重算。不是纯表操作。[安装](../../../raw/current/scripts/components/upgrademoduleowner.lua#L207-L223) [移出](../../../raw/current/scripts/components/upgrademoduleowner.lua#L227-L255)
- 修改边界：服务端；直接 PushModule 不自动检查容量，CanUpgrade 默认 true，原 WX 宿主回调才检查空间。其他角色只挂组件不会自动获得 UI／classified／动画／效果兼容。[资格](../../../raw/current/scripts/components/upgrademoduleowner.lua#L171) [原容量](../../../raw/current/scripts/prefabs/wx78.lua#L445-L452)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `GetModules(bar)`／`GetModule(bar,index)`／`GetNumModules(bar)`／`GetAllModules()` | 前者给内部栏表，别直接增删；后者聚合模块列表。bar 必须是已有 CIRCUIT_BARS 键，当前 ALPHA=0／BETA=1／GAMMA=2，非一基通用格子号。[查询](../../../raw/current/scripts/components/upgrademoduleowner.lua#L143-L166) [常量](../../../raw/current/scripts/constants.lua#L3022-L3026) |
| `GetUsedSlotCount(bar)`／`GetModuleTypeCount(name)` | 前者总占槽；后者按电量前缀统计 wx78module_<name>，并非遍历 activated 布尔状态，自动激活被禁时不能据此当实际生效数。[统计](../../../raw/current/scripts/components/upgrademoduleowner.lua#L114-L140) |
| `CanUpgrade(module)`／`PushModule(bar?,module,isloading?)` | 前者只委托 canupgradefn，无回调时 true；后者不重复资格校验／去重。原动作先查宿主，再把真实模块移出库存、推入栏。[接口](../../../raw/current/scripts/components/upgrademoduleowner.lua#L171) [安装](../../../raw/current/scripts/components/upgrademoduleowner.lua#L207-L223) |
| `UpdateActivatedModules(isloading?)` | 每栏从有效电量重新算剩余，按模块槽位依次扣；不足则 TryDeactivate，足够且允许自动激活才 TryActivate。不跨栏共享扣减后的局部剩余值。[算法](../../../raw/current/scripts/components/upgrademoduleowner.lua#L181-L190) |
| `SetAutomaticModuleActivations(enabled)` | false 只阻止后续自动激活；当前能量足够的已激活模块不会因此全部撤销，不能当总停机开关。[方法](../../../raw/current/scripts/components/upgrademoduleowner.lua#L181-L201) |
| `PopModule(bar,index?)`／`PopOneModule(bar,index?)` | PopModule 返回 module,was_activated；PopOneModule 再调 ononemodulepopped，固定返回 0 是旧兼容值。原 WX 在额外回调扣电量／同步，不能把返回零理解为免费，也不再另扣一次。[接口](../../../raw/current/scripts/components/upgrademoduleowner.lua#L227-L255) [按件](../../../raw/current/scripts/components/upgrademoduleowner.lua#L296-L307) [原成本](../../../raw/current/scripts/prefabs/wx78.lua#L408-L426) |
| `FindAndPopModule(ent)`／`PopAllModules(bar?)` | 前者定位后按件拔出；后者逐模块走 PopModule 并调 onallmodulespopped，不等于多次 PopOneModule。不要假定拆卸成本与回调完全相同。[实现](../../../raw/current/scripts/components/upgrademoduleowner.lua#L258-L307) |
| `SetChargeLevel(n)`／`DoDeltaCharge(n)` | 写实际电量并夹到 0～max_charge；有效值变化时重算激活，属性还发 energylevelupdate 并同步 classified。直接写 charge_level 不等价于完整重算。[电量](../../../raw/current/scripts/components/upgrademoduleowner.lua#L335-L345) [通知](../../../raw/current/scripts/components/upgrademoduleowner.lua#L3-L14) |
| `SetMaxCharge(n)`／`SetOverrideFullCharge(bool)` | max 同时决定容量，降低时按件弹超额模块，未在该方法直接夹实际电量；override 改 GetChargeLevel 的有效值，不改保存的真实充电值。[上限](../../../raw/current/scripts/components/upgrademoduleowner.lua#L312-L340) [有效电量](../../../raw/current/scripts/components/upgrademoduleowner.lua#L349-L375) |
| `StartInspecting(who)`／`StopInspecting()` | 更新 inspecter／标签／classified，按实际 HUD 条件开关；完整人物动作／SG 退出走宿主入口，拆卸工具见 [[reference/components/upgrademoduleremover]]。[方法](../../../raw/current/scripts/components/upgrademoduleowner.lua#L83-L109) |

### 保存与宿主恢复

OnSave 记录真实 charge_level 与每栏模块 GetSaveRecord／引用，OnLoad SpawnSaveRecord 后 PushModule(...,true)，会重放装入／激活回调。max_charge、自动激活开关和 override 不在该保存对象内。原 WX 的人物 OnLoad 再恢复受电路上限影响的当前三维值；自定义角色不能只照搬组件存读。[组件保存](../../../raw/current/scripts/components/upgrademoduleowner.lua#L403-L451) [原人物](../../../raw/current/scripts/prefabs/wx78.lua#L181-L206)

组件移除只清两个标签，不自动 PopAllModules 或撤能力；宿主删除／换身／备份身体的完整去向需另核验。本页只给精选原 WX 接入，不提供自定义多栏机器人工程。[移除](../../../raw/current/scripts/components/upgrademoduleowner.lua#L76-L79)

## 典型接入

### 原 WX-78 收到服务端奖励事件时补一格实际电量

目标模组 `modmain.lua`，复用原角色／分类同步／电路显示；私有事件由已授权服务端逻辑发，不提供客户端无限充电入口。保持原 SetChargeLevel 夹限和激活重算。[原挂载](../../../raw/current/scripts/prefabs/wx78.lua#L767-L774) [增量入口](../../../raw/current/scripts/components/upgrademoduleowner.lua#L335-L345)

```lua
AddPrefabPostInit("wx78", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:ListenForEvent("my_mod_charge_one", function(player)
        local modules = player.components.upgrademoduleowner
        local health = player.components.health
        if modules == nil or health == nil or health:IsDead()
            or player:HasTag("playerghost") then return end
        modules:DoDeltaCharge(1)
    end)
end)
```

服务端发 `player:PushEvent("my_mod_charge_one")`；1 是电量格数，不是秒或百分比，满实际电量时不额外增长。override 满电时有效值可能未变，真实值仍按原规则增加，不据此重复发自定义激活事件。[实际／有效区别](../../../raw/current/scripts/components/upgrademoduleowner.lua#L335-L375)

新实际电量随原组件保存，停用并重载不回滚已补电量，仅不再安装监听。本例不增加 max_charge／模块容量、不替换自动充电／耗能机制，也不保证仅凭电量变化就能安装任意模块。

## 常见陷阱与限制

- **NumModules() 判断当前装了几个**：旧兼容方法恒返回 0，改用有 bar 的 GetNumModules 或 GetAllModules；不要以旧 REMOVEMODULES 链复制新玩法。[旧方法](../../../raw/current/scripts/components/upgrademoduleowner.lua#L165-L166) [旧动作](../../../raw/current/scripts/actions.lua#L5822-L5834)
- **PopModule／PopOneModule／PopAllModules 可互换**：回调、返回／返还、拆卸耐久和单件电量成本有分工；正常用户拔出链不要跳过宿主回调。[原返还／成本](../../../raw/current/scripts/prefabs/wx78.lua#L397-L426) [耐久](../../../raw/current/scripts/prefabs/wx78_modules.lua#L10-L18)
- **修改 charge_level 或关自动激活可完整控制效果**：属性通知不代替重算，关自动激活不强制撤销已有能力；走对应接口并检查具体语义。[调度](../../../raw/current/scripts/components/upgrademoduleowner.lua#L181-L201) [电量接口](../../../raw/current/scripts/components/upgrademoduleowner.lua#L335-L345)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 3 处（actions APPLYMODULE／REMOVEMODULES 与 constants CIRCUIT_BARS 行号迁移，区间文本逐段比对一致；wx78.lua 本版仅改进食配方，页引区间未变）；结论修订：无，仅锚点迁移。
- 依据：枢纽 diff 审阅（actions.lua／componentactions.lua hunk 地图）＋变更清单命中 actions.lua、constants.lua、prefabs/wx78.lua。

- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖完整 owner／模块核心、APPLYMODULE、原 WX 容量／返还／电量成本／同步和保存三维恢复、模块耐久。未全面审校技能／影子插槽、备份身体交换、全部 UI／RPC 校验、死亡／换人／跨分片或容量降低异常。
- 结构检查：本批格式、来源、链接／锚点、大小与登记；完整单格充电例经已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；充放电前缀、满电 override、按件／整栏拔出、读档、死亡与其他模组同步待实测。
