---
title: "shadowlevel"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "影子魔法等级标记组件：给装备一个可查的影子等级（默认值或动态函数），驱动麦斯威尔的影随从强度与物品描述文本。"
aliases: ["影子等级", "shadowlevel", "暗影装备等级"]
tags: [component, item]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/shadowlevel.lua", "prefabs/hats.lua", "prefabs/waxwell.lua", "stategraphs/SGshadowwaxwell.lua", "stringutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# shadowlevel

## 快速判断

- 职责与适用：给物品挂一个整数影子等级，供麦斯威尔（Maxwell）相关系统与描述文本查询 `GetCurrentLevel`；等级可以是静态默认值，也可以是 `SetLevelFn` 注册的动态函数（按耐久等变化）。[取值链](../../../raw/current/scripts/components/shadowlevel.lua#L22-L24)
- 优先使用：制作影子魔法阵营装备（影甲、蝙蝠棒、暗夜剑类）想接入麦斯威尔的随从/决斗强度时挂本组件；原版挂载遍及 ruinshat、thurible、voidcloth 系列、onemanband、armor_skeleton、batbat、amulet 等。[ruinshat 挂载](../../../raw/current/scripts/prefabs/hats.lua#L525-L526)
- 重要副作用：构造加 `shadowlevel` 标签、卸载摘除；该标签与 `HasTag("shadowmagic")` 组合触发物品描述追加"影子等级"条目。[描述消费](../../../raw/current/scripts/stringutil.lua#L343-L344)
- 修改边界：组件不监听事件、不自动刷新；动态等级完全依赖 levelfn 的实现者自行计算。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `level` | `number`（默认 1） | 静态等级；`SetDefaultLevel` 覆写。原版各装备取 `TUNING.*_SHADOW_LEVEL` 常量。[赋值示例](../../../raw/current/scripts/prefabs/thurible.lua#L287-L289) |
| `SetLevelFn(fn)` | `fn(inst) -> number` | 注册动态等级函数；设置后 `GetCurrentLevel` 优先调用 fn。[实现](../../../raw/current/scripts/components/shadowlevel.lua#L18-L20) |
| `GetCurrentLevel()` | `-> number` | `levelfn ~= nil and levelfn(inst) or level`。[实现](../../../raw/current/scripts/components/shadowlevel.lua#L22-L24) |
| `shadowlevel`（标签） | — | pristine 态建议显式加（组件内注释 V2C）；`OnRemoveFromEntity` 摘除。[标签](../../../raw/current/scripts/components/shadowlevel.lua#L7) |

## 典型接入

### 让自定义装备参与麦斯威尔的影子强度计算（原版参照）

- 前提与选择：服务端挂组件；同屏装备的多件影子装备等级会**累加**——SGshadowwaxwell 遍历装备把 `GetCurrentLevel` 求和后再决定随从等级。[累加逻辑](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L154)
- 生效与撤销：waxwell.lua 在召唤/决斗时读装备等级（[waxwell.lua:193](../../../raw/current/scripts/prefabs/waxwell.lua#L193）；卸下装备或 RemoveComponent 后等级自然回落。
- 原版依据：thurible 用 `SetLevelFn(GetShadowLevel)` 按燃料动态取等级（([thurible.lua:287-289](../../../raw/current/scripts/prefabs/thurible.lua#L287-L289)）；静态装备（影冠等）只用 `SetDefaultLevel`。

```lua
-- 服务端：静态等级装备（等价 ruinshat 做法，pristine 态加 tag）
inst:AddComponent("shadowlevel")
inst.components.shadowlevel:SetDefaultLevel(TUNING.RUINSHAT_SHADOW_LEVEL)

-- 服务端：动态等级装备（等价 thurible 做法）
inst.components.shadowlevel:SetLevelFn(function(inst)
    return inst.components.fueled ~= nil and math.ceil(inst.components.fueled:GetPercent() * 4) or 0
end)
```

- 调整与验证：等级数值语义由消费方解释（waxwell 通告把等级截到 4 档，[waxwell.lua:254](../../../raw/current/scripts/prefabs/waxwell.lua#L254），组件本身不限定取值范围。

## 常见陷阱与限制

- **以为等级跨装备取最大值：** 决斗随从的等级是装备等级求和，不是最大值；两件 1 级装备合计 2 级。[求和](../../../raw/current/scripts/stategraphs/SGshadowwaxwell.lua#L154)
- **描述文本不出现：** 物品描述的影子等级行要求**检查者** `HasTag("shadowmagic")`（麦斯威尔的角色标签），普通角色查看不显示；组件标签 `shadowlevel` 挂在物品上，两个标签缺一不可。[判定](../../../raw/current/scripts/stringutil.lua#L343)
- **levelfn 返回非数值：** 组件直接透传返回值，不做类型校验；fn 内部需自行保证返回可比较的数值。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（26 行），核对 hats.lua 两处挂载、thurible.lua 动态等级、waxwell.lua 读取与通告、SGshadowwaxwell.lua 求和、stringutil.lua 描述分支；grep 确认全部原版挂载点。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 通过；Lua 示例经 parser_probe --syntax-only 解析通过。
- 游戏运行：未运行；召唤强度数值未实测。

相关：[[reference/components/fueled]]
