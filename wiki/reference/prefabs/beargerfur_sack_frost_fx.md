---
title: "极地熊獾桶霜气与伯尼大火 FX（beargerfur_sack_frost_fx、bernie_big_fire）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "熊獾掉落物与伯尼伙伴的外围 FX：极地熊獾桶放置态开桶时父挂跟随 ground 符号的联网霜气层（复用 icebox_open_fx 动画库、自定义 Kill 播 pst 退场），与伯尼！被点燃时的联网火层（Light＋AnimState＋SoundEmitter，反伤在宿主 damagereflect，20 秒后 EndBernieFire 播 pst 自删）。含 bernie_big 本体（health/combat/locomotor 完整生物）辨析。"
aliases: ["beargerfur_sack_frost_fx", "bernie_big_fire", "极地熊獾桶", "熊獾桶霜气", "开桶霜气", "伯尼大火", "伯尼燃烧", "大伯尼火层"]
tags: [prefab, fx, server-client]
size_class: standard
dst_build: "steam-buildid:26335394"
sources: ["prefabs/beargerfur_sack.lua", "prefabs/bernie_big.lua", "prefabs/bernie_common.lua", "prefabs/bernie_active.lua", "brains/berniebrain.lua", "stategraphs/SGberniebig.lua", "components/lighter.lua", "prefabs/willow_ember.lua", "prefabs/icebox_victorian_frost_fx.lua", "prefabs/icebox_kitchen_fx.lua", "prefabs/icebox_crystal_fx.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 极地熊獾桶霜气与伯尼大火 FX（beargerfur_sack_frost_fx、bernie_big_fire）

## 快速判断

- 职责与适用：**2 个不同宿主的外围 FX**。①`beargerfur_sack_frost_fx`——极地熊獾桶（beargerfur_sack）在**放置态**开桶时的联网霜气层：父挂桶体并 `FollowSymbol(桶, "ground", -25, -15, 0)`，复用 `icebox_open_fx` 动画库（pre→loop），关桶走自定义 `Kill` 播 pst 退场（[注册](../../../raw/current/scripts/prefabs/beargerfur_sack.lua#L251-L253)）；②`bernie_big_fire`——伯尼！（bernie_big）被薇洛点着的联网火层：自带 Light（半径 3）＋火焰动画＋树火循环声，20 秒后 `EndBernieFire` 转 pst 自删；真实玩法（50 点伤害反射）在宿主的 `damagereflect` 组件上（[注册](../../../raw/current/scripts/prefabs/bernie_big.lua#L775-L776)）。
- 重要副作用：点伯尼的同时宿主获得 `damagereflect`（`BERNIE_BURNING_REFLECT_DAMAGE=50`）并移除 `canlight` 标签，20 秒收尾时回挂；霜气纯表现（容器保鲜在宿主 preserver 上）。
- 优先使用：查"开箱子冷气怎么做的"走霜气链；查"大伯尼点燃反伤怎么给"走火链——注意伤害归属在宿主组件，火层只是光声画载体。
- 修改边界：只覆盖 2 注册的工厂、生成点、端与清理；熊獾桶容器 widget、伯尼转形链与同盟皮肤系统、willow_ember 余烬玩法不展开。`bernie_big` 本体辨析见[本体辨析](#berniebig-本体辨析)小节。
- 名称：`BEARGERFUR_SACK`＝"极地熊獾桶"、`BERNIE_BIG`＝"伯尼！"（官方键名 **BEARGERFUR** 系源码拼写，非 BEARERFUR，锚点见[验证记录](#验证记录)）；2 个覆盖注册名在简中 `chinese_s.po` 均无 `STRINGS.NAMES.*` 键，标题中文为已核验效果描述非官方译名。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `beargerfur_sack_frost_fx` | `fxfn`：Transform／AnimState／Follower／Network；bank＝build `icebox_open_fx`（**脚本层唯一使用者**）；`pre`→`loop` 动画；FinalOffset 3、LightOverride 0.7、整体缩放 0.7；标签 `FX`＋`NOCLICK`＋`NOBLOCK` → SetPristine 客户端早退 → 主模拟 `persists=false`；`Kill` 被替换为自定义 `FX_Kill`，`OnKillTask` 指向 `FX_OnKillTask` | 无 SoundEmitter、无 Light——"冷气"只有动画；退场是两段式：先等当前动画播完，再播 `pst` 并在 pst 长度＋2 帧后 Remove（[工厂](../../../raw/current/scripts/prefabs/beargerfur_sack.lua#L211-L248)、[退场函数](../../../raw/current/scripts/prefabs/beargerfur_sack.lua#L136-L146)） |
| `bernie_big_fire` | `firefn`：Transform／AnimState／**SoundEmitter**／**Light**／Network；Light r3／i0.7／falloff 1／色 (180,195,150)/255（黄绿）＋EnableClientModulation；bank＝build `bernie_fire_fx`，`bernie_fire_reg_pre`→`bernie_fire_reg` 循环；泛光＋`MultColour(1, 0.7, 0, 0.3)`（橙色半透明）；循环声 `dontstarve/common/treefire`；标签 `FX`＋`NOCLICK`；`EndBernieFire` 挂在 SetPristine 之后（两端实例都有）→ 主模拟监听 `animover`：**仅当前动画是 `bernie_fire_reg_pst` 时** Remove | 自删唯一入口是 pst 播完；直接 Remove（引擎级联）不播 pst（[工厂](../../../raw/current/scripts/prefabs/bernie_big.lua#L731-L773)、[EndBernieFire](../../../raw/current/scripts/prefabs/bernie_big.lua#L726-L729)、[自删监听](../../../raw/current/scripts/prefabs/bernie_big.lua#L766-L770)） |

## 关联机制导航

### 霜气的开关与退场

| 环节 | 行为与条件 | 证据 |
|---|---|---|
| 生成 | `ToggleFrostFX(inst, true)`：生成 FX、父挂桶体、`FollowSymbol(inst.GUID, "ground", -25, -15, 0)`；唯一调用在 `OnOpen`——**仅放置态**（`not inventoryitem:IsHeld()`）且延迟＝开桶动画长度；手持开桶只提前播循环声、不生霜气 | [ToggleFrostFX](../../../raw/current/scripts/prefabs/beargerfur_sack.lua#L32-L52)、[OnOpen](../../../raw/current/scripts/prefabs/beargerfur_sack.lua#L75-L97) |
| 关桶 | `OnClose` → `ToggleFrostFX(inst, false)` → 走 `Kill`：`FX_Kill` 先等当前动画播完 → `FX_OnKillTask` 播 `pst` → pst 长度＋2 帧后 Remove；`ToggleFrostFX` 开头取消未触发的延迟生成任务 | [OnClose](../../../raw/current/scripts/prefabs/beargerfur_sack.lua#L99-L120)、[Kill 两段](../../../raw/current/scripts/prefabs/beargerfur_sack.lua#L136-L146) |
| 即删路径 | 收进库存 `OnPutInInventory`、实体移除 `OnRemoveEntity` → `ToggleFrostFX(false, true)` **直接 Remove 不播 pst** | [OnPutInInventory](../../../raw/current/scripts/prefabs/beargerfur_sack.lua#L122-L127)、[OnRemoveEntity](../../../raw/current/scripts/prefabs/beargerfur_sack.lua#L129-L132) |
| 宿主 | `beargerfur_sack`：容器物品（`portablestorage`，`droponopen=true`），preserver 速率 0.05；配方＝月晶锻造二级（月神熔炉：满警戒笼＋熊獾毛＋纯粹辉煌 3＋月岩 5） | [宿主工厂](../../../raw/current/scripts/prefabs/beargerfur_sack.lua#L152-L209)、[配方](../../../raw/current/scripts/recipes.lua#L857)、[调值](../../../raw/current/scripts/tuning.lua#L7967-L7967) |

`icebox_open_fx` 动画库名带 icebox 前缀，但冰箱本体及其皮肤 FX 用的是 `icebox_kitchen`／`icebox_victorian`／`icebox_crystal` 等独立 bank——全库该 bank 的唯一消费者就是本页霜气层（负向检索见[验证记录](#验证记录)）。

### bernie_big_fire 的点燃链与收尾

| 环节 | 行为与条件 | 证据 |
|---|---|---|
| 技能门禁 | `onLeaderChanged`：伯尼的 leader 激活 `willow_burnignbernie` 技能（源码拼写如此）→ 给伯尼挂 `canlight` 并监听 `onlighterlight` | [监听注册](../../../raw/current/scripts/prefabs/bernie_big.lua#L344-L347) |
| 点燃来源 | `lighter` 组件 `Light()` 无条件推 `onlighterlight`；willow_ember 的加热目标与爆燃对 `canlight` 实体也推同事件 | [lighter.lua](../../../raw/current/scripts/components/lighter.lua#L18-L26)、[ember 加热](../../../raw/current/scripts/prefabs/willow_ember.lua#L243-L243)、[ember 爆燃](../../../raw/current/scripts/prefabs/willow_ember.lua#L283-L283) |
| **火层生成** | `OnLighterLight`：无 `fire_thorns_task`（未燃烧）→ 生成 `bernie_big_fire` 父挂伯尼；按 leader 同盟技能换装——暗影：build `bernie_fire_fx_shadow_build`＋`MultColour(0,0,0,0.5)`；月亮：build `bernie_fire_fx_lunar_build`＋`MultColour(0.3,0.3,0.3,0.3)`；`SetFinalOffset(-3)`；宿主挂 `damagereflect`（50）＋监听 `onreflectdamage`（对攻击者按 hiteffectsymbol 挂 `impact` 命中 FX）；移除 `canlight` | [OnLighterLight](../../../raw/current/scripts/prefabs/bernie_big.lua#L235-L265)、[反射命中 FX](../../../raw/current/scripts/prefabs/bernie_big.lua#L45-L59)、[调值](../../../raw/current/scripts/tuning.lua#L8127-L8127) |
| **收尾** | 点燃即排 20 秒任务（燃烧中再点燃只重置该任务、不重生火层）→ `endthornsfire`：`fire_fx:EndBernieFire()` 播 `bernie_fire_reg_pst` → FX 的 animover 自删；宿主移除 `damagereflect` 与事件监听、回挂 `canlight` | [20 秒任务](../../../raw/current/scripts/prefabs/bernie_big.lua#L262-L264)、[endthornsfire](../../../raw/current/scripts/prefabs/bernie_big.lua#L61-L76) |
| 级联清理 | 伯尼死亡 SG → `GoInactive` → 生成 `bernie_inactive` 并 `inst:Remove()` → 引擎父挂级联移除火层（**无 pst**）；`goinactive` 不调 `endthornsfire` | [死亡状态](../../../raw/current/scripts/stategraphs/SGberniebig.lua#L266-L272)、[goinactive](../../../raw/current/scripts/prefabs/bernie_big.lua#L78-L95) |
| 生成链 | `bernie_active`（行走伯尼）brain 判定 leader 低理智或 `hotheaded`（`willow_bernieai` 技能＋20 格内敌对对齐生物）→ `GoBig` → `bernie_big` | [gobig](../../../raw/current/scripts/prefabs/bernie_active.lua#L40-L71)、[brain 调用](../../../raw/current/scripts/brains/berniebrain.lua#L127-L130)、[hotheaded](../../../raw/current/scripts/prefabs/bernie_common.lua#L16-L24) |

伯尼的同盟头肩火焰（暗影／月亮形态常驻光焰）是 `CreateFlameFx` 的**本地非网络实体**（非注册 Prefab），由 `current_allegiance` 净变量 dirty 驱动生成/移除，专服跳过（[CreateFlameFx](../../../raw/current/scripts/prefabs/bernie_big.lua#L454-L482)、[暗影](../../../raw/current/scripts/prefabs/bernie_big.lua#L484-L500)、[月亮](../../../raw/current/scripts/prefabs/bernie_big.lua#L502-L518)）——不在本页覆盖清单，列出仅为划清边界。

### bernie_big 本体辨析

`bernie_big` 是**完整生物**而非 FX 或物品：`health`（2000，nofadeout）／`combat`（伤害 50、攻击周期、近远攻击距离、retarget/keep-target）／`locomotor`（走 4.5 跑 11，技能加速）／`drownable`／`embarker`／`activatable`（安抚）／`timer`／`damagetyperesist`／`damagetypebonus`，`MakeCharacterPhysics(500, .65)`、缩放 0.7、SG `SGberniebig`＋Brain `berniebigbrain`、标签 `largecreature`／`companion`／`soulless`／`crazy`／`bigbernie`（[组件装配](../../../raw/current/scripts/prefabs/bernie_big.lua#L666-L723)）。台账批 2 "本体是 health/combat/locomotor 完整生物"的结论复核属实；本页只覆盖其火层 FX，本体玩法另建主题。

### 清理路径

| 注册 | 清理方式 | 边界 |
|---|---|---|
| `beargerfur_sack_frost_fx` | 关桶 `Kill` 两段 pst 退场；收库存/移除即删；桶体移除级联 | 宿主 `OnRemoveEntity` 也显式清引用，双保险 |
| `bernie_big_fire` | 20 秒任务 → `EndBernieFire` pst → animover 自删（唯一"优雅"路径）；宿主移除级联（无 pst） | 无 `KillFX`；燃烧状态无显式 OnSave（见陷阱） |

## 典型接入

### 给可点燃宿主做"点燃 → 反伤 → 定时收尾"

- 前提与选择：宿主需自带事件入口（原版是 `onlighterlight`）或自定义点燃事件；火层与玩法组件分开——FX 父挂宿主管视觉，玩法（反射、buff）挂宿主组件，收尾时两边同步撤。
- 生效与撤销：点燃生成 FX 并 `SetDefaultDamage`；到期回调里先 `fx:EndBernieFire()`（转 pst 播完自删）再撤组件——顺序反过来会出现"组件没了火还在烧"的 20 秒窗口外残留。
- 原版依据：[OnLighterLight](../../../raw/current/scripts/prefabs/bernie_big.lua#L235-L265)、[endthornsfire](../../../raw/current/scripts/prefabs/bernie_big.lua#L61-L76)、[EndBernieFire](../../../raw/current/scripts/prefabs/bernie_big.lua#L726-L729)。

```lua
local function OnLight(inst)
    if inst.fire_task ~= nil then
        inst.fire_task:Cancel() -- 燃烧中复燃：只重置窗口
    else
        inst.fire_fx = SpawnPrefab("bernie_big_fire")
        inst.fire_fx.entity:SetParent(inst.entity)
        inst:AddComponent("damagereflect")
        inst.components.damagereflect:SetDefaultDamage(TUNING.BERNIE_BURNING_REFLECT_DAMAGE)
    end
    inst.fire_task = inst:DoTaskInTime(20, EndFire, inst)
end

local function EndFire(inst)
    if inst.fire_fx ~= nil then inst.fire_fx:EndBernieFire() end -- 转 pst，播完 animover 自删
    inst.fire_task = nil
    inst:RemoveComponent("damagereflect")
end
```

- 调整与验证：替换点是事件名与窗口时长；`EndBernieFire` 是该注册的协议方法（SetPristine 后挂载，两端可用），换火层时需自带等价退场。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：霜气只在放置态开桶出现。** 手持打开（`IsHeld()`）分支只播开桶循环声；`droponopen=true` 的掉落行为在容器组件侧，与霜气生成无关。[OnOpen](../../../raw/current/scripts/prefabs/beargerfur_sack.lua#L89-L96)
- **边界：霜气退场有两种语义。** 关桶走 `Kill`（等帧＋pst）有淡出；收进库存/桶被移除走直接 `Remove`（无淡出）；`FX_Kill` 的 `_killtask` 单槽守卫使重复调用无效。[Kill 两段](../../../raw/current/scripts/prefabs/beargerfur_sack.lua#L141-L146)
- **边界：`icebox_open_fx` 是独占复用。** 想改"冰箱冷气"别动这个动画库——冰箱本体与皮肤 FX 都不用它，改了只影响熊獾桶（与未来任何复用者）。[验证记录](#验证记录)
- **边界：伯尼火层的颜色叠层。** 基础档 `MultColour(1,0.7,0,0.3)` 已是半透明橙；同盟 build 变体直接覆写 build 与颜色——同一实例只能是一种形态，皮肤 `bernie_fire_fx.zip` 仅基础档加载。[工厂](../../../raw/current/scripts/prefabs/bernie_big.lua#L747-L752)、[变体](../../../raw/current/scripts/prefabs/bernie_big.lua#L243-L251)
- **边界：燃烧状态不持久化。** `fire_fx` 引用与 20 秒任务无 OnSave；读档后燃烧中断（火层消失、`canlight` 回到构造初值——初始即无），重新点燃即可恢复链路。`damagereflect` 读档重建后的调值恢复未逐项核验（见存疑）。[OnLighterLight](../../../raw/current/scripts/prefabs/bernie_big.lua#L235-L265)
- **边界：点燃门槛在 leader 而非点火者。** 监听只在 leader 有 `willow_burnignbernie` 时注册，任何持有薇洛打火机/余烬的玩家都能触发；`canlight` 标签同时是 willow_ember 爆燃检索的入参（[监听注册](../../../raw/current/scripts/prefabs/bernie_big.lua#L344-L347)、[ember 爆燃](../../../raw/current/scripts/prefabs/willow_ember.lua#L283-L283)）。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/beargerfur_sack.lua`、`components/lighter.lua`、`prefabs/bernie_common.lua`；按需读取 `prefabs/bernie_big.lua` 全部 FX 相关函数（ToggleFrostFX 对应文件为 beargerfur_sack；bernie_big 的 OnReflectDamage/endthornsfire/goinactive/OnLighterLight/onLeaderChanged/CreateFlameFx 族/firefn/EndBernieFire/组件装配段）、`prefabs/bernie_active.lua` gobig、`brains/berniebrain.lua` GoBig 判定段、`stategraphs/SGberniebig.lua` death 状态、`prefabs/willow_ember.lua` 两处推送、`recipes.lua` 桶配方、`tuning.lua`（BERNIE_BIG_*、BERNIE_BURNING_REFLECT_DAMAGE、BEARGERFUR_SACK_PRESERVER_RATE）、icebox 三款皮肤 FX 的 bank 对照（[icebox_victorian_frost_fx.lua](../../../raw/current/scripts/prefabs/icebox_victorian_frost_fx.lua#L26-L27)、[icebox_kitchen_fx.lua](../../../raw/current/scripts/prefabs/icebox_kitchen_fx.lua#L26-L27)、[icebox_crystal_fx.lua](../../../raw/current/scripts/prefabs/icebox_crystal_fx.lua#L26-L27)）。对 2 个覆盖注册名全域反查：`beargerfur_sack_frost_fx` 命中 3 处（依赖声明 [beargerfur_sack.lua:15](../../../raw/current/scripts/prefabs/beargerfur_sack.lua#L15)、生成 [L39](../../../raw/current/scripts/prefabs/beargerfur_sack.lua#L39)、注册 [L253](../../../raw/current/scripts/prefabs/beargerfur_sack.lua#L253)）——**生成点唯一**；`bernie_big_fire` 命中 3 处（依赖 [bernie_big.lua:28](../../../raw/current/scripts/prefabs/bernie_big.lua#L28)、生成 [L240](../../../raw/current/scripts/prefabs/bernie_big.lua#L240)、注册 [L776](../../../raw/current/scripts/prefabs/bernie_big.lua#L776)）——**生成点唯一**。两注册均不在 prefablist.lua（宿主 `beargerfur_sack`:111、`bernie_big`:129 在列）。台账批 2 的 2 注册清单复核无清单外 FX 注册（`CreateFlameFx` 系本地工厂非注册，已列边界）。
- 名称核验：`languages/chinese_s.po` 定向检索——`BEARGERFUR_SACK`＝"极地熊獾桶"（[L339219-L339223](../../../raw/current/scripts/languages/chinese_s.po#L339219-L339222)）、`BERNIE_ACTIVE`／`BERNIE_INACTIVE`＝"伯尼"、`BERNIE_BIG`＝"伯尼！"（[L339379-L339393](../../../raw/current/scripts/languages/chinese_s.po#L339379-L339392)）；`BEARGERFUR_SACK_FROST_FX`／`BERNIE_BIG_FIRE` 无 NAMES 键。标题中文使用已核验效果描述。
- 分档说明：`size_class: standard`——2 注册两条独立链（霜气开关链／点燃反伤链）加本体辨析与本地工厂边界；未选 complex，两链均单程无多状态。
- 未覆盖：熊獾桶容器 widget 与皮肤、伯尼同盟皮肤系统全貌（`OverrideItemSkinSymbol` 符号表）、`willow_ember` 余烬玩法全貌、`SGberniebig` 攻击状态机、`bernie_inactive` 转化冷却与 fueled 细节、`damagereflect` 组件读档行为、动画包帧内容；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录（0 error 0 warning 后入库）。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/bearger_swipe_fx|熊獾挥击地面特效]]（熊獾主题的另一表现链）
- [[reference/prefabs/ice_crack_grid_fx|海冰网格裂纹]]（冰面语义对照；霜气动画库与冰箱皮肤 FX 分属不同 bank）
- [[reference/prefabs/fire|火焰、火光与烟雾特效族]]（常规火焰的 firefx／heater 体系——伯尼火层同样不走该链）
- 返回 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]（熊獾掉落物与伯尼伙伴）
