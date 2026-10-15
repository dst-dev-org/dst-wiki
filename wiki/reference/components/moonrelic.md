---
title: "moonrelic"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "月亮祭坛供品标记：让物品对 moontrader 目标生成 GIVE，接受与否由目标 moontrader 的 CanAcceptFn 决定。"
aliases: ["月坛供品", "月亮传送门供品标记", "moonrelic"]
tags: [component, item, actions]
dst_build: "steam-buildid:26230496"
sources: ["components/moonrelic.lua", "componentactions.lua", "actions.lua", "prefabs/moonrockidol.lua", "prefabs/multiplayer_portal.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# moonrelic

## 快速判断

- 职责与适用：空组件，仅 `self.inst`，无字段无方法；作用对象是献给月亮相关交易者的供品物品。USEITEM 候选只看目标是否带 `moontrader` 标签（骑乘时不生成）。[组件](../../../raw/current/scripts/components/moonrelic.lua#L1-L5)、[候选](../../../raw/current/scripts/componentactions.lua#L1510-L1514)
- 优先使用：供品 prefab 主机侧加组件；接受条件完全在目标 `moontrader:SetCanAcceptFn`——原版多人传送门要求物品带 `moonportalkey` 标签且世界不在洞穴。[接受条件](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L355-L361)
- 重要副作用：组件本体无逻辑；GIVE 执行走 `moontrader:AcceptOffering(doer, invobject)`，供奉结果由 moontrader 侧决定。[执行](../../../raw/current/scripts/actions.lua#L2330-L2331)
- 修改边界：组件加在 mastersim 判定之后（[宿主](../../../raw/current/scripts/prefabs/moonrockidol.lua#L85-L89)），仅主机持有；客户端候选经 actionreplica 组件 id 列表同步，与 [[reference/components/vasedecoration]] 同一机制（[注册](../../../raw/current/scripts/componentactions.lua#L3182-L3190)）。

## 典型接入

### 让自定义物品能被献给月亮传送门

- 前提与选择：目标为挂 moontrader 组件的多人传送门（标签在 pristine 前加、组件主机侧，[目标形态](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L324-L325)）；物品侧加组件并带 `moonportalkey` 标签，否则被原版 CanAcceptFn 拒绝（[条件](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L356-L359)）。
- 生效与撤销：手持供品对传送门出现 GIVE；`moonrock_canaccept` 返回 false 时动作仍生成但 GIVE 失败，物品不被消耗——是否收下、给什么回报都在 `moontrader:AcceptOffering` 内（[执行](../../../raw/current/scripts/actions.lua#L2330-L2331)）。

```lua
AddPrefabPostInit("my_moon_idol", function(inst)
    inst:AddTag("moonportalkey")
    if not GLOBAL.TheWorld.ismastersim then
        return inst
    end
    inst:AddComponent("moonrelic")
end)
```

- 调整与验证：原版唯一宿主是 moonrockidol（另有 `SetSinks` 落地不腐等库存行为，不在本组件内，[宿主](../../../raw/current/scripts/prefabs/moonrockidol.lua#L85-L89)）；自定义接受逻辑需模组覆盖 CanAcceptFn 或自建 moontrader 宿主。

## 常见陷阱与限制

- **组件不产生任何标签**：候选判定只查目标的 `moontrader` 标签，物品侧的 `moonportalkey` 标签也要 prefab 自行添加（[原版](../../../raw/current/scripts/prefabs/moonrockidol.lua#L76)、[检查](../../../raw/current/scripts/prefabs/multiplayer_portal.lua#L356-L357)）；缺标签时传送门拒绝。
- **"月亮祭坛"不止传送门**：月球天体祭坛的组装走 moontree／moon_altar 等其他系统，本组件只服务 moontrader 的 GIVE 供奉链路，不要当通用"月之物品"标记使用。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；覆盖组件全文、USEITEM 候选、GIVE 的 moontrader 分支、moonrockidol 宿主与传送门 CanAcceptFn／OnAcceptFn 挂接。未覆盖：moontrader 组件内部与 AcceptOffering 全流程、运行验证。
- 结构检查：本批 validate_page、validate_source_links 及示例 lua 语法探测（见批级记录）。
- 游戏运行：未运行。
