---
title: "colourcube"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界后处理调色组件：按相位/季节/月相混合环境、理智、启发三条 colour cube 通道，接 playervision 覆盖并驱动鱼眼与月狂强度。"
aliases: ["调色滤镜", "后处理调色", "colourcube 组件"]
tags: [component, ui]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/colourcube.lua", "prefabs/forest.lua", "prefabs/cave.lua", "components/playervision.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# colourcube

## 快速判断

- 职责与适用：客户端视觉组件（forest.lua:630、cave.lua:345、lavaarena.lua:143、quagmire.lua:168，均在非专用服务器分支）；三通道：0=环境、1=理智、2=启发（:440-447 注释），经 `PostProcessor` 生效。
- 优先使用：给玩家视界换整套色调 → 由 playervision 的 `ccoverrides`/`ccphasefn` 玩家事件驱动（playervision.lua:191-192，组件监听 :343-344）；整屏强制单 cube → `overridecolourcube` 世界事件（:412-427，quagmire.lua:169、lavaarena.lua:144 用例）。
- 重要副作用：夜/黄昏环境音般的渐变混合由 `Blend` 管理（:150-206）；雨穹顶内持续加鱼眼畸变（:485-491）。
- 修改边界：洞穴世界固定用 `CAVE_COLOURCUBES.night` 且不监听 `seasontick`（:66-69、:95-96、:457-459）；`Profile:GetDistortionModifier()` 缩放畸变强度（:118）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetDistortionModifier` | `(modifier) -> nil` | 更新畸变系数并标记后处理刷新；玩家设置里的畸变项落到这里。:521-524 |
| `OnUpdate` | `(dt) -> nil` | 推进混合 lerp、鱼眼时间/强度、月狂强度（:469-515）；`LongUpdate` 直接完成混合（:517-519）。 |
| `Blend`（局部） | `(time) -> nil` | 目标变化才重开混合；进行中换目标则跳到当前值重混；渐隐/无激活玩家直接跳满（:129-131、:202-205）。 |
| 相位/季节映射表（局部） | `INSANITY_COLOURCUBES`、`LUNACY_COLOURCUBES`、`SEASON_COLOURCUBES`（4 季×day/dusk/night/full_moon）、`CAVE_COLOURCUBES` | 键为相位串或 "full_moon"/"regular"/"moon_storm"；缺项回退 IDENTITY_COLOURCUBE（:151-153）。:19-69 |
| 混合时长（局部） | `PHASE_BLEND_TIMES` day4/dusk6/night8/full_moon8、`SEASON_BLEND_TIME`=10、`DEFAULT_BLEND_TIME`=.25 | :71-80。 |

## 关键流程：相位判定与覆盖链

1. 相位取值：`GetCCPhase` 优先 `_overridephase.fn()`，其次洞穴固定 "night"，否则夜晚遇满月换 "full_moon"（:133-137）；lunacy 通道在满月、`moon_storm`（moonstorm 风暴事件）间切换（:144-148）。
2. 玩家覆盖：`ccoverrides` 事件数据直接替换 `_overridecctable`（:281-284）；`ccphasefn` 传 `{fn, blendtime, events}` 表，组件按 `events` 列表挂/摘刷新监听，过渡取较短 blendtime（:286-308）。
3. 强制覆盖：`overridecolourcube` 设 cube 后三通道全部锁成该 cube，取消时恢复（:412-427）；`overridecolourmodifier` 改整体色彩系数（:429-434，worldtemperature.lua:159 推）。
4. 理智/启发强度：`sanitydelta` 按 replica.sanity 模式与百分比驱动 `_lunacyspeed` 与畸变参数，`dappereffects` 标签平方化（:252-279）；强度渐变在 OnUpdate 积分（:493-514）。

## 典型接入

### 区域换色：给玩家视界套自定义色调（playervision 路径）

- 前提与选择：colourcube 只认玩家事件，入口在 playervision（`GetCCTable`/`GetCCPhaseFn`）；想覆盖单 cube 用世界事件更直接。
- 生效与撤销：`ccoverrides` 传的表键与 `SEASON_COLOURCUBES` 子表一致（day/dusk/night/full_moon）；撤销推 nil 或原表。
- 原版依据：components/playervision.lua:191-192、quagmire.lua:169。

```lua
-- 世界事件方式：整屏锁定为某个 cube（主机与客户端都会收到世界事件）
TheWorld:PushEvent("overridecolourcube", "images/colour_cubes/purple_moon_cc.tex")
-- 解除：
TheWorld:PushEvent("overridecolourcube", nil)
```

- 调整与验证：覆盖期间 `Blend` 跳过混合直接锁定目标（:155-160）；调试看 `GetDebugString` 三通道当前源/目标（:530-539）。

## 常见陷阱与限制

- 仅激活玩家生效：所有玩家事件监听绑定 `_activatedplayer`（:330-353），切人/断线自动摘除；世界事件（overridecolourcube/overridecolourmodifier）则作用于世界。
- cube 路径必须是引擎注册的 tex：写错路径无校验，`Blend` 对缺表项回退 identity（:151-153），但手写错误字符串会静默出黑/无效果。
- 洞穴世界季节表被整体绕过（:457-459），改 `SEASON_COLOURCUBES` 不影响洞穴。
- 事件名成对：月相用 `moonphasechanged2`（:454），旧 `moonphasechanged` 不被本组件消费；风暴用玩家的 `stormlevel` 事件判 `STORM_TYPES.MOONSTORM`（:310-320）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 545 行；挂载 forest.lua:630、cave.lua:345，playervision.lua:191-192、quagmire.lua:168-169、lavaarena.lua:143-144、worldtemperature.lua:159 逐行核对。
- 游戏运行：未运行。

## 相关链接

- [[reference/components/playervision]]
- [[reference/components/moonstorms]]
