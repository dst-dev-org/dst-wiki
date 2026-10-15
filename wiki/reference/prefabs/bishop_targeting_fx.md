---
title: "发条主教预警瞄准圈（bishop_targeting_fx）"
created: "2026-11-05"
updated: "2026-11-05"
type: prefab
summary: "发条主教蓄力攻击的联网预警圈：地面 reticule 动画＋淡黄光，SGbishop attack 状态生成后逐帧推位置／朝向／距离（net_float dist 同步），各端据 dist 在瞄准线上挂本地光斑，1 秒后 KillFx 转 pst 自删、开火交给 bishop_charge2_fx 结算。"
aliases: ["bishop_targeting_fx", "发条主教瞄准圈", "主教预警圈", "主教攻击预警", "主教蓄力瞄准", "电击预警圈"]
tags: [prefab, fx, creature, server-client]
size_class: standard
dst_build: "steam-buildid:26335394"
sources: ["prefabs/bishop.lua", "stategraphs/SGbishop.lua", "prefabs/bishop_charge.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 发条主教预警瞄准圈（bishop_targeting_fx）

## 快速判断

- 职责与适用：`bishop_targeting_fx` 是发条主教（bishop／bishop_nightmare 两形态共用）蓄力射击前 1 秒的**联网预警圈**——贴地 reticule 动画＋淡黄点光，指示这次电击将命中的落点；位置、朝向、与主教的距离都由宿主状态机逐帧推给它，玩家可用这 1 秒走位躲弹（[注册](../../../raw/current/scripts/prefabs/bishop.lua#L445)）。
- 本体辨析：`bishop`（官方名"发条主教"，[po](../../../raw/current/scripts/languages/chinese_s.po#L339485-L339487)）与 `bishop_nightmare`（官方名"损坏的发条主教"，[po](../../../raw/current/scripts/languages/chinese_s.po#L339490-L339492)）是同一 `MakeBishop` 包装工厂的两个注册（build／掉落／酸浸差异）；两者的攻击伤害不在主教本体直接结算，而是开火帧生成 `bishop_charge2_fx`（ bishop_charge.lua 注册）按 AOE 电击结算——见[本体侧辨析](#本体侧辨析与攻击链分工)。
- 重要副作用：FX 自带 Light（半径 0.6）且专服也参与光照；除动画与光外**零玩法组件**——伤害、击退、电击状态全在 `bishop_charge2_fx` 与宿主 combat 上。
- 修改边界：只覆盖 bishop_targeting_fx 的工厂、生成点、端与清理；主教攻击状态机其余状态、`bishop_charge2_fx` 内部结算、已弃用的 `bishop_charge` 弹体不展开。
- 名称：`BISHOP_TARGETING_FX` 在简中 `chinese_s.po` 无 `STRINGS.NAMES` 键（检索无命中），标题中文为已核验效果描述，非官方译名。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `bishop_targeting_fx` | `reticulefn`：Transform／AnimState／**Light**／Network → bank=build=`bishop`、播 `reticule_pre` → 泛光＋贴地（OnGround）＋背景层 SortOrder 3＋整体 alpha 0.3 → Light r0.6／i0.4／falloff 0.625／色 (255,255,236)/255 → 标签 `FX`＋`NOCLICK`（无 NOBLOCK）→ 共享段 `net_float dist`（`bishop_targeting_fx.dist`，dirty 事件 `distdirty`）→ SetPristine → 客户端监听 `distdirty` → 主模拟接 `PushAnimation("reticule_loop")` 循环、`persists=false`、暴露 `SetDistFromBishop`／`KillFx` 协议方法 | 客户端动画经网络实体同步获得同样的 pre→loop 序列；Light 数值随实体同步、各端可见；[工厂](../../../raw/current/scripts/prefabs/bishop.lua#L397-L437)、[动画配置](../../../raw/current/scripts/prefabs/bishop.lua#L405-L413)、[灯光](../../../raw/current/scripts/prefabs/bishop.lua#L415-L418)、[net 变量](../../../raw/current/scripts/prefabs/bishop.lua#L423) |

## 关联机制导航

### 生成与驱动链（SGbishop attack 状态）

| 环节 | 行为与条件 | 证据 |
|---|---|---|
| 进入攻击 | bishopbrain 的 doattack 事件 → SG `attack` 状态；onenter 先 KillFx 旧圈，再把目标点按 **3-12 距离钳制**（`MIN/MAX_TGT_RANGE` 硬编码）换算，跟踪速度 0.5 | [doattack](../../../raw/current/scripts/stategraphs/SGbishop.lua#L17-L21)、[范围常量](../../../raw/current/scripts/stategraphs/SGbishop.lua#L24-L27)、[目标点换算](../../../raw/current/scripts/stategraphs/SGbishop.lua#L298-L303) |
| **生成** | 攻击时间线第 6 帧置 `starttargeting` 标记，onupdate 消费标记 **SpawnPrefab 一次**（每次攻击至多一个圈） | [第 6 帧](../../../raw/current/scripts/stategraphs/SGbishop.lua#L368-L370)、[生成](../../../raw/current/scripts/stategraphs/SGbishop.lua#L346) |
| 逐帧驱动 | onupdate 每帧：目标点向真实目标位置 lerp（跟踪速度每帧 -0.02 至 0 停跟）→ `SetPosition/SetRotation`（朝向＝主教→落点）→ `SetDistFromBishop(实际距离)` | [逐帧驱动](../../../raw/current/scripts/stategraphs/SGbishop.lua#L348-L351) |
| 开火收尾 | ontimeout（1 秒）：`KillFx` 后进 `attack_shoot` 状态开火 | [ontimeout](../../../raw/current/scripts/stategraphs/SGbishop.lua#L355-L361) |
| 打断收尾 | 状态被打断走 onexit：**直接 Remove 不播 pst** | [onexit](../../../raw/current/scripts/stategraphs/SGbishop.lua#L374-L376) |

### dist 网络同步与本地光斑

- `SetDistFromBishop`（master 协议方法）只在值变化时 `dist:set()`，并**立即在主模拟本地也执行一次** dirty 处理——源码注释说明专服需要：光是实体级渲染量，专服也要参与光照计算（[SetDistFromBishop](../../../raw/current/scripts/prefabs/bishop.lua#L381-L388)）。
- 客户端由 `distdirty` 事件进同一处理（[客户端监听](../../../raw/current/scripts/prefabs/bishop.lua#L427-L428)）：`dist > 0` 时惰性创建**本地光斑** `LightTail`——非联网实体（Transform＋Light，r0.5／i0.3／falloff 0.4／同色 (255,255,236)），SetParent 到 FX 并摆放在瞄准线上：局部 (-x, 0, 0)，x 由 dist 3→12 二次缓动映射到 1→2，即光斑沿瞄准线向主教一侧回退、目标越远越靠后（[dirty 处理](../../../raw/current/scripts/prefabs/bishop.lua#L364-L379)、[光斑工厂](../../../raw/current/scripts/prefabs/bishop.lua#L344-L362)）。`dist = 0` 时移除光斑。
- 每台机器各自持有一份光斑实例（代理只同步 dist 数值，光斑本身不过网络）。

### 退场协议（KillFx）

`KillFx` 三步：`SetDistFromBishop(0)` 摘光斑 → 播 `reticule_pst` → `animover` 自删，另有 `OnEntitySleep` 兜底 Remove（[KillFx](../../../raw/current/scripts/prefabs/bishop.lua#L390-L395)）。正常路径只有 attack ontimeout 这一个调用方；被打断的路径不走 pst（见上表）。宿主 bishop 的 `OnRemoveEntity` 也会兜删 `targetingfx` 单槽引用（[宿主清理](../../../raw/current/scripts/prefabs/bishop.lua#L183-L188)）。

### 本体侧辨析与攻击链分工

- `bishop`／`bishop_nightmare` 由 `MakeBishop` 同工厂产出（装备位差异：普通版 build `bishop_build`、噩梦版 build `bishop_nightmare`＋`acidinfusible`）。attack 状态蓄力 1 秒只完成预警，**真正开火在 `attack_shoot`**：第 1 帧在落点生成 `bishop_charge2_fx`（注册于 [bishop_charge.lua:292](../../../raw/current/scripts/prefabs/bishop_charge.lua#L292)、生成点 [SGbishop.lua:402](../../../raw/current/scripts/stategraphs/SGbishop.lua#L402)）并 `StartShotFx` 把落点写进主教本体的 `shotx/shotz/showshot` net 变量（[StartShotFx](../../../raw/current/scripts/prefabs/bishop.lua#L173-L181)）。
- `bishop_charge2_fx` 是**混合载体**：自身 AnimState＋Light 表现落点电爆，同时借施放者 combat 做 AOE 电击结算（绝缘目标按 `BISHOP_INSULATED_DAMAGE_MULT` 减伤）；伤害归属见 bishop_charge.lua 的 OnUpdate 段——本页不展开。
- 弹道光束的绘制是主教本体工厂内的**本地非注册实体**：客户端收 `showshot` dirty 后用 `CreateShotSegFx` 在主教 `shot_marker` 与落点之间排随机帧的光段（[OnShowShotDirty](../../../raw/current/scripts/prefabs/bishop.lua#L164-L171)、[光段工厂](../../../raw/current/scripts/prefabs/bishop.lua#L68-L91)）——不在本页覆盖清单，列出仅为划清边界。
- 同文件还有已弃用的 `bishop_charge`／`bishop_charge_hit` 注册（旧弹体方案，[弃用标注](../../../raw/current/scripts/prefabs/bishop_charge.lua#L289-L290)），全域反查无活引用。

### 清理路径

| 路径 | 方式 | 边界 |
|---|---|---|
| 正常开火 | ontimeout → KillFx → pst → animover 自删（[唯一 KillFx 调用方](../../../raw/current/scripts/stategraphs/SGbishop.lua#L356-L358)） | 光斑随 dist=0 先摘 |
| 攻击被打断 | attack onexit 直接 Remove（无 pst） | 下一轮 attack onenter 也会先 KillFx 残留 |
| 宿主死亡/移除 | bishop `OnRemoveEntity` 兜删单槽引用，引擎级联 | FX 实体非宿主子体，靠显式引用清理 |

## 典型接入

### 复刻"怪物侧联网预警圈＋各端光斑"

- 前提与选择：Boss 技能需要给玩家 1 秒左右的落点预警时，用一个网络化 FX 承载贴地动画＋点光；可变量（如与施放者距离）用 `net_float` 同步，纯视觉装饰（光斑）各端按 dirty 事件自建非联网子体。
- 生效与撤销：施放侧状态机持有单槽引用逐帧驱动；到期回调调协议 KillFx（摘装饰→播 pst→animover 自删）；打断路径直接 Remove，下次施放前先 KillFx 旧引用。

```lua
-- 状态机侧（对应 SGbishop attack 状态的核心三步）
inst.targetingfx = SpawnPrefab("my_targeting_fx")        -- 第 N 帧生成一次
inst.targetingfx.Transform:SetPosition(tx, 0, tz)        -- 每帧驱动
inst.targetingfx:SetDistFromBishop(dist)                 -- 同步距离（驱动各端光斑）
inst.targetingfx:KillFx()                                -- 超时：摘光斑＋播 pst 自删
```

- 调整与验证：替换点是动画名与 dist 语义；dist 的 dirty 事件名与 FX 注册名一一对应（`"<prefab>.dist"`），跨注册复用要同步改名。示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **边界：圈的位置不会自己跟目标。** 生成后的位置、朝向、距离全部依赖宿主 onupdate 逐帧推送（[驱动](../../../raw/current/scripts/stategraphs/SGbishop.lua#L348-L351)）；把它 SpawnPrefab 出来不动，它会停在原地播 loop。
- **边界：dist 同步是"变化才发"。** `SetDistFromBishop` 有 `dist ~= inst.dist:value()` 守卫（[L382](../../../raw/current/scripts/prefabs/bishop.lua#L382)），静止距离下客户端不会再收 dirty——依赖 dirty 重算的逻辑要自己兜初值。
- **边界：专服也必须走 dirty 处理。** 源码在 master 分支显式调用 `OnDistFromBishopDirty` 并注释"Dedicated server needs to do this too because it's a light"（[L385-L386](../../../raw/current/scripts/prefabs/bishop.lua#L385-L386)）；复刻时别把灯光更新写进"仅客户端"分支。
- **边界：打断与超时的退场观感不同。** 超时走 KillFx 有 `reticule_pst` 淡出；被打断直接 Remove 无淡出（[onexit](../../../raw/current/scripts/stategraphs/SGbishop.lua#L374-L376)）——不要以为 pst 是必播的。
- **边界：3-12 的落点钳制是 SG 层硬编码**（[L24-L27](../../../raw/current/scripts/stategraphs/SGbishop.lua#L24-L27)），与索敌用的 `TUNING.BISHOP_TARGET_DIST`(12) 是两个独立值；调预警距离要改 SG 常量，不是调 tuning。
- **边界：与本页同文件的光段／电爆不是预警圈的一部分。** 光束段（CreateShotSegFx）是本体工厂本地实体，电爆伤害在 `bishop_charge2_fx`——给预警圈加逻辑不会影响结算。
- **误区：预警圈就是攻击判定范围。** 圈指示落点（半径 2 的 AOE 电击在 bishop_charge2_fx），圈本身无 combat、无碰撞（FX＋NOCLICK、无 NOBLOCK），踩上去不会被电。

## 验证记录

- 源码审阅：2026-11-05，steam-buildid:26335394（实读 `raw/current/BUILD.txt`）。完整读取 `prefabs/bishop.lua`、`prefabs/bishop_charge.lua`、`stategraphs/SGbishop.lua`（events、attack／attack_shoot 状态、GetTargetingXZDir／LerpTargetingXZDir）。对覆盖注册名全域反查：`bishop_targeting_fx` 命中注册（bishop.lua:445）、依赖（:22/:30）、net 名（:423）、生成（SGbishop.lua:346）——**生成点唯一**（onupdate 单槽复用同一实体）。关联反查：`bishop_charge2_fx` 命中注册（bishop_charge.lua:292）、依赖（bishop.lua:24/:32）、生成（SGbishop.lua:402）——生成点唯一；`bishop_charge`／`bishop_charge_hit` 除注册处外全域无引用（源码自标 deprecated）；`showshot`／`shotx`／`shotz` 净变量仅 bishop.lua 定义、SGbishop.lua:403 `StartShotFx` 写入、bishop.lua OnShowShotDirty 消费。台账批 2 结论复核属实（Light＋dist 同步＋reticule 动画）。
- 名称核验：`languages/chinese_s.po` 定向检索——`STRINGS.NAMES.BISHOP`＝"发条主教"、`BISHOP_NIGHTMARE`＝"损坏的发条主教"（锚点见"快速判断"）；`BISHOP_TARGETING_FX` 无 NAMES 键，标题使用效果描述。
- 分档说明：`size_class: standard`——单注册但需承载生成链、dist 双端协议、本地光斑与退场协议、本体攻击链辨析，超出单一职责；未选 simple。
- 未覆盖：SGbishop 其余状态（睡眠／眩晕／ruinsrespawn 等）、`bishop_charge2_fx` 的 AOE 结算与电图层细节、光段（CreateShotSegFx）的帧排布算法、主教 Brain 索敌、噩梦形态废墟重生链、动画包帧内容；未运行游戏。
- 结构检查：`validate_page.py` 与 `validate_source_links.py --strict` 结果见交付记录（0 error 0 warning 后入库）。
- 游戏运行：未运行；`validation` 保持 `source-reviewed`。

## 相关链接

- [[reference/prefabs/reticuleaoe|瞄准圈与 AoE 目标指示]]（玩家侧瞄准圈的本地 reticule 体系——与怪物侧联网预警圈分工对照）
- [[reference/prefabs/shadow_bishop_fx|暗影主教羽群与暗影裂隙散件]]（发条主教被腐蚀转化后的对应体）
- [[reference/prefabs/alterguardian_summon_fx|天体英雄召唤光柱]]（同为"施放前预警演出"的写法对照）
- 返回 [[reference/prefabs/fx-creature-index|特效 Prefab 索引·生物与Boss]]（发条主教）
