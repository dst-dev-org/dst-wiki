---
title: "soul"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "灵魂物品标记：构造加 soul 标签供双端判定，EAT 食魂链路按组件识别；原版唯一宿主是 wortox_soul。"
aliases: ["沃拓克斯的灵魂", "灵魂物品标记", "食魂目标", "soul"]
tags: [component, item, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/soul.lua", "prefabs/wortox_soul.lua", "components/souleater.lua", "componentactions.lua", "actions.lua", "stategraphs/SGwilson.lua", "containers.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# soul

## 快速判断

- 职责与适用：标记可被 souleater 组件宿主（食魂者）食用的灵魂物品；源码注明"for generating component actions"，构造即加 `soul` 标签（V2C 建议在 pristine 前显式加，宿主照做）。[组件](../../../raw/current/scripts/components/soul.lua#L1-L10)、[宿主标签](../../../raw/current/scripts/prefabs/wortox_soul.lua#L284-L285)
- 优先使用：食魂者侧用 souleater 组件完成进食——EAT 执行调 `souleater:EatSoul(obj)`，无 soul 组件直接拒绝；组件本体无字段无方法。[EatSoul](../../../raw/current/scripts/components/souleater.lua#L14-L18)、[EAT 分支](../../../raw/current/scripts/actions.lua#L804-L806)
- 重要副作用：`soul` 标签供双端判定使用——souljar 容器按标签收灵魂（[过滤](../../../raw/current/scripts/containers.lua#L2011-L2012)）、wortox_soul_spawn 的搜索标签表（[SOUL_TAGS](../../../raw/current/scripts/prefabs/wortox_soul.lua#L76)）等；进食前的 SG 校验也查组件（[校验](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1229-L1232)）。
- 修改边界：唯一原版宿主 wortox_soul——标签 pristine 前、组件 mastersim 后（[组件挂接](../../../raw/current/scripts/prefabs/wortox_soul.lua#L317-L318)）；动作候选两路：对自身使用（doer == target）与库存内直接食用（[USEITEM 候选](../../../raw/current/scripts/componentactions.lua#L1768-L1770)、[INVENTORY 候选](../../../raw/current/scripts/componentactions.lua#L2935-L2937)）。

## 典型接入

### 让自定义物品可被食魂者吃掉

- 前提与选择：物品加标签（双端，pristine 前）与组件（主机）；食用者带 souleater 组件即可，无需改动作。
- 生效与撤销：EatSoul 会先从 stackable 取单件再发 `oneatsoul` 事件并调 oneatsoulfn（[取件](../../../raw/current/scripts/components/souleater.lua#L17-L21)）；灵魂是否消失、回多少血都在 souleater 侧处理。

```lua
AddPrefabPostInit("my_soul", function(inst)
    inst:AddTag("soul")
    if not GLOBAL.TheWorld.ismastersim then
        return inst
    end
    inst:AddComponent("soul")
end)
```

- 调整与验证：想让灵魂入 souljar，保留 `soul` 标签并避免 `nosouljar` 标签（[过滤](../../../raw/current/scripts/containers.lua#L2011-L2012)）。

## 常见陷阱与限制

- **标签与组件缺一即断链**：只加标签不加组件——标签类消费者（souljar、搜索）认，但 EAT 执行 `EatSoul` 直接 return false（[拒绝](../../../raw/current/scripts/components/souleater.lua#L15-L16)）；只加组件不加标签——客户端 souljar 过滤等标签判定失效。
- **非食魂者不能吃**：进食校验里 obj 有 soul 组件而 inst 没有 souleater 时，发 `wonteatfood` 并中止（[校验](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1229-L1232)）；不要把灵魂物品当普通食物喂。
- **组件本身不提供灵魂数或效果**：Wortox 的灵魂资源、技能联动在 wortox 与 souleater 侧；本组件只是"这是灵魂"的身份标记。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、两处 EAT 候选、EAT 执行分支、souleater 进食门槛、SGwilson 进食校验、souljar 过滤与 wortox_soul 宿主。未覆盖：Wortox 技能与灵魂资源的完整链路、其他 soul 标签消费者逐处审阅、运行验证。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。

## 相关链接

- [[reference/components/souleater]]：食魂者：EatSoul 消费与事件结算。
