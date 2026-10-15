---
title: "moonaltarlink"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "月坛连线本体：把连线实体放到祭坛组质心并维护与各 moonaltarlinktarget 的双向引用及连线建立/断开回调。"
aliases: ["月坛连线", "moonaltarlink", "月祭坛连线"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/moonaltarlink.lua", "components/moonaltarlinktarget.lua", "prefabs/moon_altar_link.lua", "prefabs/moon_altar.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# moonaltarlink

## 快速判断

- 职责与适用：挂在 `moon_altar_link` 连线实体上；`EstablishLink` 把实体移到祭坛组质心，给每个祭坛的 `moonaltarlinktarget` 回填 link 引用并触发双方回调（[EstablishLink](../../../raw/current/scripts/components/moonaltarlink.lua#L9-L28)）。
- 优先使用：不要手工摆位——连线实体由 `moonaltarlinktarget:TryEstablishLink` 生成并调用 EstablishLink（[调用](../../../raw/current/scripts/components/moonaltarlinktarget.lua#L57)）；宿主只需挂 `onlinkfn` 接收通知（[宿主](../../../raw/current/scripts/prefabs/moon_altar_link.lua#L176-L177)）。
- 重要副作用：质心定位 `SetPosition(平均x, 0, 平均z)`（y 强制 0）；建立/断开都会回调每个祭坛与连线自身的回调函数（[断开](../../../raw/current/scripts/components/moonaltarlink.lua#L30-L47)）。
- 修改边界：祭坛实体被移除时其 `moonaltarlinktarget` 的 onremove 会经 link 反查调 `BreakLink`（[反查](../../../raw/current/scripts/components/moonaltarlinktarget.lua#L5)）；断开后 `altars` 清空，可安全重复调用。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `EstablishLink(altars)` | `altars: entity 列表 -> 无` | 累加各祭坛坐标取质心定位；写每个 `moonaltarlinktarget.link`、调其 `onlinkfn`；存 `self.altars` 后调自身 `onlinkfn(inst, altars)`。[实现](../../../raw/current/scripts/components/moonaltarlink.lua#L9-L28) |
| `BreakLink()` | `-> 无` | 清每个祭坛的 link 并调其 `onlinkbrokenfn`（对组件缺失做了保护），再调自身 `onlinkbrokenfn`，最后清 `altars`。[实现](../../../raw/current/scripts/components/moonaltarlink.lua#L30-L47) |
| `altars` | `entity 列表\|nil` | 当前连线中的祭坛组；EstablishLink 直接访问 `v.components.moonaltarlinktarget.link` 无 nil 保护。[字段](../../../raw/current/scripts/components/moonaltarlink.lua#L15) |
| `onlinkfn` / `onlinkbrokenfn` | `fn(inst, altars)` | 宿主回调；原版宿主只设 `onlinkfn`。[挂接](../../../raw/current/scripts/prefabs/moon_altar_link.lua#L176-L177) |

## 典型接入

### 月坛三角连线的建立与恢复（原版参照）

- 前提与选择：三个祭坛各带 `moonaltarlinktarget` 并完成放置检查后，TryEstablishLink 生成 `moon_altar_link` 并以三者调用 EstablishLink（[调用](../../../raw/current/scripts/components/moonaltarlinktarget.lua#L57)）；连线实体用 entitytracker 记住三坛，读档在 OnLoadPostPass 里校验 `TestAltarTriangleValid` 后重建（[重建](../../../raw/current/scripts/prefabs/moon_altar_link.lua#L114-L121)）。
- 生效与撤销：连线建立后月坛进入风暴序列；任一祭坛移除即整条断开（[反查](../../../raw/current/scripts/components/moonaltarlinktarget.lua#L5)）。

```lua
-- 原版等价的调用形态（服务端）
local link = SpawnPrefab("moon_altar_link")
link.components.moonaltarlink.onlinkfn = OnLinkEstablished
link.components.moonaltarlink:EstablishLink({ altar1, altar2, altar3 })
```

- 调整与验证：altars 内每个实体都必须带 `moonaltarlinktarget` 组件，EstablishLink 无保护，缺失直接报错；质心 y 恒为 0，祭坛高度差被忽略。

## 常见陷阱与限制

- **建立无组件保护、断开有**：`EstablishLink` 直接访问 `moonaltarlinktarget.link`（[实现](../../../raw/current/scripts/components/moonaltarlink.lua#L15)），`BreakLink` 才检查组件存在（[检查](../../../raw/current/scripts/components/moonaltarlink.lua#L33)）；传入不合规列表会在建立时崩。
- **回调顺序**：先逐个祭坛 onlinkfn，再连线自身 onlinkfn；断开时先祭坛 onlinkbrokenfn 再连线自身（[断开](../../../raw/current/scripts/components/moonaltarlink.lua#L32-L45)），依赖顺序的逻辑要按此排布。
- **专用实体**：组件与 moonaltarlinktarget 强耦合，mod 想复用"质心连线"需自带等价 target 组件；无现成解耦入口。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、moonaltarlinktarget 的建立/反查路径、moon_altar_link 宿主挂接与 OnLoadPostPass 重建、moon_altar 的 TryEstablishLink 触发点。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。

## 相关链接

- [[reference/components/moonaltarlinktarget]]：三座月坛端点组件，集齐后生成连线。
