---
title: "raindome"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "作为雨棚提供者，在服务端维护可开关的半径并同步客户端，周期给半径内 inspectable 实体写入 rainimmunity 来源，并提供全图雨棚查询。"
aliases: ["雨棚", "遮雨穹顶", "虚空伞雨棚", "免雨领域", "raindome"]
tags: [component, lifecycle, aura, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/raindome.lua", "components/rainimmunity.lua", "prefabs/voidcloth_umbrella.lua", "tuning.lua", "components/raindomewatcher.lua", "widgets/raindomeover.lua", "prefabs/rain.lua", "prefabs/caverain.lua", "prefabs/caveacidrain.lua", "prefabs/lunarhail.lua", "components/inventoryitem.lua", "components/lunarhailmanager.lua", "components/machine.lua", "componentutil.lua", "main.lua", "entityscript.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# raindome

## 快速判断

- 职责与适用：这是**提供者／领域组件**，不是免雨本身：挂在雨棚源实体上，服务端按半径周期扫描 `inspectable` 实体，为它们补 `rainimmunity` 组件并以雨棚实体作为来源。目标侧的增湿、酸雨等消费分工见 [[reference/components/rainimmunity]]。[构造](../../../raw/current/scripts/components/raindome.lua#L80-L104) [周期授予](../../../raw/current/scripts/components/raindome.lua#L183-L216)
- 优先使用：需要“进入区域即免雨”时用 `SetRadius` + `Enable`；只需要问“某点是否在雨棚内”时直接用全局 `IsUnderRainDomeAtXZ`，不要为一次查询建组件。[查询](../../../raw/current/scripts/components/raindome.lua#L45-L59)
- 重要副作用：`Enable` 会加 `raindome` 标签并启动更新循环；`Disable` 会清掉本雨棚写入的全部来源、摘标签、停更新；组件本身不能 `RemoveComponent`（会 `assert(false)`），只能停用。[启停与清理](../../../raw/current/scripts/components/raindome.lua#L146-L178) [禁止移除](../../../raw/current/scripts/components/raindome.lua#L109-L111)
- 修改边界：半径、启停与来源授予都在服务端；客户端只有 `net_float` 活动半径、标签和 `GetActiveRadius()`，`radius` 字段不在客户端初始化，不能用来算范围。[两端分支](../../../raw/current/scripts/components/raindome.lua#L84-L99) [活动半径](../../../raw/current/scripts/components/raindome.lua#L119-L121)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `radius` | number，服务端默认 `16` | 服务端配置；赋值经属性 setter `onradius`，仅在 `enabled` 时立即转发到活动半径；客户端不初始化。[L63-L67](../../../raw/current/scripts/components/raindome.lua#L63-L67) [L91-L103](../../../raw/current/scripts/components/raindome.lua#L91-L103) |
| `enabled` | boolean，服务端 | `Enable`／`Disable` 的幂等门槛；不参与存档，读档由宿主重放。[L132-L144](../../../raw/current/scripts/components/raindome.lua#L132-L144) |
| `_activeradius` | `net_float` | 客户端与查询读的“当前生效半径”；0 表示停用，也是 `raindome` 标签存在与否的边界。[L87](../../../raw/current/scripts/components/raindome.lua#L87) [L176](../../../raw/current/scripts/components/raindome.lua#L176) |
| `GetActiveRadius()` | number | 两端可调；返回网络值，不是 `radius` 配置值；HUD 也用它。[L119-L121](../../../raw/current/scripts/components/raindome.lua#L119-L121) [HUD](../../../raw/current/scripts/widgets/raindomeover.lua#L45-L49) |
| `SetRadius(radius)` | 服务端 | 只写 `self.radius`；已启用时同步缩短／扩大半径与查询范围。[L126-L130](../../../raw/current/scripts/components/raindome.lua#L126-L130) |
| `Enable()`／`Disable()` | 服务端 | 从 0 到半径／从半径到 0；重复调用无额外效果。[L132-L144](../../../raw/current/scripts/components/raindome.lua#L132-L144) |
| `SetActiveRadius_Internal(new, old)` | 内部 | 维护 `_sizes` 计数、`raindome` 标签、更新循环；归零时逐个 `RemoveSource` 并清空目标表。[L146-L178](../../../raw/current/scripts/components/raindome.lua#L146-L178) |
| `OnUpdate(dt)` | 内部 | 半径内 `inspectable` 且非 `INLIMBO` 的实体补 `rainimmunity` 并 `AddSource(self.inst)`；离开者撤销来源；扫描间隔 1 秒，睡着时 3 秒。[L180-L216](../../../raw/current/scripts/components/raindome.lua#L180-L216) |
| 全局 `GetRainDomesAtXZ(x, z)` | 返回雨棚实体数组 | 两端可用；按 `_maxsize` 与各棚 `GetActiveRadius` 过滤；由 `componentutil` 提前 require 加载。[L29-L43](../../../raw/current/scripts/components/raindome.lua#L29-L43) [提前加载](../../../raw/current/scripts/componentutil.lua#L1) |
| 全局 `IsUnderRainDomeAtXZ(x, z)` | boolean | 两端可用；任一半径覆盖即真，边界比较用 `<=`。[L45-L59](../../../raw/current/scripts/components/raindome.lua#L45-L59) |
| `_sizes`／`_maxsize` | 模块局部表／number | 活动半径计数器；服务端在启停时维护，客户端由 `_activeradiusdirty` 维护，用于限制 `FindEntities` 查询半径。[L1-L22](../../../raw/current/scripts/components/raindome.lua#L1-L22) [客户端](../../../raw/current/scripts/components/raindome.lua#L69-L78) |

### 谁写、谁读、何时撤

| 环节 | 位置与触发 | 结果 |
|---|---|---|
| 写 | 服务端 `Enable`／`Disable`／`SetRadius` → `SetActiveRadius_Internal` | 写 `enabled`、`_activeradius`、`radius`，增删 `raindome` 标签与更新循环。[L132-L178](../../../raw/current/scripts/components/raindome.lua#L132-L178) |
| 写 | 服务端 `OnUpdate` | 给半径内目标 `AddComponent("rainimmunity")` + `AddSource(self.inst)`；离开的目标 `RemoveSource`。[L183-L216](../../../raw/current/scripts/components/raindome.lua#L183-L216) |
| 读 | 全局查询、玩家 watcher、HUD、雨／酸雨／月雹粒子、物品继承世界湿度、月雹落点保护 | 按 `raindome` 标签与活动半径判定；消费点见“典型接入”。[查询](../../../raw/current/scripts/components/raindome.lua#L29-L59) |
| 撤 | `Disable` | 遍历 `self.targets` 撤销本棚来源，摘标签，停更新，活动半径归零。[L146-L163](../../../raw/current/scripts/components/raindome.lua#L146-L163) |
| 撤 | 来源实体被移除 | `rainimmunity` 对非自身来源注册的 `onremove` 先清理来源；随后 `OnRemoveEntity` 只从 `_sizes` 注销，不负责摘目标来源。[L113-L117](../../../raw/current/scripts/components/raindome.lua#L113-L117) [目标侧监听](../../../raw/current/scripts/components/rainimmunity.lua#L32-L39) |
| 撤 | 组件本身 | `OnRemoveFromEntity` 直接 `assert(false)`，没有正常移除路径；关棚必须走 `Disable`。[L109-L111](../../../raw/current/scripts/components/raindome.lua#L109-L111) |

## 典型接入

### 原版宿主：虚空伞 `voidcloth_umbrella`

- 文件与端：`prefabs/voidcloth_umbrella.lua`；`UmbrellaFn` 在 `SetPristine` 前 `AddComponent("raindome")`，注释明确“必须客户端也加，但只在服务端配置”。服务端在 pristine 后 `SetRadius(TUNING.VOIDCLOTH_UMBRELLA_DOME_RADIUS)`。[装配与配置](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L375-L391)
- 注册名与单位：prefab 名 `voidcloth_umbrella`；半径 `16` 世界坐标单位；`VOIDCLOTH_UMBRELLA_DOME_RATE=1.5` 是燃料消耗速率，不是半径。[注册](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L525-L527) [数值](../../../raw/current/scripts/tuning.lua#L7804-L7807)
- 生效：`machine:TurnOn` 调 `turnon`，在燃料非空时 `raindome:Enable()`；`turnoff` 调 `Disable()`。燃料耗尽 `OnPerish` 会先 `TurnOff`。[开](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L240-L245) [关](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L273-L277) [耗尽](../../../raw/current/scripts/prefabs/voidcloth_umbrella.lua#L108-L111)
- 读档：`machine` 只保存 `ison`，`OnLoad` 重新 `TurnOn`／`TurnOff`，因此 `Enable` 会被重放；`rainimmunity` 来源本身不存档，目标要等下一轮 `OnUpdate` 重新写入。[Machine 存读](../../../raw/current/scripts/components/machine.lua#L50-L61) [来源不存档](../../../raw/current/scripts/components/rainimmunity.lua#L1-L18)
- 撤销：`Disable` 立即撤销来源；直接移除伞实体时由 `onremove` 监听兜底；`OnRemoveEntity` 只清 `_sizes`。客户端 HUD 不读 `.radius`，只用 `GetActiveRadius()`。[HUD](../../../raw/current/scripts/widgets/raindomeover.lua#L45-L49)

### 模组接入：查询某点是否在雨棚内

- 前提与选择：`componentutil` 在 `main.lua` 提前 require 了 `components/raindome`，所以全局函数在服务端和客户端都可用；坐标是 x/z，y 不参与。[加载顺序](../../../raw/current/scripts/main.lua#L260) [签名](../../../raw/current/scripts/components/raindome.lua#L45-L59)
- 生效与撤销：只读查询，随时可调；返回的雨棚实体在下一次 `Disable`／实体移除后失效，跨帧保存时应先 `IsValid()`。
- 调整：需要“区域免雨”而不是查询时，才创建自己的提供者；两端都 `AddComponent("raindome")`，服务端 `SetRadius` + `Enable`，撤销只 `Disable`，不要 `RemoveComponent`。

```lua
local function GetDomeInfoAt(inst)
    local x, _, z = inst.Transform:GetWorldPosition()
    return IsUnderRainDomeAtXZ(x, z), GetRainDomesAtXZ(x, z)
end
```

- 验证：该片段只调用已核验全局函数，未运行游戏；语法检查见验证记录。

## 常见陷阱与限制

- **组件存在不等于生效**：`AddComponent` 后 `components.raindome ~= nil` 一直成立；判断是否生效要看 `GetActiveRadius() > 0` 或 `HasTag("raindome")`，不要用组件存在性。[停用后状态](../../../raw/current/scripts/components/raindome.lua#L146-L163)
- **不能移除组件**：`RemoveComponent("raindome")` 会触发 `OnRemoveFromEntity` 的 `assert(false)`；持久实体只能 `Disable`。[L109-L111](../../../raw/current/scripts/components/raindome.lua#L109-L111)
- **客户端读不到 `.radius`**：客户端构造不设置 `radius`，只监听 `_activeradius`；读原始字段会得到 `nil`，范围计算必须用 `GetActiveRadius()`。[L84-L99](../../../raw/current/scripts/components/raindome.lua#L84-L99)
- **授予不是瞬时**：首次启用随机延迟最多 0.5 秒，之后每 1 秒（睡着时 3 秒）扫一次；进入或离开半径最多要等一个周期才反映到 `rainimmunity`。[L170](../../../raw/current/scripts/components/raindome.lua#L170) [L215](../../../raw/current/scripts/components/raindome.lua#L215)
- **改半径后查询先变、授予后变**：已启用时 `SetRadius` 立即改活动半径与查询，但目标集合要等下一轮 `OnUpdate` 用新 `self.radius` 重扫；中间存在短暂不一致。[L126-L135](../../../raw/current/scripts/components/raindome.lua#L126-L135) [L194](../../../raw/current/scripts/components/raindome.lua#L194)
- **只扫 `inspectable`**：没有 `inspectable` 标签的实体不会被加免疫，`INLIMBO` 实体也被排除；玩家和可检视物品在范围内。[L180-L181](../../../raw/current/scripts/components/raindome.lua#L180-L181)
- **来源可叠加**：目标是按 `src` 集合维护来源的；本棚撤销只移除 `self.inst`，目标若还有其它来源（其它棚、晾晒架等）仍保持免雨。[目标侧](../../../raw/current/scripts/components/rainimmunity.lua#L32-L48)
- **停用没有广播**：`raindome` 不发事件；目标是否收到 `loserainimmunity` 取决于它是否还有其它来源，不能把“棚关了”当作目标一定失去免疫。[目标事件](../../../raw/current/scripts/components/rainimmunity.lua#L20-L30)
- **零棚早退**：没有活动半径时 `GetRainDomesAtXZ` 返回空表、`IsUnderRainDomeAtXZ` 返回 false，无法用它们发现“组件存在但停用”的棚。[L29-L31](../../../raw/current/scripts/components/raindome.lua#L29-L31) [L45-L48](../../../raw/current/scripts/components/raindome.lua#L45-L48)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 2 处（tuning／main 行号迁移）；结论修订：无。R-08 的 CanPointHaveRain 按 topology 节点标签判定，与雨棚 IsUnderRainDomeAtXZ 半径查询互不相干，授予／查询结论不变。
- 依据：全局枢纽 hunk 地图（R-08）＋变更清单命中 tuning.lua、main.lua、entityscript.lua、componentutil.lua。

- 源码审阅：2026-10-21，Steam buildid 25643519；覆盖构造与两端分支、`radius` 属性 setter、`Enable`／`Disable`／`SetActiveRadius_Internal` 的来源清理与标签更新、`OnUpdate` 周期与目标集合交换、`_sizes` 注册、全局查询实现、提前加载、`voidcloth_umbrella` 宿主、`machine` 存读、HUD 读取，以及雨／酸雨／月雹／物品湿度／月雹落点等消费者。
- 未运行游戏；未覆盖：多人联机与读档实测、多个不同半径雨棚的 `_sizes` 边界、模组自定义宿主、伞实体移除时序的运行时验证。
- Lua 语法：仅对上方 `GetDomeInfoAt` 片段做 syntax-only 检查（`parser_probe` 返回 0，临时文件已删除），不代表完整模组可运行。
- 结构检查：`validate_page.py` 无 issue（96 行，任务上限 180 行／14 KiB）；`validate_source_links.py --strict` 0 error／0 warning（53 处源码引用）。

相关：[[reference/components/rainimmunity]]；[[reference/components/raindomewatcher]]；[[reference/components/machine]]。
