# 明细：资源与本地化（M）

build 26335394 → 26474873。条目按五字段契约，编号 M-序号 在本报告内唯一。资源文案类审阅优先级低，但字符串／皮肤数据变化如实登记。

| 编号 | 变化 | 影响接入／条件 | 证据位置 | 核验状态 |
|---|---|---|---|---|
| M-1 | `strings.lua` `STALKER_NPC_PERFORM_CORRUPT[1]` 句尾补句号（"…as it may be"→"…as it may be."）。1:1 替换，全文件无行号偏移。 | 按完整字符串匹配该台词的模组（比对用）需跟改；按键索引读取不受影响。 | 旧:scripts/strings.lua:7547-7553；新:scripts/strings.lua:7547-7553 | 已核验 |
| M-2 | `languages/strings.pot` 模板同步 M-1 的 msgid 变化（`STRINGS.STALKER_NPC_PERFORM_CORRUPT.1`，1:1 替换）。 | 翻译工具链：以旧 msgid 锁定条目的 .po 合并需重新对齐；运行时无影响。 | 旧:scripts/languages/strings.pot:419915-419919；新:scripts/languages/strings.pot:419915-419919 | 已核验 |
