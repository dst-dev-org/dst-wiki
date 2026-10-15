---
title: "图勒皇冠护盾特效（forcefieldfx）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "铥矿皇冠（ruinshat）proc 触发时包裹佩戴者的护盾层：联网动画＋声音循环＋客户端本地插值的灯光渐亮渐暗（net_tinybyte 帧值驱动），由帽子的 proc/unproc 链生成与 kill_fx 收尾；与幽灵灵药玩家护盾（通用表注册）的辨析互链。"
aliases: ["forcefieldfx", "图勒皇冠护盾特效", "铥矿皇冠护盾", "forcefield 护盾", "护盾帽特效"]
tags: [prefab, fx, equipment]
dst_build: "steam-buildid:26335394"
sources: ["prefabs/forcefieldfx.lua", "prefabs/hats.lua", "componentutil.lua", "tuning.lua", "mainfunctions.lua", "prefabs/ghostly_elixirs.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 图勒皇冠护盾特效（forcefieldfx）

## 快速判断

- 职责与适用：铥矿皇冠（`ruinshat`）proc 期间包裹佩戴者的护盾表现：`open`→`idle_loop` 动画＋`forcefield_LP` 声音循环＋灯光从 0 渐亮到半径 3（proc 中受击换 `hit` 动画）；unproc 时 `close` 收尾＋灯光渐暗。工厂自身是纯表现实体，护盾的免伤／理智转移全在帽子的 armor 组件上。
- 优先使用：查"图勒皇冠开盾表现"用本页；玩家饮盾药（幽灵灵药）的护盾是另一注册 `elixir_player_forcefield`，由 [[reference/prefabs/ghostlyelixir_slowregen_fx|幽灵灵药环绕与滴落特效]] 页承载，见[辨析](#与-elixirplayerforcefield-的辨析)。
- 重要副作用：FX **无自删**，唯一收口是宿主帽子调用的实例方法 `kill_fx`（0.6 秒后移除）；灯亮灭是 netvar＋本地插值的协作（主模拟只推开关，各端自行按帧插值半径）。
- 修改边界：只覆盖 forcefieldfx 的工厂、唯一生成链与清理；皇冠 proc 的运气系统全貌、armor 组件机制、`MakeHat` 通用帽子骨架不展开。
- 名称：宿主官方名 `STRINGS.NAMES.RUINSHAT`＝"铥矿皇冠"（[po](../../../raw/current/scripts/languages/chinese_s.po#L348814-L348817)）；`FORCEFIELDFX` 无 `STRINGS.NAMES.*` 键，标题"图勒皇冠护盾特效"为已核验效果描述，**非官方译名**。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `forcefieldfx` | Transform／AnimState／SoundEmitter／Light／Network；bank／build `forcefield`、播 `open` 后接 `idle_loop` 循环；`forcefield_LP` 循环音；Light 强度 0.9／衰减 0.9／白色、半径 0 起步、`EnableClientModulation(true)`；netvar `_lightframe`（net_tinybyte，dirty 事件 `lightdirty`）与 `_islighton`（net_bool）→ Pristine → 主模拟挂 `kill_fx` 方法 | Pristine 后**主模拟与客户端都立即跑一次** `OnLightDirty`（[L73](../../../raw/current/scripts/prefabs/forcefieldfx.lua#L71-L79)），非主模拟端再监听 `lightdirty`；`_islighton` 初始 true（开盾即亮）。[工厂](../../../raw/current/scripts/prefabs/forcefieldfx.lua#L43-L84)、[netvar](../../../raw/current/scripts/prefabs/forcefieldfx.lua#L66-L69)、[注册](../../../raw/current/scripts/prefabs/forcefieldfx.lua#L86) |

灯光渐变协议（[OnUpdateLight](../../../raw/current/scripts/prefabs/forcefieldfx.lua#L8-L26)、[OnLightDirty](../../../raw/current/scripts/prefabs/forcefieldfx.lua#L28-L33)）：`lightdirty` 触发各端起一个每帧（FRAMES）本地任务，按 `_islighton` 把帧值向 0 或上限 6 推进（写 `set_local`），半径＝`3×帧值/6`；到端即取消任务。主模拟只在 `kill_fx` 时写权威值（`_islighton=false`＋重置 `_lightframe` 推 dirty）——**客户端看不到"主模拟逐帧推帧值"，亮度是各端本地插值**。

## 关联机制导航

### 唯一生成链：铥矿皇冠 proc

| 环节 | 行为与条件 | 证据 |
|---|---|---|
| 触发尝试 `tryproc` | 皇冠监听佩戴者 `attacked` 事件（`onattach` 挂监听）；`ShouldProcOnAttackedOrBlocked`（受击或格挡门禁）＋`TryLuckRoll(0.33, LuckFormulas.RuinsHatProc)`（基础 33%、运气公式 `CommonChanceLuckAdditive(.1)` 微调）才进 proc；`_task` 非空（冷却／已 proc）则跳过 | [tryproc](../../../raw/current/scripts/prefabs/hats.lua#L478-L484)、[门禁](../../../raw/current/scripts/componentutil.lua#L114)、[TryLuckRoll](../../../raw/current/scripts/componentutil.lua#L2237-L2253)、[公式](../../../raw/current/scripts/componentutil.lua#L2164-L2164) |
| proc `ruinshat_proc` | 帽子加 `forcefield` 标签；已有 FX 先 kill；`SpawnPrefab("forcefieldfx")` → 引擎 `SetParent(佩戴者.entity)` → `SetPosition(0, 0.2, 0)`（挂腰高度）；监听帽子 `armordamaged` 让 FX 重播 `hit`＋`idle_loop`；armor 吸收置满额（FULL_ABSORPTION=1）并挂 `ontakedamage` 把伤害按 5% 转移为佩戴者理智损失；4 秒（`ARMOR_RUINSHAT_DURATION`）后 unproc | [proc](../../../raw/current/scripts/prefabs/hats.lua#L455-L476)、[满额](../../../raw/current/scripts/tuning.lua#L1817-L1817)、[数值](../../../raw/current/scripts/tuning.lua#L1852-L1857) |
| unproc `ruinshat_unproc` | 摘标签、`inst._fx:kill_fx()`＋置 nil、摘 `armordamaged` 监听、恢复常规吸收（0.9）、排 5 秒（`ARMOR_RUINSHAT_COOLDOWN`）冷却任务 | [unproc](../../../raw/current/scripts/prefabs/hats.lua#L436-L453)、[冷却](../../../raw/current/scripts/prefabs/hats.lua#L432-L434) |
| 收尾兜底 | 皇冠 `OnRemoveEntity`（帽子被销毁）kill FX；`ondetach`（卸下）走 `ruinshat_unproc` 并清引用 | [onremove](../../../raw/current/scripts/prefabs/hats.lua#L505-L510)、[attach/detach](../../../raw/current/scripts/prefabs/hats.lua#L534-L552) |

全库反查结论：`SpawnPrefab("forcefieldfx")` **仅 hats.lua:460 一处**；hats.lua:6794 的 `prefabs = { "forcefieldfx" }`（MakeHat("ruins") 依赖声明，[分支](../../../raw/current/scripts/prefabs/hats.lua#L6792-L6794)；注册于 [MakeHat("ruins")](../../../raw/current/scripts/prefabs/hats.lua#L7874)，工厂内拼名 `ruinshat`）与 prefablist.lua:459 条目只是声明，不构成生成点。

### 与 elixir_player_forcefield 的辨析

| 对比项 | `forcefieldfx`（本页） | `elixir_player_forcefield` |
|---|---|---|
| 注册方式 | 独立工厂文件，自有 netvar 灯光协议 | fx.lua **通用表条目**：bank／build `abigail_shield`、`player_shield` 动画（[条目](../../../raw/current/scripts/fx.lua#L3889-L3892)） |
| 生成者 | 铥矿皇冠 proc（装备被动，可重复触发） | 玩家饮盾药时由灵药 buff 生成（[ghostly_elixirs.lua:14](../../../raw/current/scripts/prefabs/ghostly_elixirs.lua#L14-L15)），挂法为 `AddChild` |
| 清理 | 帽子 `kill_fx`（close 动画＋灯暗＋0.6 秒） | 通用表页承载的 buff 链清理 |
| 主要页 | 本页 | [[reference/prefabs/ghostlyelixir_slowregen_fx|幽灵灵药环绕与滴落特效]] |

两者都是"玩家身上护盾层"，但注册、工厂协议与宿主链完全不同，互不复用。

## 典型接入

### 复刻"proc 式护盾表现＋宿主转移免伤"

- 前提与选择：想给装备加"概率开盾"表现，把表现（本 FX 模式）与数值（armor `SetAbsorption`＋`ontakedamage` 转移）分开挂，用帽子上的单槽 `_fx` 引用与 `_task` 冷却任务管理。
- 生效与撤销：FX 引擎父挂佩戴者并偏移 (0, 0.2, 0)；撤场必须走 `kill_fx`（有 `close` 收尾动画与灯暗过渡），直接 `Remove` 会闪断。
- 原版依据：[proc](../../../raw/current/scripts/prefabs/hats.lua#L455-L476)、[kill_fx](../../../raw/current/scripts/prefabs/forcefieldfx.lua#L35-L41)。

```lua
local function SpawnShieldFx(owner)
    local fx = SpawnPrefab("forcefieldfx")
    fx.entity:SetParent(owner.entity)
    fx.Transform:SetPosition(0, 0.2, 0)
    owner._fx = fx
end

local function KillShieldFx(owner)
    if owner._fx ~= nil then
        owner._fx:kill_fx()   -- close 动画＋灯光渐暗，0.6 秒后自动 Remove
        owner._fx = nil
    end
end
```

- 调整与验证：替换点为注册名与偏移量；`kill_fx` 是主模拟侧实例方法（客户端无），撤场逻辑须只在服务端执行。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：FX 不承担免伤。** `forcefield` 期间 100% 吸收与理智转移都在帽子的 armor 组件回调上；FX 只有动画／声音／灯光，砸掉 FX 不影响伤害结算。[proc 数值段](../../../raw/current/scripts/prefabs/hats.lua#L465-L470)
- **边界：`kill_fx` 只在主模拟存在。** 客户端实体没有该方法；跨端调用会空引用——撤场回调要挂服务端逻辑。[方法挂载](../../../raw/current/scripts/prefabs/forcefieldfx.lua#L81-L83)
- **边界：灯光亮度是各端本地插值。** netvar 只传开关（`_islighton`）与 dirty 事件，帧值推进与半径换算在各端本地任务里完成；主模拟不逐帧同步亮度——改动协议时两端都要动。[协议](../../../raw/current/scripts/prefabs/forcefieldfx.lua#L8-L33)
- **边界：工厂未置 `persists=false`，但不会入存档。** 存档循环节跳过带引擎父级的实体（[mainfunctions.lua:1114](../../../raw/current/scripts/mainfunctions.lua#L1114)），FX 生成后立即父挂佩戴者，因此不产生存档残留；这与本站多数 FX 显式 `persists=false` 的写法不同。
- **边界：proc 期间再 proc 先杀旧盾。** `ruinshat_proc` 开头 `_fx ~= nil` 时先 `kill_fx` 再重建——不会叠两层盾；同一时间最多一层。[重入处理](../../../raw/current/scripts/prefabs/hats.lua#L457-L459)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/forcefieldfx.lua` 全文（工厂、灯光任务、kill_fx）；按需读取 `prefabs/hats.lua` 的 ruins 段全部函数（fxanim／oncooldown／unproc／proc／tryproc／onattach／ondetach／onremove）与 MakeHat 命名规则、`componentutil.lua` 的 ShouldProcOnAttackedOrBlocked／TryLuckRoll／LuckFormulas.RuinsHatProc、`tuning.lua` 的 ARMOR_RUINSHAT_* 与 FULL_ABSORPTION、`prefabs/ghostly_elixirs.lua` 的玩家护盾生成段、`prefabs/fx.lua` 的 elixir 条目、`mainfunctions.lua` 存档循环。对 `forcefieldfx` 以 `grep -rn "forcefieldfx" --include=*.lua` 在 `raw/current/scripts` 全域反查：命中 forcefieldfx.lua（netvar 字符串×2、注册×1）、hats.lua:460（生成）、hats.lua:6794（依赖声明）、prefablist.lua:459（清单）——**生成点唯一**；另以 `grep -rn "\"forcefield\"" ` 核对 proc 标签写入点仅 hats.lua:456。
- 名称核验：`languages/chinese_s.po` 定向检索——`STRINGS.NAMES.RUINSHAT`＝"铥矿皇冠"（[L348814-L348817](../../../raw/current/scripts/languages/chinese_s.po#L348814-L348817)）；`FORCEFIELD`／`FORCEFIELDFX` 无 `STRINGS.NAMES.*` 键，标题描述为非官方译名。
- 分档说明：`size_class: standard`——单注册但承载完整 proc 链（尝试/触发/退场/冷却）与 netvar 灯光协议、通用表注册辨析，simple 档（120 行）无法同时承载工厂协议、宿主链与验证记录。
- 未覆盖：`ShouldProcOnAttackedOrBlocked` 的格挡分支细节、运气系统的 GetEntityLuckChance 全公式、armor 组件内部机制、`MakeHat` 骨架其余帽子、`forcefield` 动画资源内容。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录（0 error 0 warning 后入库）。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/ghostlyelixir_slowregen_fx|幽灵灵药环绕与滴落特效]]（`elixir_player_forcefield` 的主要页；同为玩家护盾表现，工厂协议不同）
- [[reference/prefabs/tophat_shadow_fx|魔术师高礼帽影子特效]]（同为帽子驱动的跟随层家族）
- [[reference/prefabs/armor_lunarplant_glow_fx|装备与帽饰跟随光效]]（装备佩戴态 FX 族索引页）
- 返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]（装备／防御）
