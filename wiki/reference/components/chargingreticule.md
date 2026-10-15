---
title: "chargingreticule"
created: "2026-10-20"
updated: "2026-10-28"
type: component
summary: "本地蓄力方向指示器的跟随、鼠标／手柄投影与旋转插值；不负责蓄力计数、攻击或服务端目标授权。"
aliases: ["手柄蓄力方向转向更快", "蓄力方向指示器", "chargingreticule"]
tags: [component, ui, callback, lifecycle, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/chargingreticule.lua", "prefabs/reticulecharging.lua", "components/aoecharging.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# chargingreticule

## 快速判断

- 职责与适用：挂在本地 FX，LinkToEntity 后跟随 owner 位置，依据输入更新指向；与 [[reference/components/aoecharging]] 的计数／释放、[[reference/components/reticule]] 的通用选点配置区分。[定义](../../../raw/current/scripts/components/chargingreticule.lua#L1-L13) [关联](../../../raw/current/scripts/components/chargingreticule.lua#L41-L72)
- 优先使用：原宿主创建新 FX 后 LinkToEntity(owner)，释放时 Snap；改显示平滑用 ease／smoothing，不改实际技能范围。[宿主创建](../../../raw/current/scripts/components/aoecharging.lua#L92-L122) [显示旋转](../../../raw/current/scripts/components/chargingreticule.lua#L79-L111)
- 重要副作用：关联创建鼠标 move handler 和相机监听；原宿主换 owner 先删除旧 FX。不要多次 Link 同一实例替代重建，旧输入 handler 未先被撤销。[注册](../../../raw/current/scripts/components/chargingreticule.lua#L41-L72) [宿主清旧](../../../raw/current/scripts/components/aoecharging.lua#L92-L105)
- 修改边界：原 reticulecharging 无 Network、不持久保存，只有本地 owner.HUD 时创建；只给服务端执行的模组不能据此提供远端视觉修改。[FX 工厂](../../../raw/current/scripts/prefabs/reticulecharging.lua#L19-L46) [HUD 条件](../../../raw/current/scripts/components/aoecharging.lua#L106-L120)

## 关键接口

| 接口／字段 | 条件、作用与证据 |
|---|---|
| `LinkToEntity(target)` | 要有效 Transform 宿主，保存 owner、同步位置、取鼠标／手柄方向，缺有效方向时用宿主朝向构建 fallback，再监听相机。组件不自动监听 owner 删除。[关联](../../../raw/current/scripts/components/chargingreticule.lua#L41-L72) |
| `GetMouseTargetXZ(x,y)` | ProjectScreenPos 返回 x,z，允许 nil；首建有 fallback，相机更新也判断有效坐标，但 move handler 直接写结果，不能宣称所有投影失败都安全。[投影](../../../raw/current/scripts/components/chargingreticule.lua#L23-L27) [move 分支](../../../raw/current/scripts/components/chargingreticule.lua#L46-L55) [相机分支](../../../raw/current/scripts/components/chargingreticule.lua#L89-L104) |
| `GetControllerTargetXZ()` | 读移动轴／死区，以相机右／下向量构造单位方向，再加本 FX 当前位置；是方向点，不是攻击距离计算。[手柄](../../../raw/current/scripts/components/chargingreticule.lua#L29-L39) |
| `ease`／`smoothing` | 默认 false／6.66，原 FX 开 ease；有 dt 时对最短角差乘 clamp(dt*smoothing,0,1)。手柄相机更新传 dt，鼠标更新传 nil，因此本例不改变鼠标即时转向。[公式](../../../raw/current/scripts/components/chargingreticule.lua#L79-L104) [原配置](../../../raw/current/scripts/prefabs/reticulecharging.lua#L39-L44) |
| `Snap()` | 有 owner 时更新位置并 ForceFacePoint(targetpos)，跳过插值；不是提交攻击／RPC／同步人物朝向。[方法](../../../raw/current/scripts/components/chargingreticule.lua#L107-L111) |
| 清理 | OnRemoveEntity 删除 move handler／相机监听；没有对应 OnRemoveFromEntity，直接热卸组件不能当成等价清理。按原宿主删除整个临时 FX。[删除](../../../raw/current/scripts/components/chargingreticule.lua#L15-L21) [原模式](../../../raw/current/scripts/components/aoecharging.lua#L92-L105) |

## 典型接入

### 原蓄力指示器的手柄旋转更快跟上目标方向

目标模组 `modmain.lua`，需在实际运行视觉的客户端／主机加载，无新资源；不加 ismastersim 门禁，因为对象由本地 HUD 创建。只影响 reticulecharging，不强行改高尔夫等其他 FX。

```lua
AddPrefabPostInit("reticulecharging", function(inst)
    local reticule = inst.components.chargingreticule
    if reticule ~= nil then
        reticule.ease = true
        reticule.smoothing = 12
    end
end)
```

每个新 FX 构造时应用，下一次带 dt 的旋转更新采用新系数；12 是插值系数，不是 12 度／秒或距离。鼠标路径本来不插值，不因此增加鼠标平滑。停用并重载回原配置，不改玩家方向、蓄力帧、命中或网络数据。动画长度缩放由 prefab 的 SetChargeScale 管，不由 smoothing 管。[显示公式](../../../raw/current/scripts/components/chargingreticule.lua#L79-L104) [缩放](../../../raw/current/scripts/prefabs/reticulecharging.lua#L14-L17)

## 常见陷阱与限制

- **任意服务端实体加组件就有指示器**：构造直接访问 AnimState、输入和相机，且需要正确 FX 资源／本地 owner；复用真实本地创建链。[定义](../../../raw/current/scripts/components/chargingreticule.lua#L1-L13) [原创建](../../../raw/current/scripts/components/aoecharging.lua#L106-L120)
- **反复 Link 能自动换主人**：没有先清旧 handler 的步骤；使用原宿主删除旧 FX／新建的模式，避免旧监听遗留。[关联](../../../raw/current/scripts/components/chargingreticule.lua#L41-L72)
- **改 pingprefab 就保证组件自动放提示**：工厂确实赋字段，但本核心组件没有创建 ping 的步骤；提示效果按实际宿主消费核验，不从字段名推断行为。[工厂](../../../raw/current/scripts/prefabs/reticulecharging.lua#L39-L44)

## 验证记录

- 版本沿用（2026-10-25）：「原始哈希等价批量沿用-24」批次机械沿用——所列来源原始字节均未变，结论与行号直接沿用；页内筛查与枢纽反查无命中。共享依据见 [build-26148036 的 Wiki 沿用记录](../../updates/build-26148036/index.md#wiki-沿用记录)。


- 源码审阅：2026-10-20，Steam buildid 25643519；覆盖完整组件、原 FX 工厂与缩放、本地 HUD 创建／清旧、输入／旋转／删除路径。aoecharging／reticule 职责复用既有页；未全面审校所有指示器宿主、手柄热切换、投影失败或攻击／预测链。
- 结构检查：本批格式、来源、链接／行号、大小与登记通过；完整平滑配置例通过已有 Full Moon Lua 5.1 syntax-only。
- 游戏运行：未运行；主机／远端、手柄／鼠标差异、相机旋转、反复创建删除、投影边缘与动画缩放待实测。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。