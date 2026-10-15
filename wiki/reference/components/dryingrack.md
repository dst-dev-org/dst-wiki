---
title: "dryingrack"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "用容器格子管理晾晒计时、雨天暂停、保鲜与完成替换，并恢复各格剩余时间。"
aliases: ["晾晒架", "雨天继续晾晒", "晾晒暂停", "dryingrack"]
tags: [component, building, food, lifecycle]
dst_build: "steam-buildid:26230496"
sources: ["components/dryingrack.lua", "components/dryable.lua", "components/rainimmunity.lua", "prefabs/meatrack2.lua", "containers.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# dryingrack

## 快速判断

- 职责与适用：宿主容器中各件原料的晾晒与显示；原料产物／时间来自 [[reference/components/dryable]]。本 build 的 meatrack 是三格容器，不再按旧单挂点实现理解。[容器](../../../raw/current/scripts/containers.lua#L213-L235)／[注册](../../../raw/current/scripts/prefabs/meatrack2.lua#L584)
- 优先使用：先装 container，再装 dryingrack 并 EnableDrying；保留容器、显示回调和天气管理。要防雨优先接 rainimmunity，不反复强行 ResumeDrying。[原版装配](../../../raw/current/scripts/prefabs/meatrack2.lua#L325-L334)／[免雨响应](../../../raw/current/scripts/components/dryingrack.lua#L109-L124)
- 重要副作用：Enable 添加 preserver 并配置暴露，Disable 移除它并暂停；正在实际晾晒的原料不腐败，暂停时不享有该零倍率。[初始化倍率](../../../raw/current/scripts/components/dryingrack.lua#L26-L38)／[启用](../../../raw/current/scripts/components/dryingrack.lua#L96-L104)／[禁用](../../../raw/current/scripts/components/dryingrack.lua#L168-L194)
- 修改边界：服务端；容器默认必须已存在。它拥有 itemget／itemlose 与计时协作，勿用同槽多套自建任务；保鲜规则见 [[reference/components/preserver]]。[构造](../../../raw/current/scripts/components/dryingrack.lua#L4-L23)

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `EnableDrying()`／`DisableDrying()` | 启用／停用 | 管天气监听、暴露／保鲜与暂停；Enable 会按当前天气／免雨条件决定恢复。[启用](../../../raw/current/scripts/components/dryingrack.lua#L96)／[初始天气](../../../raw/current/scripts/components/dryingrack.lua#L157-L164)／[禁用](../../../raw/current/scripts/components/dryingrack.lua#L168) |
| `PauseDrying()`／`ResumeDrying()` | 任务与剩余秒数互换 | Resume 不自行核验 enabled／天气；不要用它长期绕过天气控制。[暂停](../../../raw/current/scripts/components/dryingrack.lua#L347)／[恢复](../../../raw/current/scripts/components/dryingrack.lua#L362) |
| `SetShowItemFn(fn)`／`SetHideItemFn(fn)` | 显示 `(rack, slot, prefab, build)`；隐藏 `(rack, slot, prefab)` | 原版宿主负责覆盖动画符号，组件只发回调。[设置](../../../raw/current/scripts/components/dryingrack.lua#L54-L60)／[显示](../../../raw/current/scripts/components/dryingrack.lua#L264-L268)／[隐藏](../../../raw/current/scripts/components/dryingrack.lua#L313-L315) |
| `GetItemInSlot(slot)` | 返回 item、显示 Prefab、build | 复用当前晾晒／完成显示信息，不等于只读 container 的实体。[实现](../../../raw/current/scripts/components/dryingrack.lua#L62-L75) |
| `GetDryingInfoSnapshot()`／`ApplyDryingInfoSnapshot(snapshot)` | 以物品实体为键的剩余时间／build 表 | 作用于现有晾晒信息；不是永久速度倍率，空快照可能为 nil，不可直接无条件传回。[取快照](../../../raw/current/scripts/components/dryingrack.lua#L488-L496)／[应用](../../../raw/current/scripts/components/dryingrack.lua#L499) |

## 典型接入

### 普通晾晒架在雨天也能继续工作

目标模组 `modmain.lua`，复用 meatrack 的容器／资源／显示；只在服务端添加永久自来源免雨。rainimmunity 构造时发 gainrainimmunity，rack 已监听并恢复晾晒；AddSource(inst) 保持该来源。[免雨事件及来源](../../../raw/current/scripts/components/rainimmunity.lua#L14-L17)／[来源](../../../raw/current/scripts/components/rainimmunity.lua#L32-L38)／[响应](../../../raw/current/scripts/components/dryingrack.lua#L109-L124)／[PostInit](../../../raw/current/scripts/modutil.lua#L593)

```lua
AddPrefabPostInit("meatrack", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.dryingrack == nil then return end
    if inst.components.rainimmunity == nil then
        inst:AddComponent("rainimmunity")
    end
    inst.components.rainimmunity:AddSource(inst)
end)
```

不新增定时器、不替换显示回调。免雨同时影响该实体的相关免雨消费者，不是只修改晾晒倍率；停用后新建／读档工厂不再授予本来源。若同实体其他功能也使用 self 作为来源键，不能把 RemoveSource(inst) 当作本例独占撤销。烧毁／禁用仍走宿主原有流程；未验证酸雨与其他模组组合，不承诺整栋防腐或防火。

## 生命周期

itemget 读取原料配置并记录任务；itemlose 取消任务、临时携带剩余时间，重入时夹在至少 10 秒与配置上限之间；堆叠变化／掉落会清除这类转移缓存。[入架](../../../raw/current/scripts/components/dryingrack.lua#L236-L261)／[移出](../../../raw/current/scripts/components/dryingrack.lua#L280-L290)／[清理](../../../raw/current/scripts/components/dryingrack.lua#L226-L233)

完成生成产物，继承湿度，删除输入并放回同槽；不复制输入堆叠数量，也没有 cookable 的新鲜度换算。按格保存剩余秒数或显示 build；普通同宿主容器要待内容恢复后再装回晾晒信息。[完成](../../../raw/current/scripts/components/dryingrack.lua#L198-L215)／[保存](../../../raw/current/scripts/components/dryingrack.lua#L422-L432)／[恢复](../../../raw/current/scripts/components/dryingrack.lua#L444-L458)

## 常见陷阱与限制

- **挂组件顺序随意：** 构造即访问 container，原版明确要求先装 container。Enable／Disable 还增删 preserver，勿在同宿主上另设独立保鲜管理而期待两者自动合并。[构造](../../../raw/current/scripts/components/dryingrack.lua#L4-L10)／[原版注释](../../../raw/current/scripts/prefabs/meatrack2.lua#L325-L334)
- **暂停就所有时间不走：** 实时任务暂停会保存剩余值，但 enabled 下 LongUpdate 同时推进活动任务和暂停记录，可能完成并把剩余离屏时间传给产物；不能据 Pause 推断全部离屏语义。[LongUpdate](../../../raw/current/scripts/components/dryingrack.lua#L393-L416)
- **放入整堆等量加工：** 原版 acceptsstacks=false，完成移除输入实体后只 Spawn 一次，改容器接收规则须同时设计数量与存档。[容器](../../../raw/current/scripts/containers.lua#L224)／[完成](../../../raw/current/scripts/components/dryingrack.lua#L198-L211)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 0 处——本页全部引用文件（dryingrack／dryable／rainimmunity／meatrack2／modutil）未变，containers.lua 本版改动在 2299 行之后的 socket_keystone 段，213-235／224 容器布局行未动，逐处内容核对一致；无结论修订。依据：变更清单命中 containers.lua＋组内共享 diff 复用。
- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖普通架装配／三格注册、晾晒与天气／免雨、保鲜协作、任务转移／保存／LongUpdate 及上述免雨示例；不代表骑乘外置架、盐架与所有损毁分支已审校。
- 结构／示例检查：本批格式、来源文件、链接／行锚点、导航登记及完整 Lua 5.1 示例语法检查通过。
- 游戏运行：未运行；降雨切换、载入时下雨、掉落／转架、离屏、烧毁与来源兼容需验证。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。