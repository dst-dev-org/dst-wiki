---
title: "aoeweapon_leap"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "落点圆形跳砸的命中、物品抛散与技能回调；人物起跳和落地位移由状态机执行。"
aliases: ["aoeweapon_leap", "跳砸武器", "落点范围攻击", "雷神锤跳跃", "超级跳跃武器"]
tags: [component, actions, callback, state-machine, equipment]
dst_build: "steam-buildid:26230496"
sources: [components/aoeweapon_leap.lua, components/aoeweapon_base.lua, stategraphs/SGwilson.lua, stategraphs/SGwilson_client.lua, prefabs/hammer_mjolnir.lua, prefabs/spear_lance.lua]
reliability: verified
validation: source-reviewed
status: active
---

# aoeweapon_leap

## 快速判断

- 职责与适用：继承 [[reference/components/aoeweapon-base]]，围绕 targetpos 结算跳砸；需要圆形落点攻击选它，沿线突刺选 [[reference/components/aoeweapon-lunge]]。[继承与配置](../../../raw/current/scripts/components/aoeweapon_leap.lua#L1-L13)、[范围结算](../../../raw/current/scripts/components/aoeweapon_leap.lua#L55-L63)
- 优先使用：沿原版 CASTAOE → combat_leap_start → SG 落地时间线 → DoLeap 的结构接入；`superjump` 标签选择另一套 SG，但仍调用 DoLeap。[动作路由](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1304-L1313)、[普通落地调用](../../../raw/current/scripts/stategraphs/SGwilson.lua#L17669-L17678)、[超级跳落地](../../../raw/current/scripts/stategraphs/SGwilson.lua#L17935-L17947)
- 重要副作用：每个候选交给基类命中分流，再独立抛散落点地面物品；结算期间临时改战斗范围标志和物品 weapon 的伤害／磨损。[命中前后](../../../raw/current/scripts/components/aoeweapon_leap.lua#L39-L73)、[抛散](../../../raw/current/scripts/components/aoeweapon_leap.lua#L78-L87)
- 修改边界：组件不移动 doer、不检查冷却、施法距离或落点通行；原版 SG 才负责运动与落地。服务端结算，并在共同工厂显式预加 `aoeweapon_leap` 标签供两端路由。[DoLeap](../../../raw/current/scripts/components/aoeweapon_leap.lua#L30-L93)、[共同标签](../../../raw/current/scripts/prefabs/hammer_mjolnir.lua#L65-L84)、[客户端路由](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L606-L607)

## 关键接口

| 接口／字段 | 参数／默认 | 条件、作用与证据 |
|---|---|---|
| `SetAOERadius(radius)` | 默认 4，世界距离 | 命中精测为 radius+目标物理半径；另以 radius+physicspadding 粗查，不能视为无碰撞半径的固定圆。[默认](../../../raw/current/scripts/components/aoeweapon_leap.lua#L6-L7)、[设置](../../../raw/current/scripts/components/aoeweapon_leap.lua#L16-L17)、[精测](../../../raw/current/scripts/components/aoeweapon_leap.lua#L55-L63) |
| `physicspadding` | 默认 3；无专用 setter | 只扩粗查询，任意超大碰撞体未必被覆盖；半径不是飞跃距离。[查询](../../../raw/current/scripts/components/aoeweapon_leap.lua#L55) |
| `SetOnPreLeapFn(fn)` | `(item,doer,startingpos,targetpos)` | 输入基础检查后、临时改战斗字段之前调用；返回值不作取消信号。[注册](../../../raw/current/scripts/components/aoeweapon_leap.lua#L20-L21)、[调用](../../../raw/current/scripts/components/aoeweapon_leap.lua#L30-L40) |
| `SetOnLeaptFn(fn)` | 同上 | 命中、字段恢复与抛散之后调用；此处适合宿主接一次性消耗或效果，但组件未内置消耗。[完成](../../../raw/current/scripts/components/aoeweapon_leap.lua#L66-L93) |
| `DoLeap(doer,startingpos,targetpos)` | 两位置对象；返回 false／true | 仅缺位置／doer／combat 时 false；其余完成流程即 true，空范围也 true。startingpos 传给回调，圆心使用 targetpos。[入口](../../../raw/current/scripts/components/aoeweapon_leap.lua#L30-L40)、[圆心](../../../raw/current/scripts/components/aoeweapon_leap.lua#L55)、[返回](../../../raw/current/scripts/components/aoeweapon_leap.lua#L90-L93) |
| 继承的伤害／标签／工作与命中回调 | 见基类 | 命中优先工作／采摘／战斗，技能物品须与角色攻击武器配套；配置边界见 [[reference/components/aoeweapon-base#关键接口]]。[继承](../../../raw/current/scripts/components/aoeweapon_leap.lua#L1-L4) |

### 圆形结算顺序

通过入口检查 → pre 回调 → 关闭 doer 的普通附带范围攻击、置 ignorehitrange=true → 临时把技能物品 weapon.attackwear 置0、damage 改成技能值 → 查询并排除自身、无效／limbo／死亡对象，以 `< (aoeradius+物理半径)^2` 判定 → 基类 OnHit → 重新启用范围攻击、置 ignorehitrange=false、恢复 weapon 字段 → 独立查询 `_inventoryitem`，排除 locomotor／INLIMBO，地面高度 `<0.2` 且落入半径则 OnToss → leapt 回调 → true。[完整顺序](../../../raw/current/scripts/components/aoeweapon_leap.lua#L30-L93)

## 典型接入

### 跳到选点后砸地，与超级跳跃共用伤害结算

普通参照 `hammer_mjolnir`：资源和 reticule 依赖、共同标签与 aoetargeting 见[工厂](../../../raw/current/scripts/prefabs/hammer_mjolnir.lua#L1-L87)。`spear_lance` 额外带 superjump、配置不同 reticule 和选点范围，走另一套人物动画状态。[lance 工厂](../../../raw/current/scripts/prefabs/spear_lance.lua#L42-L76)

这两个工厂的服务端初始化都转交 `event_server_data("lavaarena",...)`；本页不把公共工厂当完整可复制技能 demo，不推定外置回调的伤害、冷却或事件参数。[hammer 转交](../../../raw/current/scripts/prefabs/hammer_mjolnir.lua#L87)、[lance 转交](../../../raw/current/scripts/prefabs/spear_lance.lua#L76)

已核验的 SG 实现负责普通跳砸的朝向与12帧飞行、12帧落地、13帧 DoLeap；超级跳则在落地后4帧调用。新造物需要自行把 [[reference/components/aoetargeting]]、[[reference/components/aoespell]]、weapon、两端动作路由、资源与服务端事件接全，仅挂本组件或直接 DoLeap 不会跳跃。[普通运动](../../../raw/current/scripts/stategraphs/SGwilson.lua#L17606-L17624)、[普通落地](../../../raw/current/scripts/stategraphs/SGwilson.lua#L17660-L17678)、[超级跳](../../../raw/current/scripts/stategraphs/SGwilson.lua#L17935-L17947)

## 常见陷阱与限制

- **把成功返回当命中或发动授权**：true 只表示函数跑完；不重验距离、地图、冷却或装备，不能取代技能入口条件。[入口与返回](../../../raw/current/scripts/components/aoeweapon_leap.lua#L30-L40)、[结束](../../../raw/current/scripts/components/aoeweapon_leap.lua#L90-L93)
- **一群目标就消耗一群耐久**：原实现临时 attackwear=0，再恢复原值；一次施法的资源消耗要沿宿主回调核验，不能根据目标数自行扣。它仍调用 weapon 的常规攻击链，非耐久副作用不因此全部禁用。[临时字段](../../../raw/current/scripts/components/aoeweapon_leap.lua#L42-L52)、[恢复](../../../raw/current/scripts/components/aoeweapon_leap.lua#L68-L73)
- **结束会恢复原 combat 配置**：代码直接设 EnableAreaDamage(true)、ignorehitrange=false，未保存进入前的值；不要在自定义嵌套攻击中假设状态完整复原。回调抛错也没有保护恢复段。[开始](../../../raw/current/scripts/components/aoeweapon_leap.lua#L39-L40)、[结束](../../../raw/current/scripts/components/aoeweapon_leap.lua#L66-L73)
- **改命中标签也改变落地物品抛散**：抛散是独立筛选，详见 [[reference/components/aoeweapon-base#常见陷阱与限制]]；原版范围命中和物品弹起并不是同一列表。[两路查询](../../../raw/current/scripts/components/aoeweapon_leap.lua#L55)、[抛散查询](../../../raw/current/scripts/components/aoeweapon_leap.lua#L78-L87)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 6 处（SGwilson CASTAOE 路由 +38、combat_leap／superjump 段 +120~+121、SGwilson_client 路由 +12）；无结论级修订——aoeweapon_base 本版 OnToss 内部精化（mine 停用改走 DeactivateInventoryItemBeforeLaunch、落点传送失败回退原位上方并 SetLanded，aoeweapon_base.lua:142-189），本页"独立抛散查询→OnToss"的分工与顺序结论不变；aoeweapon_leap.lua、两工厂未变。依据：player-hubs-hunk-map 41 断点偏移表＋变更清单命中 SGwilson／SGwilson_client／components/aoeweapon_base.lua。
- 源码审阅：2026-10-23，Steam buildid 25643519；覆盖完整 leap／base、hammer 与 lance 公共工厂、服务端普通／超级跳落地及客户端 CASTAOE 路由。外置熔炉服务端数据未作为已核验完整接入。
- 结构检查：本页已执行页面格式／本地链接及严格源码引用检查；没有将不完整外置初始化拼成完整 Lua 示例。
- 游戏运行：未运行；联机预测、异常中断、地图边缘、特殊物理目标与技能消耗待实测。
