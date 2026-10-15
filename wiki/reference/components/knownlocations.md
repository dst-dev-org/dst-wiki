---
title: "knownlocations"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "按名称保存位置快照并随实体存读，区分坐标记录、动态实体关联与真正回家行动。"
aliases: ["记录初始位置", "保存巡逻点", "生物记住坐标", "knownlocations"]
tags: [component, creature, ai, data, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/knownlocations.lua", "components/spawner.lua", "prefabs/pigman.lua", "brains/beebrain.lua", "mainfunctions.lua", "entityscript.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# knownlocations

## 快速判断

- 职责与适用：实体侧用名称保存坐标，序列化为 name/x/y/z 后随实体存读。没有移动、寻路、定时采样或网络同步接口。[记录与保存](../../../raw/current/scripts/components/knownlocations.lua#L29-L77)
- 优先使用：`RememberLocation(name, pos, true)` 仅在该名称不存在时记录，`GetLocation` 查询，`ForgetLocation` 删除；自定义键加模组前缀，避免覆盖原版 home。
- 重要副作用：保存和返回的都是原位置对象引用，不自动复制；修改同一对象可改变记录。dont_overwrite 只保护已有非 nil 项，不阻止读档的 DeserializeLocations 覆盖同名项。[记录](../../../raw/current/scripts/components/knownlocations.lua#L61-L77) [反序列化](../../../raw/current/scripts/components/knownlocations.lua#L43-L59)
- 修改边界：坐标不会跟随家园实体移动；动态关联见 [[reference/components/entitytracker]]，家园引用／回家请求见 [[reference/components/homeseeker]]。生成器会单独设置 home 坐标，不能把本组件当成巢穴登记。[生成器协作](../../../raw/current/scripts/components/spawner.lua#L186-L209)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `locations` | 初始空表 | name→位置对象；没有预置 home 或自动登记，调试字符串只遍历已有记录。[构造](../../../raw/current/scripts/components/knownlocations.lua#L1-L4) |
| `RememberLocation(name, pos, dont_overwrite)` | 无显式返回 | 默认覆盖；第三参为真且原项非 nil 时不写。pos=nil 可清项。写入后才检查 x/y/z 是否坏数并报错，不把错误当安全拒绝／回滚；传有效有限坐标。[实现](../../../raw/current/scripts/components/knownlocations.lua#L61-L69) |
| `GetLocation(name)`／`ForgetLocation(name)` | 位置或 nil／无显式返回 | Get 返回同一引用，Forget 只清该项；不撤销正在执行的行为或家园关联。[实现](../../../raw/current/scripts/components/knownlocations.lua#L71-L77) |
| `SerializeLocations()`／`DeserializeLocations(data)` | 数组或 nil／无显式返回 | 序列化输出 name/x/y/z；反序列化创建 Vector3 并逐项写入，不先清空旧表，也不接受任意格式的“路径对象”。[实现](../../../raw/current/scripts/components/knownlocations.lua#L29-L47) |
| `OnSave()`／`OnLoad(data)` | `{locations=...}` 或 nil | 空记录无保存数据；有 locations 才恢复。宿主正常组件存读已调用，无需再复制一份到 inst.OnSave。[组件](../../../raw/current/scripts/components/knownlocations.lua#L49-L59) [宿主保存](../../../raw/current/scripts/entityscript.lua#L1928-L1950) [宿主加载](../../../raw/current/scripts/entityscript.lua#L1992-L2012) |

原版用法由消费者决定：工蜂 Brain 在初始化完成时记 home，Wander 查询该坐标；spawner 接管子体时另写家园当前位置，换掉旧子体会清其 home 坐标。键名本身没有“自动回家”的魔法。[工蜂消费者](../../../raw/current/scripts/brains/beebrain.lua#L80-L88) [生成器](../../../raw/current/scripts/components/spawner.lua#L186-L209)

## 典型接入

### 为普通猪人记录首次延迟初始化时的位置，读档后保留

放目标模组 `modmain.lua`，复用 pigman 原有组件，无新资源。使用自定义键 `my_mod_origin`，在零延迟任务中取 GetPosition；工厂 PostInit 运行时最终放置／存读可能尚未完成，不能把同步读取坐标称为出生点。原恢复流程先 SpawnPrefab，再放置、SetPersistData；本例延后记录并以第三参 true 保留已恢复的键。[猪人安装](../../../raw/current/scripts/prefabs/pigman.lua#L701) [PostInit 调用](../../../raw/current/scripts/mainfunctions.lua#L387-L417) [恢复顺序](../../../raw/current/scripts/mainfunctions.lua#L464-L503) [注册](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("pigman", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:DoTaskInTime(0, function(pig)
        if not pig:IsValid() then return end
        local locations = pig.components.knownlocations
        if locations ~= nil then
            locations:RememberLocation("my_mod_origin", pig:GetPosition(), true)
        end
    end)
end)
```

后续服务端逻辑通过 `pig.components.knownlocations:GetLocation("my_mod_origin")` 读取；这是数据接入示例，不包括巡逻或返回动作。GetPosition 创建位置对象，本例不持有／修改该对象；原版 home、Brain 和家园归属不变。[取位置](../../../raw/current/scripts/entityscript.lua#L1381-L1382)

记录表示任务实际执行时的位置：猪人可能已被巢穴收进屋、被其他逻辑移动，不能当作绝对历史出生坐标。旧档没有键就首次补记，已有键通过原存读链恢复；若代码更晚异步放置，应在其完成后记录。停用不等于清除已保存键；需要删除时显式 ForgetLocation，并按目标模组保存／迁移约定处理。

## 常见陷阱与限制

- **改 GetLocation 返回值只影响临时变量**：对象没有复制；需要独立调整时自行构造位置副本，避免直接改写共享记录。[引用返回](../../../raw/current/scripts/components/knownlocations.lua#L61-L77)
- **home 一定跟随房屋／等于当前有效家园**：它只是坐标记录，生成器写入时才更新；家园销毁和动态引用走其他组件，不靠这个键判断接收资格。[协作](../../../raw/current/scripts/components/spawner.lua#L186-L209)
- **挂了组件就会重载恢复／自动走回原点**：组件要在宿主恢复时存在，正常存读需该实体实际被保存；行为须另接消费者。本例使用原 pigman 组件，不给任意实体承诺持久化或同步。[宿主加载](../../../raw/current/scripts/entityscript.lua#L1992-L2012) [工蜂行为](../../../raw/current/scripts/brains/beebrain.lua#L80-L88)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 5 处（entityscript 存／读／GetPosition、mainfunctions PostInit 与恢复顺序；pigman knownlocations 行号未变）；无结论修订，仅锚点迁移——依据：全局枢纽 diff 审阅（R-08 湿润／距离函数改动不涉存读分发）＋变更清单命中 mainfunctions.lua、entityscript.lua、pigman.lua（本版仅饮食／标签调整，knownlocations 装配行未动），knownlocations.lua、spawner.lua、beebrain.lua 未变。
- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖记录／引用／删除、序列化合并／空数据、猪人安装、工蜂消费、spawner 家园协作和必要工厂／存读顺序。未审校所有 Brain、自定义异步放置、分片迁移或全实体保存策略。
- 结构检查：格式、来源、链接／锚点、大小与唯一登记通过；完整延迟记录示例通过已有 Full Moon Lua 5.1 syntax-only，不证明实际保存／游戏行为。
- 游戏运行：未运行；新建／读档、进屋／出屋、提前移除、延迟放置、停用后数据清理及多模组改键待实测。
