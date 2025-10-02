# Fig Platform - One-Page Summary

## What is Fig?

A **serverless data platform** that eliminates data engineering by combining hot storage (PlanetScale/Postgres), CDC streaming (RisingWave), and intelligent cold storage (Cloudflare R2 + Apache Iceberg). Inspired by Cloudflare's R2 SQL architecture.

## The Problem

Current data platforms force an impossible choice:
- **Managed (Snowflake/Databricks):** Expensive, proprietary, vendor lock-in
- **Self-Managed (Spark/Trino):** Months of setup, complex cluster management

## The Solution

Three-tier architecture with **intelligent metadata pruning**:

```
Applications → Postgres (Hot, <50ms) → RisingWave CDC → R2 Iceberg (Cold, seconds)
                    ↓                                           ↓
              Operational queries                    Serverless query engine
```

## Key Innovation: Metadata Pruning

Query petabytes by reading megabytes:

1. **Partition Pruning** - Skip entire partitions (days, regions, etc.)
2. **File Pruning** - Skip Parquet files using min/max stats
3. **Row Group Pruning** - Skip row groups within files

**Result:** 99%+ of data skipped before reading a single byte

## Value Proposition

| Metric | Fig | Snowflake | Databricks |
|--------|-----|-----------|------------|
| **Setup** | Hours | Days | Weeks |
| **Ops** | Zero | Medium | High |
| **Hot Queries** | <50ms | <100ms | N/A |
| **Cold Queries** | Seconds | Minutes | Minutes |
| **Cost (100TB)** | **$3.5k/mo** | $18k/mo | $19k/mo |
| **Egress** | $0 | High | High |
| **Lock-in** | None | High | Medium |

**80% cheaper, 10x simpler, infinitely scalable**

## Technical Stack

- **Hot:** PlanetScale/Postgres (operational queries)
- **CDC:** RisingWave on Cloudflare (real-time replication)
- **Cold:** R2 + Iceberg (petabyte analytics)
- **Query:** Apache DataFusion on edge (distributed execution)

## Why It Works

Cloudflare proved this at scale with R2 SQL. We're applying the same principles:

1. Iceberg metadata enables intelligent pruning
2. Streaming pipeline (start early, finish early)
3. Distributed edge execution
4. Open standards (no lock-in)

## Implementation

- **Phase 1 (6 weeks):** Proof of concept - validate pruning efficiency
- **Phase 2 (3-4 months):** Build query engine with DataFusion
- **Phase 3 (2-3 months):** Production deployment and migration

**Investment:** $150-200k first year (15-20 person-months)
**Savings:** $200k+/year (80% platform cost reduction)
**ROI:** 6-12 months

## Next Steps

1. **This week:** Present to stakeholders, get POC approval
2. **Next month:** Deploy POC environment, load sample data
3. **3-6 months:** Build production query engine
4. **6-12 months:** Migrate production workload, realize savings

## Contact

- Matthew Aylward - Platform architecture
- Jan Nelis - CDC streaming

---

**Fig: The future of data platforms is serverless, distributed, and open.**
