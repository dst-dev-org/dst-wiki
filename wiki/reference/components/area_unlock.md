---
title: "area_unlock"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "区域剧情解锁原型组件：监听 changearea 事件把走过的 story 区域记为已解锁，原版无任何挂载点，属于未接入的死代码。"
aliases: ["区域解锁", "area_unlock", "剧情区域追踪"]
tags: [component, data]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/area_unlock.lua", "components/areaaware.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# area_unlock

## 快速判断

- 职责与适用：构造时监听宿主的 `changearea` 事件，把与 `RegisterStory` 登记名匹配的 `area.story` 置为已解锁；`GetUnlocked` 返回已解锁名单。语义是"玩家到过哪些剧情区域"。[CheckUnlock](../../../raw/current/scripts/components/area_unlock.lua#L8-L13)
- 优先使用：没有原版使用场景。全库 grep 确认无 `AddComponent("area_unlock")`、无 `RegisterStory/GetUnlocked` 调用者；把它当"区域到访解锁"的参考实现读。
- 重要副作用：事件依赖 `changearea`，原版由 [areaaware](../../../raw/current/scripts/components/areaaware.lua#L59) 在换区时推送 `current_area_data`；宿主不挂 areaaware 则事件永不触发。
- 修改边界：无存读（解锁进度不持久化）、无标签、无网络变量；stories 表只在会话内存活。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `RegisterStory(story)` | `story: string -> nil` | 登记待解锁区域名，初值 false；重复登记无守卫。[实现](../../../raw/current/scripts/components/area_unlock.lua#L15-L17) |
| `CheckUnlock(area)`（内部） | `area: {story: string}` | 仅当 `stories[area.story] == false` 时翻转为 true；未登记或已解锁的区不做任何事。[实现](../../../raw/current/scripts/components/area_unlock.lua#L8-L13) |
| `GetUnlocked()` | `-> string[]` | 遍历 stories 收集值为 true 的键，返回无序列表。[实现](../../../raw/current/scripts/components/area_unlock.lua#L19-L27) |
| `changearea`（监听） | `data: areaaware.current_area_data` | 构造时匿名监听；依赖宿主有 areaaware 组件在换区时推送。[推送方](../../../raw/current/scripts/components/areaaware.lua#L59) |

## 常见陷阱与限制

- **期待解锁进度跨存档：** 组件无 OnSave/OnLoad，重载后 stories 全部回到未解锁；要用作真实进度需自行补持久化。[无存读](../../../raw/current/scripts/components/area_unlock.lua#L1-L29)
- **area.story 字段并非普遍存在：** CheckUnlock 直接读 `area.story`；areaaware 的 current_area_data 是否含 `story` 取决于世界生成区域数据，字段缺失时比较 `nil == false` 不成立，静默不解锁。[消费方式](../../../raw/current/scripts/components/area_unlock.lua#L9)
- **匿名监听不可单独移除：** 构造用匿名函数监听，组件未实现 `OnRemoveFromEntity`；`RemoveComponent` 后回调仍挂在实体上（直到实体销毁），复用时要意识到这一悬挂。[构造](../../../raw/current/scripts/components/area_unlock.lua#L4)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（29 行）与 areaaware.lua 推送点；全库 grep（`area_unlock`、`AreaUnlock`、`RegisterStory`、`GetUnlocked`）确认无原版消费者。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 通过。
- 游戏运行：未运行；组件从未被原版挂载，无可运行场景。

相关：[[reference/components/charliearenawatcher]]
