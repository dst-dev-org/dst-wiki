---
title: "唤月者魔杖（opalstaff）"
created: "2026-10-29"
updated: "2026-10-29"
type: prefab
summary: "唤月／冷星／极光法杖这一施法物品的 50 次地点召唤、月台转换来源，以及与极光实体和皮肤 build 的区分。"
aliases: ["opalstaff", "唤月者魔杖", "唤月", "冷星", "极光", "唤月法杖", "Moon Caller's Staff", "召唤极光"]
tags: [prefab, item, equipment, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["prefabs/staff.lua", "prefabs/staff_common.lua", "prefabs/stafflight.lua", "prefabs/moonbase.lua", "prefabskin.lua", "prefabs/skinprefabs.lua", "standardcomponents.lua", "tuning.lua", "entityscript.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 唤月者魔杖（opalstaff）

## 快速判断

- 职责与适用：这里是消耗次数、在地点生成 [[reference/prefabs/staffcoldlight|极光（staffcoldlight）]] 的法杖；“唤月、冷星、极光”按用户提供收录，不把物品与产物合成同一实体。[生成选择](../../../raw/current/scripts/prefabs/staff_common.lua#L782-L800)
- 优先使用：查法杖来源／次数／选点读本页；查照明、制冷与存续读产物页；通用施法与消耗流程复用 [[reference/prefabs/yellowstaff#施法与消耗应修改哪一层]]。
- 重要副作用：本工厂最大／初始次数 50，每次普通 createlight 消耗 1，并经持有者精神路径结算；耗尽移除物品。[工厂次数](../../../raw/current/scripts/prefabs/staff.lua#L283-L287)、[共用消耗](../../../raw/current/scripts/prefabs/staff_common.lua#L786-L799)、[耗尽](../../../raw/current/scripts/prefabs/staff.lua#L13-L16)
- 修改边界：与黄色法杖共享 commonfn 但 hasshadowlevel=false；opal_client 与主模拟 opal 分别配置选点和实际施法。没有给本法杖挂 heater 来提供制冷，制冷属于产物。[工厂](../../../raw/current/scripts/prefabs/staff.lua#L272-L298)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `opalstaff` | opal → commonfn("opal", …, true, false) → opal_client → 主模拟 opal 与 50 次 finiteuses | 普通／皮肤持握由 commonfn 处理；注册位于 staff.lua，非 opalstaff.lua。[共用构造](../../../raw/current/scripts/prefabs/staff.lua#L31-L113)、[月杖工厂](../../../raw/current/scripts/prefabs/staff.lua#L272-L298)、[注册](../../../raw/current/scripts/prefabs/staff.lua#L309) |

名称键 `STRINGS.NAMES.OPALSTAFF` 为“唤月者魔杖”／Moon Caller's Staff。[简中条目](../../../raw/current/scripts/languages/chinese_s.po#L346045-L346047)、[默认名称](../../../raw/current/scripts/entityscript.lua#L840-L844)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| finiteuses | 最大／初始 50；createlight 消耗一次，耗尽执行共用碎裂移除回调 | 主模拟；没有自动充能配置 | [[reference/components/finiteuses#关键接口]]；[初始化](../../../raw/current/scripts/prefabs/staff.lua#L74-L75)、[本体次数](../../../raw/current/scripts/prefabs/staff.lua#L286-L287)、[数值](../../../raw/current/scripts/tuning.lua#L211) |
| inspectable／inventoryitem／tradable／equippable | 共用物品、检视、交易、穿脱外观 | 主模拟；本分支未启用 shadowlevel | [[reference/components/inspectable#关键接口]]；[[reference/components/inventoryitem#关键接口]]；[[reference/components/tradable#关键接口]]；[[reference/components/equippable#关键接口]]；[共用挂载](../../../raw/current/scripts/prefabs/staff.lua#L77-L104)、[分支参数](../../../raw/current/scripts/prefabs/staff.lua#L272-L275) |
| floater／hauntable | commonfn 公共漂浮；主模拟作祟发射并加稀有召星反应 | 召星反应共用 createlight，按本体 prefab 选择冷星 | [[reference/components/floater#关键接口]]；[[reference/components/hauntable#按效果选工厂]]；[漂浮](../../../raw/current/scripts/prefabs/staff.lua#L58-L65)、[作祟调用](../../../raw/current/scripts/prefabs/staff.lua#L283-L284)、[追加反应](../../../raw/current/scripts/prefabs/staff_common.lua#L859-L873) |
| reticule／spellcaster | 手柄选点与主模拟地点施法，陆地点／水面均配置允许 | helper 位于 staff_common，reticule 不代替最终地点验证 | [[reference/components/reticule#关键接口]]；[[reference/components/spellcaster#关键接口]]；[选点](../../../raw/current/scripts/prefabs/staff_common.lua#L849-L857)、[效果绑定](../../../raw/current/scripts/prefabs/staff_common.lua#L859-L873) |
| 生成物：staffcoldlight | createlight 显式生成并放到 pos | 由本法杖 prefab 标识选中，不靠 skin_build 选中 | [[reference/prefabs/staffcoldlight]]；[实际创建](../../../raw/current/scripts/prefabs/staff_common.lua#L782-L784) |
| 来源：yellowstaff／moonbase | 月台转换分支移除原杖并生成 opalstaff，可携带 morph_skin／skin_id | 月台 Wiki 待建；本页不覆盖整场启动与防守 | [[reference/prefabs/yellowstaff#皮肤差异与转换对应]]；[转换](../../../raw/current/scripts/prefabs/moonbase.lua#L459-L489) |
| 资源／皮肤 | staffs／swap_staffs 资源，共用持握／音效皮肤初始化 | 皮肤标识和视觉 build 可不同 | [资源](../../../raw/current/scripts/prefabs/staff.lua#L1-L9)、[持握](../../../raw/current/scripts/prefabs/staff.lua#L85-L104)、[皮肤共用入口](../../../raw/current/scripts/prefabskin.lua#L3280-L3306) |

### 皮肤差异与 build 对应

`moonstaff_chess`／`moonstaff_crystal`／`moonstaff_portal`／`moonstaff_relic`／`moonstaff_swamp` 的 base_prefab 均为 opalstaff，但 build_name_override 指向对应 `starstaff_*`，init 也传该 build。名称、build 与生成 Prefab 应分开判断。[五种定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L8446-L8504)

opalstaff_init_fn／clear 是 caststaff_init_fn／clear 的别名；moonstaff_portal 额外配置 cast 音效。本次所读分支没有为这些皮肤另选星体实体，仍生成 staffcoldlight。[共用函数](../../../raw/current/scripts/prefabskin.lua#L3355-L3356)、[专属音效](../../../raw/current/scripts/prefabs/skinprefabs.lua#L8469-L8480)、[生成选择](../../../raw/current/scripts/prefabs/staff_common.lua#L782-L784)

## 典型接入

### 来源与唤星法杖的区别

两种物品都使用 createlight，唯 opalstaff 分支生成 staffcoldlight；当前最大次数分别是 20／50。月台在 mooncharge 完成且所示条件成立的分支将黄色法杖换为月杖，转换皮肤来自原杖 morph_skin；本页没有审校月相、启动、守卫和失败恢复等完整条件。[效果选择](../../../raw/current/scripts/prefabs/staff_common.lua#L782-L800)、[次数](../../../raw/current/scripts/tuning.lua#L208-L212)、[月台分支](../../../raw/current/scripts/prefabs/moonbase.lua#L459-L489)

### 查询冷星时先区分施法物品与产物

想改有限次数或精神消耗：复用 [[reference/prefabs/yellowstaff#施法与消耗应修改哪一层]]，再对照本工厂配置。想改极光持续时间、降温或精神光环：进入 [[reference/prefabs/staffcoldlight#制冷光环与存续时间]]，法杖皮肤和次数不直接替代产物组件。

## 常见陷阱与限制

- **误区：moonstaff_* 是实际 SpawnPrefab 的法杖名**：它们是皮肤标识；实际法杖注册名为 opalstaff，且视觉 build 可以用 starstaff_*。[定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L8446-L8504)、[注册](../../../raw/current/scripts/prefabs/staff.lua#L309)
- **误区：staffcoldlightfx 就是这支法杖平时召出的冷星**：createlight 明确生成 staffcoldlight。staffcoldlightfx 是同族另一个工厂，已在 [[reference/prefabs/staffcoldlight#月台附属变体-staffcoldlightfx]] 区分。
- **误区：改名或换皮肤能改变召唤类型**：共用函数判断 staff.prefab 是否为 opalstaff；修改时须分清注册名与皮肤字段。[分支](../../../raw/current/scripts/prefabs/staff_common.lua#L782-L784)

## 验证记录

- 源码审阅：2026-10-29，Steam buildid 26230496；核验 staff 共用／月杖工厂、staff_common 选点与效果配置、五种 moonstaff 皮肤、月台转换分支、星体实际注册和中文名称。消费流程与组件行为按已核验公共页复用。
- 游戏运行：未运行；未审校月台完整生命周期、全部施法动作／SG／重皮肤调用、资源包或模组兼容。无新增 Lua 示例。

返回 [[reference/prefabs/index|Prefab 索引]]。
