---
title: "damagetyperesist"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "按攻击者或武器标签返回剩余伤害倍率；区分装备抗性、受击实体抗性与整次抵挡。"
aliases: ["装备降低暗影阵营伤害", "按攻击来源标签减伤", "damagetyperesist"]
tags: [component, equipment, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/damagetyperesist.lua", "util/sourcemodifierlist.lua", "components/inventory.lua", "components/combat.lua", "prefabs/armor_lunarplant.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# damagetyperesist

## 快速判断

- 职责与适用：匹配 attacker 或 weapon 的标签，返回剩余伤害倍率；不是护甲耐久或整次免伤开关。[计算](../../../raw/current/scripts/components/damagetyperesist.lua#L32-L42)
- 优先使用：AddResist(tag,source,mult,key)，按同键 RemoveResist 撤销；0.8 是剩余伤害乘 0.8 的贡献，整次抵挡另见 [[reference/components/resistance]]。[接口](../../../raw/current/scripts/components/damagetyperesist.lua#L8-L25)
- 重要副作用：多来源、多命中标签和多件装备可乘算；attacker=nil 时返回 1，即使 weapon 有标签也不匹配。[组件](../../../raw/current/scripts/components/damagetyperesist.lua#L32-L42) [装备合并](../../../raw/current/scripts/components/inventory.lua#L408-L430)
- 修改边界：服务端；装备必须进入库存 ApplyDamage 才消费，生物自身由 combat 对应分支消费。不同消费位置不保证相同覆盖，不能宣称一处设置防住所有扣血。[自身消费](../../../raw/current/scripts/components/combat.lua#L645-L652)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `AddResist(tag,src,pct,key?)` | 为标签添加剩余伤害倍率，无内置 0～1 限幅；名称 pct 不表示已减去的百分比。[实现](../../../raw/current/scripts/components/damagetyperesist.lua#L8-L15) |
| `RemoveResist(tag,src,key?)` | 指定键只撤该项；无键则撤整个来源。来源键／实体移除协议复用 [[reference/components/damagetypebonus#来源与消费边界]]，不要覆盖宿主默认键。[组件](../../../raw/current/scripts/components/damagetyperesist.lua#L17-L25) [来源清理](../../../raw/current/scripts/util/sourcemodifierlist.lua#L103-L121) |
| `GetResistForTag(tag)` | 返回该标签已合并倍率，无配置时 1；不检查实际攻击者／武器，不能代替完整攻击查询。[实现](../../../raw/current/scripts/components/damagetyperesist.lua#L27-L30) |
| `GetResist(attacker,weapon?)` | 有 attacker 才查标签；同一 tag 即使两者都匹配只乘一次，不同 tag 均匹配则各乘一次。[实现](../../../raw/current/scripts/components/damagetyperesist.lua#L32-L42) |

库存先检查 resistance 整次抵挡，否则收集护甲并乘装备类型抗性；该合并倍率作用于普通伤害和此路径中的特殊伤害，再由后续护甲／特殊防御处理。生物自身的 combat 类型抗性是另一位置，不能把装备结论直接推广为自身特殊伤害的处理规则。[库存普通伤害](../../../raw/current/scripts/components/inventory.lua#L408-L430) [特殊伤害](../../../raw/current/scripts/components/inventory.lua#L455-L487) [自身位置](../../../raw/current/scripts/components/combat.lua#L645-L652)

尤其当前 combat 的条件受伤倍率分支与后续普通伤害行都出现 damagetypemult；本页不把存在该分支的自定义宿主概括为“全链恰好乘一次”，案例只扩展装备消费者。[具体分支](../../../raw/current/scripts/components/combat.lua#L645-L652)

## 典型接入

### 原月亮植物甲增加 shadow_aligned 来源的 0.8 倍受伤贡献

目标模组 `modmain.lua`；保留 armor_lunarplant 的原基础月亮抗性、套装启停和资源，只加独立键。该组件已由原工厂安装，普通入袋不会使库存把它当装备消费。[原组件与套装](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L220-L226) [装备槽消费](../../../raw/current/scripts/components/inventory.lua#L408-L430)

```lua
AddPrefabPostInit("armor_lunarplant", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local resist = inst.components.damagetyperesist
    if resist ~= nil then
        resist:AddResist("shadow_aligned", inst, 0.8, "my_mod_shadow")
    end
end)
```

这是装备在原受击路径生效时的额外倍率：攻击者或武器带 shadow_aligned 即可匹配，但必须有非 nil 的攻击者。不改目标／穿戴者标签，不承诺直接 health:DoDelta 也减伤；多件装备、护甲与其他倍率仍按各自路径结算。

卸下后该装备不再位于库存装备槽，贡献不再被该消费者收集；不需要清掉物品上的静态配置。当前实例如需撤配置，调用 RemoveResist("shadow_aligned",inst,"my_mod_shadow")；停用并完整重载后不再重施。套装额外条件见 [[reference/components/setbonus#典型接入]]，不要把静态抗性和套装开关混写。

## 常见陷阱与限制

- **传 0.2 表示减伤 20%**：这里返回剩余倍率，0.2 对该贡献意味着只余 20%；普通护甲吸收比例另看 [[reference/components/armor]]。[返回计算](../../../raw/current/scripts/components/damagetyperesist.lua#L32-L42)
- **只有 weapon 就足够匹配**：attacker=nil 时整段不执行；构造伤害请求要按实际消费者提供来源，不凭空改 attacker 绕门禁。[条件](../../../raw/current/scripts/components/damagetyperesist.lua#L32-L42)
- **自己有防御标签即可减伤**：匹配的是攻击者／武器，不是自身。标签重合只在同一组 OR，多个 tag 仍相乘。[匹配](../../../raw/current/scripts/components/damagetyperesist.lua#L35-L37)
- **组件自存所有新增抗性**：无自身保存钩子；原工厂静态重施，动态效果按来源生命周期恢复／撤销。来源协议见上文关联页。

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；相关调用与示例仍适用，本轮仅重定位证据（锚点 4 处），未运行游戏。

- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖完整组件、来源工具、库存普通／特殊伤害消费、combat 所引自身倍率位置与条件分支、原月亮植物甲配置／注册。未全面审校全部伤害入口、自定义条件倍率、特殊伤害类型或多栏模组。
- 结构检查：本批格式、来源、链接／锚点、大小与登记；完整装备抗性例经已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；仅武器标签、空 attacker、多件／多标签合并、穿卸／存读与特殊伤害待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。