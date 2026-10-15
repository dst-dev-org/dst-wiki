---
title: "月亮石充能光束与光柱（positronbeam_back、positronbeam_front、positronpulse）"
created: "2026-11-05"
updated: "2026-11-07"
type: prefab
summary: "月亮石满月充能期间的贴地光环前后层与中央光柱：createbeam/createpulse 双工厂、三 FX 随充能时机整组生成、pulse 两档等级与 FinishFX 收束、king_cane 换动画库。"
aliases: ["positronbeam_back", "positronbeam_front", "positronpulse", "positronbeam", "月亮石光束", "月亮石充能特效", "月亮石光柱"]
tags: [prefab, fx, server-client]
size_class: standard
dst_build: "steam-buildid:26335394"
sources: ["prefabs/positronbeam.lua", "prefabs/moonbase.lua", "tuning.lua", "prefablist.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 月亮石充能光束与光柱（positronbeam_back、positronbeam_front、positronpulse）

## 快速判断

- 职责与适用：`prefabs/positronbeam.lua` 用两个工厂注册 3 个联网 FX，全部只服务月亮石（moonbase，官方名"月亮石"）的满月充能演出——`positronbeam_back`／`positronbeam_front` 是同一 `createbeam` 工厂的贴地光环**后层／前层**（FinalOffset 0／2，仅前层带 SoundEmitter），`positronpulse` 是 `createpulse` 注册的**中央光柱**，带两档等级与参数化循环声。[注册](../../../raw/current/scripts/prefabs/positronbeam.lua#L135)、[工厂](../../../raw/current/scripts/prefabs/positronbeam.lua#L11-L58)、[光柱](../../../raw/current/scripts/prefabs/positronbeam.lua#L91-L133)
- 优先使用：想在自定义设施上复刻"持续型充能光束＋可升降级"的表现时，直接 `SpawnPrefab` 三注册并用宿主侧暴露的 `SetStaff`／`SetLevel`／`FinishFX`／`KillFX` 方法驱动；只取一层光环可单用 back 或 front。[典型接入](#在自定义设施上手动播放充能光束组)
- 重要副作用：纯表现层——无 combat、无伤害、无玩法组件；声音三处：pulse 的 `beam` 参数化循环声（intensity 随等级）＋升级声、front 的收束声 `beam_stop_fail`；三个 FX 的可见状态与等级变化都由月亮石（主模拟）驱动。
- 修改边界：只覆盖 3 个注册的工厂结构、月亮石侧的生成／停止／收尾时机与 king_cane 换库；月亮石本体玩法（workable 修理、trader 塞杖、moonbeastspawner 月兽、staffcoldlightfx 星光点）只在条目内辨析不展开。
- 名称：`MOONBASE`＝"月亮石"（[po](../../../raw/current/scripts/languages/chinese_s.po#L345605-L345607)）；3 个 FX 注册名在简中 `chinese_s.po` 无 `STRINGS.NAMES` 键，标题与正文中文为已核验效果描述，非官方译名。

## 注册与构造

两个工厂共用同一 assets（`anim/moonbase_fx.zip`＋`anim/moonbase_fx_king_cane.zip`）与 `SetStaff` 换库函数（[定义](../../../raw/current/scripts/prefabs/positronbeam.lua#L7-L9)）：staffname 为 `king_cane` 时 bank 换 `lunar_fx_king_cane`，否则 `lunar_fx`。

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `positronbeam_back` | `createbeam("back", 0)`：Transform／AnimState／Network（**无 SoundEmitter**）→ bank `lunar_fx`、build `moonbase_fx`、播 `lunar_back_pre` 接 `lunar_back_loop` 循环、bloom `shaders/anim.ksh`、FinalOffset 0 → 标签 `FX` → SetPristine → 客户端早退 → 主模拟：`persists=false`、暴露 `SetStaff`／`KillFX` | 后层光环：渲染在月亮石身后；静音层，收束也只播动画不出声（[注册](../../../raw/current/scripts/prefabs/positronbeam.lua#L57)、[主模拟段](../../../raw/current/scripts/prefabs/positronbeam.lua#L49-L52)） |
| `positronbeam_front` | `createbeam("front", 2)`：同上，但**多挂 SoundEmitter**、FinalOffset 2 | 前层光环：`KillFX` 收束时播 `dontstarve/common/together/moonbase/beam_stop_fail`（[KillFX](../../../raw/current/scripts/prefabs/positronbeam.lua#L12-L22)、[SoundEmitter 条件](../../../raw/current/scripts/prefabs/positronbeam.lua#L29-L31)） |
| `positronpulse` | `createpulse(3)`：Transform／AnimState／SoundEmitter／Network → bank／build 同上、播 `lunar_2_loop` 循环、FinalOffset 3 → 标签 `FX` → **`Hide()`（在 SetPristine 前，全端隐藏）** → 播 `beam` 循环声并置 intensity 0 → SetPristine → 客户端早退 → 主模拟：0 秒后 `InitFX`（播升级声）、`persists=false`、暴露 `SetStaff`／`SetLevel`／`FinishFX`，`KillFX = inst.Remove` | 中央光柱：出生即隐身，等 `SetLevel(2/3)` 才可见；收尾双路径见[下节](#月亮石侧的生成与收尾时机)（[注册](../../../raw/current/scripts/prefabs/positronbeam.lua#L132)、[隐藏与声音](../../../raw/current/scripts/prefabs/positronbeam.lua#L108-L111)、[主模拟段](../../../raw/current/scripts/prefabs/positronbeam.lua#L119-L127)） |

主模拟侧方法语义（仅 master 实体上有，[挂接点](../../../raw/current/scripts/prefabs/positronbeam.lua#L51-L52)、[L123-L127](../../../raw/current/scripts/prefabs/positronbeam.lua#L123-L127)）：

| 方法 | 行为 | 证据 |
|---|---|---|
| `SetStaff(name)` | 换 bank：`king_cane` → `lunar_fx_king_cane`，其余 → `lunar_fx`（三注册共用同一函数） | [SetStaff](../../../raw/current/scripts/prefabs/positronbeam.lua#L7-L9) |
| `SetLevel(level)`（仅 pulse） | `level == nil 或 < 2` → `Hide`＋intensity 0；否则播 `lunar_<min(level,3)>_loop`、`Show`、intensity 0.6/0.9、播 `beam_level_up` | [SetLevel](../../../raw/current/scripts/prefabs/positronbeam.lua#L60-L75) |
| `FinishFX()`（仅 pulse） | 杀 `beam` 声 → 播 `beam_stop` → 播 `lunar_full_pst` → 动画长度＋2 帧后 `Remove`（`_finished` 防重入） | [FinishFX](../../../raw/current/scripts/prefabs/positronbeam.lua#L77-L85) |
| `KillFX`（beam）/ `inst.Remove`（pulse） | beam：置 `_iskilled` 防重入 → front 播 `beam_stop_fail` → 播 `lunar_<layer>_pst` → `animover` 删＋长度＋1 帧兜底；pulse：立即 `Remove` | [beam KillFX](../../../raw/current/scripts/prefabs/positronbeam.lua#L12-L22)、[pulse](../../../raw/current/scripts/prefabs/positronbeam.lua#L127) |

## 关联机制导航

### 月亮石侧的生成与收尾时机

生成点唯一：月亮石工厂文件 `prefabs/moonbase.lua` 的 `StartFX` 一次生成整组（pulse [L90](../../../raw/current/scripts/prefabs/moonbase.lua#L90)、front [L97](../../../raw/current/scripts/prefabs/moonbase.lua#L97)、back [L104](../../../raw/current/scripts/prefabs/moonbase.lua#L104)），都 `SetPosition` 到月亮石本体位置并 `SetStaff(当前杖名)`；生成前先把已有的三个 `Remove`（三个 FX 总是成组刷新）。触发链（[ToggleMoonCharge](../../../raw/current/scripts/prefabs/moonbase.lua#L195-L304)、[OnTimerDone](../../../raw/current/scripts/prefabs/moonbase.lua#L435-L497)、[OnFullmoon](../../../raw/current/scripts/prefabs/moonbase.lua#L608-L616)）：

| 时机 | 条件与动作 | 证据 |
|---|---|---|
| 满月开始 | `WatchWorldState("isfullmoon")` → 起 `fullmoonstartdelay` 定时器（`MOONBASE_CHARGE_DELAY`＝10 秒，注释： fade to night 缓冲） | [OnFullmoon](../../../raw/current/scripts/prefabs/moonbase.lua#L608-L616)、[调值](../../../raw/current/scripts/tuning.lua#L2144-L2148) |
| 充能预备 | 塞入杖（`yellowstaff` 或带 corruption 的 `king_cane`）＋已修好＋可摘取＋满月 → 起 `moonchargepre`（取 fullmoonstartdelay 剩余与 1 秒的较大值） | [ToggleMoonCharge](../../../raw/current/scripts/prefabs/moonbase.lua#L257-L276) |
| StartFX | `moonchargepre` 到时 → 起 `mooncharge`（`seg_time*2-10.1`＝49.9 秒）与 `mooncharge2`（18.7 秒）→ `moonbeastspawner:Start()` → **StartFX** → 音乐 1 级 | [OnTimerDone](../../../raw/current/scripts/prefabs/moonbase.lua#L436-L446) |
| 升级 2 | `mooncharge2` 到时 → `SetLevel(2)`（光柱显形）→ 加 `intense` 标签 → 起 `mooncharge3`（18.7 秒） | [mooncharge2](../../../raw/current/scripts/prefabs/moonbase.lua#L447-L453) |
| 升级 3 | `mooncharge3` 到时 → `SetLevel(3)` → 音乐 2 级 | [mooncharge3](../../../raw/current/scripts/prefabs/moonbase.lua#L454-L458) |
| 变形完成 | `mooncharge` 到时且杖仍满足 → `yellowstaff` 换成 `opalstaff`（唤月者魔杖）→ `ShowColdStar`（生成 `staffcoldlightfx` 星光点）→ **pulse `FinishFX`**；带 corruption 的 king_cane 则生成 `shadow_despawn` 并解除腐蚀（此分支 pulse 不走 FinishFX） | [mooncharge](../../../raw/current/scripts/prefabs/moonbase.lua#L459-L496)、[FinishFX 调用](../../../raw/current/scripts/prefabs/moonbase.lua#L485-L488) |
| 中断收尾 | 变形完成后（杖已变 `opalstaff`，不再满足充能条件）或任何中断（取杖／月相结束／受锤损坏）→ `ToggleMoonCharge` 走停止分支 → `StopFX` | [ToggleMoonCharge](../../../raw/current/scripts/prefabs/moonbase.lua#L286-L301) |

`StopFX`（[实现](../../../raw/current/scripts/prefabs/moonbase.lua#L49-L72)）：pulse 走 `KillFX`（＝立即 Remove）；front／back 走 beam 版 `KillFX`（播 pst 动画自删，front 伴收束声），随后 9 帧关月亮石的 Light；月亮石本体 `OnRemoveEntity` 则对三个 FX 直接 `Remove`（[实现](../../../raw/current/scripts/prefabs/moonbase.lua#L119-L132)）。

### 端模型与父子关系

- 三注册都是**联网实体**（Network 组件＋SetPristine 分界）：工厂里 pre／loop 动画与 pulse 的隐藏、循环声在所有机器的构造期各播各的；主模拟侧的 `SetLevel`／`FinishFX`／`KillFX` 只改主模拟实体，画面经 AnimState／SoundEmitter 随网络复制。专服同样生成这三个壳（无 IsDedicated 门控，persists=false 不入存档）。
- **没有父子关系**：三个 FX 都是世界位置实体（SpawnPrefab＋SetPosition），不 AddChild 也不 `entity:SetParent`；它们不跟随月亮石（月亮石是固定地物），删除靠月亮石显式 Remove，不是引擎级联。[生成块](../../../raw/current/scripts/prefabs/moonbase.lua#L82-L117)
- `intense` 标签与 FX 同步加／移除（升级 2 加、停止删），消费方在 AOE 与交互过滤（aoeutil／componentactions 等），不属于本页展开。

### 关联 Prefab

| 注册名 | 关系 | 证据 |
|---|---|---|
| `moonbase` | 宿主设施（月亮石）：唯一生成方；被锤受损／修理／塞杖都经 `ToggleMoonCharge` 重新评估 FX | [注册](../../../raw/current/scripts/prefabs/moonbase.lua#L725) |
| `staffcoldlightfx` | 塞入 `opalstaff`（唤月者魔杖）后 `ShowColdStar` 生成、取杖时移除的星光点（月亮石的子体） | [L161-L169](../../../raw/current/scripts/prefabs/moonbase.lua#L161-L169) |
| `shadow_despawn` | 腐蚀 king_cane 充能完成时在月亮石上方生成的消散演出（非本页覆盖） | [L465](../../../raw/current/scripts/prefabs/moonbase.lua#L464-L466) |
| `yellowstaff`／`opalstaff` | 充能原料与产物：唤星者魔杖（[po](../../../raw/current/scripts/languages/chinese_s.po#L352770-L352772)）→ 唤月者魔杖（[po](../../../raw/current/scripts/languages/chinese_s.po#L346670-L346672)）；`king_cane`（国王魔杖，[po](../../../raw/current/scripts/languages/chinese_s.po#L344210-L344212)）可替换塞入并切换 FX 动画库 | [KEY_STAFF／MORPHED_STAFF](../../../raw/current/scripts/prefabs/moonbase.lua#L20-L21) |

## 典型接入

### 在自定义设施上手动播放充能光束组

- 前提与选择：需要"底座光环＋可升降级光柱"的持续表现时复用三注册；`SetLevel`／`FinishFX` 只在主模拟实体上存在，调用方须在服务端逻辑里驱动。
- 生效与撤销：三 FX 位置一次性（生成时落点），不跟随宿主——宿主要移动就自己重新落位或自行管理子挂；收尾必须调用 `FinishFX` 或 `KillFX`，否则 beam 会一直循环（FX 自身无寿命上限，pulse 隐藏但仍存在）。
- 原版依据：[StartFX 的成组生成](../../../raw/current/scripts/prefabs/moonbase.lua#L82-L117)、[SetLevel 语义](../../../raw/current/scripts/prefabs/positronbeam.lua#L60-L75)。

```lua
-- 主模拟：返回整组句柄，避免三个无寿命实体丢失后无法收尾
local function PlayMoonChargeSet(x, y, z)
    local back = SpawnPrefab("positronbeam_back")
    back.Transform:SetPosition(x, y, z)
    local front = SpawnPrefab("positronbeam_front")
    front.Transform:SetPosition(x, y, z)
    local pulse = SpawnPrefab("positronpulse")
    pulse.Transform:SetPosition(x, y, z)
    pulse:SetStaff("yellowstaff") -- 换成 "king_cane" 则换动画库
    pulse:SetLevel(2)             -- 显形并播 lunar_2_loop；再 SetLevel(3) 升档
    return { back = back, front = front, pulse = pulse }
end

-- 收尾：光环走 beam KillFX（播 pst 自删），光柱走 FinishFX（pst 后自删）
local fx = PlayMoonChargeSet(x, y, z)
-- ... 演出结束：
fx.back:KillFX()
fx.front:KillFX()
fx.pulse:FinishFX()
-- 中断立即清：fx.pulse:Remove()，光环仍用 KillFX 保留退场动画
```

- 调整与验证：替换点是落点、杖名与等级（原版只走 2／3；`SetLevel(1)` 及以下等于隐藏）。三实体均无固有寿命上限，示例保留整组句柄并给出 `KillFX`／`FinishFX` 收尾；若只示意生成而不管理句柄，实体将一直存在（pulse 隐藏但仍占实体）。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：`StopFX` 的内层条件有笔误。** 第一个内层判断写的是 `if inst._fxback ~= nil` 却调用 `inst._fxfront:KillFX()`（[L55-L58](../../../raw/current/scripts/prefabs/moonbase.lua#L55-L58)）——原版两者总在 StartFX 成对生成所以无碍；若自行只生成一侧，front 可能漏杀、back 单独存在时还会对 nil 的 front 报错。
- **边界：`beam_stop_fail` 不是"失败专用"声。** 充能**成功变形**后的 StopFX 同样让 front 层播这声音（FinishFX 只管 pulse）；它实际是 front 层收束的固定伴音。[StopFX](../../../raw/current/scripts/prefabs/moonbase.lua#L54-L62)
- **边界：pulse 没有 level 1 播放分支。** `SetLevel` 对 `nil` 或 `< 2` 直接 `Hide`、不拼动画名，并不会播 `lunar_1_loop`；即使动画资源存在，额外直接播动画或改方法也属自定义行为，不能说 `SetLevel` 能触达 level 1。[SetLevel](../../../raw/current/scripts/prefabs/positronbeam.lua#L63-L74)
- **边界：front／back 是渲染前后层，不是两个独立特效状态。** FinalOffset 0／2 决定它们相对月亮石的绘制层次；删掉任意一层不影响另一层，但原版逻辑把它们当一个整体成组管理。
- **边界：三 FX 不随月亮石存档。** `persists=false` 且无父子登记——读档后由月亮石 `init` → `ToggleMoonCharge` 重新评估充能状态并再次 StartFX 重建。[init](../../../raw/current/scripts/prefabs/moonbase.lua#L618-L622)
- **误区：这三束光是"月岛环境特效"。** 生成条件严格绑定月亮石的满月充能状态机（满月＋修好＋特定杖）；离开这套条件不会有任何生成点。[生成链](#月亮石侧的生成与收尾时机)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/positronbeam.lua` 与 `prefabs/moonbase.lua` 全文；按需读取 `tuning.lua` 的 `MOONBASE_*` 段（L2144-L2148）与 `seg_time`/`total_day_time` 定义（L19-L20）。全域反查（`grep -rn "positronbeam\|positronpulse" --include=*.lua`）：命中 positronbeam.lua 注册 2 行（L57、L132）、moonbase.lua 依赖 3 行（L13-L15）＋生成 3 行（L90、L97、L104）、prefablist.lua:950（`PREFABFILES` 的**文件名** `positronbeam`，非注册名）——生成点唯一且全在 moonbase.StartFX；`positronbeam_back`／`positronbeam_front`／`positronpulse` 三注册名与工厂返回一致。`positronbeam.lua` 内无清单外 FX 注册。台账批 21 分诊结论复核属实（工厂 L57 一带、pulse L132、moonbase 生成 L90/97/104）。
- 名称核验：`languages/chinese_s.po` 定向检索——`MOONBASE`＝"月亮石"（msgid "Moon Stone"，[L345605-L345607](../../../raw/current/scripts/languages/chinese_s.po#L345605-L345607)）；`YELLOWSTAFF`＝"唤星者魔杖"（[L352770-L352772](../../../raw/current/scripts/languages/chinese_s.po#L352770-L352772)）、`OPALSTAFF`＝"唤月者魔杖"（[L346670-L346672](../../../raw/current/scripts/languages/chinese_s.po#L346670-L346672)）、`KING_CANE`＝"国王魔杖"（[L344210-L344212](../../../raw/current/scripts/languages/chinese_s.po#L344210-L344212)）；`POSITRONBEAM`／`POSITRONPULSE` 相关键均无 `STRINGS.NAMES` 命中。
- 分档说明：`size_class: standard`——3 注册但共用两工厂、需承载月亮石满月充能的完整时机链（满月→预备→生成→两档升级→变形/中断双收尾）与主模拟方法协议；无多形态切换，未升 complex。
- 未覆盖：月亮石本体修建／修理／月兽防御玩法全貌（moonbeastspawner、moonhound/moonpig）、`staffcoldlightfx` 与 `shadow_despawn` 工厂内部、`intense` 标签全部消费方、动画包实际内容与帧长、`beam` 循环声的跨端参数同步细节；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录（0 error 0 warning 后交付）。
- 第 16 轮补正（2026-11-07）：`SetLevel(1)` 改为“没有 level 1 播放分支、nil/<2 直接 Hide”，撤回“资源存在、模组可触达”；示例改为返回整组句柄并给出 `KillFX`／`FinishFX` 收尾，删除无法清理的裸局部变量写法。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/moon_device_pillar|月装置支柱、顶盖与裂纹层]]（同属月亮设施的阶段表现族）
- [[reference/prefabs/cavelight_small|光源实体（洞穴、切斯特、读书、出生与中庭）]]（同为联网光效实体的构造对照）
- 返回 [[reference/prefabs/fx-structure-index|建筑、装置与遗迹演出]]（月亮石）
