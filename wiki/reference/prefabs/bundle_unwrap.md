---
title: "开包特效（bundle_unwrap）"
created: "2026-10-31"
updated: "2026-11-01"
type: prefab
summary: "十七个通用表注册承载包裹类物品的展开烟：名字由 bundle.lua 工厂按 inventoryimage 动态拼接（(inventoryimage or name)..\"_unwrap\"），UNWRAP 动作在战利品落点（y 压平为 0）生成，打开声由打开者播放；龙舟赛对红包年袋另有直落点复用，quagmire 两名快照内仅依赖声明。"
aliases: ["bundle_unwrap", "gift_unwrap", "redpouch_unwrap", "redpouch_yotp_unwrap", "redpouch_yotc_unwrap", "redpouch_yotb_unwrap", "redpouch_yot_catcoon_unwrap", "redpouch_yotr_unwrap", "redpouch_yotd_unwrap", "redpouch_yoth_unwrap", "yotc_seedpacket_unwrap", "yotc_seedpacket_rare_unwrap", "carnival_seedpacket_unwrap", "wetpouch_unwrap", "hermit_bundle_unwrap", "quagmire_seedpacket_unwrap", "quagmire_crate_unwrap", "开包特效", "礼物开包", "红包特效", "捆绑物资"]
tags: [prefab, fx]
dst_build: "steam-buildid:26335394"
sources: ["fx.lua", "prefabs/bundle.lua", "components/unwrappable.lua", "actions.lua", "prefabs/boatrace_start.lua", "prefabs/boatrace_seastack.lua", "prefabs/quagmire_crates.lua", "prefabs/quagmire_seedpackets.lua", "prefabs/lavaarena_abigail_flower.lua", "languages/chinese_s.po"]
reliability: verified
validation: source-reviewed
status: active
---

# 开包特效（bundle_unwrap）

## 快速判断

- 职责与适用：十七个通用表注册承载包裹类物品（捆绑物资／礼物／红包系列／种子包／湿布袋／隐士包裹等）打开时的展开烟，全部 anim=`unwrap`、无 fn 无声。工厂协议见 [[reference/mechanisms/generic-fx-factory|通用 FX 工厂]]。[条目组](../../../raw/current/scripts/fx.lua#L1543-L1642)
- 优先使用：查"开包特效／礼物开包／红包特效"到本页；打包动作读 [[reference/components/bundler|bundler]]，可开包物品读 [[reference/components/unwrappable|unwrappable]]。
- 重要副作用：FX 在战利品落点生成后由打开者播放 packaged／皮肤包裹声，包裹本体随后 Remove。[收尾](../../../raw/current/scripts/prefabs/bundle.lua#L297-L302)
- 修改边界：十五名由 bundle.lua 工厂动态拼名（17 个物品变体产出 15 个烟名，按 inventoryimage 或 name）；quagmire 两名快照内无主模拟生成点。[拼名](../../../raw/current/scripts/prefabs/bundle.lua#L217-L221)

## 注册与构造

| 注册名／变体 | 工厂与初始化分界 | 差异与证据 |
|---|---|---|
| `bundle_unwrap`／`gift_unwrap`／`redpouch_unwrap` | 通用表条目，无 fn | bank／build 分别为 `bundle`／`gift`／`redpouch`（即注册名去掉 `_unwrap` 后缀）；对应捆绑物资／礼物／红包。[条目](../../../raw/current/scripts/fx.lua#L1543-L1558) |
| `redpouch_yotp_unwrap`／`_yotc_unwrap`／`_yotb_unwrap`／`_yot_catcoon_unwrap`／`_yotd_unwrap`／`_yoth_unwrap` | 同上协议 | 红包活动六名 **bank／build 复用 `redpouch`**。[条目](../../../raw/current/scripts/fx.lua#L1561-L1595) |
| `redpouch_yotr_unwrap` | 同上协议 | 红包活动 yotr 一名用 `redpouch_yotr` 自有资源。[条目](../../../raw/current/scripts/fx.lua#L1585-L1588) |
| `yotc_seedpacket_unwrap`／`_rare_unwrap`／`carnival_seedpacket_unwrap` | 同上协议 | 三名种子包烟 **bank／build 复用 `bundle`**。[条目](../../../raw/current/scripts/fx.lua#L1603-L1618) |
| `wetpouch_unwrap`／`hermit_bundle_unwrap` | 同上协议 | bank／build 分别为 `wetpouch`／`hermit_bundle`。[条目](../../../raw/current/scripts/fx.lua#L1621-L1630) |
| `quagmire_seedpacket_unwrap`／`quagmire_crate_unwrap` | 同上协议 | 暴食（Gorge）活动两名，bank／build 分别为 `quagmire_seedpacket`／`quagmire_crate`；快照内仅依赖声明。[条目](../../../raw/current/scripts/fx.lua#L1633-L1642) |

十七名在简中 .po 均无 `STRINGS.NAMES` 键；标题中文为已核验的效果描述，非官方译名。来源物品官方名：捆绑物资 [po:339390-339392](../../../raw/current/scripts/languages/chinese_s.po#L339390-L339392)、礼物 [po:342655-342657](../../../raw/current/scripts/languages/chinese_s.po#L342655-L342657)、红包 [po:347915-347917](../../../raw/current/scripts/languages/chinese_s.po#L347915-L347917)。

## 关联机制导航

| 类型／对象 | 本 Prefab 如何使用 | 变体、端与生效条件 | Wiki／源码入口及证据 |
|---|---|---|---|
| 工厂协议 | 代理＋一帧本地表现，落点式 | 打开位置即表现位置 | [[reference/mechanisms/generic-fx-factory]] |
| 选名与生成（bundle.lua） | `MakeBundle` 工厂 17 个物品变体（bundle／gift／redpouch＋红包七名＋种子包三名＋wetpouch／hermit_bundle＋两个复用图小变体）产出 15 个不同烟名：`(inventoryimage or name).."_unwrap"` 放进 prefabs 依赖；`OnUnwrapped` 在战利品落点生成该烟、打开者播 `packaged` 或皮肤包裹声、Remove 包裹；**烧焦包裹只落 ash**，无战利品与开包烟 | 服务端；`redpouch_yoth_small`／`hermit_bundle_shells` 两变体显式传 inventoryimage，分别复用 yoth／hermit 的烟 | [依赖拼名](../../../raw/current/scripts/prefabs/bundle.lua#L217-L221)、[OnUnwrapped](../../../raw/current/scripts/prefabs/bundle.lua#L271-L302)、[接线](../../../raw/current/scripts/prefabs/bundle.lua#L374-L374)、[变体表](../../../raw/current/scripts/prefabs/bundle.lua#L678-L698) |
| 动作链 | `ACTIONS.UNWRAP` → `unwrappable:Unwrap(doer)`：落点 y 压平为 0，持有中的包裹把落点移到打开者身前（FindWalkableOffset 半径 1），先按 itemdata 生成被包物品，再推 `unwrapped` 事件并调 onunwrappedfn | 服务端；peek 容器另有支线 | [UNWRAP](../../../raw/current/scripts/actions.lua#L4375-L4380)、[Unwrap 流程](../../../raw/current/scripts/components/unwrappable.lua#L81-L138) |
| 直落点复用（龙舟赛） | `redpouch_yotd_unwrap` 三处直落点：AI 奖袋被击碎、猴子投掷工具落地消失与部署失败 | 服务端；不经 UNWRAP 动作 | [奖袋](../../../raw/current/scripts/prefabs/boatrace_start.lua#L550-L554)、[工具消失](../../../raw/current/scripts/prefabs/boatrace_seastack.lua#L273-L294) |
| 仅依赖声明 | `quagmire_crate_unwrap`／`quagmire_seedpacket_unwrap` 与熔炉 `redpouch_unwrap`：只在 prefabs 依赖列表出现 | 快照内无主模拟生成点（事件数据包模式） | [quagmire_crates](../../../raw/current/scripts/prefabs/quagmire_crates.lua#L52-L56)、[quagmire_seedpackets](../../../raw/current/scripts/prefabs/quagmire_seedpackets.lua#L13-L17)、[熔炉花](../../../raw/current/scripts/prefabs/lavaarena_abigail_flower.lua#L6-L10) |
| 资源 | 独立 build：bundle／gift／redpouch／redpouch_yotr／wetpouch／hermit_bundle／quagmire_*；复用：六名红包变体→`redpouch`、三名种子包→`bundle`；anim 统一 `unwrap` | 动画时长未核验 | [条目组](../../../raw/current/scripts/fx.lua#L1543-L1642) |

## 典型接入

### 自定义包裹接同款开包烟

- 前提与选择：宿主挂 unwrappable 组件并实现 onunwrappedfn；想在战利品落点复用原版视觉，直接落 `bundle_unwrap`（anim 统一 unwrap，无声）。
- 生效与撤销：落点一次性动画，animover 自删；包裹移除与声音需在回调内自理（照抄 bundle.lua 收尾三步：落烟→播声→Remove）。
- 原版依据：[OnUnwrapped 收尾](../../../raw/current/scripts/prefabs/bundle.lua#L297-L302)。

```lua
-- 服务端：自定义包裹工厂里接线（OnUnwrapped 由 Unwrap 以 (inst, pos, doer) 调用）
inst.components.unwrappable:SetOnUnwrappedFn(function(inst, pos, doer)
    SpawnPrefab("bundle_unwrap").Transform:SetPosition(pos:Get())
end)
```

- 调整与验证：换注册名即换烟的贴图；要带原版打开声需自播 `dontstarve/common/together/packaged`。

## 常见陷阱与限制

- **误区：按"物品注册名＋_unwrap"猜烟名对变体失效**：拼名来源是 `inventoryimage or name`——`redpouch_yoth_small`／`hermit_bundle_shells` 两变体显式传 inventoryimage，实际生成 `redpouch_yoth_unwrap`／`hermit_bundle_unwrap`（两变体本身没有专属烟注册）。[拼名](../../../raw/current/scripts/prefabs/bundle.lua#L217-L221)
- **边界：声不在 fx**：十七名条目全部无 sound 字段；打开声由打开者在 OnUnwrapped 收尾播放（皮肤可换 `skin_wrap_sound`）。[收尾](../../../raw/current/scripts/prefabs/bundle.lua#L299-L301)
- **边界：落点 y 压平为 0**：Unwrap 把落点 y 置 0 后才生成战利品与烟——表现贴地，不继承包裹原高度。[落点](../../../raw/current/scripts/components/unwrappable.lua#L82-L83)
- **边界：quagmire 两名不可当普通世界可用**：快照内只有 quagmire_crates／quagmire_seedpackets 的依赖声明，主模拟无生成点；生成逻辑在暴食（Gorge）活动数据包（快照外，活动官名见 [po:379985-379987](../../../raw/current/scripts/languages/chinese_s.po#L379985-L379987)），与熔炉 `redpouch_unwrap` 同属诚实缺口。[声明](../../../raw/current/scripts/prefabs/quagmire_crates.lua#L52-L56)
- **边界：烧焦包裹不出开包烟但有声**：`OnUnwrapped` 先判 `inst.burnt`——烧焦的包裹打开只落 ash，无战利品、无烟；但打开声判断在 burnt 分支之外，doer 有 SoundEmitter 就照播皮肤包裹声或 packaged 声。[烧焦分支](../../../raw/current/scripts/prefabs/bundle.lua#L272-L274)、[声音在分支外](../../../raw/current/scripts/prefabs/bundle.lua#L299-L301)
- **边界：两条战利品路径共用同一收尾烟**：玩家打包的物品走组件 itemdata 分支，红包／湿布袋等预置战利品走 OnUnwrapped 的 loottable 分支；两者都在收尾统一生成同一枚烟。[流程](../../../raw/current/scripts/components/unwrappable.lua#L118-L138)

## 验证记录

- 源码审阅：2026-10-31，Steam buildid 26230496。十七注册配置（含红包资源复用与 yotr 自有资源）、bundle.lua 工厂 17 变体产出 15 烟名、OnUnwrapped 烧焦分支与三步收尾、UNWRAP 动作与 Unwrap 落点流程、龙舟赛三处直落点、暴食／熔炉依赖声明、po 四处核验。未覆盖：动画资源内容、事件数据包内部、皮肤 wrap 声清单、晚加入时序。
- 游戏运行：未运行；开包视觉未实测。

返回 [[reference/prefabs/fx-material-index|采集、制作、碎裂与生成烟]]、[[reference/prefabs/index|Prefab 索引]]。
- 2026-11-01 build 26335394：本页随版本迁移升版（机械沿用／锚点迁移；语义修订如有所见上文），批次与共享依据见 [沿用记录](../../updates/build-26335394/index.md#wiki-沿用记录)（批次-2）。
