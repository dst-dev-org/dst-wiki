---
title: "homeseeker"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "记录生物家园与家园移除监听，发起回家动作；区分巢穴登记、Brain 时机、实际接收和保存重连。"
aliases: ["召回工蜂", "生物回家", "设置家园", "homeseeker"]
tags: [component, creature, ai, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/homeseeker.lua", "components/childspawner.lua", "brains/beecommon.lua", "brains/beebrain.lua", "prefabs/bee.lua", "prefabs/beehive.lua", "actions.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# homeseeker

## 快速判断

- 职责与适用：生物侧保存 home 引用、监听家园移除并可提交 GOHOME 动作；不自动找巢、不安排周期、不承诺已到家。[完整实现](../../../raw/current/scripts/components/homeseeker.lua#L1-L68)
- 优先使用：`SetHome` 换引用，`GetHome/HasHome` 查询，`GoHome(shouldrun)` 请求行动。巢穴生成的子体优先复用 [[reference/components/childspawner]] 的接管链，不能只改 home 就算加入巢穴。
- 重要副作用：家园移除时清 home，默认还移除 homeseeker 组件；调用方应动态取组件。HasHome 只判有引用且未燃烧，不保证接收方存在或承认该生物。[移除监听](../../../raw/current/scripts/components/homeseeker.lua#L1-L16)
- 修改边界：本页为主模拟工蜂与 childspawner 家园；Brain 决定何时回巢，GOHOME 决定如何接收，组件自身没有 OnSave／OnLoad。不能把通用动作当无副作用的到家测试。[行动](../../../raw/current/scripts/components/homeseeker.lua#L36-L47) [动作消费者](../../../raw/current/scripts/actions.lua#L2956-L2974)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetHome(home)`／`GetHome()` | 实体或 nil | 换引用前解除旧 home 的 onremove 监听，再监听新家；不校验新家有效性、计数归属、导航可达性，也不更新 knownlocations。[实现](../../../raw/current/scripts/components/homeseeker.lua#L22-L34) |
| `HasHome()` | boolean | home 非 nil 且无 burnable 或未燃烧；没有显式 IsValid／接收组件／归属检查。`GetHome` 仍可返回正在燃烧的家。[实现](../../../raw/current/scripts/components/homeseeker.lua#L14-L16) |
| `GoHome(shouldrun)` | 无显式成功返回 | 有 home 则创建 BufferedAction(doer, home, ACTIONS.GOHOME)；有 locomotor 用 PushAction，否则 PushBufferedAction。内部不调用 HasHome，不校验燃烧或归属，不瞬移、不直接接收。[实现](../../../raw/current/scripts/components/homeseeker.lua#L36-L47) |
| `GetHomePos()` | Vector3 或 nil | 返回当前 home 位置；不等于实际寻路终点或已找到可达路径。[实现](../../../raw/current/scripts/components/homeseeker.lua#L49-L51) |
| `GetHomeDirectTravelTime()` | 秒或 nil | 平面直线距离÷max(WILSON_WALK_SPEED, locomotor:GetWalkSpeed())；没有 locomotor 仍用默认速度，不是障碍／地形／跑步的 ETA。[实现](../../../raw/current/scripts/components/homeseeker.lua#L53-L65) |
| `removecomponent` | 默认 true | 家园 onremove 时先 SetHome(nil)，为 true 再移除此组件；设 false 只保留一个无家的组件，不会自动另找家。[实现](../../../raw/current/scripts/components/homeseeker.lua#L1-L12) |

### 家园、行为与存档的分工

childspawner 接管时可 RememberLocation("home")、动态添加本组件并 SetHome，随后登记外出；组件自身不会扫描附近巢穴认领。工蜂工厂已有 locomotor／SG／Brain，家园组件可在出巢接管时才出现，不能假定 bee 工厂结束时已有。[接管](../../../raw/current/scripts/components/childspawner.lua#L457-L475) [工厂移动](../../../raw/current/scripts/prefabs/bee.lua#L174-L181) [Brain](../../../raw/current/scripts/prefabs/bee.lua#L260-L279)

原工蜂 Brain 在夜暗、花粉足够或冬季分支尝试回家；`beecommon.GoHomeAction` 直接构造动作，检查 home 有效、带 childspawner 且未燃烧，并没有调用 `homeseeker:GoHome`。只包装这个方法不会改变 Brain 的全部回巢路径；战斗等更高优先级也会影响实际行动。[条件与优先级](../../../raw/current/scripts/brains/beebrain.lua#L57-L81) [动作构造](../../../raw/current/scripts/brains/beecommon.lua#L41-L50) 花粉门槛见 [[reference/components/pollinator]]。

GOHOME 优先转发目标 spawner，其次 childspawner，再 hideout；没有这些接收组件时，可能发 onwenthome 并直接 Remove 生物。childspawner 只接收在自己外出表登记的生物，成功发事件、移除实体、加巢内数量。SetHome 不负责完成这些账目，也不保证回家保留生命／任意字段。[动作](../../../raw/current/scripts/actions.lua#L2956-L2974) [归属与接收](../../../raw/current/scripts/components/childspawner.lua#L668-L715)

本组件没有存读回调；childspawner 模式由巢穴保存外出 GUID，再在 LoadPostPass 接管已重建实体、恢复 home。自行创建且只 SetHome 的生物没有因此自动获得家园持久化，需核验实际宿主保存链。[巢保存](../../../raw/current/scripts/components/childspawner.lua#L332-L377) [重连](../../../raw/current/scripts/components/childspawner.lua#L483-L503)

## 典型接入

### 为已归属蜂箱的工蜂提供服务端召回事件

放目标模组 `modmain.lua`，无需资源。为原 `bee` 注册自定义事件 `my_mod_recall_bee`，由你的服务端逻辑在选定工蜂上发送；本例交付事件接收端，不包含按键、范围选蜂或客户端 RPC。每次请求动态取家园组件，再检查有效性、燃烧与普通子体登记后请求 GoHome，避免把任意 SetHome 当入巢资格。[原工蜂](../../../raw/current/scripts/prefabs/bee.lua#L260-L279) [归属检查](../../../raw/current/scripts/components/childspawner.lua#L701-L715) [Hook](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("bee", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:ListenForEvent("my_mod_recall_bee", function(bee)
        if bee.components.health == nil or bee.components.health:IsDead() then
            return
        end
        local seeker = bee.components.homeseeker
        if seeker == nil then return end
        local home = seeker:GetHome()
        if home == nil or not home:IsValid() or not seeker:HasHome() then
            return
        end
        local spawner = home.components.childspawner
        if spawner ~= nil and home.components.spawner == nil
            and spawner.childrenoutside[bee] ~= nil then
            seeker:GoHome(true)
        end
    end)
end)
```

服务端调用方在选定实体上发 `bee:PushEvent("my_mod_recall_bee")`；替换事件前缀时收发两边一致。`true` 请求跑步，实际执行仍受 locomotor／SG／Brain、路径和巢穴接收条件影响；不承诺打断战斗、强制回家或在死亡／移除竞态下成功。额外排除 spawner 是因为 GOHOME 优先转发它，避免检查 childspawner 后走另一接收路径；不泛化为应急子体或其他巢穴方案。[提交动作](../../../raw/current/scripts/components/homeseeker.lua#L36-L47) [分发顺序](../../../raw/current/scripts/actions.lua#L2961-L2971)

本例不重新设 home、不接管、不直调接收方 Remove，保留原 Brain 的自然回巢；停用并重载后不再注册自定义事件，已有家园仍由原巢重连。人物随从归属与实际跟随另见 [[reference/components/follower]]，不要用 homeseeker 代替主人跟随系统。

## 常见陷阱与限制

- **GoHome 返回 nil 就是失败**：方法只提交动作，没有成功返回；应区分提交、移动、动作执行和巢穴 childgoinghome／子体 goinghome 事件，不能立刻按返回值补生成一只。[方法](../../../raw/current/scripts/components/homeseeker.lua#L36-L47) [接收事件](../../../raw/current/scripts/components/childspawner.lua#L668-L684)
- **HasHome 为 true 即安全回家**：未检查实体有效性或接收组件；巢还在但 childspawner 已被移除时，通用动作可走删除分支。召回前按目标类型检查归属与实际分发路径。[判断](../../../raw/current/scripts/components/homeseeker.lua#L14-L16) [野巢移组件](../../../raw/current/scripts/prefabs/beehive.lua#L77-L85) [动作](../../../raw/current/scripts/actions.lua#L2956-L2971)
- **SetHome(nil) 等于解除巢穴登记**：只去监听和清引用，原 childspawner 的外出表／监听不随之转移；不要以这一个方法实现换巢。默认家园移除还会卸掉组件，不能跨生命周期缓存 seeker。[引用操作](../../../raw/current/scripts/components/homeseeker.lua#L1-L34) [巢登记](../../../raw/current/scripts/components/childspawner.lua#L457-L475)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；漂移文件 actions.lua、prefabs/beehive.lua 对照 R-01/R-04 hunk 地图逐条核验：GOHOME 动作（spawner→childspawner→hideout 分发与 onwenthome／Remove 分支）逐行未变，beehive 仅移除 lifedrainable 标签（:169，引用段 77-85 在其之前不受影响）；锚点重定位 4 处、结论无修订；未覆盖：运行验证。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖组件全部方法／默认移除监听、工蜂移动与 Brain／公共动作路径、巢穴普通子体接收、GUID 家园重连和召回事件前置条件。spawner／hideout 只核对 GOHOME 的分派，不声称其内部接收或保存方式已审校。
- 结构检查：格式、来源、链接／锚点、大小和唯一登记通过；完整事件接收端示例通过已有 Full Moon Lua 5.1 syntax-only，不代表事件已发送或寻路／回巢成功。
- 游戏运行：未运行；事件发送、战斗／行动竞争、阻路、燃烧／巢毁、捕获、读档、远端客户端及多模组接收组件变化待实测。
