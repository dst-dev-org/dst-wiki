---
title: "singingshelltrigger"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "玩家身上的海螺接近触发器：逐帧扫描范围内的 singingshell，新进入即调其 _activatefn，用三态表防重复触发。"
aliases: ["海螺触发器", "接近触发", "singingshelltrigger"]
tags: [component, callback]
dst_build: "steam-buildid:26230496"
sources: ["components/singingshelltrigger.lua", "components/singingshellmanager.lua", "prefabs/player_common.lua", "prefabs/singingshell.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# singingshelltrigger

## 快速判断

- 职责与适用：装在玩家身上，每帧找 `trigger_range` 内带 `singingshell` tag 且不在 INLIMBO 的实体，首次进入范围调 `v._activatefn(v, self.inst)`；原版由 player_common 安装（[安装](../../../raw/current/scripts/prefabs/player_common.lua#L2845-L2846)）。
- 优先使用：海螺 prefab 定义 `_activatefn` 即可被触发（[定义](../../../raw/current/scripts/prefabs/singingshell.lua#L174-L175)）；启停由世界的 singingshellmanager 统一控制（[控制](../../../raw/current/scripts/components/singingshellmanager.lua#L29)）。
- 重要副作用：监听玩家 `death`／`respawnfromghost` 自动停更/续更（[监听](../../../raw/current/scripts/components/singingshelltrigger.lua#L33-L34)）；世界无 singingshellmanager 时不自启。
- 修改边界：`overlapping` 表是公开状态，海螺侧可直接写入预登记（见常见限制）；移除组件时摘 tag 与监听。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `trigger_range` | `number`，默认 4；player_common 覆盖为 `TUNING.SINGINGSHELL_TRIGGER_RANGE`（4，tuning.lua:5521） | 每帧 FindEntities 半径。[字段](../../../raw/current/scripts/components/singingshelltrigger.lua#L20) [覆盖](../../../raw/current/scripts/prefabs/player_common.lua#L2846) |
| `findentitiesfn` | `fn(inst, range) -> entity 列表`，可替换 | 默认按 `singingshell` must、`INLIMBO` cant 过滤。[默认实现](../../../raw/current/scripts/components/singingshelltrigger.lua#L2-L7) [字段](../../../raw/current/scripts/components/singingshelltrigger.lua#L21) |
| `StartUpdating()`／`StopUpdating()` | 幂等 | 切换组件更新并维护 `updating` 标记。[实现](../../../raw/current/scripts/components/singingshelltrigger.lua#L44-L56) |
| `OnUpdate()` | 组件更新循环 | 先把上一帧 true 降为 false，再对搜到的实体置 true（新实体调 `_activatefn`，已知实体跳过），仍未出现的 false 清 nil。[实现](../../../raw/current/scripts/components/singingshelltrigger.lua#L58-L75) |
| `overlapping` | `table<entity, true|false|nil>`（公开） | 三态：true=本帧在场，false=刚在场待确认，nil=离开；海螺的 PreventImmediateActivate 直接写 true 防立即触发（[写入](../../../raw/current/scripts/prefabs/singingshell.lua#L118-L126)）。 |

## 典型接入

### 新乐器／装饰物复用接近触发

- 前提与选择：目标实体需加 `singingshell` tag 并定义 `_activatefn(inst, trigger)`；宿主玩家装组件后自动生效，无需额外接线。
- 生效与撤销：进入范围首帧触发，持续在场不重复；走出范围后再次进入会再次触发；玩家死亡暂停、复活恢复。
- 原版依据：singingshell 的 PlaySound 挂进 `_activatefn`（[宿主](../../../raw/current/scripts/prefabs/singingshell.lua#L174-L175)）。

```lua
-- 可发声实体 master_postinit 内
inst:AddTag("singingshell")
inst._activatefn = function(shell, trigger)
    shell.SoundEmitter:PlaySound("my_mod/sound/chime")
end
```

- 调整与验证：触发频率等于帧率下的 FindEntities，改大 `trigger_range` 会线性增加扫描量；`findentitiesfn` 可整体替换为自定义搜索。

## 常见陷阱与限制

- `_activatefn` 是约定字段而非组件注册接口：目标实体没定义时 OnUpdate 里 `v._activatefn(v, self.inst)` 直接报错（[调用点](../../../raw/current/scripts/components/singingshelltrigger.lua#L70)），接入方必须保证带 tag 的实体都有该字段。
- 海螺出生时用 `DoTaskInTime(0)` 扫周围玩家并预写 `overlapping[inst]=true`，防止"落在玩家身边立刻发声"（[PreventImmediateActivate](../../../raw/current/scripts/prefabs/singingshell.lua#L118-L126)）；复用时注意这个隐式协议是双向耦合。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496，覆盖全部接口与 player_common／singingshellmanager／singingshell 消费链、tuning 取值；未运行游戏。
- 结构检查：validate_page.py、validate_source_links.py 与 Lua 语法探针通过。
