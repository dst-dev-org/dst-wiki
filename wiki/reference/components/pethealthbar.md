---
title: "pethealthbar"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "玩家侧宠物血条的双端同步桥接，绑定一个宠物并消费生命／附加条事件；不修改实际生命。"
aliases: ["刷新阿比盖尔血条上限", "宠物血条同步", "pethealthbar"]
tags: [component, ui, network-sync, event, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/pethealthbar.lua", "prefabs/wendy.lua", "prefabs/willow.lua", "prefabs/abigail.lua", "widgets/statusdisplays.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# pethealthbar

## 快速判断

- 职责与适用：装在显示血条的玩家，双端创建 netvar；服务端绑定一个宠物、监听生命等事件，客户端转发 clientpet*dirty 给 HUD。原温蒂在共同初始化安装，薇洛仅 lavaarena 分支安装。[双端构造](../../../raw/current/scripts/components/pethealthbar.lua#L82-L163) [温蒂](../../../raw/current/scripts/prefabs/wendy.lua#L152-L161) [薇洛](../../../raw/current/scripts/prefabs/willow.lua#L143-L148)
- 优先使用：SetPet(pet,symbol,maxhealth) 绑定有效宠物；既有绑定的显示上限用 SetMaxHealth。实际改生命用 [[reference/components/health]]，宠物登记用 [[reference/components/petleash]]，不互相代替。[绑定](../../../raw/current/scripts/components/pethealthbar.lua#L257-L309) [显示 setter](../../../raw/current/scripts/components/pethealthbar.lua#L234-L239)
- 重要副作用：换宠物会注销旧宠物与 Buff 监听、清集合，随后排零延迟初始化并启动更新；同一宠物再次 SetPet 只先更新 symbol／maxhealth 后返回。[重绑](../../../raw/current/scripts/components/pethealthbar.lua#L257-L309)
- 修改边界：不是通用多宠物 HUD；原 statusdisplays 按玩家条件建特定 badge，并读取本组件。只在服务端临时 AddComponent 无法补齐双端 netvar 和 UI 接入。[HUD 条件](../../../raw/current/scripts/widgets/statusdisplays.lua#L408-L415) [读取](../../../raw/current/scripts/widgets/statusdisplays.lua#L975-L978)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `SetPet(pet,symbol,maxhealth)` | 服务端用；pet 必须非 nil，随后直接读 pet.skinname／监听事件。换目标重设皮肤与百分比，**不能用 SetPet(nil) 解绑**；没有通用解绑方法。[实现](../../../raw/current/scripts/components/pethealthbar.lua#L257-L309) |
| `SetMaxHealth(max_health)` | 只写 _maxhealth 的 net_ushortint，发 clientpetmaxhealthdirty；不写宠物 health，不自动跟踪其 maxhealth 变化。原阿比盖尔调实际生命上限后另更新玩家显示。[方法](../../../raw/current/scripts/components/pethealthbar.lua#L234-L239) [原桥接](../../../raw/current/scripts/prefabs/abigail.lua#L59-L74) |
| `SetSymbol`／`SetSymbol2`／`SetPetSkin` | 前两者更新哈希与符号事件，皮肤 nil 映射 0；不是添加图标／皮肤资源接口。原阿比盖尔用两条符号表示已有药剂 Buff。[方法](../../../raw/current/scripts/components/pethealthbar.lua#L220-L250) [绑定初始化](../../../raw/current/scripts/prefabs/abigail.lua#L415-L425) |
| `GetPercent`／`GetMaxHealth` | 返回同步显示值；绑定后零延迟 InitPet 从宠物 health:GetPercent 取初值，随后听 healthdelta {newpercent,oldpercent,overtime}。不是服务端实时生命查询。[初始化](../../../raw/current/scripts/components/pethealthbar.lua#L252-L255) [事件消费](../../../raw/current/scripts/components/pethealthbar.lua#L37-L55) |
| `pethealthbar_bonuschange` | 事件发在宠物，支持 max、newpercent、oldpercent、overtime；_maxbonus>0 才展示附加条。只消费数据，不给予实际额外生命。[接收](../../../raw/current/scripts/components/pethealthbar.lua#L57-L76) [原发送](../../../raw/current/scripts/prefabs/abigail.lua#L615-L631) |
| `GetOverTime()`／`GetPulse()`／`ResetPulse()` | 变化箭头由冻热／饥饿／火伤／腐蚀和恢复来源推断，不是净回血率；GetOverTime 映射状态为 -2～2，实际 OnUpdate 产生 0／1／2／3 状态。ResetPulse 仅 set_local(0)，供本地 HUD 消费。[读取](../../../raw/current/scripts/components/pethealthbar.lua#L188-L215) [状态](../../../raw/current/scripts/components/pethealthbar.lua#L312-L338) |

组件没有存读和通用宠物删除清理钩子，绑定／重建归宿主；不能保留失效引用还继续 OnUpdate，也不能把热更新移除组件当完整解绑。原阿比盖尔 linktoplayer 设置跟随并绑定玩家血条，皮肤／Buff 图标也在该宿主链初始化。[宿主](../../../raw/current/scripts/prefabs/abigail.lua#L405-L425) [更新前提](../../../raw/current/scripts/components/pethealthbar.lua#L312-L326)

## 典型接入

### 模组调整生命后请求刷新温蒂当前宠物的显示上限

目标模组 `modmain.lua`，无新资源；使用原温蒂已安装的双端组件。服务端调用方修改当前绑定宠物的正常有效 health.maxhealth 后，对温蒂发 `player:PushEvent("my_mod_refresh_pet_health_max")`；这里只提供接收端，不含 RPC、授权或实际改生命代码。

```lua
AddPrefabPostInit("wendy", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst:ListenForEvent("my_mod_refresh_pet_health_max", function(player)
        local bar = player.components.pethealthbar
        local pet = bar ~= nil and bar.pet or nil
        if pet ~= nil and pet:IsValid() and pet.components.health ~= nil then
            bar:SetMaxHealth(pet.components.health.maxhealth)
        end
    end)
end)
```

仅更新当前目标的显示上限，不触发回血、不改变绑定／百分比／附加条；正常 healthdelta 仍由原监听消费。自定义巨大值、负值或小数应另核验 net_ushortint 的表示边界，不靠这段代码扩展网络类型。组件未安装、尚未绑定或宠物无效时不处理；重载仍由原宿主重绑，不保存自定义事件。停用重载移除接收端，后续阿比盖尔原逻辑仍可覆盖显示值。[原上限桥接](../../../raw/current/scripts/prefabs/abigail.lua#L59-L74)

## 常见陷阱与限制

- **血条上限提高就是宠物加血**：本组件只是同步显示；health.maxhealth 的实际修改及当前血量保留由生命机制处理。[显示 setter](../../../raw/current/scripts/components/pethealthbar.lua#L234-L239)
- **同一目标重绑会完整重置状态**：SetPet 相同目标提前返回，未重新初始化百分比、Buff 集合或皮肤；按实际数据需求使用对应入口。[提前返回](../../../raw/current/scripts/components/pethealthbar.lua#L257-L267)
- **传 nil 会清空旧绑定**：代码继续访问 pet.skinname；先核验宿主退出／隐藏协议，不能用不存在的 API 猜测解绑。[目标访问](../../../raw/current/scripts/components/pethealthbar.lua#L295-L308)
- **箭头就是实际持续生命变化**：它依据有限状态／来源提示，down 分支优先；不汇总所有回血、伤害或预计净值。[状态计算](../../../raw/current/scripts/components/pethealthbar.lua#L312-L337)

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖完整组件、双端字段／事件、重绑与缺少解绑、原温蒂／薇洛安装、阿比盖尔绑定／上限／附加条与 HUD 消费片段。未全面审校 ghostlybond 生命周期、全部 HUD 布局、netvar 引擎边界与其他游戏模式。
- 结构检查：本批格式、来源、链接／行号、大小及登记通过；完整显示刷新事件例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；主机／远端显示、绑定转换、附加条／Buff 箭头、宠物消散重建及其他模组改上限待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。