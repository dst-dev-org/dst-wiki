---
title: "beard"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "按世界天数触发毛发阶段回调、分级剃毛与恢复，保温和胡须库存依赖实际人物宿主。"
aliases: ["威尔逊胡须保温翻倍", "胡须分阶段生长", "牛毛长出", "beard"]
tags: [component, creature, lifecycle, callback, event]
dst_build: "steam-buildid:26230496"
sources: ["components/beard.lua", "prefabs/wilson.lua", "components/temperature.lua", "prefabs/beefalo.lua", "stategraphs/SGBeefalo.lua", "actions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# beard

## 快速判断

- 职责与适用：监听世界 cycles 累计 daysgrowth，按精确天数调用宿主阶段回调；bits／动画由回调负责，组件不自带通用毛发资源。无生长模型的剃取目标用 [[reference/components/shaveable]]。[每日更新](../../../raw/current/scripts/components/beard.lua#L1-L35) [威尔逊组装](../../../raw/current/scripts/prefabs/wilson.lua#L152-L158)
- 优先使用：AddCallback(day,fn) 登记阶段，EnableGrowth 控制天数监听，Reset 清生长和 bits；保温只改 insulation_factor，不直接伪造 bits。[接口](../../../raw/current/scripts/components/beard.lua#L69-L92) [登记／重置](../../../raw/current/scripts/components/beard.lua#L168-L180)
- 重要副作用：Shave 回退到前一阶段，产物数量为 oldbits-newbits；自剃可恢复精神，再更新胡须库存、发 shaved。不是每次剃到零。[剃毛流程](../../../raw/current/scripts/components/beard.lua#L107-L165)
- 修改边界：人物温度消费者会把胡须值加到冬季保温、从夏季保温扣除；GetInsulation 无空值保护地读 skilltreeupdater，不能认为任意生物只加 beard 就能安全调用全部接口。[保温](../../../raw/current/scripts/components/beard.lua#L81-L92) [消费](../../../raw/current/scripts/components/temperature.lua#L274-L278)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `AddCallback(day,fn)` | callbacks[day]=fn，覆盖同天已有回调；fn(inst,skinname)。按需包装原回调，不抹掉 bits／外观联动。[调用](../../../raw/current/scripts/components/beard.lua#L1-L5) [登记](../../../raw/current/scripts/components/beard.lua#L168-L170) |
| `EnableGrowth(enable)`／`pause` | 前者增删 cycles 监听；pause 为真仅阻止每日增长体，不注销监听。技能还可能增加额外天数并逐天触发回调；不把 daysgrowth 当实际存活天数。[监听](../../../raw/current/scripts/components/beard.lua#L69-L79) [增长](../../../raw/current/scripts/components/beard.lua#L7-L35) |
| `ShouldTryToShave(doer,item)`／`Shave(doer,item)` | bits==0 拒绝；canshavetest(inst,doer) 不传 item。Shave 重做测试后反向查回调、跳过当前阶段、执行前一阶段；到零以下 Reset。[测试](../../../raw/current/scripts/components/beard.lua#L95-L115) [回退](../../../raw/current/scripts/components/beard.lua#L118-L142) |
| `bits`／`prize` | 产物基于阶段前后 bits 差，生成后在目标上方抛出，不直接放入执行者库存。剃毛者等于宿主才给 SANITY_SMALL，且需要 sanity。[产物与精神](../../../raw/current/scripts/components/beard.lua#L144-L159) |
| `Reset()` | 清 daysgrowth、累加器、bits，调用 onreset 并更新库存；不清 skinname，也不等同走 Shave／发 shaved／给奖励。[重置](../../../raw/current/scripts/components/beard.lua#L172-L180) |
| `SetSkin(skinname)`／存读 | SetSkin 与 OnLoad 都从 0 到当前天数重放已登记回调；回调应能安全重放。保存 growth、growthaccumulator、bits、skinname，LoadPostPass 再更新库存。[恢复](../../../raw/current/scripts/components/beard.lua#L182-L227) |
| `UpdateBeardInventory()` | 结合威尔逊技能与 bits 在 BEARD 槽生成／替换／移除胡须袋，可能转移或掉落库存。不是任意宿主通用容器接口。[完整分支](../../../raw/current/scripts/components/beard.lua#L253-L290) |

构造加 bearded 标签并监听复活重置；移除组件停增长、去标签和监听。威尔逊 4／8／16 天的回调分别设置 bits 与皮肤符号；牛则把长毛交给 SG，阶段回调可能只设置 hairGrowthPending，而非立即长出。[生命周期](../../../raw/current/scripts/components/beard.lua#L38-L77) [人物阶段](../../../raw/current/scripts/prefabs/wilson.lua#L94-L130) [牛延后](../../../raw/current/scripts/prefabs/beefalo.lua#L467-L473) [SG 实现](../../../raw/current/scripts/stategraphs/SGBeefalo.lua#L821-L838)

## 典型接入

### 威尔逊胡须的基础保温贡献翻倍

目标模组 `modmain.lua`，无新资源；仅原威尔逊已有 beard，保留阶段、技能倍率、皮肤、剃毛和库存协议。

```lua
AddPrefabPostInit("wilson", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local beard = inst.components.beard
    if beard ~= nil then
        beard.insulation_factor = 2
    end
end)
```

这是胡须那一项的倍率，非总人物保温翻倍；无毛 bits=0 时不凭空增加贡献。温度下次读取时生效，也会加大夏季保温扣除，不称为无条件抗冷又抗热。字段不保存，构造时重设；停用并重载恢复默认 1，不改存档毛量／阶段。温度换热细节见 [[reference/components/temperature]]。[计算](../../../raw/current/scripts/components/beard.lua#L81-L92) [温度消费者](../../../raw/current/scripts/components/temperature.lua#L274-L278)

## 常见陷阱与限制

- **直接增加 daysgrowth 就会补发阶段回调**：普通赋值没有重放；每日增长只按经过的天数触发，读档／SetSkin 才有各自重放路径。不要跳过宿主 bits、外观和库存更新。[每日](../../../raw/current/scripts/components/beard.lua#L1-L35) [恢复](../../../raw/current/scripts/components/beard.lua#L192-L227)
- **AddCallback 同一天可以并排多个**：底层一个键对应一个函数；替换前保留旧实现，否则毛量或动画丢失。[登记](../../../raw/current/scripts/components/beard.lua#L168-L170)
- **Reset 与剃毛等价**：Reset 不给产物／自剃精神，也不发 shaved；牛 onreset 还会取消刷毛、改服从和进入剃毛状态，不能把它当纯数值清空。[重置](../../../raw/current/scripts/components/beard.lua#L172-L180) [牛宿主](../../../raw/current/scripts/prefabs/beefalo.lua#L431-L442)
- **bearded 标签表示当前毛量大于零**：本组件构造即添加，Shave 用 bits 拒绝无毛；与 shaveable 的数量联动标签不同。[构造](../../../raw/current/scripts/components/beard.lua#L42-L60) [拒绝](../../../raw/current/scripts/components/beard.lua#L107-L115)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；漂移文件 prefabs/wilson.lua、stategraphs/SGBeefalo.lua、prefabs/beefalo.lua、actions.lua 对照 R-01/R-04（actions.lua／componentactions.lua）hunk 地图逐条核验：wilson 仅改火把投掷条件（GetPointSpecialActions，与胡须无关）、SGBeefalo 仅复活音效、beefalo 内容无差异、actions.lua 的拾取放宽与禁长按不触及剃毛域；锚点重定位 2 处、结论无修订；未覆盖：运行验证。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖完整组件、精确天数回调／技能累加、分级剃毛、重置与回放、胡须袋分支，威尔逊组装、温度消费、牛宿主／长毛 SG 片段。未全面审校技能配置、胡须袋实体、其他角色与预测。
- 结构检查：本批格式、来源、链接／行号、大小及登记通过；完整保温倍率例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；冬夏贡献、技能叠加、各阶段剃毛、存读／换肤回放、复活与胡须库存迁移待实测。
