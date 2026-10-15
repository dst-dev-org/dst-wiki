---
title: "herd"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "管理群体成员、聚集／合并与群体中心，向成员更新坐标并保存关联；区分繁殖、跟随和硬数量上限。"
aliases: ["牛群目标上限改为二十", "群体成员管理", "牛群中心", "herd"]
tags: [component, creature, ai, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/herd.lua", "components/herdmember.lua", "prefabs/beefaloherd.lua", "prefabs/lightninggoatherd.lua", "prefabs/grassgekkoherd.lua", "brains/beefalobrain.lua", "entityscript.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# herd

## 快速判断

- 职责与适用：群体节点侧管理成员集合、收集／合并和中心坐标；成员侧引用／自动建群见 [[reference/components/herdmember]]。它不直接生成生物，也不等于随从主人。[构造](../../../raw/current/scripts/components/herd.lua#L15-L37) [更新](../../../raw/current/scripts/components/herd.lua#L195-L241)
- 优先使用：完整登记／退出用 AddMember／RemoveMember；SetMaxSize 改 IsFull 的目标阈值，GatherNearbyMembers 按范围／标签收集。改牛群阈值见下文，不能把它当任何路径都不能超员的硬上限。
- 重要副作用：登记会写成员 knownlocations.herd、设置 herdmember 引用并调用宿主回调；最后一个成员退出可触发宿主删除整个群体节点。繁殖、发情与实际走动由消费者实现。[增删](../../../raw/current/scripts/components/herd.lua#L101-L149) [宿主](../../../raw/current/scripts/prefabs/beefaloherd.lua#L113-L139)
- 修改边界：本例仅改主模拟中的非联网牛群节点；保留成员、发情和周期生成配置。提高阈值不保证立刻出现更多牛，密度、亲本、季节与离屏条件仍有效。[工厂](../../../raw/current/scripts/prefabs/beefaloherd.lua#L102-L139) [繁殖资格](../../../raw/current/scripts/prefabs/beefaloherd.lua#L57-L81)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `members`／`membercount`／`maxsize` | 实体→true／0／默认 12 | 默认每 6～8 秒更新一次；SetMaxSize 只赋阈值，不裁减现有集合或触发满群回调。IsFull 用 count>=max。[初值／任务](../../../raw/current/scripts/components/herd.lua#L15-L37) [阈值](../../../raw/current/scripts/components/herd.lua#L76-L98) |
| `SetMemberTag(tag)`／`SetGatherRange(range)` | 标签／范围 | tag 非 nil 时搜索标签为 herdmember 和 tag；只设置 membertag 字段会漏掉 membersearchtags 的联动。范围 nil 不自动收集。[实现](../../../raw/current/scripts/components/herd.lua#L59-L74) [消费](../../../raw/current/scripts/components/herd.lua#L151-L170) |
| `AddMember(member)` | 无显式返回 | 本集合内去重，随后计数、监听 death/onremove、记坐标／引用、调 addmember，count 恰等 max 才调 onfull。没有通用满员、距离、enabled、死亡或其他群归属校验，调用方负责资格。[实现](../../../raw/current/scripts/components/herd.lua#L101-L125) |
| `RemoveMember(member)` | 无显式返回 | 仅处理已登记者；先 removemember 回调，再去监听、清坐标／引用、减数和删记录；集合为空才调 onempty。回调不可假定记录已经删除。[实现](../../../raw/current/scripts/components/herd.lua#L127-L149) |
| `GatherNearbyMembers()` | 无显式返回 | 范围存在且未满才搜；成员须有 herdmember，若有 knownlocations 则 herd 坐标须为空，并须未死。达到目标即停；不是通用跨群迁移入口。[实现](../../../raw/current/scripts/components/herd.lua#L151-L170) |
| `MergeNearbyHerds()`／`nomerging` | 无参数／可选 true | 自身未满且允许合并，搜索同 membertag、成员数<4 且总数能容纳的邻群；先 AddMember，再移除旧节点。跨群清理顺序见下文，不把它当保留全部双向关系的安全搬群协议。[实现](../../../raw/current/scripts/components/herd.lua#L172-L193) |
| `SetUpdateRange(range)`／`updatepos`／`updateposincombat` | 范围／默认 true／默认 false | 有 updaterange 才做成员组件／物种标签清理与中心更新；超出此范围只不参与平均，不因此退群。平均还要求 updatepos 且 combat 存在无 target，或 updateposincombat=true；无 combat 不默认参与。[实现](../../../raw/current/scripts/components/herd.lua#L195-L230) |
| `updateposfn(host, average)` | 返回位置；average 可 nil | 有 updaterange 时调用，接收可用成员的平均坐标或 nil，返回值直接读 x/y/z；必须总能返回有效位置，不把 nil 当“保留原位”。[实现](../../../raw/current/scripts/components/herd.lua#L218-L230) |
| `SetAddMemberFn/SetRemoveMemberFn/SetOnEmptyFn/SetOnFullFn(fn)` | 增删 fn(host, member)；满空 fn(host) | setter 替换回调；原牛群添加回调向成员同步 entermood/leavemood，空群回调删除节点，扩展须保留所需宿主行为。[设置](../../../raw/current/scripts/components/herd.lua#L80-L94) [牛群回调](../../../raw/current/scripts/prefabs/beefaloherd.lua#L51-L55) [安装](../../../raw/current/scripts/prefabs/beefaloherd.lua#L119-L129) |

### 中心、关联与清理

每次更新先 Gather、Merge，再按上述资格计算中心；最后把当前节点位置写给所有有 knownlocations 的成员，供 Brain 消费。即使 updatepos=false，仍可广播固定节点坐标；坐标更新不是强制移动实体，牛 Brain 的 Wander 和其他高优先级行为决定实际活动。[更新顺序](../../../raw/current/scripts/components/herd.lua#L195-L241) [Brain 消费](../../../raw/current/scripts/brains/beefalobrain.lua#L241-L255) 坐标记录机制见 [[reference/components/knownlocations]]。

OnSave 仅保存成员 GUID 并返回引用；LoadPostPass 对已重建实体逐个 AddMember，因此会重新执行登记副作用，不按 maxsize 截断恢复。目标阈值、范围和回调由工厂／模组重新配置，不会因设过一次就自动持久化。[保存／重连](../../../raw/current/scripts/components/herd.lua#L243-L266)

组件卸载的 OnRemoveFromEntity 只取消采样任务并去成员监听；群体实体删除的 OnRemoveEntity 才逐个 RemoveMember、清引用／坐标。引擎将两种钩子用于不同路径，不可只移除 herd 组件就认为双向关系已清理。[组件钩子](../../../raw/current/scripts/components/herd.lua#L39-L50) [卸载调用](../../../raw/current/scripts/entityscript.lua#L671-L679) [实体删除调用](../../../raw/current/scripts/entityscript.lua#L1739-L1750)

自动合并先在新群登记，再删除旧群，旧群的 RemoveMember 也会清成员侧引用／坐标；不要复制这种顺序自行实现任意搬群。合并后的多方引用一致性保留为运行待核验项，本批不提供迁群修复或保证。[合并](../../../raw/current/scripts/components/herd.lua#L173-L193) [旧群清理](../../../raw/current/scripts/components/herd.lua#L46-L50) [成员清理](../../../raw/current/scripts/components/herd.lua#L127-L149)

## 典型接入

### 普通牛群目标上限设为 20，保留繁殖门禁

放目标模组 `modmain.lua`，无需资源，只处理 beefaloherd。原工厂用 herd 默认 maxsize，SetMaxSize 只调 IsFull 的阈值；完整生长／数量结果仍由原宿主控制。[默认值](../../../raw/current/scripts/components/herd.lua#L15-L37) [阈值](../../../raw/current/scripts/components/herd.lua#L76-L98) [宿主配置](../../../raw/current/scripts/prefabs/beefaloherd.lua#L113-L139) [Hook](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("beefaloherd", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local herd = inst.components.herd
    if herd ~= nil then
        herd:SetMaxSize(20)
    end
end)
```

`20` 是每个群体的目标阈值，不是全世界牛数或强制产量。CanSpawn 还要求合适亲本、范围内数量低于 BEEFALOHERD_MAX_IN_RANGE；periodicspawner 另有近处密度／离屏条件，mood 控制启动／停止。只改阈值不立即生成牛，也不重排生成计时。[资格](../../../raw/current/scripts/prefabs/beefaloherd.lua#L57-L81) [发情启停](../../../raw/current/scripts/prefabs/beefaloherd.lua#L28-L48) [生成配置](../../../raw/current/scripts/prefabs/beefaloherd.lua#L133-L139) 计时与密度见 [[reference/components/periodicspawner]]；发情条件、回调与存读见 [[reference/components/mood]]。

新节点和重载重新应用；已有成员按 GUID 恢复，改小阈值也不驱逐超额成员。停用重载恢复工厂阈值，不回滚已生成生物。此例不修改 lightninggoatherd：山羊群另设 TUNING 上限、nomerging=true 和自己的周期配置。草蜥蜴群只挂 herd，其周期生成段位于注释块，不能据搜索命中当作活跃繁殖。[山羊群](../../../raw/current/scripts/prefabs/lightninggoatherd.lua#L27-L41) [草蜥蜴群](../../../raw/current/scripts/prefabs/grassgekkoherd.lua#L27-L44)

## 常见陷阱与限制

- **AddMember 自动检查上限／匹配物种**：它只做本集合去重；Gather 和宿主测试才加资格。直接调用、存档恢复都可能超过目标阈值，不作为硬数量保护。[添加](../../../raw/current/scripts/components/herd.lua#L101-L125) [恢复](../../../raw/current/scripts/components/herd.lua#L257-L266)
- **设置成员 SetHerd 就登记完成／群中心就是队长**：前者只赋引用，完整增删还涉及集合／监听／坐标；中心节点不提供人物随从关系。主人跟随见 [[reference/components/leader]]／[[reference/components/follower]]。[成员赋值](../../../raw/current/scripts/components/herdmember.lua#L39-L49)
- **成员远于 updaterange 就自动离群／平均包含所有成员**：范围约束中心样本，正常远成员仍在集合；缺 combat、战斗中与 updatepos 开关会改变平均资格。[更新](../../../raw/current/scripts/components/herd.lua#L195-L238)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 1 处（entityscript OnRemoveEntity 组件分发段）；无结论修订，仅锚点迁移——依据：全局枢纽 diff 审阅（R-08 湿润查询／距离函数改动不涉组件移除钩子）＋变更清单命中 entityscript.lua，herd.lua、herdmember.lua 及各 herd prefab／brain 未变。
- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖登记／回调／阈值、收集／合并执行顺序、中心与坐标消费者、GUID 重连、组件卸载／实体删除分工，牛群／山羊群及草蜥蜴群代表配置。未全面审校 mood、繁殖／骑乘／驯化、全部 Brain 或自动合并后引用一致性。
- 结构检查：格式、来源、链接／锚点、大小及唯一登记通过；完整阈值例通过已有 Full Moon Lua 5.1 syntax-only，不证明实际牛数／繁殖效果。
- 游戏运行：未运行；聚集／满群／超额、迁移／合并、成员死亡／节点删除、中心战斗筛选、存读、发情／密度和其他模组直接加成员待实测。
