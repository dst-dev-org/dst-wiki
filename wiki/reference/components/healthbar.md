---
title: "healthbar"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "在实体世界位置上方用非网络 Image／Label 子实体显示生命百分比，服务端按 healthdelta 写 net_float 供客户端重绘；只做显示。"
aliases: ["头顶血条", "世界空间血条", "百分比血条", "healthbar"]
tags: [component, ui, network-sync, server-client, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/healthbar.lua", "components/health.lua", "entityscript.lua", "mainfunctions.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# healthbar

## 快速判断

- 职责与适用：给已联网实体画世界空间百分比血条。组件自建背景与条形两个 `Non-networked` 子实体（`Image`，条形另带 `Label`）；主模拟端把 `healthdelta.newpercent` 写入 `net_float`，客户端按 dirty 重绘。不读写存档、不改 `health`。[构造](../../../raw/current/scripts/components/healthbar.lua#L77-L104) [初始化](../../../raw/current/scripts/components/healthbar.lua#L61-L75)
- 优先使用：需要头顶世界空间百分比条时用本组件；HUD 内宠物条用 [[reference/components/pethealthbar]]，玩家自身 HUD 用 `healthbadge`＋`replica.health`，改生命用 [[reference/components/health]]。
- 重要副作用：`AddComponent` 当帧注册 `net_float` 并排队零延迟初始化；`Enable(false)` 只隐藏、不清网络值；`SetValue` 只改绘制，下一次 dirty 会覆盖。[网络值](../../../raw/current/scripts/components/healthbar.lua#L99-L100) [Enable](../../../raw/current/scripts/components/healthbar.lua#L116-L123)
- 修改边界：本快照未发现原版宿主；不改生命；组件不实现距离／遮挡／淡出等显示筛选；动态移除组件不安全（见陷阱）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `bar_atlas`／`bar_image`／`bar_colour`／`bg_colour` | 字符串与 `{r,g,b,a}`（0..1）；默认 `images/hud.xml`／`stat_bar.tex`／暗红／近黑 | 图集经 `resolvefilepath` 后设给两条 `Image`，颜色只作 `SetTint`；默认取核心 HUD 图集，自定义图集需另声明资源（未核验）。[默认值](../../../raw/current/scripts/components/healthbar.lua#L82-L89) |
| `bar_width`／`bar_height`／`bar_border` | 数字，默认 100／10／1 | 仅 `CreateBar` 时据此算 `fill_width`／`fill_height`；初始化后再改不重建几何。[填充尺寸](../../../raw/current/scripts/components/healthbar.lua#L43-L44) |
| `bar_world_offset`／`bar_ui_offset`／`label_ui_offset` | `Vector3`，默认 `(0,3,0)`／`(12,0,0)`／`(-50,0,0)` | 前两者给 `Image` 世界／界面偏移，`Label` 用同一世界偏移加自己的界面偏移；文字位置独立于条宽。[偏移](../../../raw/current/scripts/components/healthbar.lua#L10-L41) |
| `_healthpct` | `net_float`，0..1 比例，默认 1 | 服务端写、客户端读；无 getter，需要时读 `bar._healthpct:value()`。是百分比，不是当前／最大生命值。[网络值](../../../raw/current/scripts/components/healthbar.lua#L99-L100) |
| `Enable(enable)` | 缺省视为开（`enable ~= false`） | 开时按当前 `_healthpct` 重绘，关时隐藏背景、条与文字；不改网络值。初始化前 `Enable(false)` 安全，`Enable(true)`／`Enable()` 会访问 nil 的 `self.bar`。[Enable](../../../raw/current/scripts/components/healthbar.lua#L116-L123) |
| `SetValue(percent)` | 0..1 比例，>1 不封顶 | 绘制入口：`>0` 时设条宽、`Label` 显示 `max(1,floor(percent*100))%` 并居中缩放；`<=0` 整条隐藏。不写 `_healthpct`。[SetValue](../../../raw/current/scripts/components/healthbar.lua#L125-L136) |
| `healthdelta`（服务端）／`healthpctdirty`（客户端） | 事件 | `OnInit` 先建实体再按 `TheWorld.ismastersim` 分派：服务端听 `healthdelta` 取 `newpercent` 并写网络值，客户端听 dirty 重绘；初值取 `health:GetPercent()`，无 `health` 时为 1。[接线](../../../raw/current/scripts/components/healthbar.lua#L61-L75) |
| `bg`／`bar` | 实体句柄，仅 `OnInit` 后存在 | 直接操作 `Image`／`Label` 的入口；组件没有销毁它们的钩子。[句柄](../../../raw/current/scripts/components/healthbar.lua#L61-L66) |

## 典型接入

### 给生物加头顶百分比血条（双端安装）

- 前提与选择：`modmain.lua` 用 `AddPrefabPostInit`；引擎创建实体时执行该回调，服务端与客户端各自建立组件实例。目标实体服务端需有 `health`，否则长期显示 100%。默认用核心 HUD 图集，无需新资源。[postinit](../../../raw/current/scripts/mainfunctions.lua#L373-L403)
- 生效与撤销：服务端 `healthdelta` → `_healthpct` → 客户端 `healthpctdirty` → `SetValue`；停用显示用 `Enable(false)`，不要动态 `RemoveComponent`（见陷阱）。
- 原版依据：无原版宿主，下列为按接口推导的最小安装，不是原版接线，也未运行。

```lua
-- modmain.lua：实体生成后双端安装；若只在 master_postinit 安装，客户端不会创建显示实体
AddPrefabPostInit("pigman", function(inst)
    if inst.components.healthbar ~= nil then
        return
    end

    local bar = inst:AddComponent("healthbar")
    -- 这些字段必须在 OnInit（DoTaskInTime(0)）执行前设置；
    -- 初始化完成后再改尺寸／图集不会重建已创建的 Image/Label
    bar.bar_width = 80
    bar.bar_height = 8
    bar.bar_world_offset = GLOBAL.Vector3(0, 2.5, 0)
    bar.label_ui_offset = GLOBAL.Vector3(-32, 0, 0)
    bar.bar_colour = { .85, .2, .15, 1 }
    bar.bg_colour = { .05, .05, .05, 1 }
    -- 不要在这里调用 bar:Enable(true) 或 SetValue()：self.bar 尚未创建
end)
```

- 调整与验证：改宽度后同步调 `label_ui_offset`；`Label` 只显示百分比，没有名称与绝对数值。未运行。

## 常见陷阱与限制

- **未发现原版宿主**：全库 `AddComponent("healthbar")` 为 0，`entity:AddImage()` 仅出现在本组件；没有原版实例支撑其表现，不能据此推断"原版怪物血条就是这样"。
- **只在服务端安装**：`OnInit` 在端判断之前就创建两个显示实体，所以组件装在哪一端就在哪一端建实体；只在服务端安装时客户端没有组件实例、不渲染，远端玩家看不到。端分支只决定监听与更新方式（服务端写、客户端读）。[创建与分支](../../../raw/current/scripts/components/healthbar.lua#L61-L75)
- **初始化前调 Enable(true)／SetValue**：`OnInit` 由 `DoTaskInTime(0)` 排队，此时 `self.bar` 仍为 nil；构造后同帧只改字段。初始隐藏写 `bar.enabled = false`。[排队](../../../raw/current/scripts/components/healthbar.lua#L102-L104) [判空](../../../raw/current/scripts/components/healthbar.lua#L106-L114)
- **动态移除组件不安全**：没有 `OnRemoveFromEntity`；`RemoveComponent` 不注销 `healthdelta`／`healthpctdirty` 监听，残留回调随后读 `inst.components.healthbar`（已为 nil）并报错，提前隐藏显示也不能避免，`bar`／`bg` 也不会被隐藏或删除。停用请用 `Enable(false)`，撤销请删除宿主实体。[回调取值](../../../raw/current/scripts/components/healthbar.lua#L51-L59) [移除](../../../raw/current/scripts/entityscript.lua#L670-L684)
- **SetValue 不是设生命**：不改 `health`，也不写 `_healthpct`；比例不封顶（>1 超出背景），`<=0` 隐藏，文字最小 1%。[SetValue](../../../raw/current/scripts/components/healthbar.lua#L125-L136)
- **组件不做显示筛选**：没有距离、遮挡、淡出或名字，只在比例归零时隐藏，多单位叠放会互相覆盖；引擎是否对世界空间 `Image` 做屏幕外剔除本页未核验。[SetVisible](../../../raw/current/scripts/components/healthbar.lua#L106-L114)
- **宿主删除后的清理未核验**：两个显示实体用 `entity:SetParent` 直挂，未登记 `EntityScript.children`，宿主 `Remove` 的子实体循环覆盖不到；删除宿主后的引擎行为需实测。[登记](../../../raw/current/scripts/entityscript.lua#L913-L934)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 2 处（mainfunctions SpawnPrefabFromSim／modprefabinitfns 段；entityscript children 登记锚点收口检查发现漂移，修正为 RemoveChild／AddChild 两函数区间 891-912）；无结论修订，仅锚点迁移——依据：全局枢纽 diff 审阅（R-09 改动不涉 SpawnPrefabFromSim）＋变更清单命中 mainfunctions.lua、entityscript.lua（湿润／距离函数改动与本页 RemoveComponent／children 登记结论无关，healthbar.lua、health.lua 未变）。
- 源码审阅：2026-10-21，Steam buildid 25643519；完整阅读 `components/healthbar.lua`（138 行），核验创建顺序与端分支、`net_float` 写入／dirty 重绘、初始化时序、`healthdelta`／`GetPercent` 生产、`EntityScript:RemoveComponent` 与 `SetParent` 未被 `children` 登记、`AddPrefabPostInit` 执行路径；全库搜索未发现原版宿主。客户端侧的 `SpawnPrefabFromSim` 调用点在 Lua 中无直接引用（引擎调用），双端安装按 postinit 行为推定，未单独追踪。
- 结构检查：`validate_page.py` 无 issue；`validate_source_links.py --strict` 为 0 error／0 warning（本页 21 处源码引用）；示例 756 B 经 `parser_probe.exe` 完成 Lua 5.1 syntax-only。
- 游戏运行：未运行；世界坐标渲染、客户端 dirty 刷新、分辨率与多血条叠加、自定义图集均未覆盖。

相关：[[reference/components/health]]；[[reference/components/healthsyncer]]；[[reference/components/pethealthbar]]。
