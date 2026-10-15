---
title: "wanderingroutefollower"
created: "2026-10-28"
updated: "2026-10-28"
type: component
summary: "未完成的世界级漫游路线跟随组件：登记并插值路线点、周期选取路线，供巡游类实体规划期望位置；原版无任何挂载或读取者。"
aliases: ["漫游路线跟随", "路线插值巡游", "wanderingroutefollower"]
tags: [component, mechanism, lifecycle]
dst_build: "steam-buildid:26230496"
size_class: simple
sources: ["components/wanderingroutefollower.lua"]
reliability: verified
validation: source-reviewed
status: active
---

# wanderingroutefollower

## 快速判断

- 职责与适用：预期职责是让宿主沿登记的闭合路线巡游：`DefineRoute` 登记路线点并按 8 单位间距插值（components/wanderingroutefollower.lua:11-12、37-68），`Tick` 每 10 秒经 `routepickerfn` 选路（lines 11、119-125）。
- 优先使用：无。全库 grep `wanderingroutefollower` 仅命中组件自身文件，无 `AddComponent`、无任何读取者；本页按源码实际状态记录，不是可用接入入口。
- 重要副作用：`InterpolateRoute` 会生成 `purplemooneye` 调试实体标记插值点（lines 64-66）；`Tick` 内含 `print` 调试输出（lines 120、124）。
- 修改边界：仅 mastersim（断言 line 7）；`OnUpdate` 不会被自动启动——构造器没有 `StartUpdatingComponent`，必须由宿主显式驱动（lines 4-25、128-138）；有存读钩子（lines 141-165）。

## 关键接口

| 接口／字段 | 签名或已核验类型 | 条件、作用与证据 |
|---|---|---|
| `DefineRoute(routename, xzpositions)` | `({x,z} 数组或 nil)` | 保存路线并立即插值（lines 70-80） |
| `ForgetRoute(routename)` | `(string)` | 删除路线与插值缓存（lines 82-85） |
| `SetCurrentRoute(routename)` | `(string或nil)` | 切换路线并找最近插值点为起点；首次调用时 `routename` 字段为 nil（lines 87-105，字段仅注释声明 line 24） |
| `SetRoutePickerFn(fn)` | `(fn(inst)->routename)` | Tick 时自动选路（lines 107-109、121-123） |
| `GetDesiredPosition()` | `-> x, y, z` | 未完成：无路线时返回当前位置，有路线时硬编码返回 `0, 0, 0` 占位（lines 111-117） |
| `STATES` | `{IDLE=0, FOLLOWING=1}` | 存档时枚举转字符串、载入还原（lines 14-17、147-152、162） |
| `IsValidPointFn` | 内建点过滤函数 | 排除隐形地皮与洞口附近（lines 29-35），当前仅定义未被调用 |
| `OnSave`／`OnLoad` | 存 routes／routename／routeindex／routestate | routes 表整体序列化（lines 141-165） |

## 常见陷阱与限制

- **闭合插值会报错：** `InterpolateRoute` 处理末点到首点时引用了未定义变量 `xzposition[1]`（应为 `xzpositions[1]`，line 55）。调用 `DefineRoute` 传入两点以上路线即触发运行时错误，本组件当前无法完整走通插值流程。
- **期望位置是占位实现：** `GetDesiredPosition` 在有路线时返回 `0, 0, 0`（lines 111-117），`Tick` 也只做打印；跟随移动逻辑未实装，接入方拿不到可用航点。
- **不会自行更新：** 无 `StartUpdatingComponent`，`accumulator`（lines 20、128-135）只有在宿主主动调用 `OnUpdate`／`LongUpdate` 时才累计。
- 结论：这是仓库内遗留的半成品（疑似月岛漫游路线的中间产物），复制使用前必须修复上述三处；期待完整巡游应改用现有 AI／移动组件组合。

## 验证记录

- 源码审阅：2026-10-28，steam-buildid:26230496。通读 components/wanderingroutefollower.lua 全文（171 行）；全库 grep 确认无挂载者与读取者。未运行游戏。
- 结构检查：validate_page.py / validate_source_links.py 通过（见批级自验）。
