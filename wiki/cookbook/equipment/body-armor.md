---
title: 新增身体护甲
created: 2026-10-11
updated: "2026-10-28"
type: cookbook
summary: 注册 BODY 槽护甲，复用原版木甲外观与图标，接入普通伤害吸收、受击耐久和穿脱恢复，可省略人物穿戴外观
aliases: [身体护甲, 新增护甲, 新护甲骨架, 身体装备, 无穿戴外观护甲, 隐藏装备外观, equipment/body-armor, dstwiki_bodyarmor]
tags: [cookbook, item, equipment, prefab, modding, server-client]
dst_build: "steam-buildid:26230496"
sources: [prefabs/armor_wood.lua, components/armor.lua, components/equippable.lua, components/inventory.lua, components/inventoryitem.lua, components/inventoryitem_replica.lua, components/combat.lua, entityscript.lua, mods.lua, consolecommands.lua]
reliability: verified
validation: source-reviewed
status: active
---

# 新增身体护甲

## 适用条件

已有 DST 世界／服务器模组，新增可拾取、装备到 **BODY** 槽并吸收普通伤害的护甲。已有工程沿用自己的注册约定；尚无入口先从 [[cookbook/index|场景导航]] 选择起步。只改原版木甲数值直接读 [[reference/components/armor#将原版木甲基础吸收率改为-90|机制案例]]。

本例 `dstwiki_bodyarmor` 使用原版木甲地面／穿戴外观与图标，初始耐久 **600**、基础吸收率 **0.80**；正常装备时参与伤害结算，卸下或仅放物品栏时不提供该项。耐久按吸收伤害等结算扣减，归零默认删除，不是可承受 600 次攻击，也不保证所有伤害减免 80%。[inventory.lua:408](../../../raw/current/scripts/components/inventory.lua#L408)、[armor.lua:39](../../../raw/current/scripts/components/armor.lua#L39)、[:93](../../../raw/current/scripts/components/armor.lua#L93)

BODY 是原版身体槽，服装、护符和背包也可能使用它；**槽位不定义功能**。本例不带容器、位面防御或专属技能，其他类别从 [[cookbook/equipment/index|装备导航]] 判断覆盖。

## 最小 demo

### 文件与加载

保留已有模组入口，新增工厂：

```text
<目标模组>/
├── modinfo.lua
├── modmain.lua
└── scripts/prefabs/dstwiki_bodyarmor.lua
```

在已有 `modinfo.lua` 保留元信息，采用以下设置；网络 Prefab 的工厂需要服务器／客户端加载，详见 [[reference/api/modinfo/fields#运行端与用户配置]]。

```lua
client_only_mod = false
all_clients_require_mod = true
```

合并到已有 `modmain.lua`，保留原注册条目。通用加载与命名规则见 [[cookbook/new-item#modmainlua-注册|物品注册]]，无需重复读取整个道具 demo。[mods.lua:654](../../../raw/current/scripts/mods.lua#L654

```lua
PrefabFiles = PrefabFiles or {}
PrefabFiles[#PrefabFiles + 1] = "dstwiki_bodyarmor"

GLOBAL.STRINGS.NAMES.DSTWIKI_BODYARMOR = "轻木护甲"
GLOBAL.STRINGS.CHARACTERS.GENERIC.DESCRIBE.DSTWIKI_BODYARMOR =
    "穿上它，再去面对危险。"
```

### 身体护甲 Prefab

目标文件 `scripts/prefabs/dstwiki_bodyarmor.lua`，完整内容：

```lua
local assets = {
    Asset("ANIM", "anim/armor_wood.zip"),
}

local function OnEquip(inst, owner)
    owner.AnimState:OverrideSymbol("swap_body", "armor_wood", "swap_body")
end

local function OnUnequip(inst, owner)
    owner.AnimState:ClearOverrideSymbol("swap_body")
end

local function fn()
    local inst = CreateEntity()

    inst.entity:AddTransform()
    inst.entity:AddAnimState()
    inst.entity:AddNetwork()
    MakeInventoryPhysics(inst)

    inst.AnimState:SetBank("armor_wood")
    inst.AnimState:SetBuild("armor_wood")
    inst.AnimState:PlayAnimation("anim")

    inst.entity:SetPristine()
    if not TheWorld.ismastersim then
        return inst
    end

    inst:AddComponent("inspectable")
    inst:AddComponent("inventoryitem")
    inst.components.inventoryitem:ChangeImageName("armorwood")

    inst:AddComponent("armor")
    inst.components.armor:InitCondition(600, 0.80)

    inst:AddComponent("equippable")
    inst.components.equippable.equipslot = EQUIPSLOTS.BODY
    inst.components.equippable:SetOnEquip(OnEquip)
    inst.components.equippable:SetOnUnequip(OnUnequip)

    return inst
end

return Prefab("dstwiki_bodyarmor", fn, assets)
```

**资源与替换点**：地面 bank／build 为 `armor_wood`，动画为 `anim`；人物替换目标与来源 symbol 均为 `swap_body`，来源 build 为 `armor_wood`。图标用注册名 `armorwood` 对应的原版图元；源码文件名 `armor_wood.lua`、动画名和图标名不能混写。[armor_wood.lua:16](../../../raw/current/scripts/prefabs/armor_wood.lua#L16)、[:41](../../../raw/current/scripts/prefabs/armor_wood.lua#L41)、[:83](../../../raw/current/scripts/prefabs/armor_wood.lua#L83)、[inventoryitem.lua:470](../../../raw/current/scripts/components/inventoryitem.lua#L470

当前安装提供 `anim/armor_wood.zip` 和 `armorwood.tex`，图集由默认查找取得，无需复制资源或硬编码编号；图标接口见 [[reference/components/inventoryitem#关键接口]]。换新标识时统一替换文件名、注册条目、Prefab 名和字符串大写键；换原创外观还须提供实际存在的 ZIP、bank／build／动画／symbol 与 XML／TEX。

**功能与撤销**：共同实体／动画在 `SetPristine` 前，组件在服务端分支。`armor` 负责吸收与耐久，`equippable` 负责槽位／状态；本例穿脱回调仅设置／清除人物外观，不在回调里永久修改人物减伤。正常装备由 inventory 管理；归零的实体删除经 inventoryitem 移出持有者，再经 inventory 卸装清理。[equippable.lua:92](../../../raw/current/scripts/components/equippable.lua#L92)、[inventory.lua:1219](../../../raw/current/scripts/components/inventory.lua#L1219)、[:1424](../../../raw/current/scripts/components/inventory.lua#L1424)、[inventoryitem.lua:485](../../../raw/current/scripts/components/inventoryitem.lua#L485)、[entityscript.lua:1730](../../../raw/current/scripts/entityscript.lua#L1730

### 省略人物穿戴外观

头部／身体装备（含护符、背包、护甲）可按项目约定不做人物穿戴图。本例若选此方案，删除两段纯外观回调及其 `SetOnEquip`／`SetOnUnequip` 注册，保留装备槽、armor 和地面动画／图标；不要为了隐藏外观清空别件装备的符号。已有回调若包含技能、容器开关、监听或任务，只省外观逻辑，保留功能与撤销配对；头部显示、皮肤／自定义角色及多装备栏还需分别核验。角色骨架将另行讨论统一显示策略。[equippable.lua:99](../../../raw/current/scripts/components/equippable.lua#L99)、[:118](../../../raw/current/scripts/components/equippable.lua#L118)、[inventory.lua:421](../../../raw/current/scripts/components/inventory.lua#L421

### 验证护甲

隔离测试世界启用模组并重载，服务端控制台生成；远端管理员使用远程执行：

```lua
c_spawn("dstwiki_bodyarmor")
```

待执行检查：地面／库存显示、名称与检查语正确；可拾取并装备到身体槽，穿上显示木甲，卸下或更换身体装备恢复对应外观。使用普通角色、单件护甲和可重复的普通攻击验证吸收／耐久，再测耗尽移除、存档耐久恢复、死亡掉落与第二位客户端显示；可选无穿戴图方案另测穿脱。不要用直接扣生命的测试替代装备伤害链验证，也不要仅看外观判定防御生效。[consolecommands.lua:208](../../../raw/current/scripts/consolecommands.lua#L208)、[combat.lua:630](../../../raw/current/scripts/components/combat.lua#L630)、[armor.lua:116](../../../raw/current/scripts/components/armor.lua#L116

## 按需求扩展

| 新需求 | 接入入口与分工 |
|---|---|
| 调整吸收／耐久、不可破损或耗尽保留 | [[reference/components/armor#关键接口|armor]]；保留零耐久后须另核验不可用／修复状态，不能把保留实体当成自动禁用 |
| 投喂材料修复 | [[reference/components/trader#给物品投喂材料|材料接收]] 配合 [[reference/components/armor#关键接口|Repair]]；数量与材料消费按 trader 边界，不复制角色模组私有交易动作 |
| 位面防御 | [[reference/components/planardefense#关键接口|planardefense]] 独立配置；普通吸收与多件护甲的关系见 [[reference/components/armor#常见陷阱与限制|armor 限制]] |
| 装备精神贡献、速度或角色限制 | [[reference/components/equippable#关键接口|装备属性]]；字段单位、动态回调和同步范围按该页核对 |
| 保温／防雨 | [[reference/components/insulator#关键接口|insulator]]、[[reference/components/waterproofer#关键接口|waterproofer]]；与普通伤害吸收分别配置 |
| 护甲兼容容器／背包 | [[reference/components/container#新容器的客户端布局|双端布局]] 和 [[reference/components/inventory#关键接口|inventory]]；普通 BODY 护甲不会自动有格子，开关／overflow／可选 BACK 仍需核验 |
| 套装、受击特效或专属技能 | 先定向核验 setbonus／实际伤害回调与宿主、资源及清理；本页未提供完整方案，不把角色私有函数当作原版 API |

## 边界与验证

- **忘记写 BODY**：equippable 默认 HANDS；显式指定槽位。护甲、服装与背包是组装类别，不能仅凭文件名或 `armor` 标签判断组件能力。[equippable.lua:25](../../../raw/current/scripts/components/equippable.lua#L25
- **借木甲资源就继承木甲全部行为**：本例工厂独立，未加入燃料／燃烧／漂浮、特殊弱点、受击音效监听或皮肤；需要时回查对应原版代码，不扩大本页默认能力。[armor_wood.lua:19](../../../raw/current/scripts/prefabs/armor_wood.lua#L19)、[:45](../../../raw/current/scripts/prefabs/armor_wood.lua#L45)、[:62](../../../raw/current/scripts/prefabs/armor_wood.lua#L62)
- **自己扣耐久或另写存档造成重复**：inventory 的伤害链已调用 armor，组件已有 OnSave／OnLoad；本例无需再建攻击监听扣次或在 Prefab 加一份同名耐久存档。跨版本调整上限／旧存档迁移另核验。[inventory.lua:494](../../../raw/current/scripts/components/inventory.lua#L494)、[armor.lua:116](../../../raw/current/scripts/components/armor.lua#L116
- **多装备栏只需填一个新槽名**：本例使用原版 BODY，指定模组的自动分类／外观包装还受版本、配置、名称和能力影响；换槽不证明 UI、消费者或其他装备显示都兼容。省略穿戴图也须核对共用符号的管理，未实测兼容。

验证记录（2026-10-11，Steam buildid 25643519）：核验原版木甲资源、网络分界与穿脱，armor 初始化／吸收／耐久／删除／存档、inventory 装备伤害与卸装、inventoryitem 的图标和实体移除链，以及 equippable 可选回调；省略纯外观不会删除上述组件功能。只读确认动画包包含 anim.bin／build.bin／纹理，当前库存图集存在 armorwood.tex 及对应纹理，不复制游戏或模组资源。

四个 Lua 代码块经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查；页面／来源／链接与章节、大小、唯一登记及实际检索／章节读取检查通过。**未启动 DST**：生成、普通伤害与耗尽、穿脱／可选无穿戴图、存档／死亡、多人显示、骑乘、假人／皮肤、自定义角色及多装备栏兼容未实测；引擎动画／网络内部仅按原版调用模式使用。

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；相关调用与示例仍适用，本轮仅重定位证据（锚点 5 处），未运行游戏。
