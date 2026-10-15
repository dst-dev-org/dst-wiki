---
title: "rooted"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "按来源集合管理目标的零移动倍率与临时零物理质量，在最后来源撤销时恢复。"
aliases: ["rooted", "多来源定身", "根须束缚", "临时零质量", "禁止行走", "AddSource", "RemoveSource"]
tags: [component, lifecycle, event, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/rooted.lua", "components/locomotor.lua", "prefabs/wormwood_vined_debuff.lua", "prefabs/shadow_pillar.lua", "prefabs/armor_lunarplant.lua", "components/debuff.lua", "components/debuffable.lua", "components/timer.lua", "entityscript.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# rooted

## 快速判断

- 职责与适用：动态挂在**被束缚的目标**上；构造即添加 `rooted` 标签、停止当前物理运动、启用 `Physics:SetTempMass0(true)`，并在已有 locomotor 上挂目标自身／`"rooted"` 键的 0 倍速。不是附着视觉实体的组件。[rooted.lua:1](../../../raw/current/scripts/components/rooted.lua#L1
- 优先使用：各效果实体作为 `AddSource(src)` 的来源，结束时 `RemoveSource(src)` 或删除来源；最后来源消失会移除整个组件，恢复该物理开关与速度来源。[rooted.lua:19](../../../raw/current/scripts/components/rooted.lua#L19 [rooted.lua:51](../../../raw/current/scripts/components/rooted.lua#L51
- 重要副作用：只负责上述物理／速度和 `rooted`、`unrooted` 事件；没有停止脑、清战斗目标、关闭控制器、受击脱困、通用计时或动画逻辑。黏液受困应另看 [[reference/components/pinnable]]。[rooted.lua:26](../../../raw/current/scripts/components/rooted.lua#L26 [rooted.lua:29](../../../raw/current/scripts/components/rooted.lua#L29
- 修改边界：原版由主模拟端的藤蔓 debuff、暗影牢笼目标管理实体动态安装；资格、超时、死亡／传送撤销和保存重施均由这些施加者处理。[wormwood_vined_debuff.lua:12](../../../raw/current/scripts/prefabs/wormwood_vined_debuff.lua#L12 [shadow_pillar.lua:482](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L482

## 关键接口

| 接口／字段 | 参数、返回或约束 | 条件、作用与证据 |
|---|---|---|
| 添加组件 | `target:AddComponent("rooted")` | 立即施加一次物理／速度处理并发 `rooted`；无需等到首个来源。添加前通常先确认组件不存在。[rooted.lua:1](../../../raw/current/scripts/components/rooted.lua#L1 [shadow_pillar.lua:482](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L482 |
| `AddSource(src)` | 有效来源实体；无返回值 | `sources[src]=true`，同实体重复添加幂等；非目标自身的来源注册 `onremove` 监听。[rooted.lua:51](../../../raw/current/scripts/components/rooted.lua#L51 |
| `RemoveSource(src)` | 已登记的同一实体；无返回值 | 删除本来源监听和集合项；集合为空才 `RemoveComponent("rooted")`，不存在的来源无作用。[rooted.lua:60](../../../raw/current/scripts/components/rooted.lua#L60 |
| `sources` | 以实体为键的集合 | 不是引用计数；同一实体两次添加、一次撤销即移除。目标自身作为来源时不监听自身删除，存活期间需显式撤销。[rooted.lua:19](../../../raw/current/scripts/components/rooted.lua#L19 [rooted.lua:51](../../../raw/current/scripts/components/rooted.lua#L51 |
| `OnRemoveFromEntity()` | 移除组件自动调用 | 去标签、`SetTempMass0(false)`、移除目标自身／`"rooted"` 倍率、注销来源监听、发 `unrooted`；不会恢复旧物理速度。[rooted.lua:29](../../../raw/current/scripts/components/rooted.lua#L29 |

所有来源共用**一个**速度键；增减来源不会重设数值。locomotor 按独立来源／键管理并消费外部倍率，撤销 rooted 的键不直接清除其他键。[locomotor.lua:178](../../../raw/current/scripts/components/locomotor.lua#L178 [locomotor.lua:490](../../../raw/current/scripts/components/locomotor.lua#L490 [locomotor.lua:515](../../../raw/current/scripts/components/locomotor.lua#L515

原版藤蔓顺序是 `OnAttached` → 添加 rooted → `AddSource(effect)` → 监听目标死亡／入 limbo／传送／删除，以及新状态带 `flight` → `debuff:Stop()`；到期 timer 也 Stop。`OnDetached` 显式 `RemoveSource(effect)`，然后播放退出动画再删除 FX，因此移动恢复不必等动画结束。[wormwood_vined_debuff.lua:12](../../../raw/current/scripts/prefabs/wormwood_vined_debuff.lua#L12 [wormwood_vined_debuff.lua:45](../../../raw/current/scripts/prefabs/wormwood_vined_debuff.lua#L45 [wormwood_vined_debuff.lua:55](../../../raw/current/scripts/prefabs/wormwood_vined_debuff.lua#L55

## 典型接入

### 保留原藤蔓宿主，只修改新效果的定身时长

以下为已启用模组的完整 `modmain.lua`，直接加载，不新增 `PrefabFiles` 或资源。沿用原版 `wormwood_vined_debuff` 的挂接、来源和清理；原工厂已有 `weed_ivy_lunar`、`farm_soil` 动画资源、网络实体、服务端 debuff 与 timer。[modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593 [wormwood_vined_debuff.lua:63](../../../raw/current/scripts/prefabs/wormwood_vined_debuff.lua#L63

```lua
local ROOT_SECONDS = 3
local TIMER_NAME = "stop_vined_debuff"

AddPrefabPostInit("wormwood_vined_debuff", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    local timer = inst.components.timer
    if timer ~= nil and timer:TimerExists(TIMER_NAME) then
        timer:SetTimeLeft(TIMER_NAME, ROOT_SECONDS)
    end
end)
```

替换点为非负游戏秒值 `3`；Prefab 名和 timer 键与原工厂对应，不是任意替换参数。示例只改**新建藤蔓效果**的初始计时，不自行选择／束缚目标。原版触发之一是已激活相应沃姆伍德技能的亮茄盔甲穿戴者受击后，给有 locomotor 且未死亡的攻击者 `AddDebuff`；随后目标仍须通过 `AddDebuff` 的启用／存活资格。[armor_lunarplant.lua:26](../../../raw/current/scripts/prefabs/armor_lunarplant.lua#L26 [entityscript.lua:2140](../../../raw/current/scripts/entityscript.lua#L2140

自然到期、主动 `RemoveDebuff` 和上述目标状态变化均走原脱离链；其他 rooted 来源仍在时不会整体解禁。新效果保存由目标 debuffable 保存其记录，恢复时 timer 的保存剩余覆盖工厂初值，重新 Attach 再登记来源；本示例不强制把读档剩余改成 3 秒。[debuffable.lua:110](../../../raw/current/scripts/components/debuffable.lua#L110 [debuffable.lua:126](../../../raw/current/scripts/components/debuffable.lua#L126 [timer.lua:135](../../../raw/current/scripts/components/timer.lua#L135

另一原版模式：`shadow_pillar_target` 用自己做来源，由寿命 timer／目标离位等删除自身来撤销；它保存目标引用和 timer，并在 `OnLoadPostPass` 重建 rooted 来源。显示牢笼的 `shadow_pillar` 与真正管理来源的实体职责不同。[shadow_pillar.lua:416](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L416 [shadow_pillar.lua:444](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L444 [shadow_pillar.lua:553](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L553

## 常见陷阱与限制

- **只添加组件不登记来源**：构造已定身，但空来源不会自动检查并移除；它没有自带超时。紧接着登记真实来源，并保证显式撤销或来源删除。[rooted.lua:1](../../../raw/current/scripts/components/rooted.lua#L1
- **到期直接移除整个组件**：会一起清掉其他效果的定身；正常只 `RemoveSource` 本效果。使用字符串／普通表作为来源也不符合实体监听所需接口。[rooted.lua:42](../../../raw/current/scripts/components/rooted.lua#L42 [rooted.lua:51](../../../raw/current/scripts/components/rooted.lua#L51
- **当作完全禁止位移／行动**：没有拦截传送或持续停止所有运动；原牢笼还检测目标离位，藤蔓另监听 `teleported`。缺 Physics／locomotor 时相应处理跳过；之后才新增的组件不会被构造逻辑自动补处理。[rooted.lua:7](../../../raw/current/scripts/components/rooted.lua#L7 [shadow_pillar.lua:444](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L444
- **从名字推断免疫规则**：rooted 内部不检查 boss、飞行、死亡、阵营、无敌或 debuffable；原牢笼施法另筛 locomotor 标签、PVP 玩家、flight 等排除标签、可见／存活／盟友和地面。自定义施加者必须选定自己的资格与解除边界。[shadow_pillar.lua:719](../../../raw/current/scripts/prefabs/shadow_pillar.lua#L719
- **直接叠加 `SetTempMass0` 或改正常质量代替来源**：组件清理总会设临时开关为 false，多个私有开关调用没有来源计数。源码建议外部束缚用 rooted，Prefab 自有质量逻辑继续管理 `Physics:SetMass`。[rooted.lua:7](../../../raw/current/scripts/components/rooted.lua#L7
- **期待组件自动保存来源／重复施加刷新藤蔓**：rooted 没有保存接口；藤蔓没有 Extended 回调，同 debuff 键重复施加不会重置它的 timer。保存重施、刷新策略需按实际效果工厂核对。[wormwood_vined_debuff.lua:88](../../../raw/current/scripts/prefabs/wormwood_vined_debuff.lua#L88 [debuffable.lua:95](../../../raw/current/scripts/components/debuffable.lua#L95 [debuff.lua:53](../../../raw/current/scripts/components/debuff.lua#L53

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 10 处（shadow_pillar／wormwood_vined_debuff／entityscript 行号迁移；armor_lunarplant.lua#L26 函数签名新增 inst 参但行号未变，ShouldProc 门控后 AddDebuff 链保留，结论不变）；结论修订：无，仅锚点迁移。
- 依据：全局枢纽 hunk 地图＋变更清单命中 entityscript.lua、prefabs/shadow_pillar.lua、prefabs/wormwood_vined_debuff.lua、prefabs/armor_lunarplant.lua。

- 源码审阅：2026-10-21，Steam buildid 25643519；完整阅读 `components/rooted.lua`、藤蔓工厂，核验来源增删／移除清理、locomotor 消费、盔甲资格、debuff 管理／保存、timer 恢复，以及暗影牢笼目标管理、筛选和读档重施。原始脚本只读。
- 示例范围：只改原效果的初始 timer，保留原 Attach／Detach；完整 `modmain.lua` 经 Lua 5.1 语法检查。页面元数据／大小／本地链接与源码行锚点检查通过。
- 游戏运行：未运行。多来源交叠、到期／死亡／传送／flight 撤销、船体物理、保存恢复和客户端表现仍需实测；未穷举所有移动机制与各生物 SG。
