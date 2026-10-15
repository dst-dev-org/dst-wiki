---
title: "worldwind"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "世界级风向服务：周期性随机转向并在 TheWorld 推 windchange 事件，供灯笼、风滚草等读取风向角。"
aliases: ["worldwind", "世界风向", "windchange事件", "风向角"]
tags: [component]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/worldwind.lua", "prefabs/forest.lua", "prefabs/floatinglantern.lua", "prefabs/tumbleweed.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# worldwind

## 快速判断

- 职责与适用：构造即开更的定时器，每 0.5～1 个 SEG_TIME 随机换一次风向角并在世界实体推 `windchange`。([worldwind.lua:39-54](../../../raw/current/scripts/components/worldwind.lua#L39-L54) ([worldwind.lua:3-4](../../../raw/current/scripts/components/worldwind.lua#L3-L4)
- 优先使用：被动监听 `TheWorld` 的 `windchange`，或主动 `GetWindAngle()` 取当前角。([floatinglantern.lua:175-176](../../../raw/current/scripts/prefabs/floatinglantern.lua#L175-L176) ([tumbleweed.lua:464-465](../../../raw/current/scripts/prefabs/tumbleweed.lua#L464-L465)
- 重要副作用：换风事件负载 `{angle, velocity}` 在世界实体上推送，监听宿主须传 TheWorld。[worldwind.lua:50](../../../raw/current/scripts/components/worldwind.lua#L50
- 修改边界：仅森林世界安装（forest.lua master_postinit），洞穴等世界无此组件。[forest.lua:664](../../../raw/current/scripts/prefabs/forest.lua#L664

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `GetWindAngle()` | 数值角度 0～360 | 返回当前随机风向角，不做插值。([worldwind.lua:27-29](../../../raw/current/scripts/components/worldwind.lua#L27-L29) |
| `GetWindVelocity()` | 数值，恒为 1 | velocity 字段从未被修改，仅原样返回。[worldwind.lua:10](../../../raw/current/scripts/components/worldwind.lua#L10 ([worldwind.lua:31-33](../../../raw/current/scripts/components/worldwind.lua#L31-L33) |
| `Start()` / `Stop()` | 无参 | 手动开关更新循环；构造已自动 Start。[worldwind.lua:16](../../../raw/current/scripts/components/worldwind.lua#L16 ([worldwind.lua:19-25](../../../raw/current/scripts/components/worldwind.lua#L19-L25) |
| `OnUpdate(dt)` | 倒计时触发换风 | 到点随机新 angle、推 `windchange`、按 `[0.5*SEG_TIME, SEG_TIME]` 重排下次换风。([worldwind.lua:46-53](../../../raw/current/scripts/components/worldwind.lua#L46-L53) |

## 典型接入

### 让实体随风改变朝向

- 前提与选择：只在有 worldwind 的世界（森林）可用，读取前判空（原版风滚草即判空回退）。[tumbleweed.lua:464](../../../raw/current/scripts/prefabs/tumbleweed.lua#L464

```lua
local windangle = TheWorld.components.worldwind
    and TheWorld.components.worldwind:GetWindAngle()
    or math.random(360)
inst.Transform:SetRotation(windangle)
```

- 生效与撤销：角度随换风跳变；需要平滑过渡时自行对 angle 插值。

## 常见陷阱与限制

- 无"风速"概念：velocity 恒 1 且事件里原样携带，不能当强度用。[worldwind.lua:10](../../../raw/current/scripts/components/worldwind.lua#L10
- 换风间隔随机 0.5～1 个 SEG_TIME（tuning 的 seg_time），非固定周期；游戏开局第一次换风在 1 秒后（初始 timeToWindChange=1）。([worldwind.lua:3-4](../../../raw/current/scripts/components/worldwind.lua#L3-L4) [worldwind.lua:14](../../../raw/current/scripts/components/worldwind.lua#L14
- 组件装在 TheWorld 上且构造即开更，装到普通实体也能跑但事件仍推在该实体上，与原版消费者语义不同。

## 验证记录

- 源码审阅：2026-10-28，`steam-buildid:26230496`：审阅组件全文、森林安装点、floatinglantern 与 tumbleweed 两个消费路径及 SEG_TIME 引用。
- 游戏运行：未运行；未核对实际换风体感频率。
- 结构检查：validate_page.py 通过；validate_source_links.py --strict 通过（本批 16 页合并执行）。
