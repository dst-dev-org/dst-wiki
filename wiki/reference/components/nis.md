---
title: "nis"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "过场演出执行器：临时实体挂载后按 nis 脚本表跑协程演出，可点按跳过；经全局 PlayNIS 入口驱动，当前原版无调用者。"
aliases: ["nis", "过场演出", "即时演出系统"]
tags: [component, ui]
size_class: simple
dst_build: "steam-buildid:26230496"
sources: ["components/nis.lua", "mainfunctions.lua", "nis/maxwellintro.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# nis

## 快速判断

- 职责与适用：Non-Interactive Sequence 过场演出组件。挂在一个临时实体上，`Play(lines)` 依序执行 `init`、以协程跑 `script(nis, data, lines)`、结束自毁实体；注册多种控制键输入，`skippable` 为真时点按任意键取消。[播放](../../../raw/current/scripts/components/nis.lua#L65-L82)
- 优先使用：统一入口是全局函数 `PlayNIS(nisname, lines)`（`mainfunctions.lua:865-877`）：`require("nis/"..nisname)` 取脚本表、建实体、挂组件、接线后播放；模组勿绕过该入口手工组装。[入口](../../../raw/current/scripts/mainfunctions.lua#L865-L877)
- 重要副作用：构造即占用六种控制输入（PRIMARY/SECONDARY/ATTACK/INSPECT/ACTION/CONTROLLER_ACTION 全部转成跳过检测），`OnRemoveEntity` 才解除；演出期间这些按键不再进正常游戏输入。[输入占用](../../../raw/current/scripts/components/nis.lua#L8-L13)
- 修改边界：**当前快照内 `PlayNIS` 无任何调用者**，仅是暴露的全局接口；`nis/` 目录只有 `maxwellintro.lua`（脚本表）与 `xbnyn.lua`（编译字节码的彩蛋文件，非可读源码）。[无调用者事实](../../../raw/current/scripts/mainfunctions.lua#L865)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `SetName(name)` / `SetInit(fn)` / `SetScript(fn)` / `SetCancel(fn)` | 各设一个字段 | 分别写 `name`、`init(data)`、`script(nis, data, lines)`、`cancel(data)`；PlayNIS 从脚本表自动接线，手工调用仅在自建流程时需要。[设置器](../../../raw/current/scripts/components/nis.lua#L24-L38) |
| `Play(lines)` | `-> nil` | 置 playing、跑 init、`StartThread` 执行 script，script 返回后 `OnFinish`（自毁实体）；无 script 则立即结束。[实现](../../../raw/current/scripts/components/nis.lua#L65-L82) |
| `Cancel()` | `-> nil` | 杀协程线程、调 `cancel(data)`、自毁；即跳过路径。[实现](../../../raw/current/scripts/components/nis.lua#L46-L57) |
| `skippable` | boolean（默认 false） | 仅写不改读路径之外的消费——OnClick 在其为真时才 Cancel；置真后六键任意即跳过。[跳过判断](../../../raw/current/scripts/components/nis.lua#L59-L63) |
| `data` | table（构造即建） | init/script/cancel 间的共享载荷；脚本表内部按需读写。[共享载荷](../../../raw/current/scripts/components/nis.lua#L5) |
| `playing` | boolean | 状态标记；OnFinish/Cancel 归假。无外部读取者。[置位](../../../raw/current/scripts/components/nis.lua#L66) |

## 典型接入

### 脚本表驱动的演出（按 maxwellintro 还原，未接线）

- 前提与选择：在 `nis/<名字>.lua` 定义 `{name, init, script, cancel}` 表：init 摆好舞台状态（如禁玩家控制、摆相机），script 内用 `Sleep()` 编排，cancel 负责恢复现场；然后调 `PlayNIS("名字", lines)`。maxwellintro 即此结构的实例（`nis/maxwellintro.lua:3-27`：禁用玩家控制器、生成 Maxwell、推相机）。[脚本表实例](../../../raw/current/scripts/nis/maxwellintro.lua#L3-L27)
- 生效与撤销：演出实体随结束自毁；提前跳过走 Cancel（杀线程 + cancel 钩子 + 自毁），恢复逻辑全靠脚本表自己的 cancel。

```lua
-- nis/my_intro.lua：脚本表结构（参照 maxwellintro.lua）
return {
    name = "my_intro",
    init = function(dat) dat.wilson = GetPlayer() end,
    script = function(nis, dat, lines)
        Sleep(1)
        -- 演出编排……
    end,
    cancel = function(dat) end,
}
-- 触发：PlayNIS("my_intro", lines)  -- mainfunctions.lua 全局函数
```

- 调整与验证：script 协程内可用 `Sleep()`；组件自毁后不要引用其引用。`skippable` 默认假，需要可跳过需在 init 或外部置真。

## 常见陷阱与限制

- **输入占用面大：** 六个控制 handler 在组件存在期间持续拦截（跳过检测），不可跳过演出也会吞按键；确认这是演出期期望行为。[注册](../../../raw/current/scripts/components/nis.lua#L8-L13)
- **PlayNIS 无人调用：** 该体系在当前快照属预留能力——入口存在、脚本只有 maxwellintro 一份可读实现；新项目引用前先确认目标版本行为。`nis/xbnyn.lua` 是编译字节码（含 ROT13 彩蛋文案，未链接：二进制文件不可作锚点），无法作为源码依据。
- **协程被杀后 cancel 才清理：** 若 `cancel` 钩子缺失，跳过时只杀线程并自毁，init 改的相机/控制状态不会被恢复；脚本表必须提供 cancel。[跳过路径](../../../raw/current/scripts/components/nis.lua#L46-L57)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（84 行）与 `mainfunctions.lua:865-877` 入口、`nis/maxwellintro.lua` 脚本表；全库 grep 确认 `PlayNIS` 无调用者、`nis/` 目录仅两个文件（xbnyn.lua 为二进制字节码，未反编译）。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。
