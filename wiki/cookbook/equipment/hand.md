---
title: 新增手部装备
created: 2026-10-11
updated: "2026-10-28"
type: cookbook
summary: 注册可拾取的手部装备，复用原版手杖资源，实现持握显示、穿脱恢复与装备加速，按需接入武器或工具
aliases: [手部装备, 新增手持物, 手持装备骨架, 手持法杖, 新武器骨架, 新工具骨架, equipment/hand, dstwiki_handstaff]
tags: [cookbook, item, equipment, prefab, modding, server-client]
dst_build: "steam-buildid:26230496"
sources: [prefabs/cane.lua, components/equippable.lua, components/locomotor.lua, components/combat.lua, components/inventory.lua, components/inventoryitem.lua, components/inventoryitem_replica.lua, mods.lua, prefabs.lua, util.lua, consolecommands.lua]
reliability: verified
validation: source-reviewed
status: active
---

# 新增手部装备

## 适用条件

已有 DST 世界／服务器模组，新增一个普通角色可拾取、装备到 **HANDS** 槽的手持物。已有工程沿用自己的注册约定；尚无入口先从 [[cookbook/index|场景导航]] 选择模组起步。修改原版装备的属性直接查 [[reference/components/equippable#典型接入|机制案例]]。

本例 `dstwiki_handstaff` 复用原版手杖地面外观、持握外观和图标；装备时提供基础移速倍率 `1.20`，卸下后不再提供该项。它独立于原版 cane，不继承其武器伤害，也没有工作动作、耐久、施法或附加特效；这些按下文扩展。精神／加速等装备属性与物品栏携带效果要分清。[cane.lua:30](../../../raw/current/scripts/prefabs/cane.lua#L30)、[locomotor.lua:194](../../../raw/current/scripts/components/locomotor.lua#L194

## 最小 demo

### 文件与加载

保留已有模组入口，新增一个工厂文件：

```text
<目标模组>/
├── modinfo.lua
├── modmain.lua
└── scripts/prefabs/dstwiki_handstaff.lua
```

在已有 `modinfo.lua` 保留元信息并采用以下设置；新增网络 Prefab 的工厂需在服务器和客户端加载，详见 [[reference/api/modinfo/fields#运行端与用户配置]]。

```lua
client_only_mod = false
all_clients_require_mod = true
```

合并到已有 `modmain.lua`，不要覆盖原 `PrefabFiles`。注册文件名、返回的 Prefab 名和字符串键本例刻意对应；通用加载／名称规则见 [[cookbook/new-item#modmainlua-注册|物品注册]]，无需重复读取整个道具 demo。[mods.lua:654](../../../raw/current/scripts/mods.lua#L654

```lua
PrefabFiles = PrefabFiles or {}
PrefabFiles[#PrefabFiles + 1] = "dstwiki_handstaff"

GLOBAL.STRINGS.NAMES.DSTWIKI_HANDSTAFF = "轻便手杖"
GLOBAL.STRINGS.CHARACTERS.GENERIC.DESCRIBE.DSTWIKI_HANDSTAFF =
    "握在手里，走路轻快一些。"
```

### 手持 Prefab

目标文件 `scripts/prefabs/dstwiki_handstaff.lua`，完整内容：

```lua
local assets = {
    Asset("ANIM", "anim/cane.zip"),
    Asset("ANIM", "anim/swap_cane.zip"),
}

local function OnEquip(inst, owner)
    owner.AnimState:OverrideSymbol("swap_object", "swap_cane", "swap_cane")
    owner.AnimState:Show("ARM_carry")
    owner.AnimState:Hide("ARM_normal")
end

local function OnUnequip(inst, owner)
    owner.AnimState:Hide("ARM_carry")
    owner.AnimState:Show("ARM_normal")
end

local function fn()
    local inst = CreateEntity()

    inst.entity:AddTransform()
    inst.entity:AddAnimState()
    inst.entity:AddSoundEmitter()
    inst.entity:AddNetwork()
    MakeInventoryPhysics(inst)

    inst.AnimState:SetBank("cane")
    inst.AnimState:SetBuild("swap_cane")
    inst.AnimState:PlayAnimation("idle")

    inst.entity:SetPristine()
    if not TheWorld.ismastersim then
        return inst
    end

    inst:AddComponent("inspectable")
    inst:AddComponent("inventoryitem")
    inst.components.inventoryitem:ChangeImageName("cane")

    inst:AddComponent("equippable")
    inst.components.equippable.equipslot = EQUIPSLOTS.HANDS
    inst.components.equippable:SetOnEquip(OnEquip)
    inst.components.equippable:SetOnUnequip(OnUnequip)
    inst.components.equippable.walkspeedmult = 1.20

    return inst
end

return Prefab("dstwiki_handstaff", fn, assets)
```

**资源与替换点**：

- 地面 `bank = cane`、`build = swap_cane`、动画 `idle`；人物持握替换目标符号 `swap_object`，来源 build／symbol 均为 `swap_cane`。地面图和持握图分别配置，不能只改 `SetBuild` 就认为手上也显示新图。[cane.lua:1](../../../raw/current/scripts/prefabs/cane.lua#L1)、[:7](../../../raw/current/scripts/prefabs/cane.lua#L7)、[:40](../../../raw/current/scripts/prefabs/cane.lua#L40)
- `anim/cane.zip` 与 `anim/swap_cane.zip` 由当前游戏安装提供，分别包含动画与 build／纹理；不复制资源。`ChangeImageName("cane")` 使用原版图元，默认图集查找见 [[reference/components/inventoryitem#关键接口|图标接口]]；不按新注册名查不存在的图标。[inventoryitem.lua:470](../../../raw/current/scripts/components/inventoryitem.lua#L470)、[inventoryitem_replica.lua:131](../../../raw/current/scripts/components/inventoryitem_replica.lua#L131)、[util.lua:587](../../../raw/current/scripts/util.lua#L587
- 新标识统一替换文件名、注册条目、`Prefab` 名和字符串大写键；原版资源名可以保留。更换原创美术还需实际可用的 ZIP、bank／build／动画与持握 symbol，以及匹配 XML／TEX；资源制作、皮肤与假人适配另按需求核验。

**装备与撤销**：共同实体／动画在 `SetPristine` 前，服务端添加组件。HANDS 默认为 equippable 的槽位，本例显式写出；正常穿脱由持有者 inventory 管理，回调只处理外观。卸装恢复空手显示，下一件装备的回调设置自己的持握图；基础速度由 locomotor 从当前装备读取，无需在回调中永久修改人物速度或另建任务。[equippable.lua:25](../../../raw/current/scripts/components/equippable.lua#L25)、[:92](../../../raw/current/scripts/components/equippable.lua#L92)、[inventory.lua:1364](../../../raw/current/scripts/components/inventory.lua#L1364)、[cane.lua:20](../../../raw/current/scripts/prefabs/cane.lua#L20)、[locomotor.lua:194](../../../raw/current/scripts/components/locomotor.lua#L194

### 验证手持物

隔离测试世界启用模组并重载，服务端控制台生成；远端管理员使用远程执行：

```lua
c_spawn("dstwiki_handstaff")
```

待执行检查：地面外观／名称／检查语与图标正确；拾取后可装备，手上显示原版手杖；卸下、丢落或换另一件手持物后恢复对应显示。物品栏携带不加速，普通角色步行装备时增加本例基础倍率；不据此承诺所有状态的最终速度恰为原来的 1.20 倍。另测保存／重载、死亡掉落与第二位客户端显示。控制台入口见 [consolecommands.lua:208](../../../raw/current/scripts/consolecommands.lua#L208)。

## 按需求扩展

只读当前能力所需机制；手部槽并不自动提供武器、工具或主动技能。[combat.lua:853](../../../raw/current/scripts/components/combat.lua#L853

| 新需求 | 接入入口与分工 |
|---|---|
| 近战伤害／攻击触发／位面伤害 | [[reference/components/weapon#关键接口|weapon]]；客户端所需 weapon 标签按原版放共同初始化；特殊伤害再读 [[reference/components/planardamage#关键接口|planardamage]] |
| 砍树、挖矿或多功能切换 | [[reference/components/tool#关键接口|tool 动作]]；模式切换再读 [[reference/components/tool#多功能工具的模式切换|切换边界]]，不复制模组私有动作 |
| 按次损耗、持续耗能或修复 | [[reference/components/finiteuses#关键接口|次数]]、[[reference/components/fueled#关键接口|持续消耗]] 按机制择用；攻击扣次避免重复，修复按 [[reference/components/trader#关键接口|材料接收]] 核验 |
| 装备精神贡献或角色限制 | [[reference/components/equippable#关键接口|dapperness／restrictedtag]]；单位、净增长和同步范围见该页陷阱，多个允许角色的规则须另核验 |
| 右键施法、轮盘或冷却 | [[reference/components/spellcaster#典型接入|施法模式]]、[[reference/components/spellbook#典型接入|轮盘分层]]、[[reference/components/rechargeable#关键接口|充能门禁]]；输入／服务端检查分别接入，不能照搬私有 RPC／组件 |
| 手持容器 | [[reference/components/container#新容器的客户端布局|双端布局]] 和 [[reference/components/container#关键接口|开关]]；HANDS 容器不因带 container 就改成背包槽 |
| 远程／投掷、灯光或跟随特效 | [[reference/components/weapon#关键接口|远程入口]] 仅提供线索；仍须核验弹体工厂／动作和资源，附加对象／监听／任务须覆盖卸装及相关移除路径，本骨架未提供这些完整方案 |

## 边界与验证

- **只有 equippable，没有持握回调**：槽位可用不代表显示完整。保留 demo 的持握符号与穿脱配对；随意把 `swap_cane` 改为 `swap_weapon` 会查错原版 symbol。[cane.lua:7](../../../raw/current/scripts/prefabs/cane.lua#L7
- **照抄模组能力当作默认行为**：不同手持物可能是武器、工具、投掷物或容器；原版 combat 还检查 HANDS 内的 weapon，不能靠外观或文件名判能力。本例没有 weapon，也没有 finiteuses，不继承原版 cane 的伤害／耐久。[combat.lua:853](../../../raw/current/scripts/components/combat.lua#L853
- **附加效果只写 OnEquip**：简单速度字段由装备消费者处理；自建光源、任务、监听和人物修饰则需要对应清理。移除 equippable 不自动执行用户卸装回调，扩展前读 [[reference/components/equippable#常见陷阱与限制|清理边界]]，不把本例视作通用效果生命周期方案。
- **多装备栏兼容靠改槽名**：本例采用原版 HANDS；未接入额外槽位／UI／外观包装，也未实测指定多装备栏模组。兼容需核对对方实际 Hook 与配置，已有标准槽位只能作为核验起点。

验证记录（2026-10-11，Steam buildid 25643519）：核验原版 cane 的资源、网络分界和穿脱显示，equippable 回调／默认槽位、inventory 装卸、locomotor 属性消费及 combat 武器选择。安装资源只读检查确认两个动画包和 `cane.tex` 图元／对应纹理存在；图集通过默认查找取得，不硬编码编号。

四个 Lua 代码块经 Full Moon 3.0.0／Lua 5.1 语法及原字节往返检查；页面元数据、来源、链接／锚点、大小、叶级登记及实际检索／章节读取检查通过。**未启动 DST**：生成、穿脱／步行、存档／死亡、多人显示、骑乘、假人／皮肤、自定义角色动画及模组兼容未实测；引擎动画／网络内部仅按原版调用模式使用。

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；相关调用与示例仍适用，本轮仅重定位证据（锚点 4 处），未运行游戏。
