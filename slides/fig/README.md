# Fig Platform - Serverless Data Platform Presentation

## Overview

This presentation introduces **Fig**, a serverless data platform inspired by [Cloudflare's R2 SQL architecture](https://blog.cloudflare.com/r2-sql-deep-dive/). Fig eliminates data engineering complexity by combining hot storage, CDC streaming, and intelligent cold storage with a distributed query engine.

## What is Fig?

Fig is a three-tier data platform that provides:

- **🔥 Hot Storage:** PlanetScale/Postgres for sub-50ms operational queries
- **🌊 CDC Streaming:** RisingWave for real-time replication to cold storage
- **❄️ Cold Storage:** Cloudflare R2 + Apache Iceberg for petabyte-scale analytics
- **⚡ Query Engine:** Distributed, serverless query engine with intelligent metadata pruning

## Key Innovation: Metadata Pruning

The core technical innovation is **three-level intelligent pruning** using Apache Iceberg metadata:

1. **Partition Pruning** - Skip entire partitions using manifest list stats
2. **File Pruning** - Skip Parquet files using manifest column stats
3. **Row Group Pruning** - Skip row groups using Parquet footer stats

**Result:** Query petabytes in seconds by reading only megabytes (99%+ pruning efficiency).

## Value Proposition

- **80%+ cost reduction** vs. Snowflake/Databricks
- **Zero cluster management** - fully serverless
- **Sub-50ms** operational queries (hot storage)
- **Seconds** for petabyte analytics (cold storage)
- **No vendor lock-in** - open formats (Parquet, Iceberg)

## Running the Presentation

To start the slide show:

- `pnpm install`
- `pnpm dev`
- visit <http://localhost:3030>

Edit the [slides.md](./slides.md) to see the changes.

Learn more about Slidev at the [documentation](https://sli.dev/).

## Documentation

This repository includes comprehensive documentation:

- **[slides.md](./slides.md)** - The Slidev presentation
- **[ARCHITECTURE.md](./ARCHITECTURE.md)** - Technical architecture deep dive (30+ pages)
- **[TALKING_POINTS.md](./TALKING_POINTS.md)** - Presentation talking points and Q&A
- **[REFERENCE.md](./REFERENCE.md)** - Quick reference guide with examples
- **[SUMMARY.md](./SUMMARY.md)** - Executive summary and recommendations

## Architecture Inspiration

Fig is heavily inspired by **Cloudflare's R2 SQL** architecture from their [September 2025 blog post](https://blog.cloudflare.com/r2-sql-deep-dive/).

### Key Learnings from R2 SQL

1. **Metadata is Everything** - Iceberg's metadata hierarchy enables massive pruning
2. **Streaming Pipeline** - Start execution immediately, don't wait for complete plan
3. **Early Termination** - For LIMIT queries, stop when answer is guaranteed
4. **Distributed Execution** - Use edge workers for massively parallel processing
5. **Apache DataFusion** - Rust-based vectorized execution engine

### Our Additions

1. **Hot/Cold Tiering** - R2 SQL focuses on cold; we add hot storage
2. **CDC Streaming** - Automatic replication via RisingWave
3. **Complete Platform** - Not just query engine, full data platform

## Technical Stack

### Hot Tier
- **Database:** PlanetScale or Postgres 15+
- **Latency:** <50ms | **Scale:** Terabytes

### CDC Tier
- **Engine:** RisingWave on Cloudflare Containers
- **Latency:** <1 second

### Cold Tier
- **Storage:** Cloudflare R2 | **Format:** Apache Iceberg + Parquet
- **Query Engine:** Apache DataFusion on Cloudflare Workers
- **Latency:** Seconds | **Scale:** Petabytes

## Cost Comparison

Example: 100TB data, 1000 queries/day

| Platform | Monthly Cost |
|----------|--------------|
| **Fig** | **$3,450** |
| Snowflake | $17,960 |
| Databricks | $18,644 |

**Savings: 80%+**

## References

- [Cloudflare R2 SQL Deep Dive](https://blog.cloudflare.com/r2-sql-deep-dive/)
- [Apache Iceberg](https://iceberg.apache.org/)
- [Apache DataFusion](https://arrow.apache.org/datafusion/)
- [RisingWave](https://www.risingwave.dev/)

---

**Fig: Query petabytes without servers.**
