---
title: "sharkboimanagerhelper"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "鲨鱼祭司竞技场网络助手组件：把竞技场圆心与半径经 net_float 同步到客户端，供双端做入场点判定。"
aliases: ["竞技场判定", "sharkboimanagerhelper", "鲨鱼竞技场网络同步"]
tags: [component, network]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/sharkboimanagerhelper.lua", "prefabs/forest_network.lua", "components/sharkboimanager.lua", "components/map.lua", "prefabs/sharkboi.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# sharkboimanagerhelper

## 快速判断

- 职责与适用：挂在 `TheWorld.net`（世界网络实体）上，持有 `arena_origin_x/z` 与 `arena_radius` 三个 net_float；`IsPointInArena(x, y, z)` 在双端可用——半径对齐到瓦片（`ceil(radius/TILE_SCALE)*TILE_SCALE*SQRT2`）且点在可视地面上才算场内。[判定](../../../raw/current/scripts/components/sharkboimanagerhelper.lua#L18-L28)
- 优先使用：做"限定区域内才生效"的双端逻辑时参照该模式：服务端管理器写值、双端经 `TheWorld.net.components.*` 读。原版读口是 [Map:IsPointInSharkBoiArena](../../../raw/current/scripts/components/map.lua#L1487-L1492) 与 ([sharkboi.lua:642-645](../../../raw/current/scripts/prefabs/sharkboi.lua#L642-L645)。
- 重要副作用：net_float 初值 0；`IsPointInArena` 对 `radius <= 0` 恒返回 false，即"未开竞技场"语义。[零值守卫](../../../raw/current/scripts/components/sharkboimanagerhelper.lua#L19-L21)
- 修改边界：仅 forest_network 世界（主世界）的网络实体挂载（([forest_network.lua:17-21](../../../raw/current/scripts/prefabs/forest_network.lua#L17-L21)）；洞穴网络实体无此组件，相关判定需做 nil 保护。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `arena_origin_x` / `arena_origin_z` / `arena_radius` | `net_float`（名 `sharkboimanager.arena_*`） | 服务端 sharkboimanager 写入场圆心/半径，双端可读；未开战时写 0。[写值方](../../../raw/current/scripts/components/sharkboimanager.lua#L22-L31) |
| `IsPointInArena(x, y, z)` | `-> bool` | `distsq < r*r` 且 `_map:IsVisualGroundAtPoint(x, y, z)`；r 为按整瓦片放大的半径。[实现](../../../raw/current/scripts/components/sharkboimanagerhelper.lua#L18-L28) |

## 典型接入

### 复刻"世界级竞技区域"的双端判定（原版参照）

- 前提与选择：需要世界网络实体（`TheWorld.net`）。服务端在 [sharkboimanager](../../../raw/current/scripts/components/sharkboimanager.lua#L18-L31) 中写入 arena 数据或清零；客户端无写权限。
- 生效与撤销：radius 清 0 即"竞技场关闭"，所有点判定为场外；半径先向上取整到瓦片格再乘 `SQRT2`，判定范围是圆的外接放大概念，不要求圆心对齐瓦片。[对齐规则](../../../raw/current/scripts/components/sharkboimanagerhelper.lua#L26-L27)
- 原版依据：`Map:IsPointInSharkBoiArena` 封装 nil 守卫后转发本组件；sharkboi 用它同时判定"自身在场内且玩家在场内"。[封装](../../../raw/current/scripts/components/map.lua#L1487-L1492)

```lua
-- 双端：带 nil 保护的入场判定（等价 Map:IsPointInSharkBoiArena）
local function IsPointInSharkBoiArena(x, y, z)
    local net = TheWorld.net
    if net == nil or net.components.sharkboimanagerhelper == nil then
        return false
    end
    return net.components.sharkboimanagerhelper:IsPointInArena(x, y, z)
end
```

- 调整与验证：net_float 精度有限，圆心坐标直接 set 世界坐标（原版即如此）；修改判定形状要同步改瓦片对齐逻辑，否则服务端/客户端可能因取整差异不一致。

## 常见陷阱与限制

- **在洞穴世界直接访问：** 组件只挂在 forest_network；`TheWorld.net.components.sharkboimanagerhelper` 在洞穴为 nil，判定前必须守卫（原版两处读口都做了）。[挂载差异](../../../raw/current/scripts/prefabs/forest_network.lua#L17-L21)
- **把 IsPointInArena 当纯圆判定：** 结果额外要求 `_map:IsVisualGroundAtPoint`——水面上、虚空中的点即使几何在场内也返回 false；做水面舞台类逻辑要注意这一隐含条件。[地面条件](../../../raw/current/scripts/components/sharkboimanagerhelper.lua#L27)
- **误以为组件自己管理竞技场：** 本组件是纯数据与判定助手；开战/清场时序由 sharkboimanager 组件驱动，helper 不监听任何事件。[职责分离](../../../raw/current/scripts/components/sharkboimanagerhelper.lua#L1-L31)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（31 行），核对 forest_network.lua 挂载、sharkboimanager.lua 写值、map.lua 与 sharkboi.lua 读口；grep 确认消费者仅此四处。
- 结构检查：2026-10-28 validate_page.py 通过（本页）；validate_source_links.py --strict 通过；Lua 示例经 parser_probe --syntax-only 解析通过。
- 游戏运行：未运行；竞技场判定未实测。

相关：[[reference/components/charliearenawatcher]]
