---
title: "dryable"
created: "2026-10-17"
updated: "2026-10-28"
type: component
summary: "为可晾晒食材提供产物、所需秒数与架上显示 build，由晾晒架实际计时。"
aliases: ["肉干时间", "晾晒原料", "缩短晾晒", "dryable"]
tags: [component, food, item]
dst_build: "steam-buildid:26230496"
sources: ["components/dryable.lua", "components/dryingrack.lua", "prefabs/meats.lua", "containers.lua", "modutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# dryable

## 快速判断

- 职责与适用：挂在原料上，提供晾晒配置；真正入架／暂停／完成／恢复由 [[reference/components/dryingrack]] 处理。直接烤制用 [[reference/components/cookable]]。[配置](../../../raw/current/scripts/components/dryable.lua#L1-L9)／[消费](../../../raw/current/scripts/components/dryingrack.lua#L236-L261)
- 优先使用：`SetProduct`、`SetDryTime(秒)`；沿用原版资源时保留 build 配置。[设置](../../../raw/current/scripts/components/dryable.lua#L16-L45)
- 重要副作用：设置方法只赋字段，不建立任务；已入架的倒计时不会随 SetDryTime 自动改变。[时间赋值](../../../raw/current/scripts/components/dryable.lua#L24-L29)／[入架取值](../../../raw/current/scripts/components/dryingrack.lua#L248-L261)
- 修改边界：服务端配置；新增原料要在共同初始化声明 dryable 标签，并具备收纳能力，原版 rack 容器按标签筛选。[肉类工厂](../../../raw/current/scripts/prefabs/meats.lua#L161-L179)／[过滤](../../../raw/current/scripts/containers.lua#L234-L238)

## 关键接口

| 接口／字段 | 签名或约定 | 条件、作用与证据 |
|---|---|---|
| `SetProduct(name)`／`GetProduct()` | Prefab 名 | 单纯保存／返回；rack 完成时交给 SpawnPrefab，不像 cookable 支持函数选择。[设置](../../../raw/current/scripts/components/dryable.lua#L16-L21)／[生成](../../../raw/current/scripts/components/dryingrack.lua#L198-L211) |
| `SetDryTime(seconds)`／`GetDryTime()` | 秒数 | 默认 nil；与 product 都存在才创建晾晒信息，应给有效正数。[字段](../../../raw/current/scripts/components/dryable.lua#L4-L6)／[消费者](../../../raw/current/scripts/components/dryingrack.lua#L248-L261) |
| `SetBuildFile(name)`／`GetBuildFile()` | 架上原料的动画 build 名 | 不是图标或资源文件路径；rack 显示回调取得该值，缺省使用 meat_rack_food。[设置](../../../raw/current/scripts/components/dryable.lua#L32-L37)／[显示](../../../raw/current/scripts/components/dryingrack.lua#L264-L265) |
| `SetDriedBuildFile(name)`／`GetDriedBuildFile()` | 完成产物的架上 build | 未单独配置时回退 buildfile；资源仍需宿主／物品正确加载。[回退](../../../raw/current/scripts/components/dryable.lua#L40-L45)／[肉类组装](../../../raw/current/scripts/prefabs/meats.lua#L201-L206) |

## 典型接入

### 大肉放到晾晒架后只需两分钟

目标模组 `modmain.lua`；改原版 meat 的配置，保留 meat_dried 产物、动画和依赖，不复制晾晒任务。原版肉类按数据表挂 dryable，meat 注册名与工厂已核验。[装配](../../../raw/current/scripts/prefabs/meats.lua#L201-L206)／[大肉配置](../../../raw/current/scripts/prefabs/meats.lua#L384)／[注册](../../../raw/current/scripts/prefabs/meats.lua#L899)／[PostInit](../../../raw/current/scripts/modutil.lua#L593)

```lua
AddPrefabPostInit("meat", function(inst)
    if not GLOBAL.TheWorld.ismastersim then return end
    if inst.components.dryable ~= nil then
        inst.components.dryable:SetDryTime(120)
    end
end)
```

120 是可推进的晾晒秒数，不保证雨天后现实两分钟完成。新入架取新值；存档／移架已有剩余时间仍按 rack 的恢复路径处理，不自动重置为 120。停用后配置恢复，但已保存任务不保证重置；不要为提速重新手工触发 itemget。[移架剩余值](../../../raw/current/scripts/components/dryingrack.lua#L236-L261)／[存档](../../../raw/current/scripts/components/dryingrack.lua#L422-L432)

## 常见陷阱与限制

- **只有标签就能晾干：** 容器过滤只是第一层；rack 还需要实际 dryable 的 product／drytime。新增食材沿用 [[cookbook/new-item]] 的实体／收纳骨架，再在正确端配置本组件。[过滤](../../../raw/current/scripts/containers.lua#L234-L238)／[实际取值](../../../raw/current/scripts/components/dryingrack.lua#L247-L261)
- **直接晾晒整堆能得到等量产物：** 原版架子不收堆叠；完成逻辑移除整个输入实体并生成一个产物，不按输入件数复制。[容器](../../../raw/current/scripts/containers.lua#L224)／[完成](../../../raw/current/scripts/components/dryingrack.lua#L198-L211)
- **build 名就是任意新美术资源：** 设置 build 不加载资源、不保证包含所需符号；原版鱼肉使用不同 build，扩展要同时核验资源与显示回调。[鱼肉配置](../../../raw/current/scripts/prefabs/meats.lua#L501)／[肉类资源](../../../raw/current/scripts/prefabs/meats.lua#L10-L11)

## 验证记录

- 版本增量复核 26148036（2026-10-25）：锚点重定位 0 处——本页全部引用（dryable／dryingrack／meats／modutil 未变；containers.lua 本版改动在 2299 行之后的 socket_keystone 段，234-238／224 过滤与 acceptsstacks 行未动），逐处内容核对一致；无结论修订。依据：变更清单命中 containers.lua＋组内共享 diff 复用。
- 源码审阅：2026-10-17，Steam buildid 25643519；覆盖配置字段／回退、rack 取值与生成、肉类工厂、容器过滤与两分钟示例。未核验所有干制食物与美术符号。
- 结构／示例检查：本批格式、来源文件、链接／行锚点、导航登记及完整 Lua 5.1 示例语法检查通过。
- 游戏运行：未运行；雨天暂停、离屏／读档、跨架转移和美术显示需验证。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。