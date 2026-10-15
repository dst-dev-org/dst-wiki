# 本地模组参照复核

复核日期：2026-10-17。用于追溯 container／edible／fueled／preserver／stackable／tool／trader／useableitem 八页的模组片段观察；只在需要核对这些来源时读取，日常机制查询使用页面已有原版证据。

## 来源与版本边界

以下输入未随知识库分发，未确认可下载的公开地址；本机路径只是本次输入位置，不是其他用户的配置约定。哈希标识本次实际文件，不能凭它重新取得源码。异机复核须取得匹配文件并核对相对路径／SHA-256；文件不可用时只将模组观察作为设计线索。

| 输入 | 本次根目录 | modinfo 版本线索 | Git 状态 |
|---|---|---|---|
| MCW | `D:/Code/MCW` | 静态声明 `1.2.2.14` | 无 Git 元数据，无提交编号 |
| Sora | `D:/Code/Sora` | 静态声明 `11.81`，文件另有 `-dev` 拼接分支 | 无 Git 元数据，无提交编号 |
| DSTKisaki | `D:/Program Files (x86)/Steam/steamapps/common/Don't Starve Together/mods/DSTKisaki` | 声明 `1.0.0`，modinfo 有本地改动 | HEAD `67cef55096db51e75f3ee0c62a4ab6205844ec49`；本表两份 Lua 与该提交一致 |

版本声明不保证输入完整一致，也不是运行验证；以下实际文件哈希优先于模组名称或版本字符串。未运行 modinfo／Lua，不据代码推定当前启用配置或安装的游戏版本。

## 文件指纹

SHA-256 按原文件完整字节计算。行号基于本次文件，不是未来版本的稳定锚点；没有复制模组源码或资源。

| 输入 | 相对路径 | 核对行段 | SHA-256 |
|---|---|---|---|
| MCW | `modinfo.lua` | 版本声明及分支 | `b6a07863b0c3ab595250e8f8bef35b092006956dce2b77369e7176ffa20cdeaf` |
| Sora | `modinfo.lua` | 版本声明及分支 | `1d9da7c3483023d8f888ffcc400346b82ca3328f39db7eb369715143f08e725f` |
| DSTKisaki | `modinfo.lua` | 版本声明及分支 | `2da57de4f612ac09421b2d12a51fcfc4a804fb3c8960d20a0678ae3d7b2a2a94` |
| MCW | `scripts/prefabs/mcw_gembox.lua` | 214–230 | `22fb1e9f8b6eaaa255f11423d22f057e13e766d10bc6b8e574f5739a668e6cd9` |
| MCW | `scripts/prefabs/mcw_fruit_tea.lua` | 86–100 | `50f7a1a66898827a2098bfe0fd084eeba1211e97f28d72acc188eae7e8bcbf6c` |
| MCW | `scripts/prefabs/mcw_snack_sack.lua` | 85–86 | `b47c85dff6b4005403e661567324da08d8183321602323b3177cfdd39de61b4a` |
| MCW | `scripts/prefabs/mcw_lollipopstaff.lua` | 53–78、129–176 | `0a738cdcad9cd20391f37eb467cfd3651e73940addb6e77d2a3d8f0985988dd7` |
| MCW | `scripts/prefabs/mcw_magic_armor.lua` | 32–42、104–108 | `92dba18534190b475e706dcdb1aa3e3a75b4010244f03b3963c7d1537471b173` |
| Sora | `scripts/prefabs/sora3chest.lua` | 66–92 | `6c6b27458d6095081748eef2000622f215c7123e91355efda1767e57c6ed8c40` |
| Sora | `scripts/prefabs/sorafoods.lua` | 154–162 | `3e9f88a3398ae173301385fa1b6baeb388a6b4ea709a4c8d9345a3e7f3ab58a4` |
| Sora | `scripts/prefabs/sora2amulet.lua` | 84–87、247–254 | `cff6dcf1a678c74e062822d3988535e6045c0ea35034d6d29156d5010649bae2` |
| Sora | `scripts/prefabs/sora2base.lua` | 623–655 | `dc2a1d71081f16bb8835b536242379c95a6e47682edcb412cd65a39db20b7566` |
| Sora | `scripts/prefabs/sora2sword.lua` | 53–66 | `4958b5633a73d7c57e16415332cedc697b5e0e8c0bac3e7a47fb4c353151ebee` |
| DSTKisaki | `scripts/prefabs/kisaki_items.lua` | 45–59 | `a082cbc4b77195a2de28b962a8968bee1aeb8c67046b3dd6429e784b4af1a6a1` |
| DSTKisaki | `scripts/prefabs/kisaki_tool.lua` | 17–28、323–359、401–430 | `ceadc75f21ac6d9ea65654d57ad1eedb0db7720025885250b00a0703119ed582` |

## 观察结论与未覆盖范围

- 所列片段与八页记录一致：组件组合与模式切换确实存在；Sora 的 stonefn 与同文件建筑工厂分开，文件共现不代表同一 Prefab 的能力。
- MCW 的 `cantrader` 绑定确实存在，但本轮未展开其外部 Hook；DSTKisaki 的修复回调也不能证明当前原版删除材料后的生命周期安全。保留页面中的原版 GIVE／trader 边界，不将模组写法升级为推荐交易实现。
- 原版机制仍以页面注明的 Steam buildid 25643519 源码为依据。模组声明版本、片段能读取或解析通过均不证明该模组适配此 build；没有游戏运行、联机、存档回归或完整 Hook 检视。
