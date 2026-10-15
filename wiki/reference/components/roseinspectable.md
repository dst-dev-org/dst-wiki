---
title: "roseinspectable"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "定义玫瑰检查目标的残影生成资格、创建／激活回调和是否诱发使用者冷却，实际生成与流程由使用者组件完成。"
aliases: ["roseinspectable", "玫瑰眼镜目标", "残影创建回调", "强制残影冷却"]
tags: [component, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/roseinspectable.lua", "components/roseinspectableuser.lua", "prefabs/flower.lua", "prefabs/wormhole.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# roseinspectable

## 快速判断

- 职责与适用：装在被检查目标上，提供残影资格、创建／激活回调和冷却意向；不是普通检查文本，也不自行生成残影。对象生成与激活流程由 [[reference/components/roseinspectableuser]] 消费。[roseinspectable.lua:1-63](../../../raw/current/scripts/components/roseinspectable.lua#L1-L63) [roseinspectableuser.lua:63-72](../../../raw/current/scripts/components/roseinspectableuser.lua#L63-L72)
- 优先使用：改“某个目标被玫瑰检查后发生什么”时配置目标回调；装备的近距动作分派沿 [[reference/components/closeinspector]]，不要给目标挂使用者组件来替代。[roseinspectableuser.lua:177-194](../../../raw/current/scripts/components/roseinspectableuser.lua#L177-L194)
- 重要副作用：forcedinducedcooldown只要非nil就覆盖冷却回调，显式false也会覆盖；要恢复回调判定应设nil。默认未配置时不诱发冷却。[roseinspectable.lua:31-49](../../../raw/current/scripts/components/roseinspectable.lua#L31-L49)
- 修改边界：方法只委托回调，不检查装备、技能、目标合法性或当前冷却。原版花与虫洞均在服务端安装；本组件没有网络或存读钩子。[roseinspectable.lua:23-63](../../../raw/current/scripts/components/roseinspectable.lua#L23-L63) [flower.lua:161-165](../../../raw/current/scripts/prefabs/flower.lua#L161-L165) [wormhole.lua:148-150](../../../raw/current/scripts/prefabs/wormhole.lua#L148-L150)

## 关键接口

| 接口／字段 | 回调签名或返回 | 条件、作用与证据 |
|---|---|---|
| `SetCanResidueBeSpawnedBy(fn)`／`CanResidueBeSpawnedBy(doer)` | `fn(target,doer)`；无fn返回true | 生成资格回调原样返回；不在本组件内生成实体或再次查冷却。[roseinspectable.lua:19-29](../../../raw/current/scripts/components/roseinspectable.lua#L19-L29) |
| `SetForcedInduceCooldownOnActivate(value)` | 保存强制冷却值 | 非nil时优先返回该值；false表示强制不诱发，nil才放行冷却函数。[roseinspectable.lua:31-49](../../../raw/current/scripts/components/roseinspectable.lua#L31-L49) |
| `SetWillInduceCooldownOnActivate(fn)`／`WillInduceCooldownOnActivate(doer)` | `fn(target,doer)`；无强制值／fn返回false | 同一目标可在尝试生成残影和真正激活时被分别查询，不能假设回调只执行一次。[roseinspectable.lua:35-49](../../../raw/current/scripts/components/roseinspectable.lua#L35-L49) [roseinspectableuser.lua:39-50](../../../raw/current/scripts/components/roseinspectableuser.lua#L39-L50) [roseinspectableuser.lua:186-191](../../../raw/current/scripts/components/roseinspectableuser.lua#L186-L191) |
| `SetOnResidueCreated(fn)`／`HookupResidue(owner,residue)` | `fn(target,owner,residue)`；不返回动作成功值 | 使用者生成并设置残影owner后调用HookupResidue；本方法只调用已设置的fn，不保存或创建残影。[roseinspectable.lua:15-17](../../../raw/current/scripts/components/roseinspectable.lua#L15-L17) [roseinspectable.lua:51-55](../../../raw/current/scripts/components/roseinspectable.lua#L51-L55) [roseinspectableuser.lua:63-72](../../../raw/current/scripts/components/roseinspectableuser.lua#L63-L72) |
| `SetOnResidueActivated(fn)`／`DoRoseInspection(doer)` | `fn(target,doer)`；不传residue | 激活效果回调不等于创建回调；返回值不被DoRoseInspection传回，也不控制使用者是否开始冷却。[roseinspectable.lua:11-13](../../../raw/current/scripts/components/roseinspectable.lua#L11-L13) [roseinspectable.lua:57-61](../../../raw/current/scripts/components/roseinspectable.lua#L57-L61) [roseinspectableuser.lua:39-50](../../../raw/current/scripts/components/roseinspectableuser.lua#L39-L50) |

## 典型接入

### 花：创建时变玫瑰，激活后给物品并移除目标

原版花注册资格回调，要求检查者激活winona_charlie_2；创建回调把非玫瑰花改成玫瑰并播放特效。激活回调在仍是玫瑰且检查者有inventory时生成charlierose交给玩家，再移除花；注册时强制诱发冷却。创建和激活发生在不同阶段，不能把第一次近距检查理解为已完成领奖。[flower.lua:113-134](../../../raw/current/scripts/prefabs/flower.lua#L113-L134) [flower.lua:193-197](../../../raw/current/scripts/prefabs/flower.lua#L193-L197)

### 虫洞：创建回调配置残影的地图上下文

虫洞同样以winona_charlie_2作为生成资格，但这里只注册创建回调，把残影SetMapActionContext设为WORMHOLE；并未在此注册DoRoseInspection激活回调。地图动作属于后续消费者，不能从目标有本组件就推导“所有目标激活都执行同一种效果”。[wormhole.lua:83-93](../../../raw/current/scripts/prefabs/wormhole.lua#L83-L93) [wormhole.lua:177-179](../../../raw/current/scripts/prefabs/wormhole.lua#L177-L179)

上述为服务端原版接线参照。本页不提供独立demo；目标回调、使用者生成／激活和残影生命周期必须按各自职责协作，主流程见 [[reference/components/roseinspectableuser#关键流程]]。

## 常见陷阱与限制

- **DoRoseInspection会重新验证所有条件：** 它只调激活fn；公共门槛、生成资格和冷却由使用者执行。直接调用目标方法绕过这些门槛。[roseinspectable.lua:57-61](../../../raw/current/scripts/components/roseinspectable.lua#L57-L61) [roseinspectableuser.lua:163-195](../../../raw/current/scripts/components/roseinspectableuser.lua#L163-L195)
- **强制冷却是构造默认false：** 源码该行仅是注释，实际初始nil；nil／false在强制优先级上不同。修改回调前先核对是否仍有非nil强制值。[roseinspectable.lua:1-9](../../../raw/current/scripts/components/roseinspectable.lua#L1-L9) [roseinspectable.lua:39-49](../../../raw/current/scripts/components/roseinspectable.lua#L39-L49)
- **删组件会销毁残影或回滚创建效果：** 本组件无清理／存读钩子，也不持有残影引用；由目标／使用者和charlieresidue各自处理生命周期。[roseinspectable.lua:1-63](../../../raw/current/scripts/components/roseinspectable.lua#L1-L63)

## 验证记录

- 2026-10-26，`steam-buildid:26148036`：审阅组件全文、使用者生成与激活消费者、花与虫洞服务端注册及完整目标回调，覆盖签名、强制值优先级和创建／激活分工。
- 游戏运行：未运行；未测回调重入、目标移除时序或地图后续交互。残影／冷却详细生命周期在使用者页维护，未穷举所有目标。
- 结构检查：`validate_page.py`通过；两页合并执行`validate_source_links.py --strict`，78个行号引用、0错误／0警告。本页无新增Lua示例；机械检查不代替游戏测试。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。