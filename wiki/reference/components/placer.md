---
title: "placer"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "本地建造与部署预览的位置、吸附、可放置显示及输入协作，区别服务端实际生成。"
aliases: ["placer","建筑预览","放置虚影","手柄放置距离","网格吸附"]
tags: [component, recipe, ui, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/placer.lua","prefabutil.lua","components/playercontroller.lua","components/builder_replica.lua","components/builder.lua","prefabs/scienceprototyper.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# placer

## 快速判断

- 职责与适用：本地虚影的位置、旋转、吸附、合法颜色和输入；MakePlacer 建不联网、不保存、不可睡眠的独立预览 Prefab，不挂到建成建筑代替建造。[prefabutil.lua:1](../../../raw/current/scripts/prefabutil.lua#L1
- 优先使用：已有虚影用其注册名 PostInit；新对象由 MakePlacer 创建，再配 recipe.placer 或物品部署入口。SetBuilder 与实际 testfn 由 controller 接入。[playercontroller.lua:1547](../../../raw/current/scripts/components/playercontroller.lua#L1547)、[playercontroller.lua:2901](../../../raw/current/scripts/components/playercontroller.lua#L2901
- 重要副作用：SetBuilder 启动 wall-update；刷新会移动虚影、测试位置并触发附近 deployhelper。修改 oncanbuild／oncannotbuild 会替代对应默认显示分支。[placer.lua:189](../../../raw/current/scripts/components/placer.lua#L189)、[placer.lua:524](../../../raw/current/scripts/components/placer.lua#L524
- 修改边界：依赖本地 ThePlayer／输入／相机，绿影不代表服务端一定接受；实际地点制作仍检查缓冲、配方和 Map。[builder.lua:1113](../../../raw/current/scripts/components/builder.lua#L1113

## 关键接口

| 接口／字段 | 约定 | 条件、副作用与证据 |
|---|---|---|
| `SetBuilder(builder,recipe,invobject)` | 人物／配方／部署物品 | 保存上下文、调用 onbuilderset(inst)、启动更新；不是扣料或建成。[placer.lua:189](../../../raw/current/scripts/components/placer.lua#L189 |
| `testfn(pt,rot)`／`override_testfn(inst)` | 返回 can_build,mouse_blocked | override 优先；两者均无则 true,false。controller 会重新赋 testfn，不能在 Prefab PostInit 假定其已是最终函数。[placer.lua:231](../../../raw/current/scripts/components/placer.lua#L231)、[playercontroller.lua:1561](../../../raw/current/scripts/components/playercontroller.lua#L1561 |
| `offset` | 默认 1，世界单位 | 手柄普通定位用人物局部前方偏移；鼠标直接取世界点，部分吸附分支有自己的规则。[placer.lua:403](../../../raw/current/scripts/components/placer.lua#L403)、[placer.lua:453](../../../raw/current/scripts/components/placer.lua#L453 |
| `snap_to_tile/meters`／`snaptogrid` | 可选布尔字段 | 地皮中心／半整数坐标／整数网格；分支有优先级，不是几种吸附相加。[placer.lua:413](../../../raw/current/scripts/components/placer.lua#L413 |
| `fixedcameraoffset`／`rotationoffset` | 角度 | 相机朝向反向补偿及偏转；onupdatetransform(inst) 随后可修改位置／旋转，再测试。[placer.lua:508](../../../raw/current/scripts/components/placer.lua#L508 |
| `oncanbuild/oncannotbuild(inst,mouse_blocked)` | 表现回调 | 每次刷新可调用，不是边沿事件；自定义函数须自己负责显示／颜色／linked。[placer.lua:537](../../../raw/current/scripts/components/placer.lua#L537 |
| `LinkEntity(ent,lightoverride?)` | 视觉关联实体 | 登记并设置 LightOverride，默认 1；用于同步显隐／颜色，不自动设父级或包办实体删除。[placer.lua:208](../../../raw/current/scripts/components/placer.lua#L208)、[placer.lua:29](../../../raw/current/scripts/components/placer.lua#L29 |
| `GetDeployAction()` | 部署 BufferedAction 或 nil | 需要 invobject，记录 selected_pos，成功回调清该值；仍未执行动作。[placer.lua:215](../../../raw/current/scripts/components/placer.lua#L215 |

### 本地判断与执行分工

建筑预览的 testfn 走 replica.builder:CanBuildAtPoint→Map；部署预览走物品 replica.inventoryitem:CanDeploy，额外返回鼠标遮挡。override_build_point_fn 由 controller 取最终请求点，仅改虚影显示不等于改服务端规则。[builder_replica.lua:420](../../../raw/current/scripts/components/builder_replica.lua#L420)、[playercontroller.lua:2906](../../../raw/current/scripts/components/playercontroller.lua#L2906)、[playercontroller.lua:802](../../../raw/current/scripts/components/playercontroller.lua#L802

轴向辅助网格也会调用 TestCanBuild 测不同位置，测试函数应保持只读，不扣材料／生成实体／写计时器。[placer.lua:295](../../../raw/current/scripts/components/placer.lua#L295

## 典型接入

### 科学机器虚影在手柄下放到人物前方三单位

目标模组 `modmain.lua`，注册名是 researchlab_placer，复用原版 MakePlacer 的动画／资源；这是本地预览组件，不能加服务端提前返回。[scienceprototyper.lua:301](../../../raw/current/scripts/prefabs/scienceprototyper.lua#L301

```lua
AddPrefabPostInit("researchlab_placer", function(inst)
    local placer = inst.components.placer
    if placer ~= nil then
        placer.offset = 3
    end
end)
```

三为普通手柄定位的前方世界距离，不是修改 min_spacing、建造资格或鼠标位置；轴向网格开启时仍会吸附。原版 controller 负责创建、SetBuilder、测试和清理。停用后新预览恢复默认，无新增资源／任务；多人实际请求仍走原版校验。[placer.lua:486](../../../raw/current/scripts/components/placer.lua#L486)、[playercontroller.lua:1547](../../../raw/current/scripts/components/playercontroller.lua#L1547

## 常见陷阱与限制

- **虚影挂在服务端或给真实建筑加 placer**：MakePlacer 是本地独立对象，依赖本地输入；真实产物仍独立注册／生成。[prefabutil.lua:3](../../../raw/current/scripts/prefabutil.lua#L3
- **testfn 里扣材料，或只改它就放宽服务器规则**：预览与网格反复测试；实际 MakeRecipeAtPoint 重验地图，另维护必要的权威条件。[placer.lua:295](../../../raw/current/scripts/components/placer.lua#L295)、[builder.lua:1113](../../../raw/current/scripts/components/builder.lua#L1113
- **mouse_blocked 与 can_build 一样**：一个表示鼠标／显示遮挡，一个表示位置资格；二者影响显隐与物品图标，别只返回一个统一开关。[placer.lua:524](../../../raw/current/scripts/components/placer.lua#L524
- **LinkEntity 包办全部清理**：移除钩子只清轴向辅助和图标状态；额外关联 FX 需要父子／移除设计。[placer.lua:29](../../../raw/current/scripts/components/placer.lua#L29

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 0 处（playercontroller 引用区间未变——C-27 两处单行改动 :447/:2034 均不在本页引用的 802／1547／1561／2898／2903 行）；结论修订：无。
- 依据：player-hubs 辅组（C-27）＋变更清单命中 components/playercontroller.lua。

- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖组件配置／定位／测试／网格调用／显示／清理、MakePlacer 工厂、controller 建筑与部署接入、replica 地图检查和服务端地点制作。未全量审校网格几何、船边／半地皮算法、RPC 及全部特殊虚影。
- 结构检查：格式、来源、大小、链接／源码锚点及登记；完整示例经 Lua 5.1 syntax-only 检查。
- 游戏运行：未运行；鼠标／手柄、网格、相机、平台、皮肤和建造动作待测。

相关：[[reference/components/builder]]；[[reference/components/deployable]]；[[reference/components/deployhelper]]。
