---
title: "fillable"
created: "2026-10-18"
updated: "2026-10-28"
type: component
summary: "实现容器补水或填充后换物，明确水源先消费、海水动作门禁与自定义回调返回边界。"
aliases: ["洒水壶装海水", "容器补水", "填充后替换物品", "fillable"]
tags: [component, item, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/fillable.lua", "components/watersource.lua", "prefabs/wateringcan.lua", "prefabs/mosquitosack.lua", "actions.lua", "componentactions.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# fillable

## 快速判断

- 职责与适用：被填充物品侧的执行器；默认生成 filledprefab 并移除一份原物，也可由 overrideonfillfn 接管为原地补次数。[Fill](../../../raw/current/scripts/components/fillable.lua#L28-L62)
- 优先使用：换成另一种物品设 filledprefab；原地补水保留／包装 overrideonfillfn。海水候选显示与能否接受分别由 showoceanaction、acceptsoceanwater 控制。[原壶配置](../../../raw/current/scripts/prefabs/wateringcan.lua#L140-L149) [海水动作](../../../raw/current/scripts/actions.lua#L2150-L2158)
- 重要副作用：Fill 先对输入水源 Use，再调用回调或生成产物；返回 false 不保证未消耗水源，也没有自动回滚。[执行顺序](../../../raw/current/scripts/components/fillable.lua#L28-L44)
- 修改边界：本页服务端；Fill 不核验水源 available／标签、海水资格或当前容量。正常动作另有门禁，直接调用不是完整补水动作。[实现](../../../raw/current/scripts/components/fillable.lua#L28-L44) [动作](../../../raw/current/scripts/actions.lua#L2126-L2164)

## 关键接口

| 接口／字段 | 参数或默认 | 作用、条件与证据 |
|---|---|---|
| `Fill(from_object?)` | 默认分支成功 true／失败 false；回调分支原样返回 | 输入对象有 watersource 就先 Use；nil 输入不消耗实体水源，常用于海水动作。[实现](../../../raw/current/scripts/components/fillable.lua#L28-L44) |
| `overrideonfillfn` | `(item,from_object?) -> 结果` | 有回调便完全接管后续逻辑，不自动执行 filledprefab 替换；洒水壶按水源 override_fill_uses 加次数，无该字段则补满，最后 true。[委托](../../../raw/current/scripts/components/fillable.lua#L33-L35) [消费者](../../../raw/current/scripts/prefabs/wateringcan.lua#L26-L37) |
| `filledprefab` | 默认 nil | 无回调时必须为可生成 Prefab；生成失败 false，此时水源可能已消费。原蚊子血囊设置为 waterballoon。[生成](../../../raw/current/scripts/components/fillable.lua#L37-L44) [原例](../../../raw/current/scripts/prefabs/mosquitosack.lua#L42-L43) |
| `showoceanaction` | 默认 false，属性回调 | 同时维护 fillable_showoceanaction／allow_action_on_impassable 标签，只决定候选显示；不等于允许海水。原洒水壶为 true，但 acceptsoceanwater=false。[属性](../../../raw/current/scripts/components/fillable.lua#L1-L20) [原配置](../../../raw/current/scripts/prefabs/wateringcan.lua#L140-L144) |
| `acceptsoceanwater`／`oceanwatererrorreason` | 默认 false／未赋值 | 由 FILL／FILL_OCEAN 地点分支读取；拒绝时返回 false 和理由，Fill 本身不读这两个字段。[动作消费者](../../../raw/current/scripts/actions.lua#L2150-L2166) |
| `OnRemoveFromEntity()` | 移除三个组件相关标签 | 移除 fillable、fillable_showoceanaction、allow_action_on_impassable；没有此组件自有的存档字段实现。[清理](../../../raw/current/scripts/components/fillable.lua#L22-L26) |

默认替换先取 GrandOwner，向其 inventory／container 请求 RemoveItem(self.inst,false)，失败会回退 self.inst，再删原物并尝试 GiveItem 新物；无持有者时在原坐标生成，堆叠则 Get 一份。不要据此承诺任意嵌套容器都只消耗一颗、保持原格或完整转移新鲜度／自定义数据。[持有与地面分支](../../../raw/current/scripts/components/fillable.lua#L46-L61)

## 典型接入

### 允许普通洒水壶在海边装满，保留原装水回调

放目标模组 `modmain.lua`，无需新资源；普通壶本来就显示海水动作但执行时拒绝，本例只改接受开关，不重写 Fill 或消耗水源。原无 from_object 的 OnFill 分支补满 finiteuses。[原入口](../../../raw/current/scripts/prefabs/wateringcan.lua#L140-L149) [补满](../../../raw/current/scripts/prefabs/wateringcan.lua#L26-L37) [注册](../../../raw/current/scripts/prefabs/wateringcan.lua#L174-L175) [PostInit](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("wateringcan", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local fillable = inst.components.fillable
    if fillable ~= nil then
        fillable.acceptsoceanwater = true
    end
end)
```

生效于后续海水地点动作，不影响高级壶或实体水源份量；不会产生“盐水物品”、盐度或净水机制。停用并重新初始化恢复拒绝，已补次数不回减。新建其他物品若也要海水入口，须同时核验 showoceanaction 与实际动作／加载环境，不机械照搬一个开关。

原版两种模式：蚊子血囊用 filledprefab 换水球，洒水壶用 overrideonfillfn 原地补次数；不能只设置原壶的 filledprefab 就期待替换，因为回调优先。水源消费／份量见 [[reference/components/watersource]]，用水效果见 [[reference/components/wateryprotection]]。[换物](../../../raw/current/scripts/prefabs/mosquitosack.lua#L42-L43) [原地模式](../../../raw/current/scripts/prefabs/wateringcan.lua#L26-L37)

## 常见陷阱与限制

- **Fill 返回 false 就可无代价重试**：Use 发生在生成／回调之前，没有回滚；自定义调用方先校验前提，不在回调里重复消费水源。[顺序](../../../raw/current/scripts/components/fillable.lua#L28-L44)
- **海水动作成功证明回调成功**：该动作在资格通过后调用 Fill()，忽略其返回并直接 true；本例保留原壶返回 true 的回调，但其他物品必须分别检查动作和回调结果。[地点分支](../../../raw/current/scripts/actions.lua#L2150-L2158)
- **showoceanaction=true 就允许海水**：候选仅查标签／海洋点，服务端还查 acceptsoceanwater；反过来仅设置接受开关也不替新物品补出候选标签。[候选](../../../raw/current/scripts/componentactions.lua#L2109-L2113) [执行](../../../raw/current/scripts/actions.lua#L2150-L2158)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页漂移文件 actions.lua（R-01）、componentactions.lua（R-04）对照 build-26148036 更新报告与 actions／componentactions hunk 地图逐条核验；FILL／FILL_OCEAN 动作定义、执行分支与 USEITEM.fillable 海水候选均未在变更 hunk 内，结论不变，锚点重定位 6 处、结论修订 0 处；未覆盖：运行验证。

- 源码审阅：2026-10-18，Steam buildid 25643519；覆盖回调／替换优先级、先消费、持有与地面分支、标签／移除、实体／海水动作及原壶／血囊两种模式。未全面审校全部容器转移或所有装水物品。
- 结构检查：格式、来源、链接／行号、大小与登记通过；完整示例通过已有 Full Moon Lua 5.1 syntax-only，不证明海边路径或实际装水。
- 游戏运行：未运行；手柄／岸边可达性、满壶、来源消耗、嵌套容器替换、堆叠及其他模组回调兼容待实测。
