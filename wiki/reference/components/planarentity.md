---
title: "planarentity"
created: "2026-10-21"
updated: "2026-10-28"
type: component
summary: "让受击实体在 combat 中压低普通伤害，并提供非位面抵抗和未防御攻击的特效入口。"
aliases: ["位面实体普通伤害衰减", "非位面攻击抵抗特效", "planarentity"]
tags: [component, mechanism, callback, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/planarentity.lua", "components/combat.lua", "prefabs/shadowthrall_hands.lua", "prefabs/shadowthrall_centipede.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# planarentity

## 快速判断

- 职责与适用：combat 受击链对普通伤害作非线性压低；并非提供位面伤害或位面防御数值，后两者分别见 [[reference/components/planardamage]]、[[reference/components/planardefense]]。[变换](../../../raw/current/scripts/components/planarentity.lua#L7-L13) [受击入口](../../../raw/current/scripts/components/combat.lua#L653-L667)
- 优先使用：需要原版位面实体的普通伤害规则时使用；只需固定减伤或增加位面输出时，不以此组件替代相应战斗机制。[受击消费](../../../raw/current/scripts/components/combat.lua#L649-L667)
- 重要副作用：受击可生成 `planar_resist_fx`；作为攻击者时，还可让无装备位面防御的 inventory 目标出现 `planar_hit_fx`。后者调用条件检查本次攻击的 planar 特殊伤害键、攻击者组件与目标装备。[两个特效入口](../../../raw/current/scripts/components/planarentity.lua#L15-L39) [攻击者判定](../../../raw/current/scripts/components/combat.lua#L625-L627)
- 修改边界：原宿主在主模拟端添加；组件不主动攻击、不注册受击监听，也没有保存／加载或移除回调。实际生效依赖 combat 消费。[完整组件](../../../raw/current/scripts/components/planarentity.lua#L1-L42) [原宿主](../../../raw/current/scripts/prefabs/shadowthrall_hands.lua#L291-L323)

（26148036 核验：combat 受击链对 `planar_hit_fx` 新增 `spdamage.planar` 门槛——本次攻击不携带 planar 特殊伤害键时不再触发，旧版只检查攻击者组件与目标装备；受击／重定向主体改为 `GetAttacked_Internal`，`GetAttacked` 转调且不返回结果，`redirected_from` 临时字段的设置与清空逻辑未变。）

## 关键接口

| 接口／字段 | 约定、条件与作用 |
|---|---|
| `AbsorbDamage(damage, attacker, weapon, spdmg)` | 返回变换后的普通伤害和原 `spdmg`：`(sqrt(damage * 4 + 64) - 8) * 4`。不使用 weapon，也不增删特殊伤害；例如传入普通伤害 100，返回约 54.16。[实现](../../../raw/current/scripts/components/planarentity.lua#L7-L13) |
| `OnResistNonPlanarAttack(attacker)` | 在选定实体附近生成抵抗特效；有攻击者且位置不重合时参考攻击方向，无有效方向时随机。[实现](../../../raw/current/scripts/components/planarentity.lua#L15-L35) |
| `spawn_effect_on` | 可选回调 `(self.inst, attacker) -> 实体或 nil`；返回实体作为抵抗特效的位置／半径参照，nil／false 回退自身。组件没有 setter，原版由蜈蚣控制器直接赋值。[消费](../../../raw/current/scripts/components/planarentity.lua#L16-L20) [配置](../../../raw/current/scripts/prefabs/shadowthrall_centipede.lua#L398-L399) |
| `OnPlanarAttackUndefended(target)` | 将 `planar_hit_fx` 设为 target 的子实体；只产生特效，不增加伤害或绕过防御。[实现](../../../raw/current/scripts/components/planarentity.lua#L38-L39) |

combat 的顺序是：装备／鞍具处理、伤害类型与外部倍率、攻击者 bonusdamage，再对仍为正的普通伤害调用 `AbsorbDamage`；之后处理实体特殊防御、合计特殊伤害并调用 health。目标还需有 health、damage 非 nil、未重定向且非无敌，才能进入对应计算分支。[完整受击分支](../../../raw/current/scripts/components/combat.lua#L615-L678)

`AbsorbDamage` 仅在收到的 `spdmg == nil` 或 `spdmg.planar == nil` 时调用抵抗特效；它判断的是该步骤收到的键是否存在，`planar = 0` 也不满足“缺键”。普通伤害仍会变换，携带位面伤害不会取消普通伤害衰减。普通伤害为 0 的纯特殊伤害在此 combat 分支不会调用本方法。[键判定](../../../raw/current/scripts/components/planarentity.lua#L7-L12) [正普通伤害门槛](../../../raw/current/scripts/components/combat.lua#L653-L667)

## 典型接入

### 原版 shadowthrall_hands：位面实体与位面攻击分别组装

注册 `shadowthrall_hands` 的 fn 在客户端提前返回后创建 health、combat、planarentity，再单独创建 planardamage 并设置基础值。因此压低所受普通伤害与提供攻击位面值是两个组件的协作，不能只添加 planarentity 就认为有了位面攻击数值。[端边界](../../../raw/current/scripts/prefabs/shadowthrall_hands.lua#L291-L293) [组装](../../../raw/current/scripts/prefabs/shadowthrall_hands.lua#L306-L323) [注册](../../../raw/current/scripts/prefabs/shadowthrall_hands.lua#L345)

本组件没有减伤百分比配置；原宿主已完整添加，无需重复写入组件来“开启”其能力。自定义生物应在自身服务端构造链明确组装 combat／health 与所需攻击组件；本页不把原宿主的一行 AddComponent 称为独立生物完整示例。

（26148036 核验：shadowthrall_hands 新增 `OnShroudenSummon`——被 Shrouden 召唤时附带键为 "summoned" 的 planardamage 加成并进入 `shrouden_run`；上文的常驻组装与注册位置不变。）

### 蜈蚣控制器：把抵抗特效放在实际受击节段

`shadowthrall_centipede_controller` 是非联网控制实体，拥有 health、combat、planarentity；其 `spawn_effect_on` 返回 `combat.redirected_from`。combat 重定向到控制器前临时设置这个字段，调用完成后清空，使抵抗特效参照被玩家击中的节段，而伤害仍由控制器结算。[回调](../../../raw/current/scripts/prefabs/shadowthrall_centipede.lua#L357-L359) [控制器构造](../../../raw/current/scripts/prefabs/shadowthrall_centipede.lua#L368-L399) [临时字段](../../../raw/current/scripts/components/combat.lua#L693-L699) [注册](../../../raw/current/scripts/prefabs/shadowthrall_centipede.lua#L489-L491)

回调是即时选取参照实体，不转移伤害、不创建持久实体绑定；组件没有自行保存回调的协议，原宿主在构造时重建配置。[组件](../../../raw/current/scripts/components/planarentity.lua#L1-L42) [原宿主赋值](../../../raw/current/scripts/prefabs/shadowthrall_centipede.lua#L398-L399)

## 常见陷阱与限制

- **把公式理解成固定百分比减伤**：输出随输入变化，且处理的是到达该步骤的普通伤害；不能把武器面板伤害直接当最终扣血量。[公式](../../../raw/current/scripts/components/planarentity.lua#L8) [前后步骤](../../../raw/current/scripts/components/combat.lua#L624-L678)
- **认为有 planarentity 就抵消位面伤害**：`spdmg` 原样返回；实体特殊防御由后续 `ApplySpDefense` 处理。已有 [[reference/components/planardefense]] 维护该数值机制。[返回](../../../raw/current/scripts/components/planarentity.lua#L12) [后续消费](../../../raw/current/scripts/components/combat.lua#L663-L667)
- **把特效当成伤害依据**：抵抗特效依据传入键，未防御攻击特效依据本次攻击的 planar 键、攻击者组件和目标装备；两者都不证明某个固定抵消量或实际位面输出。[抵抗入口](../../../raw/current/scripts/components/planarentity.lua#L7-L18) [攻击入口](../../../raw/current/scripts/components/combat.lua#L625-L627)
- **期望接管所有扣血**：这里核验的是 combat:GetAttacked（26148036 起转调 GetAttacked_Internal）的消费链；直接调用 health 扣血不会自动经过本方法，不能当作所有伤害入口的通用防护。[调用位置](../../../raw/current/scripts/components/combat.lua#L659-L678)

## 验证记录

- 版本增量复核：2026-10-25，Steam buildid 26148036；核对范围：本页 sources 的全部变更文件对照 build-26148036 更新报告逐条核验，锚点重定位 14 处、结论修订 3 处（planar_hit_fx planar 门槛，见正文"26148036 核验"标注）；未覆盖：运行验证。

- 源码审阅：2026-10-21，Steam buildid 25643519；完整审阅 planarentity，以及 combat 的受击／特效／重定向分支、shadowthrall_hands 的主端构造与注册、蜈蚣控制器及回调。位面输出／数值防御细节复用同 build 的对应页；未枚举全部位面实体、全部伤害入口或特效资源生命周期。
- 结构检查：本次对本页执行元数据、标签、大小、本地链接及严格源码路径／行号检查；无新增完整 Lua 示例。
- 游戏运行：未运行；普通／混合／纯位面受击、装备防御与蜈蚣节段特效、存读档仍需实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。
相关：[[reference/components/combat]]；[[reference/components/planardamage]]；[[reference/components/planardefense]]。
