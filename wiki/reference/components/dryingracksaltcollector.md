---
title: "dryingracksaltcollector"
created: "2026-10-24"
updated: "2026-10-28"
type: component
summary: "为珍珠多格晾架记录各槽待回收的盐，区分槽位状态、干货图标、盐实体与宿主的取出／损毁返还。"
aliases: ["dryingracksaltcollector", "晾架收盐", "海带干副产盐", "珍珠晾架盐", "槽位存盐"]
tags: [component, food, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/dryingracksaltcollector.lua", "components/driedsalticon.lua", "prefabs/meatrack2.lua", "prefabs/veggies.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# dryingracksaltcollector

## 快速判断

- 职责与适用：保存 `slots[slot]=true` 与总数量；每个槽最多记一份待收盐，没有计时、搜索、生成盐或干制逻辑。真正加工由 [[reference/components/dryingrack]] 承担。[完整状态与增删](../../../raw/current/scripts/components/dryingracksaltcollector.lua#L1-L34)
- 优先使用：已有 `meatrack_hermit_multi` 直接复用其 itemget／itemlose 与损毁流程；新宿主需要同时实现登记、取物返还、显示和加载后校正，不能仅 AddComponent 就自动产盐。[宿主接线](../../../raw/current/scripts/prefabs/meatrack2.lua#L550-L568) [注册](../../../raw/current/scripts/prefabs/meatrack2.lua#L591)
- 重要副作用：AddSalt／RemoveSalt 只在槽状态改变时更新总数并调用回调；回调负责显示，不意味着此时 Spawn 了 saltrock。原版状态显示还加上容器中已有的 saltrock 数量。[增删](../../../raw/current/scripts/components/dryingracksaltcollector.lua#L12-L34) [显示计算](../../../raw/current/scripts/prefabs/meatrack2.lua#L370-L385)
- 修改边界：原版仅服务端安装，组件没有网络变量或物品引用。干货盐图标由另一个双端 driedsalticon 同步；普通 meatrack 和单格旧珍珠架不会因名字相近自动拥有本组件。[服务端安装](../../../raw/current/scripts/prefabs/meatrack2.lua#L550-L555) [工厂变体](../../../raw/current/scripts/prefabs/meatrack2.lua#L584-L591) [图标变量](../../../raw/current/scripts/components/driedsalticon.lua#L28-L40)

## 关键接口

| 接口／字段 | 参数、返回及证据 |
|---|---|
| `AddSalt(slot)` | 未登记则设 true、numsalts+1、调回调并返回 true；已登记返回 false。没有容器容量／合法槽号检查，原版调用者使用已有容器槽。[实现](../../../raw/current/scripts/components/dryingracksaltcollector.lua#L12-L22) |
| `RemoveSalt(slot)` | 已登记才清键、numsalts-1、回调并返回 true；不存在返回 false。不生成或返还盐实体。[实现](../../../raw/current/scripts/components/dryingracksaltcollector.lua#L24-L34) |
| `HasSalt(slot?)` | slot 为真值时只查该键；不传或传 false 则查 numsalts>0。Lua中0是真值，会查槽0，不是“任意槽”。[实现](../../../raw/current/scripts/components/dryingracksaltcollector.lua#L36-L40) |
| `GetNumSalts()` | 仅返回待收槽数；不含容器内 saltrock 堆叠数量、不含已经取出的盐。[读取](../../../raw/current/scripts/components/dryingracksaltcollector.lua#L43-L44) [宿主另加实物](../../../raw/current/scripts/prefabs/meatrack2.lua#L370-L374) |
| `SetOnSaltChangedFn(fn)` | 单回调 `fn(rack, numsalts)`，设置时不立即刷新；Add／Remove实际变更后调用，非空存档加载后也会调用一次。[注册](../../../raw/current/scripts/components/dryingracksaltcollector.lua#L8-L9) [加载](../../../raw/current/scripts/components/dryingracksaltcollector.lua#L59-L68) |
| `OnSave()`／`OnLoad(data)` | 有盐时保存槽键数组；加载合并未登记键并增加总数，**不清旧 slots／numsalts**，非空数组处理完后回调一次。常规由新实例加载，不能当热替换全部槽状态的 setter。[存读](../../../raw/current/scripts/components/dryingracksaltcollector.lua#L47-L68) |

## 典型接入

### 珍珠多格架如何把海带干与盐分开保存

原版 `meatrack_hermit_multi` 先沿共同工厂安装 container 与 dryingrack，再装 collector、初始化盐显示级别、注册增删与损毁处理。不是旧式 [[reference/components/dryer]] 的一件产物记录。[先装容器／加工器](../../../raw/current/scripts/prefabs/meatrack2.lua#L325-L334) [盐模块接线](../../../raw/current/scripts/prefabs/meatrack2.lua#L550-L568)

1. **完成物品触发登记：** 海带干的公共工厂挂 driedsalticon，服务端配置 collects=true。架子收到 itemget 后，仅当物品有该组件且 collects 为真时 AddSalt(slot)；已有槽盐还会给带图标组件的物品显示盐覆盖层。不按名字字符串或单纯 food 标签判断产盐。[海带干](../../../raw/current/scripts/prefabs/veggies.lua#L779-L800) [注册名](../../../raw/current/scripts/prefabs/veggies.lua#L1093) [入槽处理](../../../raw/current/scripts/prefabs/meatrack2.lua#L395-L410)
2. **取出后才转为实物：** itemlose 时，仍有效的 prev_item 让宿主排一个零延时静态任务；任务再查槽。槽已空则移除盐记录并把 saltrock 放回该槽；槽被无 driedsalticon 的其他物品占用时，把盐掉在世界；仍由带图标物品占用则保留记录。[移出分流](../../../raw/current/scripts/prefabs/meatrack2.lua#L430-L442) [延后处理](../../../raw/current/scripts/prefabs/meatrack2.lua#L413-L427)
3. **输入被加工删除并非普通取走：** prev_item 已无效时直接 RemoveSalt，不排返还任务；加工替换过程再由新物品入槽决定下一份记录。不能无条件把所有 itemlose 当一次产盐。[有效性检查](../../../raw/current/scripts/prefabs/meatrack2.lua#L430-L438)
4. **图标跟随另有清理：** 带盐干货移出后监听自己的 onputininventory／ondropped，触发时隐藏覆盖图标并去监听；图标变化用 driedsalticon 的 net_bool，不是 collector 在客户端复制 slots。[监听与清理](../../../raw/current/scripts/prefabs/meatrack2.lua#L389-L405) [图标同步](../../../raw/current/scripts/components/driedsalticon.lua#L58-L75)

这是一套已有 Prefab／容器／显示协议的协作参照，不提供单挂组件即可收盐的完整新架子 demo。自定义原料的加工与干货显示入口复用 [[reference/components/dryable]]、[[reference/components/dryingrack]]，只在目标模组明确实现上述协议后再接集盐。

### 加载、损毁与搬迁必须一同处理

- **加载后校正：** 原版 OnLoadPostPass 遍历真实容器槽，保留有 driedsalticon 物品下的盐；槽空则转 saltrock 回槽，槽有不兼容物品则转为世界掉落。组件自己的 OnLoad 只恢复槽记录，不能代替内容恢复后的校正。[校正](../../../raw/current/scripts/prefabs/meatrack2.lua#L517-L535)
- **锤毁／烧毁／拆解：** 宿主先 GetNumSalts，生成相应堆数的 saltrock，再移除 collector，随后处理容器／实体。废弃变破损也先执行该返还，再移除 dryingrack 和 container；直接 RemoveComponent 不会自动产出记录里的盐。[集中返还](../../../raw/current/scripts/prefabs/meatrack2.lua#L445-L455) [三种入口](../../../raw/current/scripts/prefabs/meatrack2.lua#L458-L473) [破损](../../../raw/current/scripts/prefabs/meatrack2.lua#L476-L487)
- **显示范围：** 原版回调把待收槽数与容器实际 saltrock 数相加，映射 none／low／medium／high 的动画符号。修改数值回调不自动提供相应美术资源，且替换旧回调会失去原来的显示刷新。[回调](../../../raw/current/scripts/prefabs/meatrack2.lua#L370-L385) [公共显示初始化](../../../raw/current/scripts/prefabs/meatrack2.lua#L542-L547)

## 常见陷阱与限制

- **AddSalt增加盐实体／重复叠加同槽：** 相同槽重复调用返回 false，每槽只记布尔状态；需要多份同槽盐必须另设计记录、存档与宿主返还协议，不能只调 numsalts。[去重](../../../raw/current/scripts/components/dryingracksaltcollector.lua#L12-L22)
- **换个槽位字符串就能扩架：** 本组件不验证槽存在，也不扩容；宿主加载校正只遍历 container 的实际槽数。使用越界键可能留在记录中却不按正常取物路径处理，不能把其当虚拟物品存储。[无容量检查](../../../raw/current/scripts/components/dryingracksaltcollector.lua#L12-L22) [校正范围](../../../raw/current/scripts/prefabs/meatrack2.lua#L517-L532)
- **图标亮着就一定有可收盐：** collector 槽状态、物品图标、容器盐实体是三个对象；原版靠事件与后置加载同步。自定义直接增删只更新 collector，不会替宿主整理图标／实物。[入槽协议](../../../raw/current/scripts/prefabs/meatrack2.lua#L395-L410) [取出协议](../../../raw/current/scripts/prefabs/meatrack2.lua#L413-L442)
- **重复 OnLoad 替换成存档状态：** 它合并键而非先清空；向已有实例载入较少槽不会删除旧记录。新建实例走常规恢复，热替换需显式设计重置和回调顺序。[实现](../../../raw/current/scripts/components/dryingracksaltcollector.lua#L59-L68)
- **组件移除自动返还／撤销宿主监听：** 定义没有 OnRemoveFromEntity；返还与 itemget／itemlose 监听都在宿主。原宿主回调先判断组件存在，但新宿主仍须自行安排清理与产物交付。[完整定义](../../../raw/current/scripts/components/dryingracksaltcollector.lua#L1-L71) [监听接线](../../../raw/current/scripts/prefabs/meatrack2.lua#L554-L568)

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 源码审阅：2026-10-24，Steam buildid 25643519；完整核验 collector，定向核验珍珠多格架的入槽／移出／后置加载／损毁返还与显示、海带干 collects 配置及图标网络分工；加工器基础机制复用既有 dryingrack／dryable 页。
- 结构检查：本页格式、来源、本地链接和源码锚点严格检查通过；本批两页共84处源码引用无错误／警告。无新增完整 Lua 示例，导航登记由主代理整合检查。
- 游戏运行：未运行；同帧加工替换与取物、盐掉落失败、旧档或自定义槽键、多人图标表现及其他模组回调覆盖未实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/dryingrack]]、[[reference/components/dryable]]、[[reference/components/container]]。

## 相关链接

- [[reference/components/driedsalticon]]：干货盐覆盖图标的开关与同步。
