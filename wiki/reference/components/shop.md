---
title: shop：标题配置与生成落物
created: 2026-10-27
updated: "2026-10-28"
type: component
summary: 辨认 shop 的有限职责：保存 tab/title 并按 prefab 列表生成实体，不含购买结算。
aliases: [shop, 商店落物, DeliverItems, 商店标题]
tags: [component, loot]
dst_build: "steam-buildid:26230496"
sources: [components/shop.lua]
reliability: verified
validation: source-reviewed
status: active
size_class: simple
---

# shop

## 快速判断

- 职责与适用：组件保存 `tab`／`title` 并提供 `DeliverItems`，生成列表中的 prefab。构造默认 `tab=""`、`title="Shop"`。[shop.lua:1](../../../raw/current/scripts/components/shop.lua#L1
- 优先使用：阅读旧代码时区分配置字段和实体生成；当前快照未找到原版接入，不能据组件名称推导出可用的商店框架。[shop.lua:7](../../../raw/current/scripts/components/shop.lua#L7)、[shop.lua:15](../../../raw/current/scripts/components/shop.lua#L15
- 重要副作用：`DeliverItems` 调用 `SpawnPrefab` 并设置位置，有 Physics 时额外设置速度；不会把物品放入购买者背包。[shop.lua:15](../../../raw/current/scripts/components/shop.lua#L15
- 修改边界：全文没有价格、货币扣除、商品验证、购买者身份、交易回滚、网络同步或存档实现；构造和方法内也没有主模拟端门禁，调用端须自行保证生成权限。[shop.lua:1](../../../raw/current/scripts/components/shop.lua#L1)、[shop.lua:15](../../../raw/current/scripts/components/shop.lua#L15

## 关键接口

| 接口／字段 | 参数或返回 | 作用与约束 |
|---|---|---|
| `SetStartTab(tab)` | 原样保存，无显式返回 | 不做类型校验，不打开 UI；只赋给 `self.tab`。[shop.lua:7](../../../raw/current/scripts/components/shop.lua#L7 |
| `SetTitle(title)` | 原样保存，无显式返回 | 只赋给 `self.title`，组件本身没有文本控件或标题消费者。[shop.lua:11](../../../raw/current/scripts/components/shop.lua#L11 |
| `DeliverItems(items)` | `pairs` 可遍历的 prefab 名值集合；无显式返回 | 先 `assert(items)`，遍历时忽略键，每个值生成一个实体；不把值当数量，也不解析 prefab 到数量的映射。[shop.lua:15](../../../raw/current/scripts/components/shop.lua#L15 |

### 生成位置与速度

- 宿主世界坐标为 `(x,y,z)`，有 Physics 的实体放在 `(x,y+4.5,z)`。`angle` 固定为 `0`，水平速度因此沿世界 `+x`，`z` 分量为零；`sp=math.random()*6+3`，竖直速度为 `math.random()*3+10`。随机的是速度大小，不是环形抛射方向。[shop.lua:17](../../../raw/current/scripts/components/shop.lua#L17)、[shop.lua:25](../../../raw/current/scripts/components/shop.lua#L25
- 无 Physics 的实体放在 `(x-math.random()*6,y,z-math.random()*6)`，只向世界坐标的负 x、负 z 偏移，不是以宿主为中心的对称散布。[shop.lua:31](../../../raw/current/scripts/components/shop.lua#L31
- 每次尝试先打印 `Spawning`；`SpawnPrefab` 返回空则打印 `Could not spawn` 并继续。没有汇总成功列表或撤回先前生成结果；输入集合的处理顺序由 `pairs` 决定。[shop.lua:21](../../../raw/current/scripts/components/shop.lua#L21

## 常见陷阱与限制

- **把投放当购买成功**：本方法只拿宿主坐标生成实体，无购买者、库存、价格或余额参数。需要购买机制时，必须另行设计并核验完整交易链，不能把调用本方法当作完成支付或授权。[shop.lua:15](../../../raw/current/scripts/components/shop.lua#L15
- **把键当数量**：`{log=5}` 会把值 `5` 传入 `SpawnPrefab`；若要生成重复物品，输入必须提供重复的 prefab 名值，组件没有数量展开逻辑。[shop.lua:21](../../../raw/current/scripts/components/shop.lua#L21
- **期待统一抛物效果**：只有 Physics 分支设置上抛速度；无 Physics 时直接设置偏移坐标。[shop.lua:25](../../../raw/current/scripts/components/shop.lua#L25

## 验证记录

- 2026-10-27，`steam-buildid:26148036`：审阅 `components/shop.lua` 全文，核验默认字段、两个 setter、生成失败分支及两类投放坐标／速度。
- 在所选 `scripts` 的 Lua 文件中定向搜索组件添加、`components.shop`、`SetStartTab` 与 `DeliverItems`；仅找到组件自身方法，未找到原版直接接入消费者。因此省略“典型接入”，不提供虚构商店案例；未排除动态拼接名称或引擎侧间接使用。
- 未运行游戏，未核验具体 prefab 的落地碰撞、网络表现或支付流程；本页无完整 Lua 示例，不涉及语法探针。
- `validate_page.py` 与 `validate_source_links.py --strict` 检查通过；检查范围为页面格式、预算与源码链接，不证明运行效果。

- 版本沿用：2026-10-28 原始哈希等价批量沿用-70（26148036→26230496，本页 sources 所列文件原始字节全部未变），页内未决项零标记、符号反查无命中、严格链接通过；批次依据见 [[updates/build-26230496/index#wiki-沿用记录]]。