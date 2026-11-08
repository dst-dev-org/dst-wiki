---
title: "蚊血注射液（mosquitomermsalve）"
created: "2026-11-11"
updated: "2026-11-11"
type: prefab
summary: "沃特蚊子技能线专属治疗品（2 蚊子血囊，wurt_mosquito_craft_2）：healer 组件治疗量置 0、回血全在回调，CanHealFn 只放行带 merm 标签且非鱼人伪装的目标——鱼人 +100、沃特等玩家 +16；使用者为 wurt_mosquito_craft_3 技能时给目标附加 10 秒再生 buff（鱼人 5/秒、玩家 1/秒）。堆叠 40、无新鲜度、不可燃。"
aliases: ["mosquitomermsalve", "蚊血注射液", "Blood Shot", "鱼人药膏"]
tags: [prefab, item]
dst_build: "steam-buildid:26474873"
sources: ["prefabs/mosquitomermsalve.lua", "prefabs/merm_healthregenbuff.lua", "components/healer.lua", "components/builder.lua", "prefabs/wurt.lua", "prefabs/hats.lua", "prefabs/skilltree_wurt.lua", "actions.lua", "componentactions.lua", "recipes.lua", "tuning.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 蚊血注射液（mosquitomermsalve）

## 快速判断

- 职责与适用：可拾取治疗物品（inventoryitem），专为鱼人阵营设计：`healer` 治疗量置 0，`SetCanHealFn` 只接受带 `merm` 标签且不带 `mermdisguise` 的目标，实际回血与再生 buff 都在 `SetOnHealFn` 回调里手动结算。[接线 L70-L73](../../../raw/current/scripts/prefabs/mosquitomermsalve.lua#L70-L73)
- 优先使用：查"蚊血注射液／治疗鱼人／鱼人药膏"到本页；healer 组件的通用结算与门禁读 [[reference/components/healer]]；沃特蚊子技能组结构见 [[reference/prefabs/mosquitomusk]]「获得」节。
- 重要副作用：对非鱼人目标完全无效但仍会消耗；鱼人帽伪装玩家会被 `CanHealFn` 拒绝；堆叠 40。
- 修改边界：本页写注射液本体与沃特治疗链；`merm_healthregenbuff` 只记本页消费到的结算（debuff 体系页 Wiki 待建）；鱼人 NPC 本体页 Wiki 待建。

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `mosquitomermsalve` | `fn` 单工厂：网络段（`MakeInventoryPhysics`；bank/build 均 `mosquitomermsalve`、播 `idle`；`MakeInventoryFloatable("small", 0.2, 0.95)`）→ `SetPristine` → 客户端返回 → master 段（`inspectable`／`inventoryitem`／`stackable`（上限覆写 40）／`healer`（0＋OnHeal＋CanHeal）；`MakeHauntableLaunch`） | 无变体。prefabs 依赖 `merm_healthregenbuff`。**没有 burnable／propagator——不可燃、不传火**，与血囊、小痒痒不同。[构造全文](../../../raw/current/scripts/prefabs/mosquitomermsalve.lua#L43-L78) |

- 名称键 `STRINGS.NAMES.MOSQUITOMERMSALVE`＝蚊血注射液（英文 Blood Shot）。[po L345935-L345937](../../../raw/current/scripts/languages/chinese_s.po#L345935-L345937)
- `stackable.maxsize = TUNING.STACK_SIZE_SMALLITEM`＝40（[L67-L68](../../../raw/current/scripts/prefabs/mosquitomermsalve.lua#L67-L68)、[tuning L83](../../../raw/current/scripts/tuning.lua#L83)），比默认 `STACK_SIZE_MEDITEM`＝20 翻倍。

## 获得：沃特蚊子技能配方

- 配方：`mosquitosack`（蚊子血囊）2，`TECH.NONE`，`builder_skill="wurt_mosquito_craft_2"`（[recipes L116](../../../raw/current/scripts/recipes.lua#L116)）。
- 技能链：`wurt_mosquito_craft_1`（swampmaster 组根技能）→ `_2`（激活即解锁蚊子炸弹、嗡嗡肥料与本品三条配方）→ `_3`；技能门禁由 `skilltreeupdater:IsActivated` 在制作端判定（[skilltree_wurt L179-L215](../../../raw/current/scripts/prefabs/skilltree_wurt.lua#L179-L215)、[builder 技能门禁](../../../raw/current/scripts/components/builder.lua#L954-L966)）。组结构与 `_1`/`_2` 细节见 [[reference/prefabs/mosquitomusk]]「获得」节。
- 血囊来源（蚊子 50% 掉落、谋杀、气球堆消失）见 [[reference/prefabs/mosquitosack]]「获得」节。

## 治疗结算：门禁与实际回血来源

通用 healer 链（HEAL 动作候选 → `Heal`：canhealfn → 基础量×efficientuser → `health.canheal` 才 DoDelta → onhealfn → 消耗）复用 [[reference/components/healer]]。本品的例外都在两个回调上（[CanHeal L13-L19](../../../raw/current/scripts/prefabs/mosquitomermsalve.lua#L13-L19)、[OnHeal L21-L39](../../../raw/current/scripts/prefabs/mosquitomermsalve.lua#L21-L39)）：

1. **门禁（CanHealFn）**：`target` 须带 `merm` 标签且**不带** `mermdisguise`，否则返回 `false, "NOT_MERM"`，不治疗、不执行 onheal、不消耗（healer 先查 canhealfn 再继续）。
   - 沃特本体带 `merm`（连同 `playermerm`/`mermguard` 等基础标签，[wurt.lua L403-L410](../../../raw/current/scripts/prefabs/wurt.lua#L403-L410)）→ 可以给自己用药；
   - 鱼人 NPC 带 `merm` → 有效；
   - 穿鱼人帽的伪装玩家：帽子给 `merm`＋`mermdisguise` 双标签（[hats.lua L2945-L2953](../../../raw/current/scripts/prefabs/hats.lua#L2945-L2953)）→ **被拒绝**。
2. **回血来源（OnHealFn）**：`SetHealthAmount(0)` 使组件自身 delta 为 0（0×efficientuser 乘数仍 0），真实回血全部在回调里直连 `health:DoDelta`——目标带 `player` 标签回 `HEALING_MEDSMALL×2`＝16，否则（鱼人 NPC）回 `HEALING_SUPERHUGE`＝100（[L26](../../../raw/current/scripts/prefabs/mosquitomermsalve.lua#L26)、[tuning L2023/L2030](../../../raw/current/scripts/tuning.lua#L2022-L2030)）。手动分支不乘 efficientuser。
3. **canheal 双保险**：回调里 `DoDelta` 前先查 `target.components.health.canheal`（防 `healerbuffs` 标签把 Heal 引到不可治疗对象上，[L28-L31](../../../raw/current/scripts/prefabs/mosquitomermsalve.lua#L28-L31)、[healer.lua L38](../../../raw/current/scripts/components/healer.lua#L38)）；本品自身**没有** `healerbuffs` 标签，不会给不可常规治疗的目标冒出 HEAL 候选。
4. **再生 buff（技能加成）**：`doer`（使用者）激活 `wurt_mosquito_craft_3` 时，给**目标** `AddDebuff("merm_healthregenbuff")`（[L33-L38](../../../raw/current/scripts/prefabs/mosquitomermsalve.lua#L33-L38)）。`_3` 节点自身无 onactivate，效果全在消费者侧（[skilltree_wurt L211-L215](../../../raw/current/scripts/prefabs/skilltree_wurt.lua#L211-L215)）。

### 再生 buff 结算（merm_healthregenbuff）

- 服务端专用隐藏实体（CLASSIFIED、非网络化，客户端实例自删），经 debuff 组件挂到目标（[fn L37-L67](../../../raw/current/scripts/prefabs/merm_healthregenbuff.lua#L37-L67)）。
- 周期 1 秒一跳、持续 `MERM_HEALTHREGEN_DURATION`＝10 秒：玩家目标每跳 1 点（至多 10）、其他（鱼人）每跳 5 点（至多 50）；目标死亡或计时结束即停止（[OnTick L1-L13](../../../raw/current/scripts/prefabs/merm_healthregenbuff.lua#L1-L13)、[OnAttached L15-L23](../../../raw/current/scripts/prefabs/merm_healthregenbuff.lua#L15-L23)、[tuning L8296-L8300](../../../raw/current/scripts/tuning.lua#L8296-L8300)）。

## 常见陷阱与限制

- **边界：只认 `merm` 标签**：对韦伯、其他玩家（未伪装）、任何非鱼人目标使用 → `NOT_MERM` 拒绝发生在回调门禁，但注意动作候选仍会出现（HEAL 候选只看 `CanHeal()` 通用条件，不看本品 CanHealFn），点下去才知道无效——且被拒绝时不消耗。[healer.lua L28-L34](../../../raw/current/scripts/components/healer.lua#L28-L34)、[componentactions L2737-L2744](../../../raw/current/scripts/componentactions.lua#L2737-L2744)
- **边界：鱼人帽是反例**：伪装玩家有 `merm` 标签但被 `mermdisguise` 排除；给伪装玩家用药是无效消耗前的拒绝（不消耗）。[hats.lua L2945-L2953](../../../raw/current/scripts/prefabs/hats.lua#L2945-L2953)
- **误区：治疗量在 healer 组件里**：`SetHealthAmount(0)`，照抄普通药膏改 `SetHealthAmount` 的模组做法不会改变实际回血——要改 `OnHeal` 的 delta 或覆写回调。[L70-L73](../../../raw/current/scripts/prefabs/mosquitomermsalve.lua#L70-L73)
- **边界：再生 buff 看使用者不看目标**：`_3` 技能判断的是 `doer`；无技能的沃特治疗鱼人只有一次性 +100，有技能的目标（含沃特自己）才追加再生。[L33-L38](../../../raw/current/scripts/prefabs/mosquitomermsalve.lua#L33-L38)
- **边界：不可燃**：无 `burnable`／`propagator`，作祟 `MakeHauntableLaunch` 只弹飞（[L75](../../../raw/current/scripts/prefabs/mosquitomermsalve.lua#L75)）；**无新鲜度**（全文无 `perishable`，逐文件复核）。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 组件：`healer` | 0 治疗量＋CanHealFn 门禁＋OnHealFn 手动结算 | 服务端 | [[reference/components/healer]]；[L70-L73](../../../raw/current/scripts/prefabs/mosquitomermsalve.lua#L70-L73) |
| 组件：`stackable` | 上限覆写 `STACK_SIZE_SMALLITEM`＝40 | 服务端 | [[reference/components/stackable]]；[L67-L68](../../../raw/current/scripts/prefabs/mosquitomermsalve.lua#L67-L68) |
| 标签：`merm`／`mermdisguise` | 目标门禁的选择键与排除键 | 服务端判定 | [L14-L16](../../../raw/current/scripts/prefabs/mosquitomermsalve.lua#L14-L16) |
| 数据：`builder_skill` | `wurt_mosquito_craft_2` 制作门禁 | 制作端主端＋客户端各查一次 | [recipes L116](../../../raw/current/scripts/recipes.lua#L116)、[[reference/prefabs/mosquitomusk]] |
| 关联 Prefab：`mosquitosack` | 配方材料（×2） | 本体页已有 | [[reference/prefabs/mosquitosack]] |
| 关联 Prefab：`merm_healthregenbuff` | `_3` 技能下的目标再生 | 服务端隐藏实体、debuff 体系页 Wiki 待建 | [挂载 L33-L38](../../../raw/current/scripts/prefabs/mosquitomermsalve.lua#L33-L38) |
| 关联 Prefab：`wurt`／鱼人 NPC | 治疗对象（`merm` 标签持有者） | 沃特本体页／鱼人页 Wiki 待建 | [wurt.lua L403-L410](../../../raw/current/scripts/prefabs/wurt.lua#L403-L410) |
| 资源：`anim/mosquitomermsalve.zip` | bank/build 同名，`idle` | assets 声明 | [L1-L4](../../../raw/current/scripts/prefabs/mosquitomermsalve.lua#L1-L4)、[L52-L54](../../../raw/current/scripts/prefabs/mosquitomermsalve.lua#L52-L54) |

## 常见接入：模组放大鱼人治疗量

- 前提与选择：调整实际回血须覆盖 `OnHeal`，改 `SetHealthAmount` 无效（本品组件量为 0）。
- 生效与撤销：每次实例初始化生效；`canhealfn`／技能 buff 逻辑保持原样。

```lua
AddPrefabPostInit("mosquitomermsalve", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst.components.healer:SetOnHealFn(function(inst, target, doer)
        local delta = target:HasTag("player") and 32 or 200
        if target.components.health ~= nil and target.components.health.canheal then
            target.components.health:DoDelta(delta, false, inst.prefab)
        end
    end)
end)
```

- 调整与验证：覆写会丢掉原版 `_3` 技能的再生 buff——需要保留时先存原 `onhealfn` 在新回调里补调；门禁（`canhealfn`）未动，仍只认非伪装鱼人。

## 验证记录

- 源码审阅：2026-11-11，steam-buildid:26474873（与 `raw/current/BUILD.txt` 一致）。覆盖 `mosquitomermsalve.lua` 全文、`merm_healthregenbuff.lua` 全文、`components/healer.lua` 全文与 HEAL 候选、沃特 `merm` 标签、鱼人帽伪装标签、技能三阶定义与配方行、`STACK_SIZE_SMALLITEM` 与治疗调参值、`chinese_s.po` 名称。
- 逐文件复核：`mosquitomermsalve.lua` 无 `perishable`／`burnable`／`propagator`／`healerbuffs` 挂载。
- 游戏运行：未运行；再生节奏按定义值推导，非实测。
- 机械校验（页面、导航、严格链接、索引一致性）待验收时运行。

相关：[[reference/components/healer]]、[[reference/prefabs/mosquitomusk]]、[[reference/prefabs/mosquitosack]]。

返回 [[reference/prefabs/index|原版 Prefab 索引]]。
