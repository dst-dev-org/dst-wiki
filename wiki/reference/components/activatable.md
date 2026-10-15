---
title: "activatable"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "实体激活资格与回调桥接；inactive 标签、执行返回和 onactivated 通知分开，重复可用由宿主恢复。"
aliases: ["栅栏门拒绝非玩家激活", "激活回调返回nil", "activatable"]
tags: [component, actions, callback, event, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/activatable.lua", "componentactions.lua", "actions.lua", "prefabs/fence.lua", "stategraphs/SGwilson_client.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# activatable

## 快速判断

- 职责与适用：装在被激活实体，管理 inactive／动作标签、CanActivateFn 和 OnActivate；开门／机关实际效果由宿主回调实现。[构造](../../../raw/current/scripts/components/activatable.lua#L21-L38) [栅栏门组装](../../../raw/current/scripts/prefabs/fence.lua#L732-L735)
- 优先使用：CanActivate 检查、DoActivate 执行；回调字段直接配置，无 SetOnActivateFn。原 ACTIONS.ACTIVATE 还会拒绝冒烟／燃烧，不由组件方法包办。[组件](../../../raw/current/scripts/components/activatable.lua#L48-L69) [动作](../../../raw/current/scripts/actions.lua#L3095-L3107)
- 重要副作用：有 OnActivate 时先 inactive=false，再调用回调；只有回调真值才发 onactivated {doer}，失败／nil 都不自动回滚 inactive。[顺序](../../../raw/current/scripts/components/activatable.lua#L58-L68)
- 修改边界：CanActivateFn 会覆盖默认 inactive 判断；DoActivate 不再调用 CanActivate。标签候选、服务端测试、直接调用与成功事件是不同层，不能把一个字段当完整权限锁。[测试](../../../raw/current/scripts/components/activatable.lua#L48-L55) [候选](../../../raw/current/scripts/componentactions.lua#L142-L155)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `inactive` | 默认 true，联动 inactive 标签；不是“组件关闭”。候选通常需要此标签，原门 ToggleDoor 自行恢复 true 以反复开关。[标签](../../../raw/current/scripts/components/activatable.lua#L1-L3) [宿主恢复](../../../raw/current/scripts/prefabs/fence.lua#L437-L450) |
| `CanActivateFn(target,doer)` | 存在时直接采用其 success,msg，不自动与 inactive 做 AND；没有时返回 inactive。保留旧测试或明确补回所需条件。[方法](../../../raw/current/scripts/components/activatable.lua#L48-L55) |
| `OnActivate(target,doer)`／`DoActivate(doer)` | 缺回调直接 nil；有回调先置 inactive=false，转交 success,msg。回调返回 false／nil 不恢复默认状态，由宿主决定何时再允许。[执行](../../../raw/current/scripts/components/activatable.lua#L58-L69) |
| 动作返回 | 原动作只在 CanActivate 明确 false 时拒绝；DoActivate 后用 `success ~= false`，所以 nil 可被动作视作成功，但不会使组件发 onactivated。新增回调应显式返回布尔，兼容旧宿主时保留原值。[动作兼容](../../../raw/current/scripts/actions.lua#L3095-L3107) |
| 动作属性 | standingaction／quickaction 分别联动 standingactivation／quickactivation；客户端 ACTIVATE 选状态时先处理 engineering，再站立、短动作、长动作。不要假设改标志就完成全部 SG／预测适配。[属性](../../../raw/current/scripts/components/activatable.lua#L5-L19) [客户端消费者](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L394-L406) |
| `activatable_CanActivate` | 是实体上的候选侧函数，与服务端组件 CanActivateFn 不同；候选还处理燃烧、左／右键、库存实体与 portableengineer 例外。[候选](../../../raw/current/scripts/componentactions.lua#L142-L155) |

组件没有存读钩子；宿主须恢复可用性／实际开关状态。移除只去五种自身标签，不撤销回调已经做的开门、网络值或外部任务。[移除](../../../raw/current/scripts/components/activatable.lua#L40-L46)

## 典型接入

### 原木栅栏门拒绝经 CanActivate 发起的非玩家激活

目标模组 `modmain.lua`，无新资源；只为原注册 fence_gate 增加执行者类型门禁，保留旧测试、inactive 默认判断及 ToggleDoor 的开关／恢复。它限制遵守检查协议的调用，不是用户归属或跨 RPC 授权系统。

```lua
AddPrefabPostInit("fence_gate", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local activatable = inst.components.activatable
    if activatable == nil then return end
    local previous = activatable.CanActivateFn
    activatable.CanActivateFn = function(target, doer)
        if doer == nil or not doer:HasTag("player") then
            return false
        end
        if previous ~= nil then
            return previous(target, doer)
        end
        return target.components.activatable.inactive
    end
end)
```

生效于后续 ACTIVATE 等先测试再执行的路径；其他脚本直接 DoActivate 或调用宿主开关函数仍可绕过，不改变门的当前开闭、物理／锁或原动作时长。CanActivateFn 不保存，构造时重装；停用重载恢复原测试，不回滚已发生的门状态。[注册](../../../raw/current/scripts/prefabs/fence.lua#L942-L945) [宿主回调](../../../raw/current/scripts/prefabs/fence.lua#L437-L450)

## 常见陷阱与限制

- **inactive=false 就一定阻止所有激活**：自定义 CanActivateFn 可覆盖它，直接 DoActivate 也不检查；定义完整调用边界，不能只删标签。[两入口](../../../raw/current/scripts/components/activatable.lua#L48-L69)
- **动作成功必定收到 onactivated**：nil 的旧回调被动作接受，但事件要求真值；返回语义明确后再设计后续监听。[兼容层](../../../raw/current/scripts/actions.lua#L3099-L3104) [事件](../../../raw/current/scripts/components/activatable.lua#L63-L68)
- **失败自动重新允许点击**：inactive 已先清，宿主没恢复就可能不再有候选；按真实成功／失败策略恢复，而非所有回调都无条件再开。[顺序](../../../raw/current/scripts/components/activatable.lua#L58-L68)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 漂移文件 actions.lua／componentactions.lua／stategraphs/SGwilson_client.lua 对照 build-26148036 更新报告（R-01、R-04，SGwilson_client 无专条目）逐条核验，锚点重定位 4 处、结论修订 0 处（ACTIVATE 动作、activatable_CanActivate 候选与客户端选择逻辑未变；本次 nomagiccast 拦截、CORRUPTNIGHTMARE、USEITEM.spellcaster 等新增均不触及本页）；未覆盖：运行验证。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖完整组件、候选／执行／nil 兼容、客户端动作选择、原栅栏门工厂与 ToggleDoor。未全面审校所有机关宿主、服务端 SG、客户端预测、锁系统或直接调用权限。
- 结构检查：本批格式、来源、链接／行号、大小及登记通过；完整非玩家门禁例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；玩家开关、非玩家测试、nil／失败回调、门状态存读及其他模组绕过／覆盖待实测。
