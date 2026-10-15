---
title: "船炮与迫击炮弹命中表现（cannonball_used）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "fx.lua 通用表 4 注册：船炮炮弹命中落点动画 cannonball_used 与蟹王迫击炮弹 mortarball_used/_wood/_ice（同动画，按命中面配三种落点声）；溅射伤害与漏水结算归投射物 OnHit，本族纯表现。"
aliases: ["cannonball_used", "mortarball_used", "mortarball_used_wood", "mortarball_used_ice", "炮弹命中特效", "船炮命中表现", "迫击炮命中表现", "蟹王炮塔命中"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/fx.lua", "prefabs/cannonballs.lua", "prefabs/mortarball.lua", "prefabs/crabking_cannontower.lua", "prefabs/crabking.lua", "prefabs/boat_cannon.lua", "prefabs/boat.lua", "prefabs/world.lua", "stategraphs/SGboatcannon.lua", "stategraphs/SGpowdermonkey.lua", "actions.lua", "components/boatcannon.lua", "components/complexprojectile.lua", "components/map.lua", "entityscript.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 船炮与迫击炮弹命中表现（cannonball_used）

## 快速判断

- 职责与适用：`fx.lua` 通用表 4 注册——船炮石弹命中落点动画 `cannonball_used`，与蟹王战加农炮塔（bank `crab_king_mortar`，下称迫击炮）发射的 `mortarball_used`／`_wood`／`_ice` 三变体；四条目同 bank/build/anim，唯一差异是音效字段。纯表现：溅射、船体扣血、漏水、码头损伤、物品炸飞全在投射物 OnHit 结算。协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目组](../../../raw/current/scripts/fx.lua#L2822-L2847)
- 优先使用：查"炮弹打中后什么表现／命中声怎么选"读本页；炮弹本体与发射链见下文导航；船炮瞄准圈是另一体系（本地非联网指示），见 [[reference/prefabs/cannon_aoe_range_fx|船炮瞄准范围圈]]。
- 重要副作用：无玩法作用；无 fn、无 inspectable，`cannonball_used` 条目无 sound 字段，连 SoundEmitter 都不挂。[音效分支](../../../raw/current/scripts/prefabs/fx.lua#L43-L50)
- 修改边界：服务端生成代理，各非专服一帧后建本地表现，animover 自删；落点式（生成方不父挂代理）。anim `used` 的内容与时长未核验。[回收](../../../raw/current/scripts/prefabs/fx.lua#L98-L102)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `cannonball_used` | 通用表字面量条目 → 加载器打包代理＋一帧后本地实体（公共协议，无专属 fn） | bank/build `cannonball_rock` 播 `used`；**无 sound**，全族唯一无声。[条目](../../../raw/current/scripts/fx.lua#L2822-L2826) |
| `mortarball_used` | 同上协议 | 同动画；声 `meta4/mortars/cannonball_hit`。[条目](../../../raw/current/scripts/fx.lua#L2828-L2833) |
| `mortarball_used_wood` | 同上协议 | 同动画；声 `meta4/mortars/cannonball_hit_wood`。语义由生成方选名决定，非名称自带。[条目](../../../raw/current/scripts/fx.lua#L2835-L2840) |
| `mortarball_used_ice` | 同上协议 | 同动画；声 `meta4/mortars/cannonball_hit_ice`。[条目](../../../raw/current/scripts/fx.lua#L2842-L2847) |

languages 目录全部 .po 大小写不敏感定向检索 4 个注册名：无显示名键（其余命中仅为 `CANNONBALL_ROCK(_ITEM)` 相关的名称与描述条目，均非本族四名）；本页名称依据为简中 `chinese_s.po`，标题为已核验效果描述，非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 4 名字面量条目，公共代理／本地两段式，落点式不父挂 | 全族同协议 | [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]] |
| 生成者：船炮炮弹命中 | `cannonball_rock` 的 OnHit：真落海（`IsOnOcean()` 真）改生成 [[reference/prefabs/crab_king_bubble1|crab_king_waterspout（蟹王伴随表现族）]]，本族不出；否则（地面、船甲板等）落点生成 `cannonball_used` 并对命中点码头扣血 | 服务端；船甲板按"地面"处理（命中点有 walkableplatform 实体时 `IsOceanAtPoint` 为假） | [落地分支](../../../raw/current/scripts/prefabs/cannonballs.lua#L127-L139)、[IsOceanAtPoint](../../../raw/current/scripts/components/map.lua#L105-L109) |
| 生成者：船炮炮弹撞墙 | 飞行 OnUpdate 撞上**排除发射者所在船**的墙（陆地墙与其他船的墙均可）且墙有血未死：范围攻击后落点生成 `cannonball_used` 并移除炮弹 | 服务端；cannonball 已 `SetOnUpdate` 安装该路径 | [撞墙分支](../../../raw/current/scripts/prefabs/cannonballs.lua#L161-L169)、[接线](../../../raw/current/scripts/prefabs/cannonballs.lua#L265-L266) |
| 生成者：迫击炮弹命中 | `mortarball` 的 OnHit 落地分支按局部变量 `hittype` 动态选名落点生成；真落海同样只出 `crab_king_waterspout` | 服务端；本快照内 mortarball 唯一发射者是蟹王迫击炮塔 `LaunchProjectile`（装 redgemcount、按距离调速度） | [hittype](../../../raw/current/scripts/prefabs/mortarball.lua#L58-L58)、[落地选名](../../../raw/current/scripts/prefabs/mortarball.lua#L145-L163)、[发射](../../../raw/current/scripts/prefabs/crabking_cannontower.lua#L85-L124) |
| 选名逻辑（变体语义） | 默认 `mortarball_used`；命中 `HasTag("wood")` 的船 → `_wood`；落点瓦片为 `OCEAN_ICE` → `_ice`；`dockmanager` 存在 → **无条件覆写** `_wood`（无论该处是否真有码头） | 判定有顺序：冰面先、码头后（码头覆盖冰面）。原版主世界实体 master 分支恒挂 dockmanager → 原版下落地最终恒 `_wood`；plain 与 `_ice` 仅 dockmanager 被显式移除（模组 `RemoveComponent("dockmanager")` 或不经 MakeWorld 的自定义世界）时可达 | [船分支](../../../raw/current/scripts/prefabs/mortarball.lua#L70-L87)、[冰面](../../../raw/current/scripts/prefabs/mortarball.lua#L150-L152)、[码头覆写](../../../raw/current/scripts/prefabs/mortarball.lua#L154-L159)、[dockmanager 注册](../../../raw/current/scripts/prefabs/world.lua#L585-L585)、[wood 标签](../../../raw/current/scripts/prefabs/boat.lua#L384-L384) |
| 触发窗口 | complexprojectile 下落至 y≤0.05 触发 `Hit()`（无 target 实参，OnHit 内自行 `GetPlatformAtPoint` 找船） | 服务端组件 | [Hit 与 OnUpdate](../../../raw/current/scripts/components/complexprojectile.lua#L191-L210)、[[reference/components/complexprojectile]] |
| 发射链（船炮侧） | 装填三路（玩家 LoadAmmo 动作、船炮授物、猴子 SG 直装 `cannonball_rock`）→ SGboatcannon 7 帧调 `Shoot` → `SpawnPrefab(loadedammo)` 发射 | 服务端；蟹王本体另用同 prefab 直接 Launch | [动作装填](../../../raw/current/scripts/actions.lua#L5055-L5055)、[授物装填](../../../raw/current/scripts/prefabs/boat_cannon.lua#L129-L137)、[猴子装填](../../../raw/current/scripts/stategraphs/SGpowdermonkey.lua#L114-L115)、[Shoot](../../../raw/current/scripts/stategraphs/SGboatcannon.lua#L79-L81)、[boatcannon 组件](../../../raw/current/scripts/components/boatcannon.lua#L83-L111)、[蟹王](../../../raw/current/scripts/prefabs/crabking.lua#L1529-L1556) |
| 兄弟表现与伤害归属 | 同一 OnHit 内还有：DoAreaAttack 溅射、船体 `DoDelta(-伤害/2)`、`spawnnewboatleak` 事件、鱼掉落 launch_away、`DamageDockAtPoint` | 溅射／船血／漏水先于本族生成；船炮的 `DamageDockAtPoint` 在本族生成之后、`inst:Remove()` 之前；本族不承载结算，生成调用本身可移除或替换，但其后的码头伤害与 `inst:Remove()` 顺序必须保留 | [cannonball OnHit](../../../raw/current/scripts/prefabs/cannonballs.lua#L53-L140)、[mortarball OnHit](../../../raw/current/scripts/prefabs/mortarball.lua#L56-L164)、[[reference/components/dockmanager]] |
| 资源 | bank/build `cannonball_rock`（与炮弹本体共用 build）anim `used`；三声 `meta4/mortars/*` | anim zip 内容未核验 | [条目组](../../../raw/current/scripts/fx.lua#L2822-L2847) |

## 典型接入

### 自定义投射物复用命中落点表现

- 前提与选择：服务端投射物 OnHit 内 `SpawnPrefab` 落点式生成；可仿 mortarball 用局部变量按命中面选名。注册名需经依赖声明或世界预载保证可生成（[[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]）。
- 生效与撤销：一次性动画，animover 自删；无宿主清理，投射物随后的 Remove 不影响表现。
- 原版依据：[落地分支](../../../raw/current/scripts/prefabs/cannonballs.lua#L127-L139)。

```lua
local function OnHit(inst, attacker, target)
    if inst:IsOnOcean() then
        SpawnPrefab("crab_king_waterspout").Transform:SetPosition(inst.Transform:GetWorldPosition())
    else
        -- 落地：无声命中动画；要配声可改用 mortarball_used 系注册
        SpawnPrefab("cannonball_used").Transform:SetPosition(inst.Transform:GetWorldPosition())
    end
    inst:Remove()
end
```

- 调整与验证：`mortarball_used_wood`／`_ice` 只是带声的同款动画，按命中面直接选名即可；伤害结算留在生成者，不要塞进 FX。

## 常见陷阱与限制

- **边界：mortarball.lua 里的 `cannonball_used` 生成点是死代码**：其 OnUpdateProjectile 撞墙分支写有 `SpawnPrefab("cannonball_used")`，但 `mortar_fn` 未安装 OnUpdate（该行被注释；组件 `onupdatefn` 默认 nil，仅非 nil 才被调用）。`cannonball_used` 的实际生成点只在 cannonballs.lua 两处。[定义](../../../raw/current/scripts/prefabs/mortarball.lua#L166-L194)、[未安装](../../../raw/current/scripts/prefabs/mortarball.lua#L299-L299)、[默认值](../../../raw/current/scripts/components/complexprojectile.lua#L17-L17)
- **误区：按 `_wood`／`_ice` 名推"木船／冰面专属动画"**：三变体动画完全相同，差异只在音效事件；且原版世界 dockmanager 恒存在使落地最终恒选 `_wood`——名为默认的 `mortarball_used` 与 `_ice` 在原版主世界反而不可达（检索范围与选名证据见上文导航）。[覆写顺序](../../../raw/current/scripts/prefabs/mortarball.lua#L150-L159)
- **边界：船分支的 `_wood` 只在继续走到落地分支时生效**：真落海（无平台海面）只出 `crab_king_waterspout`，`hittype` 被丢弃。[分支次序](../../../raw/current/scripts/prefabs/mortarball.lua#L70-L87)
- **边界：表现与伤害归属分离，删注册前需处理生成调用**：伤害由投射物结算；替换成仍返回有效实体／Transform 的纯表现，不影响溅射、船血、漏水与码头伤害。船炮落地分支在 `SpawnPrefab("cannonball_used")` 之后才调 `DamageDockAtPoint`；若只删注册、保留未判空的 `.Transform` 访问，生成返回 nil 会中断后续码头伤害与 `Remove`。要关闭效果，应移除或替换生成调用并保留后续结算与清理顺序；迫击炮也仍在 FX 之后移除炮弹。[船炮落地](../../../raw/current/scripts/prefabs/cannonballs.lua#L127-L139)、[迫击炮](../../../raw/current/scripts/prefabs/mortarball.lua#L154-L163)
- **边界：`IsOnOcean` 的平台语义**：默认 `allow_boats=false`，命中点存在 walkableplatform 实体（船）时按"地面"处理——炮弹落在船甲板走落地分支而非落水分支。[map.lua](../../../raw/current/scripts/components/map.lua#L105-L109)、[IsOnOcean](../../../raw/current/scripts/entityscript.lua#L1848-L1851)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。4 条目逐字段比对（fx.lua L2821-2847）；加载器 prefabs/fx.lua 的音效挂接与 animover 回收分支；消费者 cannonballs.lua／mortarball.lua OnHit 与 OnUpdate 全文、complexprojectile 触发路径与默认值、发射链（LoadAmmo 动作、boat_cannon 授物、SGpowdermonkey、SGboatcannon、boatcannon 组件、crabking、crabking_cannontower）、选名 helper（map.lua IsOceanAtPoint／IsOceanIceAtPoint／GetPlatformAtPoint、entityscript IsOnOcean、boat.lua wood 标签与 walkableplatform、world.lua dockmanager 注册）逐条核验。生成点反查：`cannonball_used|mortarball_used` 在本快照 raw/current/scripts 全域（根级＋prefabs／components／stategraphs）检索，命中仅注册表与上述两消费者（含 hittype 动态拼名）；并核对 `SpawnPrefab` 返回值未判空与 `DamageDockAtPoint` 的先后顺序，确认删注册可能中断后续结算。未覆盖：anim `used` 的动画内容与时长（二进制资源）、MONKEY_DOCK 瓦片在 TileGroupManager 的海瓦片归类（只影响码头伤害分支可达性，不影响选名结论）、专服与暂停表现未实测。
- 游戏运行：未运行。
- 结构检查：validate_page.py 与 validate_source_links.py（--strict）通过。

相关：[[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]、[[reference/prefabs/fx-table-index|通用表 FX 索引]]、[[reference/prefabs/cannon_aoe_range_fx|船炮瞄准范围圈]]、[[reference/components/complexprojectile]]、[[reference/components/boatcannon]]。

返回 [[reference/prefabs/fx-table-index|通用表 FX 索引]]、[[reference/prefabs/index|Prefab 索引]]。
