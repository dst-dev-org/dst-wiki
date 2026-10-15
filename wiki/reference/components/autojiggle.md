---
title: "autojiggle"
created: "2026-10-26"
updated: "2026-10-28"
type: "component"
summary: "跟随宿主的自动抖动表现组件：盯 owner 的移动/受击/冰冻/电击动画与预测暂停，在边沿触发循环摆动或单次回弹回调，驱动装备跟随 FX 的物理感。"
aliases: ["autojiggle", "自动抖动", "跟随摆动", "跑动摆动回调"]
tags: ["component", "callback", "server-client"]
dst_build: "steam-buildid:26230496"
sources: ["components/autojiggle.lua", "prefabs/woby_rack.lua", "prefabs/armor_voidcloth.lua", "prefabs/hats.lua"]
reliability: "verified"
validation: "source-reviewed"
status: "active"
---

# autojiggle

## 快速判断

- 职责与适用：挂在跟随实体（装备展示 FX、Woby 背架）上，逐帧读 owner 的 locomotion 状态并在状态边沿触发 `onjiggleloopfn`（进入奔跑）或 `onjiggleoneshotfn`（停止、受击）。组件只发回调，动画切换由宿主回调自己播。[autojiggle.lua:77-123](../../../raw/current/scripts/components/autojiggle.lua#L77-L123)
- 优先使用：宿主上 `AddComponent("autojiggle")` → `SetOnJiggleLoopFn`／`SetOnJiggleOneShotFn` → 需要区分走/跑时 `OverrideRunAnims` → 最后 `SetOwner(owner)` 启动；owner 离开用 `SetOwner(nil)` 停表。[autojiggle.lua:14-21](../../../raw/current/scripts/components/autojiggle.lua#L14-L21)、[armor_voidcloth.lua:204-206](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L204-L206)
- 重要副作用：`SetOwner` 立即按实体睡眠状态启停更新并补跑一帧 `OnUpdate(0)`；owner 已在奔跑时挂上即触发一次 loop 回调。[autojiggle.lua:14-21](../../../raw/current/scripts/components/autojiggle.lua#L14-L21)、[:57-66](../../../raw/current/scripts/components/autojiggle.lua#L57-L66)
- 修改边界：纯表现组件，无网络同步；可同时跑在服务端与客户端（原版即客户端挂载，预测暂停在两端用不同读法）。owner 必须带 AnimState。[armor_voidcloth.lua:211-216](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L211-L216)、[:68-75](../../../raw/current/scripts/components/autojiggle.lua#L68-L75)

## 关键接口

| 接口／字段 | 签名或默认 | 条件、作用与证据 |
|---|---|---|
| `SetOwner(owner)` | owner 实体或 nil | nil → OnEntitySleep 停更；非 nil 且宿主未睡 → OnEntityWake 启动（含 `sleepstatepending` 检查）。owner 只存字段，不建立事件连接。[autojiggle.lua:14-21](../../../raw/current/scripts/components/autojiggle.lua#L14-L21) |
| `SetOnJiggleLoopFn(fn)` | `fn(inst)` | 进入 running 边沿触发；宿主回调内自行播循环动画。[autojiggle.lua:23-25](../../../raw/current/scripts/components/autojiggle.lua#L23-L25)、[:105-110](../../../raw/current/scripts/components/autojiggle.lua#L105-L110) |
| `SetOnJiggleOneShotFn(fn)` | `fn(inst)` | 停止奔跑／停止走动／nopredict 升沿（受击？）触发。[autojiggle.lua:27-29](../../../raw/current/scripts/components/autojiggle.lua#L27-L29)、[:111-118](../../../raw/current/scripts/components/autojiggle.lua#L111-L118) |
| `OverrideRunAnims(anims)` 等 | 默认见下表 | 四组覆写整表替换，不是追加；列表用于 `OwnerMatchesAnimSet` 命中判定。[autojiggle.lua:31-45](../../../raw/current/scripts/components/autojiggle.lua#L31-L45) |
| `OwnerMatchesAnimSet(anims)` | boolean | owner 当前动画是否命中列表任一项（`AnimState:IsCurrentAnimation`）。[autojiggle.lua:68-75](../../../raw/current/scripts/components/autojiggle.lua#L68-L75) |
| `OnEntitySleep`／`OnEntityWake` | 引擎生命周期钩子 | 睡→停更并清 `_wasmoving/_wasrunning/_wasnopredict`；醒→重置为 false 并立即 `OnUpdate(0)`。[autojiggle.lua:47-66](../../../raw/current/scripts/components/autojiggle.lua#L47-L66) |

默认动画集（构造内初始化，覆写用）：`runanims = {"run_pre","run_loop"}`、`frozenanims = {"frozen","frozen_loop_pst"}`、`hitanims = {"hit","shock_loop"}`、`electrocuteanims = {"shock","shock_loop"}`。[autojiggle.lua:8-11](../../../raw/current/scripts/components/autojiggle.lua#L8-L11)

### OnUpdate 判定流程

1. **moving**：owner 有 sg 用 `sg:HasStateTag("moving")`，否则退 `owner:HasTag("moving")`。[autojiggle.lua:79-83](../../../raw/current/scripts/components/autojiggle.lua#L79-L83)
2. **running**：moving 时 `inst.isplayer or not inst:HasAnyTag("character","companion") or OwnerMatchesAnimSet(runanims)`——前两项作用于宿主实体；原版三宿主均为 FX/decor 实体（无这两标签），短路使 moving 恒判 running，runanims 匹配实际不参与。非 moving 且命中 electrocuteanims → running=true、nopredict=true（电击期用奔跑摆动）。[autojiggle.lua:84-93](../../../raw/current/scripts/components/autojiggle.lua#L84-L93)、[woby_rack.lua:193-194](../../../raw/current/scripts/prefabs/woby_rack.lua#L193-L194)、[armor_voidcloth.lua:256](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L256)、[hats.lua:7670](../../../raw/current/scripts/prefabs/hats.lua#L7670
3. **nopredict**（受击/暂停征兆）：非 moving、非冰冻时——主模拟读 `owner.sg:HasAnyStateTag("nopredict","pausepredict")`；客户端读 owner 标签或 `owner.player_classified.pausepredictionframes:value() > 0`，再或命中 hitanims。命中 frozenanims 时整段跳过（冰冻不触发受击单次）。[autojiggle.lua:95-102](../../../raw/current/scripts/components/autojiggle.lua#L95-L102)
4. **触发**：running 升沿→loop；`_wasrunning`→停 或 `_wasmoving`→停走 或 nopredict 升沿→one-shot；随后记录三个边沿标志。[autojiggle.lua:105-122](../../../raw/current/scripts/components/autojiggle.lua#L105-L122)

## 典型接入

### 原版三种宿主配置

- **woby_rack（Woby 背架，网络化 decor 实体）**：owner 随 highlightchild 换主回调（`SetOnChangeOwnerFn`）；owner 置空时整体 `RemoveComponent("autojiggle")` 连同 updatelooper 拆除，恢复时重建并配齐三件套——loop 随机播 `loop_swing_run1-3`、one-shot 播 `pst_swing_settle1-3` 后推 `idle_sway1-3`、`OverrideRunAnims({"run_woby_loop","sprint_woby_loop","run_woby_pre"})`。[woby_rack.lua:62-76](../../../raw/current/scripts/prefabs/woby_rack.lua#L62-L76)、[:94-131](../../../raw/current/scripts/prefabs/woby_rack.lua#L94-L131)、[:230](../../../raw/current/scripts/prefabs/woby_rack.lua#L230)
- **armor_voidcloth（虚空布护甲 9 帧跟随特效）**：owner 先查 `HasTag("locomotor")` 再挂组件；只配 one-shot（settle+i 后推 idle+i），不配 loop——奔跑循环摆动不需要。组件在客户端 `OnEntityReplicated` 与服务端非专用（`fx_AttachToOwner`→非 Dedicated 分支）各挂一份，本地各自跑表现。[armor_voidcloth.lua:203-209](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L203-L209)、[:211-216](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L211-L216)、[:229-232](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L229-L232)
- **bat_bosscorpsehat（蝙蝠王冠帽跟随特效，hats.lua）**：同样 `locomotor` 门槛，loop＋one-shot 双回调（`swing_loop..i` 循环、`settle..i`＋`idle..i` 回弹）。[hats.lua:7683-7690](../../../raw/current/scripts/prefabs/hats.lua#L7683-L7690)、[:7692-7704](../../../raw/current/scripts/prefabs/hats.lua#L7692-L7704)

### 给自定义跟随 FX 接奔跑摆动

- 前提与选择：模组 prefab 或 `AddPrefabPostInit`；FX 挂在 owner（玩家/生物）下，owner 带 `locomotor` 才有意义。
- 生效与撤销：挂组件即随宿主睡眠自动启停；卸 owner 用 `SetOwner(nil)`，拆组件等同 woby_rack 的清理路径。

```lua
AddPrefabPostInit("myhat_fx", function(inst)
    if not GLOBAL.TheNet:IsDedicated() then
        local owner = inst.entity:GetParent()
        if owner ~= nil and owner:HasTag("locomotor") then
            inst:AddComponent("autojiggle")
            inst.components.autojiggle:SetOnJiggleLoopFn(function(fx)
                fx.AnimState:PlayAnimation("swing_loop", true)
            end)
            inst.components.autojiggle:SetOnJiggleOneShotFn(function(fx)
                fx.AnimState:PlayAnimation("settle")
                fx.AnimState:PushAnimation("idle", false)
            end)
            inst.components.autojiggle:SetOwner(owner)
        end
    end
end)
```

- 调整与验证：动画名换成实际 build 的名称；`TheNet:IsDedicated()` 门槛沿用原版（专用服务器不跑表现）。[armor_voidcloth.lua:229-232](../../../raw/current/scripts/prefabs/armor_voidcloth.lua#L229-L232)

## 常见陷阱与限制

- **owner 必须带 AnimState**：`OwnerMatchesAnimSet` 直接 `self.owner.AnimState:IsCurrentAnimation`；owner 无 AnimState 时更新帧报错。`SetOwner(nil)` 会先停更，不会踩到。[autojiggle.lua:68-75](../../../raw/current/scripts/components/autojiggle.lua#L68-L75)
- **回调只在边沿触发**：loop 在进入 running 边沿、one-shot 在停止／受击边沿各一次；持续摆动必须在回调里播循环动画（原版均 `PlayAnimation(..., true)`）。宿主休眠期间错过的停止边沿不补发 one-shot。[autojiggle.lua:47-55](../../../raw/current/scripts/components/autojiggle.lua#L47-L55)、[:105-118](../../../raw/current/scripts/components/autojiggle.lua#L105-L118)
- **Override* 是整表替换**：自定义列表需含全部要识别的动画（pre/loop/pst），漏掉会导致该状态判不出；且如上所述，宿主无 character/companion 标签时 runanims 判定被短路跳过。[autojiggle.lua:31-45](../../../raw/current/scripts/components/autojiggle.lua#L31-L45)、[:87](../../../raw/current/scripts/components/autojiggle.lua#L87)
- **`ismastersim` 在构造时快照**：组件在客户端构造则走客户端 nopredict 读法（tags／player_classified），迁移实体实例不重判。[autojiggle.lua:3](../../../raw/current/scripts/components/autojiggle.lua#L3)、[:95-101](../../../raw/current/scripts/components/autojiggle.lua#L95-L101)

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖 autojiggle.lua 全部 125 行（含 OnUpdate 全分支与端别读法）、三个宿主的挂载/回调/清理路径与 owner 门槛（locomotor／highlightchild 换主）。未覆盖：实际动画观感、客户端预测暂停时序的运行验证。
- 游戏运行：未运行。
- 结构检查：2026-10-26 执行 validate_page.py（standard 档）与 validate_source_links.py（--strict）通过；页内 Lua 示例经 parser_probe.exe --syntax-only 语法检查通过。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。