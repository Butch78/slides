# Fig Platform Documentation - Navigation Guide

## 📚 Documentation Overview

This folder contains a complete presentation and architecture documentation for **Fig**, a serverless data platform inspired by Cloudflare's R2 SQL architecture.

## 🎯 Quick Start - Pick Your Path

### For Executives (5 minutes)
1. Read **[ONE_PAGER.md](./ONE_PAGER.md)** - Complete overview on one page
2. View **[slides.md](./slides.md)** - Visual presentation

### For Technical Leads (30 minutes)
1. Read **[SUMMARY.md](./SUMMARY.md)** - Executive summary with recommendations
2. Review **[DIAGRAMS.md](./DIAGRAMS.md)** - Visual query flow and architecture
3. Browse **[ARCHITECTURE.md](./ARCHITECTURE.md)** - Technical deep dive

### For Developers (2 hours)
1. Read **[ARCHITECTURE.md](./ARCHITECTURE.md)** - Complete technical architecture
2. Study **[REFERENCE.md](./REFERENCE.md)** - Examples and best practices
3. Review **[TALKING_POINTS.md](./TALKING_POINTS.md)** - Q&A and edge cases

### For Presenters (1 hour prep)
1. Review **[slides.md](./slides.md)** - The presentation itself
2. Study **[TALKING_POINTS.md](./TALKING_POINTS.md)** - What to say on each slide
3. Practice with **[DIAGRAMS.md](./DIAGRAMS.md)** - Visual explanations

## 📄 Document Guide

### Core Documents

#### [slides.md](./slides.md) - The Presentation
- **Type:** Slidev presentation
- **Length:** 8-10 slides
- **Audience:** Technical and business stakeholders
- **Content:**
  - Platform introduction
  - Problem/solution
  - Architecture overview
  - Metadata pruning deep dive
  - Technical stack details
  - Comparison with alternatives
  - Complete data flow

#### [ONE_PAGER.md](./ONE_PAGER.md) - Quick Summary
- **Type:** Executive summary
- **Length:** 1 page
- **Audience:** Executives, decision makers
- **Content:**
  - What is Fig?
  - Key innovation
  - Value proposition
  - Cost comparison table
  - Implementation roadmap
  - ROI analysis

#### [SUMMARY.md](./SUMMARY.md) - Executive Summary
- **Type:** Detailed executive summary with recommendations
- **Length:** 15-20 pages
- **Audience:** Technical leads, executives
- **Content:**
  - Value propositions
  - Technical innovation explained
  - Comparison with alternatives
  - Implementation roadmap (phased)
  - Risk mitigation strategies
  - Success metrics
  - Investment required
  - Key recommendations

### Technical Documentation

#### [ARCHITECTURE.md](./ARCHITECTURE.md) - Technical Deep Dive
- **Type:** Complete architecture documentation
- **Length:** 30+ pages
- **Audience:** Engineers, architects
- **Content:**
  - Three-layer architecture (hot/CDC/cold)
  - Query engine design
  - Metadata pruning strategy
  - Technical decisions and rationale
  - Cost analysis
  - Implementation roadmap
  - Security considerations
  - Monitoring and observability

#### [DIAGRAMS.md](./DIAGRAMS.md) - Visual Documentation
- **Type:** ASCII diagrams and visual explanations
- **Length:** 10-15 pages
- **Audience:** Visual learners, engineers
- **Content:**
  - Complete query flow
  - Step-by-step pruning examples
  - CDC flow diagram
  - Cost breakdown visuals
  - Execution examples
  - Before/after comparisons

#### [REFERENCE.md](./REFERENCE.md) - Quick Reference Guide
- **Type:** Practical reference with examples
- **Length:** 20-25 pages
- **Audience:** Developers, operators
- **Content:**
  - Component comparison matrix
  - Query decision tree
  - Metrics dashboard
  - Technology stack details
  - Common queries with explanations
  - Troubleshooting guide
  - Monitoring commands
  - Best practices
  - Cost calculator

### Presentation Support

#### [TALKING_POINTS.md](./TALKING_POINTS.md) - Presentation Script
- **Type:** Speaking notes and Q&A
- **Length:** 25-30 pages
- **Audience:** Presenters
- **Content:**
  - Slide-by-slide talking points
  - Key messages per slide
  - Analogies and explanations
  - Transition phrases
  - Anticipated questions with answers
  - Demo scenarios (if applicable)

#### [README.md](./README.md) - Getting Started
- **Type:** Quick start guide
- **Length:** 2-3 pages
- **Audience:** Anyone getting started
- **Content:**
  - What is Fig?
  - Running the presentation
  - Document overview
  - Architecture inspiration
  - Quick reference

## 🎨 Presentation Guide

### Running the Slides

```bash
# Install dependencies
pnpm install

# Start development server
pnpm dev

# Build for production
pnpm build

# Export to PDF
pnpm export
```

### Customizing the Presentation

**Edit [slides.md](./slides.md) to:**
- Add your company logo
- Update cost examples with your data
- Add specific use cases
- Include your team's contact info

**Key sections to customize:**
- Hero slide (line 37-85): Update team names, logo
- Cost comparison (line 287-330): Update with your numbers
- Final slide: Add call to action for your organization

## 📊 Key Concepts Explained

### The Big Idea
**Query petabytes in seconds by reading only megabytes**

How? **Three-level metadata pruning:**
1. Partition pruning (skip entire days/regions)
2. File pruning (skip files using min/max stats)
3. Row group pruning (skip chunks within files)

### The Architecture
**Three tiers working together:**
1. **Hot (Postgres):** Recent data, <50ms queries
2. **CDC (RisingWave):** Real-time replication
3. **Cold (R2 + Iceberg):** Historical data, serverless queries

### The Innovation
**Inspired by Cloudflare R2 SQL:**
- Streaming query pipeline (start early, finish early)
- Distributed execution on edge workers
- Apache DataFusion for vectorized processing
- Zero cluster management

## 💡 Using This Documentation

### For a 30-Minute Presentation
1. Use **[slides.md](./slides.md)** as your deck
2. Reference **[TALKING_POINTS.md](./TALKING_POINTS.md)** for what to say
3. Have **[ONE_PAGER.md](./ONE_PAGER.md)** as handout

### For a Technical Deep Dive (1-2 hours)
1. Present **[slides.md](./slides.md)** (30 min)
2. Walk through **[DIAGRAMS.md](./DIAGRAMS.md)** (30 min)
3. Q&A using **[ARCHITECTURE.md](./ARCHITECTURE.md)** (30-60 min)

### For a Proposal Document
Combine:
1. **[ONE_PAGER.md](./ONE_PAGER.md)** - Executive summary
2. **[SUMMARY.md](./SUMMARY.md)** - Detailed recommendations
3. **[ARCHITECTURE.md](./ARCHITECTURE.md)** - Technical appendix
4. **[REFERENCE.md](./REFERENCE.md)** - Implementation guide

### For Team Onboarding
Reading order:
1. **[ONE_PAGER.md](./ONE_PAGER.md)** - Get the big picture
2. **[DIAGRAMS.md](./DIAGRAMS.md)** - Understand the flow
3. **[ARCHITECTURE.md](./ARCHITECTURE.md)** - Deep technical knowledge
4. **[REFERENCE.md](./REFERENCE.md)** - Practical reference

## 🔗 External References

### Primary Inspiration
- [Cloudflare R2 SQL Deep Dive](https://blog.cloudflare.com/r2-sql-deep-dive/) - The blog post that inspired this architecture

### Key Technologies
- [Apache Iceberg](https://iceberg.apache.org/) - Open table format
- [Apache Parquet](https://parquet.apache.org/) - Columnar storage
- [Apache Arrow](https://arrow.apache.org/) - In-memory format
- [Apache DataFusion](https://arrow.apache.org/datafusion/) - Query engine
- [RisingWave](https://www.risingwave.dev/) - Streaming database
- [Cloudflare R2](https://www.cloudflare.com/products/r2/) - Object storage

### Related Reading
- [The Data Warehouse Toolkit](https://www.kimballgroup.com/) - Dimensional modeling
- [Designing Data-Intensive Applications](https://dataintensive.net/) - Distributed systems
- [The Log](https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying) - Streaming fundamentals

## 📈 Success Stories

This architecture is based on proven concepts:

**Cloudflare:**
- Queries petabytes of their own data using R2 SQL
- Serverless execution on their edge network
- Production-proven at massive scale

**Apache Iceberg:**
- Used by Netflix, Apple, Adobe, LinkedIn
- Petabyte-scale production deployments
- Open standard with broad tooling support

**Apache DataFusion:**
- Used by Ballista, DataBend, GreptimeDB
- High-performance Rust-based execution
- Growing ecosystem

## 🎯 Next Steps

### To Present This
1. Review [slides.md](./slides.md)
2. Study [TALKING_POINTS.md](./TALKING_POINTS.md)
3. Practice with [DIAGRAMS.md](./DIAGRAMS.md)
4. Print [ONE_PAGER.md](./ONE_PAGER.md) as handout

### To Implement This
1. Read [SUMMARY.md](./SUMMARY.md) for recommendations
2. Study [ARCHITECTURE.md](./ARCHITECTURE.md) for technical details
3. Use [REFERENCE.md](./REFERENCE.md) as implementation guide
4. Follow phased approach in [SUMMARY.md](./SUMMARY.md)

### To Customize This
1. Update slides with your branding
2. Modify cost examples with your numbers
3. Add your specific use cases
4. Include your team's expertise

## 📞 Contact

For questions about this documentation:
- **Matthew Aylward** - Platform architecture
- **Jan Nelis** - CDC streaming and infrastructure

---

**Fig: The future of data platforms is serverless, distributed, and open.**

Last Updated: October 2, 2025
