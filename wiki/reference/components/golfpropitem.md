---
title: "golfpropitem"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "嘉年华高尔夫道具的场地约束：出界自动传送回标定位、掉落带减速，并可侵蚀消失；供高尔夫球/球座/球洞使用。"
aliases: ["golfpropitem", "高尔夫道具约束", "高尔夫出界回传"]
tags: [component, item]
size_class: standard
dst_build: "steam-buildid:26230496"
sources: ["components/golfpropitem.lua", "prefabs/carnivalgame_golf_tee.lua", "prefabs/carnivalgame_golfball.lua", "prefabs/carnivalgame_golfgame.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# golfpropitem

## 快速判断

- 职责与适用：给可移动的高尔夫道具（球、球座、球洞）做 XZ 边界约束——出界后传送回 `SetTeleportXZ` 记录的标定位，掉落时以 `droppedspeedmult`（0.25）减速。构造即 `StartUpdatingComponent`，每 0.5 秒查一次出界。[构造](../../../raw/current/scripts/components/golfpropitem.lua#L11-L13) [轮询](../../../raw/current/scripts/components/golfpropitem.lua#L66-L77)
- 优先使用：做受场地限制、需要"出界归位"的道具时，参照原版高尔夫套件：`SetXZBounding` 给边界、`SetTeleportXZ` 给归位点、滚动停止时调 `CheckTeleport`。[球侧消费](../../../raw/current/scripts/prefabs/carnivalgame_golfball.lua#L100-L103)
- 重要副作用：`TeleportBack` 会生成 `carnival_confetti_fx` 特效；`Erode` 会把实体设为不持久、不可拾取并在 0.5 秒后 `ErodeAway`。[回传](../../../raw/current/scripts/components/golfpropitem.lua#L53-L57)
- 修改边界：组件只做位置约束，不参与击球物理（击球在 `golfable`/`golfclub`）；`OnUpdate` 在未 `SetXZBounding`（`minx` 为 nil）时直接跳过。[跳过条件](../../../raw/current/scripts/components/golfpropitem.lua#L72-L74)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetXZBounding(minx, maxx, minz, maxz)` | 四个 number | 设定 XZ 出界判定框；未设置时轮询空转。[实现](../../../raw/current/scripts/components/golfpropitem.lua#L16-L21) |
| `SetTeleportXZ(x, z)` | 两个 number | 设定出界归位点；未设置时回退到边界框中心。[回退](../../../raw/current/scripts/components/golfpropitem.lua#L53-L54) |
| `CheckTeleport()` | `-> nil` | 越界即 `TeleportBack`；原版在球滚动停止时调用。[调用点](../../../raw/current/scripts/prefabs/carnivalgame_golfball.lua#L101-L102) |
| `DropItem(x, z)` | `-> nil` | 从持有者移除并落到 (x, z)，按 `droppedspeedmult` 减速、清 `prevcontainer`/`prevslot`。[实现](../../../raw/current/scripts/components/golfpropitem.lua#L32-L43) |
| `TeleportBack()` | `-> nil` | 生成彩带特效并 DropItem 到归位点。[实现](../../../raw/current/scripts/components/golfpropitem.lua#L53-L57) |
| `StopUpdating()` | `-> nil` | 停止出界轮询；球座放置流程在给球设好归位点后立即调用。[调用点](../../../raw/current/scripts/prefabs/carnivalgame_golf_tee.lua#L130) |
| `Erode(x, z)` | `-> nil` | DropItem 后 0.5 秒 `ErodeAway`、停止轮询、`persists=false`、禁止拾取；当前脚本库内无调用者。[实现](../../../raw/current/scripts/components/golfpropitem.lua#L45-L51) |

## 典型接入

### 高尔夫球的边界与归位（原版参照）

- 前提与选择：球是 `persists=false` 的临时实体，由球座发射时挂组件并完成三项初始化；边界来自 `golfgame:GetWorldCoordBoundaries()`。[球座初始化](../../../raw/current/scripts/prefabs/carnivalgame_golf_tee.lua#L121-L130)
- 生效与撤销：场地物件（球座/球洞 kit）也挂组件，由 `carnivalgame_golfgame` 统一配边界与归位点；球被击中（OnHit 且有球杆）时把当前位置更新为新的归位点。[场地配置](../../../raw/current/scripts/prefabs/carnivalgame_golfgame.lua#L108-L118) [击球更新](../../../raw/current/scripts/prefabs/carnivalgame_golfball.lua#L126-L128)

```lua
-- 出界归位道具的最小接入（服务端）
inst:AddComponent("golfpropitem")
inst.components.golfpropitem:SetXZBounding(minx, maxx, minz, maxz)
inst.components.golfpropitem:SetTeleportXZ(home_x, home_z)
-- 静止实体不再需要轮询时：
inst.components.golfpropitem:StopUpdating()
```

- 调整与验证：轮询间隔 0.5 秒、`droppedspeedmult` 0.25 均为组件内定值，不改则无需配置；归位点语义由消费方决定（球=最后击球点，kit=场地内默认位）。

## 常见陷阱与限制

- **只设边界不设归位点：** `TeleportBack` 回退到边界框中心，可能与玩法预期不符；需要固定落点就先 `SetTeleportXZ`。[回退逻辑](../../../raw/current/scripts/components/golfpropitem.lua#L53-L54)
- **依赖掉落接口的实体没有 `inventoryitem`：** `DropItem` 会退化为直接 `SetPosition`，且 `Erode` 里 `inst.components.inventoryitem.canbepickedup` 会因组件缺失报错。[Erode 读取](../../../raw/current/scripts/components/golfpropitem.lua#L50)
- **`Erode` 无人调用：** 当前脚本库内没有 `golfpropitem:Erode` 的调用点，属组件预留能力，复用前自行验证 `ErodeAway`（`simutil.lua:575`）行为。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（78 行），核对 golfball 的 CheckTeleport/OnHit 消费、golf_tee 发射初始化、golfgame 场地配置与 golf_hole kit 挂载（`prefabs/carnivalgame_golf_hole.lua:236`）。全库 grep 确认 `Erode` 无调用者。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。
