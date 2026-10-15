---
title: "暗影融合体、恐惧石与诅咒特效（dreadstone_spawn_fx 族）"
created: "2026-11-01"
updated: "2026-11-01"
type: prefab
summary: "融合暗影生物出现、恐惧石堆变化、寄生附体和牛铃诅咒的四个通用表表现。"
aliases: ["fused_shadeling_spawn_fx", "dreadstone_spawn_fx", "shadowthrall_parasite_attach_poof_fx", "beef_bell_shadow_cursefx", "融合暗影出现", "恐惧石特效", "寄生附体特效", "牛铃暗影诅咒"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/fx.lua", "stategraphs/SGfused_shadeling.lua", "prefabs/nightmarefissure.lua", "prefabs/shadowthrall_parasite.lua", "prefabs/shadow_beef_bell_curse.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 暗影融合体、恐惧石与诅咒特效（dreadstone_spawn_fx 族）

## 快速判断

- 四注册分别由融合暗影生物状态图、裂隙、寄生体转化和牛铃诅咒生成；自身负责声画，物品、复活和惩罚由生成者处理。代理及动画结束清理见 [[reference/mechanisms/generic-fx-factory#关键流程|工厂流程]]。
- 简中 `.po` 定向检索四注册名无显示名键；中文为效果描述，非官方译名。

## 注册与构造

通用表工厂；未列 sound 或 fn 即该字段缺省。

| 注册名 | bank / build / anim | 差异与依据 |
|---|---|---|
| `fused_shadeling_spawn_fx` | `fused_shadeling` / 同 bank / `spawn_fx` | 无 sound、fn。[条目](../../../raw/current/scripts/fx.lua#L3161-L3165) |
| `dreadstone_spawn_fx` | `mutate_fx` / 同 bank / `mutate` | nofaced；fn 设置黑色不透明、FinalOffset 1；无 sound。[条目](../../../raw/current/scripts/fx.lua#L3167-L3175) |
| `shadowthrall_parasite_attach_poof_fx` | `shadow_thrall_parasite_transition_fx` / 同 bank / `transition` | sound 字段被注释，fn 另加 SoundEmitter 播 `hallowednights2024/thrall_parasite/possess_monster` 并在 35 帧后 KillSound。[条目](../../../raw/current/scripts/fx.lua#L3765-L3774) |
| `beef_bell_shadow_cursefx` | `shadowcurse` / `cowbell_shadow_cursefx` / `player_shadow` | sound `rifts4/beefalo_revive/player_debuff`，fn `FinalOffset1`。[条目](../../../raw/current/scripts/fx.lua#L3550-L3555)、[函数](../../../raw/current/scripts/fx.lua#L1-L3) |

## 关联机制导航

| 生成者 | 触发、端与落点 |
|---|---|
| 融合暗影生物 | `appear` 入状态在自身位置生成 `fused_shadeling_spawn_fx`，本体另播放出现音效。[状态](../../../raw/current/scripts/stategraphs/SGfused_shadeling.lua#L435-L446) |
| 梦魇裂隙 | 恐惧石冷却非加载恢复时父挂特效，7 帧后显示石堆；从控制释放时仅在可采且未睡眠分支父挂同一 FX 并延迟 7 帧切换动画，其他分支直接切换。[冷却](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L370-L385)、[释放](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L411-L431) |
| 寄生附体 | 中间实体动画结束后，目标有效且有 inventory、health，并成功生成面具时才装备面具、恢复生命、切换状态，再给 FX 代理加 Follower 绑定目标 `swap_hat`；未父挂目标，不能据此保证本地动画持续跟随符号。[附体](../../../raw/current/scripts/prefabs/shadowthrall_parasite.lua#L141-L170)、[[reference/mechanisms/generic-fx-factory#关键流程|工厂流程]] |
| 牛铃诅咒 | debuff 非加载附着 150 帧后把 FX `AddChild` 到目标，计划在 162 帧触发 Stop；目标 death 事件可提前 Stop 并进入脱离回调。脱离时对有效目标应用生命惩罚与存活时理智变化；加载分支立即触发，不等待该 FX。[调度](../../../raw/current/scripts/prefabs/shadow_beef_bell_curse.lua#L8-L35)、[FX 与惩罚](../../../raw/current/scripts/prefabs/shadow_beef_bell_curse.lua#L39-L67) |

## 常见陷阱与限制

- 寄生声效由条目 fn 管理，不是注释掉的 sound 字段；35 帧停声音与动画结束清理是不同事件。[条目](../../../raw/current/scripts/fx.lua#L3765-L3774)、[工厂](../../../raw/current/scripts/prefabs/fx.lua#L98-L105)
- 裂隙加载恢复不生成该 FX；不要把延迟显示石堆写成 FX 自身生成玩法对象。[恢复](../../../raw/current/scripts/prefabs/nightmarefissure.lua#L370-L385)

## 验证记录

- 源码审阅：2026-11-01，Steam buildid 26335394；核对四表项、工厂和上述直接生成链，全 scripts 搜索完整名及 `dreadstone_spawn`、`fused_shadeling_spawn`、`parasite_attach_poof`、`shadow_cursefx`。未核验动画资源与模组动态生成。
- 游戏运行：未运行。

返回 [[reference/prefabs/fx-table-index|通用表 FX 索引]]。
