# 船体附属、水域生存与收纳扩展组件检视

2026-10-21 完成第 102～106 组，共 13 篇组件页，依据 Steam buildid 25643519。五个子代理分组核验，主代理复核、覆盖登记与公共文件收尾；源码只读，未运行游戏。

## 分工与交付

| 组 | 子代理范围 | 页面数量 |
|---|---|---:|
| 102：船体附属 | boatrotator、boatringdata、hull | 3 |
| 103：溺水与浮具 | drownable、flotationdevice | 2 |
| 104：渔网 | fishingnet、fishingnetvisualizer | 2 |
| 105：收纳扩展 | shelf、counter、selfstacker | 3 |
| 106：雨棚与闷熄 | raindome、raindomewatcher、smotherer | 3 |

| 页面 | 行数／UTF-8 字节 | 已核验宿主与接入 |
|---|---|---|
| [boatrotator](../../../wiki/reference/components/boatrotator.md) | 84／11,278 | boat_rotator、boat_rotator_kit、boat_rotator_kit_placer |
| [boatringdata](../../../wiki/reference/components/boatringdata.md) | 86／11,419 | boat 系与 dragonboat_body／shadowboat |
| [hull](../../../wiki/reference/components/hull.md) | 81／11,236 | 同上 9 个注册名 |
| [drownable](../../../wiki/reference/components/drownable.md) | 106／14,300 | player_common（各角色）、beefalo |
| [flotationdevice](../../../wiki/reference/components/flotationdevice.md) | 77／7,500 | balloonvest（BODY 槽） |
| [fishingnet](../../../wiki/reference/components/fishingnet.md) | 80／10,460 | fishingnet |
| [fishingnetvisualizer](../../../wiki/reference/components/fishingnetvisualizer.md) | 97／14,181 | fishingnetvisualizer（无 AddNetwork） |
| [shelf](../../../wiki/reference/components/shelf.md) | 94／12,478 | birdcage、yotc_carrat_gym_*、lureplant 等 |
| [counter](../../../wiki/reference/components/counter.md) | 92／11,018 | player_common（各角色）、salty_dog、wx78 无人机 |
| [selfstacker](../../../wiki/reference/components/selfstacker.md) | 95／13,086 | houndstooth、stinger、humanmeat、monstermeat、barnacle、spoiled_food |
| [raindome](../../../wiki/reference/components/raindome.md) | 96／13,234 | voidcloth_umbrella |
| [raindomewatcher](../../../wiki/reference/components/raindomewatcher.md) | 86／11,467 | player_common（本地激活玩家） |
| [smotherer](../../../wiki/reference/components/smotherer.md) | 83／12,149 | poop／compost／guano／fertilizer／ice／snowball_item 等 11 名 |

主代理修正 boatrotator 示例的 `GLOBAL` 前缀与 print 调试行；13 页均在 180 行／14 KiB 以内。

## 语义核验与重要边界

- boatrotator 的 `SetBoat` 解绑时把新船参数传给 `RemoveEventCallback`，旧船监听残留；手动换船后旧船死亡会移除当前船 rotator。`SetBoat(nil)` 时参数回退成 `self`。
- boatringdata 只有 `_isrotating` 是 net_bool，`radius`／`segments` 是普通字段；本快照没有 `RegisterPlatform`，平台注册在 walkableplatform。
- hull 只管船唇／跳板挂接与半径，不含生命、漏水或沉没；不要与 hullhealth 混用。
- drownable 只做“位置→原因→事件”与冲岸、掉落、伤害；动画与状态归 SG。浮具只在 `TakeDrowningDamage` 里查，湿度先结算。
- flotationdevice 是标记＋回调，不检测、不阻止下沉；宿主 balloonvest 提供 `onpreventdrowningdamagefn`。
- fishingnet 的 `CastNet` 只生成 visualizer 并 `BeginCast`，耐久与动作条件在宿主；visualizer prefab 无 `AddNetwork`，客户端不可见，且承担捕获执行。
- shelf 的 `takeitemtstfn` 无 setter，构造字段 `onitemtakenfn` 与实际读取的 `ontakeitemfn` 命名不一致；`cantakeitem=false` 时 TAKEITEM 仍可能成功返回。
- counter 无事件、无 replica，是消费者拉取的被动提供者；玩家宿主存在但 scripts 内无原版消费者。
- selfstacker 只在 `OnEntityWake` 触发，`DoStack` 不复查条件；`RemoveComponent` 不清理标签与待执行任务，标签与组件必须成对。
- raindome 的 `OnRemoveFromEntity` 直接 `assert(false)`，运行期不可 `RemoveComponent`；客户端没有 `.radius`。
- raindomewatcher 只加在本地激活玩家上，每帧查询并推 enter／under／exit；事件不跨网络，移除不补发 `exitraindome`。
- smotherer 是空动作标记（组件无方法），让可燃烧物被闷熄；消耗在 `burnable`，空手路径另走 3 点火焰伤害。

## 验证与限制

- 13 页 `validate_page.py` 通过（0 issue）；`validate_source_links.py --strict` 590 处引用、0 error／0 warning。
- 子代理语法检查的示例均通过、未执行；主代理只改了 boatrotator 的 GLOBAL 与 print 行，未重跑 syntax-only。
- 全部 `source-reviewed`、未运行游戏；多人同步、读档回归与实机表现未覆盖。

## 登记与收尾

- 覆盖 CSV：13 行改为 built（批次 102～106），合并后 238 组件／560 未建。
- 索引：world-ships 增至 13 条（加 boatrotator／boatringdata／hull／drownable／flotationdevice）；items 垂钓加 fishingnet／fishingnetvisualizer、容器加 shelf／counter／selfstacker；character 温度与环境接管 rainimmunity／lightningblocker 并加 raindome／raindomewatcher；world 燃烧加 smotherer。
- PLAN／README／design 同步 238／560／246／13；日志追加一条。
