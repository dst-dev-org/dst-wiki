---
title: "lureplantspawner"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "在春季按活跃玩家轨迹和随机远点尝试生成食人花，说明生成间隔、幸运判定、地皮与空位限制及临时状态边界。"
aliases: ["lureplantspawner", "食人花自然生成", "春季不刷食人花", "玩家轨迹刷食人花"]
tags: [component, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/lureplantspawner.lua", "prefabs/forest.lua", "prefabs/world.lua", "tuning.lua", "worldsettings_overrides.lua", "componentutil.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# lureplantspawner

## 快速判断

- 职责与适用：森林世界主模拟的春季食人花自然生成器；为每名活跃玩家维护轨迹与生成任务，不管理已生成食人花的眼球草或消化行为。眼球草补生见 [[reference/components/minionspawner]]。[lureplantspawner.lua:5-49](../../../raw/current/scripts/components/lureplantspawner.lua#L5-L49) [forest.lua:639-659](../../../raw/current/scripts/prefabs/forest.lua#L639-L659)
- 优先使用：排查自然生成先核对春季启动、间隔大于 0、玩家轨迹／备用落点、幸运概率；它没有公开的即时生成或调频接口。[lureplantspawner.lua:135-189](../../../raw/current/scripts/components/lureplantspawner.lua#L135-L189) [公开方法 lureplantspawner.lua:259-285](../../../raw/current/scripts/components/lureplantspawner.lua#L259-L285)
- 重要副作用：一次尝试会消费／剔除选中的轨迹点，即使幸运判定失败也可能先消耗轨迹；成功则生成 `lureplant`，传送后进入 `spawn` 状态。[lureplantspawner.lua:92-149](../../../raw/current/scripts/components/lureplantspawner.lua#L92-L149)
- 修改边界：正常排程由春季门槛控制，单次回调只另查非冬季；不能据回调分支推导“秋夏也自然刷新”。两项间隔在构造时缓存，旧模式方法全部为空。[lureplantspawner.lua:48-49](../../../raw/current/scripts/components/lureplantspawner.lua#L48-L49) [lureplantspawner.lua:135-189](../../../raw/current/scripts/components/lureplantspawner.lua#L135-L189) [lureplantspawner.lua:259-277](../../../raw/current/scripts/components/lureplantspawner.lua#L259-L277)

## 关键接口

| 接口／字段 | 签名或约束 | 条件、作用与证据 |
|---|---|---|
| `TUNING.LUREPLANT_SPAWNINTERVAL` | 构造时缓存；秒 | 必须大于 0 才启动；默认 4 游戏日。不是已经存在的食人花成长时间。[lureplantspawner.lua:48-49](../../../raw/current/scripts/components/lureplantspawner.lua#L48-L49) [lureplantspawner.lua:168-189](../../../raw/current/scripts/components/lureplantspawner.lua#L168-L189) [tuning.lua:6019-6020](../../../raw/current/scripts/tuning.lua#L6019-L6020) |
| `TUNING.LUREPLANT_SPAWNINTERVALVARIANCE` | 构造时缓存；秒 | 每轮延迟为 interval + `math.random(-variance, variance)`，默认 ±1 游戏日；首次也使用该规则。[lureplantspawner.lua:151-159](../../../raw/current/scripts/components/lureplantspawner.lua#L151-L159) |
| `SpawnModeNever/Heavy/Normal/Med/Light()` | 全部为 deprecated 空实现 | 不更改间隔、季节或任务；世界设置通过 TUNING 覆盖实现。[lureplantspawner.lua:259-277](../../../raw/current/scripts/components/lureplantspawner.lua#L259-L277) [worldsettings_overrides.lua:611-638](../../../raw/current/scripts/worldsettings_overrides.lua#L611-L638) |
| `GetDebugString()` | 返回间隔及方差字符串 | 只报告缓存配置，不给轨迹、成功生成数或剩余等待时间。[lureplantspawner.lua:283-285](../../../raw/current/scripts/components/lureplantspawner.lua#L283-L285) |

## 关键流程

### 春季启动与按玩家调度

构造从 `AllPlayers` 创建 `{player, trail={}}` 记录，监听世界玩家加入／离开和 `seasontick`，最后 `StartUpdating(true)`。启动要求当前 `isspring` 且 interval>0；季节事件为春季时启动，其他季节停止并取消轨迹、生成两类任务。原版安装位于世界主模拟路径，客户端在调用森林 `master_postinit` 前返回。[lureplantspawner.lua:168-199](../../../raw/current/scripts/components/lureplantspawner.lua#L168-L199) [lureplantspawner.lua:230-253](../../../raw/current/scripts/components/lureplantspawner.lua#L230-L253) [world.lua:568-605](../../../raw/current/scripts/prefabs/world.lua#L568-L605)

正在更新时新玩家加入会创建空轨迹并安排两类任务；离开取消任务且删除该玩家记录。停止季节更新本身不清空仍在表内玩家的轨迹。没有按玩家附近现存食人花数、世界总数或生成后追踪表限流的逻辑。[lureplantspawner.lua:191-228](../../../raw/current/scripts/components/lureplantspawner.lua#L191-L228) [完整单次尝试 lureplantspawner.lua:135-149](../../../raw/current/scripts/components/lureplantspawner.lua#L135-L149)

默认一天为 480 秒，默认生成等待为 1440～2400 秒，即 3～5 游戏日；每次尝试结束清任务槽并续排，无论无落点、幸运失败或因冬季跳过。世界设置 never 改为0／0，rare 为10±2天，often 为3±1天，always 为2±0.5天；default 来自 tuning，覆盖表中的 default 块是注释。[tuning.lua:19-20](../../../raw/current/scripts/tuning.lua#L19-L20) [tuning.lua:6019-6020](../../../raw/current/scripts/tuning.lua#L6019-L6020) [lureplantspawner.lua:135-159](../../../raw/current/scripts/components/lureplantspawner.lua#L135-L159) [worldsettings_overrides.lua:611-638](../../../raw/current/scripts/worldsettings_overrides.lua#L611-L638)

### 轨迹记录与有效落点

轨迹每 40 秒记一次，启动时现有玩家的首延迟按索引 `i` 错开，后来加入使用默认首延迟。轨迹最多32点：每次先在满表时移除最老点，再检查当前点；当前位置距旧点小于40且当前地皮有效时，为该旧点加0.2权重，最高3，否则插入权重0.01的新点。插入新点本身不要求有效地皮，最终落点仍会重查。[lureplantspawner.lua:13-32](../../../raw/current/scripts/components/lureplantspawner.lua#L13-L32) [lureplantspawner.lua:55-77](../../../raw/current/scripts/components/lureplantspawner.lua#L55-L77) [启动错开 lureplantspawner.lua:173-185](../../../raw/current/scripts/components/lureplantspawner.lua#L173-L185) [玩家加入 lureplantspawner.lua:205-217](../../../raw/current/scripts/components/lureplantspawner.lua#L205-L217)

生成优先按轨迹权重抽点；遇到无效点会从表中删除并沿后续点继续检查，拿到有效点后也删除该点。无可用轨迹结果才用玩家当前位置作备用搜索：随机取80～200的整数半径，在同一个圆周上检查40个角度，再从有效点随机挑一个。因此80～200限制只属于备用圆周搜索，轨迹点并不要求离当前玩家这么远。[lureplantspawner.lua:92-143](../../../raw/current/scripts/components/lureplantspawner.lua#L92-L143)

有效点必须位于白名单地皮（DIRT、SAVANNA、GRASS、FOREST、DECIDUOUS、MARSH、CAVE、FUNGUS、FUNGUSRED、FUNGUSGREEN、SINKHOLE、MUD），半径1查询没有任何实体，且通过 `Map:IsDeployPointClear(pt,nil,1)`。白名单含洞穴地皮不等于原版洞穴世界会安装本组件。[白名单 lureplantspawner.lua:18-32](../../../raw/current/scripts/components/lureplantspawner.lua#L18-L32) [有效性 lureplantspawner.lua:86-90](../../../raw/current/scripts/components/lureplantspawner.lua#L86-L90) [森林安装 forest.lua:659](../../../raw/current/scripts/prefabs/forest.lua#L659

### 玩家数量与幸运判定

单次回调先确认非冬季，以 `1 / 活跃玩家数` 为基础概率调用 `TryLuckRoll(player, chance, LuckFormulas.LureplantChanceSpawn)`，再找落点；两者都成功才生成。无 `luckuser` 时幸运值为0并保留基础概率；该公式在负幸运时提高概率、正幸运时降低概率，所以不能将多人情况下的成功概率固定写成 `1/N`，也不能将单人概括为每轮必生成。[lureplantspawner.lua:135-149](../../../raw/current/scripts/components/lureplantspawner.lua#L135-L149) [公式 componentutil.lua:2084-2090](../../../raw/current/scripts/componentutil.lua#L2084-L2090) [绑定 componentutil.lua:2154](../../../raw/current/scripts/componentutil.lua#L2154 [取值 componentutil.lua:2203-2214](../../../raw/current/scripts/componentutil.lua#L2203-L2214) [掷点 componentutil.lua:2237-2251](../../../raw/current/scripts/componentutil.lua#L2237-L2251)

## 典型接入

### 用森林世界设置控制自然食人花频率

原版 `forest` 的服务端初始化安装组件，世界设置 `lureplants` 通过覆盖两项 TUNING 配置控制间隔与方差；这是构造前配置的原版参照。模组若沿用此机制，应在已核实的世界设置／构造前配置流程修改相应参数，作用范围为该世界所有活跃玩家，恢复时回到所用设置对应值。本页不提供运行中改缓存局部值或跨世界安装的 demo。[forest.lua:639-659](../../../raw/current/scripts/prefabs/forest.lua#L639-L659) [worldsettings_overrides.lua:611-638](../../../raw/current/scripts/worldsettings_overrides.lua#L611-L638) [构造缓存 lureplantspawner.lua:42-49](../../../raw/current/scripts/components/lureplantspawner.lua#L42-L49)

## 常见陷阱与限制

- **关掉自然生成会删除已有食人花：** 本组件只创建新花，没有维护花实体集合或移除既有花的路径；never=0 阻止后续正常启动，不代表清除已有实体。[lureplantspawner.lua:135-189](../../../raw/current/scripts/components/lureplantspawner.lua#L135-L189) [worldsettings_overrides.lua:614-617](../../../raw/current/scripts/worldsettings_overrides.lua#L614-L617)
- **春季结束会保存剩余秒数：** 停止时取消任务，下一次启动重新随机等待；组件没有 `OnSave`／`OnLoad`，玩家轨迹和倒计时不由它保存。构造重建空轨迹，不应依赖重载前的轨迹或剩余时间。[lureplantspawner.lua:151-199](../../../raw/current/scripts/components/lureplantspawner.lua#L151-L199) [lureplantspawner.lua:242-253](../../../raw/current/scripts/components/lureplantspawner.lua#L242-L253)
- **把直接 SpawnPrefab 当自然生成验证：** 自然路径还包含春季调度、玩家概率、轨迹或备用取点与有效点筛选；直接造实体无法验证这些条件，也不会自动执行本回调的传送和 `spawn` 状态切换。[lureplantspawner.lua:86-189](../../../raw/current/scripts/components/lureplantspawner.lua#L86-L189)
- **运行时移除世界组件就是完整停用：** 组件没有 `OnRemoveFromEntity` 清理钩子，两类任务挂在玩家实体上；不要仅删世界组件就假定任务已撤销。[lureplantspawner.lua:73-84](../../../raw/current/scripts/components/lureplantspawner.lua#L73-L84) [lureplantspawner.lua:155-166](../../../raw/current/scripts/components/lureplantspawner.lua#L155-L166)

## 验证记录

- 2026-10-26，`steam-buildid:26148036`：审阅组件全文、森林主模拟安装与世界服务端分支、tuning／世界设置对应配置、幸运公式和掷点消费者，覆盖春季任务生命周期、轨迹权重／淘汰、落点白名单、备用搜索、概率及无组件存读钩子的边界。
- 游戏运行：未运行；未验证多人实际分布、季节切换同帧竞态或 Map 原生空位测试的几何细节；未展开食人花实体的眼球草、成长和存档机制。
- 结构检查：`validate_page.py` 通过；两页合并执行 `validate_source_links.py --strict`，83 个行号引用、0 错误／0 警告。本页无新增 Lua 示例；机械检查不代替游戏测试。

相关：[[reference/components/butterflyspawner]] · [[reference/components/luckuser]]
