---
title: "大赛庆祝彩纸（confetti_fx）"
created: "2026-10-29"
updated: "2026-11-01"
type: prefab
summary: "牛年大赛颁奖与兔年枕头大战终局的一次性彩纸爆发：剪影形状发射器、约千粒彩纸加闪光、专服不建粒子；同文件另一注册 rabbit_confetti_fx。"
aliases: ["confetti_fx", "rabbit_confetti_fx", "庆祝彩纸", "彩纸特效", "confetti"]
tags: [prefab, fx, server-client]
size_class: standard
dst_build: "steam-buildid:26335394"
sources: ["prefabs/confetti_fx.lua", "components/yotb_stager.lua", "prefabs/yotb_stage.lua", "prefabs/yotr_fightring.lua", "components/golfpropitem.lua", "fx.lua", "emitters.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 大赛庆祝彩纸（confetti_fx）

## 快速判断

- 职责与适用：一次性庆祝彩纸爆发——纯 VFX 粒子实体，无 AnimState：主模拟生成网络壳，所有非专服机器各自在 0 延迟任务中按剪影形状发射 5×(150~200) 粒彩纸与 3 粒闪光，该分支发射后 Remove；专服壳无此删除任务。同文件注册 `rabbit_confetti_fx`（兔形剪影）。[注册](../../../raw/current/scripts/prefabs/confetti_fx.lua#L354-L355)
- 优先使用：查"赛果庆祝彩纸"选本页；本快照检索到的原版生成点为两处——牛年大赛裁判席颁奖（[[reference/components/yotb_stager#关键流程]]）与兔年枕头大战坑终局。
- 重要副作用：非专服分支发射完成即 `inst:Remove()`，persists=false 不存档；音效 `yotb_2021/common/fireworks` 与粒子同在非专服分支内。无玩法组件。[自毁](../../../raw/current/scripts/prefabs/confetti_fx.lua#L284)
- 修改边界：专服在 VFX 初始化前提前返回——无粒子也无音效；两注册名共用同一套发射器参数，只差三角剖分形状表。[专服分支](../../../raw/current/scripts/prefabs/confetti_fx.lua#L301-L303)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `confetti_fx` | MakeConfetti("confetti_fx", BEEFALO_SHAPE_TRIS)：Transform＋Network＋SoundEmitter＋FX 标签 → SetPristine → persists=false → 专服返回 → 非专服：注册包络＋VFXEffect(InitEmitters(6)) → DoTaskInTime(0) 发射一轮并 Remove | 彩纸发射点采样自牛形三角剖分，爆发初始呈剪影再四散。[注册](../../../raw/current/scripts/prefabs/confetti_fx.lua#L354)、[形状表](../../../raw/current/scripts/prefabs/confetti_fx.lua#L1-L33)、[工厂](../../../raw/current/scripts/prefabs/confetti_fx.lua#L287-L352) |
| `rabbit_confetti_fx` | 同上，data.tri_list 换 RABBIT_SHAPE_TRIS | 兔形剪影；生成点为枕头大战终局。[形状表](../../../raw/current/scripts/prefabs/confetti_fx.lua#L35-L106) |

两注册名均无 NAMES 条目；关联官方译名见关联机制导航（裁判席、枕头大战坑）。标题中文为已核验效果描述（confetti_fx），非官方译名。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 生成者：yotb_stager DeclareWinner | 牛年大赛宣布冠军后约 4 秒，在获胜围栏位生成 `confetti_fx` | 主模拟；比赛主链见 [[reference/components/yotb_stager#关键流程]] | [DeclareWinner](../../../raw/current/scripts/components/yotb_stager.lua#L863)、[生成](../../../raw/current/scripts/components/yotb_stager.lua#L955-L956) |
| 声明：yotb_stage 依赖 | `confetti_fx` 列于裁判席 Prefab 依赖清单（同清单另有 `yotb_confetti` 等独立条目） | 部署裁判席时注册 | [清单](../../../raw/current/scripts/prefabs/yotb_stage.lua#L13-L19) |
| 生成者：yotr_fightring do_confetti | 枕头大战坑（官方译名，[NAMES](../../../raw/current/scripts/languages/chinese_s.po#L352434-L352437)）终局在铃铛或场地旁生成 `rabbit_confetti_fx` | 主模拟 | [do_confetti](../../../raw/current/scripts/prefabs/yotr_fightring.lua#L239-L253) |
| 近名区分：carnival_confetti_fx | 独立注册（fx.lua），golfpropitem／狂欢节炮／小鸡赛跑生成的是它，不是本页 Prefab | 狂欢节彩纸族 | [golfpropitem 生成](../../../raw/current/scripts/components/golfpropitem.lua#L55)、[carnival 注册](../../../raw/current/scripts/fx.lua#L2334)、[[reference/components/golfpropitem#典型接入]] |
| VFXEffect 配置 | 发射器 0~4：confetti.tex 4 帧 UV、旋转粒子、各 200 上限、寿命 2.7、地面物理、加速度 (0,-0.8,0)、阻力 0.1；发射器 5：sparkle.tex 加法混合闪光、3 粒、寿命 0.2 | 非专服各机本地发射；NUM_EMITTERS=4 但循环 0~4，实为 5 路彩纸 | [彩纸发射器](../../../raw/current/scripts/prefabs/confetti_fx.lua#L311-L327)、[闪光发射器](../../../raw/current/scripts/prefabs/confetti_fx.lua#L329-L340)、[发射循环](../../../raw/current/scripts/prefabs/confetti_fx.lua#L264-L285) |
| API／数据：Create2DTriEmitter | 彩纸从 tri_list 三角形内随机点发出，形成剪影 | 形状采样缩放 0.5，结果用于初速度；出生位置固定 (0,3,0)。UnitRand∈[-1,1]，寿命公式范围 0~2.7 秒 | [helper](../../../raw/current/scripts/emitters.lua#L200)、[emit_confetti_fn](../../../raw/current/scripts/prefabs/confetti_fx.lua#L224-L243) |
| 资源／音效 | fx/confetti.tex、fx/sparkle.tex、shaders/vfx_particle(_add).ksh；`yotb_2021/common/fireworks` | 音效在专服提前返回之后，专服静默 | [资源](../../../raw/current/scripts/prefabs/confetti_fx.lua#L108-L126)、[音效](../../../raw/current/scripts/prefabs/confetti_fx.lua#L346) |

## 典型接入

### 原地放一发庆祝彩纸

- 前提与选择：与原版生成者一致——主模拟 SpawnPrefab 后摆位；形状由注册名选择。专服会在自删任务前返回，调用方须另行安排网络壳清理。
- 生效与撤销：非专服 0 延迟任务发射一次并 Remove；专服壳不会自行删除，示例由调用方延迟兜底清理。粒子移除后的实际渲染未运行验证。
- 原版依据：[yotb_stager](../../../raw/current/scripts/components/yotb_stager.lua#L955-L956) 即此用法。

```lua
local fx = SpawnPrefab("confetti_fx")  -- 或 "rabbit_confetti_fx"
fx.Transform:SetPosition(x, 0, z)
-- 主模拟：专服分支没有自删任务，延后兜底清理网络壳。
fx:DoTaskInTime(4, fx.Remove)
```

- 调整与验证：粒子数／寿命／重力都是工厂常量（MAX_LIFETIME 2.7 等），要改需 postinit 覆盖或自建同构工厂，无运行时接口；文件尾注释给出调试命令 `c_spawn("confetti_fx")`（[注释](../../../raw/current/scripts/prefabs/confetti_fx.lua#L357-L359)）。

## 常见陷阱与限制

- **误区：高尔夫道具触发本彩纸**：golfpropitem 回传时生成的是 `carnival_confetti_fx`（fx.lua 独立注册）；狂欢节炮、小鸡赛跑同。本页两注册名的生成点只有大赛颁奖与枕头大战终局。[证据](../../../raw/current/scripts/components/golfpropitem.lua#L55)
- **边界：专服无粒子无音效**：`IsDedicated` 提前返回且音效在其后——专服完全静默；主机与客户端各自本地发射一轮，客户端本地 Remove 的引擎行为未核验。[专服分支](../../../raw/current/scripts/prefabs/confetti_fx.lua#L301-L303)
- **边界：旋转只取单向**：角速度 `(UnitRand()-1)*5` 取值 [-10, 0]，彩纸只向一个方向自旋；速度基于相机右/上向量计算，属于屏幕对齐爆发。[emit_confetti_fn](../../../raw/current/scripts/prefabs/confetti_fx.lua#L224-L243)、[相机向量](../../../raw/current/scripts/prefabs/confetti_fx.lua#L265-L268)

## 验证记录

- 源码审阅：2026-10-30，Steam buildid 26230496；359 行工厂全文（两形状表、包络、发射器配置、发射与自毁、专服分支）、两个生成点上下文、carnival_confetti_fx 区分、Create2DTriEmitter helper、裁判席与枕头大战坑译名已核验；confetti.tex 帧内容、客户端本地 Remove 的引擎行为、yotb_confetti 与 balloonparty 等其他"彩纸"名族的关系未覆盖。
- 游戏运行：未运行。
- 结构检查：validate_page、validate_source_links --strict、label-check 通过（结果见本批交付报告）。

返回 [[reference/prefabs/fx-index|特效 Prefab 索引]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
