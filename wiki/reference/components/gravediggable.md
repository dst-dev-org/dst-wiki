---
title: gravediggable
created: 2026-10-22
updated: "2026-10-28"
type: component
summary: 为可整体迁移的墓碑配置动作标签、拆除回调与开关存档，区分提示开关和执行授权。
aliases: [gravediggable, 可迁移墓碑, 阻止整墓迁移, 墓碑搬迁目标]
tags: [component, callback, lifecycle, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: [components/gravediggable.lua, components/gravedigger.lua, actions.lua, componentactions.lua, prefabs/gravestone.lua, prefabs/graveurn.lua, components/worldstate.lua]
reliability: verified
validation: source-reviewed
status: active
---

# gravediggable

## 快速判断

- 职责与适用：挂在目标墓碑上，维护 `gravediggable` 标签并委托 `ondug`；原版 `gravestone` 在公共构造预加标签，服务端添加组件。[组件](../../../raw/current/scripts/components/gravediggable.lua#L1-L21)、[原版安装](../../../raw/current/scripts/prefabs/gravestone.lua#L230)、[回调配置](../../../raw/current/scripts/prefabs/gravestone.lua#L255-L256)
- 优先使用：提示开关用 `canbedug`，实际拆除配置 `ondug`；如果拒绝条件必须阻止工具变化，应提前拦截完整动作，见下例。
- 重要副作用：`DigUp` 不检查 `canbedug`；无回调返回 true，有回调则返回其结果。完整动作先运行工具，再调用本方法。[组件方法](../../../raw/current/scripts/components/gravediggable.lua#L23-L30)、[动作顺序](../../../raw/current/scripts/actions.lua#L6231-L6245)
- 修改边界：`canbedug=false` 会撤标签，不是服务端执行授权；直接 DigUp 或已进入执行的动作均未检查它。[标签属性](../../../raw/current/scripts/components/gravediggable.lua#L1-L17)、[候选收集](../../../raw/current/scripts/componentactions.lua#L1334-L1338)

## 关键接口

| 接口／字段 | 参数或返回 | 行为与证据 |
|---|---|---|
| `canbedug` | 默认 true | 属性变化增删动作标签；不主动处理进行中的动作，也不驱动拆除。[属性](../../../raw/current/scripts/components/gravediggable.lua#L1-L17) |
| `ondug` | 可选 `fn(target, tool, doer) → success, reason` | 直接赋值；要明确返回成功。回调遗漏返回值时 DigUp 返回 nil，而不是沿用默认 true。[调用](../../../raw/current/scripts/components/gravediggable.lua#L23-L30) |
| `DigUp(tool, doer)` | 默认 `true, nil` | 不检查工具组件、玩家标签或目标开关；直接调用绕过完整动作前置逻辑。[组件实现](../../../raw/current/scripts/components/gravediggable.lua#L23-L30) |
| `OnSave()`／`OnLoad(data)` | `{canbedug=值}` | 保存开关；加载时假值或字段缺失都设置 false，真值不会主动把现有 false 改回 true。ondug 由宿主重建。[存读](../../../raw/current/scripts/components/gravediggable.lua#L33-L41) |

移除组件只清标签；实体移除、动画、墓堆处理和记录转移均由宿主负责。原墓碑 `OnDugUp` 移除此组件、播放动画、设 persists=false，并在动画结束后移除自己，同时侵蚀墓堆；它不负责给玩家生成迁墓物品。[清理](../../../raw/current/scripts/components/gravediggable.lua#L19-L21)、[原拆除](../../../raw/current/scripts/prefabs/gravestone.lua#L67-L84)

## 典型接入

### 只允许白天通过标准动作整体迁移墓碑

在目标模组 `modmain.lua` 加完整代码，不新增资源。选择包装 `GRAVEDIG.fn`，在工具 OnUsed 前拒绝；不把拒绝写到目标 ondug，避免骨灰坛已存墓、目标却没拆除的状态。`isday` 对应世界 day 阶段，不包含黄昏／夜晚；不使用洞穴独立的 `iscaveday`。[动作入口](../../../raw/current/scripts/actions.lua#L6231-L6245)、[阶段定义](../../../raw/current/scripts/components/worldstate.lua#L63-L79)

```lua
local previous_gravedig = GLOBAL.ACTIONS.GRAVEDIG.fn
GLOBAL.ACTIONS.GRAVEDIG.fn = function(act)
    if act.target ~= nil and act.target.prefab == "gravestone"
        and not GLOBAL.TheWorld.state.isday then
        return false
    end
    return previous_gravedig(act)
end
```

仅限制标准 GRAVEDIG 执行时的 gravestone，原技能、工具、目标回调、动画和存档仍走原路径。以**执行时**阶段为准，白天开始但执行前入夜也会拒绝；客户端提示／前摇仍可能出现，示例不新增失败台词。不影响普通挖墓堆或其他目标，也不阻止其他代码直接调用工具 OnUsed／目标 DigUp。

修改条件或移除包装并重载可撤销规则；示例不写 `canbedug`，不留下额外持久化开关，也不撤回此前已完成的迁移。其他模组也包装此动作时须保留旧函数并协商顺序。

## 常见陷阱与限制

- **只改 canbedug 就是绝对保护**：它只影响标签；完整动作没有复查该字段或技能标签。需要执行约束时在工具副作用前验证，直接调用者另负授权责任。[候选](../../../raw/current/scripts/componentactions.lua#L2360-L2364)、[执行](../../../raw/current/scripts/actions.lua#L6231-L6245)
- **ondug 返回 false 能无损拒绝**：工具回调已先执行；原骨灰坛可能已经保存记录、转为 deployable。前置拒绝方案见上例，工具流程见 [[reference/components/gravedigger#典型接入]]。
- **禁用开关在重载后自动恢复**：false 会保存；OnLoad 不区分旧数据缺字段与显式 false。临时禁用应制定恢复／迁移规则，不能把卸载模组当作存档回滚。[存读](../../../raw/current/scripts/components/gravediggable.lua#L33-L41)
- **缺 ondug 也会自动移除墓碑**：默认仅报告成功，没有删除或掉落。目标宿主必须实现实际转换；原版注册名是 gravestone，不是其可部署变体 dug_gravestone。[默认返回](../../../raw/current/scripts/components/gravediggable.lua#L23-L30)、[注册](../../../raw/current/scripts/prefabs/gravestone.lua#L434-L435)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 漂移文件 actions.lua、componentactions.lua 对照 build-26148036 更新报告（R-01、R-04）逐条核验，锚点重定位 4 处、结论修订 0 处（GRAVEDIG 动作执行链与 INVENTORY/EQUIPPED 两路候选未变，EQUIPPED.gravedigger 处理器仅整体位移；本次 nomagiccast／CORRUPTNIGHTMARE／USEITEM.spellcaster 等新增不触及本页）；未覆盖：运行验证。

- 源码审阅：2026-10-22，Steam buildid 25643519；覆盖完整组件、候选与执行差异、工具先执行、墓碑注册／原拆除和开关存读；白天约束示例核对世界阶段与原动作链。未完整审校墓地生成或皮肤、装饰和小幽灵系统。
- 结构与语法：本批定向检查格式、大小、链接及源码锚点；完整 modmain 示例另存后进行 Lua 5.1 syntax-only，未执行游戏代码。
- 游戏运行：未运行；白天转夜、提示与失败表现、旧档和其他模组动作包装待验证。
