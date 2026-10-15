---
title: "madsciencelab"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "多阶段制作机的分阶段进度机：按 stages 表逐阶段计时推进，完成后回调产出；疯狂科学实验室、贝壳织机、冬季盛宴烤箱共用。"
aliases: ["madsciencelab", "疯狂科学实验室进度", "分阶段制作机"]
tags: [component, recipe]
size_class: standard
dst_build: "steam-buildid:26230496"
sources: ["components/madsciencelab.lua", "prefabs/madscience_lab.lua", "prefabs/shellweaver.lua", "prefabs/wintersfeastoven.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# madsciencelab

## 快速判断

- 职责与适用：让建筑类实体承载"开始制作 → 逐阶段动画/计时 → 完成回调产出"的流程。阶段表 `stages` 与三个回调由宿主 prefab 配置，组件只管推进与存续。[阶段推进](../../../raw/current/scripts/components/madsciencelab.lua#L23-L41)
- 优先使用：制作入口通常不直接调组件，而是经 [[reference/components/prototyper]] 的 `onactivate` 转发（`madscience_lab.lua:95-99` 配 `StartMakingScience`）；右键 COOK 动作由组件标签驱动。[动作入口](../../../raw/current/scripts/componentactions.lua#L545-L553)
- 重要副作用：阶段推进用 `DoTaskInTime` 而非 `LongUpdate`（空实现）， 世界跳帧不加速制作；完成/每阶段回调 `OnScienceWasMade`/`OnStageStarted` 是宿主驱动动画与发产物的唯一钩子。[LongUpdate 空置](../../../raw/current/scripts/components/madsciencelab.lua#L78-L79)
- 修改边界：组件存读 `name/product/stage/time_remaining`，但宿主须自行恢复 `stages` 表（prefab 重建时重配）；`stage` 字段在 `SetStage` 前为 nil，`OnSave` 可能存入 nil 值由序列化剔除。[存档](../../../raw/current/scripts/components/madsciencelab.lua#L53-L60)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `stages` | 数组，元素含 `time`（宿主可加 anim 等字段） | 阶段配置；`SetStage` 超出长度即视为完成。原版疯狂实验室 3 阶段×2 秒。[原版配置](../../../raw/current/scripts/prefabs/madscience_lab.lua#L16-L20) |
| `StartMakingScience(product, name)` | product: string；name 可选 | 记录产物（name 供配方名等场景）、从第 1 阶段开始并触发 `OnStartMakingScience`。[实现](../../../raw/current/scripts/components/madsciencelab.lua#L43-L51) |
| `SetStage(stage, time_override)` | stage: number；time_override 用于存档恢复 | 递归推进：计时 `time_override or stages[stage].time` 后进入下一阶段；完成时清状态并调 `OnScienceWasMade(inst, result)`。[实现](../../../raw/current/scripts/components/madsciencelab.lua#L23-L41) |
| `IsMakingScience()` | `-> boolean` | `task ~= nil`；宿主用于动画与状态查询（如"MIXING"）。[实现](../../../raw/current/scripts/components/madsciencelab.lua#L19-L21) |
| `OnStartMakingScience` / `OnStageStarted` / `OnScienceWasMade` | 宿主赋值的回调 | 分别在开始、每阶段开始（参数 stage）、完成（参数 result=product）时调用。[调用点](../../../raw/current/scripts/components/madsciencelab.lua#L30-L39) |
| `OnSave` / `OnLoad` | 存读钩子 | 存 `name/product/stage/time_remaining`；加载用剩余时间恢复对应阶段。[实现](../../../raw/current/scripts/components/madsciencelab.lua#L62-L68) |

## 典型接入

### 带制作台的多阶段建筑（原版参照）

- 前提与选择：宿主需 [[reference/components/prototyper]]（触发 StartMakingScience）并自行定义 stages 与三个回调；疯狂实验室在回调里推动画（`OnStageStarted`，`madscience_lab.lua:167`）与发放实验产物（`OnScienceWasMade`，`madscience_lab.lua:186`）。[回调配置](../../../raw/current/scripts/prefabs/madscience_lab.lua#L299-L302)
- 生效与撤销：开始制作即占机（`IsMakingScience` 为真），宿主动画分支据此切换；组件被移除时取消当前计时任务但不发产物。[取消](../../../raw/current/scripts/components/madsciencelab.lua#L12-L17)

```lua
-- 服务端：宿主 prefab 配置三件套
inst:AddComponent("madsciencelab")
inst.components.madsciencelab.stages = {
    { time = 2 },                       -- 字段名 time 必需，其余自定
    { time = 2 },
}
inst.components.madsciencelab.OnStageStarted = function(inst, stage) end
inst.components.madsciencelab.OnScienceWasMade = function(inst, product)
    inst.components.lootdropper:SpawnLootPrefab(product)
end
```

- 调整与验证：烧制时长按阶段累加；恢复存档时 `SetStage(data.stage, data.time_remaining)` 会先跑剩余时间再继续，勿在 OnLoad 里重置 stage。

## 常见陷阱与限制

- **忘了重配 `stages`：** 组件不持久化 stages，载入后若宿主未重建该表，续阶段会索引 nil 报错；三个原版宿主都在 master postinit 赋值。[宿主赋值](../../../raw/current/scripts/prefabs/madscience_lab.lua#L302)
- **期待 LongUpdate 加速：** `LongUpdate` 为空，跳过时间不会推进制作；阶段计时用 `DoTaskInTime`，受休眠影响由宿主实体唤醒机制决定。[空实现](../../../raw/current/scripts/components/madsciencelab.lua#L78-L79)
- **同宿主并发开始：** 组件无占用保护，外部在制作中再调 `StartMakingScience` 会覆盖 product/name 并从第 1 阶段重启；入口侧需用 `IsMakingScience()` 拦截（原版 prototyper 入口未拦，依赖 `onactivate` 时机）。[覆盖行为](../../../raw/current/scripts/components/madsciencelab.lua#L43-L46)
- **同组件异用途：** 贝壳织机（`shellweaver.lua:228`）与冬季盛宴烤箱（`wintersfeastoven.lua:353`）共用本组件但 stages/回调/产物各异，参考实现时选对应 prefab。

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取组件（81 行），核对 madscience_lab 的 stages/回调/prototyper 链（`prefabs/madscience_lab.lua:95-99,299-302`）、shellweaver 与 wintersfeastoven 挂载、componentactions COOK 条件。
- 结构检查：2026-10-28 validate_page.py 通过；validate_source_links.py --strict 锚点通过（见批级记录）。
- 游戏运行：未运行。
