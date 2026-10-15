---
title: "oceanwhirlportalphysics"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "海洋漩涡门户的涡流物理：按快慢双节拍扫描并卷动范围内船与漂浮物，中心触发宿主回调做传送或沉没。"
aliases: ["漩涡涡流物理", "漩涡卷船", "oceanwhirlportalphysics"]
tags: [component, state-machine]
dst_build: "steam-buildid:26230496"
sources: ["components/oceanwhirlportalphysics.lua", "prefabs/oceanwhirlbigportal.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# oceanwhirlportalphysics

## 快速判断

- 职责与适用：唯一挂载在 `oceanwhirlbigportal`（大漩涡门户，[装配](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L338-L343)），对半径内海面实体施加向心＋顺时针切向速度，中心处触发宿主回调；不适合通用吸力场（强绑漩涡语义）。
- 优先使用：改场强调 `SetFocalRadius`／`SetRadius`／`SetPullStrength`／`SetRadialStrength`（[L64-L100](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L64-L100)）；中心行为用 `SetOnEntityTouchingFocalFn(fn)`（[L98-L100](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L98-L100)）。
- 重要副作用：卷入的实体会被加 `physicsmodifiedexternally` 组件并由本组件设定速度（[L121-L141](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L121-L141)）；质量 0 的静态物直接受伤或被摧毁（[L143-L165](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L143-L165)）。
- 修改边界：仅主机模拟（[断言](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L29)）；`SetEnabled(false)` 或宿主休眠即停更并释放全部实体（[L47-L62](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L47-L62)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetEnabled(enabled)` | bool | 启用时下一帧即扫描；禁用时停更新并对每个被卷实体 `ForgetEntity`；宿主休眠时延迟生效（[L47-L62](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L47-L62)）。 |
| `SetFocalRadius` / `SetRadius` / `SetPullStrength` / `SetRadialStrength` | 数字 | 中心半径／作用半径／向心力／切向力；改前两者会重算力衰减指数（[L34-L41](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L34-L41)、[L64-L96](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L64-L96)）。 |
| `SetOnEntityTouchingFocalFn(fn)` | fn(inst, ent) | 实体中心触及判定 `dist - 物理半径×0.5 <= focalradius` 时回调（[L241-L243](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L241-L243)）。 |
| `RememberEntity` / `ForgetEntity` | ent | 卷入登记与释放；释放时恢复其物理控制权（physicsmodifiedexternally RemoveSource）（[L121-L141](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L121-L141)）。 |
| `OnUpdate(dt)` | 组件更新 | 快节拍 0.1s（有卷入物或 5s 冷却内）／慢节拍 1s；扫描用累积器，施力恒按 0.1s 步长（[L247-L267](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L247-L267)）。 |
| `OnEntitySleep` / `OnEntityWake` | 引擎回调 | 宿主休眠停更并清空卷入，唤醒恢复更新；由引擎在实体休眠／唤醒时调用（[L102-L115](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L102-L115)）。 |

## 力学（关键流程）

1. 扫描：`FindEntities(radius + MAX_PHYSICS_RADIUS)` 排除 FX／DECOR／INLIMBO／`oceanwhirlportal`／flying／ghost／playerghost／shadow；只收在海上或带 `boat` 标签者，且 `距离-物理半径 <= radius`（[L14](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L14)、[L167-L220](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L167-L220)）。
2. 质量分派：`bird` 标签直接驱离（`flyaway`）；`Physics:GetMass()==0` 的静态物走 `TryToBreakStaticObject`（有 health 按 `OCEANWHIRLBIGPORTAL_BOAT_PERCENT_DAMAGE_PER_TICK=0.1` 扣血，否则 workable 销毁）；`winchtarget` 特例按角度挪到环带或打捞上岸（[L143-L165](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L143-L165)、[L185-L211](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L185-L211)）。
3. 施力：速度向心＋顺时针切向，`forcemodifier = (focalradius/距离)^forceexponent`，指数由"半径处衰减到 0.1 倍"反解（[L222-L245](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L222-L245)、[L16](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L16)、[L34-L36](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L34-L36)）。
4. 中心效果（宿主装配）：触水保护＋尝试 `worldmigrator:Activate` 传送去洞穴；不可迁移者按船扣血或直接沉没，成功传送的实体有 3s 冷却（[L227-L253](../../../raw/current/scripts/prefabs/oceanwhirlbigportal.lua#L227-L253)）。
5. 原版参数：focal 4／radius 14／pull 10／radial 20（[tuning](../../../raw/current/scripts/tuning.lua#L8966-L8970)）。

## 常见陷阱与限制

- **卷入期间实体速度被接管**：被 `RememberEntity` 的实体的物理速度由本组件按快节拍覆写；想解救实体需对其 `ForgetEntity` 或禁用组件，否则外力会被下一节拍覆盖。
- **快慢节拍不对称**：扫描按快慢节拍切换，但 `PullEntities` 恒用 `TICK_FAST_PERIOD` 计算速度步长，注释说明是为配合物理模拟；修改节拍参数时两处要一起核对（[L262-L266](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L262-L266)）。
- **断言文案是复制的**：断言文本写 "Wagpunk Arena Manager"，实为漩涡组件，属源码遗留文案；不要据此推断用途（[L29](../../../raw/current/scripts/components/oceanwhirlportalphysics.lua#L29)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、oceanwhirlbigportal 装配与中心回调、OCEANWHIRLBIGPORTAL_* 调 tuning 段；未覆盖 physicsmodifiedexternally 内部实现与客户端表现。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/worldmigrator]]（中心传送目标）、[[reference/components/wateryprotection]]。
