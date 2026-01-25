---
name: clickhouse-io
description: ClickHouse 数据库模式、查询优化、分析和高性能分析工作负载的数据工程最佳实践。
---

# ClickHouse 分析模式

用于高性能分析和数据工程的 ClickHouse 特定模式。

## 概述

ClickHouse 是一个面向列的数据库管理系统 (DBMS)，用于在线分析处理 (OLAP)。它针对大型数据集上的快速分析查询进行了优化。

**关键特性：**
- 面向列存储
- 数据压缩
- 并行查询执行
- 分布式查询
- 实时分析

## 表设计模式

### MergeTree 引擎（最常见）

```sql
CREATE TABLE markets_analytics (
    date Date,
    market_id String,
    market_name String,
    volume UInt64,
    trades UInt32,
    unique_traders UInt32,
    avg_trade_size Float64,
    created_at DateTime
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(date)
ORDER BY (date, market_id)
SETTINGS index_granularity = 8192;
```

### ReplacingMergeTree（去重）

```sql
CREATE TABLE user_events (
    event_id String,
    user_id String,
    event_type String,
    timestamp DateTime,
    properties String
) ENGINE = ReplacingMergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (user_id, event_id, timestamp)
PRIMARY KEY (user_id, event_id);
```

### AggregatingMergeTree（预聚合）

```sql
CREATE TABLE market_stats_hourly (
    hour DateTime,
    market_id String,
    total_volume AggregateFunction(sum, UInt64),
    total_trades AggregateFunction(count, UInt32),
    unique_users AggregateFunction(uniq, String)
) ENGINE = AggregatingMergeTree()
PARTITION BY toYYYYMM(hour)
ORDER BY (hour, market_id);
```

## 查询优化模式

### 高效过滤

```sql
-- ✅ 好：首先使用索引列
SELECT *
FROM markets_analytics
WHERE date >= '2025-01-01'
  AND market_id = 'market-123'
  AND volume > 1000
ORDER BY date DESC
LIMIT 100;

-- ❌ 坏：首先过滤非索引列
SELECT *
FROM markets_analytics
WHERE volume > 1000
  AND market_name LIKE '%election%'
  AND date >= '2025-01-01';
```

### 聚合

```sql
-- ✅ 好：使用 ClickHouse 特定的聚合函数
SELECT
    toStartOfDay(created_at) AS day,
    market_id,
    sum(volume) AS total_volume,
    count() AS total_trades,
    uniq(trader_id) AS unique_traders,
    avg(trade_size) AS avg_size
FROM trades
WHERE created_at >= today() - INTERVAL 7 DAY
GROUP BY day, market_id
ORDER BY day DESC, total_volume DESC;
```

## 数据插入模式

### 批量插入（推荐）

```typescript
// ✅ 批量插入（高效）
async function bulkInsertTrades(trades: Trade[]) {
  const values = trades.map(trade => `(
    '${trade.id}',
    '${trade.market_id}',
    '${trade.user_id}',
    ${trade.amount},
    '${trade.timestamp.toISOString()}'
  )`).join(',')

  await clickhouse.query(`
    INSERT INTO trades (id, market_id, user_id, amount, timestamp)
    VALUES ${values}
  `).toPromise()
}
```

### 流式插入

```typescript
// 用于连续数据摄取
async function streamInserts() {
  const stream = clickhouse.insert('trades').stream()

  for await (const batch of dataSource) {
    stream.write(batch)
  }

  await stream.end()
}
```

## 物化视图

### 实时聚合

```sql
-- 创建用于每小时统计的物化视图
CREATE MATERIALIZED VIEW market_stats_hourly_mv
TO market_stats_hourly
AS SELECT
    toStartOfHour(timestamp) AS hour,
    market_id,
    sumState(amount) AS total_volume,
    countState() AS total_trades,
    uniqState(user_id) AS unique_users
FROM trades
GROUP BY hour, market_id;
```

## 性能监控

### 查询性能

```sql
-- 检查慢查询
SELECT
    query_id,
    user,
    query,
    query_duration_ms,
    read_rows,
    read_bytes,
    memory_usage
FROM system.query_log
WHERE type = 'QueryFinish'
  AND query_duration_ms > 1000
  AND event_time >= now() - INTERVAL 1 HOUR
ORDER BY query_duration_ms DESC
LIMIT 10;
```

## 常见分析查询

### 时间序列分析

```sql
-- 每日活跃用户
SELECT
    toDate(timestamp) AS date,
    uniq(user_id) AS daily_active_users
FROM events
WHERE timestamp >= today() - INTERVAL 30 DAY
GROUP BY date
ORDER BY date;
```

### 漏斗分析

```sql
-- 转化漏斗
SELECT
    countIf(step = 'viewed_market') AS viewed,
    countIf(step = 'clicked_trade') AS clicked,
    countIf(step = 'completed_trade') AS completed,
    round(clicked / viewed * 100, 2) AS view_to_click_rate,
    round(completed / clicked * 100, 2) AS click_to_completion_rate
FROM (
    SELECT
        user_id,
        session_id,
        event_type AS step
    FROM events
    WHERE event_date = today()
)
GROUP BY session_id;
```

## 数据流水线模式

### ETL 模式

```typescript
// 提取、转换、加载
async function etlPipeline() {
  // 1. 从源提取
  const rawData = await extractFromPostgres()

  // 2. 转换
  const transformed = rawData.map(row => ({
    date: new Date(row.created_at).toISOString().split('T')[0],
    market_id: row.market_slug,
    volume: parseFloat(row.total_volume),
    trades: parseInt(row.trade_count)
  }))

  // 3. 加载到 ClickHouse
  await bulkInsertToClickHouse(transformed)
}
```

## 最佳实践

### 1. 分区策略
- 按时间分区（通常为月或日）
- 避免过多分区（性能影响）
- 对分区键使用 DATE 类型

### 2. 排序键
- 首先放置最常过滤的列
- 考虑基数（高基数优先）
- 顺序影响压缩

### 3. 数据类型
- 使用最小合适的类型（UInt32 vs UInt64）
- 对重复字符串使用 LowCardinality
- 对分类数据使用 Enum

### 4. 避免
- SELECT *（指定列）
- FINAL（在查询前合并数据）
- 太多 JOIN（为分析非规范化）
- 频繁的小批量插入（改为批量）

**记住**：ClickHouse 擅长分析工作负载。为您的查询模式设计表，批量插入，并利用物化视图进行实时聚合。
