---
title: "simplebook"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "轻量阅读回调与可见性检查；原烹饪指南用它打开弹窗，不要求 reader 或自动消费／精神结算。"
aliases: ["烹饪指南阅读后发送模组事件", "普通角色阅读物品", "simplebook"]
tags: [component, item, ui, callback, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/simplebook.lua", "prefabs/cookbook.lua", "actions.lua", "componentactions.lua", "stategraphs/SGwilson.lua", "stategraphs/SGwilson_client.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# simplebook

## 快速判断

- 职责与适用：Read(doer) 检查 CanEntitySeeTarget 后执行 onreadfn(item,doer)，原 cookbook 由此 ShowPopUp(COOKBOOK,true)。需要技能书消费／精神规则时看 [[reference/components/book]] 和 [[reference/components/reader]]。[组件](../../../raw/current/scripts/components/simplebook.lua#L14-L22) [原效果](../../../raw/current/scripts/prefabs/cookbook.lua#L6-L8)
- 优先使用：直接设置 onreadfn 字段，没有 SetOnReadFn；改现成物品时包装原回调，保留弹窗行为。[字段与调用](../../../raw/current/scripts/components/simplebook.lua#L2-L22) [原挂载](../../../raw/current/scripts/prefabs/cookbook.lua#L40-L45)
- 重要副作用：可见性失败返回 false，其余路径通常 nil；READ 动作丢弃这个返回值并返回 true，因此动作成功不证明回调已执行或 UI 已显示。[方法](../../../raw/current/scripts/components/simplebook.lua#L14-L22) [动作](../../../raw/current/scripts/actions.lua#L1328-L1339)
- 修改边界：默认库存候选不检查 reader；原 SG 将库存 simplebook 分到 cookbook_open。组件不提供任意图书 UI、数据存档、扣次数、精神变化或完整客户端同步。[候选](../../../raw/current/scripts/componentactions.lua#L2895-L2897) [服务端路由](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1034-L1039) [客户端路由](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L363-L368)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `onreadfn` | 可选 fn(item,doer)，返回值被丢弃；回调自行承担业务效果，没有等待完成／关闭弹窗的协议。[调用](../../../raw/current/scripts/components/simplebook.lua#L14-L22) |
| `Read(doer)` | 依赖有效执行者／物品供可见性函数使用；没有额外距离、持有权或冷却检查，不能据库存候选推断任意直接调用安全。[完整方法](../../../raw/current/scripts/components/simplebook.lua#L14-L22) |
| `simplebook` 标签 | 构造添加、移组件删除；原 cookbook 在 pristine 前预放标签，服务端才挂组件，客户端 SG 通过标签分派。[标签](../../../raw/current/scripts/components/simplebook.lua#L2-L12) [工厂](../../../raw/current/scripts/prefabs/cookbook.lua#L28-L45) |

## 典型接入

### 原烹饪指南发起阅读后，通知服务端模组逻辑

目标模组 `modmain.lua`，无新资源；只扩展原 cookbook，不创建新屏幕。原回调发起烹饪指南弹窗，再向读者发送私有事件，供模组已有逻辑监听。

```lua
AddPrefabPostInit("cookbook", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local book = inst.components.simplebook
    if book == nil or book.onreadfn == nil then return end
    local previous = book.onreadfn
    book.onreadfn = function(item, doer)
        previous(item, doer)
        doer:PushEvent("my_mod_cookbook_read", { book = item })
    end
end)
```

事件发在 doer 自身，data.book 是本次物品；仅在可见性检查放行且原回调正常返回后发出。不代表弹窗渲染成功、关闭或阅读完成；本例没有奖励、任务存档或去重，重复发起会重复通知。停用并重载不再包装原回调；不向客户端新增 RPC，也不把服务器事件当全客户端广播。[检查与调用](../../../raw/current/scripts/components/simplebook.lua#L14-L22) [原弹窗入口](../../../raw/current/scripts/prefabs/cookbook.lua#L6-L8)

若要做自定义图鉴／教程 UI，应另核验窗口、数据与客户端展示链，不能只把 prefab 改名就继承 cookbook 的全部上下文。原 prefab 资源与库存挂载可作宿主参照，但这里交付的是现成物品的回调扩展。[工厂](../../../raw/current/scripts/prefabs/cookbook.lua#L10-L45)

## 常见陷阱与限制

- **onreadfn 返回 false 可让 READ 失败**：组件忽略回调返回值，动作又忽略组件返回值；拒绝提示／动作返回需要另设计实际执行链，不从布尔值推断。[动作](../../../raw/current/scripts/actions.lua#L1328-L1339)
- **加 reader 才能读烹饪指南**：simplebook 分支无需 reader，reader 也不会帮它派发 OnRead；先区分技能书与轻量 UI 物品。[动作优先级](../../../raw/current/scripts/actions.lua#L1328-L1339)
- **组件自己生成书页或保存阅读进度**：文件只有标签、可见性与回调；UI 和进度归宿主或独立机制，不能从名字补造接口。[完整定义](../../../raw/current/scripts/components/simplebook.lua#L2-L24)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件（actions.lua=R-01、componentactions.lua=R-04、SGwilson.lua=SB-17、SGwilson_client.lua 无专条目）逐条核验，锚点重定位 6 处、结论修订 0 处——READ.fn、simplebook 库存候选与两端 READ 路由原文未变，SB-17 改动集中在 CORRUPTNIGHTMARE／传送等新增状态，不触及 cookbook_open；未覆盖：运行验证。

- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整 simplebook／cookbook、READ 分支与库存候选、双方 READ 路由。未完整审校 cookbook_open 状态、弹窗网络／内容、窗口关闭、任意新 UI 或阅读进度保存。
- 结构检查：本批格式、来源、源码引用／行号、大小与登记通过；完整保留原回调并通知例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；不同角色、可见性失败、重复通知、主机／远端弹窗与其他模组回调包装待实测。
