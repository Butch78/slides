# Fig Platform - Quick Reference Guide

## Architecture At-a-Glance

```
┌─────────────────────────────────────────────────────────────────────┐
│                          FIG PLATFORM                                │
└─────────────────────────────────────────────────────────────────────┘

         HOT TIER              CDC TIER              COLD TIER
    ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
    │ PlanetScale/ │────▶│  RisingWave  │────▶│  R2 + Iceberg│
    │  Postgres    │     │     CDC      │     │              │
    │              │     │              │     │  Query       │
    │  < 50ms      │     │  < 1s lag    │     │  Engine      │
    │  90 days     │     │              │     │  Seconds     │
    │  TBs         │     │              │     │  Petabytes   │
    └──────────────┘     └──────────────┘     └──────────────┘
         ↑                                            ↑
         │                                            │
         └────────── Application Queries ────────────┘
```

## Component Comparison Matrix

| Component | Technology | Purpose | Latency | Scale | Cost/GB/mo |
|-----------|-----------|---------|---------|-------|------------|
| **Hot** | PlanetScale/Postgres | Operational queries | < 50ms | TBs | ~$0.30 |
| **CDC** | RisingWave | Real-time replication | < 1s | Streaming | ~$0.05 |
| **Cold** | R2 + Iceberg | Historical analytics | Seconds | Petabytes | $0.015 |

## Query Flow Decision Tree

```
User submits query
       │
       ▼
┌──────────────────┐
│ Query Analyzer   │
└────────┬─────────┘
         │
         ├─ Timestamp < 90 days? ───YES──▶ Route to Postgres (Hot)
         │                                  Return in < 50ms
         │
         └─ Timestamp >= 90 days? ──YES──▶ Route to Query Engine (Cold)
                                            │
                                            ▼
                                     Query Planner
                                            │
                                            ├─ Partition Pruning
                                            ├─ File Pruning  
                                            └─ Row Group Pruning
                                            │
                                            ▼
                                     Distribute to Workers
                                            │
                                            ▼
                                     Return in seconds
```

## Key Metrics Dashboard

### System Health
```
┌─────────────────────────────────────────────┐
│ Metric                 │ Target   │ Current │
├────────────────────────┼──────────┼─────────┤
│ Hot Query Latency (p95)│ < 100ms  │ 47ms    │
│ CDC Lag                │ < 2s     │ 0.4s    │
│ Cold Query Time (avg)  │ < 5s     │ 2.8s    │
│ Pruning Efficiency     │ > 99%    │ 99.7%   │
│ Worker Utilization     │ 60-80%   │ 73%     │
│ Daily Cost             │ Budget   │ $142    │
└────────────────────────┴──────────┴─────────┘
```

### Query Statistics (Example Day)
```
Total Queries:    12,847
  ├─ Hot:         11,204 (87%) │ avg: 38ms  │ p95: 89ms
  └─ Cold:         1,643 (13%) │ avg: 2.9s  │ p95: 8.7s

Data Processed:   3.2 TB
  ├─ Hot:         0.8 TB
  └─ Cold:        2.4 TB (98.8% pruned!)

Cost Breakdown:
  ├─ Hot Storage:  $24.50
  ├─ Cold Storage: $12.80
  ├─ Query Compute: $31.20
  └─ Total:        $68.50
```

## Technology Stack Detail

### Hot Tier (PlanetScale/Postgres)
```
┌─────────────────────────────────┐
│ PlanetScale/Postgres            │
├─────────────────────────────────┤
│ Version:  Postgres 15+          │
│ Config:   Multi-region replicas │
│ Writes:   Primary only          │
│ Reads:    Round-robin replicas  │
│ Backup:   Point-in-time (7d)    │
│ Schema:   Versioned migrations  │
└─────────────────────────────────┘
```

**Key Tables:**
- `events` - User activity events
- `logs` - Application logs
- `metrics` - Time-series metrics

**Retention Policy:**
- Keep last 90 days
- Older data in cold tier

### CDC Tier (RisingWave)
```
┌─────────────────────────────────┐
│ RisingWave CDC                  │
├─────────────────────────────────┤
│ Deploy:   Cloudflare Containers │
│ Source:   Postgres WAL          │
│ Sink:     R2 (Parquet)          │
│ Format:   Apache Parquet        │
│ Batch:    5 min or 100MB        │
│ Commit:   Iceberg atomic        │
└─────────────────────────────────┘
```

**Streaming Jobs:**
```sql
-- Example RisingWave Job
CREATE SINK events_to_iceberg AS
SELECT 
    event_id,
    user_id,
    event_type,
    timestamp,
    properties
FROM events_cdc
WITH (
    connector = 'iceberg',
    table = 'events',
    catalog = 'r2',
    format = 'parquet'
);
```

### Cold Tier (R2 + Iceberg + Query Engine)
```
┌─────────────────────────────────┐
│ Cloudflare R2                   │
├─────────────────────────────────┤
│ Bucket:   fig-cold-storage      │
│ Region:   Auto (global edge)    │
│ Egress:   $0 (free)             │
│ Class:    Standard              │
└─────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────┐
│ Apache Iceberg                  │
├─────────────────────────────────┤
│ Version:  1.5+                  │
│ Catalog:  R2-native             │
│ Format:   Parquet (Snappy)      │
│ Partition: day(timestamp)       │
│ Sort:     timestamp             │
└─────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────┐
│ Query Engine                    │
├─────────────────────────────────┤
│ Planner:  Metadata pruning      │
│ Workers:  Cloudflare Workers    │
│ Engine:   Apache DataFusion     │
│ Format:   Arrow IPC             │
└─────────────────────────────────┘
```

## Query Examples with Explanations

### Example 1: Recent Data (Hot Path)
```sql
-- Query: Last 24 hours of events
SELECT event_type, COUNT(*) as count
FROM events
WHERE timestamp >= NOW() - INTERVAL '24 hours'
GROUP BY event_type
ORDER BY count DESC;

-- Execution:
-- ✓ Timestamp < 90 days → Route to Postgres
-- ✓ Index on timestamp used
-- ✓ Parallel aggregate on replicas
-- ✓ Result: 34ms
```

### Example 2: Historical Data (Cold Path)
```sql
-- Query: Full year analysis
SELECT 
    DATE_TRUNC('day', timestamp) as day,
    COUNT(*) as events,
    COUNT(DISTINCT user_id) as unique_users
FROM events
WHERE timestamp BETWEEN '2024-01-01' AND '2024-12-31'
  AND event_type = 'purchase'
GROUP BY day
ORDER BY day;

-- Execution:
-- ✓ Timestamp >= 90 days → Route to Query Engine
-- ✓ Manifest list: Skip Q1/Q2/Q3/Q4 2023, 2025
-- ✓ Manifest files: Read 2024 manifests
-- ✓ Column stats: Skip files where event_type != 'purchase'
-- ✓ Result: 2147 files → 8 files after pruning
-- ✓ Distribute 8 files × row groups to workers
-- ✓ Result: 1.8s, processed 4.2GB out of 850GB
```

### Example 3: Complex Filter (Cold Path with Heavy Pruning)
```sql
-- Query: Specific user errors
SELECT *
FROM logs
WHERE timestamp > '2024-06-01'
  AND timestamp < '2024-06-02'
  AND status = 500
  AND region = 'us-east'
  AND user_id = 'user_12345'
ORDER BY timestamp DESC
LIMIT 10;

-- Execution:
-- ✓ Partition pruning: Keep only 2024-06-01 partition
-- ✓ File pruning (stats):
--   • Skip files where status ∈ [200, 404]
--   • Skip files where region != 'us-east'
-- ✓ Row group pruning:
--   • Skip row groups not matching filters
-- ✓ Early termination: Stop after finding 10 results
-- ✓ Result: 847ms, processed 12MB out of 48GB
-- ✓ Pruning efficiency: 99.975%
```

## Pruning Examples Visualized

### Scenario: Find errors in one day from 10TB dataset

```
Total Data: 10,000 GB (10TB)
Total Files: 10,000 files (1GB each)
Query: WHERE day = '2024-06-15' AND status = 500

Step 1: Partition Pruning (Manifest List)
┌────────────────────────────────────────┐
│ Total Manifests: 365 (one per day)    │
│ Filter: day = '2024-06-15'            │
│ Result: 1 manifest                     │
│ Data Remaining: 27.4 GB                │
│ Pruned: 99.7%                          │
└────────────────────────────────────────┘

Step 2: File Pruning (Manifest Files)
┌────────────────────────────────────────┐
│ Files in Manifest: 27 files           │
│ Filter: status = 500                   │
│ Column Stats: min/max status per file  │
│ Result: 3 files match                  │
│ Data Remaining: 3.0 GB                 │
│ Pruned: 99.97%                         │
└────────────────────────────────────────┘

Step 3: Row Group Pruning (Parquet)
┌────────────────────────────────────────┐
│ Row Groups: 300 (100 per file)        │
│ Filter: status = 500                   │
│ Row Group Stats: min/max per group    │
│ Result: 8 row groups match            │
│ Data Remaining: 80 MB                  │
│ Pruned: 99.999%                        │
└────────────────────────────────────────┘

Final: Read 80MB, found 1,247 matching rows
Time: 0.8 seconds
Cost: $0.008
```

## Cost Comparison Calculator

### Scenario: 100TB Data, 1000 Queries/Day, 30 Days

#### Fig Platform
```
Storage:
  Hot (5TB × 30 days):       $450/month
  Cold (100TB):              $1,500/month
  
Query Compute:
  Hot (500 queries × 30):    $0 (included)
  Cold (500 queries × 30):   $1,500 (avg 10GB/query)
  
Total: $3,450/month
```

#### Snowflake
```
Storage:
  On-demand (100TB):         $2,300/month
  
Compute:
  Medium warehouse (24/7):   $8,760/month
  Query credits (1000/day):  $6,000/month
  
Egress (estimated 10TB):     $900/month

Total: $17,960/month
```

#### Databricks
```
Storage:
  S3 (100TB):                $2,300/month
  
Compute:
  Cluster (4 nodes, 24/7):   $10,944/month
  DBU consumption:           $4,500/month
  
Egress (estimated 10TB):     $900/month

Total: $18,644/month
```

#### Savings
```
Fig vs Snowflake:  80.8% cheaper  ($14,510/month saved)
Fig vs Databricks: 81.5% cheaper  ($15,194/month saved)
```

## Common Queries Reference

### Analytics Queries (Cold)
```sql
-- Daily active users
SELECT DATE(timestamp), COUNT(DISTINCT user_id)
FROM events
WHERE timestamp >= '2024-01-01'
GROUP BY DATE(timestamp);

-- Funnel analysis
SELECT 
    event_type,
    COUNT(*) as total,
    COUNT(DISTINCT user_id) as unique_users
FROM events
WHERE timestamp >= '2024-01-01'
GROUP BY event_type;

-- Cohort retention
SELECT 
    DATE(first_seen) as cohort,
    DATE(timestamp) as activity_date,
    COUNT(DISTINCT user_id) as active_users
FROM events
JOIN (
    SELECT user_id, MIN(timestamp) as first_seen
    FROM events
    GROUP BY user_id
) cohorts USING (user_id)
GROUP BY cohort, activity_date;
```

### Operational Queries (Hot)
```sql
-- Recent user activity
SELECT *
FROM events
WHERE user_id = 'user_12345'
  AND timestamp > NOW() - INTERVAL '24 hours'
ORDER BY timestamp DESC;

-- Real-time dashboard
SELECT 
    event_type,
    COUNT(*) as count,
    AVG(latency_ms) as avg_latency
FROM events
WHERE timestamp > NOW() - INTERVAL '1 hour'
GROUP BY event_type;

-- Error monitoring
SELECT *
FROM logs
WHERE level = 'ERROR'
  AND timestamp > NOW() - INTERVAL '5 minutes'
ORDER BY timestamp DESC
LIMIT 100;
```

## Troubleshooting Guide

### Issue: Query Slow on Cold Storage

**Check:**
```sql
-- See query plan and pruning stats
EXPLAIN ANALYZE
SELECT ...;
```

**Common Causes:**
1. Poor partition key choice
2. Missing filters on partition columns
3. Wide date ranges
4. No LIMIT clause for large result sets

**Solutions:**
1. Add partition filters (e.g., `WHERE day = '2024-01-15'`)
2. Use column stats (filter on indexed columns)
3. Add LIMIT to enable early termination
4. Consider re-partitioning data

### Issue: High Query Costs

**Check:**
```sql
-- View query statistics
SELECT 
    query_id,
    bytes_scanned,
    bytes_pruned,
    execution_time_ms,
    cost_usd
FROM query_log
WHERE date = CURRENT_DATE
ORDER BY cost_usd DESC
LIMIT 10;
```

**Common Causes:**
1. Full table scans
2. Poor pruning efficiency
3. Large result sets

**Solutions:**
1. Add more selective filters
2. Use partition columns in WHERE
3. Add LIMIT clauses
4. Pre-aggregate into summary tables

### Issue: CDC Lag Increasing

**Check:**
```bash
# Monitor CDC lag
curl https://api.fig.io/v1/cdc/lag
```

**Common Causes:**
1. High write volume
2. Large transactions
3. Network issues

**Solutions:**
1. Increase RisingWave parallelism
2. Batch smaller transactions
3. Check Cloudflare connectivity

## Monitoring Commands

### Health Check
```bash
# Overall system health
curl https://api.fig.io/v1/health

# Component status
curl https://api.fig.io/v1/status/hot
curl https://api.fig.io/v1/status/cdc
curl https://api.fig.io/v1/status/cold
```

### Metrics
```bash
# Query statistics
curl https://api.fig.io/v1/metrics/queries?hours=24

# Storage metrics
curl https://api.fig.io/v1/metrics/storage

# Cost tracking
curl https://api.fig.io/v1/metrics/cost?date=2024-09-25
```

### Logs
```bash
# Query logs
curl https://api.fig.io/v1/logs/queries?limit=100

# CDC logs
curl https://api.fig.io/v1/logs/cdc?level=error

# Worker logs
curl https://api.fig.io/v1/logs/workers?worker_id=w-12345
```

## Best Practices

### Schema Design
- ✅ Use timestamp for partition key
- ✅ Include high-cardinality columns early (e.g., user_id, event_type)
- ✅ Denormalize for query performance
- ✅ Use appropriate data types (INT over VARCHAR for IDs)

### Query Patterns
- ✅ Always filter on partition columns
- ✅ Use LIMIT for top-N queries
- ✅ Aggregate in query vs. application code
- ✅ Use column projection (SELECT specific columns)

### Data Management
- ✅ Compact Iceberg tables weekly
- ✅ Expire old snapshots
- ✅ Monitor hot storage growth
- ✅ Archive old partitions

### Cost Optimization
- ✅ Monitor query patterns
- ✅ Pre-aggregate common queries
- ✅ Use hot storage for recent data
- ✅ Optimize partition strategy
- ✅ Enable query result caching

## Next Steps

1. **Proof of Concept**
   - Set up development environment
   - Load sample data
   - Test query patterns
   - Measure performance

2. **Pilot Deployment**
   - Deploy to staging
   - Migrate subset of production data
   - Run parallel queries (Fig vs. existing)
   - Validate cost savings

3. **Production Migration**
   - Plan cutover strategy
   - Migrate data tier by tier
   - Update application clients
   - Monitor and optimize

4. **Optimization**
   - Tune partition strategy
   - Create summary tables
   - Set up alerts
   - Implement cost controls
