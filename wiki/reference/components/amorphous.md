---
title: "amorphous"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "按容器物品标签和形态优先级选择变形，协调进入／退出回调、开箱延后检查与形态名称存读。"
aliases: ["amorphous", "容器物品触发变形", "哈奇形态", "物品标签选择形态", "形态优先级"]
tags: [component, lifecycle, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/amorphous.lua", "components/container.lua", "prefabs/hutch.lua", "stategraphs/SGchester.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# amorphous

## 快速判断

- 职责与适用：为宿主维护有序forms表，按容器物品标签选择一个形态并调用回调；不换Prefab、不自动消耗材料，也不提供柔性碰撞。原版哈奇在服务端安装，视觉与能力切换由宿主回调及SG完成。[amorphous.lua:11-19](../../../raw/current/scripts/components/amorphous.lua#L11-L19) [amorphous.lua:71-109](../../../raw/current/scripts/components/amorphous.lua#L71-L109) [hutch.lua:395-399](../../../raw/current/scripts/prefabs/hutch.lua#L395-L399) [hutch.lua:453-467](../../../raw/current/scripts/prefabs/hutch.lua#L453-L467)
- 优先使用：容器内存在特定标签物品后改变同一实体的表现／能力，且需要优先级与默认形态时，可参照哈奇forms注册；初始形态须由宿主明确建立。[hutch.lua:286-323](../../../raw/current/scripts/prefabs/hutch.lua#L286-L323) [hutch.lua:458-462](../../../raw/current/scripts/prefabs/hutch.lua#L458-L462)
- 重要副作用：切换先执行旧形态exit，赋currentform，再执行新形态enter；同一个form表不重复切换。GetCurrentForm返回名称，而不是回调实际完成的视觉状态。[amorphous.lua:55-81](../../../raw/current/scripts/components/amorphous.lua#L55-L81) [hutch.lua:303-310](../../../raw/current/scripts/prefabs/hutch.lua#L303-L310)
- 修改边界：自动CheckForMorph在开箱或宿主死亡时不变形，直接MorphToForm绕过这些门槛；组件本身不含网络变量，原版按服务端接线改变表现。[amorphous.lua:71-89](../../../raw/current/scripts/components/amorphous.lua#L71-L89) [hutch.lua:395-399](../../../raw/current/scripts/prefabs/hutch.lua#L395-L399)

## 关键接口

| 接口／字段 | 签名或约束 | 条件、作用与证据 |
|---|---|---|
| `AddForm(form)` | 按调用顺序追加表 | 不去重、排序或立即检查；非最后候选通常提供 `name`、`itemtags`列表和可选进入／退出函数，末项是无条件兜底。[amorphous.lua:59-61](../../../raw/current/scripts/components/amorphous.lua#L59-L61) [amorphous.lua:83-109](../../../raw/current/scripts/components/amorphous.lua#L83-L109) |
| `FindForm(name)` | 第一个同名form表或nil | 线性按顺序找名称；重复名称会使读档恢复到第一个匹配项，不能靠名称区分重复表。[amorphous.lua:63-69](../../../raw/current/scripts/components/amorphous.lua#L63-L69) |
| `GetCurrentForm()` | 当前形态name或nil | 构造currentform=nil；返回当前表的name，不校验其仍在forms表内。[amorphous.lua:13-14](../../../raw/current/scripts/components/amorphous.lua#L13-L14) [amorphous.lua:55-57](../../../raw/current/scripts/components/amorphous.lua#L55-L57) |
| `MorphToForm(form,instant)` | 形态表；instant透传给回调 | 比较表身份，不按name；不验证form属于forms、容器条件或宿主生命状态。调用旧 `exitformfn(inst,instant)`，更新当前表，再调用新 `enterformfn(inst,instant)`。[amorphous.lua:71-81](../../../raw/current/scripts/components/amorphous.lua#L71-L81) |
| `CheckForMorph()` | 无参数；自动事件检查入口 | 无形态、容器打开或已死时返回；有容器时按顺序匹配非末项，首个全标签满足者胜出，否则取最后项；自动调用instant=false。[amorphous.lua:83-109](../../../raw/current/scripts/components/amorphous.lua#L83-L109) |
| `LoadPostPass()` | 加载后监听接线 | 注册itemget／itemlose；仅POPULATING为真时主动CheckForMorph。非加载构造会自行调用此方法注册监听，但不因此立即选初始形态。[amorphous.lua:11-19](../../../raw/current/scripts/components/amorphous.lua#L11-L19) [amorphous.lua:47-53](../../../raw/current/scripts/components/amorphous.lua#L47-L53) |

## 关键流程

### 标签匹配与默认形态

forms最后一项永远是兜底，不检查它的itemtags；只有形态数大于1且宿主有container，才逐个检查前面的候选。每个候选的itemtags中，每种标签分别调用一次 `container:FindItem(item:HasTag(tag))`；各标签可以由不同物品满足，也可由同一个多标签物品满足。它不要求独立物品数量、不计堆叠数或耐久、不消费物品；相同标签重复写也不表示需要多件。[amorphous.lua:83-109](../../../raw/current/scripts/components/amorphous.lua#L83-L109) [container.lua:763-769](../../../raw/current/scripts/components/container.lua#L763-L769)

非末候选itemtags为空表时会无条件满足；漏写itemtags则在迭代时无防御。没有容器时直接选末项。需要所有材料类型同时存在时用标签列表；需要消耗、数量或复杂条件时不能靠这份接口推导，应另核验实际宿主机制。[amorphous.lua:91-109](../../../raw/current/scripts/components/amorphous.lua#L91-L109)

### 容器事件与延后变形

构造立即监听宿主onclose；itemget／itemlose在LoadPostPass注册，回调只在容器未打开时检查。CheckForMorph自身再次拒绝打开状态，container.IsOpen以opencount>0为准，所以一个使用者关闭但仍有人打开时也不切换；最后关闭后的事件才允许按当时物品重新选形态。[amorphous.lua:1-19](../../../raw/current/scripts/components/amorphous.lua#L1-L19) [amorphous.lua:47-53](../../../raw/current/scripts/components/amorphous.lua#L47-L53) [amorphous.lua:83-89](../../../raw/current/scripts/components/amorphous.lua#L83-L89) [container.lua:719-725](../../../raw/current/scripts/components/container.lua#L719-L725)

这里没有轮询、计时或“物品标签变化”事件监听；同一物品在原槽内变化标签不会仅凭本组件自动触发重新选择，需要后续已有事件或显式CheckForMorph。直接MorphToForm不受上述检查限制，instant也只是回调参数，组件不会自行播放即时／延迟动画。[amorphous.lua:11-25](../../../raw/current/scripts/components/amorphous.lua#L11-L25) [amorphous.lua:47-81](../../../raw/current/scripts/components/amorphous.lua#L47-L81)

### 保存、恢复与删除

仅当前形态非nil且不是forms末项时，OnSave返回 `{form=当前name}`；默认末项不存。OnLoad按name找形态，找到便 MorphToForm(form,true)，因此直接走瞬时回调且不检查容器／死亡；找不到仅打印提示。LoadPostPass在加载期再按实际容器内容CheckForMorph，保存名称不是恢复库存材料或保证永久锁定形态。[amorphous.lua:27-53](../../../raw/current/scripts/components/amorphous.lua#L27-L53)

形态定义、注册顺序、回调及回调附带的能力状态不由本组件保存，宿主须在读档前构造相同的forms。OnRemoveFromEntity只撤onclose、itemget、itemlose监听，不调用旧形态exit，也不恢复默认build、能力或其他副作用。[amorphous.lua:21-45](../../../raw/current/scripts/components/amorphous.lua#L21-L45) [amorphous.lua:71-81](../../../raw/current/scripts/components/amorphous.lua#L71-L81)

## 典型接入

### 哈奇：优先河豚、再音乐、最后普通

原版服务端哈奇将三形态依次AddForm：FUGU要求lightbattery和pointy，MUSIC要求lightbattery和band，NORMAL为末项。两个特殊条件同时满足时先选FUGU；普通形态的itemtags可以为nil，因为它只作兜底。注册后宿主显式MorphToForm(末项,true)，说明AddForm不会替它完成初始形态建立。[hutch.lua:314-323](../../../raw/current/scripts/prefabs/hutch.lua#L314-L323) [hutch.lua:453-467](../../../raw/current/scripts/prefabs/hutch.lua#L453-L467)

哈奇CreateForm的enter回调在instant=true时立即设置build、图标、光照并CheckBattery；非instant时只发 `{morphfn=enterfn}` 的morph事件。SGchester接事件进入morph状态，第40帧执行实际enterfn；若状态提前退出且尚未执行，也补执行该函数。故组件currentform已更新时，模型变化可能仍在等待动画，不能把GetCurrentForm当动画完成通知。[hutch.lua:286-312](../../../raw/current/scripts/prefabs/hutch.lua#L286-L312) [SGchester.lua:89-91](../../../raw/current/scripts/stategraphs/SGchester.lua#L89-L91) [SGchester.lua:332-395](../../../raw/current/scripts/stategraphs/SGchester.lua#L332-L395)

CheckBattery由哈奇另行维护：按当前形态和有效电池条件开关发光、反伤或音乐效果；这些能力、耗用与移除不是amorphous内置行为。反伤接线可复用 [[reference/components/damagereflect#典型接入]]。这是同实体变形的原版接线参照；自定义宿主需提供资源与enter／exit恢复协议，本页不造只注册forms的完整demo。[hutch.lua:191-259](../../../raw/current/scripts/prefabs/hutch.lua#L191-L259) [hutch.lua:338-344](../../../raw/current/scripts/prefabs/hutch.lua#L338-L344)

## 常见陷阱与限制

- **相同name就不会重复变形：** MorphToForm比较表身份；不同表即使同名仍会exit／enter，FindForm却只找首个同名表。需要稳定读档身份时保持名称唯一，并复用已登记表。[amorphous.lua:63-81](../../../raw/current/scripts/components/amorphous.lua#L63-L81)
- **AddForm会立即检查、自动建默认形态：** 它只追加；构造非POPULATING时的LoadPostPass也仅接监听。参照哈奇显式建立初态，后续再由事件检查。[amorphous.lua:11-19](../../../raw/current/scripts/components/amorphous.lua#L11-L19) [amorphous.lua:47-61](../../../raw/current/scripts/components/amorphous.lua#L47-L61) [hutch.lua:458-462](../../../raw/current/scripts/prefabs/hutch.lua#L458-L462)
- **组件删除会撤销形态效果：** 删除只撤监听；宿主若要运行时卸载，必须自行处理当前形态exit及外部副作用，不能依赖组件自动退回末项。[amorphous.lua:21-25](../../../raw/current/scripts/components/amorphous.lua#L21-L25)

## 验证记录

- 2026-10-26，`steam-buildid:26148036`：审阅组件全文、container.FindItem／IsOpen及关闭事件、哈奇服务端注册／初态／完整CreateForm与CheckBattery／加载后监听、SGchester morph事件与完整状态，覆盖优先级、标签存在性、回调时序、开箱门槛与存读／移除边界。
- 游戏运行：未运行；未测开箱多人时序、变形动画打断、读档重新选态或自定义重复名称／回调重入。未展开哈奇全部电池耗用与宿主生命周期。
- 结构检查：`validate_page.py`通过；两页合并执行`validate_source_links.py --strict`，92个行号引用、0错误／0警告。本页无新增Lua示例；机械检查不代替游戏测试。

- 2026-10-28 build 26230496：SGchester 旧自制 knockback 态删除改接 commonstates 击退框架（SB-10），morph 事件与状态锚点重定位，变形行为未变。
相关：[[reference/components/container]] · [[reference/components/damagereflect]]
