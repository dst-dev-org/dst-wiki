---
title: "aoeweapon_lunge"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "沿起终点线段与终点前方结算突刺命中、物品抛散和轨迹效果；位移由人物状态机完成。"
aliases: ["aoeweapon_lunge", "突刺武器", "闪电长矛冲刺", "沿线范围攻击", "突刺宽度"]
tags: [component, actions, callback, state-machine, equipment]
dst_build: "steam-buildid:26230496"
sources: [components/aoeweapon_lunge.lua, components/aoeweapon_base.lua, components/combat.lua, prefabs/spear_wathgrithr.lua, prefabs/spear_gungnir.lua, stategraphs/SGwilson.lua, stategraphs/SGwilson_client.lua]
reliability: verified
validation: source-reviewed
status: active
---

# aoeweapon_lunge

## 快速判断

- 职责与适用：继承 [[reference/components/aoeweapon-base]]，结算起点到终点附近及终点前方的攻击和地面物品抛散；圆形落点跳砸用 [[reference/components/aoeweapon-leap]]。[继承与默认](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L1-L18)、[命中几何](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L70-L117)
- 优先使用：扩展现有闪电长矛技能配置，保留 aoespell 发 combat_lunge、SG 调 DoLunge、完成回调放电的整条链。[施法事件](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L157-L159)、[技能配置](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L481-L497)、[SG 调用](../../../raw/current/scripts/stategraphs/SGwilson.lua#L17422-L17437)
- 重要副作用：临时改 weapon 的 damage／attackwear、关闭普通范围攻击；独立抛散物品后可生成轨迹 FX，最后才调用 onlungedfn。[开始](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L53-L68)、[后半段](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L119-L232)
- 修改边界：DoLunge 不移动角色、不自行播放 sound、不检查冷却或距离；SG 在成功结算后播音并 Teleport，原版完整链仍有地图边缘与状态退出处理。[SG 音效](../../../raw/current/scripts/stategraphs/SGwilson.lua#L17436-L17443)、[位移与退出警告](../../../raw/current/scripts/stategraphs/SGwilson.lua#L17487-L17504)

## 关键接口

| 接口／字段 | 参数／默认 | 条件、作用与证据 |
|---|---|---|
| `SetSideRange(range)` | 默认1，世界距离 | 线段侧向容差加目标物理半径；不是突刺长度。粗查仍有默认 physicspadding=3，不承诺任意大宽度完整覆盖。[默认](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L6-L7)、[setter](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L21-L22)、[粗查与精测](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L77-L94) |
| `SetOnPreLungeFn(fn)`／`SetOnLungedFn(fn)` | `(item,doer,startingpos,targetpos)` | pre 在临时字段修改前；lunged 在命中、恢复、抛散和轨迹后。返回值不取消流程。[注册](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L25-L30)、[调用前](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L44-L55)、[完成](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L228-L232) |
| `SetTrailFX(prefab,spacing)` | 名称；spacing>0 才生成 | 沿路径反向排布 FX，可调用 FX 的 FastForward／SetMotion，并假定有 Transform／AnimState；需提供真实 Prefab 资源，未做生成失败保护。[设置](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L33-L35)、[生成](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L186-L224) |
| `SetSound(path)` | 默认未赋值 | 仅存 sound；消费方是 SG，未配时使用 SG 后备音效。[setter](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L38-L39)、[消费](../../../raw/current/scripts/stategraphs/SGwilson.lua#L17436-L17437) |
| `DoLunge(doer,startingpos,targetpos)` | 位置对象；false／true | 缺位置／doer／combat 时 false；否则完成流程即 true，不代表打中或允许发动。[检查](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L44-L51)、[返回](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L228-L232) |
| 继承的伤害／标签／工作与命中回调 | 见 [[reference/components/aoeweapon-base#关键接口]] | 自身 weapon 暂时用技能伤害且 attackwear=0；基类实际攻击由 doer 当前手持武器决定。[临时武器字段](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L57-L68)、[基类攻击](../../../raw/current/scripts/components/aoeweapon_base.lua#L119-L120)、[战斗选武器](../../../raw/current/scripts/components/combat.lua#L1120-L1132) |

### 沿线与终点的两段判定

有非零起终点距离时，粗查路径附近对象，以目标点到起终点线段的距离判是否进入 siderange；期间 ignorehitrange=true。随后置 false，再查终点附近，按人物当前朝向向前延伸 combat.hitrange，并同时用距离和侧宽过滤。[路径段](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L70-L98)、[终点段](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L101-L117)

toskip 在第一轮查询时就登记所有候选，并非只登记真正命中者；第二段不会再次尝试这些候选。起终点相同仍可执行终点攻击、抛散与单个轨迹 FX。真实效果不要简化为“无条件扫过矩形”。[去重登记](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L85-L94)、[终点过滤](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L103-L113)、[零距离轨迹](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L187-L190)

## 典型接入

### 调整现有闪电长矛的技能伤害，保留普攻与冷却

目标模组 `modmain.lua` 完整片段；普通、充能闪电长矛分别从 CommonFn 接到同一 lunge 基础配置，因此都显式登记。沿用原版动画、技能树／enabled 门禁和事件链，不新增资源、任务或持久字段。[两个工厂](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L549-L572)、[共同配置](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L481-L493)、[实际注册名](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L693-L694)

```lua
local function ConfigureLunge(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local lunge = inst.components.aoeweapon_lunge
    if lunge ~= nil then
        lunge:SetDamage(80)
    end
end

AddPrefabPostInit("spear_wathgrithr_lightning", ConfigureLunge)
AddPrefabPostInit("spear_wathgrithr_lightning_charged", ConfigureLunge)
```

80 是本例技能基础伤害替换点，不是保证最终扣血值；普通武器 damage 在技能结束后恢复，角色／目标的战斗修正仍有效。未覆盖原版 SetOnLungedFn：它生成落点 FX、放电并清命中修复计数；SetOnHitFn 还可按合法受害者限制修复次数。停用本例后重新加载物品回到原版配置。[临时恢复](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L119-L126)、[完成与命中回调](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L174-L194)

### 新造突刺武器的装配参照

闪电长矛在共同阶段显式加 aoeweapon_lunge 标签、挂 aoetargeting，然后 SetPristine 并服务端补 lunge／aoespell／rechargeable；CASTAOE 的两端 SG 依据标签进入突刺起手，施法函数发事件，再由 SG 定向、结算与移动。[共同配置](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L421-L438)、[工厂端分界](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L365-L384)、[技能配置](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L481-L497)、[服务端路由](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1312)、[客户端路由](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L606)

`spear_gungnir` 可参考直线 reticule 与轨迹 FX 的资源实现，但其服务端初始化转交熔炉 event_server_data，不能据公共文件补造完整攻击配置。[瞄准工厂与转交](../../../raw/current/scripts/prefabs/spear_gungnir.lua#L71-L95)、[轨迹 FX](../../../raw/current/scripts/prefabs/spear_gungnir.lua#L114-L145)

## 常见陷阱与限制

- **把 DoLunge 当瞬移接口**：函数只结算效果；SG 还要求 lunge_lag 动画并处理面向、地图边缘、物理质量与退出恢复。直接发事件或自调函数不保证整套技能成立。[SG 入场条件](../../../raw/current/scripts/stategraphs/SGwilson.lua#L17422-L17437)、[SG 位移](../../../raw/current/scripts/stategraphs/SGwilson.lua#L17487-L17504)
- **调大 siderange 就能打到任意宽度**：粗查由长度、combat.hitrange、固定 padding 决定；宽度只用于精测，大范围需重新核验选区实现。[粗查](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L77-L85)、[精测](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L90-L93)
- **替换完成回调只为了加 FX**：会覆盖原版放电／计数清理；闪电长矛的命中回调也有修复逻辑，增修时保留已核验协作。[原回调](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L174-L194)、[绑定](../../../raw/current/scripts/prefabs/spear_wathgrithr.lua#L485-L487)
- **回调后仍假设角色在原 SG 状态**：原版明确警告 DoLunge 可使角色退出当前状态；临时 combat 标志也未保存原值而直接重新启用／置 false，不是可重入的状态栈。[状态警告](../../../raw/current/scripts/stategraphs/SGwilson.lua#L17498-L17503)、[标志赋值](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L83)、[恢复](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L98)、[范围恢复](../../../raw/current/scripts/components/aoeweapon_lunge.lua#L119)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；相关调用与示例仍适用，本轮仅重定位证据（锚点 9 处），未运行游戏。

- 源码审阅：2026-10-23，Steam buildid 25643519；覆盖完整 lunge／base、Combat 武器选择、闪电长矛配置和回调、gungnir 公共入口与 FX、服务端突刺状态及客户端动作路由。未审校全部伤害修正或熔炉外置服务端数据。
- 结构检查：本页已执行页面格式／本地链接及严格源码引用检查；上述完整 modmain 示例通过 Lua 5.1 syntax-only，不代表名称绑定或游戏效果验证。
- 游戏运行：未运行；技能树／冷却、跨边缘位移、空距离、轨迹资源失败与其他模组叠加待实测。
