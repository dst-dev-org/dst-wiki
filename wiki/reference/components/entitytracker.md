---
title: "entitytracker"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "按名称记录关联实体、监听移除，并通过 GUID 引用恢复同一存档内的关系。"
aliases: ["关联实体", "记录建造者", "实体关系存档", "entitytracker"]
tags: [component, lifecycle, event]
dst_build: "steam-buildid:26230496"
sources: ["components/entitytracker.lua", "components/builder.lua", "prefabs/treasurechest.lua", "prefabs/wagboss_robot_leg.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# entitytracker

## 快速判断

- 职责与适用：给宿主保存“名称 → 实体”的关系；不是搜索器、随从控制器或网络同步器。[构造](../../../raw/current/scripts/components/entitytracker.lua#L1-L4)
- 优先使用：`ForgetEntity(name)` 后再 `TrackEntity(name, entity)` 安全替换；`GetEntity` 获取当前记录。[接口](#关键接口)
- 重要副作用：目标移除时自动清空该名称；只是忘记关系，不删除目标。[监听](../../../raw/current/scripts/components/entitytracker.lua#L20-L33)
- 修改边界：存档依赖 GUID 与加载后的实体映射，不能据此保证跨分片、重连后的玩家身份追踪；也不自动赋予访问权限。[保存恢复](../../../raw/current/scripts/components/entitytracker.lua#L39-L64)

## 关键接口

| 接口 | 条件、作用与证据 |
|---|---|
| `TrackEntity(name, inst)` | 目标须是可监听事件的实体；直接覆盖表项，不主动解除同名旧监听。[实现](../../../raw/current/scripts/components/entitytracker.lua#L20-L26) |
| `ForgetEntity(name)` | 有记录才解除目标的 `onremove` 监听并清空；无记录无操作。[实现](../../../raw/current/scripts/components/entitytracker.lua#L28-L33) |
| `GetEntity(name)` | 返回所记实体或 nil，不额外查找／校验有效性。[实现](../../../raw/current/scripts/components/entitytracker.lua#L35-L37) |
| `OnSave()` | 非空时返回 `{entities={{name, GUID}, ...}}` 与 GUID references；空表不返回数据。[实现](../../../raw/current/scripts/components/entitytracker.lua#L39-L53) |
| `LoadPostPass(ents, data)` | 仅恢复 `ents[GUID]` 中存在的实体，重新注册监听；缺失目标跳过。[实现](../../../raw/current/scripts/components/entitytracker.lua#L55-L64) |
| `OnRemoveFromEntity()` | 解除当前表项的监听；不负责处理曾被直接覆盖而遗留的监听。[实现](../../../raw/current/scripts/components/entitytracker.lua#L6-L10) |

## 典型接入

### 给新建木箱记录本次建造者

放入目标模组 `modmain.lua`；服务端为原版 `treasurechest` 增加关系记录，不改资源、箱子开关或原回调。建造流程向产物发送 `onbuilt`，数据含 `builder`；控制台生成／既有箱子不会补造建造记录。[事件发送](../../../raw/current/scripts/components/builder.lua#L818-L826)、[木箱监听位置](../../../raw/current/scripts/prefabs/treasurechest.lua#L176)、[Hook](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("treasurechest", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end
    if inst.components.entitytracker == nil then
        inst:AddComponent("entitytracker")
    end
    inst:ListenForEvent("onbuilt", function(chest, data)
        local builder = data ~= nil and data.builder or nil
        if builder ~= nil and builder:IsValid() then
            local tracker = chest.components.entitytracker
            tracker:ForgetEntity("mod_chest_builder")
            tracker:TrackEntity("mod_chest_builder", builder)
        end
    end)
end)
```

读取用 `chest.components.entitytracker:GetEntity("mod_chest_builder")`，主动解除用 `ForgetEntity`。此示例仅记录实体；玩家实体移除后记录消失，长期所有权应另设计稳定身份与授权逻辑，不用此关系直接做私有箱判断。

原版重绑参照：机械巨人腿更换 boss 前先解除自己的额外监听并 `ForgetEntity("boss")`，随后再绑定新 boss；宿主额外监听仍由宿主负责。[StartTrackingBoss](../../../raw/current/scripts/prefabs/wagboss_robot_leg.lua#L108-L122)

## 常见陷阱与限制

- **同名直接重绑：** 旧目标的回调仍能清空新表项，导致新目标“丢失”；先 Forget，再 Track。[覆盖与闭包](../../../raw/current/scripts/components/entitytracker.lua#L20-L33)
- **把关联当控制：** 不会移动、招募或同步目标；实际随从关系看 [[reference/components/leader]]／[[reference/components/follower]]。
- **存档引用不等于创建目标：** LoadPostPass 只认加载映射；未被加载的目标不会凭 GUID 重建，玩家跨世界也不属于本页保证范围。[恢复](../../../raw/current/scripts/components/entitytracker.lua#L55-L64)

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖关系增删、同名监听、GUID 保存恢复、木箱建造事件与原版重绑模式，依据见上文。未全面审阅所有实体保存／玩家迁移路径。
- 结构与语法：本批执行格式、登记、链接／源码锚点检查及完整示例 Lua 5.1 语法检查；结果见本批交付报告。
- 游戏运行：未运行；木箱建造、目标离场、保存恢复、专服及其他模组同名键冲突待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。