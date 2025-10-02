# Fig Platform Architecture

## Executive Summary

Fig is a serverless data platform that eliminates data engineering complexity by combining hot storage (PlanetScale/Postgres), real-time CDC streaming (RisingWave), and petabyte-scale analytics (Cloudflare R2 + Apache Iceberg). Inspired by Cloudflare's R2 SQL architecture, Fig provides a distributed query engine that uses intelligent metadata pruning to query petabytes in seconds without managing clusters.

## Architecture Overview

```
┌─────────────┐
│ Application │
└──────┬──────┘
       │
       ├─────────────────────────────────────┐
       │                                     │
       ▼ Writes (hot)                        ▼ Reads (recent)
┌──────────────────┐                   ┌──────────────┐
│ PlanetScale/     │ ◄─────────────── │  < 50ms      │
│ Postgres         │                   │  Latency     │
│ (Hot Storage)    │                   └──────────────┘
└────────┬─────────┘
         │ CDC Events
         ▼
┌─────────────────────┐
│ RisingWave CDC      │
│ Cloudflare Workers  │
│ / Containers        │
└────────┬────────────┘
         │ Parquet files
         ▼
┌─────────────────────┐
│ Cloudflare R2       │
│ Apache Iceberg      │
│ (Cold Storage)      │
└────────┬────────────┘
         │
         ▼ Historical queries
┌─────────────────────┐
│ Query Planner       │
│ - Metadata pruning  │
│ - Work streaming    │
└────────┬────────────┘
         │ Work units
         ▼
┌─────────────────────┐
│ Distributed Workers │
│ Apache DataFusion   │
│ Cloudflare Edge     │
└────────┬────────────┘
         │
         ▼ Results (seconds)
┌─────────────────────┐
│ Application         │
└─────────────────────┘
```

## Three-Layer Architecture

### 1. Hot Storage Layer (PlanetScale/Postgres)

**Purpose:** Operational queries on recent data

**Technology:** PlanetScale or Postgres with read replicas

**Characteristics:**
- Sub-50ms query latency
- Stores last 30-90 days of data (configurable)
- Familiar SQL interface
- Horizontal scaling via read replicas
- ACID transactions for writes

**Use Cases:**
- Dashboard queries
- Real-time analytics
- Application queries
- Recent data lookups

### 2. CDC Streaming Layer (RisingWave)

**Purpose:** Real-time replication from hot to cold storage

**Technology:** RisingWave streaming SQL engine

**Deployment:** Cloudflare Workers/Containers (edge compute)

**Characteristics:**
- Sub-second replication latency
- Streaming SQL transformations
- Native Postgres CDC support
- Writes directly to Parquet format
- Manages Iceberg table metadata

**Data Flow:**
1. Capture Postgres CDC events (WAL)
2. Apply streaming transformations
3. Buffer and batch events
4. Write Parquet files to R2
5. Update Iceberg manifest files

### 3. Cold Storage Layer (R2 + Iceberg)

**Purpose:** Petabyte-scale historical analytics

**Technology:** 
- **Storage:** Cloudflare R2 object storage
- **Format:** Apache Parquet (columnar)
- **Catalog:** Apache Iceberg (metadata management)
- **Query Engine:** Custom distributed engine inspired by R2 SQL

**Characteristics:**
- Petabyte scale
- $0.015/GB/month storage cost
- Zero egress fees
- Open table format (no vendor lock-in)
- Seconds latency for complex queries

## Query Engine: The Heart of Fig

### Inspired by Cloudflare R2 SQL

The Fig query engine is heavily inspired by [Cloudflare's R2 SQL architecture](https://blog.cloudflare.com/r2-sql-deep-dive/), which demonstrates how to build a serverless query engine that can query petabytes without managing servers.

### Key Innovation: Intelligent Metadata Pruning

Traditional query engines (Spark, Trino, Presto) read data files to determine what to process. Fig's query engine uses Apache Iceberg's metadata hierarchy to **prune the search space before reading a single byte of data**.

#### Three-Level Pruning Strategy

```
┌─────────────────────────────────────────────────────┐
│ Query: SELECT * FROM events                         │
│        WHERE day = '2025-09-25' AND status = 500    │
│        ORDER BY timestamp DESC LIMIT 5              │
└─────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────┐
│ 1. PARTITION PRUNING (Manifest List)                │
│    - Read manifest list metadata                    │
│    - Use partition stats: min/max day               │
│    - Skip manifests where day ≠ '2025-09-25'       │
│    - Result: 1 manifest (out of 1000s)             │
└─────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────┐
│ 2. FILE PRUNING (Manifest Files)                    │
│    - Read manifest file                             │
│    - Use column stats: min/max status               │
│    - Skip files where status ∈ [200, 404]          │
│    - Result: 5 files (out of 10,000s)              │
└─────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────┐
│ 3. ROW GROUP PRUNING (Parquet Footers)             │
│    - Read Parquet file footers                      │
│    - Use row group stats                            │
│    - Skip row groups not matching filters           │
│    - Result: 12 row groups (out of 1000s)          │
└─────────────────────────────────────────────────────┘
                         │
                         ▼
                    Work Units
```

**Result:** Instead of scanning petabytes, we read only megabytes.

### Streaming Query Pipeline

The query planner and executor work as a concurrent pipeline:

```rust
// Simplified pseudo-code
async fn execute_query(sql: Query) -> Results {
    // Start planning and execution concurrently
    let (work_tx, work_rx) = channel();
    
    // Planner task: Stream work units
    spawn(async move {
        let snapshot = fetch_snapshot(table).await;
        let manifest_list = fetch_manifest_list(snapshot).await;
        
        // Order manifests by query's ORDER BY
        let ordered_manifests = order_manifests(
            manifest_list,
            sql.order_by
        );
        
        for manifest in ordered_manifests {
            let files = fetch_manifest(manifest).await;
            let pruned_files = prune_by_column_stats(files, sql.filters);
            
            for file in pruned_files {
                let row_groups = get_row_groups(file).await;
                work_tx.send(row_groups).await;
            }
        }
    });
    
    // Executor task: Process work units
    spawn(async move {
        let mut top_n_heap = BoundedHeap::new(sql.limit);
        let mut high_water_mark = get_initial_hwm();
        
        while let Some(work_unit) = work_rx.recv().await {
            // Distribute to workers
            let results = execute_on_workers(work_unit).await;
            
            // Update heap
            top_n_heap.insert(results);
            
            // Early termination check
            if can_stop_early(top_n_heap, high_water_mark) {
                break; // We have the answer!
            }
        }
    });
}
```

**Key Benefits:**
1. **Start Early:** Begin processing data while planning continues
2. **Finish Early:** Stop when top N results are guaranteed
3. **Parallel Execution:** Distribute work across edge workers

### Distributed Execution with Apache DataFusion

Each work unit (row group) is processed by a query worker running [Apache DataFusion](https://arrow.apache.org/datafusion/):

**Why DataFusion?**
- Written in Rust (fast, safe)
- Vectorized execution (process batches, not individual rows)
- Native Parquet support with predicate pushdown
- Column pruning (read only needed columns)
- Uses Apache Arrow for zero-copy data structures

**Worker Execution Flow:**
1. Receive work unit (row group metadata)
2. Use R2 ranged reads to fetch only required columns
3. Apply filters during Parquet decoding (predicate pushdown)
4. Process batches using vectorized operators
5. Return results using Arrow IPC format

## Comparison with Traditional Approaches

### Traditional: Manage Your Own Iceberg Query Engine

```
┌─────────────────────────────────────────────────────┐
│ YOUR RESPONSIBILITY:                                 │
│ - Provision Spark/Trino/Presto clusters            │
│ - Configure cluster sizes                           │
│ - Manage availability and uptime                    │
│ - Handle autoscaling                                │
│ - Debug complex distributed systems                 │
│ - Pay for idle compute                              │
└─────────────────────────────────────────────────────┘
```

### Fig: Serverless Query Engine

```
┌─────────────────────────────────────────────────────┐
│ YOUR RESPONSIBILITY:                                 │
│ - Write SQL query                                   │
│ - (That's it)                                       │
│                                                     │
│ FIG HANDLES:                                        │
│ - Query planning and optimization                   │
│ - Work distribution to edge workers                 │
│ - Scaling compute to match query complexity         │
│ - Early termination for LIMIT queries               │
│ - Pay only for data processed                       │
└─────────────────────────────────────────────────────┘
```

## Cost Analysis

### Storage Costs

| Component | Technology | Cost | Notes |
|-----------|-----------|------|-------|
| Hot Storage | PlanetScale/Postgres | ~$0.30/GB/month | Includes read replicas |
| Cold Storage | Cloudflare R2 | $0.015/GB/month | No egress fees |
| Egress | R2 → Internet | $0 | vs S3: $0.09/GB |

### Query Costs

**Fig Model:** Pay per query based on data scanned
- Metadata operations: Free (small JSON files)
- Data scanning: ~$0.10 per GB processed
- Worker compute: Amortized across all queries

**Traditional Model:** Pay for running clusters
- Spark cluster: $0.50-2.00/hour minimum
- Idle time costs the same as active time
- Complex autoscaling configuration

### Example Scenario

**Workload:** 100TB data, 1000 queries/day, avg 10GB scanned per query

| Approach | Monthly Cost | Notes |
|----------|--------------|-------|
| **Fig** | **~$2,000** | $1,500 storage + $500 query compute |
| **Databricks** | **~$15,000** | Cluster costs + storage + egress |
| **Snowflake** | **~$20,000** | Credit consumption + storage |

## Technical Decisions and Rationale

### Why PlanetScale/Postgres for Hot Storage?

**Pros:**
- ✅ Sub-50ms latency
- ✅ Familiar SQL interface
- ✅ Battle-tested reliability
- ✅ Strong ACID guarantees
- ✅ Easy to scale horizontally

**Cons:**
- ❌ Not designed for petabyte scale (hence cold tier)
- ❌ Retention limited by cost

**Decision:** Use for recent, frequently accessed data only.

### Why RisingWave for CDC?

**Pros:**
- ✅ Streaming SQL (familiar interface)
- ✅ Native Postgres CDC support
- ✅ Can write directly to Parquet
- ✅ Handles backpressure and retries
- ✅ Runs efficiently on Cloudflare

**Alternatives Considered:**
- **Debezium:** More complex, requires Kafka
- **Airbyte:** Batch-oriented, not true streaming
- **Custom CDC:** Development overhead

**Decision:** Best balance of features and operational simplicity.

### Why Cloudflare R2 + Iceberg?

**R2 Advantages:**
- ✅ Zero egress fees (vs. S3's $0.09/GB)
- ✅ S3-compatible API
- ✅ Global edge network
- ✅ Native integration with Workers

**Iceberg Advantages:**
- ✅ Open table format (no lock-in)
- ✅ Rich metadata for query optimization
- ✅ ACID transactions
- ✅ Schema evolution
- ✅ Time travel queries

**Alternatives Considered:**
- **Delta Lake:** More Spark-centric
- **Hudi:** Complex configuration
- **Raw Parquet:** No metadata, no transactions

**Decision:** Iceberg's metadata structure is essential for query pruning.

### Why Build a Custom Query Engine?

**Alternatives:**
- **Apache Spark:** Heavy, cluster management
- **Trino/Presto:** Complex deployment, resource-intensive
- **Dremio/Starburst:** Commercial, expensive
- **DuckDB:** Single-node, not distributed

**Why Custom Engine:**
- ✅ Serverless execution model
- ✅ Leverages Cloudflare's edge network
- ✅ Optimized for Iceberg metadata
- ✅ Early termination for LIMIT queries
- ✅ No cluster management overhead

**Decision:** Inspired by Cloudflare R2 SQL's success, proves it's viable.

## Implementation Roadmap

### Phase 1: Proof of Concept (Months 1-3)
- [ ] Set up PlanetScale/Postgres instance
- [ ] Deploy RisingWave on Cloudflare Workers
- [ ] Configure CDC from Postgres to R2
- [ ] Implement basic query planner
- [ ] Test metadata pruning strategies
- [ ] Benchmark query performance

### Phase 2: Query Engine MVP (Months 4-6)
- [ ] Implement streaming query pipeline
- [ ] Integrate Apache DataFusion workers
- [ ] Deploy distributed workers to Cloudflare edge
- [ ] Implement early termination logic
- [ ] Add support for common SQL operations
- [ ] Build query monitoring dashboard

### Phase 3: Production Readiness (Months 7-9)
- [ ] Add authentication and authorization
- [ ] Implement query rate limiting
- [ ] Build cost tracking and billing
- [ ] Add query optimization hints
- [ ] Create developer documentation
- [ ] Set up monitoring and alerting

### Phase 4: Advanced Features (Months 10-12)
- [ ] Add complex aggregations support
- [ ] Implement query caching
- [ ] Add full-text search indexes
- [ ] Support geospatial queries
- [ ] Build visual query analyzer
- [ ] Add multi-tenant support

## Monitoring and Observability

### Key Metrics

**Hot Storage:**
- Query latency (p50, p95, p99)
- Read throughput
- Write throughput
- Replication lag

**CDC Pipeline:**
- CDC lag (seconds behind source)
- Parquet files written per minute
- Iceberg commits per minute
- Buffer sizes

**Cold Storage Query Engine:**
- Metadata read time
- Data pruned percentage
- Files scanned per query
- Row groups processed
- Worker utilization
- Query completion time

### Example Dashboard

```
┌─────────────────────────────────────────────────────┐
│ Fig Platform Health                                  │
├─────────────────────────────────────────────────────┤
│ Hot Storage:          98.7% data in < 50ms          │
│ CDC Lag:              0.3 seconds                    │
│ Cold Query Avg:       2.4 seconds                    │
│ Pruning Efficiency:   99.8% (data skipped)          │
│ Active Workers:       127 / 200                      │
│ Cost (today):         $127.50                        │
└─────────────────────────────────────────────────────┘
```

## Security Considerations

### Data Access Control
- Row-level security in Postgres
- Iceberg table-level permissions
- Query-time authorization checks
- Audit logging for all queries

### Network Security
- TLS for all connections
- Private VPC for hot storage
- Cloudflare Zero Trust for worker access
- API key rotation

### Data Encryption
- At-rest: R2 server-side encryption
- In-transit: TLS 1.3
- Column-level encryption (optional)

## Conclusion

Fig demonstrates that you don't need Snowflake or Databricks to build a modern data platform. By combining:

1. **Hot storage** (PlanetScale/Postgres) for operational queries
2. **CDC streaming** (RisingWave) for real-time replication
3. **Cold storage** (R2 + Iceberg) for petabyte-scale analytics
4. **Serverless query engine** (inspired by R2 SQL) for intelligent pruning

...you can build a platform that is:
- **Simpler:** No cluster management
- **Faster:** Sub-50ms hot queries, seconds for cold
- **Cheaper:** 10x cost reduction vs. traditional platforms
- **Open:** No vendor lock-in with Iceberg/Parquet

The future of data platforms is serverless, distributed, and built on open formats.

## References

- [Cloudflare R2 SQL Deep Dive](https://blog.cloudflare.com/r2-sql-deep-dive/)
- [Apache Iceberg Documentation](https://iceberg.apache.org/)
- [Apache DataFusion](https://arrow.apache.org/datafusion/)
- [RisingWave CDC](https://www.risingwave.dev/)
- [PlanetScale Architecture](https://planetscale.com/)
