# Changelog

## 3.1.0 (2026-08-09)

### 新增：推文库 (Ledger)

- `src/xtf/ledger.py`：SQLite 推文归档核心（schema 与 tweet-ledger 兼容）
  - `archive_tweets`：按 `tweet_id` 去重归档（`INSERT OR IGNORE`），写导入回执
  - `query_ledger`：关键词查询（`full_text` 子串匹配，支持 limit/offset）
  - `ledger_stats`：总量/回复/引用/媒体/链接/语言分布/时间范围统计
- CLI 新参数：
  - `--ledger <db>`：抓取模式（`--user`/`--search`/`--list`/`--replies`/`--url`）下自动归档
  - `--ledger <db> --query <term>`：查询归档库
  - `--ledger <db> --stats`：统计归档库
- 兼容性：
  - 无 `--ledger` 时行为与 3.0.0 完全一致
  - 归档失败不阻断抓取（结果带 `ledger_error`）
  - 单推（fxtwitter）dict 缺少 `tweet_id` 时由 CLI 从 URL 注入
- 测试：89+ 用例通过（归档/去重/查询/统计/CLI 集成/回归）

### 修复

- `ledger_stats`：缺库/空表/无 tweets 表时不再返回 None 或抛错，恒返回完整键集

### 文档

- README 新增「推文库 (Ledger)」章节；`docs/e2e-integration.md` 端到端验证记录
