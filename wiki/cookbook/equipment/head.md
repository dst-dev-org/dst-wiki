---
title: 新增头部装备
created: 2026-10-11
updated: "2026-10-28"
type: cookbook
summary: 注册 HEAD 槽帽子，复用草帽资源，配对帽子与头发显示及卸装恢复，按需扩展头盔防御、防雨、隔热和护目镜能力，可省略穿戴外观
aliases: [新增帽子, 头部装备, 帽子骨架, 头盔骨架, 护目镜骨架, 无穿戴外观帽子, equipment/head, dstwiki_hat]
tags: [cookbook, item, equipment, prefab, modding, server-client]
dst_build: "steam-buildid:26230496"
sources: [prefabs/hats.lua, components/equippable.lua, components/sanity.lua, components/inventory.lua, components/inventoryitem.lua, tuning.lua, mods.lua, consolecommands.lua]
reliability: verified
validation: source-reviewed
status: active
---

# 新增头部装备

## 适用条件

已有 DST 世界／服务器模组，新增可拾取并装备到 **HEAD** 槽的帽子；头盔、护目镜按需加功能和选择外观模式。已有工程沿用注册约定，尚无入口从 [[cookbook/index|场景导航]] 选择起步。只改已有装备数值直接读 [[reference/components/equippable#关键接口|装备机制]]。

本例 `dstwiki_hat` 独立工厂复用原版草帽地面／穿戴外观与图标，装备时提供 **3 / 60／秒**的精神基础贡献，卸下或仅携带不提供此项；角色、潮湿、环境等仍影响实际净变化。没有耐久、防御、防雨或隔热，借用外观不会继承原版草帽组件。[hats.lua:188](../../../raw/current/scripts/prefabs/hats.lua#L188)、[:242](../../../raw/current/scripts/prefabs/hats.lua#L242)、[sanity.lua:510](../../../raw/current/scripts/components/sanity.lua#L510)、[equippable.lua:196](../../../raw/current/scripts/components/equippable.lua#L196

## 最小 demo

### 文件与加载

保留已有入口，新增工厂：

```text
<目标模组>/
├── modinfo.lua
├── modmain.lua
└── scripts/prefabs/dstwiki_hat.lua
```

在已有 `modinfo.lua` 保留元信息并采用以下设置，网络 Prefab 双端加载；说明见 [[reference/api/modinfo/fields#运行端与用户配置]]。

```lua
client_only_mod = false
all_clients_require_mod = true
```

合并到已有 `modmain.lua`，保留原条目。共用注册规则见 [[cookbook/new-item#modmainlua-注册|物品注册]]。[mods.lua:654](../../../raw/current/scripts/mods.lua#L654

```lua
PrefabFiles = PrefabFiles or {}
PrefabFiles[#PrefabFiles + 1] = "dstwiki_hat"

GLOBAL.STRINGS.NAMES.DSTWIKI_HAT = "悠闲草帽"
GLOBAL.STRINGS.CHARACTERS.GENERIC.DESCRIBE.DSTWIKI_HAT =
    "戴上它，放松一下。"
```

### 帽子 Prefab

目标文件 `scripts/prefabs/dstwiki_hat.lua`，完整内容：

```lua
local assets = {
    Asset("ANIM", "anim/hat_straw.zip"),
}

local function OnEquip(inst, owner)
    owner.AnimState:OverrideSymbol("swap_hat", "hat_straw", "swap_hat")
    owner.AnimState:ClearOverrideSymbol("headbase_hat")
    owner.AnimState:Show("HAT")
    owner.AnimState:Show("HAIR_HAT")
    owner.AnimState:Hide("HAIR_NOHAT")
    owner.AnimState:Hide("HAIR")
    if owner.isplayer then
        owner.AnimState:Hide("HEAD")
        owner.AnimState:Show("HEAD_HAT")
        owner.AnimState:Show("HEAD_HAT_NOHELM")
        owner.AnimState:Hide("HEAD_HAT_HELM")
    end
end

local function OnUnequip(inst, owner)
    owner.AnimState:ClearOverrideSymbol("headbase_hat")
    owner.AnimState:ClearOverrideSymbol("swap_hat")
    owner.AnimState:Hide("HAT")
    owner.AnimState:Hide("HAIR_HAT")
    owner.AnimState:Show("HAIR_NOHAT")
    owner.AnimState:Show("HAIR")
    if owner.isplayer then
        owner.AnimState:Show("HEAD")
        owner.AnimState:Hide("HEAD_HAT")
        owner.AnimState:Hide("HEAD_HAT_NOHELM")
        owner.AnimState:Hide("HEAD_HAT_HELM")
    end
end

local function fn()
    local inst = CreateEntity()
    inst.entity:AddTransform()
    inst.entity:AddAnimState()
    inst.entity:AddNetwork()
    MakeInventoryPhysics(inst)

    inst.AnimState:SetBank("strawhat")
    inst.AnimState:SetBuild("hat_straw")
    inst.AnimState:PlayAnimation("anim")
    inst:AddTag("hat")

    inst.entity:SetPristine()
    if not TheWorld.ismastersim then
        return inst
    end

    inst:AddComponent("inspectable")
    inst:AddComponent("inventoryitem")
    inst.components.inventoryitem:ChangeImageName("strawhat")

    inst:AddComponent("equippable")
    inst.components.equippable.equipslot = EQUIPSLOTS.HEAD
    inst.components.equippable.dapperness = 3 / 60
    inst.components.equippable:SetOnEquip(OnEquip)
    inst.components.equippable:SetOnUnequip(OnUnequip)

    return inst
end

return Prefab("dstwiki_hat", fn, assets)
```

**资源与替换点**：bank 为 `strawhat`，build 为 `hat_straw`，地面动画为 `anim`；人物目标与来源 symbol 都是 `swap_hat`，图标名为 `strawhat`。这些名称由 `MakeHat("straw")` 的命名／工厂链确认，不能把文件名、bank 和 build 混用。[hats.lua:18](../../../raw/current/scripts/prefabs/hats.lua#L18)、[:197](../../../raw/current/scripts/prefabs/hats.lua#L197)、[:6755](../../../raw/current/scripts/prefabs/hats.lua#L6755)、[:6983](../../../raw/current/scripts/prefabs/hats.lua#L6983)

当前安装有动画包及 `strawhat.tex` 配套图集；图标经默认查找取得，不硬编码图集编号，接口见 [[reference/components/inventoryitem#关键接口]]。换标识时统一改文件／注册／Prefab／字符串键；原创外观须提供实际 ZIP、bank／build／动画／symbol 和 XML／TEX。

**显示与功能**：本例采用普通帽子的完整显示分组及卸装恢复，穿脱回调只管理外观；精神贡献由人物 sanity 消费装备槽，无需另建周期任务或在卸装时扣回。共同实体／动画／标签在 `SetPristine` 前，组件在服务端分支。[hats.lua:31](../../../raw/current/scripts/prefabs/hats.lua#L31)、[:50](../../../raw/current/scripts/prefabs/hats.lua#L50)、[:77](../../../raw/current/scripts/prefabs/hats.lua#L77)、[sanity.lua:514](../../../raw/current/scripts/components/sanity.lua#L514

### 省略人物穿戴外观

可不制作头部／身体装备的穿戴图。本例删除 **OnEquip／OnUnequip 两段纯显示回调及其注册**，保留 HEAD、dapperness 和地面／库存资源；只注释 `OverrideSymbol` 而保留 Show／Hide 仍会改变头发。已有回调若含技能、容器、耗能、监听或任务，保留功能与撤销配对，不为隐藏外观清空其他装备符号。自定义角色／皮肤和多装备栏另核验，统一角色显示策略留角色骨架。[equippable.lua:99](../../../raw/current/scripts/components/equippable.lua#L99)、[:118](../../../raw/current/scripts/components/equippable.lua#L118)、[hats.lua:64](../../../raw/current/scripts/prefabs/hats.lua#L64

### 验证帽子

隔离测试世界启用并重载模组，服务端控制台生成；远端管理员使用远程执行：

```lua
c_spawn("dstwiki_hat")
```

待执行：检查地面／库存、名称与检查语、拾取与 HEAD 装备；戴上显示帽子及对应头发，卸下／换帽恢复。用普通角色、干燥与可控环境比较装备前后精神贡献，不能把净变化当基础值；再测存档重载、死亡掉落、第二位客户端，以及与原版普通帽／露顶帽／全包头盔来回切换。可选省图方案单独测穿脱。[consolecommands.lua:208](../../../raw/current/scripts/consolecommands.lua#L208)、[inventory.lua:1219](../../../raw/current/scripts/components/inventory.lua#L1219

## 按需求扩展

| 新需求 | 接入入口与分工 |
|---|---|
| 头盔吸收伤害／受击耐久 | [[reference/components/armor#关键接口|armor]]，保留 HEAD；参考 [football 工厂](../../../raw/current/scripts/prefabs/hats.lua#L354)。防御能力与是否遮脸分别选择 |
| 防雨或冬季保温／夏季隔热 | [[reference/components/waterproofer#关键接口|waterproofer]]、[[reference/components/insulator#关键接口|insulator]]；不由帽子名称／动画推断 |
| 装备耗时、卸装停止及耗尽 | [[reference/components/fueled#关键接口|fueled]]；补穿脱 Start／Stop 与耗尽处理，假人按 [原版收尾](../../../raw/current/scripts/prefabs/hats.lua#L173) 核验 |
| 动态精神、速度与角色限制 | [[reference/components/equippable#关键接口|equippable]]；贡献单位、回调覆盖与同步边界见该页 |
| 露顶帽／眼镜外观 | [opentop_onequip](../../../raw/current/scripts/prefabs/hats.lua#L116) 保留正常头发／头部；换资源并配对卸装，按原版选择 `open_top_hat`；本例普通帽显示不能直接套用 |
| 全包头盔外观 | [fullhelm_onequip／onunequip](../../../raw/current/scripts/prefabs/hats.lua#L130) 另处理 headbase_hat、脸／胡须和 HeadHatExchange；需匹配资源与恢复，不只切换一个 HEAD_HAT_HELM |
| 护目镜防沙等能力 | [deserthat](../../../raw/current/scripts/prefabs/hats.lua#L2208) 在共同初始化加 `goggles`，而 [goggleshat](../../../raw/current/scripts/prefabs/hats.lua#L2242) 明确不提供该标签收益；实际消费者／显示与自定义过滤仍须定向核验，本页未提供完整防沙 demo |
| 帽子兼容容器／专属技能 | [[reference/components/container#新容器的客户端布局|container 双端布局]]；穿脱开关、转移与监听清理另核验，不照搬角色私有组件 |

## 边界与验证

- **槽位与能力混淆**：equippable 默认 HANDS，必须显式写 HEAD；`hat` 或 `goggles` 标签、头盔名称及图像均不自动提供 armor／保温等组件能力。[equippable.lua:25](../../../raw/current/scripts/components/equippable.lua#L25)、[hats.lua:379](../../../raw/current/scripts/prefabs/hats.lua#L379
- **只恢复贴图**：漏还原头发／头部分组会残留显示；普通帽、露顶帽、全包头盔是不同美术模式。私有角色或皮肤可能另管符号，本例不接皮肤、特效或角色锁图。[hats.lua:77](../../../raw/current/scripts/prefabs/hats.lua#L77)、[:116](../../../raw/current/scripts/prefabs/hats.lua#L116)、[:160](../../../raw/current/scripts/prefabs/hats.lua#L160)
- **直接调组件 Equip 或移除 equippable**：绕过人物 inventory 不能保证槽位／卸装清理。用正常装备流程；宿主删除与组件移除边界见 [[reference/components/equippable#常见陷阱与限制]]。
- **默认继承或兼容**：本例未接燃烧、漂浮、假人专属回调、皮肤等原版完整工厂功能。使用原版 HEAD；`NECK or HEAD` 只是可选槽位回退，不保证消费者、多装备栏或共用外观兼容，未联合实测。[hats.lua:188](../../../raw/current/scripts/prefabs/hats.lua#L188)、[:242](../../../raw/current/scripts/prefabs/hats.lua#L242)

### 版本增量复核 26148036（2026-10-25）
处理范围：锚点重定位 19 处（hats.lua 各穿脱／工厂／注册锚点 +1 或按新增帽型段平移至 6755/6983，mods.lua PrefabFiles 段 -39）；无结论修订，仅锚点迁移——equippable／sanity／inventory／consolecommands 未变。依据：全局枢纽 diff 审阅（mods.lua R-11 改动为 Release ID 注册表与 moderror，不涉 PrefabFiles 加载）＋变更清单命中 prefabs/hats.lua（本版改动均为其他帽型新增）、components/inventoryitem.lua、tuning.lua、mods.lua。

验证记录（2026-10-11，Steam buildid 25643519）：审阅原版帽子命名／注册链、资源与图标入口、网络分界、普通／露顶／全包头盔穿脱，以及装备精神消费者；demo 只采用普通帽显示和基础贡献。只读确认动画包含 anim.bin／build.bin／纹理、当前图集有 strawhat.tex 和配套纹理，未复制资源。

四个 Lua 块与可选省图工厂经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返、显式工厂结构检查；页面／来源／链接／唯一登记、大小及实际检索／章节读取通过。**未启动 DST**：生成、穿脱／换帽／省图、精神变化、存档／死亡、多人显示、骑乘／假人／皮肤、自定义角色、多装备栏及扩展能力未实测；引擎动画／网络内部仅按原版调用模式使用。
