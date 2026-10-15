---
title: repellent：解散随从与清除附近战斗目标
created: 2026-10-23
updated: "2026-10-28"
type: component
summary: 配置驱散工具的随从标签、附近搜索与耐久消耗，区分解除跟随和一次性清战斗目标。
aliases: [repellent, 蜘蛛驱散器, 解散蜘蛛随从, 清除附近攻击目标]
tags: [component, item, ai, actions, callback]
dst_build: "steam-buildid:26230496"
sources: [components/repellent.lua, components/follower.lua, componentactions.lua, actions.lua, stategraphs/SGwilson.lua, prefabs/spider_repellent.lua, prefabs/player_common.lua, prefabs/webber.lua, modutil.lua]
reliability: verified
validation: source-reviewed
status: active
---

# repellent：解散随从与清除附近战斗目标

## 快速判断

- 职责与适用：工具一次使用先尝试解除使用者的匹配随从，再按开关清工具附近匹配实体的战斗目标，最后扣次数；不是恐惧、逃跑或持续驱敌光环。[Repel 全流程](../../../raw/current/scripts/components/repellent.lua#L82-L106)
- 优先使用：改原版 `spider_repellent` 的 `SetOnlyRepelsFollowers`／标签／半径；跟随关系与重新选敌分别参考 [[reference/components/follower]]、[[reference/components/combat]]。[原版配置](../../../raw/current/scripts/prefabs/spider_repellent.lua#L24-L39)
- 重要副作用：随从分支不使用 `ignore_tags`，附近战斗分支才用；没有匹配对象也会消耗 `finiteuses`。`Repel` 无成功返回值，`ACTIONS.REPEL` 在组件存在时返回 true。[过滤实现](../../../raw/current/scripts/components/repellent.lua#L34-L48) [消费](../../../raw/current/scripts/components/repellent.lua#L82-L106) [动作返回](../../../raw/current/scripts/actions.lua#L5670-L5676)
- 修改边界：服务端组件直接依赖 `doer.components.leader.followers`；原版玩家具备 leader，不能直接给任意无 leader 的实体调用。菜单另要求使用者有 `spiderwhisperer`，不是组件内部限制。[玩家挂载](../../../raw/current/scripts/prefabs/player_common.lua#L2799-L2801) [动作门禁](../../../raw/current/scripts/componentactions.lua#L2873-L2875)

## 关键接口

| 接口／字段 | 条件、默认值与作用 |
|---|---|
| `AddRepelTag(tag)`／`RemoveRepelTag(tag)` | 去重增删 `repel_tags`；随从必须拥有其中全部标签；初始空表。[标签表](../../../raw/current/scripts/components/repellent.lua#L1-L7) [筛选](../../../raw/current/scripts/components/repellent.lua#L34-L56) |
| `AddIgnoreTag(tag)`／`RemoveIgnoreTag(tag)` | 初始忽略 FX／NOCLICK／DECOR／INLIMBO；仅作为附近 `FindEntities` 的排除标签，`ValidateTargetTags` 内对应检查已注释。[忽略表](../../../raw/current/scripts/components/repellent.lua#L1-L7) [过滤差异](../../../raw/current/scripts/components/repellent.lua#L34-L64) [附近搜索](../../../raw/current/scripts/components/repellent.lua#L92-L101) |
| `SetOnlyRepelsFollowers(enabled)` | 默认 false；true 仅跳过附近清战斗分支，仍先尝试解散全部匹配随从并扣次数。[配置](../../../raw/current/scripts/components/repellent.lua#L78-L80) [执行](../../../raw/current/scripts/components/repellent.lua#L82-L106) |
| `SetRadius(radius)` | 附近搜索半径，未设置时为 1；中心是工具 `self.inst` 的世界坐标，随从分支不检查半径。[配置与消费](../../../raw/current/scripts/components/repellent.lua#L66-L68) [搜索](../../../raw/current/scripts/components/repellent.lua#L92-L101) |
| `SetUseAmount(amount)` | 未设置时每次扣 1；仅在工具存在 `finiteuses` 时调用 `Use`，原版设为 10，耗尽回调移除工具。[配置与消费](../../../raw/current/scripts/components/repellent.lua#L70-L72) [扣次](../../../raw/current/scripts/components/repellent.lua#L103-L106) [宿主](../../../raw/current/scripts/prefabs/spider_repellent.lua#L32-L39) |
| `SetOnRepelFollowerFn(fn)` | `fn(tool, follower)` 在匹配随从调用 `StopFollowing()` 后执行，不接收 doer，也不保证跟随已解除。[设置](../../../raw/current/scripts/components/repellent.lua#L74-L76) [调用](../../../raw/current/scripts/components/repellent.lua#L82-L90) |

`Repel(doer)` 遍历 leader 名单而非找附近盟友，匹配者调用 `follower:StopFollowing()`；该方法遇到 `neverexpire` 会直接返回，回调仍会继续执行。随后附近分支只检查有 combat 且已有目标，直接 `DropTarget()`，不区分敌我、目标是否为使用者，也不禁止 Brain 后续重新选敌。[随从分支](../../../raw/current/scripts/components/repellent.lua#L82-L101) [永久随从例外](../../../raw/current/scripts/components/follower.lua#L399-L410)

## 典型接入

### 蜘蛛驱散器只解散自己的蜘蛛随从，不清周围战斗目标

目标模组 `modmain.lua`，沿用原版工具、美术、韦伯菜单与动画。原版工厂在服务端挂载组件，设 `spider` 必需标签、`spiderqueen` 排除标签、搜索半径与每次 10 点耐久；本例只关闭第二阶段。[工厂／注册](../../../raw/current/scripts/prefabs/spider_repellent.lua#L24-L49) [PostInit](../../../raw/current/scripts/modutil.lua#L593-L595)

```lua
AddPrefabPostInit("spider_repellent", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local repellent = inst.components.repellent
    if repellent ~= nil then
        repellent:SetOnlyRepelsFollowers(true)
    end
end)
```

效果在每次 `REPEL` 动作执行时发生，不是初始化时立刻解散；玩家 SG 在第 17 帧执行动作。保留原版 finiteuses 和耗尽移除；无随从时仍扣次数。字段由初始化配置，没有组件自己的存读回调；停用后新初始化工具恢复原版开关，已解除关系不自动恢复。[SG 执行](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22728-L22742) [完整组件](../../../raw/current/scripts/components/repellent.lua#L1-L108)

不通过给所有角色加 `spiderwhisperer` 来扩展菜单，该标签还承担其他角色行为；若另造通用工具，需独立核验动作及 SG 资源。韦伯另加载驱散动画 build，本例不改它。[门禁](../../../raw/current/scripts/componentactions.lua#L2873-L2875) [韦伯动画](../../../raw/current/scripts/prefabs/webber.lua#L52-L60)

## 常见陷阱与限制

- **给蜘蛛女王加 IgnoreTag 就绝不解约**：忽略表不参与随从检查，原版 `spiderqueen` 排除也只用于附近搜索；要排除随从需调整筛选设计，不依赖被注释的代码。[过滤差异](../../../raw/current/scripts/components/repellent.lua#L34-L48)
- **以回调／动作 true 证明成功驱散**：永久随从可能没有解除；无匹配目标也会返回动作 true 并消耗。需要结果时自行核验关系／combat 状态，不能用 `Repel` 的返回值。[回调和消费](../../../raw/current/scripts/components/repellent.lua#L82-L106) [动作](../../../raw/current/scripts/actions.lua#L5670-L5676)
- **当作范围和平或击退**：随从不限距离，附近分支不移动实体也不施加持续约束；只清当前 combat 目标，原 AI 可再获取。只有 `onlyfollowers=true` 才彻底跳过这一分支。[两阶段](../../../raw/current/scripts/components/repellent.lua#L82-L101)

## 验证记录

- 版本沿用：2026-10-25，`steam-buildid:26148036`；原结论复用、锚点迁移 4 处（actions.lua REPEL、componentactions.lua 门禁、SGwilson.lua repel_followers、player_common.lua leader 挂载）；依据：actions.lua／componentactions.lua 枢纽 diff 审阅＋变更清单命中 actions.lua、componentactions.lua、SGwilson.lua、player_common.lua、webber.lua，第 17 帧执行、spiderwhisperer 门禁与 leader 挂载同旧版；webber 本版仅给 attacked 监听加装备判定，不涉本页结论。未运行游戏。
- 源码审阅：2026-10-23，Steam buildid 25643519；覆盖完整组件、标签／半径／消费分支、永久随从例外、spider_repellent 工厂与服务端配置、玩家 leader、REPEL 菜单／动作／SG 以及韦伯资源接入。
- 结构检查：本页格式、链接、源码锚点与示例 Lua 5.1 语法检查通过。
- 游戏运行：未运行；永久随从、附近重选敌、读档与其他模组重配字段、多人动画与耐久表现待实测。
