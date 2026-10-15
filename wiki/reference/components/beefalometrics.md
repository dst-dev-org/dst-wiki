---
title: "beefalometrics"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "原牛行为统计桥接与最近驯养参与者记录；返回值可为玩家实体、userid 或 nil，不代表所有权。"
aliases: ["读取牛最近驯养参与者userid", "牛行为统计", "beefalometrics"]
tags: [component, creature, event, data, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/beefalometrics.lua", "prefabs/beefalo.lua", "components/brushable.lua", "components/eater.lua", "stats.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# beefalometrics

## 快速判断

- 职责与适用：原牛监听喂食、刷毛、驯化、骑乘、受击等事件，将指定字段交给 Stats，并保存最近驯养参与者线索；不是驯化数值或绑定所有权的组件。[注册](../../../raw/current/scripts/components/beefalometrics.lua#L123-L137) [统计桥接](../../../raw/current/scripts/components/beefalometrics.lua#L4-L8) [牛安装](../../../raw/current/scripts/prefabs/beefalo.lua#L1263-L1264)
- 优先使用：仅需该线索时读 GetLastDomesticator，区分实体／字符串／nil；改驯化用 [[reference/components/domesticatable]]，改骑乘用 [[reference/components/rideable]]／[[reference/components/rider]]。[查询](../../../raw/current/scripts/components/beefalometrics.lua#L139-L158)
- 重要副作用：SetLastDomesticator 会清旧 userid 缓存并存实体；喂食／刷毛事件在宿主回调之前更新记录，方便随后的驯化事件归因。[设置](../../../raw/current/scripts/components/beefalometrics.lua#L139-L142) [喂食](../../../raw/current/scripts/components/beefalometrics.lua#L17-L29) [刷毛](../../../raw/current/scripts/components/beefalometrics.lua#L32-L44)
- 修改边界：多个处理器直接读 uniqueid、domesticatable、rideable、食物 edible 和事件字段；不能给任意生物挂载后用伪造事件测试完整机制。[载荷前提](../../../raw/current/scripts/components/beefalometrics.lua#L4-L29) [骑乘伤害](../../../raw/current/scripts/components/beefalometrics.lua#L65-L107)

## 关键接口

| 接口／事件 | 条件、作用与证据 |
|---|---|
| `SetLastDomesticator(player)` | 保存传入引用并清 lastdomesticator_id，无玩家类型／有效性验证；仅供真实宿主归因，不改变驯化或牛铃绑定。[设置](../../../raw/current/scripts/components/beefalometrics.lua#L139-L142) |
| `GetLastDomesticator()` | 有引用直接返回；只有 userid 时尝试 UserToPlayer，找到则缓存实体，否则返回字符串；两者都无则 nil。调用者不能无条件 `value.userid`。[返回域](../../../raw/current/scripts/components/beefalometrics.lua#L144-L158) |
| `oneat`／`brushed` | 前者用 data.feeder 更新记录，后者用 data.doer；食物／奖品等统计另外读取对应字段。实际 eater／brushable 都先发事件再调宿主回调。[喂食发送](../../../raw/current/scripts/components/eater.lua#L270-L273) [刷毛发送](../../../raw/current/scripts/components/brushable.lua#L53-L56) |
| `domesticationdelta`／`domesticated`／`goneferal` | 0→正值记录开始，完成读取 data.tendencies，野化读取宿主倾向；记录行为不驱动这些转换。[开始](../../../raw/current/scripts/components/beefalometrics.lua#L10-L15) [完成／野化](../../../raw/current/scripts/components/beefalometrics.lua#L47-L63) |
| `riderchanged` | 新骑手出现记 ridestarttime；下骑计算 GetTime 差。不是累计骑乘时长存档接口，也没有默认起始值兜底。[计时](../../../raw/current/scripts/components/beefalometrics.lua#L82-L95) |
| 存读 | 只存 lastdomesticator_id；读档只恢复该字段，在线实体延迟由 getter 解析。未保存事件历史、统计总表或骑乘开始时间。[存读](../../../raw/current/scripts/components/beefalometrics.lua#L160-L173) |

## 典型接入

### 为原牛提供只读的最近驯养参与者 userid 查询

目标模组 `modmain.lua`，无新资源。服务端调用方用 `beef:GetMyModLastDomesticatorUserId()` 取得字符串或 nil；只规范原 getter 的混合返回，不写统计、不修改驯化，也不把结果用于物品／技能授权。

```lua
AddPrefabPostInit("beefalo", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst.GetMyModLastDomesticatorUserId = function(beef)
        local metrics = beef.components.beefalometrics
        local value = metrics ~= nil and metrics:GetLastDomesticator() or nil
        if GLOBAL.type(value) == "string" then
            return value ~= "" and value or nil
        end
        if value ~= nil and value:IsValid() then
            return value.userid ~= "" and value.userid or nil
        end
        return nil
    end
end)
```

只面向原组件按正常事件维护的实体／userid，其他模组塞任意类型不在前提内；有原引用时 getter 不自动改查离线 userid，失效引用在本例返回 nil。该函数不保存自身数据，重载重装并读取组件恢复结果；停用重载移除扩展。线索可能是最后喂食／刷毛的人，不等于牛铃主人、唯一驯养人或当前骑手。

Stats 入口构建上下文、合并 values、JSON 编码并调用 TheSim:SendProfileStats；这段入口未验证事件 id 或证明平台接受，调用也不产生本页所述的游戏奖励。本例不调用统计发送接口。[统计入口](../../../raw/current/scripts/stats.lua#L298-L314)

## 常见陷阱与限制

- **getter 永远给在线玩家**：读档只有 userid、未找到玩家时返回字符串；已缓存引用也未由 getter 检查有效性。[返回](../../../raw/current/scripts/components/beefalometrics.lua#L144-L158)
- **刷毛统计的 player 参数一定是 doer**：本版本先用 data.doer 更新记录，但 PushEvent 调用实际传 data.feeder；brushable 发的是 doer／numprizes。原注释里的 onbrushed 也不等于实际监听名 brushed；按实现识别，不据注释补造字段。[处理器](../../../raw/current/scripts/components/beefalometrics.lua#L32-L44) [发送](../../../raw/current/scripts/components/brushable.lua#L53-L56)
- **主动 PushEvent 可以模拟驯化／骑乘过程**：此组件只是消费者，其他监听有自己的载荷和顺序前提；伪造缺字段事件可让处理器失败，不能代替真实 gameplay 接口。[事件载荷](../../../raw/current/scripts/components/beefalometrics.lua#L47-L95)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：eater.lua 对照 build-26148036 更新报告 C-12；oneat 载荷 `{food, feeder}` 与先发事件后调宿主回调的顺序无语义变化，仅锚点平移（eater.lua 302-305→270-273）；SetCanEat* 弃用与新增 ResetEdibleTagsCache 不触及本页结论。未覆盖：运行验证。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖完整组件、归因记录／返回域／存读、原牛安装、刷毛与进食发送顺序、Stats 上下文／编码／发送入口。未全面审校统计上下文构造与平台接受、所有事件生产路径、牛铃归属与跨分片用户解析。
- 结构检查：本批格式、来源、链接／行号、大小及登记通过；完整只读 getter 例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；在线／离线／失效引用、保存恢复、刷毛载荷差异和其他模组修改记录待实测。
