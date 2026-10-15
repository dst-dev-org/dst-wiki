---
title: "aoecharging"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "双端按住蓄力的 tick、指示器和松手事件；人物状态机决定何时执行攻击。"
aliases: ["aoecharging","按住蓄力","蓄力tick","松手攻击"]
tags: ["component","network","state-machine","callback"]
dst_build: "steam-buildid:26230496"
sources: ["components/aoecharging.lua","prefabs/slingshot.lua","components/playercontroller.lua","stategraphs/SGwilson.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# aoecharging

## 快速判断

- 职责与适用：装备持有者的持续蓄力与方向／指示器；松手通知 SG，攻击效果经 ReleaseChargedAttack 调工厂，不是地点施法或普通冷却。[components/aoecharging.lua:124](../../../raw/current/scripts/components/aoecharging.lua#L124)、[components/aoecharging.lua:226](../../../raw/current/scripts/components/aoecharging.lua#L226
- 优先使用：保留原版 SG 的启动／退出与攻击时点；共同工厂配 reticuleprefab／刷新函数，服务端配 SetOnChargedAttackFn。[prefabs/slingshot.lua:645](../../../raw/current/scripts/prefabs/slingshot.lua#L645)、[prefabs/slingshot.lua:714](../../../raw/current/scripts/prefabs/slingshot.lua#L714
- 重要副作用：tick 每次 OnUpdate 加一，不按 dt 累加秒；计数是 net_byte，不能无限累积。组件松手只发 chargingreticulereleased，不直接攻击。[components/aoecharging.lua:48](../../../raw/current/scripts/components/aoecharging.lua#L48)、[components/aoecharging.lua:124](../../../raw/current/scripts/components/aoecharging.lua#L124
- 修改边界：双端、SetPristine 前构造网络变量；SetEnabled／SetChargeTicks／攻击函数仅服务端有效。新物品还需预测／服务端 SG，不能只挂此组件。[components/aoecharging.lua:199](../../../raw/current/scripts/components/aoecharging.lua#L199)、[stategraphs/SGwilson.lua:18745](../../../raw/current/scripts/stategraphs/SGwilson.lua#L18745

## 关键接口

| 接口／字段 | 参数／返回 | 条件、副作用与证据 |
|---|---|---|
| `SetEnabled(bool)`／`IsEnabled()` | 服务端设置，默认 true | 关闭清 charging owner 并向其 SG 发 chargingreticulecancelled。[components/aoecharging.lua:1](../../../raw/current/scripts/components/aoecharging.lua#L1)、[components/aoecharging.lua:199](../../../raw/current/scripts/components/aoecharging.lua#L199 |
| `SetAllowRiding(val)` | 默认 true，false 禁止 | controller 检查手持物品是否可发动；不是 ReleaseChargedAttack 自带授权。[components/aoecharging.lua:70](../../../raw/current/scripts/components/aoecharging.lua#L70)、[components/playercontroller.lua:1593](../../../raw/current/scripts/components/playercontroller.lua#L1593 |
| `SetChargingOwner(owner)` | 实体或 nil | 清旧指示器／更新，再重置计数；有 HUD 才生成充能 reticule 并 LinkToEntity。不要用反复同 owner 设置代替持续蓄力。[components/aoecharging.lua:92](../../../raw/current/scripts/components/aoecharging.lua#L92 |
| `GetChargeTicks()`／`SetChargeTicks(ticks)` | 网络 byte／服务端且有 owner 才写 | 原版用于开局加速；更新不使用 dt，不能把数值直接称秒。[components/aoecharging.lua:78](../../../raw/current/scripts/components/aoecharging.lua#L78)、[components/aoecharging.lua:208](../../../raw/current/scripts/components/aoecharging.lua#L208 |
| `SetRefreshChargeTicksFn(fn)` | `(item, reticule, ticks)` | 本地表现回调，不是伤害计算；原版以 ticks×FRAMES 映射视觉。[components/aoecharging.lua:82](../../../raw/current/scripts/components/aoecharging.lua#L82)、[prefabs/slingshot.lua:638](../../../raw/current/scripts/prefabs/slingshot.lua#L638 |
| `SetOnChargedAttackFn(fn)` | `(item,doer,chargeticks)` | 服务端注册；ReleaseChargedAttack 仅检查端和函数，不重验持有／enabled／距离。[components/aoecharging.lua:219](../../../raw/current/scripts/components/aoecharging.lua#L219 |
| `reticuleprefab`／`pingprefab` | 默认 nil | 蓄力实体需要 chargingreticule；当前松手代码仅用 pingprefab 判是否启用，实际生成名硬编码 reticulelongping。[components/aoecharging.lua:106](../../../raw/current/scripts/components/aoecharging.lua#L106)、[components/aoecharging.lua:163](../../../raw/current/scripts/components/aoecharging.lua#L163 |

### 松手与清理

OnUpdate 检查 GrandOwner／装备／SG 蓄力或同步状态，失效则清 owner；松手 Snap、更新方向、取 ticks、清 owner，再发 SG 事件。原版 SG 接事件切到射击状态，时间线才 ReleaseChargedAttack；退出会 SetChargingOwner(nil)。组件 OnRemoveEntity 也清 owner。[components/aoecharging.lua:124](../../../raw/current/scripts/components/aoecharging.lua#L124)、[components/aoecharging.lua:66](../../../raw/current/scripts/components/aoecharging.lua#L66)、[stategraphs/SGwilson.lua:18789](../../../raw/current/scripts/stategraphs/SGwilson.lua#L18789)、[stategraphs/SGwilson.lua:18660](../../../raw/current/scripts/stategraphs/SGwilson.lua#L18660

## 典型接入

### 蓄力弹弓禁止骑乘发动但保留蓄力伤害

目标模组 `modmain.lua`；注册名 slingshotex，复用其资源、SG、弹药与技能树检查；双端改普通 allowriding，不替换攻击函数或强制 SetEnabled(true)。[prefabs/slingshot.lua:650](../../../raw/current/scripts/prefabs/slingshot.lua#L650)、[prefabs/slingshot.lua:674](../../../raw/current/scripts/prefabs/slingshot.lua#L674)、[prefabs/slingshot.lua:912](../../../raw/current/scripts/prefabs/slingshot.lua#L912

```lua
AddPrefabPostInit("slingshotex", function(inst)
    local charging = inst.components.aoecharging
    if charging ~= nil then
        charging:SetAllowRiding(false)
    end
end)
```

本例影响 controller 的蓄力发动资格，不提供防作弊完整授权。原版攻击将 ticks 减五帧、归一化并平方后供弹体使用；不要再在此处重复乘伤害。停用后新建／载入物品恢复默认，无新增任务或持久字段。[prefabs/slingshot.lua:656](../../../raw/current/scripts/prefabs/slingshot.lua#L656

## 常见陷阱与限制

- **松手事件就是施法**：组件没有自动调用攻击回调，SG 决定时点；直接自调 ReleaseChargedAttack 则绕过 SG 的完整检查。[components/aoecharging.lua:226](../../../raw/current/scripts/components/aoecharging.lua#L226)、[stategraphs/SGwilson.lua:18660](../../../raw/current/scripts/stategraphs/SGwilson.lua#L18660
- **把 ticks 设置为秒或无限蓄力**：它是按更新次数增长的 byte；最大值行为和超时不能凭 Lua 字段推断，原版 SG 有 90 帧超时处理。[components/aoecharging.lua:51](../../../raw/current/scripts/components/aoecharging.lua#L51)、[stategraphs/SGwilson.lua:18771](../../../raw/current/scripts/stategraphs/SGwilson.lua#L18771
- **改 pingprefab 就替换松手 FX**：当前实现硬编码 reticulelongping；本页记录实际行为，不将字段名当完整可配置承诺。[components/aoecharging.lua:163](../../../raw/current/scripts/components/aoecharging.lua#L163
- **移组件必走实体删除钩子**：本文件定义 OnRemoveEntity，而未定义 OnRemoveFromEntity；主动拆组件前自行 SetChargingOwner(nil)。[components/aoecharging.lua:66](../../../raw/current/scripts/components/aoecharging.lua#L66

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 4 处（SGwilson 蓄力检查／reticule 释放事件／退出清理／90 帧超时均 +120，显示标签同步）；无结论修订，仅锚点迁移——quickbufferedaction 第 14 帧新增的副攻击键连发快进（player-hubs 域 7）不涉本页蓄力→chargingreticulereleased→射击链，aoecharging.lua、slingshot.lua 未变，playercontroller 本版两处单行改动（PullUpMap/GetPickupAction）不在 1593 引用点。依据：player-hubs-hunk-map 41 断点偏移表＋变更清单命中 playercontroller.lua、stategraphs/SGwilson.lua。
- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖完整 aoecharging、slingshotex 工厂与攻击映射、controller 发动资格及 SG 服务端蓄力／松手／退出。客户端 SG 全部预测转换未审校。
- 结构检查：发布时检查格式、链接／锚点及唯一登记，完整示例作 Lua 5.1 syntax-only 检查。
- 游戏运行：未运行；网络延迟、byte 边界、手柄、骑乘切换、异常退出和多模组待测。

相关：[[reference/components/aoetargeting]]；[[reference/components/projectile]]；[[reference/components/weapon]]。
