---
title: "treegrowthsolution"
created: "2026-10-18"
updated: "2026-10-28"
type: component
summary: "执行树木催长并消费一份材料，区分目标自定义回调、普通成长、动作筛选与实际效果返回。"
aliases: ["树木催长", "催长剂无成长入口不消耗", "堆肥剂催长树木", "treegrowthsolution"]
tags: [component, item, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/treegrowthsolution.lua", "prefabs/treegrowthsolution.lua", "prefabs/oceantree.lua", "components/growable.lua", "componentactions.lua", "actions.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# treegrowthsolution

## 快速判断

- 职责与适用：材料侧的树木催长执行器，按目标回调或 growable 请求成长，再消费一份；不是持续成长倍率或现代农田施肥入口。[完整组件](../../../raw/current/scripts/components/treegrowthsolution.lua#L1-L34)
- 优先使用：沿正常 ADVANCE_TREE_GROWTH 动作执行 GrowTarget；已有自定义树木入口用目标 override_treegrowthsolution_fn，普通阶段链交 growable。[动作](../../../raw/current/scripts/actions.lua#L5679-L5685) [执行分派](../../../raw/current/scripts/components/treegrowthsolution.lua#L19-L31)
- 重要副作用：通过组件标签门禁后先生成可选 FX，再调成长，最后扣材料；回调返回及 DoGrowth 的 false 均未采用，甚至两种成长入口都没有也会消费并 true。[顺序](../../../raw/current/scripts/components/treegrowthsolution.lua#L6-L31) [DoGrowth 可失败](../../../raw/current/scripts/components/growable.lua#L121-L125)
- 修改边界：本页服务端。动作候选要求 tree 并排除 monster／leif 等，但直接 GrowTarget 只拒绝 no_force_grow／stump／fire／burnt；直接调用不含全部候选门禁或 target=nil 保护。[候选](../../../raw/current/scripts/componentactions.lua#L1869-L1879) [组件门禁](../../../raw/current/scripts/components/treegrowthsolution.lua#L6-L13)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `GrowTarget(target)` | 被拒标签时 false；完成消费路径 true | target 需为可调用 HasTag 的实体。true 只说明执行到结尾，不证明成长阶段改变或催长效果成功。[实现](../../../raw/current/scripts/components/treegrowthsolution.lua#L6-L32) |
| `fx_prefab` | 构造未赋值，可 nil | 非 nil 时先 SpawnPrefab 并定位目标坐标，未检查生成结果；无效 FX 可先报错而未进入成长／材料消费。[FX 分支](../../../raw/current/scripts/components/treegrowthsolution.lua#L15-L17) |
| `target.override_treegrowthsolution_fn` | 方法 `(target,item)` | 使用冒号调用，优先于 growable；返回值被忽略，false 不取消后续消费。[分派](../../../raw/current/scripts/components/treegrowthsolution.lua#L19-L23) |
| `target.components.growable:DoGrowth()` | 只在无自定义回调时调用 | 请求原阶段成长；停止状态可直接 false，组件仍消费。它不是把当前树强行设为最大阶段。[分派](../../../raw/current/scripts/components/treegrowthsolution.lua#L19-L23) [成长实现](../../../raw/current/scripts/components/growable.lua#L121-L150) |
| 材料消费 | 有多件 stackable 则件数减 1，否则 Remove | 不使用 finiteuses，也不调用 fertilizer:OnApplied；不要在自定义目标回调再次扣同一材料。[消费](../../../raw/current/scripts/components/treegrowthsolution.lua#L25-L31) |

原物品同时安装 fertilizer、treegrowthsolution、boatpatch、repairer 和部署 helper；树木催长、农田施肥及修复是不同消费者，不因同一物品就共享全部消耗／效果。农田三养分见 [[reference/components/fertilizer]]；普通阶段成长见 [[reference/components/growable]]。[物品组装](../../../raw/current/scripts/prefabs/treegrowthsolution.lua#L55-L80)

## 典型接入

### 催长剂遇到没有任何成长入口的目标时，返回失败且不扣材料

放目标模组 `modmain.lua`，无需新资源。包装原 treegrowthsolution 物品的 GrowTarget，在原 FX／消费之前补检查；保留已有标签拒绝、榕树特殊回调与原一份消费，不改其他施肥／修复路径。[原逻辑](../../../raw/current/scripts/components/treegrowthsolution.lua#L6-L32) [组件配置](../../../raw/current/scripts/prefabs/treegrowthsolution.lua#L61-L65) [注册](../../../raw/current/scripts/prefabs/treegrowthsolution.lua#L86) [PostInit](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("treegrowthsolution", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local solution = inst.components.treegrowthsolution
    if solution == nil then return end
    local original = solution.GrowTarget
    solution.GrowTarget = function(self, target)
        if target == nil or (target.override_treegrowthsolution_fn == nil
                and target.components.growable == nil) then
            return false
        end
        return original(self, target)
    end
end)
```

本例只保证没有入口的目标不执行原效果／消费，不保证存在入口时一定成长；停长的 growable 或返回 false 的自定义回调仍沿原消费行为。正常候选范围不扩大，也不会给任意植物新增菜单。停用并重新初始化恢复原方法，已变化的树阶段／剩余件数不回滚。

原版参照：oceantree 注册目标自定义回调，较小阶段走 DoGrowth，阶段 3 或已有高树进度时改为累计进度并安排 enriched_cooldown，后续定时逻辑才决定转高树；不能把每份效果统一理解为立即加一普通阶段。[目标接入](../../../raw/current/scripts/prefabs/oceantree.lua#L661-L665) [回调](../../../raw/current/scripts/prefabs/oceantree.lua#L475-L486) [定时消费者](../../../raw/current/scripts/prefabs/oceantree.lua#L452-L473)

## 常见陷阱与限制

- **目标回调 return false 就能阻止扣材料**：组件不读取其结果；需要修改明确的调用／消费协议，不能仅靠回调返回。本文示例只补入口存在性检查。[调用与消费](../../../raw/current/scripts/components/treegrowthsolution.lua#L19-L31)
- **调用 DoGrowth 等于持续加速**：它按现有阶段／计时状态执行一次，可失败；持续倍率／暂停恢复仍由 growable 管理。[DoGrowth](../../../raw/current/scripts/components/growable.lua#L121-L150)
- **有 tree 标签即可安全直接调用**：直接入口并不验证 tree，也不检查 FX 生成成功或接口实际效果；按真实目标与工厂核验，不绕开动作筛选后声称同等安全。[候选](../../../raw/current/scripts/componentactions.lua#L1869-L1879) [组件](../../../raw/current/scripts/components/treegrowthsolution.lua#L6-L31)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对本页 sources 变更文件 actions.lua、componentactions.lua（R-01/R-04）：ADVANCE_TREE_GROWTH 执行与 treegrowthsolution 候选未被本次 hunk 触及（componentactions 位移源于新增 spellcaster 处理器），结论无修订；锚点重定位 3 处；未覆盖：运行验证。

- 源码审阅：2026-10-18，Steam buildid 25643519；覆盖标签门禁、FX／成长／消费顺序、回调方法签名与返回忽略、原物品多用途、动作筛选、growable 停止返回和 oceantree 特殊进度／定时消费者。未全面审校榕树全生命周期、所有树种及船体修复路径。
- 结构检查：格式、来源、链接／行号、大小与登记通过；完整示例通过已有 Full Moon Lua 5.1 syntax-only，不证明实际成长、动画或消费效果。
- 游戏运行：未运行；单件／堆叠、停长／成熟树、榕树进度／读档、无效目标及其他模组方法包装顺序待实测。
