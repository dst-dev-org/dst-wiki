---
title: "vase"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "花瓶插花接收端：接收 DECORATEVASE 放入的花，管理新鲜／蔫化状态、发光定时与花样式推送回调，支持读档与跳时间。"
aliases: ["花瓶", "插花容器", "vase组件"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/vase.lua", "prefabs/decor_flowervase.lua", "actions.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# vase

## 快速判断

- 职责与适用：挂在花瓶类家具（decor_flowervase、endtable、vaultrelic，[装配例](../../../raw/current/scripts/prefabs/decor_flowervase.lua#L169-L172)）上，接收玩家用 [[reference/components/vasedecoration]] 物品发动的 DECORATEVASE，保存花样式并驱动蔫化与灯光。
- 优先使用：`Decorate(giver, item)` 是动作执行入口（[执行](../../../raw/current/scripts/actions.lua#L2452-L2455)）；程序插花用 `SetFlower(flowerid, wilt_time)`（[L119-L154](../../../raw/current/scripts/components/vase.lua#L119-L154)）。
- 重要副作用：`enabled` 字段带赋值监听，同步宿主 `vase` 标签（[L1-L3](../../../raw/current/scripts/components/vase.lua#L1-L3)）；发光花会在宿主上开周期灯光任务，进出 limbo 自动熄灭／恢复（[L13-L31](../../../raw/current/scripts/components/vase.lua#L13-L31)）。
- 修改边界：可插花种类完全由 `TUNING.VASE_FLOWER_MAP` 决定（[映射](../../../raw/current/scripts/tuning.lua#L8007-L8017)）；存读只存花 id 与剩余蔫化时间（[L230-L244](../../../raw/current/scripts/components/vase.lua#L230-L244)）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `Decorate(giver, item)` | → `bool` | 检查 `enabled`；按 `VASE_FLOWER_MAP[item.prefab]` 随机选一个样式，堆叠取一或从持有者移除，`deleteitemonaccept=true` 时销毁物品，`wilt_time` 取自 perishable 剩余百分比 × `TUNING.ENDTABLE_FLOWER_WILTTIME`（[L198-L228](../../../raw/current/scripts/components/vase.lua#L198-L228)）。 |
| `SetFlower(flowerid, wilt_time)` | wilt_time 秒，nil 用默认 | `wilt_time == 0` 直接记蔫；非发光花只记新鲜；发光花开 `DoTaskInTime` 蔫化任务＋`DoPeriodicTask` 灯光任务（[L119-L154](../../../raw/current/scripts/components/vase.lua#L119-L154)；[蔫化时长](../../../raw/current/scripts/tuning.lua#L341-L342)）。 |
| `WiltFlower()` / `ClearFlower()` | 无参 | 蔫化保留花样式仅转状态；清除移除样式并停任务、灭灯（[L156-L196](../../../raw/current/scripts/components/vase.lua#L156-L196)）。 |
| `SetOnUpdateFlowerFn(fn)` / `SetOnUpdateLightFn(fn)` / `SetOnDecorateFn(fn)` | 回调注入 | `PushFlower`／`PushLight` 转发到宿主回调做换图与灯光；`ondecorate(inst, giver, item, flowerid)` 在 Decorate 成功后调用（[L67-L109](../../../raw/current/scripts/components/vase.lua#L67-L109)、[L223-L225](../../../raw/current/scripts/components/vase.lua#L223-L225)）。 |
| `HasFlower()` / `HasFreshFlower()` / `GetTimeToWilt()` | 查询 | 宿主用它们决定换图、理智增益与"整理"动作可达性（[消费例](../../../raw/current/scripts/prefabs/decor_flowervase.lua#L57-L74)）。 |
| `LongUpdate(dt)` | 秒 | 跳睡时间推进蔫化任务，剩余不足即蔫（[L246-L259](../../../raw/current/scripts/components/vase.lua#L246-L259)）。 |

## 典型接入

### 自定义花瓶家具接收装饰（原版同构）

- 前提与选择：宿主 prefab 在 pristine 前加 `vase` 标签（动作候选靠它，[候选说明](../../../raw/current/scripts/componentactions.lua#L1915-L1921)），主机侧加组件并注入三个回调；原版 decor_flowervase 用 `OnUpdateFlower` 换动画符号、`OnUpdateLight` 调 Light（[装配](../../../raw/current/scripts/prefabs/decor_flowervase.lua#L169-L172)）。
- 生效与撤销：DECORATEVASE 执行时先检查目标 `components.vase.enabled`，再调 `Decorate`；`enabled=false` 时动作仍出现但执行静默失败（[执行](../../../raw/current/scripts/actions.lua#L2452-L2455)）。

```lua
local function fn()
    -- ... CreateEntity、pristine 段内: inst:AddTag("vase") ...
    local vase = inst:AddComponent("vase")
    vase:SetOnUpdateFlowerFn(function(inst_, flowerid, fresh)
        -- 按样式换动画 / 贴图
    end)
    vase:SetOnUpdateLightFn(function(inst_, radius, intensity, falloff)
        -- PushLight 转发于此; radius==0 表示熄灯
    end)
    vase:SetOnDecorateFn(function(inst_, giver, item, flowerid)
        -- 可选: 装饰成功回调
    end)
    return inst
end
```

- 调整与验证：花的样式索引写在 `TUNING.VASE_FLOWER_SWAPS`（`lightsource` 决定是否发光），`VASE_FLOWER_MAP` 把物品 prefab 映射到样式下标数组；映射缺失的物品插不进去（[调 tuning](../../../raw/current/scripts/tuning.lua#L7988-L8017)）。

## 常见陷阱与限制

- **enabled 与标签强绑定**：`enabled` 赋值监听直接增删宿主 `vase` 标签；绕过字段直接 `Disable()` 后再手工加标签会与动作判定不一致，务必用 `Enable/Disable`（[L1-L3](../../../raw/current/scripts/components/vase.lua#L1-L3)）。
- **蔫化只在发光花上计时**：`TUNING.VASE_FLOWER_SWAPS[flowerid].lightsource == false` 的花永远 `fresh = true` 不蔫；`wilt_time` 参数传 0 才会立即记蔫（[L132-L151](../../../raw/current/scripts/components/vase.lua#L132-L151)）。
- **OnLoad 依赖 PushFlower 回调**：读档走 `SetFlower`，宿主没注入 `onupdateflowerfn` 时读档后画面不会更新花；组件只管状态不管表现（[L240-L244](../../../raw/current/scripts/components/vase.lua#L240-L244)）。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。覆盖组件全文、decor_flowervase 装配与查询消费、DECORATEVASE 执行行、VASE_FLOWER_SWAPS/MAP 与 ENDTABLE 调 tuning 段；未覆盖 endtable／vaultrelic 的差异分支与皮肤系统联动。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、Lua 语法探测通过（见批级记录）。

相关：[[reference/components/vasedecoration]]（物品端标记）、[[reference/components/perishable]]（蔫化时长来源）。
