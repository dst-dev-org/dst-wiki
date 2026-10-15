---
title: "klaussacklock"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "克劳斯式目标的用钥匙回调与单件消费；成功、失败原因和是否消费是三个独立返回值。"
aliases: ["赃物袋服务端玩家检查", "错误钥匙也消耗", "消耗型钥匙目标", "klaussacklock"]
tags: [component, callback, item, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/klaussacklock.lua", "components/klaussackkey.lua", "prefabs/klaus_sack.lua", "prefabs/fence.lua", "componentactions.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# klaussacklock

## 快速判断

- 职责与适用：目标侧把钥匙交给 onusekeyfn，再按第三个返回值消费一件；自身没有 islocked、钥匙匹配表或保存锁状态。原赃物袋和暴食公园门复用此壳，各自决定业务。[完整组件](../../../raw/current/scripts/components/klaussacklock.lua#L1-L38) [袋子](../../../raw/current/scripts/prefabs/klaus_sack.lua#L52-L76) [门](../../../raw/current/scripts/prefabs/fence.lua#L465-L482)
- 优先使用：SetOnUseKey(fn)，返回 success,fail_msg,consumed；配 [[reference/components/klaussackkey]]。要把实体留在锁中并允许归还，先比较 [[reference/components/lock]]。[接口](../../../raw/current/scripts/components/klaussacklock.lua#L14-L35)
- 重要副作用：先回调，后消费；consumed 与 success 独立。原普通鹿角对袋子可召唤 boss、产骨片，最终 false,WRONGKEY,true；“失败”不等于无效果或退款。[执行](../../../raw/current/scripts/components/klaussacklock.lua#L23-L35) [错误分支](../../../raw/current/scripts/prefabs/klaus_sack.lua#L78-L125)
- 修改边界：服务端回调必须自行校验类型、玩家和目标状态；组件只检查钥匙非 nil／有效和回调存在。标签仅产生候选，不替代最终验证。[门禁](../../../raw/current/scripts/components/klaussacklock.lua#L18-L23) [候选](../../../raw/current/scripts/componentactions.lua#L1470-L1476)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `SetOnUseKey(fn)`／`onusekeyfn` | fn(target,key,doer)；业务效果应在此完成，无默认匹配逻辑，替换前保留原回调和三返回值。[方法](../../../raw/current/scripts/components/klaussacklock.lua#L14-L23) |
| `UseKey(key,doer)` | 无效 key 或无回调返回 false；真值 success 返回 true，否则 false,fail_msg，第三值不会传回动作调用者。[实现](../../../raw/current/scripts/components/klaussacklock.lua#L18-L35) |
| `consumed` 回调返回值 | 真值时 stackable:Get():Remove()，无堆叠组件则 Remove；组件消费时机在业务回调之后，回调不要再删除同一把钥匙。[消费](../../../raw/current/scripts/components/klaussacklock.lua#L23-L29) |
| `klaussacklock` 标签／移除钩子 | 构造添加标签，推荐 pristine 前也加；移除组件清标签。没有自动持久化锁状态或解锁动画。[构造与移除](../../../raw/current/scripts/components/klaussacklock.lua#L1-L15) |

### 两种原版宿主的结果协议

| 宿主 | 决策与效果 |
|---|---|
| `klaus_sack` | 无钥匙组件直接拒绝；真钥匙且已有 boss 时 false,KLAUS,false；无 boss 时开袋／掉落、移除锁组件，true,nil,true。普通钥匙走骨片／召唤逻辑，最终 false,WRONGKEY,true。[判断](../../../raw/current/scripts/prefabs/klaus_sack.lua#L52-L84) [召唤与结果](../../../raw/current/scripts/prefabs/klaus_sack.lua#L115-L125) |
| `quagmire_park_gate` | 钥匙 keytype 匹配目标 ID 才设置双方门的 _isunlocked 并 ToggleDoor，true,nil,true；不匹配返回 false,QUAGMIRE_WRONGKEY,false。门自身处理开关／保存，不用 lock.islocked。[回调](../../../raw/current/scripts/prefabs/fence.lua#L465-L482) [工厂](../../../raw/current/scripts/prefabs/fence.lua#L732-L745) |

## 典型接入

### 原赃物袋在执行时要求有效非幽灵玩家

目标模组 `modmain.lua`，沿用原袋子资源／召唤／掉落流程，增加服务端调用资格检查；只包装原目标，不改变全部 klaussacklock。正常钥匙／boss 判断仍交给原回调，不在新包装里复制。[原安装](../../../raw/current/scripts/prefabs/klaus_sack.lua#L197-L212) [动作](../../../raw/current/scripts/actions.lua#L3239-L3246)

```lua
AddPrefabPostInit("klaus_sack", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local lock = inst.components.klaussacklock
    if lock == nil or lock.onusekeyfn == nil then return end
    local old = lock.onusekeyfn
    lock:SetOnUseKey(function(target, key, doer)
        if doer == nil or not doer:IsValid()
            or not doer:HasTag("player") or doer:HasTag("playerghost") then
            return false, nil, false
        end
        return old(target, key, doer)
    end)
end)
```

拒绝路径不调用原业务、不消费钥匙；通过后完整透传三个返回值，保留普通鹿角的失败消费与真钥匙开袋。这里返回 nil 失败原因，未新增语言资源或承诺特定提示。原回调使用 doer 发射骨片，新增检查也避免服务端代码以 nil 玩家进入这一分支。[调用与消费](../../../raw/current/scripts/components/klaussacklock.lua#L23-L35) [骨片](../../../raw/current/scripts/prefabs/klaus_sack.lua#L78-L82)

包装随目标重建安装；停用并重载不再加此检查，不恢复已被打开／移除的袋子或已消费鹿角。直接调用已保存的旧回调会绕过包装；本例也不新增距离／服务器权限或防刷系统。

## 常见陷阱与限制

- **只返回 true 就会扣一件**：第三值 nil 不消费；反之 false,reason,true 仍消费。必须明确写三值语义。[顺序](../../../raw/current/scripts/components/klaussacklock.lua#L23-L35)
- **回调自行 Remove 钥匙后再返回 consumed=true**：组件仍会访问／取用它；消费只保留一个责任方，业务应先使用必要数据再交组件处理。[消费点](../../../raw/current/scripts/components/klaussacklock.lua#L23-L29)
- **UseKey 会检 truekey／keytype 或自动保存开锁状态**：这些全在宿主，给任意箱子挂组件不等于可解锁容器。[组件](../../../raw/current/scripts/components/klaussacklock.lua#L1-L38) [门宿主](../../../raw/current/scripts/prefabs/fence.lua#L465-L482)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：actions.lua（R-01）、componentactions.lua（R-04）对照 build-26148036 更新报告逐条核验，锚点重定位 1 处、结论修订 0 处：USEKLAUSSACKKEY 执行（actions.lua:3229-3236）与 klaussackkey 候选（componentactions.lua:1470-1476）逐行无变化；hunk 地图 R-07（containers.lua 容器 widget 重构）不在本页 sources，无命中；未覆盖：运行验证。

- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整组件、动作回传、候选、袋子钥匙分支／召唤结果及暴食门匹配／开门、组件与宿主分工。未全面审校 boss、掉落内容、门网络表现或所有保存分支。
- 结构检查：本批格式、来源、链接／锚点、大小和登记检查；完整资格包装经已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；真假钥匙／重复使用、幽灵或 nil 调用、消耗、配对门、原动画及模组冲突待实测。
