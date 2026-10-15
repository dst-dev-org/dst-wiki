---
title: "moonaltarlinktarget"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "月坛可连线端点：搜索半径内的另两座月坛组成三角，交给 moonstormmanager 校验后生成月坛连线实体。"
aliases: ["月坛连线端点", "月坛链接目标", "moonaltarlinktarget"]
tags: [component]
dst_build: "steam-buildid:26230496"
sources: ["components/moonaltarlinktarget.lua", "components/moonaltarlink.lua", "prefabs/moon_altar.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# moonaltarlinktarget

## 快速判断

- 职责与适用：装在三阶月坛（moon_altar／moon_altar_cosmic／moon_altar_astral）上，参与三坛三角连线；原版唯一宿主是 moon_altar（[安装](../../../raw/current/scripts/prefabs/moon_altar.lua#L619-L623)）。
- 优先使用：让祭坛在完成/就位时机调 `TryEstablishLink()`（原版在祭坛安装与建造完成时调用，[调用](../../../raw/current/scripts/prefabs/moon_altar.lua#L359)）。
- 重要副作用：`link` 字段由 [[reference/components/moonaltarlink]] 回填；祭坛移除时经 onremove 监听反查 `BreakLink`（[断链](../../../raw/current/scripts/components/moonaltarlinktarget.lua#L3-L7)）。
- 修改边界：仅服务端；`canbelinkedfn` 由宿主按阶段设置（[示例](../../../raw/current/scripts/prefabs/moon_altar.lua#L488)），未设置视为恒可连。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `TryEstablishLink()` | 无参数 | 半径 `link_radius` 搜带 `moonaltarlinktarget` tag 的实体，匹配另两种月坛 prefab、彼此距离 ≥ `MOON_ALTAR_LINK_ALTAR_MIN_RADIUS_SQ`（2.5²，tuning.lua:6246）；集满三座经 `moonstormmanager:TestAltarTriangleValid` 后生成 `moon_altar_link` 并 EstablishLink。[实现](../../../raw/current/scripts/components/moonaltarlinktarget.lua#L32-L65) |
| `CanBeLinked()` | → `boolean` | 有 `canbelinkedfn` 时返回其结果，否则恒 true。[实现](../../../raw/current/scripts/components/moonaltarlinktarget.lua#L67-L69) |
| `link` | `entity 或 nil` | 连线实体引用，由 moonaltarlink 建立时写入、断开时清空（[回填](../../../raw/current/scripts/components/moonaltarlink.lua#L15-L18)）。 |
| `link_radius` | `number`，默认 20；月坛设为 `TUNING.MOON_ALTAR_ESTABLISH_LINK_RADIUS`（20，tuning.lua:4067） | 搜索另两座坛的范围。[字段](../../../raw/current/scripts/components/moonaltarlinktarget.lua#L12) [宿主覆盖](../../../raw/current/scripts/prefabs/moon_altar.lua#L623) |
| `onlinkfn`／`onlinkbrokenfn`／`onfoundotheraltarfn` | `fn(inst, other) 或 fn(inst, link)` | 三个可选回调，均由宿主赋值；找到他坛时逐个触发 onfoundotheraltarfn。[声明](../../../raw/current/scripts/components/moonaltarlinktarget.lua#L14-L19) [宿主赋值](../../../raw/current/scripts/prefabs/moon_altar.lua#L620-L622) |

## 典型接入

### 复用三坛连线机制到自定义祭坛

- 前提与选择：组件按 prefab 名匹配 `moon_altar`／`moon_altar_cosmic`／`moon_altar_astral` 三种（[白名单](../../../raw/current/scripts/components/moonaltarlinktarget.lua#L37-L38)），自定义祭坛想入组需替换查找逻辑或借道 prefab 名，否则只能复用"两两搜索"骨架。
- 生效与撤销：三坛集齐即生成连线并双向回填引用；任一祭坛移除自动断链并清空全部 link。
- 原版依据：moon_altar 安装与回调赋值（[宿主](../../../raw/current/scripts/prefabs/moon_altar.lua#L619-L623)）。

```lua
-- 自定义祭坛 master_postinit 内（复用组件但保持原版三坛流程）
inst:AddComponent("moonaltarlinktarget")
inst.components.moonaltarlinktarget.onlinkfn = function(inst, link)
    inst.AnimState:PlayAnimation("proximity_loop", true)
end
inst.components.moonaltarlinktarget.canbelinkedfn = function(inst)
    return inst._stage == 3
end
```

- 调整与验证：搜索以 tag `moonaltarlinktarget` 为准；三角形有效性最终由 TheWorld 的 moonstormmanager 判定，本组件不做几何校验。

## 常见陷阱与限制

- `TryEstablishLink` 靠 `looking_for_altars[self.inst.prefab] = nil` 排除自己 prefab 类型：同 prefab 的两座坛永远无法互连（[实现](../../../raw/current/scripts/components/moonaltarlinktarget.lua#L38)），复制月坛类型做自定义组网会失效。
- 找到第三座坛但三角校验失败时直接 return，本次搜索不作记录；下次调用会重新从零搜索。

## 相关链接

- [[reference/components/moonaltarlink]]：连线实体本体与质心定位。
- [[reference/components/worldmeteorshower]]：moon_altar.lua 同时挂接的流星掉率修正。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖全部接口与 moon_altar／moonaltarlink 消费链、tuning 取值；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
