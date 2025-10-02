# Fig Platform - Presentation Talking Points

## Opening (Slide 1)

**Key Message:** "What if you could query petabytes of data without managing a single server?"

**Talking Points:**
- Traditional data platforms force you to choose: either pay for expensive managed services (Snowflake, Databricks) or spend months setting up and managing your own infrastructure (Spark, Trino)
- Fig eliminates this false choice by building a serverless data platform inspired by Cloudflare's R2 SQL architecture
- Three core components: Hot storage for recent data, CDC streaming for real-time replication, and a serverless query engine for petabyte-scale analytics

**Hook:** "Cloudflare proved you can query petabytes without servers. We're bringing that to everyone."

---

## The Problem (Slide 2)

**Key Message:** "Current platforms are either expensive and proprietary, or complex and self-managed"

**Talking Points:**
- **Snowflake/Databricks:** Expensive credits, egress fees, vendor lock-in, proprietary formats
- **DIY Iceberg:** You manage Spark/Trino clusters, deal with availability, scaling, and operations
- **Result:** Teams spend more time on infrastructure than extracting value from data

**Transition:** "Fig solves this with a three-tier architecture that handles both operational and analytical workloads..."

---

## Platform Overview (Slide 3)

**Key Message:** "Three tiers working together: Hot, Stream, Cold"

**Talking Points:**

### Hot Tier (PlanetScale/Postgres)
- "Recent data lives here - last 30-90 days depending on your needs"
- "Sub-50ms queries for dashboards, applications, operational analytics"
- "This is what users interact with 95% of the time"

### Stream Tier (RisingWave CDC)
- "Captures every change from Postgres using CDC"
- "Runs on Cloudflare containers at the edge"
- "Writes directly to Parquet format in near real-time"
- "This is your data pipeline - but you never configure it"

### Cold Tier (R2 + Iceberg)
- "Petabytes of historical data in Apache Iceberg format"
- "Open standard - no vendor lock-in"
- "This is where the magic happens with our query engine"

**Key Point:** "Most platforms make you choose hot OR cold. Fig gives you both, with automatic tiering."

---

## The Secret Sauce: Metadata Pruning (Slide 4)

**Key Message:** "The most efficient way to process data is to not read it in the first place"

**Talking Points:**

### The Problem
- "Imagine you have 100TB of data, but your query only needs 1GB"
- "Traditional approach: Read everything, filter later"
- "That's wasteful and slow"

### Our Solution: Three-Level Pruning
1. **Partition Pruning (Manifest List)**
   - "Iceberg organizes data into partitions - like folders by date"
   - "We can skip entire partitions immediately using metadata"
   - "Query for Jan 15th? Skip Dec, Feb, etc. instantly"

2. **File Pruning (Manifest Files)**
   - "Each partition has many files. Iceberg tracks min/max values per file"
   - "Looking for HTTP 500 errors? Skip files where max status is 404"
   - "Prune 99% of files before reading a single byte"

3. **Row Group Pruning (Parquet)**
   - "Even within files, we can skip row groups"
   - "Parquet stores stats every ~1000 rows"
   - "Final precision pruning"

**Analogy:** "It's like using a book's index and table of contents instead of reading every page to find one paragraph."

**Key Stat:** "In practice, we skip 99.9% of data for typical queries. Query petabytes, read megabytes."

---

## Streaming Pipeline (Slide 4 continued)

**Key Message:** "Start processing immediately, stop as soon as we have the answer"

**Talking Points:**

### Start Early
- "We don't wait to plan the entire query"
- "As soon as we know which files to read, we start reading them"
- "Planner and executor work in parallel"

### Finish Early
- "For queries with LIMIT, we maintain a 'top N' heap"
- "We know the timestamp range of remaining data"
- "The moment our top N is guaranteed, we stop"
- "Example: `LIMIT 5` might read 0.001% of matching data"

### Process in Parallel
- "Work units distributed to Cloudflare edge workers globally"
- "Each worker runs Apache DataFusion for vectorized execution"
- "Process thousands of row groups simultaneously"

**Impact:** "Queries that would take hours on Spark complete in seconds on Fig."

---

## Architecture Diagram (Slide 3)

**Key Message:** "Simple architecture, sophisticated execution"

**Talking Points:**

**Data Write Path:**
1. "Application writes to Postgres - normal OLTP operations"
2. "RisingWave captures CDC events in real-time"
3. "Batches and writes Parquet files to R2"
4. "Updates Iceberg metadata - atomic commits"

**Query Path (Recent Data):**
- "Application queries Postgres - sub-50ms response"
- "This handles 95% of queries"

**Query Path (Historical Data):**
1. "Application sends SQL query to Fig"
2. "Query planner reads Iceberg metadata from R2"
3. "Performs three-level pruning"
4. "Streams work units to distributed workers"
5. "Workers read Parquet from R2, process, return results"
6. "Response in seconds"

**Key Point:** "One unified platform. Write once, query anywhere. Hot or cold."

---

## Deep Dive: Query Engine (Slide 6)

**Key Message:** "This is what makes serverless petabyte queries possible"

### Why Not Spark/Trino?

**Talking Points:**
- "These are great engines, but they require cluster management"
- "You provision nodes, configure memory, handle failures"
- "You pay for idle time"
- "We wanted zero ops"

### Our Approach: Inspired by R2 SQL

**Talking Points:**
- "Cloudflare proved this works at massive scale"
- "They query petabytes for their own analytics"
- "We're applying the same principles"

### Technical Components

1. **Query Planner**
   - "Reads Iceberg metadata"
   - "Performs intelligent pruning"
   - "Streams work units"

2. **Query Coordinator**
   - "Receives user query"
   - "Manages planner"
   - "Aggregates results from workers"

3. **Query Workers**
   - "Distributed across Cloudflare edge"
   - "Run Apache DataFusion"
   - "Process row groups in parallel"
   - "Return results via gRPC"

4. **Apache DataFusion**
   - "Rust-based query engine"
   - "Vectorized execution - process batches, not rows"
   - "Native Parquet support"
   - "Column pruning - read only needed columns"
   - "Predicate pushdown - filter during read"

**Analogy:** "It's like having thousands of workers who can be hired instantly, work for seconds, and disappear. You only pay for those seconds."

---

## Technical Decisions (Slide 7)

**Key Message:** "Every choice optimizes for simplicity, performance, or cost"

### Why PlanetScale/Postgres?
- "Battle-tested reliability"
- "Sub-50ms latency"
- "Everyone knows SQL"
- "Easy horizontal scaling"
- **Trade-off:** "Not designed for petabytes - that's why we have cold storage"

### Why RisingWave?
- "Streaming SQL - familiar interface"
- "Native Postgres CDC"
- "Writes Parquet directly"
- "Runs efficiently on Cloudflare"
- **Alternative considered:** "Debezium + Kafka too complex"

### Why R2 + Iceberg?
- "R2: Zero egress fees (vs S3's $0.09/GB)"
- "Iceberg: Open format, rich metadata, ACID transactions"
- **Key benefit:** "No vendor lock-in. Your data, your format"

### Why Custom Query Engine?
- "Spark/Trino require cluster management"
- "Commercial options (Dremio, Starburst) expensive"
- "Cloudflare proved it's viable"
- **Decision:** "We can build serverless on their shoulders"

---

## Comparison (Slide 5)

**Key Message:** "10x simpler, 5x cheaper, infinitely more scalable"

### Fig Platform
- ✅ Serverless - no clusters to manage
- ✅ Sub-50ms hot queries
- ✅ Seconds for petabyte analytics
- ✅ Open formats (Parquet, Iceberg)
- ✅ Zero egress fees
- ✅ Pay only for what you process

### Traditional Platforms

**Snowflake:**
- ❌ Warehouse provisioning
- ❌ Proprietary format
- ❌ High egress fees ($0.09/GB)
- ❌ Credit-based pricing (opaque)

**Databricks:**
- ❌ Manage Spark clusters
- ❌ Complex autoscaling
- ❌ DBU pricing model
- ❌ Idle compute costs

**DIY Iceberg (Trino/Presto):**
- ❌ Deploy and manage clusters
- ❌ Handle availability
- ❌ Configure memory/CPU
- ❌ Debug distributed systems

**Cost Example:**
- "100TB data, 1000 queries/day"
- "Fig: ~$2,000/month"
- "Databricks: ~$15,000/month"
- "Snowflake: ~$20,000/month"

---

## Putting It All Together (Slide 8)

**Key Message:** "One platform, three tiers, zero data engineering"

**Scenario Walkthrough:**

### User Journey
1. "Data scientist submits query: 'Show me all HTTP 500 errors in Q3 2024 for US customers'"

2. "Query hits Fig's coordinator"

3. "Planner thinks: 'That's historical data - go to cold storage'"

4. "Planner reads Iceberg manifest list"
   - "Skip manifests for Q1, Q2, Q4"
   - "Keep only Q3 manifests"

5. "For Q3 manifests, read manifest files"
   - "Skip files where status < 500"
   - "Skip files where region != 'US'"
   - "Result: 1,247 candidate files (out of 8 million)"

6. "Read Parquet footers for those files"
   - "Skip row groups not matching"
   - "Result: 8,432 row groups to process"

7. "Distribute row groups to 200 workers across Cloudflare edge"
   - "Each worker processes ~42 row groups"
   - "Parallel execution"
   - "Results stream back to coordinator"

8. "Coordinator aggregates and returns results"
   - "Total time: 3.7 seconds"
   - "Data scanned: 89GB (out of 100TB total)"
   - "Cost: $8.90"

**Comparison:**
- "Same query on Snowflake: 4-5 minutes, $150+"
- "DIY Spark cluster: 10-15 minutes, cluster must be running"

---

## What You Get (Slide 8)

**Key Message:** "All the benefits, none of the headaches"

### For Data Teams
✅ "Write SQL, get answers"
✅ "No pipeline engineering"
✅ "No cluster tuning"
✅ "No capacity planning"

### For DevOps
✅ "No infrastructure to manage"
✅ "No uptime concerns"
✅ "No scaling problems"

### For Finance
✅ "Predictable costs"
✅ "Pay for value, not idle time"
✅ "10x cheaper than alternatives"

### For the Business
✅ "Faster insights"
✅ "No vendor lock-in"
✅ "Open data formats"
✅ "Infinite scale"

---

## Closing

**Key Message:** "The future of data platforms is serverless, distributed, and open"

**Talking Points:**
- "Snowflake and Databricks built their empires on proprietary formats and vendor lock-in"
- "Cloudflare showed us a better way with R2 SQL"
- "Fig brings that vision to everyone"

**Call to Action:**
- "Imagine: No more cluster management"
- "No more complex pipelines"
- "No more vendor lock-in"
- "Just write SQL and get answers"

**Final Statement:**
"One platform. Zero data engineering. Infinite scale. This is Fig."

---

## Anticipated Questions & Answers

### Q: "How does this compare to ClickHouse?"
**A:** "ClickHouse is excellent for real-time analytics on hot data. But it's not designed for petabyte-scale cold storage. We use Postgres for hot (similar use case to ClickHouse) and our distributed engine for cold. Best of both worlds."

### Q: "What about data freshness? How soon is CDC data available in cold storage?"
**A:** "Sub-second CDC latency. RisingWave streams continuously. Parquet files are written every few seconds to minutes depending on volume. Iceberg commits are atomic, so data is immediately queryable once committed."

### Q: "Can I use my existing BI tools?"
**A:** "Yes! We expose standard SQL interface. Tableau, Looker, Mode - all work out of the box. For hot queries, direct Postgres connection. For cold queries, Fig's SQL API."

### Q: "What about JOIN performance across hot and cold?"
**A:** "Great question. For joins between hot and cold, we push filters to both sides. Cold data is filtered via metadata pruning, then joined with hot results. For large joins, we can materialize hot data to cold first."

### Q: "How do you handle schema evolution?"
**A:** "Iceberg has first-class schema evolution support. Add columns, rename, change types. RisingWave detects schema changes in Postgres and updates Iceberg accordingly. No downtime."

### Q: "What's the failover story?"
**A:** "Hot storage: Postgres native replication. CDC: RisingWave checkpointing ensures exactly-once. Cold queries: Distributed execution means individual worker failures don't break queries. Cloudflare's edge provides redundancy."

### Q: "Why not just use DuckDB?"
**A:** "DuckDB is incredible for single-node analytics. But it's not distributed. For petabyte-scale data across thousands of files, you need distributed execution. That said, we could potentially use DuckDB as an execution engine in workers for small queries!"

### Q: "How do you handle updates and deletes in Iceberg?"
**A:** "Iceberg supports ACID transactions including updates/deletes. RisingWave can emit UPDATE/DELETE events via CDC. Iceberg uses delete files to mark rows as deleted. Compaction merges these periodically."

### Q: "What about compliance (GDPR, right to be forgotten)?"
**A:** "Hot storage: Standard Postgres DELETE. Cold storage: Iceberg delete files + periodic compaction. For immediate deletion, we can mark files for exclusion in metadata. Compaction rewrites files without deleted rows."

### Q: "How does pricing work exactly?"
**A:** "Three components: 1) R2 storage ($0.015/GB/month), 2) Query compute (based on data processed, ~$0.10/GB), 3) Hot storage (your Postgres costs). No egress fees, no per-query fees, no warehouses to provision."

---

## Demo Scenarios (if applicable)

### Scenario 1: Simple Time-Range Query
```sql
SELECT user_id, COUNT(*) as events
FROM user_events
WHERE event_date >= '2024-01-01' AND event_date < '2024-02-01'
GROUP BY user_id
ORDER BY events DESC
LIMIT 10;
```

**What to highlight:**
- Query spans 1 month of data
- Partition pruning skips other months instantly
- Results in ~2 seconds
- Show cost: minimal because of pruning

### Scenario 2: Complex Filtering
```sql
SELECT *
FROM logs
WHERE timestamp > NOW() - INTERVAL '7 days'
  AND status_code = 500
  AND region = 'us-east'
  AND user_agent LIKE '%Chrome%'
ORDER BY timestamp DESC
LIMIT 100;
```

**What to highlight:**
- Multi-level pruning in action
- File-level stats eliminate non-matching files
- Early termination with LIMIT
- Show execution plan with pruning stats

### Scenario 3: Hot vs Cold Query
```sql
-- Hot query (last 24 hours)
SELECT COUNT(*) FROM events 
WHERE timestamp > NOW() - INTERVAL '1 day';
-- Result: 23ms from Postgres

-- Cold query (last year)
SELECT COUNT(*) FROM events
WHERE timestamp > NOW() - INTERVAL '1 year';
-- Result: 2.4s from Iceberg via query engine
```

**What to highlight:**
- Same table, different time ranges
- Automatic routing to hot or cold
- Both complete in reasonable time
