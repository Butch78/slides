# Fig Platform - Visual Query Flow

## Complete Query Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                         USER SUBMITS QUERY                           │
│  "SELECT * FROM events WHERE day='2024-06-15' AND status=500"       │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      QUERY ROUTER / ANALYZER                         │
└────────────────────────────┬────────────────────────────────────────┘
                             │
           ┌─────────────────┴─────────────────┐
           │                                   │
           ▼ Recent data?                      ▼ Historical data?
    ┌──────────────┐                    ┌──────────────┐
    │  HOT PATH    │                    │  COLD PATH   │
    │  (Postgres)  │                    │ (Query Eng.) │
    └──────┬───────┘                    └──────┬───────┘
           │                                   │
           ▼                                   ▼
    Return in <50ms              ┌────────────────────────┐
                                 │   QUERY PLANNER        │
                                 │  (Metadata Pruning)    │
                                 └────────┬───────────────┘
                                          │
                    ┌─────────────────────┼─────────────────────┐
                    │                     │                     │
                    ▼                     ▼                     ▼
            ┌───────────────┐    ┌───────────────┐    ┌───────────────┐
            │ 1. PARTITION  │    │ 2. FILE       │    │ 3. ROW GROUP  │
            │    PRUNING    │───▶│    PRUNING    │───▶│    PRUNING    │
            │               │    │               │    │               │
            │ Manifest List │    │ Manifest      │    │ Parquet       │
            │ Stats         │    │ Files         │    │ Footers       │
            └───────────────┘    └───────────────┘    └───────────────┘
                    │                     │                     │
                    │                     │                     │
                    └─────────────────────┴─────────────────────┘
                                          │
                                          ▼
                              ┌────────────────────────┐
                              │   WORK UNITS QUEUE     │
                              │   (Row groups to read) │
                              └────────┬───────────────┘
                                       │
                    ┌──────────────────┼──────────────────┐
                    │                  │                  │
                    ▼                  ▼                  ▼
            ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
            │ WORKER 1     │  │ WORKER 2     │  │ WORKER N     │
            │ DataFusion   │  │ DataFusion   │  │ DataFusion   │
            └──────┬───────┘  └──────┬───────┘  └──────┬───────┘
                   │                  │                  │
                   │                  │                  │
                   └──────────────────┴──────────────────┘
                                      │
                                      ▼
                          ┌────────────────────────┐
                          │  QUERY COORDINATOR     │
                          │  (Aggregate results)   │
                          └────────┬───────────────┘
                                   │
                                   ▼
                          ┌────────────────────────┐
                          │   RETURN TO USER       │
                          │   (Seconds elapsed)    │
                          └────────────────────────┘
```

## Metadata Pruning: Step by Step

### Starting Point
```
Total Data:    10,000 files × 1GB = 10TB
Total Rows:    100 billion rows
Query:         WHERE day='2024-06-15' AND status=500
```

### Step 1: Partition Pruning (Manifest List)

```
┌────────────────────────────────────────────────────────┐
│                    MANIFEST LIST                        │
│  (One entry per day - 365 manifests in this example)   │
├────────────────────────────────────────────────────────┤
│ Manifest      │ Partition  │ Files │ Min Date   │ Max Date   │
│ manifest-001  │ 2024-01-01 │  27   │ 2024-01-01 │ 2024-01-01 │ ❌ SKIP
│ manifest-002  │ 2024-01-02 │  27   │ 2024-01-02 │ 2024-01-02 │ ❌ SKIP
│ ...           │ ...        │  ...  │ ...        │ ...        │ ❌ SKIP
│ manifest-166  │ 2024-06-15 │  27   │ 2024-06-15 │ 2024-06-15 │ ✅ KEEP
│ ...           │ ...        │  ...  │ ...        │ ...        │ ❌ SKIP
│ manifest-365  │ 2024-12-31 │  27   │ 2024-12-31 │ 2024-12-31 │ ❌ SKIP
└────────────────────────────────────────────────────────┘

Result: 1 manifest (out of 365)
Data Remaining: 27GB (out of 10TB)
Pruned: 99.7%
```

### Step 2: File Pruning (Manifest File)

```
┌──────────────────────────────────────────────────────────────────┐
│              MANIFEST FILE for 2024-06-15                         │
│  (Tracks all Parquet files for this partition)                   │
├──────────────────────────────────────────────────────────────────┤
│ File         │ Size │ Rows │ status    │ region   │ Action      │
│              │      │      │ (min-max) │          │             │
├──────────────────────────────────────────────────────────────────┤
│ data-001.pqt │ 1GB  │ 1M   │ 200-299   │ us-east  │ ❌ SKIP     │
│ data-002.pqt │ 1GB  │ 1M   │ 200-299   │ us-west  │ ❌ SKIP     │
│ data-003.pqt │ 1GB  │ 1M   │ 300-399   │ us-east  │ ❌ SKIP     │
│ data-004.pqt │ 1GB  │ 1M   │ 400-404   │ us-east  │ ❌ SKIP     │
│ data-005.pqt │ 1GB  │ 1M   │ 500-599   │ us-east  │ ✅ KEEP     │
│ data-006.pqt │ 1GB  │ 1M   │ 500-599   │ us-west  │ ✅ KEEP     │
│ data-007.pqt │ 1GB  │ 1M   │ 500-599   │ eu-west  │ ✅ KEEP     │
│ ...          │ ...  │ ...  │ ...       │ ...      │ ❌ SKIP     │
│ data-027.pqt │ 1GB  │ 1M   │ 200-299   │ us-east  │ ❌ SKIP     │
└──────────────────────────────────────────────────────────────────┘

Result: 3 files (out of 27)
Data Remaining: 3GB (out of 27GB)
Pruned: 99.97% (cumulative)
```

### Step 3: Row Group Pruning (Parquet Footer)

```
┌──────────────────────────────────────────────────────────────────┐
│           PARQUET FILE: data-005.pqt (1GB, 1M rows)              │
│  (Each row group = ~10,000 rows = ~10MB)                         │
├──────────────────────────────────────────────────────────────────┤
│ Row Group │ Rows  │ Size │ status    │ timestamp        │ Action │
│           │       │      │ (min-max) │ (min-max)        │        │
├──────────────────────────────────────────────────────────────────┤
│ RG-00     │ 10K   │ 10MB │ 500-504   │ 00:00 - 00:14    │ ✅ KEEP│
│ RG-01     │ 10K   │ 10MB │ 500-509   │ 00:15 - 00:29    │ ✅ KEEP│
│ RG-02     │ 10K   │ 10MB │ 503-510   │ 00:30 - 00:44    │ ✅ KEEP│
│ RG-03     │ 10K   │ 10MB │ 504-599   │ 00:45 - 00:59    │ ✅ KEEP│
│ RG-04     │ 10K   │ 10MB │ 510-599   │ 01:00 - 01:14    │ ❌ SKIP│
│ RG-05     │ 10K   │ 10MB │ 520-599   │ 01:15 - 01:29    │ ❌ SKIP│
│ ...       │ ...   │ ...  │ ...       │ ...              │ ❌ SKIP│
│ RG-99     │ 10K   │ 10MB │ 550-599   │ 23:45 - 23:59    │ ❌ SKIP│
└──────────────────────────────────────────────────────────────────┘

Result per file: 4 row groups (out of 100)
Total across 3 files: 12 row groups
Data Remaining: ~120MB (out of 3GB)
Pruned: 99.999% (cumulative from 10TB)
```

## Final Execution

```
┌─────────────────────────────────────────────────────────┐
│            DISTRIBUTED EXECUTION                         │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  12 row groups distributed to workers:                  │
│                                                          │
│  Worker 1: data-005.pqt [RG-00, RG-01, RG-02, RG-03]   │
│  Worker 2: data-006.pqt [RG-00, RG-01, RG-02, RG-03]   │
│  Worker 3: data-007.pqt [RG-00, RG-01, RG-02, RG-03]   │
│                                                          │
│  Each worker:                                            │
│  1. Reads only needed columns (SELECT *)                │
│  2. Applies filters during Parquet decode               │
│  3. Returns matching rows via Arrow IPC                 │
│                                                          │
│  Coordinator aggregates results                          │
│                                                          │
└─────────────────────────────────────────────────────────┘

Final Result:
- Scanned: 120MB (out of 10TB = 0.001%)
- Time: 0.8 seconds
- Matching rows: 1,247
- Cost: $0.012
```

## Comparison: Without Pruning

```
WITHOUT INTELLIGENT PRUNING (Traditional Approach):

1. Read all 10,000 files
2. Decompress all data
3. Filter in memory
4. Return results

Data Scanned: 10TB
Time: 5-10 minutes
Cost: $1,000+
```

## Early Termination Example

Query with LIMIT:
```sql
SELECT * FROM events 
WHERE day='2024-06-15' AND status=500
ORDER BY timestamp DESC
LIMIT 10
```

```
┌─────────────────────────────────────────────────────────┐
│           EARLY TERMINATION STRATEGY                     │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  1. Planner orders row groups by timestamp DESC         │
│  2. Stream work units in that order                     │
│  3. Workers process most recent data first              │
│  4. Coordinator maintains Top-10 heap                   │
│  5. Track high-water mark of unprocessed data           │
│                                                          │
│  When heap[10].timestamp > remaining.max_timestamp:     │
│    → STOP! We have the answer.                          │
│                                                          │
└─────────────────────────────────────────────────────────┘

Result:
- Processed: 2 row groups (out of 12 candidates)
- Scanned: 20MB (out of 120MB candidates, 10TB total)
- Time: 0.2 seconds
- Found: Exactly 10 most recent matches
```

## CDC Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                   POSTGRES (HOT STORAGE)                     │
│              Applications write here normally                │
└────────────────────────┬────────────────────────────────────┘
                         │
                         │ WAL (Write-Ahead Log)
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                   RISINGWAVE CDC                             │
│                (Cloudflare Containers)                       │
├─────────────────────────────────────────────────────────────┤
│  1. Capture: Read Postgres WAL                              │
│  2. Transform: Apply streaming SQL transformations          │
│  3. Buffer: Batch events (5 min or 100MB)                   │
│  4. Write: Create Parquet file                              │
│  5. Commit: Update Iceberg manifest atomically              │
└────────────────────────┬────────────────────────────────────┘
                         │
                         │ Parquet files + Metadata
                         ▼
┌─────────────────────────────────────────────────────────────┐
│                 CLOUDFLARE R2 STORAGE                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  /events/                                                    │
│    ├── metadata/                                             │
│    │   ├── v1.metadata.json                                 │
│    │   └── snap-166.avro                                    │
│    ├── data/                                                 │
│    │   ├── day=2024-06-15/                                  │
│    │   │   ├── data-001.parquet                             │
│    │   │   ├── data-002.parquet                             │
│    │   │   └── ...                                           │
│    │   └── day=2024-06-16/                                  │
│    │       └── ...                                           │
│    └── manifests/                                            │
│        ├── manifest-166.avro                                 │
│        └── manifest-167.avro                                 │
│                                                              │
└─────────────────────────────────────────────────────────────┘

Characteristics:
- Lag: <1 second (near real-time)
- Format: Parquet (compressed, columnar)
- Commits: Atomic (via Iceberg)
- Guarantees: Exactly-once delivery
```

## Cost Breakdown Example

```
Scenario: 100TB data, 1000 queries/day, 30 days

┌─────────────────────────────────────────────────────────┐
│                  FIG PLATFORM COSTS                      │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  HOT STORAGE (5TB × 30 days)                            │
│    PlanetScale/Postgres:              $450/month        │
│                                                          │
│  COLD STORAGE (100TB)                                   │
│    R2 storage: $0.015/GB/mo           $1,500/month      │
│                                                          │
│  CDC STREAMING                                          │
│    RisingWave on Cloudflare:          $150/month        │
│                                                          │
│  QUERY COMPUTE                                          │
│    Hot (500 queries/day × 30):        Included          │
│    Cold (500 queries/day × 30):                         │
│      Avg 10GB scanned per query                         │
│      15,000 queries × 10GB × $0.01    $1,500/month      │
│                                                          │
│  TOTAL:                               $3,600/month      │
│                                                          │
└─────────────────────────────────────────────────────────┘

vs. Snowflake:  $17,960/month → 80% savings
vs. Databricks: $18,644/month → 81% savings
```

## Summary: Why Fig Works

```
┌──────────────────────────────────────────────────────────────┐
│                  THE FIG ADVANTAGE                            │
├──────────────────────────────────────────────────────────────┤
│                                                               │
│  1. METADATA-DRIVEN PRUNING                                  │
│     Skip 99%+ of data before reading                         │
│     → Query petabytes in seconds                             │
│                                                               │
│  2. HOT/COLD TIERING                                         │
│     Recent data in Postgres (<50ms)                          │
│     Historical data in Iceberg (seconds)                     │
│     → Right performance for each workload                    │
│                                                               │
│  3. SERVERLESS EXECUTION                                     │
│     No clusters to manage                                    │
│     Auto-scale to query complexity                           │
│     → Zero ops overhead                                      │
│                                                               │
│  4. OPEN STANDARDS                                           │
│     Parquet, Iceberg, Arrow                                  │
│     No vendor lock-in                                        │
│     → Full data portability                                  │
│                                                               │
│  5. EDGE DISTRIBUTION                                        │
│     Cloudflare's global network                              │
│     Workers near data                                        │
│     → Lowest latency worldwide                               │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```
