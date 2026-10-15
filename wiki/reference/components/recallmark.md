---
title: "recallmark"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "保存回溯坐标与分片 ID、复制及原怀表标记刷新；不负责实际移动、冷却或跨分片可用性。"
aliases: ["服务端重设回溯表标记", "回溯点与分片", "recallmark"]
tags: [component, item, callback, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/recallmark.lua", "prefabs/pocketwatch.lua", "prefabs/pocketwatch_common.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# recallmark

## 快速判断

- 职责与适用：物品侧保存 recall_x／y／z 和 recall_worldid，原 pocketwatch_recall 用它记回溯点；实际分片可用性、冷却与 SG 移动由怀表宿主处理。[标记](../../../raw/current/scripts/components/recallmark.lua#L7-L26) [消费](../../../raw/current/scripts/prefabs/pocketwatch.lua#L226-L247)
- 优先使用：MarkPosition(x,y,z,worldid?) 写入并触发原刷新回调，GetMarkedPosition 只读当前分片坐标；不要直接改字段而漏原标记 FX 更新。[写入](../../../raw/current/scripts/components/recallmark.lua#L7-L26) [读取](../../../raw/current/scripts/components/recallmark.lua#L35-L49) [原刷新](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L113-L135)
- 重要副作用：MarkPosition(nil) 不清除旧数据，却仍可调用 onMarkPosition 并返回 true；Copy 不透传成功／失败结果。两者都不是“清空标记”接口。[写入](../../../raw/current/scripts/components/recallmark.lua#L7-L33)
- 修改边界：标记仅用同一点到自身的传送许可检查，不证明未来落点安全或对应分片在线；原版施法另查 Shard_IsWorldAvailable 并写 warpback 状态。[许可](../../../raw/current/scripts/components/recallmark.lua#L7-L20) [宿主](../../../raw/current/scripts/prefabs/pocketwatch.lua#L226-L247)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `MarkPosition(x,y,z,worldid?)` | x 非 nil 才写数据；y／z 缺省为 0，worldid 缺省当前分片。许可失败返回 false,NO_TELEPORT_ZONE，旧数据与回调不动；成功返回 true。[方法](../../../raw/current/scripts/components/recallmark.lua#L7-L26) |
| `onMarkPosition` | 直接字段 fn(item,x,y,z,worldid)，传入 worldid 可能仍为 nil，即使已保存当前分片 ID；原回调读取组件当前数据重建 marker。[调用](../../../raw/current/scripts/components/recallmark.lua#L19-L25) [原消费者](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L113-L135) |
| `IsMarked()`／`IsMarkedForSameShard()` | 前者仅检查 worldid 非 nil，后者比较当前 ID；不校验分片可访问或坐标通行。[方法](../../../raw/current/scripts/components/recallmark.lua#L35-L41) |
| `GetMarkedPosition()` | 同分片返回 x,y,z，其他情况 nil；跨分片返回 nil 不表示记录不存在，不能据此覆盖远端标记。[方法](../../../raw/current/scripts/components/recallmark.lua#L43-L49) |
| `Copy(rhs)` | 参数是带 recallmark 的实体；经 MarkPosition 复制字段，不返回其结果，未标记源也不清旧记录。[方法](../../../raw/current/scripts/components/recallmark.lua#L28-L33) |
| `OnSave()`／`OnLoad(data)` | 保存四个标记字段，恢复时重新调用 MarkPosition，仍可能受当前区域许可影响；不是直接恢复已存在的 FX 或跨分片链接。[存读](../../../raw/current/scripts/components/recallmark.lua#L51-L64) |

## 典型接入

### 服务端将原回溯表重新标记到当前持有者位置

目标模组 `modmain.lua`，无新资源；原 pocketwatch_recall 已通过 MakeRecallMarkable 挂组件、设置回调并安装库存／掉落／移除监听，保留这些逻辑。[安装](../../../raw/current/scripts/prefabs/pocketwatch.lua#L288-L306) [辅助函数](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L143-L158)

```lua
AddPrefabPostInit("pocketwatch_recall", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:ListenForEvent("my_mod_remark_recall", function(item)
        local inventoryitem = item.components.inventoryitem
        local owner = inventoryitem ~= nil and inventoryitem:GetGrandOwner() or nil
        local mark = item.components.recallmark
        if mark == nil or owner == nil or not owner:HasTag("player")
            or owner:HasTag("playerghost") then return end
        local x, y, z = owner.Transform:GetWorldPosition()
        mark:MarkPosition(x, y, z)
    end)
end)
```

服务端模组逻辑向已取得的怀表发 `watch:PushEvent("my_mod_remark_recall")`，仅尝试重设标记，不执行回溯、施法冷却或新 RPC。末端持有者选择复用 [[reference/components/inventoryitem#关键接口]]；地面／非玩家／幽灵不处理。许可拒绝时旧记录保留，本例不提供 UI 反馈；原 onMarkPosition 处理已有 marker 的替换，不手工 Spawn 新 FX。[拒绝／刷新](../../../raw/current/scripts/components/recallmark.lua#L7-L26) [原 FX 链](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L97-L135)

成功后的四字段随原组件存档，移除本例不会恢复重设前的位置；停用并重载只是不再安装私有监听。它是明确的服务端模组入口，不替代原施法资格检查。原掉落清理的是显示／主人监听，不是把 recallmark 变回未标记。[保存](../../../raw/current/scripts/components/recallmark.lua#L51-L64) [掉落](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L104-L111)

## 常见陷阱与限制

- **MarkPosition(nil) 或 Copy(空表) 用来清标记**：不会清已有 worldid／坐标；构造的 recall_unmarked 标签也没有在此恢复。需要清空功能应另设计完整数据／标签／显示协议，不编造 Clear 方法。[完整写入](../../../raw/current/scripts/components/recallmark.lua#L1-L33)
- **GetMarkedPosition 返回 nil 就是未标记**：它隐藏其他分片坐标；先区分 IsMarked 与同分片，跨分片流程看实际宿主。[查询](../../../raw/current/scripts/components/recallmark.lua#L35-L49)
- **组件自动创建标记或完成传送**：原 marker、持有者／死亡／复活监听与 warpback 分别在辅助函数／怀表施法中，不由四字段提供。[显示](../../../raw/current/scripts/prefabs/pocketwatch_common.lua#L113-L158) [施法](../../../raw/current/scripts/prefabs/pocketwatch.lua#L226-L247)

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整 recallmark、原回溯表注册／安装／施法、标记辅助函数及显示清理；末端持有者接口复用同版本 inventoryitem 页。未完整审校 pocketwatch 能力组件、跨分片迁移 SG、marker 客户端、配方复制或存档区域变化。
- 结构检查：本批格式、来源、源码引用／行号、大小和登记通过；完整重设事件例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；库存／背包／地面、标记替换、区域拒绝、跨分片／读档、死亡／复活显示与其他模组监听待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。