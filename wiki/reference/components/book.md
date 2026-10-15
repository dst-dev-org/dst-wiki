---
title: "book"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "技能书的阅读／翻阅效果、成功后按次消费与精神结算；保留读者分支、回调返回和原书效果。"
aliases: ["睡前故事阅读精神消耗减半", "技能书阅读效果", "book"]
tags: [component, item, callback, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/book.lua", "components/reader.lua", "prefabs/books.lua", "actions.lua", "componentactions.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# book

## 快速判断

- 职责与适用：物品侧提供 OnRead／OnPeruse，由人物 [[reference/components/reader]] 选择正式阅读或翻阅；不自动赋予任何角色读书能力。[分派](../../../raw/current/scripts/components/reader.lua#L46-L60) [动作](../../../raw/current/scripts/actions.lua#L1328-L1339)
- 优先使用：SetOnRead／SetOnPeruse 配效果，SetReadSanity／SetPeruseSanity 配有符号精神增量；原书局部数值修改优先保留既有回调。[设置](../../../raw/current/scripts/components/book.lua#L5-L24)
- 重要副作用：Interact 只在存在回调且返回真值时 ConsumeUse；回调遗漏 return 得到 nil，不扣次数也不走默认成功结算。没有回调则默认成功但不扣次数。[执行](../../../raw/current/scripts/components/book.lua#L26-L43)
- 修改边界：正式阅读成功且人物有 sanity 才生成组件配置的 FX 并结算 read_sanity×读者倍率；翻阅用 peruse_sanity，不乘该倍率。业务回调仍可能自己处理失败罚值，不把默认分支等同全部书。[结算](../../../raw/current/scripts/components/book.lua#L45-L80)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `SetOnRead(fn)`／`SetOnPeruse(fn)` | fn(book,reader) 返回 success,reason；OnRead 透传二者，OnPeruse 只返回 success。函数命名相近不表示两条效果相同。[交互](../../../raw/current/scripts/components/book.lua#L32-L52) [正式阅读](../../../raw/current/scripts/components/book.lua#L61-L80) |
| `SetReadSanity(delta)`／`SetPeruseSanity(delta)` | 单次精神增量，负值扣除、正值增加，未设置按 0；正式阅读依赖 reader:GetSanityPenaltyMultiplier()。不是每秒速率。[数值](../../../raw/current/scripts/components/book.lua#L45-L59) |
| `ConsumeUse()` | 有 finiteuses 时 Use(1)，不是所有 READ 动作自动扣次数；原书把耗尽回调设为移除实体。[消费](../../../raw/current/scripts/components/book.lua#L26-L30) [原挂载](../../../raw/current/scripts/prefabs/books.lua#L1023-L1036) |
| `SetFx(fx,fxmount)` | 骑乘 FX 未提供时回落普通 FX；正式阅读按骑乘选择、定位／旋转，骑乘时转 SixFaced。无 sanity 的读者不走这段默认 FX。[配置](../../../raw/current/scripts/components/book.lua#L21-L24) [消费](../../../raw/current/scripts/components/book.lua#L61-L80) |

原书在 pristine 前加 book 标签，服务端按 def 配组件、回调、数值及有限次数；组件构造本身不补客户端标签。地面候选检查 reader 标签和 fire／smolder，库存候选只检查 reader；直接调用不继承这些候选门禁。[工厂](../../../raw/current/scripts/prefabs/books.lua#L1008-L1036) [地面](../../../raw/current/scripts/componentactions.lua#L228-L232) [库存](../../../raw/current/scripts/componentactions.lua#L2582-L2586)

## 典型接入

### 原睡前故事正式阅读的基础精神消耗减半

目标模组 `modmain.lua`，不新增资源；仅改 book_sleep 的负 read_sanity，保留原催眠／翻阅回调、FX、使用次数与耗尽移除。原 def 中正式阅读为负值、翻阅为正值，两者不能一起机械乘半。[原配置与效果](../../../raw/current/scripts/prefabs/books.lua#L383-L438)

```lua
AddPrefabPostInit("book_sleep", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local book = inst.components.book
    if book ~= nil and book.read_sanity ~= nil and book.read_sanity < 0 then
        book:SetReadSanity(book.read_sanity * 0.5)
    end
end)
```

0.5 是该物品的基础增量比例，最终仍乘正式读者倍率；翻阅不变，不改变谁能阅读，也不绕过无目标时的失败。构造时应用，原组件没有本字段的存读，重载由工厂和本 Hook 重建；停用并重载回原配置。其他模组再次设置该字段会按实际 Hook 顺序覆盖，不声称独立倍率叠加。[原失败／成功](../../../raw/current/scripts/prefabs/books.lua#L389-L428) [结算](../../../raw/current/scripts/components/book.lua#L55-L80)

## 常见陷阱与限制

- **回调完成效果但没 return true**：Interact 得 nil，默认成功消费／精神／FX 不发生，而回调副作用未回滚；明确约定成功和失败。[顺序](../../../raw/current/scripts/components/book.lua#L32-L43)
- **SetReadSanity(10) 表示消耗十点**：这里是传给 DoDelta 的有符号增量，默认正值增加；负消耗与读者倍率分别设计。[结算](../../../raw/current/scripts/components/book.lua#L55-L59)
- **只加 book 就复现完整阅读**：客户端候选、reader、实际 SG、资源与物品宿主仍需接入；本例只修改现成原书，不交付新技能书工程。[动作](../../../raw/current/scripts/actions.lua#L1328-L1339) [工厂](../../../raw/current/scripts/prefabs/books.lua#L1008-L1036)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件（actions.lua=R-01、componentactions.lua=R-04）对照更新报告逐条核验，锚点重定位 3 处、结论修订 0 处——READ.fn 与书本地面／库存候选块原文未变，READ 未加 invalid_hold_action，nomagiccast 拦截不触及 READ／book（book 无 spellcaster）；未覆盖：运行验证。

- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整 book／reader、READ 与两类候选、原书通用挂载及 book_sleep 的正式／翻阅效果。未审校全部书效果、特殊失败罚值、完整 SG／预测或自定义角色注册。
- 结构检查：本批格式、来源、源码引用／行号、大小与登记通过；完整半耗例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；正式／翻阅、不同读者倍率、失败无目标、耗尽及其他模组覆盖待实测。

相关：[[reference/components/finiteuses]]；[[reference/components/sanity]]。
