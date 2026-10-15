---
title: "枕头大战攻击碎屑与作物挖起表现族（attackfx_handpillow_petals 等 7 注册）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "两个共享工厂、互不相干宿主的 7 个联网一次性表现注册：pillow_equipment.lua 按 pillow_defs 四材质动态拼名的枕头攻击碎屑（零伤害武器的命中表现，击退结算在 pillow_common）、plant_normal.lua 的作物挖起三档尘土（挖起裁决在 workable 回调）；另记 plant_dug_small_fx 的骑枪冲锋与驼鹿冲撞两处清单外生成点。"
aliases: ["attackfx_handpillow_petals", "attackfx_handpillow_kelp", "attackfx_handpillow_beefalowool", "attackfx_handpillow_steelwool", "plant_dug_small_fx", "plant_dug_medium_fx", "plant_dug_large_fx", "枕头大战", "yotr 枕头", "枕头攻击碎屑", "枕头命中特效", "作物挖起特效", "挖起尘土", "冲锋尘土", "驼鹿冲撞尘土"]
tags: [prefab, fx, server-client]
size_class: complex
dst_build: "steam-buildid:26335394"
sources: ["prefabs/pillow_equipment.lua", "prefabs/pillow_defs.lua", "prefabs/pillow_common.lua", "prefabs/plant_normal.lua", "prefabs/veggies.lua", "prefabs/seeds.lua", "components/grower.lua", "components/crop.lua", "prefabs/player_common_extensions.lua", "stategraphs/SGwilson.lua", "prefabs/woodie.lua", "prefabs/cozy_bunnyman.lua", "brains/cozy_bunnymanbrain.lua", "prefabs/yotr_fightring.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 枕头大战攻击碎屑与作物挖起表现族（attackfx_handpillow_petals 等 7 注册）

## 快速判断

- 职责与适用：两个共享工厂、两个互不相干宿主的 **7 个联网一次性表现注册**。①`pillow_equipment.lua` 的 4 个枕头攻击碎屑 `attackfx_handpillow_petals`／`_kelp`／`_beefalowool`／`_steelwool`（按 `pillow_defs` 四材质动态拼名，yotr 2023 兔子年枕头大战的手持枕命中表现）；②`plant_normal.lua` 的 3 个作物挖起尘土 `plant_dug_small_fx`／`plant_dug_medium_fx`／`plant_dug_large_fx`（农作物与野生作物被铲挖起的三档表现）。注册行：[枕头 4 注册](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L292-L297)、[挖起 3 注册](../../../raw/current/scripts/prefabs/plant_normal.lua#L321-L325)
- 两种载体都是**联网动画壳、零玩法**：7 注册均 Transform／AnimState／Network＋`FX`／`NOCLICK` 标签＋`persists=false`，无 physics、无 combat。枕头碎屑寿命＝动画长度＋2 帧（`FX_EXTRA_TIME`）；挖起尘土监听 `animover` 走 `ErodeAway` 侵蚀。
- 重要副作用：玩法结算都不在 FX 上——枕头命中是 **`TUNING.PILLOW_DAMAGE = 0` 的零伤害武器**（[tuning.lua](../../../raw/current/scripts/tuning.lua#L6909)），击退只在兔子人枕头大战小游戏里由 `pillow_common.DoKnockback` 推 `knockback` 事件，非小游戏目标只推 `damage = 0` 的 `attacked` 事件（仇恨）；挖起的产物掉落与本体移除在 workable 的 `OnDigUp` 与 `crop:Harvest`。
- 修改边界：只覆盖 7 注册的工厂、生成点、触发链、端模型与清理；枕头大战玩法（yotr_fightring 场地、兔人 AI、奖品结算）与农耕玩法（crop 生长、农场土壤）只在辨析内说明不展开。清单外发现：`plant_dug_small_fx` 另有两处生成点——yoth 骑枪冲锋（`player_common_extensions.lua` 的 `OnStartJoust`）与 Woodie 驼鹿形态冲撞（`SGwilson` 的 `tackle_start` 状态），已并入生成点表。
- 名称：宿主官方名——手持枕 4 件 `HANDPILLOW_*`（"花朵枕头／海带枕头／皮弗娄牛毛枕头／钢丝绵枕头"）、身体枕 4 件 `BODYPILLOW_*`（"××枕头盔甲"）、`PLANT_NORMAL`＝"农作物"、`PLANT_NORMAL_GROUND`＝"野生作物"（锚点见[验证记录](#验证记录)）；7 个覆盖注册名自身在简中 `chinese_s.po` 均无 `STRINGS.NAMES.*` 键，标题与正文效果描述**非官方译名**。

## 注册与构造

### 枕头攻击碎屑（pillow_equipment.lua 4 注册）

4 注册由同一工厂 `MakePillowFX(materialname, data)` 生成（[工厂](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L261-L290)），在文件尾循环 `require("prefabs/pillow_defs")` 的 4 个材质键逐一创建 body／hand／FX 三件（[循环](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L292-L297)），动态拼名 `Prefab("attackfx_handpillow_"..materialname)`（[拼名](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L289)）。

| 注册名 | 动画 | 差异与证据 |
|---|---|---|
| `attackfx_handpillow_petals` | `debris_petals` | 花朵枕头（strengthmult 1.0、laglength 0.75）命中碎屑。[defs](../../../raw/current/scripts/prefabs/pillow_defs.lua#L3-L12) |
| `attackfx_handpillow_kelp` | `debris_kelp` | 海带枕头（1.4／0.60）。[defs](../../../raw/current/scripts/prefabs/pillow_defs.lua#L13-L22) |
| `attackfx_handpillow_beefalowool` | `debris_beefalowool` | 皮弗娄牛毛枕头（1.8／0.40）。[defs](../../../raw/current/scripts/prefabs/pillow_defs.lua#L23-L32) |
| `attackfx_handpillow_steelwool` | `debris_steelwool` | 钢丝绵枕头（2.2／0.20）。[defs](../../../raw/current/scripts/prefabs/pillow_defs.lua#L33-L42) |

工厂共性：Transform／AnimState／Network（**无 SoundEmitter**——命中声 `yotr_2023/common/pillow_hit_<材质>` 由手持枕实体代播）＋标签 `FX`／`NOCLICK`＋bank=build `pillow_equipment_fx`＋播 `debris_<材质>`；`SetPristine` 后客户端早退；主模拟 `persists=false`＋`DoTaskInTime(动画长度 + 2 帧, Remove)` 自删（[自删](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L282)）。壳方法 `FastForward(pct)`：取消定时、`SetTime(长度*pct)`、按剩余时长＋2 帧重排移除（[FastForward](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L251-L259)）——**快照内无任何调用方**，是预留接口（同名的 groundpoundring／激光拖尾 FastForward 是其它注册的独立方法）。

### 作物挖起尘土（plant_normal.lua 3 注册）

3 注册由同一工厂 `MakeFX(size)` 生成（[工厂](../../../raw/current/scripts/prefabs/plant_normal.lua#L290-L319)），与本体 `plant_normal`／`plant_normal_ground` 同文件返回（[返回](../../../raw/current/scripts/prefabs/plant_normal.lua#L321-L325)）。

| 注册名 | 动画 | 差异与证据 |
|---|---|---|
| `plant_dug_small_fx` | `dug_small` | 未长成（growthpercent<0.4）、枯萎、无 crop 防御路径与全部清单外生成点都用这一档。[工厂](../../../raw/current/scripts/prefabs/plant_normal.lua#L290-L319) |
| `plant_dug_medium_fx` | `dug_medium` | growthpercent 0.4–0.7 的挖起。[选择](../../../raw/current/scripts/prefabs/plant_normal.lua#L112-L116) |
| `plant_dug_large_fx` | `dug_large` | 成熟挖起与"已成熟直接收获"两分支。[选择](../../../raw/current/scripts/prefabs/plant_normal.lua#L108-L116) |

工厂共性：Transform／AnimState／**SoundEmitter（未播放任何声）**／Network＋标签 `FX`／`NOCLICK`＋bank=build `plant_normal_ground`＋播 `dug_<size>`；客户端早退；主模拟监听 `animover` → `ErodeAway` 侵蚀、`persists=false`（[收尾](../../../raw/current/scripts/prefabs/plant_normal.lua#L312-L313)）。生成端 `SpawnDugFX` 会对**非 small 档且宿主不是 `plant_normal_ground`** 的尘土做 `OverrideSymbol("leaves", "plant_normal", "leaves")`，让叶片符号匹配农作物 build（[覆盖](../../../raw/current/scripts/prefabs/plant_normal.lua#L22-L28)）。

### 本体辨析（两个宿主家族，不计 FX 覆盖）

- **手持枕／身体枕**（`handpillow_<材质>`／`bodypillow_<材质>`，[注册](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L115)、[L239](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L239)）：yotr 兔子年活动物品，经 `TECH.RABBITOFFERING_THREE` 献礼配方制作（[recipes](../../../raw/current/scripts/recipes.lua#L1135-L1143)，actionstr `PERDOFFERING`）。手持枕是零伤害武器（`weapon:SetDamage(TUNING.PILLOW_DAMAGE)`、副攻击距离 `TUNING.PILLOW_HIT_RANGE`=2.5，[武器段](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L224-L227)）；身体枕带 `_defense_amount`（0.1–0.7）在被击退时抵扣强度。材质按 Pillow_def 递进：花瓣→海带→牛毛→钢丝绵。
- **农作物／野生作物**（`plant_normal`／`plant_normal_ground`）：同一 `MakePlant` 工厂的两份注册（[工厂](../../../raw/current/scripts/prefabs/plant_normal.lua#L192-L288)）。`plant_normal`（农作物）由农场土壤 `grower` 组件播种时生成且 `persists=false`（[grower.lua](../../../raw/current/scripts/components/grower.lua#L129-L132)）；`plant_normal_ground`（野生作物，官方名见验证记录）**在快照 scripts 内没有活的生成点**——只有依赖声明（[seeds.lua](../../../raw/current/scripts/prefabs/seeds.lua#L14)、[veggies.lua](../../../raw/current/scripts/prefabs/veggies.lua#L178)）与一段注释掉的旧播种路径（[veggies.lua](../../../raw/current/scripts/prefabs/veggies.lua#L196-L202)），世界生成引用应在快照外数据文件，不判死代码。两者都装 `workable`（DIG，1 次）与 `crop`。

## 关联机制导航

### 生成点与宿主调用

| 注册 | 生成语句（file:line） | 消费文件与条件 |
|---|---|---|
| `attackfx_handpillow_*` 4 注册 | 动态拼名 `SpawnPrefab("attackfx_handpillow_"..materialname)`：[pillow_equipment.lua#L161](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L161) | 唯一生成点＝手持枕 `weapon:SetOnAttack(onweaponattack)` 回调（[接线](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L227)）：命中目标后在**目标位置**（玩家 +1.0、NPC +2.5 垂直偏移）落一枚碎屑，不挂父级不跟随；命中声在武器上播，随后按是否在小游戏分流击退/仇恨 |
| `plant_dug_small_fx` | 动态拼名 `SpawnPrefab("plant_dug_"..size.."_fx")`：[plant_normal.lua#L23](../../../raw/current/scripts/prefabs/plant_normal.lua#L23)；直接：[player_common_extensions.lua#L1183](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L1183)（骑枪）、[SGwilson.lua#L22103](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22103)（驼鹿） | ①宿主挖起（见[挖起裁决链](#挖起裁决链与档位选择)）；②yoth 骑枪 `YOTH_LANCE`（官方名"冲锋骑枪"）冲锋：`joustuser` 组件触发 `OnStartJoust` 的周期任务，每 4–6 帧在玩家身后落一枚（50% 镜像、缩放 0.5–0.8）；③Woodie 驼鹿形态 `tackle_start` 冲撞状态：同型周期任务（缩放 0.8–1.3、偏移 1.6 格）。`woodie.lua` 依赖表另声明该注册（[woodie.lua#L59](../../../raw/current/scripts/prefabs/woodie.lua#L59)） |
| `plant_dug_medium_fx`／`plant_dug_large_fx` | 同上动态拼名（[plant_normal.lua#L23](../../../raw/current/scripts/prefabs/plant_normal.lua#L23)） | 仅作物挖起一条链；全 `raw/current/scripts` 反查无其它生成点、无直接字面量调用 |

### 枕头命中链：谁结算什么

1. **武器攻击**：手持枕是 `weapon` 组件载体，伤害恒 0；`onweaponattack` 在武器实体播 `pillow_hit_<材质>`、在目标位置生成碎屑 FX（[回调](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L153-L176)）。
2. **小游戏击退**：目标是 `minigame_participator` 且 `CurrentMinigameType() == "bunnyman_pillowfighting"`（兔人枕头大战，[注册](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L645-L646)）→ `pillow_common.DoKnockback(target, attacker, {amount=inst._knockback, strengthmult=inst._strengthmult})`（[分流](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L165-L172)）。`DoKnockback`（[pillow_common.lua](../../../raw/current/scripts/prefabs/pillow_common.lua#L4-L25)）跳过 `epic`／`nopush` 目标，用目标**身上身体枕的 `_defense_amount`** 抵扣强度，给目标推 `knockback` 事件（radius＝攻击者物理半径(默认 0.5)＋amount、`forcelanded`＝目标穿身体枕）；`knockback` 事件的实际位移由各 SG 的 knockback 状态消费。
3. **非小游戏目标**：只推 `attacked {attacker, weapon, damage = 0}`——零伤害的仇恨事件（[L174](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L174)）。
4. **节奏与奖品**：材质的 `_laglength` 是玩家挥枕状态的 timeout（[SGwilson.lua#L11135](../../../raw/current/scripts/stategraphs/SGwilson.lua#L11135)，兔人 AI 攻击延迟同源（[brain](../../../raw/current/scripts/brains/cozy_bunnymanbrain.lua#L749)））；`_prize_value`（0.25–1.0）在枕头战终局计入奖品数（[yotr_fightring.lua](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L275-L278)，上限 `PILLOWFIGHT_PRIZE_CAP`=10）。**FX 自身零结算**；出界彩纸 `pillowfight_confetti_fx` 是通用表条目，见 [[reference/prefabs/farm_plant_happy|作物表情与活动彩纸]]。

### 挖起裁决链与档位选择

`workable`（DIG 1 次）的 `OnFinishCallback = OnDigUp`（[接线](../../../raw/current/scripts/prefabs/plant_normal.lua#L249-L252)），档位与本体去留全在该回调（[OnDigUp](../../../raw/current/scripts/prefabs/plant_normal.lua#L100-L128)）：

| 宿主状态 | FX 档位 | 本体去留 |
|---|---|---|
| 成熟或枯萎（`crop:Harvest` 成功） | 枯萎→small，否则 large（[L108](../../../raw/current/scripts/prefabs/plant_normal.lua#L108)） | `Harvest` 内部已处理：产物落地/给收获者，无 grower 则本体 `Remove`，有 grower 则 `RemoveCrop`（[crop.lua](../../../raw/current/scripts/components/crop.lua#L205-L244)、[grower.lua](../../../raw/current/scripts/components/grower.lua#L149-L150)）——FX 生成时本体已不在 |
| 生长中（growthpercent<0.4／<0.7／其余） | small／medium／large（[L112-L116](../../../raw/current/scripts/prefabs/plant_normal.lua#L112-L116)） | 有有效 grower → `RemoveCrop`（内部 `crop:Remove()`）后返回；无 grower → 本体 `Remove`（[L118-L127](../../../raw/current/scripts/prefabs/plant_normal.lua#L118-L127)） |
| 无 `crop` 组件（防御分支，两个注册本体都装 crop） | small（[L125](../../../raw/current/scripts/prefabs/plant_normal.lua#L125)） | 本体 `Remove` |

挖起声与铲土动画由玩家的 DIG 动作状态播放，尘土 FX 上无声音。

### 层、端与清理

| 对象 | 本族如何使用 | 边界 | 证据 |
|---|---|---|---|
| 枕头碎屑 | 联网壳复制位置；动画在壳上由主模拟驱动；不挂父级、无 physics | 专用服务器同样生成；`FastForward` 无调用方 | [工厂](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L261-L290) |
| 挖起尘土 | 联网壳＋`animover` 侵蚀（`ErodeAway`）；无 physics | 尘土**不入存档**；`plant_normal` 农作物本体 `persists=false`（播种时设置），读档由农场土壤重建 | [工厂尾](../../../raw/current/scripts/prefabs/plant_normal.lua#L312-L313)、[persists](../../../raw/current/scripts/components/grower.lua#L130) |
| 骑枪／驼鹿尘土 | 复用 small 档注册做"冲锋拖尾"，周期任务按朝向偏移落点、随机缩放与镜像 | 与挖起语义无关，仅复用视觉资产 | [OnStartJoust](../../../raw/current/scripts/prefabs/player_common_extensions.lua#L1170-L1196)、[tackle_start](../../../raw/current/scripts/stategraphs/SGwilson.lua#L22085-L22114) |

## 典型接入

### 给零伤害武器配命中表现

- 前提与选择：`weapon:SetOnAttack` 回调在主模拟生成联网壳并手动落位；想换表现只替换注册名与偏移，不做父挂。
- 生效与撤销：FX 自删，无需清理；命中声与击退由武器回调另行处理，不要在 FX 上找这些结算。

```lua
local function OnWeaponAttack(inst, attacker, target)
    if not target then return end
    local fx = SpawnPrefab("attackfx_handpillow_petals") -- 替换点：注册名
    local offset = (target.isplayer and Vector3(0, 1.0, 0)) or Vector3(0, 2.5, 0)
    fx.Transform:SetPosition((target:GetPosition() + offset):Get())
    target:PushEvent("attacked", { attacker = attacker, weapon = inst, damage = 0 })
end
```

- 调整与验证：原版对应 [onweaponattack](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L153-L176)；小游戏击退判断与 `pillow_common.DoKnockback` 需自行复用。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：枕头 FX 会"结算击退"或"扣血"。** 碎屑是纯动画壳：伤害恒 0，击退在 `pillow_common.DoKnockback` 推 `knockback` 事件后由 SG 完成，且只对枕头大战小游戏参与者生效；对普通目标只推零伤害 `attacked`。[分流](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L165-L174)
- **边界：4 个枕头 FX 与 4 个材质一一对应且靠动态拼名命中。** 手持枕实体把 `"attackfx_handpillow_"..materialname` 写进自己的 prefabs 依赖（[L134-L139](../../../raw/current/scripts/prefabs/pillow_equipment.lua#L134-L139)）；模组新增材质必须同时注册同名 FX，否则命中时 `SpawnPrefab` 返回 nil、下一行 `fx.Transform` 直接报错。
- **边界：`plant_dug_medium_fx`／`_large_fx` 只有挖起一条生成链。** 冲锋拖尾类清单外复用只取 small 档；按 `"plant_dug_"` 前缀检索时注意其余两档没有宿主外生成点。[反查记录](#验证记录)
- **边界：挖起 FX 的叶片符号在"农作物＋medium/large"时被覆盖。** `SpawnDugFX` 只对非 small 且宿主非 `plant_normal_ground` 的情形 override `leaves`；直接 `SpawnPrefab` 得到的是野生作物叶子，需要自己补 override（[覆盖](../../../raw/current/scripts/prefabs/plant_normal.lua#L22-L28)）。
- **误区：挖起会"收获作物"。** 只有成熟/枯萎宿主的 DIG 才经 `Harvest` 出产物；生长中宿主被铲是纯移除（产物不落地）。FX 档位反映的是生长百分比而非收获结果。[裁决表](#挖起裁决链与档位选择)
- **边界：`plant_normal_ground`（野生作物）在 scripts 快照内无生成点。** 依赖声明与注释路径都不是生成；引用它时其世界生成来源在快照外数据，本页不做生成点断言。[检索记录](#验证记录)

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `pillow_equipment.lua`、`pillow_defs.lua`、`pillow_common.lua`、`plant_normal.lua`；按需读取 `veggies.lua` 播种段、`seeds.lua` 依赖表、`components/grower.lua` 播种与 RemoveCrop、`components/crop.lua` Harvest、`player_common_extensions.lua` Jousting 段、`SGwilson.lua` attack_pillow_pre／tackle_start 段、`cozy_bunnyman.lua` 小游戏注册、`cozy_bunnymanbrain.lua` 攻击延迟段、`yotr_fightring.lua` 奖品结算段、`recipes.lua` 枕头配方、`tuning.lua` 相关调值。全 `raw/current/scripts` 反查（`grep -rn "注册名" --include=*.lua`）：`attackfx_handpillow` 命中 pillow_equipment.lua 依赖（L136）、拼名生成（L161）、注册（L289）共 3 行，**生成点唯一**；`plant_dug_` 命中 plant_normal.lua 依赖（L14-L16）、拼名生成（L23）、注册（L318）＋清单外 `player_common_extensions.lua:1183`、`SGwilson.lua:22103`、`woodie.lua:59` 依赖——medium/large 无宿主外生成点；`plant_normal_ground` 全域命中 plant_normal.lua 工厂、seeds.lua:14／veggies.lua:178 依赖与 veggies.lua:196 注释块，无活的生成语句。清单外发现 `pillowfight_confetti_fx`（fx.lua:3113 通用表条目）已辨析不入覆盖。未运行游戏。
- 名称核验：`languages/chinese_s.po` 定向检索——`HANDPILLOW_PETALS`＝"花朵枕头"（[L343725-L343727](../../../raw/current/scripts/languages/chinese_s.po#L343725-L343727)）、`HANDPILLOW_KELP`＝"海带枕头"（[L343720-L343722](../../../raw/current/scripts/languages/chinese_s.po#L343720-L343722)）、`HANDPILLOW_BEEFALOWOOL`＝"皮弗娄牛毛枕头"（[L343715-L343717](../../../raw/current/scripts/languages/chinese_s.po#L343715-L343717)）、`HANDPILLOW_STEELWOOL`＝"钢丝绵枕头"（[L343730-L343732](../../../raw/current/scripts/languages/chinese_s.po#L343730-L343732)）、`BODYPILLOW_PETALS`＝"花朵枕头盔甲"（[L339815-L339817](../../../raw/current/scripts/languages/chinese_s.po#L339815-L339817)）、`BODYPILLOW_KELP`＝"海带枕头盔甲"（[L339810-L339812](../../../raw/current/scripts/languages/chinese_s.po#L339810-L339812)）、`BODYPILLOW_BEEFALOWOOL`＝"皮弗娄牛枕头盔甲"（[L339805-L339807](../../../raw/current/scripts/languages/chinese_s.po#L339805-L339807)）、`BODYPILLOW_STEELWOOL`＝"钢丝绵枕头盔甲"（[L339820-L339822](../../../raw/current/scripts/languages/chinese_s.po#L339820-L339822)）、`PLANT_NORMAL`＝"农作物"（[L346999-L347002](../../../raw/current/scripts/languages/chinese_s.po#L346999-L347002)）、`PLANT_NORMAL_GROUND`＝"野生作物"（[L347004-L347007](../../../raw/current/scripts/languages/chinese_s.po#L347004-L347007)）、`YOTH_LANCE`＝"冲锋骑枪"（[L353024-L353027](../../../raw/current/scripts/languages/chinese_s.po#L353024-L353027)）；`ATTACKFX_HANDPILLOW`／`PLANT_DUG` 在该 po 全文 0 命中，正文使用已核验效果描述。
- 分档说明：`size_class: complex`——7 注册分属 2 个工厂文件与 2 条互不相干宿主链（枕头链横跨武器回调／`pillow_common` 击退／小游戏与奖品结算；挖起链横跨 workable／crop／grower 三组件并另有骑枪与驼鹿两处清单外生成点），混合族需要整体对照，超出 standard 单页容量。
- 未覆盖：动画包内容与帧长、`pillow_hit_<材质>` 四条音效差异、枕头大战场地玩法全貌（yotr_fightring 计分规则、兔人 AI 行为树）、`knockback` 事件在各 SG 的位移实现、农耕生长数值、骑枪冲锋动作全链（`joustsource` 组件细节）；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见本页交付记录。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/farm_plant_happy|作物表情与活动彩纸]]（同含 `pillowfight_confetti_fx` 枕头战出界彩纸与作物压力表情）
- [[reference/components/minigame|小游戏组件]]（枕头大战场地的玩法载体；参与资格判定即 FX 击退分流条件）
- [[reference/components/workable|workable 组件]]（DIG 完成回调与本体移除的宿主协议）
- 返回 [[reference/prefabs/fx-world-index|特效 Prefab 索引·环境与世界]]
