---
title: "草／海带船侵蚀与垫料特效（boat_grass_erode 族）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "草船与海獭窝船的沉没侵蚀残影、垫料受击碎屑与水草补丁到期断裂：通用表 7 注册的生成点、触发窗口与端边界。"
aliases: ["boat_grass_erode", "boat_grass_erode_water", "boat_otterden_erode", "boat_otterden_erode_water", "fx_grass_boat_fluff", "fx_kelp_boat_fluff", "repaired_kelp_timeout_fx", "船体侵蚀特效", "垫料碎屑", "草船沉没特效", "海獭窝船沉没", "水草补丁到期"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/boat.lua", "prefabs/cookiecutter.lua", "prefabs/boatpatch.lua", "prefabs/anchor.lua", "prefabs/alterguardian_laser.lua", "prefabs/cannonballs.lua", "prefabs/mortarball.lua", "prefabs/crabking.lua", "prefabs/gnarwail.lua", "prefabs/trident.lua", "prefabs/waterplant_seed.lua", "prefabs/waterpump.lua", "prefabs/winch.lua", "prefabs/fish_box.lua", "stategraphs/SGboat.lua", "stategraphs/SGcookiecutter.lua", "stategraphs/SGcrabking_cannontower.lua", "stategraphs/SGeyeofterror.lua", "stategraphs/SGalterguardian_phase2.lua", "components/boatleak.lua", "components/hullhealth.lua", "components/cookiecutterdrill.lua", "scheduler.lua", "tuning.lua", "debugkeys.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 草／海带船侵蚀与垫料特效（boat_grass_erode 族）

## 快速判断

- 职责与适用：`fx.lua` 通用表中草／海带材质船与水草修补物的 **7 个一次性表现注册**：草船／海獭窝船沉没后的侵蚀残影 4 名（成对生成）、垫料受击碎屑 2 名、水草补丁到期断裂 1 名；全部纯视觉无玩法作用，代理与本地实体分工见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。
- 优先使用：查"草船／海獭窝船沉没残留""啃船草屑／海带屑""水草补丁到期断裂"读本页；沉没开裂／码头／海冰的 crackle-pop 声画在姊妹页 [[reference/prefabs/fx_boat_crackle|海洋平台破坏声画]]；水草补丁状态机正文在 [[reference/components/boatleak]]。
- 重要副作用：erode 对在沉没结算落点生成、不随船，船本体随即 `Remove`；`erode_water` 18 帧淡出；水草补丁到期 fx 落点生成后漏点回 `med_leak` 重新漏水。[postsinkfn](../../../raw/current/scripts/prefabs/boat.lua#L860-L867)、[到期链](../../../raw/current/scripts/components/boatleak.lua#L33-L60)
- 修改边界：生成者全在服务端模拟（SG／组件／postsinkfn），各模拟端按工厂各自建本地表现；不父挂宿主、不入存档；船体自磨碎屑走另一注册 `degrade_fx_grass`（见陷阱）。

## 注册与构造

| 注册名 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `boat_grass_erode` | 数据表条目经 `MakeFx` 注册；fn：缩放 0.75、`OCEAN_BOAT` 排序、FinalOffset 1、OnGround 朝向、`LAYER_BACKGROUND`；`nofaced` | 草船沉没残影，bank/build `boat_grass` 播 `erode`；[条目](../../../raw/current/scripts/fx.lua#L2766-L2778) |
| `boat_grass_erode_water` | 同上 fn，另起 `DoPeriodicTask(0, …)` 每帧 alpha −1/18，18 帧后停在 0；任务从不取消 | 同 bank 播 `erode_water` 的水面余纹淡出层；寿命仍走 animover；[条目](../../../raw/current/scripts/fx.lua#L2780-L2799)、[period 0 取下一 tick](../../../raw/current/scripts/scheduler.lua#L280-L290) |
| `boat_otterden_erode` | 与 `boat_grass_erode` 同 fn、同 `nofaced` | bank/build 换 `boat_otterden`；[条目](../../../raw/current/scripts/fx.lua#L3396-L3408) |
| `boat_otterden_erode_water` | 与 `boat_grass_erode_water` 同 fn（同 18 帧淡出） | 同上换 bank；[条目](../../../raw/current/scripts/fx.lua#L3410-L3429) |
| `fx_grass_boat_fluff` | 无 fn；sound 行被注释，无声；默认四向 | bank/build `fx_portal_items` 播 `grass` 的草屑；[条目](../../../raw/current/scripts/fx.lua#L2899-L2904) |
| `fx_kelp_boat_fluff` | `transform` 三轴 0.75 缩放 + `FinalOffsetNegative1` | bank/build `boat_repair_kelp_fx` 播 `break` 的海带屑；[条目](../../../raw/current/scripts/fx.lua#L3431-L3437) |
| `repaired_kelp_timeout_fx` | `fn = FinalOffset1` | 同 bank 播 `break`、无缩放的水草补丁断裂；[条目](../../../raw/current/scripts/fx.lua#L3389-L3394) |

7 个注册名均无 `nameoverride`／`description` 字段，`languages/chinese_s.po` 定向检索也只有角色台词、无对应显示名键，标题中文为已核验效果描述，**非官方译名**。

## 触发窗口与生成者

### 沉没侵蚀（erode × 4）

- **谁生成**：草船 `postsinkfn`（[boat.lua L860-L867](../../../raw/current/scripts/prefabs/boat.lua#L860-L867)）与海獭窝船 `postsinkfn`（[L1242-L1249](../../../raw/current/scripts/prefabs/boat.lua#L1242-L1249)）各自在船中心连落两枚（先 erode 后 erode_water）。
- **何时**：两条沉没终局路径都调 `postsinkfn`——SGboat `popping` 态（death → ready_to_snap 0.75 秒全船 abandon_ship → snapping → popping，[SGboat.lua L121-L132](../../../raw/current/scripts/stategraphs/SGboat.lua#L121-L132)）与 `InstantlyBreakBoat`（[boat.lua L463-L480](../../../raw/current/scripts/prefabs/boat.lua#L463-L480)；蟹王、漩涡传送门、鲨鱼人竞技场调用）；两条路径都是 `sinkloot → postsinkfn → inst:Remove()` 同步序。
- **清理边界**：fx 不父挂、不随船；船同帧 `Remove` 后残影是独立世界实体，按公共工厂 animover 自删（代理 1 秒回收，`persists=false`）。`erode_water` 的淡出任务不取消，alpha 停 0 后实体仍留到动画播完。
- **检索范围**：`boat_grass_erode`／`boat_otterden_erode*` 在快照 scripts 全域（根级、prefabs/、components/、stategraphs/，排除 languages）仅命中上述生成点与 boat.lua 依赖声明，无其他 SpawnPrefab。

### 垫料碎屑（fluff × 2）

- **草屑路径一（漏点事件）**：草船收到 `spawnnewboatleak` 时，`OnSpawnNewBoatLeak_Grass` 在漏点 (x, 0, z) 落 `fx_grass_boat_fluff` 与伴生 `splash_green_small`，再按漏点等级扣船血（小漏 15／中漏 30，[tuning](../../../raw/current/scripts/tuning.lua#L4187-L4190)）；门禁 `inst.material == "grass"`（[boat.lua L215-L233](../../../raw/current/scripts/prefabs/boat.lua#L215-L233)）。海獭窝船挂同一监听（[L1212](../../../raw/current/scripts/prefabs/boat.lua#L1212)）但 material 为 `"kelp"`，不出草屑；草船自身 `leakproof` 无漏点实体，此事件即草船"漏水"的全部表现。
- **事件源**：全域 `PushEvent("spawnnewboatleak")` 共 17 处，均对带 `boat` 标签的船推事件（草船中炮同样出草屑）——钻船完成 [cookiecutterdrill.lua L48](../../../raw/current/scripts/components/cookiecutterdrill.lua#L48)、抛锚 [anchor.lua L36](../../../raw/current/scripts/prefabs/anchor.lua#L36)、天体激光 [alterguardian_laser.lua L222](../../../raw/current/scripts/prefabs/alterguardian_laser.lua#L222)、船炮弹 [cannonballs.lua L67](../../../raw/current/scripts/prefabs/cannonballs.lua#L67)、迫击炮弹 [mortarball.lua L78](../../../raw/current/scripts/prefabs/mortarball.lua#L78)、蟹王本体 [crabking.lua L1851](../../../raw/current/scripts/prefabs/crabking.lua#L1851) 与炮塔 [SGcrabking_cannontower.lua L251](../../../raw/current/scripts/stategraphs/SGcrabking_cannontower.lua#L251)、恐怖之眼 [SGeyeofterror.lua L915](../../../raw/current/scripts/stategraphs/SGeyeofterror.lua#L915)、天体二阶段 [SGalterguardian_phase2.lua L712](../../../raw/current/scripts/stategraphs/SGalterguardian_phase2.lua#L712)、角鲸 [gnarwail.lua L51](../../../raw/current/scripts/prefabs/gnarwail.lua#L51)、三叉戟 [trident.lua L171](../../../raw/current/scripts/prefabs/trident.lua#L171)、[L175](../../../raw/current/scripts/prefabs/trident.lua#L175)、水草种弹 [waterplant_seed.lua L70](../../../raw/current/scripts/prefabs/waterplant_seed.lua#L70)、水泵 [waterpump.lua L29](../../../raw/current/scripts/prefabs/waterpump.lua#L29)、绞盘 [winch.lua L67](../../../raw/current/scripts/prefabs/winch.lua#L67)、鱼箱 [fish_box.lua L93](../../../raw/current/scripts/prefabs/fish_box.lua#L93)、调试键 [debugkeys.lua L504](../../../raw/current/scripts/debugkeys.lua#L504)。
- **草屑路径二／海带屑（钻船中）**：SGcookiecutter `drill` 态按目标船 `material` 选名并以 `DoPeriodicTask(0.3, …)` 周期落点——草船出 `fx_grass_boat_fluff`（±0.25 随机偏移）、kelp 船出 `fx_kelp_boat_fluff`（无偏移）（[SGcookiecutter.lua L560-L573](../../../raw/current/scripts/stategraphs/SGcookiecutter.lua#L560-L573)）。
- **检索范围**：`fx_grass_boat_fluff` 全域仅两处 SpawnPrefab（漏点事件＋drill）与 boat.lua 两处依赖声明；`fx_kelp_boat_fluff` 仅 drill 一处与 cookiecutter.lua 依赖声明。

### 水草补丁到期（repaired_kelp_timeout_fx）

- **动态拼名**：boatleak 按 `SpawnPrefab(current_state.."_timeout_fx")` 生成（[boatleak.lua L38](../../../raw/current/scripts/components/boatleak.lua#L38)）；`SetRepairedTime` 由 `repaired_kelp` 分支、动态 leak 存档恢复（boatleak OnLoad）、碰撞 leak 读档（`HullHealth:LoadPostPass`）与 `LongUpdate` 重排调用（[L251-L257](../../../raw/current/scripts/components/boatleak.lua#L251-L257)、[L306-L319](../../../raw/current/scripts/components/boatleak.lua#L306-L319)、[hullhealth L272-L288](../../../raw/current/scripts/components/hullhealth.lua#L272-L288)、[LongUpdate L323-L337](../../../raw/current/scripts/components/boatleak.lua#L323-L337)），快照内实际拼出的只有本名。
- **链条（正常定时到期）**：REPAIR_LEAK 动作用水草补丁（`boatpatch_kelp` 设 `patch_type = "kelp"`，[boatpatch.lua L109](../../../raw/current/scripts/prefabs/boatpatch.lua#L109)）→ `BoatLeak:Repair` → `SetState("repaired_kelp")` 挂到期任务（中漏 0.5 天／小漏 0.75 天，[tuning](../../../raw/current/scripts/tuning.lua#L8255-L8256)）→ 到期漏点播 `pre_timeout`，动画长 +2 帧后在漏点位置落本 fx，漏点回 `med_leak`。**LongUpdate 补算到期**是另一条直接路径：剩余时间大于零时重排任务，否则跳过 `pre_timeout` 与延迟，直接 `set_repair_timeout_state` 落 fx 并回 `med_leak`。[LongUpdate 实现](../../../raw/current/scripts/components/boatleak.lua#L323-L337) 之外的状态机与动作条件正文见 [[reference/components/boatleak]]。
- **检索范围**：字面名全域无 SpawnPrefab（仅注册与 boatpatch.lua 依赖声明），生成完全经动态拼名；`.."_timeout_fx"` 拼接模式仅此一处。

## 关联机制导航

| 类型／对象 | 本族如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议：`MakeFx`（prefabs/fx.lua） | 7 条目均按公共工厂展开为网络代理＋各端本地动画实体 | 专服只有代理；本地实体 animover 自删 | [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]] |
| 宿主 Prefab：`boat_grass`／`boat_otterden` | `postsinkfn` 字段挂两条沉没终局路径；`material` 字段门禁草屑 | 服务端构造段赋值；两船均走默认 SGboat（data 无 stategraph 键，[绑定](../../../raw/current/scripts/prefabs/boat.lua#L586-L586)）；Wiki 待建 | [grass_fn](../../../raw/current/scripts/prefabs/boat.lua#L806-L870)、[otterden_fn](../../../raw/current/scripts/prefabs/boat.lua#L1167-L1269) |
| 组件：hullhealth | 草船／海獭窝船 `leakproof`（不出漏点实体）；草船自磨 `SetSelfDegrading(5)` 周期落 `degrade_fx_grass` | 服务端 | [[reference/components/hullhealth]]；[leakproof 分支](../../../raw/current/scripts/components/hullhealth.lua#L110-L113)、[碎屑生成](../../../raw/current/scripts/components/hullhealth.lua#L224-L235) |
| 组件：boatleak／boatpatch | 水草补丁到期链的状态机宿主与材料来源 | 服务端；到期 fx 是 boatleak 状态机的产物 | [[reference/components/boatleak]]、[[reference/components/boatpatch]] |
| 生成者：SGcookiecutter | `drill` 态按目标 material 周期落屑 | 服务端生物 SG，宿主 cookiecutter | [drill 态](../../../raw/current/scripts/stategraphs/SGcookiecutter.lua#L547-L593) |
| 关联 Prefab：`splash_green_small` | 草屑伴生水花，同一事件同点生成 | 仅漏点事件路径 | [[reference/prefabs/splash_green|splash_green 族]] |
| 关联 Prefab：`degrade_fx_grass`／`degrade_fx_ice` | 船体自磨碎屑，另一注册族（本批外） | hullhealth `SpawnDegadeDebris` 落点 | [grass 条目](../../../raw/current/scripts/fx.lua#L2747-L2763)、[ice 条目](../../../raw/current/scripts/fx.lua#L3255-L3270)；见 [[reference/prefabs/junk_break_fx|废料／退化特效（degrade_fx 族）]] |
| 资源 | bank/build `boat_grass`、`boat_otterden`、`boat_repair_kelp_fx`、`fx_portal_items` | 声明与使用已核验，包内容未解包 | 各条目行号见注册表 |

## 典型接入

### 给沉没流程追加自定义表现

- 前提与选择：`postsinkfn` 是船实体上的普通字段（服务端构造段赋值），`AddPrefabPostInit` 在构造后包装即可追加而不替换原表现；客户端该字段为 nil，须判端。
- 生效与撤销：调用点与船 `Remove` 同帧、紧邻其前；回调内保存的 inst 引用在 Remove 后悬空，不能延后使用；撤销即还原字段。

```lua
-- modmain.lua：给草船沉没追加一枚落点水花（保留原侵蚀对）
AddPrefabPostInit("boat_grass", function(inst)
    if not GLOBAL.TheWorld.ismastersim or inst.postsinkfn == nil then
        return
    end
    local postsink = inst.postsinkfn
    inst.postsinkfn = function(inst)
        postsink(inst)
        local x, y, z = inst.Transform:GetWorldPosition()
        GLOBAL.SpawnPrefab("splash_ocean").Transform:SetPosition(x, 0, z)
    end
end)
```

- 调整与验证：`splash_ocean` 可换成任意已注册 fx；两枚原版侵蚀 fx 已在 `postsink(inst)` 内生成，追加项注意排序层次（注册表 FinalOffset 值）；示例只做语法检查，未在游戏内执行。

## 常见陷阱与限制

- **误区：erode 是船体耐久侵蚀过程的表现**：船体自磨的周期碎屑是 `degrade_fx_grass`（hullhealth `degradefx` 字段，[boat.lua L844](../../../raw/current/scripts/prefabs/boat.lua#L844)）；`boat_*_erode` 只在沉没终局 `postsinkfn` 出现一次。
- **边界：erode 双生不成对清理**：两枚是独立实体、各自 animover；`erode_water` 18 帧后不可见但实体仍在，直到动画结束——淡出是颜色变化，不是实体移除。
- **边界：海獭窝船漏点无垫料屑、无漏点实体**：`OnSpawnNewBoatLeak_Grass` 门禁 `material == "grass"`，kelp 船只扣血放声；海带屑只在 cookiecutter 钻船中出现。
- **误区：`current_state.."_timeout_fx"` 可按补丁类型自由扩展**：`SetRepairedTime` 的调用点含 `repaired_kelp` 分支、两条存档恢复（boatleak OnLoad 与 `HullHealth:LoadPostPass`）与 `LongUpdate` 重排；木补丁／胶带等修复态不进入 `repaired_kelp`，无到期任务，不生成到期 fx；自制补丁类型需同时接 `SetState` 分支才有意义（见 [[reference/components/boatleak]] 陷阱）。
- **边界：落点 y 值固定**：漏点路径固定 y=0，drill 路径同样硬编码 y=0（海面标高），非取实体自身高度、也非跟随船体的下沉伴随物。

## 验证记录

- 源码审阅：2026-11-01，steam-buildid:26335394。fx.lua 7 条目区段全文；生成者 boat.lua（grass／otterden fn、两 postsinkfn、InstantlyBreakBoat、漏点事件处理）、SGboat `popping`、SGcookiecutter `drill`、boatleak 修复与正常到期链及 LongUpdate 补算直接落 fx 路径、hullhealth leakproof／degradefx 与碰撞 leak 读档 SetRepairedTime、tuning 数值、scheduler period-0 语义核验；`spawnnewboatleak` 事件源全域 17 处全量反查；7 名全域反查（含 `.."_timeout_fx"` 动态拼名追踪）无遗漏生成点。未核验：动画包内部时长与循环性、多客户端表现。
- 游戏运行：未运行。
- 结构检查：validate_page 通过（0 issue；112 行、13955 字节，standard 档内）；validate_source_links --strict 通过（34 处引用、0 error／0 warning）；Lua 示例另存临时文件经 parser_probe --syntax-only 通过（只查语法，未执行，临时文件已删）。

返回 [[reference/prefabs/fx-table-world-index|通用表 FX·环境与物件]]。
