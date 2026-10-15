---
title: "replayproxy"
created: 2026-10-26
updated: "2026-10-28"
type: component
summary: "在代理实体上保存真实实体GUID与Prefab名称，并提供GUID原值及带GUID前缀的格式化名称读取。"
aliases: ["replayproxy", "entityproxy身份记录", "代理实体GUID", "回放代理名称格式"]
tags: [component, data]
dst_build: "steam-buildid:26230496"
sources: ["components/replayproxy.lua", "prefabs/entityproxy.lua", "input.lua"]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# replayproxy

## 快速判断

- 职责与适用：仅保存两个身份字段并提供setter／getter；不解析GUID为实体，不复制真实实体状态，也不实现录制、回放、输入重定向或网络同步。[replayproxy.lua:1-23](../../../raw/current/scripts/components/replayproxy.lua#L1-L23)
- 优先使用：维护原版entityproxy的身份记录时，区分返回原GUID的getter与返回格式化文本的名称getter；当前Lua快照中只核验到entityproxy安装，未找到这些setter／getter的其他Lua调用点。[entityproxy.lua:1-17](../../../raw/current/scripts/prefabs/entityproxy.lua#L1-L17) [replayproxy.lua:7-21](../../../raw/current/scripts/components/replayproxy.lua#L7-L21)
- 重要副作用：`GetRealEntityPrefabName()`返回的是`GUID - 名称`，并非所存的裸Prefab名称；不能把该返回值直接当作SpawnPrefab参数或Prefab键。[replayproxy.lua:15-21](../../../raw/current/scripts/components/replayproxy.lua#L15-L21)
- 修改边界：默认GUID=nil、名称空串；setter不校验输入。先设置可用于`%d`格式化的GUID，再读取格式化名称，不能把默认构造状态当完整身份记录。[replayproxy.lua:1-21](../../../raw/current/scripts/components/replayproxy.lua#L1-L21)

## 关键接口

| 接口／字段 | 参数／返回 | 条件、作用与证据 |
|---|---|---|
| `SetRealEntityGUID(guid)` | 原样赋值；无显式返回 | 不检查实体是否存在、GUID是否有效或是否属于本代理；格式化getter要求其能供`%d`使用。[replayproxy.lua:7-9](../../../raw/current/scripts/components/replayproxy.lua#L7-L9) [replayproxy.lua:19-21](../../../raw/current/scripts/components/replayproxy.lua#L19-L21) |
| `GetRealEntityGUID()` | 返回保存值，初始nil | 不查Ents、不返回EntityScript、不验证真实实体仍存活。[replayproxy.lua:3](../../../raw/current/scripts/components/replayproxy.lua#L3 [replayproxy.lua:11-13](../../../raw/current/scripts/components/replayproxy.lua#L11-L13) |
| `SetRealEntityPrefabName(name)` | 原样赋值，初始空串 | 仅写入real_entity_prefab_name，不改代理自身prefab或显示资源，也不自动追随真实实体变化。[replayproxy.lua:4](../../../raw/current/scripts/components/replayproxy.lua#L4 [replayproxy.lua:15-17](../../../raw/current/scripts/components/replayproxy.lua#L15-L17) |
| `GetRealEntityPrefabName()` | `string.format("%d - %s",guid,name)`的结果 | 例如记录GUID=123、名称="pigman"时形状为`123 - pigman`；没有独立的裸名称getter。[replayproxy.lua:19-21](../../../raw/current/scripts/components/replayproxy.lua#L19-L21) |

## 典型接入

原版`entityproxy`工厂创建不可休眠实体，只添加Transform、AnimState与replayproxy，设置persists=false和entityproxy标签后返回；本工厂没有Network组件或ismastersim分支，也没有填入真实GUID／名称。身份数据由何处注入，当前已审Lua调用链未提供依据，本页不补造可运行回放demo。[entityproxy.lua:1-17](../../../raw/current/scripts/prefabs/entityproxy.lua#L1-L17)

定向检查input.lua的鼠标实体入口：GetAllEntitiesUnderMouse返回已缓存列表，GetWorldEntityUnderMouse按鼠标启用、实体有效可见及存在Transform筛选后返回hoverinst；这两个入口不读取replayproxy或转换成记录中的GUID。本轮未找到输入层使用上述身份getter的证据，不能据组件名推导鼠标操作会转发给真实实体。[input.lua:263-274](../../../raw/current/scripts/input.lua#L263-L274)

## 常见陷阱与限制

- **构造后立即取格式化名称：** 默认GUID为nil，却直接交给`%d`，有格式参数错误风险；这是由实现推导的输入前提，未运行复现。调用方应先完成身份赋值，并避免不兼容的GUID／名称形状。[replayproxy.lua:1-5](../../../raw/current/scripts/components/replayproxy.lua#L1-L5) [replayproxy.lua:19-21](../../../raw/current/scripts/components/replayproxy.lua#L19-L21)
- **记录会自动随真实实体删除或改名更新：** 本组件没有监听、更新、清理或存读钩子，提供的更改入口仅为两个setter；它不持有真实实体引用，生命周期与有效性由外部调用方负责。[replayproxy.lua:1-23](../../../raw/current/scripts/components/replayproxy.lua#L1-L23)
- **代理身份会保存并在重载后继续回放：** 组件没有OnSave／OnLoad，原版entityproxy还明确persists=false；已审实现不足以支持这样的持久化或回放结论。[entityproxy.lua:9-14](../../../raw/current/scripts/prefabs/entityproxy.lua#L9-L14) [replayproxy.lua:1-23](../../../raw/current/scripts/components/replayproxy.lua#L1-L23)

## 验证记录

- 2026-10-26，`steam-buildid:26148036`：审阅组件与entityproxy工厂全文、输入层两个鼠标实体查询函数，并定向检索身份接口的Lua调用点，覆盖字段默认值、setter／getter返回形状、宿主构造及无自动解析／同步／存读边界。
- 游戏运行：未运行；未验证错误输入的实际报错文本、引擎侧身份写入或快照外的回放工具。未找到Lua调用点不等于证明引擎侧没有消费者。
- 结构检查：`validate_page.py`通过；两页合并执行`validate_source_links.py --strict`，47个行号引用、0错误／0警告。本页无新增Lua示例；机械检查不代替游戏测试。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。