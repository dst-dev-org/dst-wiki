---
title: "distancefade"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "按实体相对摄像机的DownVec投影覆盖AnimState透明度，可叠加宿主强度回调，由唤醒接线驱动墙钟更新。"
aliases: ["distancefade", "摄像机投影淡出", "树冠遮挡透明度", "光束透明度", "非玩家距离淡出"]
tags: [component, lifecycle, callback, ui, server-client]
dst_build: "steam-buildid:26230496"
sources: ["components/distancefade.lua", "cameras/followcamera.lua", "entityscript.lua", "mainfunctions.lua", "update.lua", "prefabs/light_rays.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# distancefade

## 快速判断

- 职责与适用：本地视觉组件，用实体位置减摄像机currentpos，再点乘GetDownVec，结果决定AnimState的覆盖乘色alpha；不计算玩家到实体的欧氏距离。默认range=25、fadedist=15。[distancefade.lua:1-6](../../../raw/current/scripts/components/distancefade.lua#L1-L6) [distancefade.lua:25-47](../../../raw/current/scripts/components/distancefade.lua#L25-L47)
- 优先使用：已有AnimState的树冠／光束等视觉宿主需要随摄像机投影淡出时，参照原版非专用服务器安装与墙钟更新接线；不是游戏范围、交互距离或碰撞控制。[light_rays.lua:132-142](../../../raw/current/scripts/prefabs/light_rays.lua#L132-L142)
- 重要副作用：每次更新写OverrideMultColour(1,1,1,alpha)，公式带1.7因子且不clamp结果，可能传入负alpha；不能把它概括成0～1之间的标准线性淡出。[distancefade.lua:38-47](../../../raw/current/scripts/components/distancefade.lua#L38-L47)
- 修改边界：构造只赋参数，不启动更新；OnEntityWake启动墙钟更新，OnEntitySleep停止。无ThePlayer时强制白色alpha=1并提前返回，既不读摄像机也不调额外函数。[distancefade.lua:1-6](../../../raw/current/scripts/components/distancefade.lua#L1-L6) [distancefade.lua:17-29](../../../raw/current/scripts/components/distancefade.lua#L17-L29)

## 关键接口

| 接口／字段 | 签名或约束 | 条件、作用与证据 |
|---|---|---|
| `Setup(range,fadedist)` | 投影阈值与衰减计算长度 | 直接存值，无范围／类型检查，也不立即更新；使用正常公式时由调用方保证fadedist>0，不能把0当禁用开关。[distancefade.lua:8-11](../../../raw/current/scripts/components/distancefade.lua#L8-L11) [distancefade.lua:39-43](../../../raw/current/scripts/components/distancefade.lua#L39-L43) |
| `SetExtraFn(fn)` | 可选 `fn(inst,dt)` | 有ThePlayer时每次更新调用；返回值乘alpha，nil／false回退1，数值0有效。未自行钳制返回值。[distancefade.lua:13-15](../../../raw/current/scripts/components/distancefade.lua#L13-L15) [distancefade.lua:25-47](../../../raw/current/scripts/components/distancefade.lua#L25-L47) |
| `OnWallUpdate(dt)` | 需宿主Transform／AnimState及有效摄像机接口 | 直接完成采样和覆盖乘色；dt仅透传extrafn，内置公式不按时间累计淡入淡出。[distancefade.lua:25-48](../../../raw/current/scripts/components/distancefade.lua#L25-L48) |

## 关键流程

令 `d=(实体位置-TheCamera.currentpos)·GetDownVec()`，`e=extrafn结果或1`。有ThePlayer时，`d<=range`传alpha=e；否则传 `alpha=(1-1.7*min(d-range,fadedist)/fadedist)*e`。常规fadedist>0时，alpha在超出阈值约fadedist/1.7处已到0，在超出完整fadedist后为-0.7e；默认参数且d=40、e=1时即传-0.7。这里只描述Lua传值，未核验渲染器最终如何处理负alpha。[distancefade.lua:31-47](../../../raw/current/scripts/components/distancefade.lua#L31-L47)

FollowCamera.GetDownVec使用headingtarget生成水平向量 `(cos(heading),0,sin(heading))`；因此投影有方向和正负，也不是相机到实体的三维距离或简单高度差。ThePlayer只作为是否进入计算的门槛，不参与距离公式。[followcamera.lua:109-112](../../../raw/current/scripts/cameras/followcamera.lua#L109-L112) [distancefade.lua:25-36](../../../raw/current/scripts/components/distancefade.lua#L25-L36)

墙钟更新的实际入口是OnEntityWake。mainfunctions的实体唤醒／休眠分发会遍历组件调用相应钩子，EntityScript.StartWallUpdatingComponent把组件登记到墙钟更新表，WallUpdate再调用OnWallUpdate(dt)。AddComponent本身不调用唤醒钩子，不能仅凭添加组件就声称已开始更新。[distancefade.lua:17-23](../../../raw/current/scripts/components/distancefade.lua#L17-L23) [mainfunctions.lua:717-721](../../../raw/current/scripts/mainfunctions.lua#L717-L721) [mainfunctions.lua:751-755](../../../raw/current/scripts/mainfunctions.lua#L751-L755) [entityscript.lua:504-523](../../../raw/current/scripts/entityscript.lua#L504-L523) [entityscript.lua:653-667](../../../raw/current/scripts/entityscript.lua#L653-L667) [update.lua:53-68](../../../raw/current/scripts/update.lua#L53-L68)

组件无存读或OnRemoveFromEntity恢复钩子；休眠只停更新，不还原上次覆盖乘色。删除组件时框架停止墙钟更新，也不因此自动恢复颜色；运行时撤销视觉效果需宿主明确处理目标AnimState的恢复策略。[distancefade.lua:17-23](../../../raw/current/scripts/components/distancefade.lua#L17-L23) [entityscript.lua:670-685](../../../raw/current/scripts/entityscript.lua#L670-L685)

## 典型接入

原版`lightrays_canopy`在非专用服务器端添加本组件、Setup(15,25)，并将getintensity注册为额外倍率；getintensity返回宿主当前intensity或1。时间段事件与宿主自己的updateintensity另行改变强度，distancefade只把它乘进本帧投影alpha。[light_rays.lua:40-58](../../../raw/current/scripts/prefabs/light_rays.lua#L40-L58) [light_rays.lua:74-76](../../../raw/current/scripts/prefabs/light_rays.lua#L74-L76) [light_rays.lua:106-142](../../../raw/current/scripts/prefabs/light_rays.lua#L106-L142)

这是本地视觉宿主的接线参照，不是独立可运行demo。自定义宿主需有对应AnimState、有效摄像机环境，并核对实际唤醒／墙钟更新启动时机；修改Setup只改变以后计算参数，不能替代更新注册。[distancefade.lua:8-23](../../../raw/current/scripts/components/distancefade.lua#L8-L23)

## 常见陷阱与限制

- **距离越远一定按圆形范围淡出：** 判断是带方向的投影，摄像机方向改变也会改变结果；不能用range作为玩家周边圆形半径。[distancefade.lua:31-46](../../../raw/current/scripts/components/distancefade.lua#L31-L46)
- **休眠／无玩家会继续执行额外倍率：** 休眠撤墙钟更新，无ThePlayer分支直接设alpha=1并返回；额外函数不能覆盖这个分支。[distancefade.lua:17-29](../../../raw/current/scripts/components/distancefade.lua#L17-L29)

## 验证记录

- 2026-10-26，`steam-buildid:26148036`：审阅组件全文、FollowCamera.GetDownVec、组件墙钟注册／框架唤醒分发／墙钟更新、AddComponent与删除流程、光束宿主安装和强度函数，覆盖公式、负值、额外倍率与更新生命周期。
- 游戏运行：未运行；未测负alpha的渲染器结果、相机切换、组件添加后的首次唤醒或其他覆盖乘色效果叠加。
- 结构检查：`validate_page.py`通过；两页合并执行`validate_source_links.py --strict`，47个行号引用、0错误／0警告。本页无新增Lua示例；机械检查不代替游戏测试。
