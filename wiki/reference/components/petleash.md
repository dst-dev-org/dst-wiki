---
title: "petleash"
created: "2026-10-19"
updated: "2026-10-28"
type: component
summary: "主人侧管理宠物集合、两套容量、生成／接管及嵌套存读；解除登记与跟随／销毁分开。"
aliases: ["威尔逊普通宠物容量改为二", "宠物生成与接管", "petleash"]
tags: [component, creature, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/petleash.lua", "prefabs/player_common.lua", "prefabs/critters.lua", "actions.lua", "prefabs/walter.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# petleash

## 快速判断

- 职责与适用：装在主人，维护 pets 集合、普通数量及按 Prefab 限额的数量，提供生成／接管和内嵌保存；有 leader 时另登记随从，实际行为仍依赖宠物宿主。[关联](../../../raw/current/scripts/components/petleash.lua#L160-L173) [保存](../../../raw/current/scripts/components/petleash.lua#L274-L307)
- 优先使用：SpawnPetAt 生成，AttachPet 接管已有实体，DespawnPet 请求退场；GetPets 得到的是内部集合，不直接改表代替接口。[生成](../../../raw/current/scripts/components/petleash.lua#L175-L225) [集合](../../../raw/current/scripts/components/petleash.lua#L152-L158)
- 重要副作用：关联设置 pet.persists=false 并监听宠物 onremove；有 leader 时 AddFollower。普通数量与专用 Prefab 数量互斥计数，不把 GetNumPets 当全体数量。[关联顺序](../../../raw/current/scripts/components/petleash.lua#L160-L173) [删除扣数](../../../raw/current/scripts/components/petleash.lua#L15-L27)
- 修改边界：组件不补 Brain／动画／资源，也不自动提供客户端领养入口；容量、特殊角色政策、动作和配方门禁须分别核验。原沃尔特将普通容量设为 0。[角色例外](../../../raw/current/scripts/prefabs/walter.lua#L1192)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `SetPetPrefab(prefab)`／`SetMaxPets(n)` | 默认产物 nil、普通上限 1；SetMaxPets 只写上限，不消灭超额宠物，也不触发 UI 更新。[默认](../../../raw/current/scripts/components/petleash.lua#L1-L13) [方法](../../../raw/current/scripts/components/petleash.lua#L31-L60) |
| `SetMaxPetsForPrefab(prefab,n)` | 将该 Prefab 列为独立限额类别；不要在已有宠物存活时随意切换类别，计数没有自动迁移。IsFullForPrefab 包含 reserved 数量，普通 IsFull 不包含。[限额](../../../raw/current/scripts/components/petleash.lua#L63-L92) [预留](../../../raw/current/scripts/components/petleash.lua#L95-L140) |
| `SpawnPetAt(x,y,z,prefaboverride,skin)` | override 优先，否则默认；按所属计数检查容量，SpawnPrefab 成功后关联、定位、onspawnfn(inst,pet)，返回实体／nil。预留不会自动在此释放。[流程](../../../raw/current/scripts/components/petleash.lua#L175-L205) |
| `AttachPet(pet)` | 已登记或满额 false；成功只 LinkPet 后 true，不调用 onspawnfn。须有效实体，不能当跨主人的完整移交协议。[接管](../../../raw/current/scripts/components/petleash.lua#L208-L225) |
| `DetachPet(pet)` | 去集合、扣计数并移除 onremove 监听；不调用 onpetremoved，不恢复 persists，也不解除 leader 跟随／移除实体。[解除](../../../raw/current/scripts/components/petleash.lua#L228-L240) |
| 三种回调 | SetOnSpawnFn、SetOnDespawnFn、SetOnRemovedFn 均 fn(owner,pet)；第三者只在已登记宠物真实 onremove 时调用。自定义退场回调必须完成自己的删除／解除，否则集合仍占容量。[登记](../../../raw/current/scripts/components/petleash.lua#L35-L45) [移除](../../../raw/current/scripts/components/petleash.lua#L15-L27) [退场](../../../raw/current/scripts/components/petleash.lua#L242-L260) |
| 查询 | GetPets 返回实体→实体内部表；GetPetsWithPrefab 先看该 Prefab 专用计数，普通宠物未必能用它取到。按整体集合的 pet.prefab 筛选须自己只读遍历。[查询](../../../raw/current/scripts/components/petleash.lua#L103-L119) [集合](../../../raw/current/scripts/components/petleash.lua#L143-L158) |

保存所有登记宠物的实体记录，读档直接 LinkPet 并执行 onspawnfn，不重新走当前容量检查；降低上限不保证旧存档收敛。预留、配置与回调未独立保存，需宿主重建。移除组件只撤销宠物监听，删除宿主则调用 DespawnAllPets；TransferComponent 当前为空实现，不提供完整转移。[存读／清理](../../../raw/current/scripts/components/petleash.lua#L274-L320)

## 典型接入

### 仅将威尔逊的普通宠物容量设为二

目标模组 `modmain.lua`，无新资源；修改已有 petleash 的普通上限，保留专用 Prefab 限额、原生成／退场回调和其他角色政策。

```lua
AddPrefabPostInit("wilson", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    local leash = inst.components.petleash
    if leash ~= nil then
        leash:SetMaxPets(2)
    end
end)
```

影响后续普通生成／接管，不自动产生宠物或修复所有配方／客户端提示。原玩家配置普通上限 1、独立 gestalt_guard_evolved 限额及生成／删除回调；原小宠物领养代理调用 SpawnPetAt。[玩家配置](../../../raw/current/scripts/prefabs/player_common.lua#L2806-L2810) [回调](../../../raw/current/scripts/prefabs/player_common.lua#L1514-L1527) [领养代理](../../../raw/current/scripts/prefabs/critters.lua#L296-L301)

配置每次构造应用，停用并重载恢复上限；已存的多只宠物仍可被 OnLoad 恢复，不自动删超额。跟随关系复用 [[reference/components/leader]]／[[reference/components/follower]]，不在此重复完整 AI 方案。

## 常见陷阱与限制

- **DespawnPet 一调用就释放名额**：自定义 ondespawnfn 可以延迟删除；计数通常等待宠物 onremove。不要同时手改数量和调用删除。[退场](../../../raw/current/scripts/components/petleash.lua#L242-L249) [删除监听](../../../raw/current/scripts/components/petleash.lua#L15-L27)
- **DetachPet＋AttachPet 等于所有权完整迁移**：还需原随从关系、宿主事件、失败回滚等政策。原转交寄居蟹动作另有目标容量、接管结果、抚摸／adopted_critter 和失败回接。[原动作](../../../raw/current/scripts/actions.lua#L6661-L6681)
- **降低 maxpets 会改读档数量／全部专用宠物**：两个容量池分开，恢复不检查容量；保留明确迁移策略，不借一次赋值声称完成存档清理。[分类](../../../raw/current/scripts/components/petleash.lua#L160-L173) [恢复](../../../raw/current/scripts/components/petleash.lua#L287-L307)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：sources 中漂移文件 actions.lua、player_common.lua、walter.lua（更新报告 R-01、P-83、P-128）——TRANSFER_CRITTER、玩家 petleash 配置与 walter 普通容量 0 均未变（P-83 为 Worm 消化音效 netvar、P-128 为 walter OnAttacked 豁免，页内无此二结论）；锚点重定位 3 处；未覆盖：运行验证。

- 源码审阅：2026-10-19，Steam buildid 25643519；覆盖完整组件、两类容量／预留、生成／接管／解除、回调／保存、玩家配置和领养／转交消费者。未全面审校迁移／角色重选、所有专用宠物、领养配方 UI 或 Brain。
- 结构检查：本批格式、来源、链接／行号、大小及登记通过；完整容量例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；第二只领养、独立限额、超额存读、延迟退场、转交失败与多模组覆盖待实测。
