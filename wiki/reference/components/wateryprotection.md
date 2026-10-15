---
title: "wateryprotection"
created: "2026-10-18"
updated: "2026-10-28"
type: component
summary: "施加洒水、灭火、降温和抗枯萎效果，区分实体范围、单格土壤与工具次数消费。"
aliases: ["洒水壶浇水量翻倍", "范围灭火", "洒水降温", "wateryprotection"]
tags: [component, item, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/wateryprotection.lua", "componentutil.lua", "prefabs/wateringcan.lua", "actions.lua", "componentactions.lua", "components/farming_manager.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# wateryprotection

## 快速判断

- 职责与适用：效果源侧的即时施加器，可处理灭火、抗枯萎、冷冻积累、温度与湿度；不是防雨装备或定时光环。[实体处理](../../../raw/current/scripts/components/wateryprotection.lua#L21-L41)
- 优先使用：地点用 `SpreadProtectionAtPoint`，实体所在点用 `SpreadProtection`；只处理指定实体才用 `ApplyProtectionToEntity`，后者不浇土壤也不调用范围收尾回调。[接口](../../../raw/current/scripts/components/wateryprotection.lua#L21-L61)
- 重要副作用：范围调用先处理附近实体，再浇中心点所在单格，最后 onspreadprotectionfn；即使没有命中实体也会执行收尾。原洒水壶在该回调扣一次 finiteuses。[顺序](../../../raw/current/scripts/components/wateryprotection.lua#L43-L56) [消费](../../../raw/current/scripts/prefabs/wateringcan.lua#L69-L73)
- 修改边界：本页服务端。正常动作先检查拟态与剩余次数；直接调用组件不含这些门禁，不要用它证明空壶也可合法浇水。[动作](../../../raw/current/scripts/actions.lua#L5416-L5434)

## 关键接口

| 接口／字段 | 参数或默认 | 作用、条件与证据 |
|---|---|---|
| `SpreadProtectionAtPoint(x,y,z,dist?,noextinguish?)` | 世界坐标；无显式返回 | 实体半径依次取 dist、protection_dist、4；按 ignoretags 排除。土壤仅中心点一格，不随半径遍历地块。[实现](../../../raw/current/scripts/components/wateryprotection.lua#L43-L56) |
| `SpreadProtection(target,dist?,noextinguish?)` | target 需 Transform | 取 target 世界坐标后执行上项；不是只处理 target。[实现](../../../raw/current/scripts/components/wateryprotection.lua#L58-L61) |
| `ApplyProtectionToEntity(ent,noextinguish?)` | 无显式返回 | 有 burnable 才进入抗枯萎／灭火分支；抗枯萎还要求 witherable 和正时长，灭火要求 extinguish 真、无 noextinguish 且正在燃烧／冒烟。[实现](../../../raw/current/scripts/components/wateryprotection.lua#L21-L31) |
| `witherprotectiontime`／`extinguish`／`extinguishheatpercent` | 0 秒／true／0 | 分别传 Protect、灭火开关和 Extinguish 参数；noextinguish 仅抑制灭火，不关其他效果。[默认](../../../raw/current/scripts/components/wateryprotection.lua#L4-L9) [消费](../../../raw/current/scripts/components/wateryprotection.lua#L21-L31) |
| `addcoldness`／`temperaturereduction` | 默认各 0 | 正数才处理；前者给 freezable:AddColdness，后者经工具函数减实体温度。不是温度恒定或立即冻结保证。[处理](../../../raw/current/scripts/components/wateryprotection.lua#L32-L37) [温度分派](../../../raw/current/scripts/componentutil.lua#L2417-L2424) |
| `addwetness` | 默认 0 | 正数才增加实体湿度，人物可受防水系数影响，物品走 inventoryitem；范围调用另把该值传土壤接口。负数不会减少实体湿度，却可走土壤减水。[分支](../../../raw/current/scripts/components/wateryprotection.lua#L38-L50) [湿度分派](../../../raw/current/scripts/componentutil.lua#L2400-L2408) |
| `AddIgnoreTag(tag)` | 默认排除 FX／DECOR／INLIMBO／burnt | 去重加入实体搜索排除列表；不影响单格土壤更新。原洒水壶额外忽略 player。[实现](../../../raw/current/scripts/components/wateryprotection.lua#L12-L19) [原配置](../../../raw/current/scripts/prefabs/wateringcan.lua#L131-L138) |
| `onspreadprotectionfn` | `(source,x,y,z)` | 所有实体／土壤处理之后调用；返回值不被组件使用，不能靠 false 回滚或阻止已执行效果。[位置](../../../raw/current/scripts/components/wateryprotection.lua#L43-L56) |

右键农田地面产生 POUR_WATER_GROUNDTILE，右键 withered／fire／smolder／canpourwateron 目标产生 POUR_WATER；两个动作共用执行函数。组件本身不包含这些候选条件。[地面候选](../../../raw/current/scripts/componentactions.lua#L2231-L2235) [实体候选](../../../raw/current/scripts/componentactions.lua#L2528-L2532) [共用](../../../raw/current/scripts/actions.lua#L5440-L5442)

## 典型接入

### 普通洒水壶每次浇水量翻倍，保留原一次消耗

放目标模组 `modmain.lua`，无需新资源；只改 wateringcan 的正 addwetness，不改原回调、半径或次数。原普通／高级壶是不同 Prefab，本例只选普通壶。[配置](../../../raw/current/scripts/prefabs/wateringcan.lua#L131-L149) [注册](../../../raw/current/scripts/prefabs/wateringcan.lua#L174-L175) [PostInit](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("wateringcan", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local protection = inst.components.wateryprotection
    if protection ~= nil and protection.addwetness > 0 then
        protection.addwetness = protection.addwetness * 2
    end
end)
```

影响后续单格土壤加水和允许命中的实体湿度；原壶忽略 player，不能称给人物湿度翻倍。原 onspreadprotectionfn 仍只 Use 一次，不额外设 finiteuses 动作消费。土壤上限／世界湿度下限见 [[reference/components/farming-manager#关键接口]]。

同字段还被壶烧毁时按剩余次数计算释水量，本例也会改变该量；不是只改一次右键浇水。停用并重新初始化恢复原配置，已改变的湿度／土壤水分不回滚。[烧毁消费者](../../../raw/current/scripts/prefabs/wateringcan.lua#L61-L67) 补水入口见 [[reference/components/fillable]]，水源份量见 [[reference/components/watersource]]。

## 常见陷阱与限制

- **扩大 protection_dist 就能浇多格田**：实体范围扩大，土壤仍只调一个点；多格玩法须明确地块去重、剂量和消费规则。[范围顺序](../../../raw/current/scripts/components/wateryprotection.lua#L43-L56)
- **换掉收尾回调却忘记保留原函数**：效果继续但原工具可能不扣水；反之额外调用 Use 会重复消费。动作只检查现有次数，实际扣费在原回调。[动作](../../../raw/current/scripts/actions.lua#L5422-L5432) [回调](../../../raw/current/scripts/prefabs/wateringcan.lua#L69-L73)
- **只给目标加 witherable 就能被本组件保护**：抗枯萎分支还要求目标有 burnable，且时长 >0；noextinguish 不是取消所有保护。[实体分支](../../../raw/current/scripts/components/wateryprotection.lua#L21-L31)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的变更文件（actions.lua、componentactions.lua、componentutil.lua）对照更新报告 R-01/R-04/R-05 与 hunk 地图逐条核验，POUR_WATER 双候选、共用执行函数与湿度／温度分派 helper 均无语义变化，锚点重定位 7 处、结论修订 0 处；未覆盖：运行验证。

- 源码审阅：2026-10-18，Steam buildid 25643519；覆盖实体分支、湿度／温度分派、搜索半径／标签、单格土壤、收尾回调与原壶消费、动作门禁及烧毁消费者。土壤限幅复用当前 farming_manager 页；未全面审校所有灭火／投射物消费者。
- 结构检查：格式、来源、链接／行号、大小与登记通过；完整示例通过已有 Full Moon Lua 5.1 syntax-only，不证明游戏效果或耗水次数。
- 游戏运行：未运行；空壶、灭火／防水、范围边界、土壤上下限、烧毁、多人及其他模组参数覆盖待实测。
