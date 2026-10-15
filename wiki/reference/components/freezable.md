---
title: "freezable"
created: "2026-10-16"
updated: "2026-10-28"
type: component
summary: "配置生物冻结抗性与寒冷积累、冻结和解冻阶段，核对受击解冻、状态机及保存边界。"
aliases: ["冰冻", "冻结抗性", "冰杖冻结", "寒冷积累", "解冻", "freezable"]
tags: [component, creature, state-machine, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/freezable.lua", "standardcomponents.lua", "prefabs/staff.lua", "prefabs/staff_common.lua", "prefabs/pigman.lua", "prefabs/player_common.lua", "stategraphs/SGpig.lua", "stategraphs/commonstates.lua", "entityscript.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# freezable

## 快速判断

- 职责与适用：维护目标 `NORMAL → FROZEN → THAWING` 状态、`coldness` 积累及抗性。它不是摄氏温度，也不替代人物体温、冬季保温或热源；后者看 [[reference/components/temperature]]、[[reference/components/insulator]]。[freezable.lua:37](../../../raw/current/scripts/components/freezable.lua#L37
- 优先使用：施加冰冻积累用 `AddColdness`，目标难冻程度用 `SetResistance`；主动解除冻结用 `Unfreeze`。源码注明优先 AddColdness，而非直接 Freeze 绕过积累判定。[freezable.lua:137](../../../raw/current/scripts/components/freezable.lua#L137 [freezable.lua:247](../../../raw/current/scripts/components/freezable.lua#L247
- 重要副作用：Freeze 停 Brain、清战斗目标并停止 locomotor，状态变化时发 `freeze`；解冻阶段 `IsFrozen()` 仍为 true。真正动画／状态跳转还依赖宿主 SG。[freezable.lua:115](../../../raw/current/scripts/components/freezable.lua#L115 [freezable.lua:248](../../../raw/current/scripts/components/freezable.lua#L248
- 修改边界：原版目标在服务端工厂添加组件；新实体只挂组件不足以接好冻结动画／行为。优先改已有目标配置，新增宿主再核验 SG 事件、冻结资源与网络初始化。[pigman.lua:634](../../../raw/current/scripts/prefabs/pigman.lua#L634 [pigman.lua:724](../../../raw/current/scripts/prefabs/pigman.lua#L724

## 关键接口

| 接口／字段 | 约定 | 条件、作用与证据 |
|---|---|---|
| `SetResistance(resist)`／`ResolveResistance()` | 设置基础值／读取生效阈值 | 默认基础值 1，setter 不夹范围或重算现存状态；有 extraresist 时阈值为 `min(基础×2.5, 基础+额外)`。按需求使用正数，不能仅改阈值就假定立即冻结／解冻。[freezable.lua:73](../../../raw/current/scripts/components/freezable.lua#L73 [freezable.lua:231](../../../raw/current/scripts/components/freezable.lua#L231 |
| `AddColdness(amount, freezetime?, nofreeze?)` | 积累量、冻结首阶段秒数?、门禁标志? | 先执行 redirectfn；未接管则积累最低为 0。正数才检查冻结／重计时，负数不退出冻结；第三参数的实际限制见下节。[freezable.lua:133](../../../raw/current/scripts/components/freezable.lua#L133 |
| `IsFrozen()`／`IsThawing()` | boolean | Frozen 包含 FROZEN 与 THAWING；只判解冻阶段用 IsThawing，不与 SG 的 `frozen` 标签混为同一查询。[freezable.lua:115](../../../raw/current/scripts/components/freezable.lua#L115 |
| `SetDefaultWearOffTime(t)`／`StartWearingOff(t?)` | 秒；默认 10 | 前者只改后续默认值；后者取消旧 task 并调度新阶段，ResolveWearOffTime 会因额外抗性缩短时间。未冻时每次到期减 1 点积累，不是每秒连续衰减。[freezable.lua:10](../../../raw/current/scripts/components/freezable.lua#L10 [freezable.lua:163](../../../raw/current/scripts/components/freezable.lua#L163 |
| `GetTimeToWearOff()`／`GetFreezePercent()` | 当前 task 剩余秒或 nil／积累÷生效阈值 | 前者只代表当前 task，不是到完全解冻的总时长；后者未夹到 0..1，不作安全进度值直接展示。[freezable.lua:170](../../../raw/current/scripts/components/freezable.lua#L170 [freezable.lua:243](../../../raw/current/scripts/components/freezable.lua#L243 |
| `Unfreeze()`／`Thaw(thawtime?)` | 立即解除／进入解冻阶段 | Unfreeze 只在 IsFrozen 时清积累与受击累计、生成碎冰，活体重启 Brain、发 unfreeze 并短暂屏蔽攻击；Thaw 不恢复行动。[freezable.lua:284](../../../raw/current/scripts/components/freezable.lua#L284 |
| `SetRedirectFn(fn)` | `fn(inst, amount, freezetime, nofreeze)` | 返回真时整个 AddColdness 提前返回；单回调槽，已有行为应保留，不当来源叠加接口。[freezable.lua:133](../../../raw/current/scripts/components/freezable.lua#L133 |
| `diminishingreturns`／`SetExtraResist(x)` | 可选开关／额外抗性 | 默认未启用；状态转入 FROZEN 时可加基础×0.25，额外值影响阈值与时长，30 秒任务按当前积累衰减。不能把全部怪物假定为固定次数冻住。[freezable.lua:209](../../../raw/current/scripts/components/freezable.lua#L209 [freezable.lua:275](../../../raw/current/scripts/components/freezable.lua#L275 |

## 关键流程

### 积累、冻结与两段解冻

`AddColdness` 的正增量在未冻结时：低于抗性 → 重启衰减 task；达到阈值且有 SG 的 nofreeze 参数／状态标签 → 夹到阈值并重启 task；否则尝试 Freeze。**传 nofreeze 仅在目标有 SG 的这条分支有效**；目标已 Frozen 时更早进入 Freeze 刷新分支，不受该判断拦截。[freezable.lua:143](../../../raw/current/scripts/components/freezable.lua#L143

Freeze 要求实体可见且未死亡；成功设置 FROZEN、刷新 task、停 Brain／战斗／移动，状态变化才发 freeze，但 `onfreezefn(inst)` 在每次通过门禁时都先调用。第一段 task 到期执行 Thaw，置 coldness=0、发 `onthaw` 并启动第二段；第二段到期才 Unfreeze。**AddColdness 的 freezetime 不是完整冻住时长**。[freezable.lua:248](../../../raw/current/scripts/components/freezable.lua#L248 [freezable.lua:305](../../../raw/current/scripts/components/freezable.lua#L305

无额外抗性、未受击／刷新时，两段通常是 `freezetime或默认 + 默认`。启用 extraresist 后 Thaw 内先 ResolveWearOffTime、StartWearingOff 内再计算一次，第二段不能假定只缩短一次；本页保留实际调用顺序，不把它概括为固定倍率。[freezable.lua:163](../../../raw/current/scripts/components/freezable.lua#L163 [freezable.lua:237](../../../raw/current/scripts/components/freezable.lua#L237

### 状态机、受击与清理

猪人注册 SGpig、OnFreeze 和 CommonStates.AddFrozenStates；SG 处理 `freeze／onthaw／unfreeze`，播放冻结资源、切状态与退出清理。通用 OnFreeze 要求 health 存在且活着；无 health 的对象需另查 OnFreezeEx 及宿主，不把角色 helper 当通用完整工厂。[pigman.lua:311](../../../raw/current/scripts/prefabs/pigman.lua#L311 [SGpig.lua:22](../../../raw/current/scripts/stategraphs/SGpig.lua#L22 [SGpig.lua:347](../../../raw/current/scripts/stategraphs/SGpig.lua#L347 [commonstates.lua:51](../../../raw/current/scripts/stategraphs/commonstates.lua#L51 [commonstates.lua:1344](../../../raw/current/scripts/stategraphs/commonstates.lua#L1344

组件监听目标 `attacked`，冻结期间累加 `abs(data.damage)`，达到 damagetobreak 就 Unfreeze；默认该阈值为 0，因此不能认为只有正伤害才能打碎。并非所有改血入口都会发此事件，伤害路径见 [[reference/components/combat]]。[freezable.lua:24](../../../raw/current/scripts/components/freezable.lua#L24

OnRemoveFromEntity 只移监听／freezable 标签、取消两个 task，**不会调用 Unfreeze、重启 Brain 或发送退出事件**；活体撤销先按宿主执行正常解冻。Reset 也只改 state／coldness／tint，不清受击累计、取消 task 或发事件，不能用作普通解冻。[freezable.lua:62](../../../raw/current/scripts/components/freezable.lua#L62 [freezable.lua:315](../../../raw/current/scripts/components/freezable.lua#L315 Brain 按停止原因成对恢复见 [entityscript.lua:1100](../../../raw/current/scripts/entityscript.lua#L1100)。

## 典型接入

### 提高普通猪人的冻结抗性与阶段时长

追加到目标模组 `modmain.lua`；原版 pigman 已有组件、SG 与冻结资源，不添加组件、不替换 Brain／SG，无额外资源。只修改注册名 pigman 的新建／重建实例，不扩展至 pigguard 或全部生物。[pigman.lua:724](../../../raw/current/scripts/prefabs/pigman.lua#L724 [modutil.lua:593](../../../raw/current/scripts/modutil.lua#L593

```lua
AddPrefabPostInit("pigman", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local freezable = inst.components.freezable
    if freezable ~= nil then
        freezable:SetResistance(4)
        freezable:SetDefaultWearOffTime(5)
    end
end)
```

4 是基础积累阈值，不是摄氏度；5 是默认阶段／衰减间隔秒数，不是完整冻结五秒。无额外抗性／受击／其他刷新时，正常两阶段合计约十秒；实际玩法还受事件、变身与其他模组配置影响。停用本例后新建／重建实例用原工厂值；运行中反向关闭需恢复该实例原配置，不硬编码其他模组修改之前的默认值。

### 调整原版冰杖每次命中的积累

追加到目标模组 `modmain.lua`，各端使用同一模组版本，无新增资源。只修改 icestaff 的 `icestaff_coldness`，保留原 weapon 回调的熄火、唤醒、目标／有效性检查及碎冰，不覆盖 SetOnAttack；独立入口显式读 GLOBAL，已有代理项目沿用项目约定。[回调](../../../raw/current/scripts/prefabs/staff_common.lua#L159) [耗冷消费](../../../raw/current/scripts/prefabs/staff_common.lua#L199)26148036 起 `onattack_blue` 随法杖工厂迁入 staff_common.lua（`fns.blue` 安装），行为与旧版 staff.lua 内一致。

```lua
AddPrefabPostInit("icestaff", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    inst.icestaff_coldness = 2
end)
```

这是该 Prefab 的工厂字段，不是 freezable 通用 setter；26148036 起注册链为 `blue_common`（staff.lua，设 `icestaff_coldness` 后调 `stafffns.blue`）→ staff_common.lua 的 `fns.blue` → `SetOnAttack(onattack_blue)`，原版 icestaff2／3 分别使用自己的注册名，未被此例修改。积累会衰减，目标可有额外抗性／重定向／SG 门禁，不保证每次都按固定击数冻结。[工厂](../../../raw/current/scripts/prefabs/staff.lua#L149-L172) [blue 入口](../../../raw/current/scripts/prefabs/staff.lua#L174-L175) [注册](../../../raw/current/scripts/prefabs/staff.lua#L301-L303)

### 新宿主的原版参考

- 猪人用 `MakeMediumFreezableCharacter(inst, "pig_torso")`：helper 加组件、设碎冰与基础抗性 2；SG／Brain 由工厂另外设置，不能省略。[standardcomponents.lua:344](../../../raw/current/scripts/standardcomponents.lua#L344
- 原玩家用 MakeLarge，再覆盖抗性为 4 和玩家默认时长；不能按 helper 名称推定最终参数。[player_common.lua:2712](../../../raw/current/scripts/prefabs/player_common.lua#L2712
- helper 不负责声明全部 Prefab／动画资源，新生物需核对实际工厂资源与 SG，当前不提供“任意生物只加组件即可冻结”的 demo。

## 常见陷阱与限制

- **负数 AddColdness 就是解冻**：负数只降低积累，不切 state，也不立即取消现存 task；明确解除用 Unfreeze。即使减到 0，也不能只看 coldness 判断是否冻结。[freezable.lua:141](../../../raw/current/scripts/components/freezable.lua#L141
- **把 Reset 或删组件当解冻**：它们不完成正常退出；SG 与 Brain 可能仍保留冻结状态。生命周期处理先核对上面的状态／清理顺序，不只删除标签。
- **GetTimeToWearOff 就是总控制剩余**：只给当前 task，FROZEN 后还有 THAWING；同帧多个状态变化还需考虑 SG 排队，CommonStates 会在入状态时重新对照组件。[commonstates.lua:1290](../../../raw/current/scripts/stategraphs/commonstates.lua#L1290
- **冻结进度和时长都会完整存档**：OnSave 只保存正 extraresist、向下量化到 0.1；OnLoad 只重建额外抗性与其衰减 task。coldness、冻结 state、阶段剩余与自定义基础配置不在该组件保存中，持久配置应由工厂／自己的状态恢复流程提供。[freezable.lua:321](../../../raw/current/scripts/components/freezable.lua#L321
- **加组件就自动显示完整冻结**：组件 tint 与 SG 动画是两层。tint 优先用 colouradder，否则直接改 AnimState.AddColour；其他染色、动画资源和自定义 SG 协作需单独核验。[freezable.lua:174](../../../raw/current/scripts/components/freezable.lua#L174

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 7 处；结论修订：icestaff 的 onattack_blue 回调与耗冷消费证据随 P-112 法杖重构迁至 prefabs/staff_common.lua（行为未变，工厂字段 `icestaff_coldness` 仍由 staff.lua `blue_common` 设置），注册链描述同步更新；依据：全局枢纽 diff 审阅（entityscript 本版改动仅湿润查询／距离函数，不涉本页冻结路径）＋变更清单命中 staff.lua、pigman.lua、player_common.lua、commonstates.lua、entityscript.lua。
- 源码审阅：2026-10-16，Steam buildid 25643519；核验积累／门禁、两段计时、额外抗性、受击、清理与保存；猪人 helper／SG 配合、原玩家参数和冰杖注册／消费者。既有体温／保温／战斗结论复用所链接页面。
- 结构检查：元数据、来源路径、大小、链接及唯一叶级登记通过；两个独立 modmain 示例经 Full Moon 3.0.0／Lua 5.1 解析与原字节往返检查。所选冰杖接入与积累流程章节可完整读取，检查不代表游戏运行通过。
- 游戏运行：未运行；连续／间隔命中、解冻阶段重冻、受击打碎、死亡／变身／读档、客户端动画和模组兼容待实测。未审校全部 SG、Boss 抗性策略或新增生物完整网络工厂。

相关：[[reference/components/temperature]]；[[reference/components/insulator]]；[[reference/components/weapon]]；[[reference/components/combat]]。
