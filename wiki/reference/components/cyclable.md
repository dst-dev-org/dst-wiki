---
title: "cyclable"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "循环切换离散档位并回调宿主，区分右键动作资格、直接切档和读档重放。"
aliases: ["循环切换模式", "右键切换档位", "贝壳音阶", "cyclable"]
tags: [component, callback, lifecycle, actions, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/cyclable.lua", "componentactions.lua", "actions.lua", "prefabs/singingshell.lua", "prefabs/carnivalgame_golf_tee.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# cyclable

## 快速判断

- 职责与适用：给实体保存一个 `1..num_steps` 档位，正向／反向循环后通知宿主；声音、动画、模式效果由回调实现。默认三档、第一档。[初始化](../../../raw/current/scripts/components/cyclable.lua#L17-L29) [切档](../../../raw/current/scripts/components/cyclable.lua#L48-L62)
- 优先使用：`SetStep(step, doer?, ignore_callback?)` 指定档位，`Cycle(doer?, negative?)` 循环一档；使用 `SetOnCycleFn(fn)` 绑定实际效果。[接口](../../../raw/current/scripts/components/cyclable.lua#L32-L62)
- 重要副作用：指定当前同一档也会回调；`OnLoad` 通过 `SetStep` 重放效果且没有 doer。回调须容忍初始化／读档，不宜无条件发一次性奖励。[指定](../../../raw/current/scripts/components/cyclable.lua#L40-L44) [加载](../../../raw/current/scripts/components/cyclable.lua#L72-L75)
- 修改边界：已核验宿主在服务端添加；`cancycle` 维护候选标签并被 CYCLE 动作复查，直接调用方法不会检查它。设置 false 只限制该动作路径。[宿主](../../../raw/current/scripts/prefabs/singingshell.lua#L165-L168) [安装](../../../raw/current/scripts/prefabs/singingshell.lua#L206-L209) [候选](../../../raw/current/scripts/componentactions.lua#L349-L351) [执行](../../../raw/current/scripts/actions.lua#L5181-L5188)

## 关键接口

| 接口／字段 | 约定、副作用与证据 |
|---|---|
| `SetNumSteps(num)`／`num_steps` | 赋值触发夹取现有 step，但不调 oncyclefn；不检查正整数或自动取整。宿主应提供合法正整数，改变档数后按需要再 SetStep 刷新效果。[属性](../../../raw/current/scripts/components/cyclable.lua#L13-L14) [方法](../../../raw/current/scripts/components/cyclable.lua#L32-L33) |
| `SetOnCycleFn(fn)` | 单回调 `fn(inst, step, doer)`，不是订阅列表；构造函数形参 activcb 没有被采用。设置函数也不立即应用当前档位。[构造](../../../raw/current/scripts/components/cyclable.lua#L17-L24) [设置](../../../raw/current/scripts/components/cyclable.lua#L36-L37) |
| `SetStep(step, doer?, ignore_callback?)` | 夹到至少 1、至多 num_steps（应为正数）；第三参为真仅改数值，不执行效果。回调返回值不撤销档位。[实现](../../../raw/current/scripts/components/cyclable.lua#L40-L44) |
| `Cycle(doer?, negative?)` | negative 为真减一，越过 1 绕到末档；否则加一，越界绕回 1。更新值后回调；没有计时、冷却或交互资格检查。[实现](../../../raw/current/scripts/components/cyclable.lua#L48-L62) |
| `cancycle` | 默认 true；属性增删同名标签。右键候选只查标签，动作执行查组件字段；两者不能代替自定义入口的权限校验。[属性](../../../raw/current/scripts/components/cyclable.lua#L1-L8) [候选](../../../raw/current/scripts/componentactions.lua#L349-L351) [动作](../../../raw/current/scripts/actions.lua#L5181-L5188) |
| `OnSave()`／`OnLoad(data)` | 仅保存 step；data 非 nil 时恢复并 SetStep，重新按当前档数夹取、执行回调。档数、cancycle、函数须由工厂／Hook 重建。[存读](../../../raw/current/scripts/components/cyclable.lua#L66-L75) |

## 典型接入

### 禁止贝壳右键调音，保留原有发声机制

目标模组 `modmain.lua`；使用原版 `singingshell_octave3/4/5`，不需要额外资源或 GLOBAL 代理。以下完整接入只关闭 CYCLE 动作资格，每次创建／加载实例都重新应用，不改变保存的音阶。[注册工厂](../../../raw/current/scripts/prefabs/singingshell.lua#L232) [原安装](../../../raw/current/scripts/prefabs/singingshell.lua#L206-L209)

```lua
for _, prefab in ipairs({"singingshell_octave3", "singingshell_octave4", "singingshell_octave5"}) do
    AddPrefabPostInit(prefab, function(inst)
        if not GLOBAL.TheWorld.ismastersim then return end
        if inst.components.cyclable ~= nil then
            inst.components.cyclable.cancycle = false
        end
    end)
end
```

这不会禁止作祟函数直接 `Cycle(haunter, negative)`，也不关闭发声或照料作物；原声音回调会使用新 step 播音并照料范围内植物。若目标是彻底固定音阶，需要分别核验／处理直接调用者，不应把本例当成总开关。[作祟](../../../raw/current/scripts/prefabs/singingshell.lua#L81-L84) [效果](../../../raw/current/scripts/prefabs/singingshell.lua#L52-L66)

停用模组并重新加载后，由原构造恢复 cancycle=true；不承诺移除运行中 Hook 后立即恢复现有实体。[默认](../../../raw/current/scripts/components/cyclable.lua#L17-L24)

### 按宿主选择回调与初始化顺序

- **有声音副作用的音阶**：贝壳先装回调、设 12 档，再 `SetStep(random, nil, true)`，避免初始随机选音就播音；读档则经组件 OnLoad 正常回调，二者有意不同。[初始化](../../../raw/current/scripts/prefabs/singingshell.lua#L206-L209) [播放](../../../raw/current/scripts/prefabs/singingshell.lua#L52-L66) [加载](../../../raw/current/scripts/components/cyclable.lua#L72-L75)
- **更新显示的数值档位**：嘉年华高尔夫球座设 10 档与第 1 档后才安装 OnCycle；回调更新标准杆符号，且用 POPULATING 避免部分动画／声音。宿主 SetPar 直接 SetStep，允许自定义时才开放 cancycle；不要只看动作标签推断档位不可变。[配置](../../../raw/current/scripts/prefabs/carnivalgame_golf_tee.lua#L437-L445) [显示／入口](../../../raw/current/scripts/prefabs/carnivalgame_golf_tee.lua#L330-L346) [开放](../../../raw/current/scripts/prefabs/carnivalgame_golf_tee.lua#L353-L356)

## 常见陷阱与限制

- **把 inactive 当成内置禁用字段**：贝壳 OnActivate 确实写 `cyclable.inactive=true`，但本组件切档与动作均不读取它。需要动作门禁用 cancycle；真正发声条件仍查宿主。[宿主赋值](../../../raw/current/scripts/prefabs/singingshell.lua#L69-L71) [直接切档](../../../raw/current/scripts/components/cyclable.lua#L48-L62) [执行](../../../raw/current/scripts/actions.lua#L5181-L5188)
- **读档回调假定有玩家**：OnLoad 没传 doer；贝壳 OnCycle 直接转发它给播放／照料流程。自定义回调应明确 nil 的处理，并区分恢复外观与玩家操作奖励。[恢复](../../../raw/current/scripts/components/cyclable.lua#L72-L75) [宿主](../../../raw/current/scripts/prefabs/singingshell.lua#L52-L66)
- **修改 num_steps 会同步所有模式表现**：该属性只夹数值，不回调、不注册网络变量；应通过宿主真实的网络／动画／声音入口应用效果，不能仅靠客户端读取服务端字段。[属性](../../../raw/current/scripts/components/cyclable.lua#L13-L14) [组件初始化](../../../raw/current/scripts/components/cyclable.lua#L17-L29) [宿主效果](../../../raw/current/scripts/prefabs/carnivalgame_golf_tee.lua#L330-L337)

## 验证记录

- 版本沿用：2026-10-25，`steam-buildid:26148036`；原结论复用、锚点迁移，见[本轮依据](../../updates/build-26148036/index.md#wiki-沿用记录)。
- 源码审阅：2026-10-23，Steam buildid 25643519；覆盖完整 cyclable、右键候选与动作执行、贝壳安装／发声／作祟及球座档位显示／开放条件。未全面审校贝壳触发器、植物照料机制或嘉年华存档系统。
- 结构检查：本页格式、大小、来源与本地链接、源码锚点已通过；完整 modmain 示例通过 Full Moon Lua 5.1 syntax-only，不代表 API 绑定或游戏效果已测。
- 游戏运行：未运行；客户端候选更新、作祟绕过、保存恢复回调及模组冲突待实测。
