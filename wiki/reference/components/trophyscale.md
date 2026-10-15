---
title: "trophyscale"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "奖杯称重比较台：接收 weighable 物品，只保留最重纪录并移除较轻者，可按纪录重新生成物品供领取。"
aliases: ["称重台", "奖杯纪录", "鱼秤", "trophyscale"]
tags: [component, item, actions]
dst_build: "steam-buildid:26230496"
size_class: standard
sources: ["components/trophyscale.lua", "prefabs/trophyscale_fish.lua", "prefabs/trophyscale_oversizedveggies.lua", "actions.lua", "constants.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# trophyscale

## 快速判断

- 职责与适用：展示"最重纪录"的建筑组件；与 `weighable` 组件按 `trophyscale_<type>`/`weighable_<type>` 标签配对，类型只有 fish/oversizedveggies 两种（constants.lua:2265-2268）。原版消费者为 trophyscale_fish 与 trophyscale_oversizedveggies 两个建筑。
- 优先使用：接入比较动作→`accepts_items` + 类型标签（actions.lua:4655-4675 `COMPARE_WEIGHABLE`）；领取纪录→`trophycanbetaken` 标签 + `REMOVE_FROM_TROPHYSCALE`（actions.lua:5154-5170）。
- 重要副作用：比较成功即移除被比物品（堆叠取一件），并推 `"onnewtrophy"` 事件（components/trophyscale.lua:78-84；监听例 prefabs/trophyscale_fish.lua:366）。
- 修改边界：服务器组件；整份 `item_data` 随实体存读（:152-160）。

## 关键接口

| 接口/字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `type` | `"fish" \| "oversizedveggies"` | 赋值器同步 `trophyscale_<type>` 标签（:1-8）；动作筛选依赖该标签。 |
| `accepts_items` | `boolean = true` | false 时 `COMPARE_WEIGHABLE` 动作拒绝（actions.lua:4668-4669）；原版建筑建造中/烧毁时会切换（trophyscale_fish.lua:124、:164、:167）。 |
| `Compare` | `(inst_compare, doer) -> ok, reason\|nil` | 空纪录或更重时覆盖 `item_data`，移除被比物（堆叠取一件 :78-82），推事件；否则 `false, "<type>_TOO_SMALL"`。:57-90 |
| `TakeItem` | `(receiver) -> ok` | 经 `SpawnItemFromData` 生成物品入 receiver 背包，回调 `onitemtakenfn` 后清纪录。:131-150 |
| `SpawnItemFromData` | `(override_data\|nil) -> ent\|nil` | 按纪录 SpawnPrefab 并回填 weighable 的重量与归属字段。:96-117 |
| `SetItemCanBeTaken` | `(bool)` | 开关 `trophycanbetaken` 标签（:119-129）；原版建成即关（trophyscale_fish.lua:340）。 |
| `item_data` | `table\|nil` | `weight/is_heavy/prefab/build/owner_userid/owner_name/prefab_override_owner`；is_heavy 按 `TUNING.WEIGHABLE_HEAVY_WEIGHT_PERCENT`=0.8 判定。:63-72、tuning.lua:337 |
| 回调族 | `SetComparePostFn/SetOnSpawnItemFromDataFn/SetTakeItemTestFn/SetOnItemTakenFn` | 比较后、纪录实体生成、领取测试与领取后回调。:37-51 |

## 典型接入

### 自定义建筑接入比较（服务器）

- 前提与选择：类型标签必须进 pristine 阶段（客户端动作筛选依赖），组件在 master 端挂载；参照 trophyscale_fish.lua:314-315（标签）与 :336-340（组件）。
- 生效与撤销：比较动作由玩家对建筑使用 weighable 物品触发（componentactions.lua:896-914、:1451-1452）；纪录存于建筑，锤毁不自动转存（原版靠自身回调 DropItem）。

```lua
AddPrefabPostInit("my_scale", function(inst)
    if not TheWorld.ismastersim then
        return
    end
    inst:AddComponent("trophyscale")
    inst.components.trophyscale.type = TROPHYSCALE_TYPES.FISH
    inst.components.trophyscale:SetOnSpawnItemFromDataFn(function(item, data)
        -- 纪录实体生成后、入包前的调整入口
    end)
end)
```

- 调整与验证：客户端动作筛选只看标签（componentactions.lua:903-904），组件参数改动不影响动作可用性；标签没进 pristine 会导致客户端不出动作。

## 常见陷阱与限制

- `Compare` 直接访问 `inst_compare.components.weighable`（:58），缺组件会报错；动作入口已前置过滤（actions.lua:4667），绕过动作直调需自行保证。
- `TakeItem` 自身不做 `trophycanbetaken` 检查——门禁在 `takeitemtestfn` 与动作层（actions.lua:5160-5169）；直接调用会无条件生成并清纪录。
- `item_data` 整体作为存档数据（:152-154），往里加函数或实体引用会破坏存读。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，通读组件全部 161 行；核验两个原版建筑挂载（trophyscale_fish.lua:336-340、trophyscale_oversizedveggies.lua:401-405）与两个动作入口（actions.lua:4655-4675、:5154-5170）。
- 游戏运行：未运行。

## 相关链接

- 配对重量来源：[[reference/components/weighable]]；口袋秤同类标签组件：[[reference/components/itemweigher]]。
