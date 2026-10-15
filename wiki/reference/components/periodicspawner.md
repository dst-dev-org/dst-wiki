---
title: "periodicspawner"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "按随机间隔尝试生成独立产物，提供密度／间距、生成测试、重试与保存计时，区分住户管理。"
aliases: ["格罗姆后续产黏液加快", "周期生成物品", "生成密度限制", "periodicspawner"]
tags: [component, creature, item, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/periodicspawner.lua", "prefabs/glommer.lua", "stategraphs/SGglommer.lua", "prefabs/babybeefalo.lua", "prefabs/grassgator.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# periodicspawner

## 快速判断

- 职责与适用：宿主按间隔尝试产生独立对象，可加资格、附近密度与间距；不登记子体／家园，不负责召回和住户数量。[生成](../../../raw/current/scripts/components/periodicspawner.lua#L117-L202) 住户与巢内储备分别见 [[reference/components/spawner]]／[[reference/components/childspawner]]。
- 优先使用：`SetRandomTimes(base, variance, no_reset)` 定后续间隔，Start／SafeStart／Stop 控制排程，生成条件用测试或密度配置。随机公式是 `base + random()*variance`，默认 40～100 秒，不是 base±variance。[初值](../../../raw/current/scripts/components/periodicspawner.lua#L14-L29) [设置／启动](../../../raw/current/scripts/components/periodicspawner.lua#L39-L111)
- 重要副作用：DoSpawn／ForceNextSpawn 都会尝试生成并重启下一轮，仍受门禁；TrySpawn 只尝试、不排下一轮，返回 true 也不保证 ent 非 nil。回调必须按实际生成结果处理。[尝试结果](../../../raw/current/scripts/components/periodicspawner.lua#L178-L202) [别名](../../../raw/current/scripts/components/periodicspawner.lua#L239-L240)
- 修改边界：本页核验格罗姆与小牛主模拟宿主；间隔缩短不承诺提高最终产量，密度、离屏、海上分支与宿主 SG 仍影响结果。读档保存剩余时间，不保存全部配置／停用状态。[格罗姆](../../../raw/current/scripts/prefabs/glommer.lua#L106-L147) [保存](../../../raw/current/scripts/components/periodicspawner.lua#L223-L240)

## 关键接口

| 接口／字段 | 参数或返回 | 条件、作用与证据 |
|---|---|---|
| `SetPrefab(prefab)` | 已注册名或 fn(host) | 字符串须存在于 Prefabs，函数的返回候选在尝试时求值；setter 不启动排程，也不保证函数永远返回有效对象名。[设置](../../../raw/current/scripts/components/periodicspawner.lua#L33-L37) [求值](../../../raw/current/scripts/components/periodicspawner.lua#L117-L125) |
| `SetRandomTimes(base, variance, no_reset)` | 数字秒数；第三参可选 | 写 basetime／randtime；已有 task 且 no_reset 不为真时立即 Start 重排。仅断言数字，不替你保证正间隔，调用方应使用有效正时长。[实现](../../../raw/current/scripts/components/periodicspawner.lua#L39-L48) [公式](../../../raw/current/scripts/components/periodicspawner.lua#L89-L96) |
| `Start(timeoverride)`／`SafeStart(timeoverride)`／`Stop()` | 明确秒数或随机／无参数 | Start 先 Stop 再设置 target_time 和任务；SafeStart 只在 target_time=nil 时启动；Stop 清时刻／任务，不删除已生成对象。[实现](../../../raw/current/scripts/components/periodicspawner.lua#L89-L111) |
| `SetDensityInRange(range, density)`／`SetMinimumSpacing(spacing)` | 数字 | 在宿主周围搜索，排除 INLIMBO；默认只数同 prefab，也可用额外过滤。只设 spacing 而未给正 density，TrySpawn 会直接拒绝。[设置](../../../raw/current/scripts/components/periodicspawner.lua#L50-L61) [实际筛选](../../../raw/current/scripts/components/periodicspawner.lua#L139-L169) |
| `SetDensityFilterFn(fn)` | fn(host, nearby) | 与 nearby.prefab==当前产物名做 OR，增加计数对象，不替换原同类匹配。触发间距冲突或 count>=density 时拒绝。[设置](../../../raw/current/scripts/components/periodicspawner.lua#L83-L85) [消费](../../../raw/current/scripts/components/periodicspawner.lua#L149-L169) |
| `SetOnlySpawnOffscreen(true)`／`SetSpawnTestFn(fn)` | 离屏限制／fn(host) | 前者要求宿主 IsAsleep，后者失败拒绝；不是离屏时自动补齐所有漏产。资格失败给 base×0.25 重试时长。[门禁](../../../raw/current/scripts/components/periodicspawner.lua#L117-L125) |
| `SetGetSpawnPointFn(fn)` | fn(host)→位置或 nil | 默认宿主位置，回调为 nil 时拒绝并给 15 秒重试；密度候选仍从宿主周围取，间距的部分判定使用新落点，远距离自定义点不能假定检查覆盖完整。[落点](../../../raw/current/scripts/components/periodicspawner.lua#L127-L169) |
| `SetOnSpawnFn(fn)`／`TrySpawn(prefab)` | fn(host, ent)／boolean, 可选重试秒数 | TrySpawn 不返回产物实体；走到末尾无条件 true，即使 SpawnPrefab 返回 nil，onspawn 也会收到 nil。密度拒绝只返回 false，无专用短重试。[生成回调](../../../raw/current/scripts/components/periodicspawner.lua#L178-L197) |
| `DoSpawn()`／`ForceNextSpawn()` | 无显式返回 | 同一实现：TrySpawn 后按其第二返回值或正常随机间隔 Start；并不跳过测试、密度或海上分支。手动调用会重启原本停止的调度。[实现](../../../raw/current/scripts/components/periodicspawner.lua#L199-L202) [别名](../../../raw/current/scripts/components/periodicspawner.lua#L239-L240) |

### 生成、离屏与存读

海上非视觉陆地且无平台、世界有 flotsamgenerator、未设 ignoreflotsamgenerator 时转交 SpawnFlotsam；其他分支 SpawnPrefab 并定位。该开关只跳过转交，不提供通用有效地面／避障保证。[分支](../../../raw/current/scripts/components/periodicspawner.lua#L178-L196)

LongUpdate 过期时只 DoSpawn 一次，未过期则重排剩余时长，不按漏掉的周期循环补产。OnSave 只存未来 target_time 的向下取整剩余秒数，OnLoad 有 time 才 Start；Stop 的停用状态不单独入档，原 Prefab 的 Start 可在重建时重新开启。组件移除调用 Stop，不回收此前产物。[追赶](../../../raw/current/scripts/components/periodicspawner.lua#L207-L219) [保存／清理](../../../raw/current/scripts/components/periodicspawner.lua#L223-L240)

## 典型接入

### 格罗姆后续产黏液间隔减半，保留当前倒计时

放目标模组 `modmain.lua`，无需新资源。原 glommer 已设置产物、回调及 2～4 个游戏日的间隔，并已 Start；本例把 base 和随机增量同时乘 0.5，第三参 true 保留这次任务，后续正常间隔变为 1～2 个游戏日。[原配置](../../../raw/current/scripts/prefabs/glommer.lua#L139-L147) [设置语义](../../../raw/current/scripts/components/periodicspawner.lua#L39-L48) [Hook](../../../raw/current/scripts/modutil.lua#L593-L599)

```lua
AddPrefabPostInit("glommer", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local spawner = inst.components.periodicspawner
    if spawner ~= nil then
        spawner:SetRandomTimes(spawner.basetime * 0.5, spawner.randtime * 0.5, true)
    end
end)
```

新生成格罗姆的首轮原任务也被保留；读档仍由 OnLoad 恢复原剩余秒数，不自动按比例缩短。后续 DoSpawn 才使用新 base／randtime，停用重载恢复后续默认参数，已存倒计时不回滚。不要每帧重复调用重排，否则可能一直等不到到期。[循环](../../../raw/current/scripts/components/periodicspawner.lua#L199-L202) [读档](../../../raw/current/scripts/components/periodicspawner.lua#L223-L234)

保留原 OnSpawnFuel：它把产物交给 SG 的 goo 状态，先隐藏，在动画时间线再显示；本例只改计时，不改动画／产物／跟随。不能把回调视作尚未生成产物的“生成前”通知。[回调](../../../raw/current/scripts/prefabs/glommer.lua#L59-L61) [状态消费者](../../../raw/current/scripts/stategraphs/SGglommer.lua#L71-L100)

另一实际模式是 babybeefalo 的 poop 产物、20 范围内 2 个的密度限制和最小间距 8；不能把这些门禁推给未配置密度的格罗姆。grassgator 搜索命中的同类挂载在块注释内，不作为活跃宿主示例。[小牛](../../../raw/current/scripts/prefabs/babybeefalo.lua#L165-L170) [注释分界](../../../raw/current/scripts/prefabs/grassgator.lua#L199-L208)

## 常见陷阱与限制

- **variance 表示上下浮动／只设 base 就够**：实际是单向增量；要整段间隔缩半，同时改 base 与 randtime，并明确是否重排当前任务。[公式与更新](../../../raw/current/scripts/components/periodicspawner.lua#L39-L48) [排程](../../../raw/current/scripts/components/periodicspawner.lua#L89-L96)
- **TrySpawn=true 就能使用返回实体**：第一返回是 boolean，可能生成 nil；应在 onspawn 内判 ent，不能直接访问 TrySpawn 第一返回的组件。[实际返回](../../../raw/current/scripts/components/periodicspawner.lua#L178-L197)
- **ForceNextSpawn 绕过所有门禁／Stop 会永久停产**：前者仍走 TrySpawn 并重排，后者没有保存单独停用标记；持续禁用需按实际宿主约定处理，而非只调一次 Stop。[别名](../../../raw/current/scripts/components/periodicspawner.lua#L239-L240) [存读](../../../raw/current/scripts/components/periodicspawner.lua#L223-L234)
- **自定义落点仍会完整核验附近密度／只设间距就能工作**：候选搜索中心在宿主，spacing 配置还要求正 density；按实际范围配置，远距离落点另核验。[密度实现](../../../raw/current/scripts/components/periodicspawner.lua#L139-L169)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 0 处——sources 全部文件（periodicspawner／glommer／SGglommer／babybeefalo／grassgator／modutil）本版均未变，无结论修订。依据：变更清单交叉核对（本页无命中改动文件）。
- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖计时重排／保留、门禁／重试、密度与间距、回调／返回、海上分派、单次离屏追赶、剩余时间存读／清理及格罗姆 SG／小牛模式。只核验 SpawnFlotsam 的调用分支，未审校其内部或所有生成宿主；grassgator 注释不当作活跃挂载。
- 结构检查：格式、来源、链接／锚点、大小及唯一登记通过；完整间隔调整示例通过已有 Full Moon Lua 5.1 syntax-only，不证明实际产量、落点或动画效果。
- 游戏运行：未运行；首轮／后续／读档、Stop 后重建、密度阻塞、离屏追赶、海上／平台、回调 nil 和多模组重复改倍率待实测。
