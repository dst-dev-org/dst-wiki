---
title: "tacklesketch"
created: "2026-10-23"
updated: "2026-10-28"
type: component
summary: "识别渔具图纸交给工作站的旧组件路径；当前原版图纸实际使用 teacher，不能按同名 Prefab 假定挂载。"
aliases: ["tacklesketch", "渔具图纸", "海钓图纸", "GIVE_TACKLESKETCH", "渔具工作台教学"]
tags: [component, recipe, actions, lifecycle]
dst_build: "steam-buildid:26230496"
sources: [components/tacklesketch.lua, prefabs/tacklesketch.lua, prefabs/tacklestation.lua, components/craftingstation.lua, components/teacher.lua, actions.lua, componentactions.lua, stategraphs/SGwilson.lua, stategraphs/SGwilson_client.lua]
reliability: verified
validation: source-reviewed
status: active
---

# tacklesketch

## 快速判断

- 职责与适用：这个短组件把载体对应图纸／配方写入目标 craftingstation，通知工作台后删除载体；名称中的 tackle 指渔具，与 [[reference/components/tackler]] 冲撞无关。[完整组件](../../../raw/current/scripts/components/tacklesketch.lua#L1-L20)
- 优先使用：维护当前原版海钓图纸，先查 [[reference/components/teacher#典型接入]]；当前 `prefabs/tacklesketch.lua` 已注释挂载同名组件，实际添加 teacher，并由具体图纸工厂设置配方。[实际挂载](../../../raw/current/scripts/prefabs/tacklesketch.lua#L79-L84)、[配方设置](../../../raw/current/scripts/prefabs/tacklesketch.lua#L108-L119)
- 重要副作用：组件 `Teach(target)` 无成功返回、无重复／燃烧／参数检查；成功走完即推事件并删除图纸，站点去重由 craftingstation 和动作层分工。[Teach](../../../raw/current/scripts/components/tacklesketch.lua#L13-L17)、[动作门禁](../../../raw/current/scripts/actions.lua#L5138-L5151)
- 修改边界：本页记录仍保留的旧实现与当前替代，不能据此声称原版图纸现在可直接投喂工作台；不建议为了沿用旧动作给当前图纸重新挂这个组件。[旧动作](../../../raw/current/scripts/actions.lua#L5144-L5148)、[当前工厂](../../../raw/current/scripts/prefabs/tacklesketch.lua#L82-L84)

## 关键接口

| 接口／约定 | 条件与行为 | 证据 |
|---|---|---|
| 构造／`OnRemoveFromEntity()` | 添加／移除 `tacklesketch` 标签；自身无 netvar 或存档字段 | [构造与清理](../../../raw/current/scripts/components/tacklesketch.lua#L1-L10) |
| `Teach(target)` | 目标必须有 craftingstation；载体必须实现 `GetSpecificSketchPrefab()` 和 `GetRecipeName()`。依次 LearnItem、向目标推 `onlearnednewtacklesketch`、删除载体，无显式返回 | [实现](../../../raw/current/scripts/components/tacklesketch.lua#L13-L17) |
| `GetSpecificSketchPrefab()`／`GetRecipeName()` | 是图纸 Prefab 提供的宿主方法，不是组件的方法；由当前 sketchid 返回具体图纸 Prefab 名／配方名 | [宿主实现](../../../raw/current/scripts/prefabs/tacklesketch.lua#L38-L43)、[方法安装](../../../raw/current/scripts/prefabs/tacklesketch.lua#L100-L103) |
| `onlearnednewtacklesketch` | 无额外参数的目标事件；工作台监听后播放接收动画／声音，不代表自动给所有玩家解锁配方 | [发送](../../../raw/current/scripts/components/tacklesketch.lua#L15)、[接收表现](../../../raw/current/scripts/prefabs/tacklestation.lua#L99-L107)、[监听](../../../raw/current/scripts/prefabs/tacklestation.lua#L198) |

站点 item→recipe 配对、保存恢复、去重和配方可用范围只在 [[reference/components/craftingstation#关键接口]] 维护；人物永久学习的 builder 门禁及 teacher 消耗则见 [[reference/components/teacher#关键接口]]。

## 典型接入

### 判断当前渔具图纸应该扩展哪条学习路径

1. **核对实际组件而非名字：** 原版公共工厂保留 `tacklesketch` 标签和统一实体名，但 `AddComponent("tacklesketch")` 已注释；有 teacher 不等于同时具备旧站点组件。[公共工厂](../../../raw/current/scripts/prefabs/tacklesketch.lua#L63-L84)
2. **当前人物学习：** 具体 `<item>_tacklesketch` 工厂设置 teacher.recipe；学习回调给 learner 发送 `learnrecipe`。需要学习奖励／改配方时沿用 [[reference/components/teacher#典型接入]]，不重写一份教学代码。[具体工厂](../../../raw/current/scripts/prefabs/tacklesketch.lua#L108-L139)、[回调](../../../raw/current/scripts/prefabs/tacklesketch.lua#L46-L47)
3. **基础实体与读档：** 公共工厂默认 sketchid=1，但并未在此设置 teacher.recipe；OnLoad 根据保存的 sketchid 重新设置配方／名称。扩展要考虑具体工厂与读档，不把生成基础 `tacklesketch` 等同生成有效的指定图纸。[默认](../../../raw/current/scripts/prefabs/tacklesketch.lua#L97-L103)、[保存恢复](../../../raw/current/scripts/prefabs/tacklesketch.lua#L23-L35)、[teacher 空配方处理](../../../raw/current/scripts/components/teacher.lua#L10-L13)

本页不新增旧式投喂 demo；当前原版使用已核验的人物教学路径。目标模组若确实保留旧组件，须按下节核对自己的载体与工作台，不能把原版标签当作完整组件契约。

### 读旧模组时追踪 GIVE_TACKLESKETCH 的完整边界

- **操作收集：** 该组件动作收集器要求目标 `tacklestation` 且无 fire／smolder，再提供 GIVE_TACKLESKETCH。收集器按已注册的 actioncomponents 调用，单独添加载体标签不能代替组件注册。[收集条件](../../../raw/current/scripts/componentactions.lua#L1834-L1838)、[注册](../../../raw/current/scripts/componentactions.lua#L3182-L3188)、[调度](../../../raw/current/scripts/componentactions.lua#L3235-L3245)
- **执行与时机：** 人物两端将该动作映射到 give；服务端动作要求 invobject／target、目标 craftingstation、未烧毁且未燃烧，按具体图纸名检查 DUPLICATE 后直接调载体组件。动作没有再防御性检查载体同名组件存在，不能仅构造动作就绕过完整前提。[SG 服务端](../../../raw/current/scripts/stategraphs/SGwilson.lua#L1554)、[SG 客户端](../../../raw/current/scripts/stategraphs/SGwilson_client.lua#L825)、[动作](../../../raw/current/scripts/actions.lua#L5138-L5151)
- **站点结果与回收：** 原版工作台保留 craftingstation／FISHING prototyper、接收事件表现与按站点条目掉落图纸的代码。锤毁回收已记录条目，烧毁先掉条目再 ForgetAllItems；不能据此反推当前图纸仍能走旧入口。[站点组装](../../../raw/current/scripts/prefabs/tacklestation.lua#L182-L198)、[掉落](../../../raw/current/scripts/prefabs/tacklestation.lua#L21-L38)、[烧毁](../../../raw/current/scripts/prefabs/tacklestation.lua#L120-L124)

## 常见陷阱与限制

- **直接对任意目标 Teach：** 缺 craftingstation 或两个宿主方法会直接访问失败；直接调用也不检查重复／燃烧，重复条目仍可能推事件并消耗载体。原版完整旧动作先检查，新增流程需明确实现自己的前提。[组件](../../../raw/current/scripts/components/tacklesketch.lua#L13-L17)、[去重](../../../raw/current/scripts/components/craftingstation.lua#L9-L13)、[动作](../../../raw/current/scripts/actions.lua#L5138-L5148)
- **把工作站教学当人物解锁：** 这里只登记目标站点，没有调用人物 builder；目标是人物永久学习时应使用 teacher 路径，不能向人物硬塞 craftingstation 冒充学习。[站点写入](../../../raw/current/scripts/components/tacklesketch.lua#L14)、[当前 teacher](../../../raw/current/scripts/components/teacher.lua#L10-L26)
- **任意重排图纸列表：** Prefab 以 sketchid 保存索引，恢复直接访问 SKETCHES；原注释要求只在末尾增加、不重排。改载体映射时须另行设计旧档迁移，不将组件无存档误读为图纸无存档。[列表约定](../../../raw/current/scripts/prefabs/tacklesketch.lua#L6-L20)、[存取](../../../raw/current/scripts/prefabs/tacklesketch.lua#L23-L35)

## 验证记录

### 版本增量复核 26148036（2026-10-25）

- 处理范围：锚点重定位 9 处（componentactions／actions／SGwilson／SGwilson_client 行号迁移，区间文本逐段比对一致；GIVE_TACKLESKETCH 收集与执行语义未变）；结论修订：无，仅锚点迁移。
- 依据：枢纽 diff 审阅（actions.lua／componentactions.lua hunk 地图）＋变更清单命中 actions.lua、componentactions.lua、stategraphs/SGwilson.lua、stategraphs/SGwilson_client.lua。

- 源码审阅：2026-10-23，Steam buildid 25643519；完整 tacklesketch 组件与图纸工厂、定向旧动作与收集注册、人物 SG 映射、工作台接收与图纸回收；复用已建 teacher／craftingstation 页的学习和站点基础机制。没有把保留代码认定为当前原版可达玩法，也未审校所有海钓配方或模组迁移。
- 结构检查：本页元数据、链接、大小与源码引用定向检查通过；没有新增完整 Lua 示例。
- 游戏运行：未运行；旧模组兼容、图纸存档迁移及当前人物学习的实际联机表现未实测。
