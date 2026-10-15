---
title: "herdmember"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "保存成员侧群体引用、搜索标签与延迟建群开关，区分退出、重新建群及群体侧存档登记。"
aliases: ["草蜥蜴不自动入群", "生物退出群体", "自动创建牛群", "herdmember"]
tags: [component, creature, ai, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/herdmember.lua", "components/herd.lua", "prefabs/grassgekko.lua", "prefabs/grassgekkoherd.lua", "prefabs/beefalo.lua", "brains/beefalobrain.lua", "entityscript.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# herdmember

## 快速判断

- 职责与适用：成员侧保存 herd 引用，enabled 联动 herdmember 搜索标签，延迟尝试创建 herdprefab；群体的集合、中心和持久关联由 [[reference/components/herd]] 管理。[构造／标签](../../../raw/current/scripts/components/herdmember.lua#L1-L29)
- 优先使用：Enable(false) 退出并关闭自动建群／成员搜索标签；Leave 在 enabled 时会安排 5 秒后再尝试建群。SetHerd 只赋引用，不是完整加入。[引用](../../../raw/current/scripts/components/herdmember.lua#L39-L49) [退出／开关](../../../raw/current/scripts/components/herdmember.lua#L63-L80)
- 重要副作用：CreateHerd 新建的是群体节点，再让节点 GatherNearbyMembers，不直接指定“创建者已加入”；标签、旧 herd 坐标或节点配置不合，创建后仍可能没登记。[创建](../../../raw/current/scripts/components/herdmember.lua#L51-L61) [收集资格](../../../raw/current/scripts/components/herd.lua#L151-L170)
- 修改边界：本页例为主模拟草蜥蜴；本组件无 OnSave／OnLoad，群体读档可直接 AddMember，不能只禁用搜索就假定不会恢复登记。人物随从另查 [[reference/components/follower]]，不通过 herdmember 设置主人。[读档](../../../raw/current/scripts/components/herd.lua#L243-L266)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `enabled`／`herd`／`herdprefab` | 默认 true／nil／beefaloherd | enabled 的属性回调增删 herdmember 标签；构造排 8.1 秒 OnInit 尝试建群。原生物可在 pristine 阶段提前加标签，不等于已有有效群。[组件](../../../raw/current/scripts/components/herdmember.lua#L1-L29) [草蜥蜴标签](../../../raw/current/scripts/prefabs/grassgekko.lua#L102-L113) |
| `GetHerd()`／`SetHerd(herd)` | 引用或 nil／无显式返回 | Get 不验证 IsValid；Set 不维护群体 members／计数／监听／坐标，也不保证只属于一个群。正常加入由 herd:AddMember 完成这些协作。[引用](../../../raw/current/scripts/components/herdmember.lua#L39-L49) [群体登记](../../../raw/current/scripts/components/herd.lua#L101-L125) |
| `SetHerdPrefab(prefab)` | 群体节点名 | 只赋值；不验证注册、标签匹配，不立即生成，也不把原群替换掉。草蜥蜴设 grassgekkoherd，不能沿用默认牛群名。[方法](../../../raw/current/scripts/components/herdmember.lua#L43-L45) [实际配置](../../../raw/current/scripts/prefabs/grassgekko.lua#L139-L142) |
| `CreateHerd()` | 无显式实体返回 | enabled、无有效 herd 且未死时 SpawnPrefab，在成员位置建节点并调用其 GatherNearbyMembers（若有组件）；不直接设置自身 herd，也不周期检测群丢失。[实现](../../../raw/current/scripts/components/herdmember.lua#L51-L61) |
| `Leave()` | 无显式返回 | 有效旧群先 RemoveMember；enabled 仍真时排 5 秒 OnInit，所以不保证长期独立。没有有效旧群时也不显式清理失效引用或残留 knownlocations.herd。[实现](../../../raw/current/scripts/components/herdmember.lua#L63-L71) |
| `Enable(enabled)` | boolean | 禁用时从有效群移除，最后写 enabled；启用且无有效群时排 5 秒任务。它不是无副作用的幂等 setter，重复 Enable(true)／Leave 可创建多项任务，不自动取消旧任务。[实现](../../../raw/current/scripts/components/herdmember.lua#L73-L80) [Leave 排程](../../../raw/current/scripts/components/herdmember.lua#L63-L70) |
| `OnRemoveFromEntity()` | 组件卸载钩子 | 取消当前 self.task 并去标签，但不主动 Leave／RemoveMember；群体后续清理与 updaterange 等配置有关，不把卸组件当立即双向退群。[实现](../../../raw/current/scripts/components/herdmember.lua#L31-L37) [群体清理](../../../raw/current/scripts/components/herd.lua#L195-L215) |

### 初始化、行为与保存协作

herd 默认采样间隔 6～8 秒，herdmember 构造等待 8.1 秒，注释意图是先让已有群收集；实际宿主还可能调用 Enable(true) 再排 5 秒任务，例如牛的初始化。不能把注释当作全宿主仅有一项 8.1 秒任务，也不要每帧反复启用。[群体采样](../../../raw/current/scripts/components/herd.lua#L15-L36) [成员任务](../../../raw/current/scripts/components/herdmember.lua#L9-L29) [牛安装](../../../raw/current/scripts/prefabs/beefalo.lua#L1243-L1250)

群体 OnSave 保存成员 GUID，LoadPostPass 直接 AddMember；成员侧 enabled／herdprefab 不由本组件独立存档。原牛在驯化／回野生阶段通过 Enable(false/true) 切换，不能把自动入群禁用当完整驯化 API。群体坐标经 knownlocations 提供给 Brain，不保证生物立刻回到中心。[重连](../../../raw/current/scripts/components/herd.lua#L243-L266) [驯化入口](../../../raw/current/scripts/prefabs/beefalo.lua#L530-L554) [Brain 消费](../../../raw/current/scripts/brains/beefalobrain.lua#L241-L255) 坐标机制见 [[reference/components/knownlocations]]。

## 典型接入

### 草蜥蜴不自动入群，并在原存档群体重连后退出

放目标模组 `modmain.lua`，无需资源。原 grassgekko 已有 herdmember、knownlocations 和 grassgekkoherd 配置；在零延迟任务中禁用，覆盖同步初始化／群体 LoadPostPass 对原成员的重新登记。若只在 PrefabPostInit 同步改 enabled，AddMember 本身不检查该值，旧群仍可登记回来。[原生物](../../../raw/current/scripts/prefabs/grassgekko.lua#L112-L142) [重连](../../../raw/current/scripts/components/herd.lua#L257-L266) [添加](../../../raw/current/scripts/components/herd.lua#L101-L125) [Hook](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("grassgekko", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:DoTaskInTime(0, function(gekko)
        if not gekko:IsValid() then return end
        local member = gekko.components.herdmember
        if member ~= nil then
            member:Enable(false)
        end
    end)
end)
```

Enable(false) 经旧群 RemoveMember 清集合／计数／herd 坐标与引用，然后移除搜索标签；若退出后空群，原 onempty 会删群节点。构造任务没有因此全部取消，稍后 CreateHerd 会因 enabled=false 不建群。本例保留组件，避免只卸组件而遗留登记。[开关与创建](../../../raw/current/scripts/components/herdmember.lua#L51-L80) [退群](../../../raw/current/scripts/components/herd.lua#L127-L149) [空群删除](../../../raw/current/scripts/prefabs/grassgekkoherd.lua#L27-L31)

本例阻止 grassgekkoherd 按标签自动收集和成员自动建群；未配置搜索标签的群体或更晚直接 AddMember 仍可绕过，需在对应调用方处理，不增加永久轮询。不会把它变成人物随从、禁止移动或删除现有草蜥蜴；未改掉尾、采集或其他 SG。草蜥蜴群的周期繁殖段在注释块内，不宣称此例关掉了当前活跃的“定时生蜥蜴”。[收集入口](../../../raw/current/scripts/components/herd.lua#L151-L170) [群体注释分界](../../../raw/current/scripts/prefabs/grassgekkoherd.lua#L27-L44)

每次创建／重载重新应用；本组件不保存 disabled 状态。停用并重载后恢复默认资格，能否被附近群收集或重新建群仍受标签／已有坐标／节点配置影响，不承诺回到原节点；不把 Leave 当作该例替代。

## 常见陷阱与限制

- **Leave 一次就长期离群**：启用状态下会 5 秒后再尝试；需要退出标准自动群体流程时用 Enable(false)，并核验保存重连及其他显式登记者。[实现](../../../raw/current/scripts/components/herdmember.lua#L63-L80)
- **GetHerd 非 nil／CreateHerd 调完就保证入群**：引用可能失效，新节点还依赖 Gather 的标签／坐标／存活资格。先验证真实节点及其集合，不直接 SetHerd 伪造双向关系。[引用／创建](../../../raw/current/scripts/components/herdmember.lua#L39-L61) [收集](../../../raw/current/scripts/components/herd.lua#L151-L170)
- **默认关闭后读档会自动记住／移除组件就完成退群**：本组件没有保存开关，群体另存关联；卸载钩子不去成员集合。按宿主重连后清理，避免依赖未来采样来修复关系。[卸载](../../../raw/current/scripts/components/herdmember.lua#L31-L37) [群体保存](../../../raw/current/scripts/components/herd.lua#L243-L266)
- **反复 Enable(true) 只是确保打开**：会额外排任务并覆盖 self.task 引用；不要在循环中调用，也不承诺卸载时能靠单个引用取消此前所有重复任务。[排程](../../../raw/current/scripts/components/herdmember.lua#L63-L80)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 3 处（grassgekko 标签段／herdmember 配置段整体 +1，本版仅新增 plantcreature 标签行）；无结论修订，仅锚点迁移——依据：变更清单命中 prefabs/grassgekko.lua（单行 AddTag 插入）＋全局枢纽 diff 审阅（entityscript 改动不涉本页引用路径），herdmember.lua、herd.lua、beefalo.lua 及相关 brain 未变。
- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖组件完整方法／标签／延迟任务、群体登记／重连／清理消费者、草蜥蜴配置与注释、牛启用及驯化阶段分工和 Brain 坐标使用。未全面审校驯化／骑乘、跨群合并、全部宿主或任意异步读档／更晚显式登记。
- 结构检查：格式、来源、链接／锚点、大小与唯一登记通过；完整禁用例通过已有 Full Moon Lua 5.1 syntax-only，不证明读档退出／实际活动范围。
- 游戏运行：未运行；原群存档／重连退出、空群删除、初始化多任务、存活／死亡、换群、停用后恢复及其他模组显式登记待实测。
