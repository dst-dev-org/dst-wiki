---
title: oar
created: 2026-10-21
updated: "2026-10-28"
type: component
summary: "手部船桨的划船入口：校验平台与 boatphysics、按朝向施加划船力并触发划船事件，时机失败时加湿并返回台词键。"
aliases: ["船桨组件", "划船组件", "划桨", "划船失败", "Row", "RowFail", "BAD_TIMING", "oar"]
tags: [component, item, equipment]
dst_build: "steam-buildid:26230496"
sources: ["components/oar.lua", "prefabs/oar.lua", "actions.lua", "componentactions.lua", "components/boatphysics.lua", "components/expertsailor.lua", "components/mightiness.lua", "tuning.lua", "stategraphs/commonstates.lua", "prefabs/wolfgang.lua", "prefabs/wx78_possessedbody.lua", "speech_wilson.lua", "components/crewmember.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# oar

## 快速判断

- 职责与适用：把手持桨的一次“划”转成对**脚下平台** `boatphysics` 的推力；自身不做船速判断、不扣耐久、无冷却，平台或 `boatphysics` 缺失时静默返回。[components/oar.lua:11-33](../../../raw/current/scripts/components/oar.lua#L11-L33)
- 优先使用：物品加 `oar` 并设 `force`／`max_velocity`，动作沿用 `ROW`、`ROW_CONTROLLER`、`ROW_FAIL`；原版五种桨共用同一工厂。[prefabs/oar.lua:65-179](../../../raw/current/scripts/prefabs/oar.lua#L65-L179)
- 重要副作用：推进后执行者收到 `rowing`，平台收到 `rowed`（参数为划船者）；失败走 `RowFail`：加湿半径 2 内实体、给随从发 `leader_failed_row`、返回 `BAD_TIMING<n>`。[components/oar.lua:29-32](../../../raw/current/scripts/components/oar.lua#L29-L32)、[:35-51](../../../raw/current/scripts/components/oar.lua#L35-L51)
- 修改边界：组件只在服务端物品实例上添加，`Row` 不校验 `boat` 标签；客户端只负责收集动作、按动画帧判早划并播表现。[prefabs/oar.lua:106-108](../../../raw/current/scripts/prefabs/oar.lua#L106-L108)、[componentactions.lua:15-56](../../../raw/current/scripts/componentactions.lua#L15-L56)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `Row(doer, pos)` | `doer` 玩家实体，`pos` 世界坐标（`Vector3` 或 `{x,z}`）；无返回值 | 取 `doer:GetCurrentPlatform()`，平台无效或无 `boatphysics` 直接返回；方向由 `pos - doer` 归一化。[oar.lua:11-19](../../../raw/current/scripts/components/oar.lua#L11-L19) |
| 控制器分支 | 无参数，读 `doer.components.playercontroller.isclientcontrollerattached` | 为真时方向改为 `doer - 平台中心`，即“向船外侧划”；服务端按该分支重算，因此控制器与鼠标的方向语义不同。[oar.lua:21-24](../../../raw/current/scripts/components/oar.lua#L21-L24) |
| `force` | `number`；构造默认 `0.4` | 每划的力，宿主用 `TUNING.BOAT.OARS.*.FORCE` 覆盖（0.3／0.5／0.8／0.6）。[oar.lua:8](../../../raw/current/scripts/components/oar.lua#L8)、[prefabs/oar.lua:107](../../../raw/current/scripts/prefabs/oar.lua#L107 |
| `max_velocity` | `number`；构造默认 `TUNING.BOAT.MAX_FORCE_VELOCITY`（3.5） | 传给 `ApplyRowForce` 的阻尼参考速度，**不是**船速硬上限；宿主覆盖为 2／3.5／5／3。[oar.lua:7](../../../raw/current/scripts/components/oar.lua#L7)、[boatphysics.lua:310-337](../../../raw/current/scripts/components/boatphysics.lua#L310-L337) |
| `RowFail(doer)` | 返回 `"BAD_TIMING" .. fail_idx` | 不校验平台、桨或 `boatphysics`；先自增 `fail_idx`，再加湿、通知随从。[oar.lua:35-50](../../../raw/current/scripts/components/oar.lua#L35-L50) |
| `fail_wetness` / `fail_string_count` | 默认 `9` / `3` | 加湿量按 `DoDeltaMoistureToEntity` 语义扣防水；台词键循环 `BAD_TIMING0..2`。[oar.lua:4-5](../../../raw/current/scripts/components/oar.lua#L4-L5)、[:36](../../../raw/current/scripts/components/oar.lua#L36) |
| 消耗与冷却 | 组件无字段、无 `OnSave` | 耐久由宿主 `finiteuses` 按动作扣：`ROW`／`ROW_CONTROLLER` 各 1，`ROW_FAIL` 为 `ROW_FAIL_WEAR`（原版 25，马巴罗斯喙桨 6）。[prefabs/oar.lua:137-144](../../../raw/current/scripts/prefabs/oar.lua#L137-L144) |
| 专家水手修正 | 无显式字段 | `doer.components.expertsailor` 存在时取力倍率与额外速度；用 `and ... or 1/0` 兜底，字段为 nil 时取默认值。[oar.lua:26-27](../../../raw/current/scripts/components/oar.lua#L26-L27) |

动作链（服务端生效）：客户端右击水面/目标 → [componentactions.lua:2152-2157](../../../raw/current/scripts/componentactions.lua#L2152-L2157) 的 `oar` 收集器（要求所在平台带 `boat` 标签）→ `ROW`／`ROW_CONTROLLER`／`ROW_FAIL`（定义见 [actions.lua:565-567](../../../raw/current/scripts/actions.lua#L565-L567)）→ 服务端 [actions.lua:1356-1384](../../../raw/current/scripts/actions.lua#L1356-L1384) 取手持桨并调用 `Row`，随后补发 `working`。`ROW_FAIL` 由动画帧判定（`row_pre` 前 30 帧或 `row` 前 4 帧内再按），服务端在 `row` 状态第 8 帧执行动作。[componentactions.lua:32-54](../../../raw/current/scripts/componentactions.lua#L32-L54)、[commonstates.lua:1973-2006](../../../raw/current/scripts/stategraphs/commonstates.lua#L1973-L2006)

## 典型接入

### 自定义船桨物品（服务端，参照原版 `oar`／`oar_driftwood`）

- 前提与选择：目标文件 `scripts/prefabs/my_oar.lua`，由 `modmain.lua` 的 `PrefabFiles` 加载；必须在 `entity:SetPristine()` 之后、`TheWorld.ismastersim` 分支内加组件，与 [prefabs/oar.lua:106-108](../../../raw/current/scripts/prefabs/oar.lua#L106-L108) 一致。资源沿用 `anim/oar.zip`、`anim/swap_oar.zip`。
- 生效与撤销：玩家装备到手部后，在带 `boat` 标签且有 `boatphysics` 的平台上右击水面即可划；卸下或离开平台后 `Row` 自动不生效。耐久只在 `finiteuses` 配了对应消耗时扣，撤销即移除该配置。
- 原版依据：[prefabs/oar.lua:57-63](../../../raw/current/scripts/prefabs/oar.lua#L57-L63) 的主船员耗次修正、[:124-144](../../../raw/current/scripts/prefabs/oar.lua#L124-L144) 的装备与耐久配置。

```lua
local assets =
{
    Asset("ANIM", "anim/oar.zip"),
    Asset("ANIM", "anim/swap_oar.zip"),
}

local function onequip(inst, owner)
    owner.AnimState:OverrideSymbol("swap_object", "swap_oar", "swap_oar")
    owner.AnimState:Show("ARM_carry")
    owner.AnimState:Hide("ARM_normal")
end

local function onunequip(inst, owner)
    owner.AnimState:Hide("ARM_carry")
    owner.AnimState:Show("ARM_normal")
end

local function fn()
    local inst = CreateEntity()

    inst.entity:AddTransform()
    inst.entity:AddAnimState()
    inst.entity:AddNetwork()

    inst:AddTag("allow_action_on_impassable")
    MakeInventoryPhysics(inst)
    MakeInventoryFloatable(inst, "small", nil, 0.68)

    inst.AnimState:SetBank("oar")
    inst.AnimState:SetBuild("oar")
    inst.AnimState:PlayAnimation("idle")

    inst.entity:SetPristine()
    if not TheWorld.ismastersim then
        return inst
    end

    inst:AddComponent("inventoryitem")

    local oar = inst:AddComponent("oar")
    oar.force = 0.5          -- 每划推力
    oar.max_velocity = 3.5   -- 阻尼参考速度，非硬上限

    inst:AddComponent("equippable")
    inst.components.equippable:SetOnEquip(onequip)
    inst.components.equippable:SetOnUnequip(onunequip)

    local finiteuses = inst:AddComponent("finiteuses")
    finiteuses:SetMaxUses(400)
    finiteuses:SetUses(400)
    finiteuses:SetConsumption(ACTIONS.ROW, 1)
    finiteuses:SetConsumption(ACTIONS.ROW_CONTROLLER, 1)
    finiteuses:SetConsumption(ACTIONS.ROW_FAIL, 25)

    return inst
end

return Prefab("my_oar", fn, assets)
```

- 调整与验证：`force` 与 `max_velocity` 只有相对意义，实际速度还受船体阻尼、载重和 `TUNING.BOAT.MAX_ALLOWED_VELOCITY`（10）限制；要改手感先对照 `TUNING.BOAT.OARS` 四档。案例未运行，见验证记录。

## 常见陷阱与限制

- 平台没有 `boatphysics` → `Row` 在 [oar.lua:15-16](../../../raw/current/scripts/components/oar.lua#L15-L16) 直接返回，无报错、无推力；在深渊柱等“可行走平台”上划桨表现为完全没反应。平台如何维护成员见 [[reference/components/walkableplatform]]。
- 客户端是否出现划船动作由 `oar` 收集器决定，它要求所在平台 `HasTag("boat")`（[componentactions.lua:15-19](../../../raw/current/scripts/componentactions.lua#L15-L19)）；但服务端 `Row` 不检查该标签。自定义平台若想用鼠标划船必须补 `boat` 标签，否则只能用代码直接压 `BufferedAction`。
- `RowFail` 不检查平台与桨（[oar.lua:35-43](../../../raw/current/scripts/components/oar.lua#L35-L43)），原版 WX78 身体就是直连 `ROW_FAIL`（[prefabs/wx78_possessedbody.lua:267-275](../../../raw/current/scripts/prefabs/wx78_possessedbody.lua#L267-L275)）；用它做“失败反馈”时不要假定角色一定在船上。
- `fail_idx` 初值 0、先自增后取模：第一次失败返回 `BAD_TIMING1`，随后 2、0 循环。自定义角色需提供 `BAD_TIMING0..2` 三条台词，否则 `talker:Say(nil)` 无有效文本（原版见 [speech_wilson.lua:363-368](../../../raw/current/scripts/speech_wilson.lua#L363-L368)）。
- 组件本身无冷却、无耐久；早划判定依赖动画帧与 `is_rowing` 标签，服务端并不阻止连续调用 `Row`。要限速应在动作层或宿主侧做，而不是改 `oar.lua` 的这两处事件；耗次配置见 [[reference/components/finiteuses]]。
- `max_velocity` 只参与 `ApplyRowForce` 的分向阻尼插值；船速最终仍被 `DoApplyForce` 截到 `TUNING.BOAT.MAX_ALLOWED_VELOCITY`，把它当“最高船速”会得到错误预期。[boatphysics.lua:300-308](../../../raw/current/scripts/components/boatphysics.lua#L300-L308)
- 船员／随从划船走另一条链：`crewmember:Row()` 直接调 `ApplyRowForce` 并只发 `rowed`，不经过 `oar`，也不会发 `rowing`；给 AI 船加“划船加成”时不要只改 `oar`。[components/crewmember.lua:93-164](../../../raw/current/scripts/components/crewmember.lua#L93-L164)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 锚点重定位 4 处（componentactions.lua oar 收集器、actions.lua row 函数、commonstates.lua row 状态、speech_wilson.lua BAD_TIMING 台词）；结论修订：无，仅锚点迁移。依据：枢纽 diff 审阅＋变更清单命中 actions.lua、componentactions.lua、stategraphs/commonstates.lua、speech_wilson.lua、prefabs/wx78_possessedbody.lua（命中区域逐行核对均未漂移）。
- 源码审阅：2026-10-21，`steam-buildid:25643519`；覆盖构造默认值与宿主覆盖、`Row` 的平台/物理校验与双方向语义、`RowFail` 的加湿与随从事件、动作三入口与客户端帧判定、`finiteuses` 耗次、`expertsailor`／沃尔夫冈加成、`rowed` 消费者（水獭巢船）与船员对照。未覆盖：实际划船数值与手感、不同网络延迟下的动作时序、模组覆盖 `actions`/`componentactions` 的冲突、专用服务器实机。
- 游戏运行：未运行。
- 结构检查：见本页交付报告（validate_page、validate_source_links、示例 `--syntax-only` 的实际结果）。
