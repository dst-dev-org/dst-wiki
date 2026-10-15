---
title: "carnivaldecorranker"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "统计范围内狂欢节装饰价值并换算广场等级 rank，等级变化时回调 onrankchanged，供外观与乌鸦小孩行为读取。"
aliases: ["carnivaldecorranker", "装饰评分", "广场等级", "狂欢节装饰统计"]
tags: [component, data]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/carnivaldecorranker.lua", "components/carnivaldecor.lua", "prefabs/carnival_plaza.lua", "brains/carnival_crowkidbrain.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# carnivaldecorranker

## 快速判断

- 职责与适用：装在广场类建筑上；维护"装饰实体 → 价值"表，求和换算成 `rank`（1 起，封顶 3），用 `onrankchanged` 回调通知宿主。([carnivaldecorranker.lua:32-46](../../../raw/current/scripts/components/carnivaldecorranker.lua#L32-L46)
- 优先使用：AddComponent 后设置 `onrankchanged` 再等装饰自动登记；装饰侧由 carnivaldecor 组件按 `carnivaldecor_ranker` 标签找到 ranker 调 AddDecor／RemoveDecor。([carnival_plaza.lua:399-400](../../../raw/current/scripts/prefabs/carnival_plaza.lua#L399-L400) ([carnivaldecor.lua:4-20](../../../raw/current/scripts/components/carnivaldecor.lua#L4-L20)
- 重要副作用：构造后首帧 `DoTaskInTime(0)` 全量重扫周围装饰；统计排除落在船／平台上的装饰。[carnivaldecorranker.lua:29](../../../raw/current/scripts/components/carnivaldecorranker.lua#L29 ([carnivaldecorranker.lua:8-13](../../../raw/current/scripts/components/carnivaldecorranker.lua#L8-L13)
- 修改边界：扫描半径与档位来自 tuning（RANGE=8、每 100 点一级、MAX=3）；组件不提供重新全扫的公开接口。([tuning.lua:5943-5945](../../../raw/current/scripts/tuning.lua#L5943-L5945)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `AddDecor(decor)` / `RemoveDecor(decor)` | 装饰实体 | 增删装饰条目并重算；装饰放置／移除时由 carnivaldecor 自动调用。([carnivaldecorranker.lua:48-56](../../../raw/current/scripts/components/carnivaldecorranker.lua#L48-L56) [carnivaldecor.lua:9](../../../raw/current/scripts/components/carnivaldecor.lua#L9 |
| `UpdateDecorValue(snap)` | snap 透传回调 | 求和 totalvalue；rank=min(floor(总价值/100)+1, 3)；变化且设了回调才更新 self.rank 并调 `onrankchanged(inst, new, prev, snap)`。([carnivaldecorranker.lua:32-46](../../../raw/current/scripts/components/carnivaldecorranker.lua#L32-L46) |
| `rank` / `totalvalue` / `decor` | 数值／数值／{实体=价值} | 公开只读用途；乌鸦小孩脑读 home 的 rank 决定行为。([carnivaldecorranker.lua:21-23](../../../raw/current/scripts/components/carnivaldecorranker.lua#L21-L23) [carnival_crowkidbrain.lua:63](../../../raw/current/scripts/brains/carnival_crowkidbrain.lua#L63 |
| （依赖）`carnivaldecor:GetDecorValue()` | 数值 | 装饰价值来源；无该组件的条目在 AddDecor 里存 nil。[carnivaldecorranker.lua:49](../../../raw/current/scripts/components/carnivaldecorranker.lua#L49 |

## 典型接入

### 自定义广场建筑接入装饰等级

- 前提与选择：建筑需在统计半径内接收装饰（原版按 `carnivaldecor_ranker` 标签 FindEntities）；GetStatus 等展示直接读 `rank`。([carnivaldecor.lua:4-9](../../../raw/current/scripts/components/carnivaldecor.lua#L4-L9) ([carnival_plaza.lua:169-170](../../../raw/current/scripts/prefabs/carnival_plaza.lua#L169-L170)

```lua
inst:AddComponent("carnivaldecorranker")
inst.components.carnivaldecorranker.onrankchanged = function(inst, new_rank, prev_rank, snap)
    -- 按等级换外观／奖励，snap 为 true 表示初始化全量重扫
end
```

- 生效与撤销：装饰实体带 carnivaldecor 组件放置后自动入表；装饰被移除或建筑被拆时由对端清理。

## 常见陷阱与限制

- **不设 onrankchanged 则 rank 永远为 0**：rank 只在回调分支内赋值；只读 `rank` 做展示前必须先设回调。([carnivaldecorranker.lua:39-45](../../../raw/current/scripts/components/carnivaldecorranker.lua#L39-L45)
- 全量重扫只在构造后首帧；之后完全依赖 AddDecor／RemoveDecor 事件，手工放置无 carnivaldecor 组件的装饰不会被统计。[carnivaldecorranker.lua:29](../../../raw/current/scripts/components/carnivaldecorranker.lua#L29
- 装饰按标签+组件值统计，且 `GetCurrentPlatform() == nil` 才计入：船上的装饰不影响广场等级。([carnivaldecorranker.lua:8-13](../../../raw/current/scripts/components/carnivaldecorranker.lua#L8-L13)

## 验证记录

- 源码审阅：2026-10-28，`steam-buildid:26230496`：审阅组件全文、carnivaldecor 双向登记、广场安装与 onrankchanged、乌鸦小孩脑读取及三处 tuning 值。
- 游戏运行：未运行；未复现装饰放置到升级的完整流程。
- 结构检查：validate_page.py 通过；validate_source_links.py --strict 通过（本批 16 页合并执行）。

相关：[[reference/components/carnivaldecor]]
