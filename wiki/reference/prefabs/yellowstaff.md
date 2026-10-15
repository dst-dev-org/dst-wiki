---
title: "唤星者魔杖（yellowstaff）"
created: "2026-10-29"
updated: "2026-10-29"
type: prefab
summary: "唤星／矮星法杖这一施法物品的地点召唤、次数和精神消耗，以及皮肤映射到唤月法杖的转换入口。"
aliases: ["yellowstaff", "唤星者魔杖", "唤星", "矮星", "唤星法杖", "Star Caller's Staff", "召唤矮星"]
tags: [prefab, item, equipment, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["prefabs/staff.lua", "prefabs/staff_common.lua", "prefabs/stafflight.lua", "prefabs/moonbase.lua", "prefabskin.lua", "prefabs/skinprefabs.lua", "components/spellcaster.lua", "standardcomponents.lua", "tuning.lua", "entityscript.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 唤星者魔杖（yellowstaff）

## 快速判断

- 职责与适用：这里是消耗次数的施法物品；实际照明／供热实体是 [[reference/prefabs/stafflight|矮星（stafflight）]]。两者都可能被称为“矮星”，查温度和持续时间应转产物页。[生成与消耗](../../../raw/current/scripts/prefabs/staff_common.lua#L782-L800)
- 优先使用：物品施法和选点查本页；通用门禁查 [[reference/components/spellcaster#关键接口]]，次数查 [[reference/components/finiteuses#关键接口]]。本工厂通过 staff_common 的黄色分支配置，而非在 staff.lua 内完整实现。
- 重要副作用：createlight 先生成星体，再扣一次 finiteuses，并从持有者的 staffsanity 或 sanity 路径结算精神变化；耗尽会播碎裂声并移除法杖。[效果](../../../raw/current/scripts/prefabs/staff_common.lua#L782-L800)、[耗尽回调](../../../raw/current/scripts/prefabs/staff.lua#L13-L16)
- 修改边界：公共初始化后还调用 yellow_client 添加 reticule；主模拟再装 spellcaster、作祟和次数。光源效果、月台转换与法杖工厂分开核验。[黄色工厂](../../../raw/current/scripts/prefabs/staff.lua#L204-L229)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `yellowstaff` | yellow → commonfn("yellow", …, hasskin=true, hasshadowlevel=true) → yellow_client → 主模拟 yellow | commonfn 内 SetPristine 后客户端返回，但外层仍执行 client helper；主模拟设满 YELLOWSTAFF_USES。[共用初始化](../../../raw/current/scripts/prefabs/staff.lua#L31-L113)、[外层](../../../raw/current/scripts/prefabs/staff.lua#L204-L229)、[注册](../../../raw/current/scripts/prefabs/staff.lua#L308) |

名称键 `STRINGS.NAMES.YELLOWSTAFF` 为“唤星者魔杖”／Star Caller's Staff；“唤星、矮星”为用户提供的检索叫法。[译名](../../../raw/current/scripts/languages/chinese_s.po#L352140-L352142)、[默认取名](../../../raw/current/scripts/entityscript.lua#L840-L844)

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| finiteuses | 初始／最大 20；createlight 调用 Use(1)，耗尽 onfinished 移除 | 主模拟；不是 rechargeable 定时补充 | [[reference/components/finiteuses#关键接口]]；[配置](../../../raw/current/scripts/prefabs/staff.lua#L74-L75)、[满次数](../../../raw/current/scripts/prefabs/staff.lua#L216-L217)、[数值](../../../raw/current/scripts/tuning.lua#L208) |
| inspectable／inventoryitem／tradable／equippable | 检视拾取交易、穿脱持握，支持皮肤外观 | 主模拟共用工厂 | [[reference/components/inspectable#关键接口]]；[[reference/components/inventoryitem#关键接口]]；[[reference/components/tradable#关键接口]]；[[reference/components/equippable#关键接口]]；[挂载与回调](../../../raw/current/scripts/prefabs/staff.lua#L77-L104) |
| floater／shadowlevel／hauntable | 公共 floater；黄色启用 shadowlevel；主模拟先添加作祟，再追加自定义稀有召星反应 | shadowlevel 为本分支实际挂载；自定义作祟同样调用 createlight | [[reference/components/floater#关键接口]]；[[reference/components/shadowlevel#关键接口]]；[[reference/components/hauntable#按效果选工厂]]；[漂浮](../../../raw/current/scripts/prefabs/staff.lua#L58-L65)、[影子等级](../../../raw/current/scripts/prefabs/staff.lua#L107-L110)、[作祟绑定](../../../raw/current/scripts/prefabs/staff_common.lua#L833-L847) |
| reticule／spellcaster | client helper 配手柄选点；主模拟绑定 createlight，允许陆地点与水面位置 | reticule 在外层端分界前添加，不等于服务端最终许可 | [[reference/components/reticule#关键接口]]；[[reference/components/spellcaster#原版法杖的施法模式]]；[两层 helper](../../../raw/current/scripts/prefabs/staff_common.lua#L819-L847) |
| 生成物：stafflight | 显式 SpawnPrefab 并放到目标 pos；能力独立维护在产物页 | 主模拟施法或稀有作祟反应 | [[reference/prefabs/stafflight]]；[实际生成](../../../raw/current/scripts/prefabs/staff_common.lua#L782-L800)、[作祟调用](../../../raw/current/scripts/prefabs/staff_common.lua#L802-L817) |
| 转换：opalstaff／moonbase | 月台在 mooncharge 完成分支以 morph_skin 和 skin_id 生成唤月法杖并替换原物 | 月台 Wiki 待建；只核验转换分支，不提供完整启动条件／防守流程 | [[reference/prefabs/opalstaff#来源与唤星法杖的区别]]；[转换分支](../../../raw/current/scripts/prefabs/moonbase.lua#L459-L489) |
| 资源 | staffs／swap_staffs 动画包，staff_common 脚本；普通和皮肤持握分流 | 资源声明与实际解包验证分开 | [声明](../../../raw/current/scripts/prefabs/staff.lua#L1-L9)、[持握](../../../raw/current/scripts/prefabs/staff.lua#L85-L104) |

### 皮肤差异与转换对应

五种皮肤 `starstaff_chess`／`starstaff_crystal`／`starstaff_portal`／`starstaff_relic`／`starstaff_swamp` 分别以 granted_items[1] 指向对应 `moonstaff_*`。yellowstaff_init_fn 将该值保存为 morph_skin，clear 清除；月台转换时消费它。[定义](../../../raw/current/scripts/prefabs/skinprefabs.lua#L11783-L11840)、[写入与恢复](../../../raw/current/scripts/prefabskin.lua#L3344-L3353)、[消费](../../../raw/current/scripts/prefabs/moonbase.lua#L468-L475)

上述定义走共用 caststaff 外观／音效初始化；本次核验的 createlight 始终生成 stafflight，没有把皮肤对应的 moonstaff 标识当成星体 Prefab。星体的行为不复制到每个皮肤项。[共用皮肤入口](../../../raw/current/scripts/prefabskin.lua#L3280-L3306)、[召唤选择](../../../raw/current/scripts/prefabs/staff_common.lua#L782-L784)

## 典型接入

### 施法与消耗应修改哪一层

spellcaster:SetSpellFn(createlight) 决定效果；canuseonpoint／water 配置地点能力，最终地点门禁仍由 spellcaster:CanCast 处理。createlight 本身不重做这层检查，直接调用回调不能视为完整动作验证。[绑定](../../../raw/current/scripts/prefabs/staff_common.lua#L833-L847)、[地点门禁](../../../raw/current/scripts/components/spellcaster.lua#L152-L174)

普通黄色／月色法杖走 finiteuses:Use(1)；共用函数还支持 rechargeable+dodischarge 的其他宿主分支，不能因为函数中有 Discharge 就给这两种普通法杖标“自动充能”。精神变化读持有者：有 staffsanity 时优先 DoCastingDelta，否则 sanity:DoDelta，基础量为 -SANITY_MEDLARGE（本版 20），不承诺所有角色最终都扣 20。[分支与精神](../../../raw/current/scripts/prefabs/staff_common.lua#L786-L799)、[基础量](../../../raw/current/scripts/tuning.lua#L2036)；人物倍率见 [[reference/components/staffsanity#关键接口]]。

### 修改召唤物能力时转到矮星页

想改持续时长、供热、光圈或精神光环时，直接读 [[reference/prefabs/stafflight#能力与持续时长的修改入口]]。只改法杖次数不会改变已生成星体的 timer；已生成星体没有在此回调中挂为法杖子实体或登记回收监听。[创建路径](../../../raw/current/scripts/prefabs/staff_common.lua#L782-L800)

## 常见陷阱与限制

- **误区：“矮星”只指法杖或只指光源**：两个页面均可检索到该称呼；本页是施法物品，[[reference/prefabs/stafflight]] 是生成实体，应按次数／施法和温度／持续时间区分。
- **误区：依赖数组列出 stafflight 就证明一定生成**：关系依据是 createlight 的实际调用；依赖表只负责声明。[依赖声明](../../../raw/current/scripts/prefabs/staff_common.lua#L37-L41)、[生成](../../../raw/current/scripts/prefabs/staff_common.lua#L782-L784)
- **误区：皮肤 granted_items 就是额外召唤物**：这里是转换后的皮肤标识，实际消费位于月台转换分支，不是一次施法生成多种实体。[读取](../../../raw/current/scripts/prefabskin.lua#L3344-L3349)、[消费](../../../raw/current/scripts/prefabs/moonbase.lua#L468-L475)

## 验证记录

- 源码审阅：2026-10-29，Steam buildid 26230496；核验 staff 共用／黄色工厂、staff_common 地点召唤与稀有作祟反应、必要 spellcaster 门禁、五种皮肤与 morph_skin 消费、星体注册、TUNING 和名称。月台只覆盖所引转换分支；helper 挂载复用已读 standardcomponents。
- 游戏运行：未运行；未审校完整施法动作／玩家 SG、月台充能防守流程、全部换肤时序和资源包内容。无新增 Lua 示例。

返回 [[reference/prefabs/index|Prefab 索引]]。
