---
title: "magiciantool"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "魔术工具侧使用者握手与开始／停止回调；原高礼帽动态转换、容器收尾及人物暂存协作。"
aliases: ["从魔术高礼帽结束当前会话", "魔术工具回调", "magiciantool"]
tags: [component, item, callback, lifecycle, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/magiciantool.lua", "components/magician.lua", "prefabs/hats.lua", "prefabs/waxwelljournal.lua", "actions.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# magiciantool

## 快速判断

- 职责与适用：工具侧保存一个 user，提供开始／停止业务回调；取物、隐藏、归还及人物状态由 [[reference/components/magician]] 管，不能只挂此组件就复现魔术帽容器。[工具定义](../../../raw/current/scripts/components/magiciantool.lua#L1-L30) [人物取物](../../../raw/current/scripts/components/magician.lua#L44-L76)
- 优先使用：SetOnStartUsingFn／SetOnStopUsingFn 配 fn(item,doer)，正常开始走人物 StartUsingTool；工具主动结束用 StopUsing，委托人物完整归还链。[接口](../../../raw/current/scripts/components/magiciantool.lua#L24-L62)
- 重要副作用：OnStartUsing 有 user 就直接返回，否则先记 user 再回调；OnStopUsing 只接受同一 doer，先清 user 再回调。返回值不作成功／失败协议，不能在 onstart 中返回 false 来回滚人物已取走的物品。[握手](../../../raw/current/scripts/components/magiciantool.lua#L33-L52) [人物调用](../../../raw/current/scripts/components/magician.lua#L62-L75)
- 修改边界：原 tophat 在转换时才添加组件、配置容器回调及装备／FX；读档按 data.magician 重建转换。构造时暂时没有组件不代表它永远不支持，不强行给所有帽子加组件。[转换](../../../raw/current/scripts/prefabs/hats.lua#L987-L1017) [恢复](../../../raw/current/scripts/prefabs/hats.lua#L1019-L1035)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `user`／`magiciantool` 标签 | user 初值 nil，构造加标签；它不是物品库存 owner，使用中物品已由人物暂存。直接改 user 不执行人物归还或容器回调。[默认](../../../raw/current/scripts/components/magiciantool.lua#L1-L8) [人物暂存](../../../raw/current/scripts/components/magician.lua#L60-L75) |
| 两个回调 setter | 各保存一个 fn(item,doer)，覆盖会替换原业务；原帽开始创建／打开代理并播放循环声，停止关闭、移除代理并停声。[设置](../../../raw/current/scripts/components/magiciantool.lua#L24-L30) [原业务](../../../raw/current/scripts/prefabs/hats.lua#L910-L934) |
| `OnStartUsing(doer)`／`OnStopUsing(doer)` | 由人物组件调用的握手；只改工具 user 与执行回调，不负责取出／归还工具，不能直接拿它们实现完整角色交接。[工具](../../../raw/current/scripts/components/magiciantool.lua#L33-L52) [人物停止](../../../raw/current/scripts/components/magician.lua#L78-L117) |
| `StopUsing()` | 有 user 且对端 magician 存在时委托其 StopUsing；没有对端则本地 OnStopUsing(user)。无成功布尔契约，也不修复其他模组造成的双向引用错配。[方法](../../../raw/current/scripts/components/magiciantool.lua#L54-L62) |
| 移组件／删实体 | 移组件先 StopUsing 再去标签；实体删除有 user 时先让人物 DropToolOnStop，再 StopUsing。后者改人物归还策略，不把这两个钩子当相同过程。[钩子](../../../raw/current/scripts/components/magiciantool.lua#L10-L22) [策略](../../../raw/current/scripts/components/magician.lua#L39-L42) |

原工具没有 user 的独立存读；转换标志和显示／装备回调由 hats.lua 保存重建，人物另保存暂存物品。新增工具仍要提供真实资源、业务清理及人物／动作／SG，不从 magiciantool 标签推断完整资格。[帽子存读](../../../raw/current/scripts/prefabs/hats.lua#L1019-L1068) [人物存读](../../../raw/current/scripts/components/magician.lua#L119-L138) [候选](../../../raw/current/scripts/componentactions.lua#L2774-L2788)

## 典型接入

### 从已转换的原高礼帽主动结束当前使用会话

目标模组 `modmain.lua`，无新资源；真实物品仍叫 tophat，tophat_magician 在原代码中是触发转换的配方名，不用作本例 PostInit 的 prefab 名。[命名](../../../raw/current/scripts/prefabs/hats.lua#L16-L20) [配方转换](../../../raw/current/scripts/prefabs/hats.lua#L1031-L1035)

```lua
AddPrefabPostInit("tophat", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:ListenForEvent("my_mod_stop_tophat_use", function(item)
        local tool = item.components.magiciantool
        if tool ~= nil then
            tool:StopUsing()
        end
    end)
end)
```

服务端向已取得的帽子发 `hat:PushEvent("my_mod_stop_tophat_use")`；普通未转换帽子／无使用者时不处理，转换后事件执行时再取组件。保留原转换、user 握手和容器关闭链，不主动强制转换或覆盖两回调；原容器的关闭回调本来也这样请求工具停止。[原关闭入口](../../../raw/current/scripts/prefabs/hats.lua#L906-L908) [委托](../../../raw/current/scripts/components/magiciantool.lua#L54-L62)

沿人物停止链恢复工具并发 magicianstopped，实际归还／回穿由人物现状决定；不单独 Remove 容器或手工清 user。停用并重载不再安装私有事件，但帽子原转换标志继续存档；本例没有按键、RPC 或新容器 UI。[人物收尾](../../../raw/current/scripts/components/magician.lua#L78-L117) [原回调](../../../raw/current/scripts/prefabs/hats.lua#L924-L934)

## 常见陷阱与限制

- **给 item.user 换个人就完成交接**：人物仍可能持有旧暂存关系；正常结束旧会话，再由新人物合法开始。内部握手不做完整交接和回滚。[握手](../../../raw/current/scripts/components/magiciantool.lua#L33-L62)
- **移掉 magiciantool 不用管容器／声音**：原业务清理来自停止回调；组件钩子依赖这条委托链，覆盖回调或破坏 user 后不能保证恢复。[移除](../../../raw/current/scripts/components/magiciantool.lua#L10-L22) [原清理](../../../raw/current/scripts/prefabs/hats.lua#L924-L934)
- **搜索到 waxwelljournal 的 TopHatSpellFn 就是当前转换入口**：该定义处于整段注释；当前可执行的帽子转换／配方／读档分支须单独核验，不把旧注释当正在运行的消费方案。[注释边界](../../../raw/current/scripts/prefabs/waxwelljournal.lua#L173-L201) [现行分支](../../../raw/current/scripts/prefabs/hats.lua#L987-L1035)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 中变更文件 actions.lua、componentactions.lua、prefabs/hats.lua、prefabs/waxwelljournal.lua（magiciantool.lua／magician.lua 本版未变）对照 build-26148036 更新报告核验：EQUIPPED.magiciantool 候选与 USEMAGICTOOL／STOPUSINGMAGICTOOL 动作无语义变化，waxwelljournal 变更在本页锚点之后且不触及注释段，hats 变更不触及高礼帽魔术业务（本页锚点整体行移 −1）；锚点重定位 11 处、结论无修订；未覆盖：运行验证。
- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整工具／人物组件、原高礼帽名称／转换／回调与存读、动作／候选及旧书注释边界。人物停止 SG 消费复用 magician 页；未完整审校容器代理、全部配方／转换调用、客户端预测、删实体与损坏引用恢复。
- 结构检查：本批格式、来源、源码引用／行号、大小和登记通过；完整帽子停止事件例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；转换前后、容器关闭／重复停止、人物回穿／掉落、存读、工具删除和其他模组包装待实测。
