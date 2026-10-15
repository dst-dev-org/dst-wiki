---
title: "消息瓶与歌贝特效（messagebottle_break_fx 等）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "消息瓶破碎和浮动、歌贝生物木质与岩质落点的四个通用表动画特效。"
aliases: ["messagebottle_break_fx", "messagebottle_bob_fx", "singingshell_creature_rockfx", "singingshell_creature_woodfx", "消息瓶破碎", "歌贝落点特效"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/messagebottle.lua", "prefabs/singingshell.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 消息瓶与歌贝特效（messagebottle_break_fx 等）

## 快速判断

- 四名是通用表动画表现；消息瓶落在陆地或平台时用破碎效果，歌贝生物根据平台材质选两种落点效果。浮动注册名在本次源码检索未找到生成点。通用代理、端与回收见 [[reference/mechanisms/generic-fx-factory#关键流程]]。
- 标题与别名的中文为根据条目和调用语境核验的效果描述，非官方译名；`languages/chinese_s.po` 按四个注册名检索无显示名键。

## 注册与构造

共同入口是通用 FX 工厂，表中未列 `sound` 或 `fn` 的条目对应字段均未设置。动画资产的实际外观未运行验证。

| 注册名 | bank / build；anim | 其他字段与证据 |
|---|---|---|
| `messagebottle_break_fx` | `bottle` / `bottle`；`break` | sound `dontstarve/creatures/monkey/poopsplat`；fn `FinalOffset2` 设置绘制偏移 2。[条目](../../../raw/current/scripts/fx.lua#L2213-L2218)、[函数](../../../raw/current/scripts/fx.lua#L5-L6) |
| `messagebottle_bob_fx` | `bottle` / `bottle`；`bob` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L2221-L2224) |
| `singingshell_creature_rockfx` | 同名 bank / build；`idle` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L2227-L2230) |
| `singingshell_creature_woodfx` | 同名 bank / build；`idle` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L2233-L2236) |

## 关联机制导航

| 生成者／触发 | 落点与差异 | 证据 |
|---|---|---|
| 消息瓶投射物 `OnHit` 与黏液瓶命中回调 | 非视觉地面且无平台时，消息瓶本体播放 `bob`，黏液瓶播放 `bob_gelblob`，动画结束移除；否则在本体坐标生成破碎 FX 并移除瓶子。黏液瓶地面分支另释放黏液。命中回调由主模拟的 complexprojectile 使用。 | [消息瓶](../../../raw/current/scripts/prefabs/messagebottle.lua#L285-L295)、[回调安装](../../../raw/current/scripts/prefabs/messagebottle.lua#L330-L335)、[黏液瓶](../../../raw/current/scripts/prefabs/messagebottle.lua#L386-L399) |
| 歌贝生物 `critterfn` 的 animover 处理 | `jump_pre` 结束后进入 `jump_lag`，随机 0～7/30 秒后 `jump_post`，再过 3/30 秒播放挖掘音效；若当前平台有 `wood` 标签用 woodfx，否则用 rockfx。主模拟侧在生物当前位置生成，不父挂。 | [端分界与选名](../../../raw/current/scripts/prefabs/singingshell.lua#L270-L288) |

## 常见陷阱与限制

- 瓶子水中播放自身 `bob` 不等于生成 `messagebottle_bob_fx`。在 `raw/current/scripts` 对完整名、`messagebottle_bob` 和相关短名前缀递归检索，未找到该注册名的生成点；这不证明其他版本或外部模组不存在调用。
- 消息瓶打开、黏液释放与歌贝音效由生成者负责，不能从 FX 条目推断玩法。共享协议的落点和清理边界见 [[reference/mechanisms/generic-fx-factory#常见陷阱与限制]]。

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394；逐条检查表项及上述调用。递归检索 scripts 中完整注册名及 `messagebottle_`、`singingshell_creature` 短名前缀，以检查动态选名；歌贝两个名称在源码中直接选择。未核验动画素材内容与游戏表现。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-table-world-index|通用表 FX 索引]]。
