---
title: "shaveable"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "无毛发生长模型的剃取目标：资格、奖品和宿主回调；产后重置与再生由宿主管理。"
aliases: ["一级蜘蛛巢不允许剃取", "剃蜘蛛巢", "剃海草藤壶", "shaveable"]
tags: [component, creature, building, actions, callback, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/shaveable.lua", "actions.lua", "componentactions.lua", "prefabs/spiderden.lua", "prefabs/waterplant.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# shaveable

## 快速判断

- 职责与适用：目标端保存奖品及数量、调用资格与剃后回调；适合蜘蛛巢降级、海上植物剃取等宿主。需要毛发生长／分阶段剃毛的对象另用 beard；SHAVE 优先 beard，之后才 shaveable。[实现](../../../raw/current/scripts/components/shaveable.lua#L26-L65) [动作分流](../../../raw/current/scripts/actions.lua#L2833-L2843)
- 优先使用：SetPrize(prefab,count)，字段 can_shave_test／on_shaved 提供回调；Shave 自行调用 CanShave，不须重复手工产物循环。[入口](../../../raw/current/scripts/components/shaveable.lua#L26-L42)
- 重要副作用：先给奖品（继承目标世界湿度），后 on_shaved；组件不减少 prize_count、不自动再生，也不统一发 shaved 事件。[发放与回调](../../../raw/current/scripts/components/shaveable.lua#L45-L65)
- 修改边界：没有默认角色／死亡／燃烧／冷冻门禁；宿主必须实现。bearded 标签只随正 prize_count 联动，不能代替 CanShave 的所有检查。[标签](../../../raw/current/scripts/components/shaveable.lua#L1-L7) [默认测试](../../../raw/current/scripts/components/shaveable.lua#L31-L37)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `SetPrize(prize_prefab, prize_count)` | 两字段直接赋值；构造中的同名注释不是默认初始化。给有效 Prefab 名与非负整数；count>0 时有 bearded 标签。[构造](../../../raw/current/scripts/components/shaveable.lua#L9-L20) [设置](../../../raw/current/scripts/components/shaveable.lua#L26-L29) |
| `can_shave_test(target,shaver,implement)` | 可返回 pass,reason；没有测试默认 true。蜘蛛巢要求 spiderwhisperer 且活着、未燃烧／冻结。[测试接口](../../../raw/current/scripts/components/shaveable.lua#L31-L37) [巢门禁](../../../raw/current/scripts/prefabs/spiderden.lua#L632-L635) |
| `Shave(shaver,implement)` | 测试失败转交原因；配置奖品后生成 count 份，有执行者库存就 GiveItem，否则 LaunchAt；成功最终 true，哪怕未配置奖品。[结算](../../../raw/current/scripts/components/shaveable.lua#L39-L65) |
| `on_shaved(target,shaver,implement)` | 奖励后执行，返回值不参与成功判断；巢在一级被杀死，较高等级降级并清升级进度。包装要保留这些联动。[宿主后果](../../../raw/current/scripts/prefabs/spiderden.lua#L637-L662) |
| 存读／移除 | 仅保存 prize_count；恢复赋值联动标签，prize_prefab 和回调靠宿主重配。移除组件去 bearded 标签。[存读](../../../raw/current/scripts/components/shaveable.lua#L68-L76) [移除](../../../raw/current/scripts/components/shaveable.lua#L22-L24) |

## 典型接入

### 原蜘蛛巢保留高等级剃取，禁止剃毁一级巢

目标模组 `modmain.lua`，无新资源。对原文件注册的三种 Prefab 均安装附加测试；使用运行时 growable 阶段，不用 Prefab 名推断当前等级。原角色、健康、燃烧／冻结门禁和剃后降级仍保留。

```lua
local function ProtectSmallDen(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local shaveable = inst.components.shaveable
    if shaveable == nil then return end
    local previous = shaveable.can_shave_test
    shaveable.can_shave_test = function(target, shaver, implement)
        local growable = target.components.growable
        if growable == nil or growable:GetStage() <= 1 then
            return false
        end
        if previous ~= nil then
            return previous(target, shaver, implement)
        end
        return true
    end
end

for _, name in ipairs({ "spiderden", "spiderden_2", "spiderden_3" }) do
    AddPrefabPostInit(name, ProtectSmallDen)
end
```

原蜘蛛巢的 growable 阶段管理见 [[reference/components/growable]]；本例只加剃取限制，不阻止战斗／锤击／火烧。保存中没有此闭包，重载按构造重装；停用重载回原门禁，不回滚已发生的等级变化。原注册与回调依据为 [三工厂](../../../raw/current/scripts/prefabs/spiderden.lua#L838-L840)、[配置](../../../raw/current/scripts/prefabs/spiderden.lua#L813-L815) 与 [降级](../../../raw/current/scripts/prefabs/spiderden.lua#L637-L662)。

原版另一模式：waterplant 的 harvestable 生长／采收同步 prize_count，剃后清 harvestable.produce 并重新生长；同一份资源由宿主协调，不能仅另加奖励而保留采收库存。[同步](../../../raw/current/scripts/prefabs/waterplant.lua#L136-L162) [剃后](../../../raw/current/scripts/prefabs/waterplant.lua#L165-L185) [组装](../../../raw/current/scripts/prefabs/waterplant.lua#L446-L449)

## 常见陷阱与限制

- **Shave 会把数量扣到零**：没有这个步骤；宿主须通过资格、产量同步或阶段变化防止重复收益。只有组件而没有宿主消费协议容易重复产物。[完整结算](../../../raw/current/scripts/components/shaveable.lua#L39-L65)
- **蜘蛛巢候选只给韦伯就足够安全**：候选和服务器测试分开；自定义直接调用也须遵守宿主测试，shaver 不能随意传 nil，原巢测试直接读其标签。[候选](../../../raw/current/scripts/componentactions.lua#L1716-L1726) [门禁](../../../raw/current/scripts/prefabs/spiderden.lua#L632-L635)
- **与 beard 可随意同挂**：动作优先 beard，新增 shaveable 可能完全不被调用；先选模型，不用重复组件拼同一份奖励。[分流](../../../raw/current/scripts/actions.lua#L2833-L2843)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：sources 中漂移文件 actions.lua、componentactions.lua（更新报告 R-01/R-04）——SHAVE 分流与 shaver 候选处理器未变；锚点重定位 2 处；未覆盖：运行验证。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖完整组件、动作分流／候选、三巢工厂、准入／剃后降级和 waterplant 资源协作。growable 阶段接口复用已有页；未全面审校蜘蛛巢死亡／掉落和植物成长战斗。
- 结构检查：本批格式、来源、链接／行号、大小及登记通过；完整三巢门禁例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；三级到二级到一级、一级拒绝／角色与冻结拒绝、存读和其他模组包装顺序待实测。
