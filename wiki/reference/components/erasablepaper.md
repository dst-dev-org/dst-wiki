---
title: "erasablepaper"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "将纸张物品在制图桌擦除为指定产物，控制单次返还数量与消耗、交付顺序。"
aliases: ["纸张回收产物", "地图擦除返纸", "蓝图回收", "erasablepaper"]
tags: [component, item, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/erasablepaper.lua", "components/papereraser.lua", "actions.lua", "componentactions.lua", "entityscript.lua", "prefabs/mapscroll.lua", "prefabs/papyrus.lua", "components/stackable.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# erasablepaper

## 快速判断

- 职责与适用：挂在被擦除的物品上，默认每次消耗一个输入，返还一张 `papyrus`；擦除目标由 [[reference/components/papereraser]] 承担。[默认值](../../../raw/current/scripts/components/erasablepaper.lua#L12-L17) [消耗](../../../raw/current/scripts/components/erasablepaper.lua#L43-L46)
- 优先使用：`SetErasedPrefab(prefab)` 设置产物，`SetStackSize(size)` 设置每次返还数量；原版 `mapscroll` 已有该组件，可直接 PostInit 调整。[设置](../../../raw/current/scripts/components/erasablepaper.lua#L19-L29) [宿主](../../../raw/current/scripts/prefabs/mapscroll.lua#L64-L75)
- 重要副作用：先生成并验证首个产物是物品，再消耗输入；随后交给 doer 物品栏，或放到擦除工具位置并调用 `OnDropped(true)`。[完整处理](../../../raw/current/scripts/components/erasablepaper.lua#L31-L65) [交付](../../../raw/current/scripts/components/erasablepaper.lua#L1-L9)
- 修改边界：原版只在服务端挂载并执行；无保存／恢复钩子，配置由构造／PostInit 重设。工具不消耗，纸张额外元数据不复制到产物。[宿主端边界](../../../raw/current/scripts/prefabs/mapscroll.lua#L91-L100) [完整组件](../../../raw/current/scripts/components/erasablepaper.lua#L12-L68)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetErasedPrefab(prefab)` | 已注册 Prefab 名 | 断言 `Prefabs[prefab]` 存在，只设置产物名；不提前验证是否物品，也不声明资源依赖。[设置](../../../raw/current/scripts/components/erasablepaper.lua#L19-L23) |
| `SetStackSize(size)` | 数值，默认 1 | 只断言 `size >= 1`，调用方应限制为合理正整数；不做取整或普通堆叠上限检查。[设置](../../../raw/current/scripts/components/erasablepaper.lua#L25-L29) |
| `DoErase(eraser, doer)` | 首个生成的产物实体；首个产物无效为 nil | 要求 eraser 可取得位置；有物品栏的 doer 走 `GiveItem`，否则掉在工具处。返回实体不等于已成功放入物品栏。[交付](../../../raw/current/scripts/components/erasablepaper.lua#L1-L9) [主体](../../../raw/current/scripts/components/erasablepaper.lua#L31-L65) |
| 输入与产物堆叠 | 每次消耗 1 个输入 | 输入为堆叠时 `Get():Remove()`；可堆叠产物用一实体设置数量，否则循环生成多个。[分支](../../../raw/current/scripts/components/erasablepaper.lua#L43-L61) [Get 默认量](../../../raw/current/scripts/components/stackable.lua#L109-L116) |

## 典型接入

### 地图卷轴擦除后返还两张莎草纸

目标模组的 `modmain.lua` 添加以下完整入口代码；沿用原版地图、制图桌和纸张资源，无需新动画。仅调整 `mapscroll`，不包含另一个注册名 `mapscroll_tricker`；后者虽然共用服务端工厂，也需另行注册 PostInit 才会命中。[共同组件](../../../raw/current/scripts/prefabs/mapscroll.lua#L64-L75) [两种工厂与注册](../../../raw/current/scripts/prefabs/mapscroll.lua#L91-L118)

```lua
AddPrefabPostInit("mapscroll", function(inst)
    if not GLOBAL.TheWorld.ismastersim then
        return
    end

    local paper = inst.components.erasablepaper
    if paper ~= nil then
        paper:SetStackSize(2)
    end
end)
```

默认产物 `papyrus` 保持不变，其工厂在服务端提供 `inventoryitem` 和 `stackable`，因此每次生成一堆、数量为 2；服务端动作仍要求目标具备 `papereraser` 且未着火／烧毁。若换自定义产物，应在模组声明实际 Prefab 与资源依赖，再设置已注册名称。[默认值](../../../raw/current/scripts/components/erasablepaper.lua#L12-L17) [产物组件](../../../raw/current/scripts/prefabs/papyrus.lua#L26-L48) [动作门禁](../../../raw/current/scripts/actions.lua#L3931-L3939)

配置在实例构造时生效，移除模组／本段代码后，新建或重新加载的实例恢复原版默认；本组件不持久化数量。需要物品栏交付／堆叠细节时查 [[reference/components/inventory]]、[[reference/components/stackable]]，不要在此复制分发机制。

## 常见陷阱与限制

- **把首个产物检查当成全程回滚：** 首个产物不存在或缺 `inventoryitem` 时保留输入；输入消耗后，非堆叠产物的后续生成没有 nil 防护，生成／交付失败并不回滚。自定义产物先核验注册与稳定生成。[顺序](../../../raw/current/scripts/components/erasablepaper.lua#L31-L65)
- **小数或超大数量：** setter 只检查下限；堆叠分支直接设置数量，非堆叠分支按 `for i = 2, size` 迭代，两种行为不能当作统一的输入修正。配置时自行限制合理整数。[分支](../../../raw/current/scripts/components/erasablepaper.lua#L25-L61) [底层 SetStackSize](../../../raw/current/scripts/components/stackable.lua#L103-L106)
- **把纸张变成工具：** 本组件是 `invobject` 侧，目标标签来自工作台；添加 `papereraser` 标签到纸张并不会替代这套动作方向。[动作发现](../../../raw/current/scripts/componentactions.lua#L1221-L1228) [动作执行](../../../raw/current/scripts/actions.lua#L3931-L3939)
- **自定义物品客户端没有完整组件：** 原版服务端添加组件时会登记并同步动作组件列表；无需为了动作发现在客户端构造本组件。新增网络物品仍须按可拾取物品骨架正确构造网络／pristine。[登记调用](../../../raw/current/scripts/entityscript.lua#L655-L665) [同步](../../../raw/current/scripts/componentactions.lua#L3182-L3187) 参见 [[reference/components/inventoryitem]]。
- **期待返还原纸张信息：** 实现只按 Prefab 生成新物品，没有把输入的地图、名称、皮肤或其他组件数据复制过去；这是回收为产物，不是清除某字段后保留原实例。[生成与移除](../../../raw/current/scripts/components/erasablepaper.lua#L31-L46)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件（actions.lua=R-01、componentactions.lua=R-04、entityscript.lua=R-08）对照更新报告逐条核验，锚点重定位 3 处、结论修订 0 处——ERASE_PAPER.fn、erasablepaper 候选块与 RegisterComponentActions 同步块原文未变，R-08 改动（GetWet 系／距离函数，746 行后）不触及 AddComponent 登记域；未覆盖：运行验证。

- 源码审阅：2026-10-23，`steam-buildid:25643519`；覆盖完整组件、工具转发、动作发现／执行及动作组件登记、地图两种注册工厂、莎草纸物品／堆叠组件和拆堆默认量。示例仅改原版地图返还数量。
- 游戏运行：未运行；自定义产物、满物品栏和跨模组覆盖未实测。
- 结构检查：页面格式／链接与源码锚点严格检查通过；完整 `modmain.lua` 示例通过 Lua 5.1 语法检查，未执行。
