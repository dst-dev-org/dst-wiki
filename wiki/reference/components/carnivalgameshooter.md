---
title: "carnivalgameshooter"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "狂欢节射击小游戏的瞄准与发射组件：角度表往返摆动、按当前角在目标弧上生成抛射物。"
aliases: ["carnivalgameshooter", "射击小游戏瞄准", "狂欢节射击", "角度瞄准器"]
tags: [component]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/carnivalgameshooter.lua", "prefabs/carnivalgame_shooting.lua", "tuning.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# carnivalgameshooter

## 快速判断

- 职责与适用：装在射击小站台（minigame 实体）上；维护瞄准角与摆动方向，按角度把弹丸抛向以站台为中心的目标弧。([carnivalgameshooter.lua:36-59](../../../raw/current/scripts/components/carnivalgameshooter.lua#L36-L59)
- 优先使用：AddComponent 后立即 `Initialize()` 设初值；服务器每帧 `UpdateAiming(dt)` 推进摆动并返回 `angle, meterdirection`。([carnivalgame_shooting.lua:565-566](../../../raw/current/scripts/prefabs/carnivalgame_shooting.lua#L565-L566) ([carnivalgame_shooting.lua:320-326](../../../raw/current/scripts/prefabs/carnivalgame_shooting.lua#L320-L326)
- 重要副作用：`Shoot()` 生成 `carnivalgame_shooting_projectile`（带 complexprojectile 弹道），并记录 `projectile.shooter`。([carnivalgameshooter.lua:39-40](../../../raw/current/scripts/components/carnivalgameshooter.lua#L39-L40) ([carnivalgameshooter.lua:56-58](../../../raw/current/scripts/components/carnivalgameshooter.lua#L56-L58)
- 修改边界：发射时机由外部控制——原版仅在 minigame 进行中且无连发延迟时响应按钮事件。([carnivalgame_shooting.lua:573-575](../../../raw/current/scripts/prefabs/carnivalgame_shooting.lua#L573-L575)

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `Initialize()` | 无参 | angle=(MIN+MAX)/2、power=TUNING.CARNIVALGAME_SHOOTING_POWER、meterdirection=1。([carnivalgameshooter.lua:11-15](../../../raw/current/scripts/components/carnivalgameshooter.lua#L11-L15) |
| `UpdateAiming(dt)` | 返回 angle, meterdirection | angle 按 `SPEED*direction` 推进，到 MIN／MAX 折返；调用方负责用返回值驱动箭头表现。([carnivalgameshooter.lua:17-30](../../../raw/current/scripts/components/carnivalgameshooter.lua#L17-L30) ([carnivalgame_shooting.lua:320-325](../../../raw/current/scripts/prefabs/carnivalgame_shooting.lua#L320-L325) |
| `SetAim()` | 无参 | 播放 `summerevent/cannon/place` 音效。([carnivalgameshooter.lua:32-34](../../../raw/current/scripts/components/carnivalgameshooter.lua#L32-L34) |
| `Shoot()` | 无参 | 由 self.angle 换算弧上落点（TUNING.CARNIVALGAME_SHOOTING_TARGET_ARC_R=10、X=2.5），按站台旋转后 complexprojectile：Speed 20、Gravity -40、Launch。([carnivalgameshooter.lua:42-58](../../../raw/current/scripts/components/carnivalgameshooter.lua#L42-L58) ([tuning.lua:5962-5963](../../../raw/current/scripts/tuning.lua#L5962-L5963) |

## 典型接入

### 复用瞄准摆动做自定义小游戏

- 前提与选择：组件不含输入与计分逻辑；瞄准循环由宿主 onUpdate 驱动（原版 Server_OnUpdateAiming）。([carnivalgame_shooting.lua:319-326](../../../raw/current/scripts/prefabs/carnivalgame_shooting.lua#L319-L326)

```lua
inst:AddComponent("carnivalgameshooter")
inst.components.carnivalgameshooter:Initialize()

local angle, dir = inst.components.carnivalgameshooter:UpdateAiming(dt)
inst.components.carnivalgameshooter:Shoot()
```

- 生效与撤销：angle 状态保留在组件里，Initialize 可随时复位；换弹丸 prefab 需同步改 Shoot 内 SpawnPrefab 名。

## 常见陷阱与限制

- `TUNING.CARNIVALGAME_SHOOTING_POWER` 在本快照 tuning.lua 中不存在：Initialize 里 power 实际为 nil（组件内也未读取 power），不要依赖该字段。[carnivalgameshooter.lua:13](../../../raw/current/scripts/components/carnivalgameshooter.lua#L13
- Shoot 不先 UpdateAiming：未摆动就 Shoot 会用 Initialize 初值角发射。
- 弹丸 prefab 与其 complexprojectile 参数写死在组件内；换目标弧尺寸需同时改 TARGET_ARC_R／X 与 Launch 落点换算。

## 验证记录

- 源码审阅：2026-10-28，`steam-buildid:26230496`：审阅组件全文、站台安装与按钮事件触发、Server_OnUpdateAiming 消费、相关 tuning 键存在性。
- 游戏运行：未运行；未复现完整游玩回合。
- 结构检查：validate_page.py 通过；validate_source_links.py --strict 通过（本批 16 页合并执行）。

相关：[[reference/components/complexprojectile]]
