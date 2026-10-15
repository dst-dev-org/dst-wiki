---
title: "blinkstaff"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "懒人魔杖短距瞬移的资格、延迟落地与特效／消费回调；区分动作接受、实际移动和 SG 恢复。"
aliases: ["懒人魔杖关闭瞬移音效", "橙色法杖瞬移", "blinkstaff"]
tags: [component, item, callback, actions, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/blinkstaff.lua", "prefabs/staff.lua", "prefabs/staff_common.lua", "actions.lua", "componentactions.lua", "stategraphs/SGwilson.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# blinkstaff

## 快速判断

- 职责与适用：对一个 caster 安排 0.25 秒延迟瞬移，检查地图通行／阻挡与传送限制；原宿主是 orangestaff，不是 telestaff 紫杖，也不是 [[reference/components/teleporter]] 的配对出口。[执行](../../../raw/current/scripts/components/blinkstaff.lua#L62-L97) [原注册](../../../raw/current/scripts/prefabs/staff.lua#L301-L309)
- 优先使用：正常交互沿 BLINK／quicktele；局部视觉修改用 SetFX、SetSoundFX。26148036 已移除组件内的 quicktele 状态限制，直接调用不再因此失败；正常玩家仍应走原动作／SG，直接调用须自管权限与恢复。[检查](../../../raw/current/scripts/components/blinkstaff.lua#L62-L70) [动作](../../../raw/current/scripts/actions.lua#L3574-L3595)
- 重要副作用：先安排延迟，再立即执行 onblinkfn(item,pt,caster)、向物品派发 `onblink`（`{ pt = pt, caster = caster }`），最后返回 true；原回调已扣精神和一次耐久，延迟落点后来不合法时不会因此自动退还。[时点](../../../raw/current/scripts/components/blinkstaff.lua#L89-L97) [原消费](../../../raw/current/scripts/prefabs/staff_common.lua#L479-L493) [落地复查](../../../raw/current/scripts/components/blinkstaff.lua#L51-L60)
- 修改边界：人物隐藏／无敌／阴影由 quicktele 回调及中断退出协作；无 SG 分支由组件直接恢复，未保存此前无敌值。组件无存读或移除清理钩子，不把取消任务等同完整复原；带 SG 的自定义 caster 需自行提供对应恢复流程。[组件](../../../raw/current/scripts/components/blinkstaff.lua#L36-L60) [状态协作](../../../raw/current/scripts/stategraphs/SGwilson.lua#L17279-L17343)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `Blink(pt,caster)` | pt 需支持坐标字段和 Get；通过检查后取消本组件旧 blinktask，再以 caster 的任务调度延迟。方法内没有按动作距离 36 再限幅，不替调用方补授权／范围。[方法](../../../raw/current/scripts/components/blinkstaff.lua#L62-L97) [动作距离](../../../raw/current/scripts/actions.lua#L457) |
| `onblinkfn` | 直接字段，fn(item,pt,caster)，返回不作结果判定；原宿主负责 staffsanity／sanity 及 finiteuses，组件自身不扣次数。orangestaff 经工厂调用共享 fns.orange 安装回调；新共享函数另有 rechargeable 放电分支，不能把普通法杖的耐久消费推广到所有宿主。[挂载](../../../raw/current/scripts/prefabs/staff_common.lua#L534-L543) [工厂调用](../../../raw/current/scripts/prefabs/staff.lua#L257-L267) [回调](../../../raw/current/scripts/prefabs/staff_common.lua#L479-L493) |
| `SetFX(front,back)` | 配出发与落地特效；SpawnEffect 在当前 caster 坐标生成，不负责资源注册或 FX 有效性校验。[方法](../../../raw/current/scripts/components/blinkstaff.lua#L11-L34) |
| `SetSoundFX(pre,post)`／`ResetSoundFX()` | 原默认两端均 staff_blink；nil 或空串可跳过对应声音，Reset 恢复组件默认，不是宿主／SG 的全部声音开关。[设置](../../../raw/current/scripts/components/blinkstaff.lua#L16-L24) [播放](../../../raw/current/scripts/components/blinkstaff.lua#L56-L59) [出发](../../../raw/current/scripts/components/blinkstaff.lua#L72-L75) |

落地任务先恢复显示相关状态，再取动态位置并重新检查；失败不移动，但仍生成落地 FX／声音。不能从返回 true、`onblink` 或播放 FX 推断人物已经到达；可选 `DoGemShine()` 也在落点复查前调用。正常候选还限制地表／平台、船只操控与目的地；26148036 在候选与 BLINK 执行两侧增加物品 `nomagiccast` 拦截，BLINK 也禁用长按连发。这些动作门禁不由组件直调自动补齐。SG 仍在第 8 帧执行动作。[动作](../../../raw/current/scripts/actions.lua#L3574-L3595) [长按](../../../raw/current/scripts/actions.lua#L457)[落地](../../../raw/current/scripts/components/blinkstaff.lua#L36-L60) [候选](../../../raw/current/scripts/componentactions.lua#L2059-L2067) [时点](../../../raw/current/scripts/stategraphs/SGwilson.lua#L17312-L17324)

## 典型接入

### 原懒人魔杖关闭组件的出发与落地瞬移音效

目标模组 `modmain.lua`，无新资源；保留原瞬移 FX、onblinkfn、精神／次数、装备与 SG。物品实际注册名是 orangestaff。[工厂](../../../raw/current/scripts/prefabs/staff.lua#L247-L270)

```lua
AddPrefabPostInit("orangestaff", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local blink = inst.components.blinkstaff
    if blink ~= nil then
        blink:SetSoundFX("", "")
    end
end)
```

仅跳过组件两个播放点；quicktele 自己的 attack_weapon 声音仍在，不能称为整次动作静音。原回调即时消费和延迟复查保持不变。字段不存档，构造时重建；停用并重载回组件默认，其他模组后续 setter 仍可覆盖。[播放条件](../../../raw/current/scripts/components/blinkstaff.lua#L56-L59) [出发条件](../../../raw/current/scripts/components/blinkstaff.lua#L72-L75) [SG 声音](../../../raw/current/scripts/stategraphs/SGwilson.lua#L17283-L17310)

## 常见陷阱与限制

- **返回 true／收到 onblink 就可按已到达执行后续逻辑**：二者只表明进入延迟瞬移流程，0.25 秒后还会复查落点；onblink 是物品上的开始阶段事件，没有本组件的公开到达成功回调。[安排](../../../raw/current/scripts/components/blinkstaff.lua#L89-L97) [复查](../../../raw/current/scripts/components/blinkstaff.lua#L51-L60)
- **onblinkfn 是落地后回调**：实际立即执行，原版在此扣次／精神；覆盖它会丢消费，追加行为也不能当到达证据。[回调](../../../raw/current/scripts/prefabs/staff_common.lua#L479-L493)
- **只取消 blinktask 就安全停下**：显示／阴影／无敌恢复另由延迟函数或 SG 退出承担，组件未提供统一取消清理；不能裸改任务代替宿主退出流程。[恢复](../../../raw/current/scripts/components/blinkstaff.lua#L36-L47) [中断](../../../raw/current/scripts/stategraphs/SGwilson.lua#L17335-L17343)

## 验证记录

- 版本增量复核：2026-10-25，`steam-buildid:26148036`；复用更新条目 C-05／R-01／R-04／P-112，针对事件时序缺口核对 Blink diff 与共享 onblink 回调，修正 quicktele 限制、开始事件／落点复查边界、动作门禁及源码迁移；其他原结论按相同代码块沿用。未重新运行原示例或游戏。
- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整组件、orangestaff 工厂／注册与消费、BLINK／候选及完整 quicktele。未完整审校地图重映射、DynamicPosition 内部、客户端预测、平台移动或无 SG 实体取消流程。
- 结构检查：本批格式、来源、源码引用／行号、大小和登记通过；完整声音配置例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；主机／远端、骑乘／中断、落点失效、耗尽、声音边界和其他模组覆盖待实测。
