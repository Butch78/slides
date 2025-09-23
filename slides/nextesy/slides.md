---
# You can also start simply with 'default'
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://cover.sli.dev
# some information about your slides (markdown enabled)
title: AI & Data Engineer Interview
info: |
  ## Technical Interview Presentation
  Showcasing Data Pipeline Engineering and Cross-Cloud Migration Expertise
  
  Demonstrating practical skills in AWS → GCP migration, ML pipelines, and orchestration

  Learn more at [Sli.dev](https://sli.dev)
# apply unocss classes to the current slide
class: text-center
# https://sli.dev/features/drawing
drawings:
  persist: false
# slide transition: https://sli.dev/guide/animations.html#slide-transitions
transition: slide-left
# enable MDC Syntax: https://sli.dev/features/mdc
mdc: true
# open graph
seoMeta:
  ogImage: https://cover.sli.dev
  ogTitle: AI & Data Engineer Interview
  ogDescription: Technical Interview Presentation - Data Pipeline Engineering and Cross-Cloud Migration
routerMode: hash

addons:
  - excalidraw

---

# AI & Data Engineer Technical Interview

**Building Scalable Cross-Cloud Data Infrastructure**

*Matthew Aylward, Milan Kuzmanovic PhD, Miloš Čalija*

<div @click="$slidev.nav.next" class="mt-12 py-1" hover:bg="white op-10">
  Press Space for next page <carbon:arrow-right />
</div>

<div class="mt-8 text-lg">
  <div class="flex items-center justify-center space-x-8">
    <div class="flex flex-col items-center">
      <carbon:cloud-upload class="text-4xl mb-2 text-blue-400" />
      <span class="text-sm">Cross-Cloud Data Migration</span>
    </div>
    <div class="flex flex-col items-center">
      <carbon:data-structured class="text-4xl mb-2 text-green-400" />
      <span class="text-sm">Production ML Pipelines</span>
    </div>
    <div class="flex flex-col items-center">
      <carbon:workflow-automation class="text-4xl mb-2 text-purple-400" />
      <span class="text-sm">Enterprise Orchestration</span>
    </div>
  </div>
</div>

<div class="abs-br m-6 text-xl">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="slidev-icon-btn">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" class="slidev-icon-btn">
    <carbon:logo-github />
  </a>
</div>




<!--
Welcome to this presentation that demonstrates my solution with cross-cloud data engineering, ML pipeline development, and production-ready orchestration systems.
-->



---
transition: fade-out
---


## Challenge

<div v-click="1" class="bg-gray-100 dark:bg-gray-800 p-2 gap-2 rounded text-center">
  <carbon:warning class="text-red-500 mx-auto mb-1 text-lg" />
  <strong class="text-sm block">Challenge</strong>
  <p class="text-xs mt-1">Enterprise data migration from AWS to GCP at scale</p>
</div>

<br/>

## Solution

<div v-click="2"class="mt-4 grid grid-cols-4 gap-2">


<div v-click="3" class="bg-gray-100 dark:bg-gray-800 p-2 rounded text-center">
  <carbon:light class="text-yellow-500 mx-auto mb-1 text-lg" />
  <strong class="text-sm block">Architecture</strong>
  <p class="text-xs mt-1">Modern lakehouse with Apache Iceberg & Parquet optimization</p>
</div>

<div v-click="3" class="bg-gray-100 dark:bg-gray-800 p-2 rounded text-center">
  <carbon:tools class="text-blue-500 mx-auto mb-1 text-lg" />
  <strong class="text-sm block">Technology</strong>
  <p class="text-xs mt-1">Python, Rust, Orchestration, Docker, Apache Iceberg</p>
</div>

<div v-click="4" class="bg-gray-100 dark:bg-gray-800 p-2 rounded text-center">
  <carbon:result class="text-green-500 mx-auto mb-1 text-lg" />
  <strong class="text-sm block">Impact</strong>
  <p class="text-xs mt-1">Production-ready pipeline, optimized for performance and cost</p>
</div>

<div v-click="5" class="bg-gray-100 dark:bg-gray-800 p-2 rounded text-center">
  <carbon:arrow-right class="text-purple-500 mx-auto mb-1 text-lg" />
  <strong class="text-sm block">Evolution</strong>
  
  <p class="text-xs mt-1">MLOps integration and real-time streaming capabilities</p>
</div>

</div>

<!--
As you have outlined to me Milan, Nextesy will soon be moving large volumes of enterprise data from AWS to GCP for ML model processing. To ensure we can scale to any amount of clients, I proposed a modern lakehouse architecture leveraging Apache Iceberg for data storage and Parquet for efficient data serialization.
-->

---
transition: slide-up
layout: two-cols
layoutClass: gap-6
---
## Project Overview

<div class="space-y-4">

**Challenge:** Moving Enterprise data from AWS → GCP for ML model processing

**Solution:** Modern Lakehouse architecture with Parquet optimization and Orchestration

<div class="bg-blue-50 dark:bg-blue-900/20 p-4 rounded-lg">
  <h4 class="font-bold mb-2">Key Components:</h4>
  <ul class="text-sm space-y-1">
    <li>Enterprise data ingestion and processing</li>
    <li>High-performance Parquet compression with Rust</li>
    <li>Apache Iceberg lakehouse architecture</li>
    <li>Seamless AWS to GCP cross-cloud transfer</li>
    <li>Production-ready orchestration pipeline</li>
  </ul>
</div>

</div>

::right::

## Architecture Flow

<div class="flex justify-center">
  <div class="mt-4 p-4 rounded-lg">

```mermaid {scale: 0.8}
graph TD
    A[Application Data] --> B[Data Processing<br/>& Transformation]
    
    %% Hot Path - Vector DB
    B -->|Hot Data<br/>Recent/Active| D[Vector Database<br/>Real-time Queries]
    D --> E[Real-time Queries<br/>]
    
    %% Cold Path - Apache Iceberg
    B -->|Cold Data<br/>Historical/Archive| F[Apache Iceberg/<br/>S3 Vector Storage]

    
    F --> G[AWS to GCP<br/>ETL Transfer]
    
    %% Styling
    style A fill:#ff9999
    style B fill:#ffeb3b
    style D fill:#4caf50
    style F fill:#ffcc99
    style G fill:#99ff99
```

  </div>
</div>


<!--
We will have both Hot and Cold storage architecture for embeddings. Hot data for real-time low-latency access via a Vector DB, and Cold data for cost-effective archival storage. Also, once the data has been processed and stored in S3, we will transfer it to GCP for ML model processing.
-->


---
transition: fade-out
---

# Hot & Cold Storage Architecture for Embeddings

<div class="mt-6 space-y-3">

<div class="bg-gradient-to-r from-red-400 to-orange-400 text-white p-2 rounded-lg">
  <div class="grid grid-cols-4 gap-4 items-center">
    <div class="font-bold text-lg">🔥 Hot Data Layer</div>
    <div class="text-center">
      <div class="font-bold">< 50ms</div>
      <div class="text-xs">Latency</div>
    </div>
    <div class="text-sm">
      <div>🔍 Real-time search</div>
      <div>📢 Live user interactions</div>
    </div>
    <div class="text-sm">
      <div class="font-semibold">Pinecone, Milvus</div>
      <div class="text-xs">Traditional vector databases</div>
    </div>
  </div>
</div>

</div>

<div class="bg-gradient-to-r from-blue-400 to-blue-600 text-white p-4 rounded-lg">
  <div class="grid grid-cols-4 gap-4 items-center">
    <div class="font-bold text-lg">❄️ Cold Data Layer</div>
    <div class="text-center">
      <div class="font-bold">> 500ms</div>
      <div class="text-xs">Latency</div>
    </div>
    <div class="text-sm">
      <div>📦 Historical data archiving</div>
      <div>🔍 Data lake queries</div>
    </div>
    <div class="text-sm">
      <div class="font-semibold">S3 + Apache Iceberg</div>
      <div class="text-xs">Cost-optimized storage</div>
    </div>
  </div>
</div>


<div class="mt-4 grid grid-cols-3 gap-4">

<div class="bg-red-50 dark:bg-red-900/20 p-3 rounded-lg text-center">
  <h4 class="font-bold text-red-700 dark:text-red-300 mb-2 text-sm">🔥 Hot Storage (Pinecone)</h4>
  <p class="text-xs mb-2">10M vectors, 250k queries/month</p>
  <div class="text-lg font-bold text-red-600">$300–$500</div>
  <p class="text-xs italic">Always-on, high-performance</p>
</div>

<div class="bg-blue-50 dark:bg-blue-900/20 p-3 rounded-lg text-center">
  <h4 class="font-bold text-blue-700 dark:text-blue-300 mb-2 text-sm">❄️ Cold Storage (S3)</h4>
  <p class="text-xs mb-2">Same data, archival access</p>
  <div class="text-lg font-bold text-blue-600">$30–$50</div>
  <p class="text-xs italic">Pay-as-you-go model</p>
</div>

<div class="bg-green-50 dark:bg-green-900/20 p-3 rounded-lg text-center">
  <h4 class="font-bold text-green-700 dark:text-green-300 mb-2 text-sm">💡 Hybrid Strategy</h4>
  <p class="text-xs mb-2">Hot + Cold tier optimization</p>
  <div class="text-lg font-bold text-green-600">70-90%</div>
  <p class="text-xs italic">Total cost reduction</p>
</div>

</div>


<div class="mt-4 text-center">
  <p class="text-xs italic text-gray-600 dark:text-gray-400">
    Source: <a href="https://zilliz.com/blog/will-amazon-s3-vectors-kill-vector-databases-or-save-them" class="underline">Will Amazon S3 Vectors Kill Vector Databases or Save Them?</a>
  </p>
</div>




--- 

# Cost Benefit Analysis - Parquet vs CSV

*~32k rows × 768-dim float32 embeddings, ~94 MB in memory*

<div class="mt-4">

| Format | Disk Size | Portability | Performance | Cost Impact |
|--------|-----------|-------------|-------------|-------------|
| **CSV** | ~630 MB | ✅ Any tool | ❌ Slow I/O | **High storage + transfer** |
| **Numpy** | ~95 MB | ⚠️ Limited metadata | ⚡ Fast | Neutral, lacks flexibility |
| **Parquet** | ~60 MB | ✅ Open standard | ⚡ Fast columnar | **Lowest cost, scalable** |

</div>

<div class="mt-6 grid grid-cols-2 gap-4">

<div class="bg-green-50 dark:bg-green-900/20 p-3 rounded-lg">
  <h4 class="font-bold text-green-700 dark:text-green-300 mb-2">✅ Parquet Benefits</h4>
  <ul class="text-xs space-y-1">
    <li>10x storage reduction vs CSV</li>
    <li>Zero-copy read with Polars</li>
    <li>Cross-platform compatibility</li>
    <li>Storage in Apache Iceberg</li>
  </ul>
</div>

<div class="bg-blue-50 dark:bg-blue-900/20 p-3 rounded-lg">
  <h4 class="font-bold text-blue-700 dark:text-blue-300 mb-2">💰 AWS → GCP Transfer Cost</h4>
  <ul class="text-xs space-y-1">
    <li>CSV: 630 MB × $0.09/GB ≈ $0.06 per dataset</li>
    <li>Parquet: 60 MB × $0.09/GB ≈ $0.005 per dataset</li>
    <li class="font-bold text-green-600">~90% cheaper to move across clouds</li>
  </ul>
</div>

</div>

<div class="mt-4 text-center">
  <p class="text-xs italic text-gray-600 dark:text-gray-400">
    Source: <a href="https://minimaxir.com/2025/02/embeddings-parquet/" class="underline">Embeddings & Parquet Performance Analysis</a>
  </p>
</div>



--- 
layout: two-cols
layoutClass: gap-6
---


# ETL Implementation Details

<div class="space-y-4">

<div v-click="1" class="border-l-4 border-blue-500 pl-4">
  <h4 class="font-bold text-blue-600">Python Core Application</h4>
  <p class="text-sm">Data science workflows, API integration, ML preprocessing</p>
</div>

<div v-click="2" class="border-l-4 border-orange-500 pl-4">
  <h4 class="font-bold text-orange-600">Rust for Parquet Conversion</h4>
  <p class="text-sm">Performance-critical transformations, memory efficiency</p>
  <p class="text-xs italic">Considering Lambda deployment for serverless scaling</p>
</div>

<div v-click="3" class="border-l-4 border-green-500 pl-4">
  <h4 class="font-bold text-green-600">Source of Truth: Apache Iceberg in AWS</h4>
  <p class="text-sm">Source of truth for data lake, enabling ACID transactions</p>
</div>

<div v-click="4" class="border-l-4 border-purple-500 pl-4">
  <h4 class="font-bold text-purple-600">Prefect Orchestration</h4>
  <p class="text-sm">Workflow management, retry logic, observability, scheduling</p>
</div>

</div>

::right::

```mermaid {scale: 0.8}
graph TD
    A[Application Data] --> B[Data Processing<br/>& Transformation]
    
    
    %% Cold Path - Apache Iceberg
    B -->|Cold Data<br/>Historical/Archive| F[Apache Iceberg/<br/>S3 Vector Storage]

    F --> |ETL Tool to pull data from AWS| G[AWS to GCP<br/>ETL Transfer]

    %% Styling
    style A fill:#ff9999
    style B fill:#ffeb3b
    style F fill:#ffcc99
    style G fill:#99ff99
```

---
layout: two-cols
layoutClass: gap-4
---

## 📈 Scaling Architecture

<div class="space-y-3 mt-6">
  <div class="bg-blue-50 dark:bg-blue-900/20 p-3 rounded">
    <strong>Distributed Parallel Processing</strong>
    <p class="text-sm">Orchestration-orchestrated parallel task execution with intelligent batching for optimal throughput</p>
  </div>
  
  <div class="bg-purple-50 dark:bg-purple-900/20 p-3 rounded">
    <strong>Serverless Auto-Scaling</strong>
    <p class="text-sm">AWS Lambda deployment for Rust converters enabling elastic scaling based on demand</p>
  </div>
  
  <div class="bg-green-50 dark:bg-green-900/20 p-3 rounded">
    <strong>Future: Hybrid Processing Strategy</strong>
    <p class="text-sm">Streams for real-time analytics, batch processing for high-volume historical data</p>
  </div>
</div>

::right::


## 🚀 Production MLOps Pipeline

<div class="space-y-3 mt-4">
  <div class="border border-blue-300 p-3 rounded">
    <strong class="text-blue-600">GCP AI Platform Integration</strong>
    <p class="text-sm">Managed ML model serving with auto-scaling inference and built-in monitoring</p>
  </div>
  
  <div class="border border-purple-300 p-3 rounded">
    <strong class="text-purple-600">Containerized Model Deployment</strong>
    <p class="text-sm">Docker-based deployments with semantic versioning and rollback capabilities</p>
  </div>
  
  <div class="border border-green-300 p-3 rounded">
    <strong class="text-green-600">Continuous Model Validation</strong>
    <p class="text-sm">A/B testing framework with automated performance tracking and model drift detection</p>
  </div>
</div>

--- 
transition: fade-out
---


# Potential Hazards

Cloud Lock-in

Mitigation Strategies:
- Use open standards (Apache Iceberg, Parquet)
- Abstract cloud-specific APIs
- Containerization with Docker
- Multi-cloud orchestration with Prefect
<br/>


---
class: px-20
---

# Strategic Thinking: Business Impact

<div class="grid grid-cols-3 gap-6 mt-8">

<div v-click="1">
  <div class="bg-gradient-to-br from-blue-500 to-blue-600 text-white p-4 rounded-lg h-full">
    <carbon:dashboard class="text-3xl mb-3" />
    <h3 class="font-bold mb-2">Observability</h3>
    <ul class="text-sm space-y-1">
      <li>Prometheus metrics</li>
      <li>CloudWatch/GCP monitoring</li>
      <li>Custom data quality alerts</li>
      <li>Pipeline SLA tracking</li>
    </ul>
  </div>
</div>

<div v-click="2">
  <div class="bg-gradient-to-br from-green-500 to-green-600 text-white p-4 rounded-lg h-full">
    <carbon:deployment-unit-technical-execution class="text-3xl mb-3" />
    <h3 class="font-bold mb-2">CI/CD for Data</h3>
    <ul class="text-sm space-y-1">
      <li>Automated ETL testing</li>
      <li>Schema validation</li>
      <li>Data quality gates</li>
      <li>Deployment automation</li>
    </ul>
  </div>
</div>

<div v-click="3">
  <div class="bg-gradient-to-br from-purple-500 to-purple-600 text-white p-4 rounded-lg h-full">
    <carbon:growth class="text-3xl mb-3" />
    <h3 class="font-bold mb-2">Business Value</h3>
    <ul class="text-sm space-y-1">
      <li>Faster ML model iteration</li>
      <li>Reduced data processing costs</li>
      <li>Improved decision latency</li>
      <li>Cross-cloud flexibility</li>
    </ul>
  </div>
</div>

</div>

<!--
Shows enterprise-level thinking about data engineering operations and business impact
-->


---


# Technical Q&A

<div class="space-y-4">

<div class="bg-red-50 dark:bg-red-900/20 p-3 rounded">
  <strong class="text-red-700 dark:text-red-300">Q: "How do you handle failures?"</strong>
  <p class="text-sm mt-1">
    <strong>A:</strong> "Prefect provides built-in retry logic with exponential backoff. I implement circuit breakers, dead letter queues, and comprehensive logging with correlation IDs for debugging."
  </p>
</div>

<div v-click="1" class="bg-blue-50 dark:bg-blue-900/20 p-3 rounded">
  <strong class="text-blue-700 dark:text-blue-300">Q: "How do you scale this to thousands per day?"</strong>
  <p class="text-sm mt-1">
    <strong>A:</strong> "Parallel processing with Prefect Orchestration, serverless Rust functions on Lambda for auto-scaling, and choosing between batch vs streaming based on latency requirements."
  </p>
</div>

<div v-click="2" class="bg-purple-50 dark:bg-purple-900/20 p-3 rounded">
  <strong class="text-purple-700 dark:text-purple-300">Q: "How do you secure data during transfer?"</strong>
  <p class="text-sm mt-1">
    <strong>A:</strong> "End-to-end encryption, IAM roles with least privilege, VPC endpoints for private communication, and comprehensive audit logging."
  </p>
</div>

</div>

