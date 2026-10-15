---
title: "探测杖距离环与 WX-78 特效（dr_warm_loop_1 等）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "探测杖按距离选用四种着色环，以及 WX-78 音乐电路音符、旧蒸汽特效的注册与生成边界。"
aliases: ["dr_warm_loop_1", "dr_warm_loop_2", "dr_warmer_loop", "dr_hot_loop", "wx78_heat_steam", "wx78_musicbox_fx", "探测杖距离环", "WX78音乐特效", "WX78蒸汽"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/diviningrod.lua", "tuning.lua", "wx78_moduledefs.lua", "prefabs/wx78_modules.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 探测杖距离环与 WX-78 特效（dr_warm_loop_1 等）

## 快速判断

- 六名是通用表动画特效。探测杖按目标距离选四色环；音乐电路生成音符；`wx78_heat_steam` 在表中标注 Deprecated，在本页检索范围未发现生成点。通用代理、本地表现及回收见 [[reference/mechanisms/generic-fx-factory#关键流程]]。
- 探测距离和音乐电路的照料、理智光环由生成者实现，不由这些动画特效承担。音符条目无 `sound`，电路循环音由 WX 自身播放。[音乐电路](../../../raw/current/scripts/wx78_moduledefs.lua#L1017-L1029)
- 中文标题与别名是按源码用途和动画名核验的效果描述，非官方译名。简中 `languages/chinese_s.po` 定向检索六名 `STRINGS.NAMES` 显示名键无命中。

## 注册与构造

均由 `fx.lua` 数据表经通用加载器注册。六条 `sound` 均未设置；前五条也无 `fn`。

| 注册名 | bank／build | anim 与其他字段 | 条目 |
|---|---|---|---|
| `dr_warm_loop_1` | `diviningrod_fx`／同左 | `warm_loop`，tint RGB (105,160,255)/255 | [条目](../../../raw/current/scripts/fx.lua#L468-L472) |
| `dr_warm_loop_2` | 同上 | `warm_loop`，tint (105,182,239)/255 | [条目](../../../raw/current/scripts/fx.lua#L475-L479) |
| `dr_warmer_loop` | 同上 | `warmer_loop`，tint (255,163,26)/255 | [条目](../../../raw/current/scripts/fx.lua#L482-L486) |
| `dr_hot_loop` | 同上 | `hot_loop`，tint (181,32,32)/255 | [条目](../../../raw/current/scripts/fx.lua#L489-L493) |
| `wx78_heat_steam` | `wx_fx`／同左 | `steam`；源码条目旁标 Deprecated | [条目](../../../raw/current/scripts/fx.lua#L2713-L2716) |
| `wx78_musicbox_fx` | `wx_fx`／同左 | 初始 `music1`，`nofaced=true`；fn 随机播放 `music1`～`music4` 并设置最终偏移 1 | [条目](../../../raw/current/scripts/fx.lua#L2719-L2727) |

## 关联机制导航

| 生成者／关系 | 触发、选名、落点及清理 | 证据 |
|---|---|---|
| 探测杖 `diviningrod` | 装备且未禁用时启动检测；按最近有效目标选择距离档，通过 `EFFECTS` 将 cold／warm／warmer／hot 映射为 `_1`／`_2`／`warmer`／`hot`。默认阈值依序 50、100、200、400；循环每次先设档再判断距离，超出最后阈值仍保留 cold 档。没有目标则不选 FX。主模拟回调生成代理，给代理加 Follower 跟随持有者 `swap_object`；没有调用 SetParent。 | [目标和距离](../../../raw/current/scripts/prefabs/diviningrod.lua#L24-L48)、[选名生成](../../../raw/current/scripts/prefabs/diviningrod.lua#L53-L87)、[距离表](../../../raw/current/scripts/tuning.lua#L2248-L2254)、[端分界](../../../raw/current/scripts/prefabs/diviningrod.lua#L175-L179) |
| 探测杖停止 | 生成下一次时旧效果由其自身结束；卸下取消任务并移除仍有效的最新代理。Follower 作用于代理；本地表现的代理取位和父挂协议见 [[reference/mechanisms/generic-fx-factory#关键流程]]，不要把 Follower 当成 SetParent。 | [生成与调度](../../../raw/current/scripts/prefabs/diviningrod.lua#L82-L93)、[装备](../../../raw/current/scripts/prefabs/diviningrod.lua#L96-L104)、[卸下](../../../raw/current/scripts/prefabs/diviningrod.lua#L107-L120) |
| WX-78 音乐模块 | 主模拟侧模块激活后，周期更新照料范围内可照料作物，在 WX 当前位置落点生成音符，无父挂；技能启用时取消该周期任务，仅在 leaderrollcall 不存在时创建组件并安装生成音符的回调，已有组件时该分支不安装回调。停用最后模块时取消周期任务，循环声音由 WX 的 SoundEmitter 结束。 | [生成和周期回调](../../../raw/current/scripts/wx78_moduledefs.lua#L937-L956)、[技能切换](../../../raw/current/scripts/wx78_moduledefs.lua#L977-L1003)、[激活](../../../raw/current/scripts/wx78_moduledefs.lua#L1009-L1029)、[停用](../../../raw/current/scripts/wx78_moduledefs.lua#L1042-L1065)、[模块端与绑定](../../../raw/current/scripts/prefabs/wx78_modules.lua#L60-L75) |
| 依赖声明 | 探测杖声明四环，音乐模块在 `extra_prefabs` 声明音符；声明不是生成调用。 | [杖声明与映射](../../../raw/current/scripts/prefabs/diviningrod.lua#L8-L22)、[模块声明](../../../raw/current/scripts/wx78_moduledefs.lua#L1073-L1085) |

## 常见陷阱与限制

- 探测杖的 warm／hot 是目标远近档位，不代表角色体温或热源。距离阈值与音调由探测杖和 tuning 数据决定，动画自身不测距。
- `wx78_musicbox_fx` 自身无音效字段；随机动画由各本地表现的 fn 执行。WX 的音乐声音和照料逻辑是单独的电路行为。
- `wx78_heat_steam` 的 Deprecated 注释与未找到生成点只能支持保守缺口，不证明该注册无法使用或已删除。

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394。逐项核对六条 bank／build／anim／sound／fn 和上列生成链。对 `raw/current/scripts` 全树检索六个完整注册名，并查 `dr_`／`wx78_` 前缀、拼名、`EFFECTS` 映射及 `music_spawn_fx` 间接调用；除上述链和声明外，未找到 `wx78_heat_steam` 生成点，检索不等于证明不存在。动画实际外观、长度以及 Follower 在各客户端的呈现未运行核验。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-character-index|角色技能、施法与读书]]。
