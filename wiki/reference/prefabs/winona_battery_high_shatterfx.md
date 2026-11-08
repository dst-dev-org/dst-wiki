---
title: "薇诺娜宝石发电机设备特效族（winona_battery_high_shatterfx 等 5 注册）"
created: "2026-11-05"
updated: "2026-11-08"
type: prefab
summary: "薇诺娜工程设备外围的 5 个 FX 注册：宝石发电机的碎宝石动画与纯粹辉煌宝石槽跟随层（由电池结构／便携物品的同一组宝石函数生成）、电线火花（4 台带电路设备的连线/拆除共用、不走通用代理工厂）、聚光灯灯头（本体隐藏发光层由独立联网实体承载）与传送伞持握层（装备时按帧区间分工的两层换装覆盖）。"
aliases: ["winona_battery_high_shatterfx", "winona_battery_sparks", "winona_spotlight_head", "winona_telebrella_swap_fx", "purebrilliance_symbol_fx", "宝石发电机碎裂特效", "薇诺娜电线火花", "聚光灯灯头", "传送伞持握层", "纯粹辉煌符号特效", "薇诺娜的宝石发电机", "薇诺娜的聚光灯", "传送伞"]
tags: [prefab, fx, server-client]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/winona_battery_high.lua", "prefabs/winona_battery_sparks.lua", "prefabs/winona_spotlight.lua", "prefabs/winona_telebrella.lua", "prefabs/purebrilliance.lua", "prefabs/winona_teleport_pad.lua", "prefabs/winona_catapult.lua", "prefabs/winona_remote.lua", "prefabs/winona_storage_robot.lua", "prefabs/alterguardianhatshard.lua", "mainfunctions.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 薇诺娜宝石发电机设备特效族（winona_battery_high_shatterfx 等 5 注册）

## 快速判断

- 职责与适用：5 个注册分三类载体——①`winona_battery_high_shatterfx`（宝石碎裂动画，电池充放电／拆卸时表现宝石损失）、②`purebrilliance_symbol_fx`（纯粹辉煌宝石的槽位跟随层，电池生成）、③`winona_spotlight_head`（聚光灯灯头：本体隐藏发光层、独立联网实体承载灯罩转向与光柱）、④`winona_telebrella_swap_fx`（传送伞持握层：装备时两层按帧区间覆盖持握符号）、⑤`winona_battery_sparks`（电线火花：4 台带电路设备的连线与拆除共用爆点）。
- 优先使用：查"发电机掉宝石／宝石碎裂"用 shatterfx 条目；查"电线冒火花"用 sparks 8 处生成表；查"灯头／光柱跟随"用 spotlight_head；查"伞面 LED 持握层"用 swap_fx；查"电池上的纯粹辉煌光点"用 symbol_fx。
- 重要副作用：除 spotlight_head 外全部 `persists=false`；`winona_battery_sparks` 工厂**明确不走通用代理工厂**（[注释](../../../raw/current/scripts/prefabs/winona_battery_sparks.lua#L6-L7)"needs to be synced to other entities' anims"——要跟宿主 `wire` 符号的动画帧）；shatterfx 与 symbol_fx 无统一回收方法，清理责任分散在宿主宝石函数。
- 修改边界：只覆盖 5 注册的工厂、生成点、父子／跟随与清理；发电机本体充放电玩法、聚光灯照明／加热逻辑、传送伞传送玩法、投石机与传送板本体不展开（充放电仅记到能解释 shatter 触发的深度）。
- 名称：官方名 `WINONA_BATTERY_HIGH`＝"薇诺娜的宝石发电机"（[po](../../../raw/current/scripts/languages/chinese_s.po#L352049-L352052)）、`WINONA_SPOTLIGHT`＝"薇诺娜的聚光灯"（[po](../../../raw/current/scripts/languages/chinese_s.po#L352094-L352097)）、`WINONA_TELEBRELLA`＝"传送伞"（[po](../../../raw/current/scripts/languages/chinese_s.po#L352104-L352107)）、`PUREBRILLIANCE`＝"纯粹辉煌"（[po](../../../raw/current/scripts/languages/chinese_s.po#L347384-L347387)）；5 个 FX 注册名均无 `STRINGS.NAMES.*` 键，标题中文为已核验效果描述，**非官方译名**。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `winona_battery_high_shatterfx` | Transform／AnimState／Network；bank／build `gems`、默认播 `redgem_shatter`；标签 `FX`＋`NOCLICK`；主模拟 `persists=false`＋`animover` 自删 | 无 SoundEmitter（碎裂音由宿主播）；生成后 `LoseGem` 按 `gemname.."_shatter"` 重播对应动画（如 `bluegem_shatter`），并二选一定位：跟随宝石槽符号或落到世界坐标。[工厂](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L1439-L1463)、[LoseGem](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L149-L162)、[注册](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L1578-L1581) |
| `purebrilliance_symbol_fx` | Transform／AnimState／Follower／Network；bank／build `purebrilliance`、`idle_sparkle` 循环、泛光 shader＋符号提亮；标签 `FX`＋`NOCLICK`；主模拟随机起始帧、`persists=false` | 纯粹辉煌物品文件内的**符号跟随专用工厂**（[注释](../../../raw/current/scripts/prefabs/purebrilliance.lua#L101)"Used as symbol follower by winona_battery_high"）；无自删，由电池 `UnsetGem` 显式移除。同文件另有物品本体 `purebrilliance`（本页不覆盖）。[工厂](../../../raw/current/scripts/prefabs/purebrilliance.lua#L102-L126)、[注册](../../../raw/current/scripts/prefabs/purebrilliance.lua#L128-L129) |
| `winona_spotlight_head` | Transform／AnimState／Network；`EightFaced`；bank／build `winona_spotlight`、播 `idle`、Hide `leg`／`ground_shadow`／`wire`、`FinalOffset(1)`、初始 `SetHeadTilt(1,false)`；非专用端自建本地光柱子体（`light_shimmer` 循环、泛光、LightOverride 0.7、跟随自身 `light_shaft_follow` 符号）；主模拟加 `colouradder` | 灯罩／光柱层独立成实体：本体 `fn` 把自己的 `light`／`light_tilt*`／`light_shaft*` 层全部 Hide，发光部分都由灯头承载；`SetHeated` 方法热浪换 `light_shimmer_heat`。`persists` 未置 false（随本体存档语义，见清理）。[工厂](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L1228-L1269)、[光柱子体](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L1184-L1204)、[本体藏层](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L1056-L1061)、[注册](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L1375-L1378) |
| `winona_telebrella_swap_fx` | Transform／AnimState／Follower／Network；bank／build `winona_telebrella`、播 `swap1`；`led_on` 泛光＋提亮；标签 `FX`；Pristine 前挂 `highlightchild`；主模拟加 `colouradder`＋`SetLedEnabled` 方法（`led_on`/`led_off` 符号互换＋泛光开关）；`persists=false` | 持握层不是贴图而是**动画覆盖**：两层实例按帧区间分工跟随 `swap_object` 符号（见生成点）；LED 亮灭由技能 `winona_wagstaff_2` 门控。[工厂](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L490-L523)、[FX_SetLedEnabled](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L474-L488)、[注册](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L525-L526) |
| `winona_battery_sparks` | Transform／AnimState／Network；bank／build `sparks`、Pristine 前播 `sparks_1`、`SetAddColour(1,1,0,0)` 黄色加色、LightOverride 0.3；标签 `FX`＋`NOCLICK`；主模拟随机 1–3 重选 `sparks_2/3`＋`animover` 自删＋`persists=false` | 无 SoundEmitter（电音由宿主播）；单实体直挂 `Follower`，不需要代理＋本地双层结构。[工厂](../../../raw/current/scripts/prefabs/winona_battery_sparks.lua#L7-L38)、[注册](../../../raw/current/scripts/prefabs/winona_battery_sparks.lua#L40) |

## 关联机制导航

### 发电机充放电与 shatter 触发（winona_battery_high_shatterfx、purebrilliance_symbol_fx 的宿主）

- 燃料放电：`fueled` 6 段（NUM_LEVELS）、`maxfuel`＝6 天、MAGIC 燃料类型（[fuel 装配](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L1331-L1336)、[tuning](../../../raw/current/scripts/tuning.lua#L3728-L3740)）；放电速率 `UpdateCircuitPower` 按连接节点的 `powerload` 均摊（[L350-L382](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L350-L382)）。供电：`circuitnode` 连入 `engineeringbatterypowered` 组后每 0.5 秒给每个节点 `AddBatteryPower`（[L327-L341](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L327-L341)）。外部抽电走 `battery` 组件（`CanBeUsedAsBattery`／`UseAsBattery`，单次成本 [WINONA_WXBATTERY_COST](../../../raw/current/scripts/tuning.lua#L3722-L3722) fuel＝1/3 天）。
- 宝石：3 槽（GEMSLOTS）、每颗宝石＝2 段（LEVELS_PER_GEM）；普通宝石符文换图，`purebrilliance`／`alterguardianhatshard` 走符号跟随 FX 并累计 `_brilliance_level`／`_shard_level`（[SetGem](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L263-L292)、[槽位符号](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L170-L172)）；纯粹辉煌需要技能 `winona_lunar_2`（[交易门禁](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L1152-L1171)）。

| shatter 触发 | 链路（file:line） | 定位模式 |
|---|---|---|
| 燃料段位下降 | `OnFuelSectionChange` → `ShatterGems(inst, ceil(new/2))`（[L820-L844](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L820-L844)）→ `UnsetGem`（[L294-L317](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L294-L317)）→ `LoseGem`（[L149-L162](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L149-L162)） | `FollowSymbol(电池, 宝石槽符号)` |
| 燃料耗尽 | `OnFuelEmpty` → `ShatterGems(inst, 0)`（[L795-L818](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L795-L818)） | 同上 |
| 砸毁／烧毁／拆除 | `OnWorkFinished`（[L622-L632](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L622-L632)）／`OnBurnt`（[L642-L666](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L642-L666)）／`ondeconstructstructure`（结构 [L1360-L1362](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L1360-L1362)、物品 [L1563](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L1563)）→ `DropGems`：除最后一颗全部弹成物品，最后一颗仅在非整段且非帽片时碎（[DropGems](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L600-L620)） | 世界坐标 `x, 2.5+0.75×slot, z` |
| 读档修正 | 结构 `OnLoad`（[L926](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L926)）／物品 `Item_OnLoad`（[L1511](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L1511)）→ `ShatterGems` | FollowSymbol |

`purebrilliance_symbol_fx` 的生成链：`SetGem` 里 `gemname == "purebrilliance"` 分支生成 → `SetParent(电池.entity)` → `FollowSymbol(电池, gem<4-slot>)` → 存入 `_gemsymfollowers[slot]`（[L268-L274](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L268-L274)）。**便携物品形态同样会生成**：拆解成物品（`ChangeToItem`→`CopyAllProperties`，[L542-L563](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L542-L563)）与物品重新部署（`OnDeploy`，[L1467-L1480](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L1467-L1480)）、物品 `Item_OnLoad` 都经由同一 `SetGem`。
清单外连带：同一分支的 `alterguardianhatshard` 走 `alterguardianhatshard_symbol_fx`（注册于 [alterguardianhatshard.lua:197](../../../raw/current/scripts/prefabs/alterguardianhatshard.lua#L197)），由 [[reference/prefabs/armor_lunarplant_glow_fx|装备与帽饰跟随光效]] 页覆盖，本页不展开。

### winona_battery_sparks 的 8 处生成

| 生成语句（file:line） | 调用函数与条件 | 定位 |
|---|---|---|
| [winona_telebrella.lua:345](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L343-L345) | `DoWireSparks`：电路接入 `OnConnectCircuit`（未连线→清除 wire 遮罩）与完全断开 `OnDisconnectCircuit`，非 POPULATING 才播 | FollowSymbol 自身 `wire` 符号 |
| [winona_spotlight.lua:910](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L908-L910) | 同上模式（[连接/断开](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L920-L945)） | FollowSymbol `wire` |
| [winona_catapult.lua:908](../../../raw/current/scripts/prefabs/winona_catapult.lua#L906-L908) | 同上模式（[L918-L946](../../../raw/current/scripts/prefabs/winona_catapult.lua#L918-L946)） | FollowSymbol `wire` |
| [winona_remote.lua:617](../../../raw/current/scripts/prefabs/winona_remote.lua#L615-L617) | 同上模式（[L627-L646](../../../raw/current/scripts/prefabs/winona_remote.lua#L627-L646)） | FollowSymbol `wire` |
| [winona_storage_robot.lua:614](../../../raw/current/scripts/prefabs/winona_storage_robot.lua#L612-L614) | 同上模式（[L624-L643](../../../raw/current/scripts/prefabs/winona_storage_robot.lua#L624-L643)） | FollowSymbol `wire` |
| [winona_spotlight.lua:779](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L777-L779) | `ChangeToItem` 拆解成物品时 `_wired` 才生成 | SetPosition 拆解点 |
| [winona_catapult.lua:207](../../../raw/current/scripts/prefabs/winona_catapult.lua#L205-L207) | 同上 | SetPosition |
| [winona_teleport_pad.lua:330](../../../raw/current/scripts/prefabs/winona_teleport_pad.lua#L328-L330) | 同上（传送板拆除） | SetPosition |

### winona_spotlight_head 与本体的父子／同步

- 生成与挂接：本体 master 初始化 `SpawnPrefab("winona_spotlight_head")` 后引擎 `SetParent`，登记进 `highlightchildren` 并 `colouradder:AttachChild`（[L1113-L1116](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L1113-L1116)、[L1126-L1127](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L1126-L1127)）；客户端侧 `OnHeadEntityReplicated` 回填 `highlightchildren` 与 `_clientheadinst`（[L1217-L1226](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L1217-L1226)）。
- 本体驱动灯头：转向与俯仰每帧由 `OnUpdateLightServer` 写 `Transform:SetRotation`＋`SetHeadTilt` 三档（[L634-L643](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L634-L643)、[SetHeadTilt](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L216-L231)）；点亮／熄灭切换 bloom 与 LightOverride（[EnableLight](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L561-L595)）；LED 闪烁由本体 `SetLedEnabled` 直接改灯头符号（[L236-L254](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L236-L254)）；受击／放置动画双实体成对播（[L783-L790](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L783-L790)）。热浪换层由 `SetHeated` 承担（夏季关闭，非夏季开，[L33-L47](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L33-L47)）。
- 端差异：光柱子体与热浪判断只存在非专服端（[L1252-L1256](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L1252-L1256)）。

### winona_telebrella_swap_fx 的两层换装语义

本体 `OnEquip` 只做静态 `swap_object` 符号覆盖（[L53](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L52-L66)）；动画与 LED 由 `SetFxOwner` 生成的两层承载（[L20-L50](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L20-L50)）：两层都 `SetParent(owner.entity)`＋跟随持有者 `swap_object` 符号，层 1 播工厂默认 `swap1`（帧区间参数 0 起 19 帧）、层 2 播 `swap20`（帧参数 19 起）；帧区间分工对应持握动画不同帧段（第 8/9 参的帧起始／帧数用法与 backpack、gestalt_cage 等处一致）。两层挂 `highlightchild:SetOwner` 并接入持有者 `colouradder`；技能 `winona_wagstaff_2` 未激活则两层 LED 关闭，`onactivateskill_server`／`ondeactivateskill_server` 时经 `RefreshAttunedSkills` 刷新（[L14-L18](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L14-L18)、[监听](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L47-L48)）。

### 清理路径

| 注册 | 清理方式 | 边界与证据 |
|---|---|---|
| `winona_battery_high_shatterfx` | 主模拟 `animover` 自删 | `persists=false`；双定位模式都不影响自删。[工厂尾](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L1459-L1460) |
| `purebrilliance_symbol_fx` | `UnsetGem`／再次 `SetGem` 先 `Remove` 旧层（`_gemsymfollowers` 槽位引用）；或电池实体移除时是否随引擎父级回收未核验 | 无自删；物品形态恢复时经 `SetGem` 重建。[移除](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L296-L300) |
| `winona_spotlight_head` | **无任何显式 Remove**：实际是否随引擎父级回收未核验；砸毁／烧毁路径只 `Hide` 灯头（本体随后 `ErodeAway` 退场） | 灯头 `persists` 未置 false——存档循环节跳过带引擎父级的实体（[mainfunctions.lua:1114](../../../raw/current/scripts/mainfunctions.lua#L1114)），随本体存档语义成立；烧毁后灯头保留但隐藏。[摧毁藏层](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L824-L827)、[烧毁藏层](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L854-L858) |
| `winona_telebrella_swap_fx` | `OnUnequip` → `SetFxOwner(inst, nil)`：两层一律 `Remove`（[L21-L26](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L21-L26)） | `persists=false`；跟随持有者引擎父挂，换手／下线由 unequip 链兜底。[调用点](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L81) |
| `winona_battery_sparks` | 主模拟 `animover` 自删 | `persists=false`；宿主只负责生成与音效。[工厂尾](../../../raw/current/scripts/prefabs/winona_battery_sparks.lua#L34-L35) |

## 典型接入

### 给"可插宝石的设施"做符号跟随层

- 前提与选择：设施把某物化成动画符号（槽位宝石）且需要泛光／粒子增强时，复刻 `purebrilliance_symbol_fx` 模式：独立工厂＋`Follower:FollowSymbol(宿主, 槽位符号)`＋宿主槽位数组管理。
- 生效与撤销：生成后父挂宿主即可；卸下／替换槽位时由宿主 `Remove`——不要依赖 animover。
- 原版依据：[SetGem 分支](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L268-L274)、[移除](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L296-L300)。

```lua
-- 服务端：给电池宿主的宝石槽挂一层联网符号跟随 FX
local function AttachGemSymbolFx(battery, slot)
    local fx = SpawnPrefab("purebrilliance_symbol_fx")
    fx.entity:SetParent(battery.entity)
    fx.Follower:FollowSymbol(battery.GUID, "gem" .. (4 - slot), 0, 0, 0, true)
    battery._gemsymfollowers[slot] = fx   -- 卸宝石/替换时先 Remove 该槽引用
end
```

- 调整与验证：替换点为注册名与槽位符号（`GetGemSymbol` 是 slot→`gem3/gem2/gem1` 的反向映射，[L170-L172](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L170-L172)）。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：碎宝石不等于掉宝石。** `LoseGem` 只生成碎裂动画并播 `gem_shatter`；真正以物品弹出的走 `FlingGem`。帽片（`alterguardianhatshard`）在两条链里都走 FlingGem 永不碎裂（[UnsetGem 分支](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L310-L316)、[DropGems noloss](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L612-L616)）。
- **边界：sparks 的随机动画只在主模拟重选。** Pristine 前固定 `sparks_1`，主模拟生成后随机改播 `sparks_2/3`；客户端看到的是联网实体动画同步结果，工厂无 netvar。[重选](../../../raw/current/scripts/prefabs/winona_battery_sparks.lua#L29-L32)
- **边界：灯头不要单独 Spawn 使用。** 灯头的符号层（`light*`）依赖本体隐藏自身层后才在视觉上成立，转向／俯仰完全由本体每帧写入；单独生成只会得到一个静止灯罩壳。[本体驱动](../../../raw/current/scripts/prefabs/winona_spotlight.lua#L634-L643)
- **边界：telebrella 持握层是两层成对的。** 手动 `SpawnPrefab("winona_telebrella_swap_fx")` 只会得到单层（默认 `swap1`），没有帧区间分工、highlightchild 归属与 LED 门控；必须走 `SetFxOwner` 或复刻其全部设置。[SetFxOwner](../../../raw/current/scripts/prefabs/winona_telebrella.lua#L20-L50)
- **边界：symbol fx 与 shatterfx 都无统一 Kill 方法。** 与本批 `slingshotammo_slow_debuff_fx` 的 `KillFX` 协议不同，清理只能靠宿主引用；引擎父级回收未核验。模组复刻时若忘记摘 `_gemsymfollowers` 引用会残留悬浮光点。[对比](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L294-L317)
- **边界：shatterfx 的动画名按宝石名拼接。** `gemname.."_shatter"` 依赖 `gems` 动画库内存在对应动画（`purebrilliance_shatter` 等未逐一核验资源）；`LoseGem` 先生成新壳，再比较所需动画与新壳默认 `redgem_shatter`——相同则不重播，不同才切到对应宝石动画。这不是跨调用的同帧判重，也不会限制同帧 FX 数量。[拼接与比较](../../../raw/current/scripts/prefabs/winona_battery_high.lua#L149-L154)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/winona_battery_high.lua`（宝石／燃料／电路全函数族、shatterfx 工厂、结构与物品两形态）、`prefabs/winona_battery_sparks.lua` 全文、`prefabs/winona_telebrella.lua` 全文（本体与 swap fx 两工厂）；按需读取 `prefabs/winona_spotlight.lua`（灯头工厂、本体生成与驱动、清理路径、DoWireSparks）、`prefabs/purebrilliance.lua`（symbolfxfn）、`prefabs/winona_teleport_pad.lua`／`winona_catapult.lua`／`winona_remote.lua`／`winona_storage_robot.lua` 的 sparks 生成段、`prefabs/alterguardianhatshard.lua` 注册行、`tuning.lua` 发电机段。对 5 个注册名以 `grep -rn "注册名" --include=*.lua` 在 `raw/current/scripts` 全域反查：`winona_battery_high_shatterfx` 命中注册（winona_battery_high.lua:1580）、依赖（:23）、生成（:150）——**生成点唯一**；`purebrilliance_symbol_fx` 命中注册（purebrilliance.lua:129）、依赖（battery_high:25）、生成（battery_high:269）——**生成点唯一**；`winona_spotlight_head` 命中注册（winona_spotlight.lua:1375）、依赖（:19）、生成（:1113）——**生成点唯一**；`winona_telebrella_swap_fx` 命中注册（winona_telebrella.lua:526）、依赖（:9）、生成（:28、:33）——**两处均在 SetFxOwner**；`winona_battery_sparks` 命中注册（winona_battery_sparks.lua:40）、prefablist.lua:1446、5 个设备文件的依赖表与 8 处生成（正文表格）——**8 处生成与台账一致**。
- 名称核验：`languages/chinese_s.po` 定向检索——四个宿主官方名见"快速判断"锚点；`WINONA_BATTERY_HIGH_SHATTERFX`／`WINONA_BATTERY_SPARKS`／`WINONA_SPOTLIGHT_HEAD`／`WINONA_TELEBRELLA_SWAP_FX`／`PUREBRILLIANCE_SYMBOL_FX` 均无 `STRINGS.NAMES.*` 键。
- 分档说明：`size_class: standard`——5 注册分三类载体但每条链都可独立成小节，生成点与清理以表格压缩；未选 complex 因不存在需要整体保留的多状态机（发电机充放电仅记到 shatter 触发深度，玩法本体另有展开空间）。
- 未覆盖：发电机超载（overload）与帽片蓄电的完整数值流程、`engineeringbatterypowered` 组内其他设备细节、聚光灯照明／加热与索敌逻辑、传送伞遥控传送玩法、`purebrilliance` 物品的沃特施法用法、`gems` 动画库内各 `*_shatter` 资源、皮肤对持握层 build 的影响。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录（0 error 0 warning 后入库）。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/armor_lunarplant_glow_fx|装备与帽饰跟随光效]]（`alterguardianhatshard_symbol_fx` 的主要页；同族符号跟随层协议）
- [[reference/prefabs/sparks|电击命中火花（sparks 族）]]（同名 bank 的另一独立工厂；对照代理 vs 直挂两种联网方案）
- [[reference/prefabs/slingshotammo_purebrilliance_debuff_fx|弹弓弹药减益表现]]（`purebrilliance` 主题的另一条表现链：命中标记双层）
- 返回 [[reference/prefabs/fx-structure-index|建筑、装置与遗迹演出]]（装备／薇诺娜设备）
- 2026-11-08 build 26474873：本页随版本迁移升版（锚点迁移 1 处），批次与共享依据见 [沿用记录](../../updates/build-26474873/index.md#wiki-沿用记录)（批V-12）。
