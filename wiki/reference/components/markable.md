---
title: "markable"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "牛年评选目标的按执行者标记／取消、编号池与业务回调；不是通用地图标记或持久投票系统。"
aliases: ["牛年评选柱最多四个标记", "牛年投票标记", "重复标记取消", "markable"]
tags: [component, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/markable.lua", "components/markable_proxy.lua", "components/yotb_stager.lua", "prefabs/yotb_post.lua", "actions.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# markable

## 快速判断

- 职责与适用：目标保存按 doer 实体区分的 marks，重复 Mark 会取消原标记；原 yotb_post 用回调生成／移除彩带，stage 管评选阶段与跨柱唯一选择。[算法](../../../raw/current/scripts/components/markable.lua#L33-L66) [彩带宿主](../../../raw/current/scripts/prefabs/yotb_post.lua#L81-L141)
- 优先使用：新标记资格包 canmarkfn，保留原参与者条件；统一结束清理用 Unmarkall。点击牛转到柱子的入口见 [[reference/components/markable-proxy]]。[资格](../../../raw/current/scripts/components/markable.lua#L50-L63) [清理](../../../raw/current/scripts/components/markable.lua#L70-L79)
- 重要副作用：已有 doer 先执行 unmarkfn／归还 ID／删除记录，不走 canmarkfn；新标记先取 ID 并回调，之后才入 marks，回调内尚看不到该条记录。[顺序](../../../raw/current/scripts/components/markable.lua#L33-L63)
- 修改边界：服务端；SetMarkable 只改候选标签，Mark 不查 canbemarked。保存返回空表、加载为空，不是持久化 userid 投票表。[标签](../../../raw/current/scripts/components/markable.lua#L1-L17) [开关／存读](../../../raw/current/scripts/components/markable.lua#L78-L105)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `Mark(doer)` | 已有标记则取消并 true；新标记需 canmarkfn 返回真，否则 false,reason。markfn／unmarkfn 返回值不影响最终结果，不内检玩家、距离、阶段或编号耗尽。[算法](../../../raw/current/scripts/components/markable.lua#L33-L66) |
| `canmarkfn`／`markfn`／`unmarkfn` | 前者 fn(target,doer)→allowed,reason；另两者 fn(target,doer,id)。默认不设 canmarkfn 就不能新增。原参与者检测由柱子提供，原 stage 的跨柱唯一选择也在宿主回调。[组件](../../../raw/current/scripts/components/markable.lua#L33-L66) [跨柱](../../../raw/current/scripts/prefabs/yotb_post.lua#L81-L97) [参与者](../../../raw/current/scripts/prefabs/yotb_post.lua#L213-L238) |
| `markpool_reset`／`markpool`／`getid()` | 默认 1～8 随机不重复取用；池空返回 nil，Mark 却仍可调 markfn／尝试添加记录，不自动拒绝第九位。容量需由业务资格保护。[默认／取号](../../../raw/current/scripts/components/markable.lua#L9-L30) [使用](../../../raw/current/scripts/components/markable.lua#L55-L63) |
| `HasMarked(doer)` | 比较当前 doer 实体，命中 true，未命中无显式返回；不是稳定 userid，也不是 bool false 保证。[查询](../../../raw/current/scripts/components/markable.lua#L90-L95) |
| `Unmarkall()`／`unmarkallfn` | 先 fn(target) 清业务显示，再重建编号池和 marks；不逐项调用 unmarkfn，不修改 canbemarked。原比赛收尾显式使用。[方法](../../../raw/current/scripts/components/markable.lua#L70-L79) [收尾](../../../raw/current/scripts/components/yotb_stager.lua#L1151-L1161) |
| `SetMarkable(bool)`／`canbemarked` | 维护 markable 标签，不拦直接 Mark；原柱子初始 false，由赛程阶段启停。[方法](../../../raw/current/scripts/components/markable.lua#L78-L79) [安装](../../../raw/current/scripts/prefabs/yotb_post.lua#L208-L213) |

原 MARK 动作先试目标 markable，未成功才试 markable_proxy；文案的 UNMARK 分支仅查直接 markable，不应据显示文字反推代理标记状态。[动作](../../../raw/current/scripts/actions.lua#L2767-L2787) [候选](../../../raw/current/scripts/componentactions.lua#L578-L587)

## 典型接入

### 原牛年评选柱最多接受四名执行者的当前标记

目标模组 `modmain.lua`，沿用原柱子、彩带和 stage；仅增加每柱上限，保留参与者门禁。取消已有标记发生在新资格之前，所以达到上限后旧参与者仍能取消。[原资格](../../../raw/current/scripts/prefabs/yotb_post.lua#L208-L238) [算法顺序](../../../raw/current/scripts/components/markable.lua#L33-L63)

```lua
AddPrefabPostInit("yotb_post", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local markable = inst.components.markable
    if markable == nil or markable.canmarkfn == nil then return end
    local old = markable.canmarkfn
    markable.canmarkfn = function(post, doer)
        local allowed, reason = old(post, doer)
        if not allowed then return allowed, reason end
        if #markable.marks >= 4 then return false end
        return allowed, reason
    end
end)
```

上限计当前 marks 条目，不改编号池、地图位置或总参赛人数；它也会影响原 NPC 若其走相同 Mark。达到上限返回 false 且未新增本地化原因。旧选择跨柱取消、彩带显示和统一收尾仍在原宿主，不另增一套显示对象。[跨柱／彩带](../../../raw/current/scripts/prefabs/yotb_post.lua#L81-L121) [统一清理](../../../raw/current/scripts/prefabs/yotb_post.lua#L135-L141)

只在实例重建时安装，不主动删已有超四条记录；它们仍可取消，降到四条以下才放行新标记。停用并重载不再安装上限；组件本来不保存 marks，本例也不保存投票。本例不启用比赛、不替代 stage 的时间窗或新增服务端权限验证。

## 常见陷阱与限制

- **SetMarkable(false) 后任何 Mark 都拒绝**：只有候选标签变化；直接调用或已排队动作仍需按真实入口核验阶段，代理的局部补强见其页面。[组件](../../../raw/current/scripts/components/markable.lua#L33-L79)
- **canmarkfn 同时限制取消／自动限制八人**：取消在前且没有资格测试，编号池耗尽也未设错误返回；两种需求不能只靠池大小表达。[算法](../../../raw/current/scripts/components/markable.lua#L20-L66)
- **markfn 返回 false 就不记票，或 OnSave 会保存 marks**：返回值不被读取，存读为空；显示清理／持久化必须根据宿主实际协议设计。[记录](../../../raw/current/scripts/components/markable.lua#L55-L63) [存读](../../../raw/current/scripts/components/markable.lua#L98-L105)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 变更文件（actions.lua、componentactions.lua，更新报告 R-01/R-04）逐条核验，MARK 动作与 markable／markable_proxy 候选采集无语义变化，锚点重定位 1 处、无结论修订；未覆盖：运行验证。

- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整 markable／代理、MARK 候选与执行、柱子参与者／彩带、stage 跨柱查询／投票启停／结束清理。未全面审校完整比赛、玩家退出后的旧 doer、评分／NPC 调度或所有显示销毁。
- 结构检查：本批格式、来源、链接／锚点、大小和登记；完整上限例经已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；四票上限、旧票取消／转投、NPC、阶段关闭与已排队动作、读档／断线待实测。
