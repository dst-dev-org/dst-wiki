---
title: "markable_proxy"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "牛侧把标记转交评选柱，复用同一份 marks；代理绑定、候选开关与目标资格并非自动同步。"
aliases: ["牛侧标记执行复查代理状态", "点击牛给评选柱投票", "标记代理", "markable_proxy"]
tags: [component, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/markable_proxy.lua", "components/markable.lua", "components/yotb_stager.lua", "prefabs/beefalo.lua", "actions.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# markable_proxy

## 快速判断

- 职责与适用：点击代理实体时把 Mark／HasMarked 转给 proxy.components.markable，记录仍在目标。原 beefalo 服务端已有组件，赛程将 proxy 指向被拴评选柱。[转交](../../../raw/current/scripts/components/markable_proxy.lua#L9-L34) [原挂载](../../../raw/current/scripts/prefabs/beefalo.lua#L1280-L1284) [绑定](../../../raw/current/scripts/components/yotb_stager.lua#L788-L797)
- 优先使用：绑定有效目标后 SetMarkable(bool)，解除前先 SetMarkable(false) 再清 proxy；标记内容与规则在 [[reference/components/markable]]，不要在牛上复制 marks。[原绑定](../../../raw/current/scripts/components/yotb_stager.lua#L792-L797) [解除](../../../raw/current/scripts/components/yotb_stager.lua#L824-L831)
- 重要副作用：代理 SetMarkable 会**同时修改目标**可标记性，再复制目标值到自身；它不是仅隐藏当前牛的局部开关。直接修改目标不会反向自动刷新代理标签。[设置](../../../raw/current/scripts/components/markable_proxy.lua#L25-L34)
- 修改边界：服务端；Mark 不查自身／目标 canbemarked，也不检查目标 IsValid；缺目标返回 false，HasMarked 缺目标无显式返回。原赛程负责绑定、关闭和清引用，不是通用实体代理框架。[方法](../../../raw/current/scripts/components/markable_proxy.lua#L18-L34) [原关闭](../../../raw/current/scripts/components/yotb_stager.lua#L240-L247)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `proxy` | 默认 nil，保存目标实体；自身无 setter、有效性监听或自动持久化。原 stage 绑定到 post，不是把牛设成自己的目标。[默认](../../../raw/current/scripts/components/markable_proxy.lua#L9-L15) [原绑定](../../../raw/current/scripts/components/yotb_stager.lua#L792-L797) |
| `Mark(doer)` | 有 proxy 且其有 markable 就完整透传 Mark 的结果；否则 false。仍会走目标重复标记取消／资格／编号逻辑，代理不另存票。[实现](../../../raw/current/scripts/components/markable_proxy.lua#L18-L22) |
| `SetMarkable(bool)` | 目标有效组件存在才改目标，再把其 canbemarked 复制本地；若先清 proxy，再调 false，**旧代理标签不会被该方法清除**。[实现](../../../raw/current/scripts/components/markable_proxy.lua#L25-L28) |
| `canbemarked` | 属性只维护本实体 markable_proxy 标签；直接赋值不会同步目标。多个代理共享目标时，各自旧标签也不会自动跟随。[属性](../../../raw/current/scripts/components/markable_proxy.lua#L1-L15) |
| `HasMarked(doer)` | 转发到目标；未命中或无目标可为 nil。动作 UNMARK 文案目前只查直接 markable，未调用代理此方法。[转发](../../../raw/current/scripts/components/markable_proxy.lua#L32-L34) [文案](../../../raw/current/scripts/actions.lua#L2767-L2770) |

### 原赛程的绑定与清理顺序

BuildSuspense 将牛的 proxy 设为 post，再 SetMarkable(true) 开放双方；计时结束／比赛收尾先 SetMarkable(false)，再 proxy=nil，并关闭柱子。统一清票是目标 Unmarkall，解除代理本身不取消该 doer 或其他人的记录。[绑定](../../../raw/current/scripts/components/yotb_stager.lua#L788-L797) [时间窗](../../../raw/current/scripts/components/yotb_stager.lua#L824-L831) [清票](../../../raw/current/scripts/components/yotb_stager.lua#L1151-L1161)

组件没有 OnSave／OnLoad 或移除清理方法；不要认为它随牛存档就能恢复目标，也不要在目标移除后继续信任旧引用。自定义多代理场景要明确共享目标的开关归属，避免一端关闭所有目标候选。[完整组件](../../../raw/current/scripts/components/markable_proxy.lua#L1-L39)

## 典型接入

### 牛侧执行标记时复查当前代理和目标仍已启用

目标模组 `modmain.lua`，沿用原牛／柱子资源与赛程绑定；不预先给任何牛绑新柱子，只为原 Mark 加执行门禁。用于拒绝阶段关闭后仍到达该代理方法的旧动作，直接柱子入口未在本例修改。[牛挂载](../../../raw/current/scripts/prefabs/beefalo.lua#L1280-L1284) [原转交](../../../raw/current/scripts/components/markable_proxy.lua#L18-L22)

```lua
AddPrefabPostInit("beefalo", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local proxy = inst.components.markable_proxy
    if proxy == nil then return end
    local old = proxy.Mark
    proxy.Mark = function(self, doer)
        local target = self.proxy
        if not self.canbemarked or target == nil or not target:IsValid() then
            return false
        end
        local markable = target.components.markable
        if markable == nil or not markable.canbemarked then return false end
        return old(self, doer)
    end
end)
```

开启期间保留目标原参与者检查、已有标记取消和失败原因；关闭期间拒绝新增及取消，不改目标的 marks 或可用标签。MARK 动作若同时有直接 markable 会先尝试它，本例只保护牛侧代理分支，不声称封锁所有标记入口或提供权限系统。[动作分派](../../../raw/current/scripts/actions.lua#L2773-L2787)

不新增持久状态，停用并重载恢复原方法；赛程仍负责先关闭再清引用。已有票不会因拒绝旧动作自动清掉，需保留原统一收尾。示例不保证客户端立即隐藏按钮，候选仍按本地 markable_proxy 标签筛选。[候选](../../../raw/current/scripts/componentactions.lua#L584-L587)

## 常见陷阱与限制

- **SetMarkable(false) 只关当前代理**：它也关目标，多个入口共享目标时会连带影响；局部标签与最终执行资格要分开设计。[设置](../../../raw/current/scripts/components/markable_proxy.lua#L25-L28)
- **proxy=nil 再 SetMarkable(false) 就清干净**：此时方法不更新任何字段；原 stage 顺序是先 false 再 nil，不应倒置。[实现](../../../raw/current/scripts/components/markable_proxy.lua#L25-L28) [原顺序](../../../raw/current/scripts/components/yotb_stager.lua#L824-L831)
- **代理能自动恢复／清票**：无存读或目标移除监听，关闭不改 marks；不要和 entitytracker 的 GUID 恢复混用概念。[完整代理](../../../raw/current/scripts/components/markable_proxy.lua#L1-L39) [目标清理](../../../raw/current/scripts/components/markable.lua#L70-L79)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 变更文件（actions.lua、componentactions.lua、prefabs/beefalo.lua，更新报告 R-01/R-04）逐条核验，MARK 动作分派与候选采集无语义变化（beefalo.lua 本次无内容差异），锚点重定位 2 处、无结论修订；未覆盖：运行验证。

- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整代理、目标 Mark／开关／清理、牛挂载、赛程绑定／关闭／统一清票、动作分派和候选。未完整审校所有赛程中断、牛死亡／卸载、客户端文案或自定义多代理竞争。
- 结构检查：本批格式、来源、链接／锚点、大小与登记；完整执行门禁例经已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；评选开始／结束、旧动作、目标删除、取消／转投、断线重连与其他模组方法包装待实测。
