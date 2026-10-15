---
title: "centipedebody"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "多节蜈蚣型 Boss 的体节编排组件：控制器实体统一生成/约束/驱动头与躯干链，支持双头换控、睡眠停摆与吃瘴气增节。"
aliases: ["centipedebody", "蜈蚣体节", "影子蜈蚣控制器", "多节 Boss 编排"]
tags: [component, creature, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/centipedebody.lua", "prefabs/shadowthrall_centipede.lua", "brains/shadowthrall_centipede_controller_brain.lua", "brains/shadowthrall_centipede_brain.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# centipedebody

## 快速判断

- 职责与适用：装在“控制器”实体上（原版为隐形 `shadowthrall_centipede`，带 planarentity 与控制大脑），负责生成、物理串联（P2P 约束）与逐节转向一整条体节链；头/躯干是独立 prefab，通过 `controller` 反向引用控制器。[安装](../../../raw/current/scripts/prefabs/shadowthrall_centipede.lua#L398-L405) [体节反引用](../../../raw/current/scripts/components/centipedebody.lua#L131)
- 优先使用：整体成链用 `CreateFullBody()`；战中增节用 `GrowNewSegment(index?)`（上限 `max_torso`）；查询/移交控制权用 `GetControllingHead()`/`GiveControlToOtherHead()`。[成链](../../../raw/current/scripts/components/centipedebody.lua#L107-L117) [增节](../../../raw/current/scripts/components/centipedebody.lua#L173-L184)
- 重要副作用：任一体节 `onremove` 会移除控制器，控制器移除又解开并删除全部体节；体节休眠令全链 Halt（大脑停摆）。[级联移除](../../../raw/current/scripts/components/centipedebody.lua#L27-L32) [停摆](../../../raw/current/scripts/components/centipedebody.lua#L64-L72)
- 修改边界：构造即 StartUpdatingComponent 并监听控制器 death 停更新；有 OnSave（体节 GUID 清单）/LoadPostPass（重挂与随机选头），OnLoad 为空。无迁移支持——换控制器不能沿用旧链。[更新与死亡](../../../raw/current/scripts/components/centipedebody.lua#L38-L62) [存读](../../../raw/current/scripts/components/centipedebody.lua#L309-L338)

## 关键接口

| 接口 | 条件、作用与证据 |
|---|---|
| `CreateFullBody()` | 生成头 + `num_torso` 个躯干 + 第二个头（尾端反向），控制权给 `heads[1]`。原版控制器在生成后 0 秒任务里调用（非 POPULATING）。[实现](../../../raw/current/scripts/components/centipedebody.lua#L107-L117) [调用](../../../raw/current/scripts/prefabs/shadowthrall_centipede.lua#L352-L355) |
| `SpawnSegment(prefab,pos?,index?)` / `SpawnHead` / `SpawnTorso` | 缺省位置为前一节背后一个直径处；插到 index 时同时重连前后约束。[实现](../../../raw/current/scripts/components/centipedebody.lua#L157-L171) |
| `SetSegment(body,index?)` | 登记 controller 反引用、插入链表、为相邻节加 P2P 约束（下一帧生效并强制刷新枢轴）、监听 sleep/wake/onremove；头部置 `has_brain_control=false`，第二个头 `flipped=true`（尾向）。[实现](../../../raw/current/scripts/components/centipedebody.lua#L127-L155) |
| `GrowNewSegment(index?)` | `#bodies >= max_torso` 时拒绝；index 缺省在非末尾随机插入；推 `grow_segment` 事件。体节吃瘴气的大脑调用它成长（[调用](../../../raw/current/scripts/brains/shadowthrall_centipede_brain.lua#L74)）。[实现](../../../raw/current/scripts/components/centipedebody.lua#L173-L184) |
| `GiveControlToHead(head)` / `GiveControlToOtherHead()` / `GiveControlToRandomHead()` | 停所有头 locomotor 后换控制头；按头的 flipped 状态给各节设 `SetBackwardsLocomotion`，非头体节 rot 反转 180。控制大脑在特定节点换头（[消费](../../../raw/current/scripts/brains/shadowthrall_centipede_controller_brain.lua#L32)）。[实现](../../../raw/current/scripts/components/centipedebody.lua#L207-L231) |
| `GetControllingHead()` / `SegmentHasControl(seg)` / `IsNonControllingHead(head)` | 控制权查询；体节大脑以 SegmentHasControl 决定谁行动（[消费](../../../raw/current/scripts/brains/shadowthrall_centipede_brain.lua#L116)）。[实现](../../../raw/current/scripts/components/centipedebody.lua#L99-L105) [L186-L188](../../../raw/current/scripts/components/centipedebody.lua#L186-L188) |
| `Halt()` / `CheckUnhalt()` / `IsHalted()` | 任一体节 entitysleep → 全头 `StopBrain("centipedebody_halt")`；全部体节 entitywake 后恢复。[触发](../../../raw/current/scripts/components/centipedebody.lua#L13-L25) [恢复](../../../raw/current/scripts/components/centipedebody.lua#L74-L93) |
| `OnUpdate(dt,force?)` | 仅当控制头 SG 带 `moving` 状态标签或强制刷新时运行；按 `turnspeed` 插值逐节朝向并沿链设置 P2P 枢轴。[实现](../../../raw/current/scripts/components/centipedebody.lua#L257-L296) |
| `OnRemoveEntity()`（含 OnRemoveFromEntity） | 逐节解除约束并 Remove 全部体节。[实现](../../../raw/current/scripts/components/centipedebody.lua#L298-L307) |

## 典型接入

### 仿原版接入自己的多节 Boss

- 前提与选择：服务端、控制器实体在 master 侧工厂里 AddComponent 后先配置参数再延迟成链。体节 prefab 必须实现被组件调用的协议面：locomotor、`SetBackwardsLocomotion`、`IsFlipped`、`StopBrain`/`RestartBrain`、health（死亡强杀）、带半径的 Physics（[调用面](../../../raw/current/scripts/components/centipedebody.lua#L207-L231)）。缺一项会在对应路径报错。
- 生效与撤销：成链后由 OnUpdate 每帧驱动；撤销走控制器 Remove（自动级联），不要逐节 Remove 后还保留控制器。[级联](../../../raw/current/scripts/components/centipedebody.lua#L27-L32)

```lua
-- 服务端：控制器实体工厂内，参照原版 shadowthrall_centipede 的写法
inst:AddComponent("centipedebody")
inst.components.centipedebody.turnspeed = 1.5   -- 转向插值（原版 TUNING.SHADOWTHRALL_CENTIPEDE.TURNSPEED）
inst.components.centipedebody.num_torso = 5     -- CreateFullBody 的初始躯干数
inst.components.centipedebody.max_torso = 20    -- GrowNewSegment 的上限
inst.components.centipedebody.headprefab = "my_boss_head"
inst.components.centipedebody.torsoprefab = "my_boss_body"
if not POPULATING then
    inst:DoTaskInTime(0, function() inst.components.centipedebody:CreateFullBody() end)
end
```

- 调整与验证：`turnspeed` 是每秒朝向插值比例（1.5 为原版值，[调参](../../../raw/current/scripts/tuning.lua#L8885)）；验证顺序——成链后拖动控制头，各节应延迟跟随且转向平滑。

## 常见陷阱与限制

- **max_torso 名不符实：** 上限按 `#self.bodies`（含两个头）比较，`max_torso=20` 实际最多 18 个躯干；按名字理解会多算 2 节。[比较处](../../../raw/current/scripts/components/centipedebody.lua#L174)
- **直接 Remove 体节等于杀全虫：** 体节 onremove → 控制器 Remove → 全链删除；“断节”玩法必须改造协议而不是直接移除实体。[级联](../../../raw/current/scripts/components/centipedebody.lua#L27-L32)
- **控制头依赖 SG 状态标签：** OnUpdate 检查控制头 `sg:HasStateTag("moving")`，自定义头缺少该标签时链条只在强制刷新（约束建立、换头）时摆一次姿态。[门禁](../../../raw/current/scripts/components/centipedebody.lua#L258-L261)
- **存档只存 GUID 清单：** OnLoad 为空，重挂全部依赖 LoadPostPass 拿到 newents 里的体节实体；体节必须可持久化，否则读档后链条缺失且控制权随机重指（TODO 注释未完善）。[读档](../../../raw/current/scripts/components/centipedebody.lua#L327-L338)
- **睡眠停摆是设计行为：** 玩家远离导致部分体节休眠时全链 Halt；调试“Boss 不动”先查体节是否跨睡眠边界。[停摆](../../../raw/current/scripts/components/centipedebody.lua#L64-L72)

## 验证记录

- 源码审阅：2026-10-28，Steam buildid 26230496；完整读取 centipedebody.lua，核验成链/增节/换头/停摆/移除级联/存读，及控制器与体节大脑的全部调用点、TUNING 参数块。未运行游戏；攻击、瘴气进食等战斗行为属大脑与 prefab 页面范围。

相关：[[reference/components/aoespell]]、[[reference/components/shadowdominance]]。
