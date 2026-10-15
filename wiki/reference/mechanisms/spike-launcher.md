---
title: "尖刺发射物与发射传送回退（build 26148036）"
created: "2026-10-26"
updated: "2026-10-28"
type: mechanism
summary: "fossilspike／fossilspike2／sandspike 族／sharkboi_icespike 四族地面尖刺共用 SpikeLaunch 发射模式（角度抖动＋弧线抛射＋SetLanded 空中标记）；本 build 统一给发射起点加 TryTeleportToLaunchPos 失败回退——洞穴 void 传送失败时垫高原地发射，不再卡进地下。"
aliases: ["尖刺发射", "SpikeLaunch", "发射传送回退", "groundspike"]
tags: [mechanism, data]
dst_build: "steam-buildid:26230496"
sources: ["prefabs/fossil_spike.lua", "prefabs/fossil_spike2.lua", "prefabs/sand_spike.lua", "prefabs/sharkboi_icespike.lua", "physics.lua", "prefabs/stalker.lua", "stategraphs/SGantlion_angry.lua", "components/lootdropper.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# 尖刺发射物与发射传送回退（build 26148036）

## 快速判断

- 职责与适用：四族"自身作为攻击载体"的地面尖刺（化石×2／沙晶／冰刺）各自持有同名局部 `SpikeLaunch(inst, launcher, basespeed, startheight, startradius)`，用于把自身或邻近物品按发射者朝向抛出；本 build 四文件的发射段逐字加入同一条回退。[fossil_spike.lua:32-53](../../../raw/current/scripts/prefabs/fossil_spike.lua#L32-L53)、[sharkboi_icespike.lua:66-86](../../../raw/current/scripts/prefabs/sharkboi_icespike.lua#L66-L86)
- 优先使用：抛物品复用各尖刺的 `SpikeLaunch`（原版用法：尖刺生成时把碰撞到的 `_inventoryitem` 实体抛飞）；一次性发射单实体可直接用全局 `Launch`（physics.lua），它内部同样调 `SetLanded(false, true)`；自定义弧起点再配 `TryTeleportToLaunchPos`。[physics.lua:65-80](../../../raw/current/scripts/physics.lua#L65-L80)
- 重要副作用：发射先把实体传送到弧起点再赋初速——旧版传送失败（洞穴 void）时实体停在原地且初速向上，可能钻进地形；新版失败回退为在**当前位置**垫高到 `max(y1, startheight)` 后发射，并把 `inventoryitem` 置为未落地（`SetLanded(false, true)`，落地事件与拾取判定见 [[reference/components/inventoryitem]]）。[fossil_spike.lua:46-52](../../../raw/current/scripts/prefabs/fossil_spike.lua#L46-L52)
- 修改边界：发射与破坏链全部服务端；`TryTeleportToLaunchPos` 是 physics.lua 全局（海洋世界恒成功，洞穴按陆地瓦判定），同类回退在 lootdropper FlingItem 中另有变体（见陷阱）。[physics.lua:55-62](../../../raw/current/scripts/physics.lua#L55-L62)

## 关键接口

| 接口／配置项 | 签名或数据约束 | 消费者、条件、作用与证据 |
|---|---|---|
| `TryTeleportToLaunchPos(inst, x, y, z)` | -> boolean（已传送为 true） | 全局：`TheWorld.has_ocean` 或目标点为陆地瓦才 `Physics:Teleport`；洞穴 void 返回 false 不动实体。消费者：四尖刺发射段、全局 Launch 不用（直接原地起跳）、lootdropper FlingItem 睡眠/抛射分支。[physics.lua:55-62](../../../raw/current/scripts/physics.lua#L55-L62) |
| `Launch(inst, launcher, basespeed)` | 全局抛物助手 | 无弧起点参数：实体原地、速度 `(cos·spd, 10, sin·spd)`；同样 `SetLanded(false, true)`。与 SpikeLaunch 的区别只在起跳位置策略。[physics.lua:65-80](../../../raw/current/scripts/physics.lua#L65-L80) |
| `SpikeLaunch` 发射参数 | basespeed／startheight／startradius | 各族取值见下表；角度 = 发射者→实体方位 ±10° 随机，初速竖直分量 `speed * 5 + rand * 2`。[fossil_spike.lua:37-49](../../../raw/current/scripts/prefabs/fossil_spike.lua#L37-L49) |
| `inventoryitem:SetLanded(false, true)` | 置未落地 | 发射后物品在空中不参与地面拾取/收纳判定，落地后恢复；四尖刺与全局 Launch 行为一致。[fossil_spike.lua:50-52](../../../raw/current/scripts/prefabs/fossil_spike.lua#L50-L52) |

## 关键流程

| 入口／条件 | 当前步骤与结果 | 后续／中断／清理及证据 |
|---|---|---|
| 尖刺生成 → `StartSpike`/`DoDamage` | 化石尖刺：落地动画播完 `ChangeToObstacle`（质量 0、只碰 ITEMS/CHARACTERS）；`DoDamage` 按 COLLAPSIBLE_WORK_ACTIONS 摧毁工作物、采拾 pickable、击杀/攻击无位移单位，再把半径内 `_inventoryitem` 实体逐个 `SpikeLaunch` 抛飞（`DeactivateInventoryItemBeforeLaunch` 先去激活）。[fossil_spike.lua:70-113](../../../raw/current/scripts/prefabs/fossil_spike.lua#L70-L113) | 存续由 `task` 计时（5+rand 秒）到期 `KillSpike` → 播 base_pst → erode_ash 侵蚀移除；`RestartSpike` 可续期。[fossil_spike.lua:115-178](../../../raw/current/scripts/prefabs/fossil_spike.lua#L115-L178) |
| 发射段（四族共用形状） | ① 算角度与速度 → ② `TryTeleportToLaunchPos` 到弧起点（发射者位置 + startradius 偏移、startheight 高度）→ ③ **失败回退**：`Physics:Teleport(x1, max(y1, startheight), z1)` 原地垫高 → ④ `SetVel` 弧线初速 → ⑤ `SetLanded(false, true)`。[fossil_spike.lua:46-52](../../../raw/current/scripts/prefabs/fossil_spike.lua#L46-L52)、[sand_spike.lua:66-72](../../../raw/current/scripts/prefabs/sand_spike.lua#L66-L72) | 回退后仍可能落在 void 边缘（只保证不卡地下，不保证落点可通行）；物品落地由物理结算恢复 landed。 |
| 各族宿主与差异 | 化石：`stalker` 一阶段环形刷 `fossilspike`（可通行且非洞边取点）、二阶段 `fossilspike2`（变体带 `RestartSpike(0, variation, level)`）；沙晶：`SGantlion_angry` 刷 `sandspike_<short/med/tall>` 与 `sandblock`，破坏时玻璃化 `glassspike_*`；冰刺：sharkboi 域，`islarge` net_bool 双档半径/工作量，击中推 `knockback`（`strengthmult 0.3`、小刺 `forcelanded=true`）。[stalker.lua:566-569](../../../raw/current/scripts/prefabs/stalker.lua#L566-L569)、[:845-847](../../../raw/current/scripts/prefabs/stalker.lua#L845-L847)、[SGantlion_angry.lua:68](../../../raw/current/scripts/stategraphs/SGantlion_angry.lua#L68)、[sharkboi_icespike.lua:104-152](../../../raw/current/scripts/prefabs/sharkboi_icespike.lua#L104-L152) | 化石尖刺 `NON_COLLAPSIBLE` 排除 flying/ghost，`fossil_spike2` 注释放行这两类可被击中；沙晶按 `TUNING.SANDSPIKE.LIFETIME` 计时破碎。[fossil_spike2.lua:35](../../../raw/current/scripts/prefabs/fossil_spike2.lua#L35)、[sand_spike.lua:86-96](../../../raw/current/scripts/prefabs/sand_spike.lua#L86-L96) |

## 典型接入

### 给自定义尖刺/抛射实体加带回退的发射

- 前提与选择：模组 prefab 服务端实例，带 Physics 与 `inventoryitem`（或无需落地标记的纯 FX）；本例复刻四尖刺发射段最小形状。
- 生效与撤销：发射即生效；无撤销——实体已按物理飞行，落地由引擎结算。

```lua
local function MyLaunch(inst, launcher, basespeed, startheight, startradius)
    local x0, y0, z0 = launcher.Transform:GetWorldPosition()
    local x1, y1, z1 = inst.Transform:GetWorldPosition()
    local dx, dz = x1 - x0, z1 - z0
    local angle = (dx * dx + dz * dz) > 0
        and math.atan2(dz, dx) + (math.random() * 20 - 10) * DEGREES
        or TWOPI * math.random()
    local sina, cosa = math.sin(angle), math.cos(angle)
    local speed = basespeed + math.random()
    if not GLOBAL.TryTeleportToLaunchPos(inst, x0 + startradius * cosa, startheight, z0 + startradius * sina) then
        inst.Physics:Teleport(x1, math.max(y1, startheight), z1) -- 洞穴 void 回退：原地垫高
    end
    inst.Physics:SetVel(cosa * speed, speed * 5 + math.random() * 2, sina * speed)
    if inst.components.inventoryitem then
        inst.components.inventoryitem:SetLanded(false, true)
    end
end
```

- 调整与验证：竖直速度系数 5 与随机 ±2 沿用原版弧线手感；`TryTeleportToLaunchPos` 是全局可直接 `GLOBAL.` 取用；本例为模式复刻，未运行。

## 常见陷阱与限制

- **回退目标是自身位置不是弧起点**：与 lootdropper FlingItem 的回退不同——FlingItem 失败且 `y_offset > 0` 时传送到**掉落源实体**上空（`pt.x, pt.y + y_offset`），尖刺族传送到**实体当前位置**垫高；照抄时别混用两种语义。[lootdropper.lua:352-358](../../../raw/current/scripts/components/lootdropper.lua#L352-L358)、[fossil_spike.lua:46-48](../../../raw/current/scripts/prefabs/fossil_spike.lua#L46-L48)
- **`TryTeleportToLaunchPos` 海洋世界恒成功**：地面（`has_ocean`）不查瓦片直接传送，只有洞穴才做陆地判定；在地面模组里测不到回退分支。[physics.lua:56-58](../../../raw/current/scripts/physics.lua#L56-L58)
- **`SetLanded(false, true)` 是发射的配套动作**：跳过它会在空中维持"已落地"状态，物品飞行途中即可被拾取/收纳；第二参 `true` 表示立即同步不等待落地事件。[fossil_spike.lua:50-52](../../../raw/current/scripts/prefabs/fossil_spike.lua#L50-L52)
- **四族 `SpikeLaunch` 是局部副本不是公共库**：行为当前逐字一致（本 build 同批改四份），但历史上游过漂移；引用具体行为时锚到所在文件，不要假设"改一处全族生效"。[fossil_spike2.lua:21](../../../raw/current/scripts/prefabs/fossil_spike2.lua#L21)、[sand_spike.lua:52](../../../raw/current/scripts/prefabs/sand_spike.lua#L52)、[sharkboi_icespike.lua:66](../../../raw/current/scripts/prefabs/sharkboi_icespike.lua#L66
- **尖刺抛物品会先 `DeactivateInventoryItemBeforeLaunch`**：被抛实体若在容器内先移出再抛；模组包装拾取逻辑需容忍"被尖刺抛起"的短暂未落地状态。[fossil_spike.lua:106-111](../../../raw/current/scripts/prefabs/fossil_spike.lua#L106-L111)

## 验证记录

- 源码审阅：2026-10-26，Steam buildid 26148036；覆盖 fossil_spike.lua 全文与另三族的 SpikeLaunch／DoDamage 差异段、physics.lua 全局发射族、stalker／SGantlion_angry 生成点、lootdropper FlingItem 回退对照段；旧新快照 diff 确认四文件发射段本 build 逐字同改（+6/−1）。更新报告 P-42／P-43／P-96／P-105 四行此前为待审，本页按当前源码转正。未覆盖：sharkboi_icespike 的生成入口与生长切换全流程、glass_spike 关联行为、游戏运行。
- 游戏运行：未运行。
- 结构检查：2026-10-26 执行 validate_page.py（standard 档）与 validate_source_links.py（--strict）通过；页内 Lua 示例经 parser_probe.exe --syntax-only 语法检查通过。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/inventoryitem]]（SetLanded 语义与拾取门禁）、[[reference/mechanisms/vault-virtualroom-migration]]（同为 26148036 行为域转正）。
