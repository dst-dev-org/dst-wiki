---
title: "aoediminishingreturns"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "为 WX-78 旋转等调用链提供有来源的副效果倍率，供吸血和工具热量显式读取。"
aliases: ["旋转攻击副效果递减", "WX78范围吸血递减", "aoediminishingreturns"]
tags: [component, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/aoediminishingreturns.lua", "util/sourcemodifierlist.lua", "wx78_moduledefs.lua", "stategraphs/SGwx78_common.lua", "stategraphs/SGwilson.lua", "prefabs/wx78.lua", "prefabs/shadow_battleaxe.lua", "components/fumaroletool.lua", "componentutil.lua", "tuning.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# aoediminishingreturns

## 快速判断

- 职责与适用：仅提供 `mult = SourceModifierList(inst)`；组件不寻找范围目标、不计数、不主动降低伤害。调用者负责设置倍率，武器／工具消费端显式读取。[完整组件](../../../raw/current/scripts/components/aoediminishingreturns.lua#L1-L12) [旋转设置及注释](../../../raw/current/scripts/stategraphs/SGwx78_common.lua#L504-L521)
- 优先使用：自定义范围攻击需要让已接入的副效果按同一来源倍率缩放时使用；实际伤害、耐久效率分别走 combat 与 efficientuser，不能只添加本组件就获得通用 AOE 递减。[原调用链](../../../raw/current/scripts/stategraphs/SGwx78_common.lua#L499-L521)
- 重要副作用：同一宿主上的倍率会影响所有读取它的消费者；已核验暗影战斧生命恢复与 fumaroletool 热量，不保证每种武器效果都读取。[吸血](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L295-L306) [热量](../../../raw/current/scripts/components/fumaroletool.lua#L121-L130)
- 修改边界：原宿主是激活 spin 模块的 WX-78，最后一个 spin 模块失活时移除组件；没有 OnSave／OnLoad，移除时 Reset 来源表。[模块生命周期](../../../raw/current/scripts/wx78_moduledefs.lua#L1526-L1551) [移除](../../../raw/current/scripts/components/aoediminishingreturns.lua#L8-L9)

## 关键接口

| 接口／字段 | 约定、条件与作用 |
|---|---|
| `mult:Get()` | 返回当前来源倍率乘积；默认 1，组件构造没有设置递减值。[构造](../../../raw/current/scripts/components/aoediminishingreturns.lua#L3-L5) [默认／乘积](../../../raw/current/scripts/util/sourcemodifierlist.lua#L8-L25) [读取与重算](../../../raw/current/scripts/util/sourcemodifierlist.lua#L38-L58) |
| `mult:SetModifier(source, m, key?)` | 按来源和键设置／替换贡献；nil 来源忽略，省略键用 `"key"`，m 为 nil 或基础值 1 时移除此键。接口不把数值限制在 0～1。[实现](../../../raw/current/scripts/util/sourcemodifierlist.lua#L64-L99) |
| `mult:RemoveModifier(source, key?)` | 有键仅删除该键；无键删除该来源全部贡献，并重算乘积。[实现](../../../raw/current/scripts/util/sourcemodifierlist.lua#L103-L121) |
| `OnRemoveFromEntity()` | 调用 `mult:Reset()`，清来源及来源 onremove 监听，恢复默认 1；正常结束自定义效果应撤销自己的来源／键，不能 Reset 他人的贡献。[组件](../../../raw/current/scripts/components/aoediminishingreturns.lua#L8-L9) [Reset](../../../raw/current/scripts/util/sourcemodifierlist.lua#L124-L134) |

来源若是实体，SourceModifierList 监听该来源的 `onremove`，来源删除时自动清掉该来源贡献；字符串来源没有这个自动清理条件。原模块会在失活时删除整个组件，故其他模组依附同一组件的贡献也随之消失，不能假定该组件终身存在。[来源监听](../../../raw/current/scripts/util/sourcemodifierlist.lua#L79-L92) [原模块删除](../../../raw/current/scripts/wx78_moduledefs.lua#L1541-L1551)

### 原版旋转与两个消费端

WX-78 主端创建 upgrademoduleowner；spin 定义的 activatefn 负责添加本组件，服务端 SGwilson 引入通用旋转状态。[主端宿主](../../../raw/current/scripts/prefabs/wx78.lua#L734-L774) [模块定义](../../../raw/current/scripts/wx78_moduledefs.lua#L1526-L1561) [状态接入](../../../raw/current/scripts/stategraphs/SGwilson.lua#L29019)

`wx_spin` 的攻击分支先用已有 `statemem.dim` 设置来源 inst／键 `"wx78_spin"`，再 DoAttack，随后将 dim 更新为 `(dim or 1) * aoe_dim`。原值 `aoe_dim = 0.5`：首次尚无本键贡献，后续依次使用 0.5、0.25……；这是同一状态攻击序列的副效果倍率，其他来源仍会相乘。范围遍历结束后移除此键，并非永久写入角色。[键](../../../raw/current/scripts/stategraphs/SGwx78_common.lua#L185) [读取配置](../../../raw/current/scripts/stategraphs/SGwx78_common.lua#L401-L403) [设置／更新](../../../raw/current/scripts/stategraphs/SGwx78_common.lua#L504-L521) [结束清理](../../../raw/current/scripts/stategraphs/SGwx78_common.lua#L580-L588) [配置值](../../../raw/current/scripts/tuning.lua#L9437)

- 暗影战斧：持有者受伤且目标可被吸取生命（26148036 起改用 `componentutil.IsLifeDrainable`：不在 NON_LIFEFORM_TARGET_TAGS、或带 `lifedrainable` 标签豁免，且无 `nolifedrainable` 拒绝，shadow_battleaxe.lua:298）时，生命恢复量 `_lifesteal * mult`；紧随其后的理智损耗没有乘 mult。没有该组件时倍率回退 1。[DoLifeSteal](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L295-L306) [判定](../../../raw/current/scripts/componentutil.lua#L185-L188)
- fumaroletool：配置 heatonuse 时，`heatonuse * mult` 同时用于物品增温和使用者降温；读取的是 doer 的组件。攻击事件／物品使用入口再调用 OnUsed，不是修改攻击伤害。[攻击事件](../../../raw/current/scripts/components/fumaroletool.lua#L33-L35) [热量处理](../../../raw/current/scripts/components/fumaroletool.lua#L121-L130) [物品使用](../../../raw/current/scripts/components/fumaroletool.lua#L149-L152)

## 典型接入

### 原旋转模块激活期间，让已接入副效果额外乘 0.8

目标模组 `modmain.lua`；复用原 `wx78module_spin`、模块工厂和资源。模块名称／回调绑定及加载恢复协议见 [[reference/components/upgrademodule#典型接入]]；此处仅包装激活／失活，保留原回调和 isloading。0.8 是来源倍率贡献，不是普通伤害倍率，也不局限于旋转命中的当刻。[spin 定义](../../../raw/current/scripts/wx78_moduledefs.lua#L1526-L1561) [PostInit 注册](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("wx78module_spin", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local module = inst.components.upgrademodule
    if module == nil then return end
    local activate = module.onactivatedfn
    local deactivate = module.ondeactivatedfn
    module.onactivatedfn = function(item, owner, isloading)
        if activate ~= nil then activate(item, owner, isloading) end
        local aoe = owner ~= nil and owner.components.aoediminishingreturns or nil
        if aoe ~= nil then aoe.mult:SetModifier(item, 0.8, "my_mod_spin") end
    end
    module.ondeactivatedfn = function(item, owner)
        local aoe = owner ~= nil and owner.components.aoediminishingreturns or nil
        if aoe ~= nil then aoe.mult:RemoveModifier(item, "my_mod_spin") end
        if deactivate ~= nil then deactivate(item, owner) end
    end
end)
```

多枚激活模块分别作为来源相乘，例如两枚额外贡献 0.64；同一枚重复设置替换自身贡献。原旋转键的清理不会清掉本例独立来源。本例有意影响模块激活期间所有显式读取消费者，包括普通单次攻击的吸血与工具热量；若只要逐目标递减，应在自定义范围攻击循环内成对设置／撤销，不套用此持续贡献例。[来源重算／替换](../../../raw/current/scripts/util/sourcemodifierlist.lua#L47-L99) [消费者](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L295-L306)

能量不足或正常拔出经原模块失活链撤销；加载时原模块重新激活并重施贡献，组件不直接保存 mult。停用模组后重载不再安装包装。绕过模块宿主手动清 target 或删除组件不属于本例正常恢复链；保存／恢复依据复用 [[reference/components/upgrademodule#关键接口]]。

## 常见陷阱与限制

- **以名字推断所有 AOE 伤害自动递减**：组件只有来源表和移除重置；原 SG 注释明确该倍率用于独特武器行为，而非 damage。自定义消费者必须主动读取。[组件](../../../raw/current/scripts/components/aoediminishingreturns.lua#L3-L9) [原注释](../../../raw/current/scripts/stategraphs/SGwx78_common.lua#L504-L508)
- **吸血和理智代价同比缩放**：生命恢复读取倍率，理智损耗使用未缩放的 `_lifesteal`；不能据回血变少推断理智代价也变少。[实现](../../../raw/current/scripts/prefabs/shadow_battleaxe.lua#L300-L305)
- **临时修改无须撤销，或用 Reset 收尾**：只设置不会自行按时间过期；实体删除清理也不替代效果结束时的 RemoveModifier。共享表 Reset 会移除全部来源。[来源清理](../../../raw/current/scripts/util/sourcemodifierlist.lua#L85-L92) [显式撤销／重置](../../../raw/current/scripts/util/sourcemodifierlist.lua#L103-L134)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 5 处（SGwilson 旋转状态接入 +234、wx78_moduledefs spin 定义段 -3、tuning WX78_SPIN_AOE_DIMINISHING +17）；结论修订 1 处：暗影战斧 DoLifeSteal 资格门 26148036 起由 `not HasOneOfTags(NON_LIFEFORM_TARGET_TAGS)` 改为 `componentutil.IsLifeDrainable(target)`（可被 lifedrainable 标签豁免、nolifedrainable 拒绝），倍率读取与理智不缩放结论不变；WX78_SPIN_AOE_DIMINISHING=0.5 值未变。依据：player-hubs-hunk-map＋tuning-hunk-map＋变更清单命中 wx78_moduledefs.lua、prefabs/wx78.lua（master_postinit 两行增删净零）、prefabs/shadow_battleaxe.lua、componentutil.lua（新增 IsLifeDrainable）、tuning.lua。
- 源码审阅：2026-10-21，Steam buildid 25643519；完整审阅本组件、SourceModifierList，核验 spin 模块定义／增删、SG 旋转攻击设置及清理、配置值、暗影战斧和 fumaroletool 消费。精确标识符定位覆盖本快照四个协作文件；模块工厂绑定与保存恢复复用同 build 的 upgrademodule 页。未完整审校旋转全部动作、热量系统和所有武器效果。
- 结构检查：本次对本页执行元数据、标签、大小、本地链接及严格源码路径／行号检查；完整 modmain 例用现有 Full Moon Lua 5.1 syntax-only 检查，不执行示例。
- 游戏运行：未运行；多目标顺序、吸血／理智、热量、模块充放电／拔出和加载恢复仍需实测。

相关：[[reference/components/efficientuser]]；[[reference/components/upgrademodule]]；[[reference/components/upgrademoduleowner]]。
