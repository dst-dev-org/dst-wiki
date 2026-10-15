---
title: "writeable"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "牌子等实体的单写入者会话、文本提交／保存与编辑界面协作；区分直接赋字、提交和自动关闭距离。"
aliases: ["木牌编辑保持五单位距离", "牌子题字", "文本提交与编辑会话", "writeable"]
tags: [component, ui, callback, lifecycle, network-sync, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/writeable.lua", "components/writeable_replica.lua", "prefabs/homesign.lua", "writeables.lua", "actions.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# writeable

## 快速判断

- 职责与适用：保存文本／作者标识，维护一个 writer 与编辑界面；原 homesign 先有 inspectable 再加 writeable。不是通用任意 UI 或全客户端共享编辑器。[构造](../../../raw/current/scripts/components/writeable.lua#L57-L88) [宿主](../../../raw/current/scripts/prefabs/homesign.lua#L75-L86)
- 优先使用：真实编辑走 BeginWriting／Write／EndWriting；直接设定既知文本用 SetText，但它不替代提交校验／回调／会话收尾。[会话](../../../raw/current/scripts/components/writeable.lua#L162-L240) [直接设置](../../../raw/current/scripts/components/writeable.lua#L128-L130)
- 重要副作用：Write 只接收当前非 nil writer，检查 UTF-8 长度，设文本、调 onwritten、结束会话，最后可按 remove_after_write 删除实体；EndWriting 即使取消也会记录玩家身份并调用结束回调。[提交](../../../raw/current/scripts/components/writeable.lua#L184-L205) [结束](../../../raw/current/scripts/components/writeable.lua#L207-L240)
- 修改边界：SetWriteableDistance 控制已打开会话的保持距离；骑乘、离远或不可见会关闭，不等于改变 WRITE 动作接近距离。默认 WRITE 拒绝已有文本／占用，但黑暗中未开窗也返回 true。[自动关闭](../../../raw/current/scripts/components/writeable.lua#L246-L255) [动作](../../../raw/current/scripts/actions.lua#L4083-L4099)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `GetText()`／`SetText(text)`／`IsWritten()` | nil 表示未写，空串也算已写；SetText 不做长度／平台过滤或提交回调。GetText 在 XB1 可含作者标识控制字符，不一概当裸文本。[读写](../../../raw/current/scripts/components/writeable.lua#L119-L130) [判定](../../../raw/current/scripts/components/writeable.lua#L176-L182) |
| `BeginWriting(doer)` | 仅在无 writer 时启动更新、登记执行者并监听其关闭弹窗／移除事件；有 HUD 时创建编辑框。方法不自行检查可见性或距离，正常入口还有动作门禁。[方法](../../../raw/current/scripts/components/writeable.lua#L162-L174) |
| `Write(doer,text)` | text 为 nil 或长度不超布局 maxcharacters／全局默认上限；非 nil 文本依赖本 prefab 已注册布局。Rail 路径应用词语过滤；客户端 replica 也先检查长度再发送 RPC。[服务端](../../../raw/current/scripts/components/writeable.lua#L184-L205) [客户端](../../../raw/current/scripts/components/writeable_replica.lua#L93-L102) [布局查询](../../../raw/current/scripts/writeables.lua#L92-L94) |
| 两个回调 setter | onwritten(inst,text,doer) 在 EndWriting 前；onwritingended(inst) 属会话结束，可因取消／离远触发，不能当“成功保存”通知。[设置](../../../raw/current/scripts/components/writeable.lua#L111-L117) [调用](../../../raw/current/scripts/components/writeable.lua#L193-L240) |
| `SetDefaultWriteable(bool)` | 控制 writeable 候选标签／WRITING 符号；不构成直接 BeginWriting／Write 的权限开关，已有文字默认隐藏写入候选。[属性](../../../raw/current/scripts/components/writeable.lua#L27-L41) [setter](../../../raw/current/scripts/components/writeable.lua#L136-L156) [候选](../../../raw/current/scripts/componentactions.lua#L965-L969) |
| `SetAutomaticDescriptionEnabled(bool)` | 为现有 inspectable 设置／清空 getspecialdescription；显示带文本过滤上下文／作者信息。改前确认没有其他模组的描述回调需要保留。[描述](../../../raw/current/scripts/components/writeable.lua#L3-L19) [属性](../../../raw/current/scripts/components/writeable.lua#L47-L55) |

### 保存与清理

OnSave 仅保存 text／netid／userid；OnLoad 还原并按 Rail 分支过滤文本，不恢复正在编辑的 writer／screen。移组件调用 EndWriting、去标签／onbuilt 监听，仅当描述仍是自身函数时移除；实体删除也映射 EndWriting。[存读](../../../raw/current/scripts/components/writeable.lua#L91-L109) [清理](../../../raw/current/scripts/components/writeable.lua#L259-L271)

writer 属性调用 replica:SetWriter，将 classified 数据访问目标给当前写入者；这不等于把全部文本作为公开 netvar 广播。replica 的 OnRemoveFromEntity 明确不支持，不能从服务端组件清理推断热卸载整套网络端安全。[属性](../../../raw/current/scripts/components/writeable.lua#L43-L45) [目标](../../../raw/current/scripts/components/writeable_replica.lua#L122-L128) [限制](../../../raw/current/scripts/components/writeable_replica.lua#L24-L43)

## 典型接入

### 原木牌编辑会话在五单位距离内继续保持

目标模组 `modmain.lua`，不新增资源或界面；原 homesign 已有 writeable 和对应布局，保留原写入／平台过滤／保存／拆除。组件默认保持距离为 3，本例仅将该参数设为 5。[默认](../../../raw/current/scripts/components/writeable.lua#L74-L78) [原布局](../../../raw/current/scripts/writeables.lua#L7-L20)

```lua
AddPrefabPostInit("homesign", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local writeable = inst.components.writeable
    if writeable ~= nil then
        writeable:SetWriteableDistance(5)
    end
end)
```

5 是与 inst:IsNear 同一世界距离单位的阈值，不是格数、时间或输入字符数。仅影响已有会话的更新判定；仍可能因骑乘、不可见、关闭弹窗或写入者移除而结束。参数不由组件存档，原工厂与 Hook 重载重建；停用并重载恢复默认。不直接扩大动作距离、不延迟提交，也不更改正在保存的文本或作者。[setter](../../../raw/current/scripts/components/writeable.lua#L158-L160) [关闭判定](../../../raw/current/scripts/components/writeable.lua#L246-L255) [监听](../../../raw/current/scripts/components/writeable.lua#L162-L174)

## 常见陷阱与限制

- **用 SetText 接收客户端输入就等同 Write**：绕过写入者／长度／平台过滤／提交回调与 EndWriting；正常输入沿现有提交链，直接设置只用于自己已核验的数据。[对比](../../../raw/current/scripts/components/writeable.lua#L128-L130) [提交](../../../raw/current/scripts/components/writeable.lua#L184-L205)
- **新 prefab 只加组件就有现成木牌界面**：非 nil 提交会取 GetLayout(prefab).maxcharacters，布局未注册时不能依赖默认上限兜底；还需原宿主式 inspectable、网络与资源。[布局](../../../raw/current/scripts/writeables.lua#L92-L94) [使用](../../../raw/current/scripts/components/writeable.lua#L184-L188)
- **清文本用空串即可恢复写字候选**：IsWritten 检查非 nil，空串仍是已写，属性仍隐藏默认候选；实际未写状态是 nil。[标签](../../../raw/current/scripts/components/writeable.lua#L27-L41) [判定](../../../raw/current/scripts/components/writeable.lua#L176-L178)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件（actions.lua=R-01、componentactions.lua=R-04）对照更新报告逐条核验，锚点重定位 1 处、结论修订 0 处——WRITE.fn 与 writeable 候选块原文未变（动作区整体 +3 行移），本次无 invalid_hold_action／nomagiccast 触及 WRITE；未覆盖：运行验证。

- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整组件／replica、WRITE 与候选、homesign 宿主及布局定义／查询，核验会话、提交、作者、存读与清理边界。未完整审校 UI 创建器、classified prefab、RPC 接收端、平台实际词语过滤或热卸载同步。
- 结构检查：本批格式、来源、源码引用／行号、大小与登记通过；完整距离配置例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；主机／远端、竞争写入、字符长度、空串／nil、距离／骑乘关闭、取消／重载与平台差异待实测。
