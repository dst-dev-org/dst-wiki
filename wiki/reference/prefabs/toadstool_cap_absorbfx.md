---
title: "毒菌蟾蜍帽毒雾吸收与释放（toadstool_cap_absorbfx、toadstool_cap_releasefx）"
created: "2026-11-05"
updated: "2026-11-07"
type: prefab
summary: "毒菌蟾蜍帽变暗／复原的两段叠加演出：MakeFX 工厂一鱼两吃（吸收用蘑菇动画库、释放复用金丝雀爆炸动画），引擎父挂帽子并逐帧驱动宿主染色；触发链是中毒金丝雀爆毒与暗态计时器，不是 Boss 状态机。"
aliases: ["toadstool_cap_absorbfx", "toadstool_cap_releasefx", "毒菌蟾蜍帽吸收", "毒菌蟾蜍帽释放", "毒菌蟾蜍帽变暗", "蛤蟆菇帽吸收", "蛤蟆菇帽释放", "帽子吸收毒雾"]
tags: [prefab, fx, server-client]
size_class: standard
dst_build: "steam-buildid:26335394"
sources: ["prefabs/toadstool_cap.lua", "stategraphs/SGcanarypoisoned.lua", "prefabs/birds.lua", "prefabs/birdcage.lua", "components/toadstoolspawner.lua", "map/terrain.lua", "map/static_layouts/toadstool_arena.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 毒菌蟾蜍帽毒雾吸收与释放（toadstool_cap_absorbfx、toadstool_cap_releasefx）

## 快速判断

- 职责与适用：`prefabs/toadstool_cap.lua` 用一个 `MakeFX` 工厂注册 2 个联网 FX：`toadstool_cap_absorbfx`（帽子**吸收毒雾变暗**）与 `toadstool_cap_releasefx`（帽子**释放毒雾复原**）。它们是毒菌蟾蜍帽（toadstool_cap，官方名"毒菌蟾蜍"，旧称蛤蟆菇帽）切暗态／复原时叠在帽子上的短命演出，自身无伤害、无声音，但会逐帧改写宿主帽子的染色（黑→正常渐显）。[注册](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L377-L379)、[工厂](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L340-L375)
- 优先使用：要给固定地物复刻"事件触发的一过性变装演出＋宿主染色"时，`SpawnPrefab` 后引擎父挂到宿主即可——客户端侧的染色协议经 `OnEntityReplicated` 自动挂接，非专服主机需照原版手动调一次（见[典型接入](#在自定义宿主上复刻染色演出)）。
- 重要副作用：染色目标是**宿主**（`parent.AnimState:OverrideMultColour`）不是 FX 自身；FX 删除时经 `OnRemoveEntity` 复位宿主染色。生成有"非 instant"门禁：帽子睡眠（离屏）或读档时直接改 build 不出 FX。
- 修改边界：只覆盖 2 个注册的工厂、生成点、染色协议与触发链；帽子本体玩法（砍帽出 Boss、重生计时、`ms_spawntoadstool` 世界事件）只在辨析与链路里带过，Boss（toadstool／toadstool_dark）战斗不展开。
- 名称：`TOADSTOOL`／`TOADSTOOL_CAP`＝"毒菌蟾蜍"（[po](../../../raw/current/scripts/languages/chinese_s.po#L350420-L350422)、[L350425-L350427](../../../raw/current/scripts/languages/chinese_s.po#L350425-L350427)）、`TOADSTOOL_CAP_DARK`／`TOADSTOOL_DARK`＝"悲惨的毒菌蟾蜍"（[L350430-L350432](../../../raw/current/scripts/languages/chinese_s.po#L350430-L350432)、[L350435-L350437](../../../raw/current/scripts/languages/chinese_s.po#L350435-L350437)）、`TOADSTOOL_HOLE`＝"洞"、`CANARY_POISONED`＝"金丝雀（中毒）"（[L340090-L340092](../../../raw/current/scripts/languages/chinese_s.po#L340090-L340092)）；2 个 FX 注册名在简中 `chinese_s.po` 无 `STRINGS.NAMES` 键；标题采用宿主官方名“毒菌蟾蜍”，旧称“蛤蟆菇帽”保留为别名。

## 注册与构造

`MakeFX(name, assetname, animname, assets)`（[定义](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L340-L375)）：Transform／AnimState／Network → bank `assetname`、build `assetname.."_build"`、播 `animname`（一次性）、FinalOffset 1 → 标签 `FX`＋`NOCLICK` → SetPristine → 非专服挂 `OnRemoveEntity = OnRemoveFXEntity`；客户端另挂 `OnEntityReplicated = OnFXEntityReplicated` 后早退 → 主模拟：`persists=false`、动画长度＋2 帧定时自删（**无 animover 监听**）。

| 注册名 | 资源与动画 | 差异与证据 |
|---|---|---|
| `toadstool_cap_absorbfx` | assets `toadstool_actions.zip`＋`toadstool_build.zip`；bank `toadstool`、build `toadstool_build`、动画 `absorbfx` | 吸收段：帽子同库动画，配合暗 build 的渐显（[注册](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L378)、[assets](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L10-L14)） |
| `toadstool_cap_releasefx` | assets `canary.zip`＋`canary_build.zip`；bank `canary`、build `canary_build`、动画 `explodefx` | 释放段：**复用金丝雀的爆炸动画**（中毒金丝雀爆毒的同一套资源），注册名读不出这一点（[注册](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L379)、[assets](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L16-L20)） |

### 宿主染色协议

两个 FX 共用一套"父体染色"协作（各端本地执行，无 netvar）：

- 客户端：FX 复制到位后 `OnFXEntityReplicated` 自动执行——先把父（帽子）`OverrideMultColour(0,0,0,1)` 全黑，再 `DoPeriodicTask(0, OnFXUpdate)` 逐帧按 FX 动画进度把父染回 `k = 1-(1-t/T)²`（起点黑、终点正常）（[实现](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L43-L49)、[更新](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L37-L41)）。
- 非专服主机：`OnEntityReplicated` 不会触发，由生成方（帽子自己）在生成后手动调同一函数（[生成块](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L56-L61)）；专服不做任何染色。
- 收尾：FX 删除时 `OnRemoveFXEntity` 把存活的父复位 `(1,1,1,1)`（[实现](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L30-L35)）——协议跨实体配对，删 FX 即撤染色。

## 关联机制导航

### 生成点

| 注册 | 生成语句 | 条件 |
|---|---|---|
| `toadstool_cap_absorbfx` | `SpawnPrefab` ＋ `fx.entity:SetParent(inst.entity)`：[toadstool_cap.lua#L78-L79](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L77-L83) | `setdark(inst, duration, instant)` 的 `instant == false` 分支（帽子未睡眠） |
| `toadstool_cap_releasefx` | 同结构：[toadstool_cap.lua#L56-L57](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L51-L67) | `setnormal(inst, instant)` 的 `instant == false` 分支且帽子当前确实处于暗态（`_dark` 为真） |

生成点唯一：两个注册名全域反查只命中本文件（依赖表 [L26-L27](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L22-L28)、生成 L56／L78、注册 L378-L379）。父挂是引擎级 `entity:SetParent`（网络父子复制，FX 随帽子移动）。

### 触发链（金丝雀爆毒 → 帽子变暗 → 复原）

触发不在 Boss 状态机里——Boss（toadstool／toadstool_dark）的 SG 与工厂不触碰这两个 FX；链路两端是中毒金丝雀与帽子自己的计时器：

1. **放气前提**：世界组件 `toadstoolspawner` 的重生定时器停着（即任一帽子 `HasToadstool()` 为真——Boss 存活或蘑菇长出）时 `IsEmittingGas()` 为真（[实现](../../../raw/current/scripts/components/toadstoolspawner.lua#L155-L160)、[状态变化](../../../raw/current/scripts/components/toadstoolspawner.lua#L69-L75)）；判定是全球性的，不查金丝雀与帽子的距离。
2. **金丝雀积毒**：进鸟笼的金丝雀每游戏段（`SEG_TIME`）吸气一次，`_gaslevel` +1（[StartInhalingGas](../../../raw/current/scripts/prefabs/birds.lua#L159-L163)、[OnInhaleGas](../../../raw/current/scripts/prefabs/birds.lua#L141-L146)）；超过 12 级后概率随等级增大，命中即推 `birdpoisoned`（[TestGasLevel](../../../raw/current/scripts/prefabs/birds.lua#L131-L138)）。
3. **换病鸟**：鸟笼收 `birdpoisoned` → 删原鸟、笼进 SICK 状态、生成 `canary_poisoned`（金丝雀（中毒））（[OnBirdPoisoned](../../../raw/current/scripts/prefabs/birdcage.lua#L457-L461)）。
4. **爆毒**：`canary_poisoned` 的 SG `explode` 状态第 52 帧掉羽毛，并对半径 3 内带 `absorbpoison` 标签的实体推 `poisonburst`（[SGcanarypoisoned](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L268-L270)、[过滤标签](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L3)）——原版唯一持有该标签的就是毒菌蟾蜍帽（[加标签](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L285)）。
5. **吸收**：帽子处于非 0 状态（长着蘑菇；协议上有状态 1／2，原版写入路径只出现 2，见[本体辨析](#toadstoolcap-本体辨析)）时监听 `poisonburst`（[挂接](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L181)），`onabsorbpoison` → `setdark(inst, TUNING.TOTAL_DAY_TIME, inst:IsAsleep())`（一天＝480 秒，[常量](../../../raw/current/scripts/tuning.lua#L19-L20)）：`_dark` 置真、build 换 `toadstool_dark_build`、小地图换暗图标，未睡眠时生成 absorbfx；已暗则把 `darktimer` 剩余时间补到不短于新时长（[setdark](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L69-L91)）。
6. **释放**：`darktimer` 到期 → `setnormal(inst, inst:IsAsleep())`（[定时回调](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L163-L174)）：build 换回 `toadstool_build`，未睡眠时生成 releasefx；`setstate(0)`（砍帽出 Boss 后变洞）与读档后处理走 `instant = true`，**不出 FX**。

暗态的玩法后果：下一次砍帽出的是 `toadstool_dark`（悲惨的毒菌蟾蜍）而非普通 Boss（[onspawntoad](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L122-L133)）——两个 FX 就是这套"毒雾转化"玩法的可见载体。

### toadstool_cap 本体辨析

- `toadstool_cap` 是**固定地物**（世界生成 `toadstool_arena` 布局摆放，tile 约束见 [map/terrain.lua#L101](../../../raw/current/scripts/map/terrain.lua#L101)、[布局](../../../raw/current/scripts/map/static_layouts/toadstool_arena.lua#L114)），不是 Boss 实体：显示名按状态切换（洞／毒菌蟾蜍／悲惨的毒菌蟾蜍，[displaynamefn](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L228-L232)），状态机为 0 洞（`picked`，不监听毒爆）↔ 2 长出中（可砍 3 刀，砍完在原地生成 Boss 并回到 0）；状态 1（`inground`）在 `setstate` 协议里有定义但本文件没有任何写入路径（[setstate](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L176-L220)）。
- Boss 死后帽子经 `respawn`／`respawndark` 计时重长（respawndark 直接以暗态长出，`instant` 不出 FX）；世界侧的重生调度与 `ms_spawntoadstool` 触发在 `toadstoolspawner` 组件里，不属本页展开。

## 典型接入

### 在自定义宿主上复刻染色演出

- 前提与选择：宿主需有 AnimState 且能当引擎父（联网实体最佳——FX 的位置经父子复制）；要的是"宿主闪黑再渐显"的变装演出时复用两注册，换动画需另建注册（`MakeFX` 参数在文件内闭包，不可运行时改）。
- 生效与撤销：主模拟生成并父挂；非专服主机要手动执行染色协议（客户端自动），FX 到时自删并复位染色；若宿主可能先于 FX 消失，`OnRemoveFXEntity` 已判父有效性，无需额外清理。
- 原版依据：[setdark 生成块](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L77-L83)、[染色协议](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L43-L49)。

```lua
local function PlayCapAbsorb(cap)
    local fx = SpawnPrefab("toadstool_cap_absorbfx")
    if fx == nil then return end
    fx.entity:SetParent(cap.entity)
    if TheWorld.ismastersim and not TheNet:IsDedicated() then
        -- 客户端经工厂挂接的 OnEntityReplicated 自动执行同一套协议；
        -- 这里按原版 setdark 的做法给非专服主机补跑本机染色
        cap.AnimState:OverrideMultColour(0, 0, 0, 1)
        fx:DoPeriodicTask(0, function(fx) -- 任务随 FX 自删而止
            if not cap:IsValid() then return end
            local t = fx.AnimState:GetCurrentAnimationTime()
            local T = math.max(fx.AnimState:GetCurrentAnimationLength(), 0.0001)
            local k = 1 - (1 - t / T) * (1 - t / T) -- 黑(0) → 正常(1)
            cap.AnimState:OverrideMultColour(k, k, k, 1)
        end)
    end
end
```

- 调整与验证：FX 删除时其自带 `OnRemoveEntity` 会把宿主染色复位 (1,1,1,1)，无需手动清理；直接生成 `toadstool_cap_releasefx` 则得到金丝雀爆毒外观。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：FX 自删只靠定时，没有 animover。** 主模拟删点是"动画长度＋2 帧"；动画缺失或被打断也按定时收口，壳不会滞留。[工厂尾](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L368-L369)
- **边界：睡眠／读档路径完全不出 FX。** `instant = true` 时 `setdark`／`setnormal` 只切 build 与小地图图标——玩家离屏错过变暗过程，回来只见结果；这是刻意取舍不是漏接。[setdark](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L77-L83)
- **边界：染色挂在宿主上，FX 删了才复位。** 期间若宿主自己改染色或换 build，FX 的周期任务会逐帧覆盖回来；要长期自定义帽子染色就得先停掉 FX 的任务或等它自删。[协议](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L37-L49)
- **边界：releasefx 的外观是金丝雀爆炸，不是蘑菇。** bank／build／动画全取自 canary 资源；按注册名猜"释放=蘑菇动画"会找错资源库。[注册](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L379)
- **边界：`poisonburst` 不是通用毒雾事件。** 全脚本只有 `canary_poisoned` 的爆炸推送它，也只有 `absorbpoison` 标签（原版仅帽子持有）会收到；状态 0（洞）时帽子不监听——Boss 存活期间爆金丝雀对帽子无效。[推送](../../../raw/current/scripts/stategraphs/SGcanarypoisoned.lua#L268-L270)、[监听窗口](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L179-L192)
- **边界：已暗时重复吸收会续时并再放一次 absorbfx。** `darktimer` 取"剩余"与"新时长"的较大值重起，未睡眠时再生成一枚吸收演出——连续爆金丝雀会看到多次吸收动画。[刷新逻辑](../../../raw/current/scripts/prefabs/toadstool_cap.lua#L84-L90)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/toadstool_cap.lua`；按需读取 `stategraphs/SGcanarypoisoned.lua`（EXPLODE_MUST_TAGS 与 explode 状态）、`prefabs/birds.lua`（gas 吸呼与 TestGasLevel）、`prefabs/birdcage.lua`（OnBirdPoisoned）、`components/toadstoolspawner.lua`（重生定时与 IsEmittingGas）、`tuning.lua`（seg_time/total_day_time，TOTAL_DAY_TIME＝480）。全域反查（`grep -rn "toadstool_cap_absorbfx\|toadstool_cap_releasefx"`）：仅命中 toadstool_cap.lua（依赖 L26-L27、生成 L56／L78、注册 L378-L379）——生成点唯一且全在本文件；`poisonburst` 全域仅 SGcanarypoisoned.lua:269 推送、toadstool_cap.lua 监听；`absorbpoison` 标签仅 toadstool_cap.lua:285 持有（另见 SGcanarypoisoned.lua:3 的过滤表）。台账批 21 分诊结论复核属实（releasefx L56、absorbfx L78、MakeFX L340-L375、FX+NOCLICK、persists=false）；台账"触发链（Boss 状态机）"的预期与实际不符——见[触发链](#触发链金丝雀爆毒--帽子变暗--复原)，触发源是金丝雀 SG 与帽子计时器。
- 名称核验：`languages/chinese_s.po` 定向检索——`TOADSTOOL`＝"毒菌蟾蜍"（[L350420-L350422](../../../raw/current/scripts/languages/chinese_s.po#L350420-L350422)）、`TOADSTOOL_CAP` 同名"毒菌蟾蜍"（[L350425-L350427](../../../raw/current/scripts/languages/chinese_s.po#L350425-L350427)）、`TOADSTOOL_CAP_DARK`／`TOADSTOOL_DARK`＝"悲惨的毒菌蟾蜍"（[L350430-L350432](../../../raw/current/scripts/languages/chinese_s.po#L350430-L350432)）、`TOADSTOOL_HOLE`＝"洞"（[L350440-L350442](../../../raw/current/scripts/languages/chinese_s.po#L350440-L350442)）、`CANARY`＝"金丝雀"（[L340085-L340087](../../../raw/current/scripts/languages/chinese_s.po#L340085-L340087)）、`CANARY_POISONED`＝"金丝雀（中毒）"（[L340090-L340092](../../../raw/current/scripts/languages/chinese_s.po#L340090-L340092)）；`TOADSTOOL_CAP_ABSORBFX`／`RELEASEFX` 无 `STRINGS.NAMES` 键。
- 分档说明：`size_class: standard`——2 注册单工厂，但需承载跨 5 文件的触发链、各端本地染色协议与本体辨析；无多形态生命周期，未升 complex。
- 未覆盖：toadstool／toadstool_dark Boss 战与 SG、`toadstoolspawner` 重生调时全貌（TOADSTOOL_RESPAWN_TIME/SPAWN_TIME）、sleepbomb 配方侧的中毒金丝雀用途、动画包实际内容与帧长、canary_poisoned 其余状态；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录（0 error 0 warning 后交付）。
- 第 16 轮补正（2026-11-07）：正文引用的 `map/terrain.lua`、`map/static_layouts/toadstool_arena.lua` 补入 sources 并核验锚点。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/mushtree_tall_webbed_burntfx|生物残骸与植物／菌类次级表现]]（同为宿主文件内的次级表现注册）
- 返回 [[reference/prefabs/fx-status-index|增益、受击、护盾与状态反馈]]（毒菌蟾蜍 Boss 帽子形态）
