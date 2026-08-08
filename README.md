# xtf-ledger 集成项目

将 tweet-ledger 推文归档功能集成进 [x-tweet-fetcher](https://github.com/ythx-101/x-tweet-fetcher)（xtf），
让 xtf 成为「抓取 + 归档 + 查询」一体化的本地推文库工具。

- 抓取：复用 xtf 3.x 多后端（Nitter / FxTwitter / Browser），抓取即归档、自动去重
- 归档：SQLite 本地推文库（schema 与 tweet-ledger 兼容，`tweet_id` 主键、`INSERT OR IGNORE`）
- 查询：关键词搜索归档库、统计报表，全程离线可用

## 安装 / 运行

```bash
pip install -e .
# 或直接用源码：PYTHONPATH=src python -m xtf.cli ...
```

## 推文库 (Ledger)

### 功能

- **一键归档**：抓取时间线/搜索/列表/回复/单推后自动写入 SQLite，按 `tweet_id` 去重
- **跨后端去重**：同一推文无论来自 Nitter 还是 FxTwitter，第二次抓取自动识别为重复
- **本地推文库**：`raw_json` 全量保留原文，可离线查询、二次加工
- **幂等**：重复抓取不产生重复行；归档失败不阻断抓取（结果信封带 `ledger_error`）

### 用法示例

抓取并归档（时间线）：

```bash
xtf --user YuLin807 --limit 20 --ledger ~/tweets.db
```

查询归档库：

```bash
xtf --ledger ~/tweets.db --query "sop"
```

统计归档库：

```bash
xtf --ledger ~/tweets.db --stats
```

### 示例输出

```jsonc
// 抓取 + 归档
{ "username": "YuLin807", "count": 5,
  "ledger": { "input_records": 5, "inserted": 5, "duplicates": 0, "skipped": 0 } }

// 查询
{ "ledger": "/tmp/xtf-e2e.db", "query": "sop", "count": 2, "tweets": [
    { "tweet_id": "2086132781533544665", "full_text": "今日份文生视频工作流的探索…", ... } ] }

// 统计
{ "ledger": "/tmp/xtf-e2e.db", "stats": {
    "exists": true, "total_tweets": 5, "total_replies": 0, "total_quoted": 0,
    "total_retweeted": 0, "total_with_media": 0, "total_with_urls": 0,
    "first_created_at": "Aug 8, 2026 · 4:16 PM UTC", "last_created_at": "Aug 8, 2026 · 4:55 PM UTC",
    "last_imported_at": "2026-08-08T16:54:54.970208+00:00", "langs": {} } }
```

### 归档表结构（`tweets`）

`tweet_id` (PK) · `created_at` · `full_text` · `lang` · `source_file` · `is_reply` ·
`in_reply_to_status_id` · `retweeted_status_id` · `quoted_status_id` · `urls_json` ·
`media_json` · `raw_json` · `imported_at`

与 tweet-ledger（OpenClaw）schema 兼容，同一 DB 可被两套工具互读。

## 集成测试

端到端真实抓取记录见 [docs/e2e-integration.md](docs/e2e-integration.md)。
