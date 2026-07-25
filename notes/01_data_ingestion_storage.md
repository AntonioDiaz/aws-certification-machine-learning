# 01 — Data Ingestion & Storage <!-- omit in toc -->

> Source: [`01_MLA-C01_data_ingestion_storage.pdf`](../udemy_notes/01_MLA-C01_data_ingestion_storage.pdf)

---

## Table of Contents <!-- omit in toc -->

- [Why This Topic Matters](#why-this-topic-matters)
- [Types of Data](#types-of-data)
- [Properties of Data (The 3 V's)](#properties-of-data-the-3-vs)
- [Data Storage Architectures](#data-storage-architectures)
  - [Data Warehouse](#data-warehouse)
  - [Data Lake](#data-lake)
  - [Comparison: Warehouse vs. Lake](#comparison-warehouse-vs-lake)
  - [Data Lakehouse](#data-lakehouse)
  - [Data Mesh](#data-mesh)
- [ETL Pipelines](#etl-pipelines)
- [Data Formats](#data-formats)
- [Amazon S3](#amazon-s3)
  - [Key Concepts](#key-concepts)
  - [Security](#security)
  - [Versioning](#versioning)
  - [Replication](#replication)
  - [Storage Classes](#storage-classes)
  - [Lifecycle Rules](#lifecycle-rules)
  - [Event Notifications](#event-notifications)
  - [Performance](#performance)
  - [Encryption](#encryption)
  - [Access Points & Object Lambda](#access-points--object-lambda)
- [EC2 Instance Storage](#ec2-instance-storage)
  - [EBS (Elastic Block Store)](#ebs-elastic-block-store)
  - [EFS (Elastic File System)](#efs-elastic-file-system)
  - [EBS vs. EFS](#ebs-vs-efs)
  - [Amazon FSx](#amazon-fsx)
  - [FSx for Lustre Deployment Options](#fsx-for-lustre-deployment-options)
- [Amazon Kinesis](#amazon-kinesis)
  - [Kinesis Data Streams (KDS)](#kinesis-data-streams-kds)
  - [Amazon Data Firehose (ADF)](#amazon-data-firehose-adf)
  - [Data Streams vs. Firehose](#data-streams-vs-firehose)
  - [Producer Troubleshooting](#producer-troubleshooting)
  - [Consumer Troubleshooting](#consumer-troubleshooting)
  - [Managed Service for Apache Flink (MSAF)](#managed-service-for-apache-flink-msaf)
- [Amazon MSK (Managed Streaming for Apache Kafka)](#amazon-msk-managed-streaming-for-apache-kafka)
  - [MSK vs. Kinesis Data Streams](#msk-vs-kinesis-data-streams)
  - [MSK Security](#msk-security)
  - [MSK Monitoring](#msk-monitoring)
  - [MSK Connect](#msk-connect)
  - [MSK Serverless](#msk-serverless)

---

## Why This Topic Matters

Every ML pipeline starts with data that lives somewhere and has to get somewhere else. This domain covers **where data rests** (S3, EBS, EFS, FSx) and **how it moves** (Kinesis, MSK, ETL pipelines). It is the foundation the rest of the exam builds on: you cannot reason about SageMaker training input modes or a Bedrock RAG pipeline without knowing the storage layer underneath.

The mental model is a two-axis decision. First axis: **is the data at rest or in motion?** Data at rest goes to object storage (S3), block storage (EBS), or file storage (EFS/FSx). Data in motion goes to a streaming service (Kinesis, MSK). Second axis: **who needs to read it, and how fast?** One instance or many, one AZ or several, milliseconds or minutes.

**On the MLA-C01 exam**, this domain tests your ability to:
- Pick the right storage service given access patterns, latency needs, and number of consumers.
- Choose between Kinesis Data Streams and Amazon Data Firehose based on latency, replay, and transformation requirements.
- Match a data format (CSV, JSON, Avro, Parquet) to a workload — this drives both cost and query performance.
- Apply S3 lifecycle rules and storage classes to hit a cost target without breaking retrieval SLAs.
- Recognize when a scenario calls for Kafka compatibility (MSK) rather than a native AWS stream (Kinesis).

> [!TIP]
> Most storage questions collapse to a single discriminator. "Shared across hundreds of instances" → EFS. "Single instance, low latency block device" → EBS. "Training data for SageMaker at sub-ms latency" → FSx for Lustre. "Anything else, at scale, cheap" → S3.

---

## Types of Data

Data type determines what you can do with the data before you transform it. Structured data is queryable as-is; unstructured data needs a preprocessing step before any model can consume it. This classification drives the choice of storage architecture in the next section.

| Type | Definition | Characteristics | Examples |
|---|---|---|---|
| **Structured** | Organized in a defined schema, typically in relational databases | Easily queryable, rows & columns, consistent structure | Database tables, CSV files, Excel spreadsheets |
| **Unstructured** | No predefined structure or schema | Not easily queryable without preprocessing, various formats | Text files, videos, images, emails |
| **Semi-Structured** | Some structure via tags, hierarchies, or patterns | More flexible than structured, less chaotic than unstructured | XML/JSON files, email headers, log files |

---

## Properties of Data (The 3 V's)

The 3 V's are the vocabulary AWS uses to justify why one service beats another. When an exam question stresses one of these dimensions, it is signalling which service family to reach for.

- **Volume** — the amount or size of data (GB to petabytes). Challenges in storing, processing, and analyzing at scale.
- **Velocity** — the speed at which data is generated and processed. High velocity requires real-time or near-real-time processing (e.g., IoT sensors, high-frequency trading).
- **Variety** — the different types, structures, and sources of data (structured + unstructured + semi-structured from multiple origins).

---

## Data Storage Architectures

These four architectures answer the same question — "where does the organization put its analytical data?" — with different trade-offs between structure and flexibility. The key axis is **when the schema is applied**: before writing (rigid, fast to query) or when reading (flexible, cheap to ingest).

<img width="2752" height="1536" alt="Image" src="https://github.com/user-attachments/assets/90365e14-df40-4cf6-a7c5-3eee295d98f5" />
&nbsp;

### Data Warehouse

- Centralized repository optimized for analysis with structured data.
- Schema-on-write (ETL: Extract → Transform → Load).
- Star or snowflake schema; optimized for read-heavy complex queries.
- Examples: Amazon Redshift, Google BigQuery, Azure SQL DW.

### Data Lake

- Stores vast amounts of raw data in native format (structured + semi + unstructured).
- Schema-on-read (ELT: Extract → Load → Transform).
- Supports batch, real-time, and stream processing.
- Examples: S3 as a data lake, Azure Data Lake Storage, HDFS.

### Comparison: Warehouse vs. Lake

| Dimension | Data Warehouse | Data Lake |
|---|---|---|
| Schema | Schema-on-write (ETL) | Schema-on-read (ELT) |
| Data Types | Primarily structured | Structured + unstructured |
| Agility | Less agile (predefined schema) | More agile (raw data) |
| Cost | Higher (query optimization) | Lower storage, higher processing |
| Use Case | BI & analytics, complex queries | ML, data discovery, flexible future needs |

> [!TIP]
> Organizations often use both — raw data in a lake, refined data in a warehouse. If a question offers "lake or warehouse" as an exclusive choice but the scenario describes both raw ingestion and BI reporting, the answer is usually a lakehouse.

### Data Lakehouse

- Hybrid combining the flexibility of a data lake with the performance of a data warehouse.
- Supports schema-on-write AND schema-on-read.
- ACID transactions via technologies like Delta Lake.
- Examples: AWS Lake Formation (S3 + Redshift Spectrum), Delta Lake, Databricks, Azure Synapse.

### Data Mesh

- Governance/organizational concept (coined 2019), not a specific technology.
- Individual teams own "data products" within a domain ("domain-based data management").
- Federated governance with central standards + self-service infrastructure.

> [!WARNING]
> Data Mesh is an organizational pattern, not an AWS service. If an answer option presents it as something you "deploy", it is a distractor.

---

## ETL Pipelines

ETL is the plumbing that moves data from where it is produced to where it is analyzed. On the exam the phases themselves are rarely the question — the orchestration tool is. Knowing which AWS service coordinates the pipeline matters more than knowing the definition of "transform".

| Phase | Description |
|---|---|
| **Extract** | Retrieve raw data from DBs, CRMs, flat files, APIs, streams. Real-time or batch. |
| **Transform** | Cleanse (remove duplicates), enrich, reformat dates/strings, aggregate, handle missing values |
| **Load** | Move transformed data to target warehouse — batch or streaming |

**ETL Orchestration tools on AWS:** AWS Glue · EventBridge · Amazon MWAA · AWS Step Functions · Lambda · Glue Workflows.

**Data Sources:** JDBC (platform-independent, language-dependent) · ODBC (platform-dependent, language-independent) · Raw logs · APIs · Streams.

> [!TIP]
> Orchestration shortcuts: complex branching and retries → Step Functions. Existing Airflow DAGs → MWAA. Simple event-driven trigger → EventBridge + Lambda. ETL native to the Glue Data Catalog → Glue Workflows.

---

## Data Formats

Format choice is a cost and performance decision, not a stylistic one. The split that matters is **row-oriented vs. columnar**: row formats are efficient when you write records one at a time or read whole rows; columnar formats are efficient when you scan a few columns across millions of rows — exactly what analytics and ML feature extraction do.

| Format | Type | Best For | Systems |
|---|---|---|---|
| **CSV** | Text, tabular | Small–medium datasets, human-readable interchange | Databases, Excel, Pandas, ETL tools |
| **JSON** | Text, key-value | Web APIs, flexible/nested schemas, configs | REST APIs, NoSQL, web browsers |
| **Avro** | Binary + schema | Big data, schema evolution, efficient serialization | Kafka, Spark, Flink, Hadoop |
| **Parquet** | Columnar binary | Analytics on large datasets, column-selective reads | Spark, Hive, Impala, Redshift Spectrum |

> [!IMPORTANT]
> **Parquet is the default right answer for analytics and ML on AWS.** It reduces Athena and Redshift Spectrum scan costs because those services bill per byte scanned, and columnar storage lets them skip columns entirely. Reach for Avro instead when the scenario emphasizes schema evolution or Kafka producers.

---

## Amazon S3

S3 is the backbone of nearly every ML workload on AWS: it is where training data lands, where model artifacts are written, and where inference output is stored. Its scale is effectively unlimited, which means the exam focuses less on "can it hold this?" and more on **cost, access control, and durability trade-offs**.

<img width="2752" height="1536" alt="Image" src="https://github.com/user-attachments/assets/a97b471b-28d1-4a23-bf9e-d1ff48480921" />
&nbsp;

### Key Concepts

- Objects stored in globally-unique-named **buckets** (region-level).
- Object key = full path (prefix + object name); no real directories.
- Max object size: **5 TB** (multi-part upload required for > 5 GB).
- Objects can have metadata, tags (up to 10), and version IDs.

### Security

- **IAM Policies** — user-based API access control.
- **Bucket Policies** — resource-based (JSON), supports cross-account.
- **ACLs** — object or bucket level (can be disabled).
- Access granted when: IAM allows OR resource policy allows, AND no explicit DENY.

### Versioning

- Enabled at bucket level; protects against accidental deletes and allows rollback.
- Files not versioned before enabling get version `null`.
- Suspending versioning does not delete previous versions.

### Replication

- Requires versioning enabled on source and destination.
- **CRR** (Cross-Region): compliance, lower latency, cross-account replication.
- **SRR** (Same-Region): log aggregation, prod/test sync.
- Only new objects are replicated after activation; use S3 Batch Replication for existing objects.
- Delete markers optionally replicated; version-ID deletions are NOT replicated (prevents malicious deletes).
- No chaining: bucket1 → bucket2 → bucket3 does NOT auto-replicate bucket1 to bucket3.

> [!WARNING]
> Two replication gotchas the exam likes: existing objects are **not** back-filled when you enable replication (you need S3 Batch Replication), and replication is **not transitive** across a chain of buckets.

### Storage Classes

| Class | Availability | Min Duration | Retrieval Fee | Use Case |
|---|---|---|---|---|
| Standard | 99.99% | None | None | Frequently accessed data |
| Intelligent-Tiering | 99.9% | None | None | Unknown/changing access patterns |
| Standard-IA | 99.9% | 30 days | Per GB | Disaster recovery, backups |
| One Zone-IA | 99.5% | 30 days | Per GB | Recreatable secondary backups |
| Glacier Instant Retrieval | 99.9% | 90 days | Per GB | Quarterly access, ms retrieval |
| Glacier Flexible Retrieval | 99.99% | 90 days | Per GB | 1–5 min (expedited), 3–5 h (standard), 5–12 h (bulk) |
| Glacier Deep Archive | 99.99% | 180 days | Per GB | Long-term archiving; 12 h / 48 h retrieval |

All classes share **11 9's durability** (99.999999999%).

> [!IMPORTANT]
> Durability is identical across every class — only **availability**, minimum storage duration, and retrieval cost change. An answer that claims Glacier is "less durable" is wrong.

### Lifecycle Rules

- **Transition actions** — move objects to cheaper class after N days (e.g., Standard → Standard-IA after 60 days).
- **Expiration actions** — delete objects after N days.
- Rules can target by prefix or object tags.
- Use **S3 Analytics** (updated daily, 24–48 h to start) to get recommendations on Standard → Standard-IA transitions.

> [!TIP]
> "Access pattern is unknown or changes over time" → Intelligent-Tiering, not a lifecycle rule. Lifecycle rules require you to know the pattern in advance.

### Event Notifications

- Triggers: `ObjectCreated`, `ObjectRemoved`, `ObjectRestore`, `Replication`, etc.
- Destinations: Lambda, SQS, SNS.
- Via **EventBridge**: advanced JSON filtering, 18+ destinations (Step Functions, Kinesis, etc.), archive & replay.

### Performance

- Baseline: 3,500 PUT/COPY/POST/DELETE and 5,500 GET/HEAD requests/sec per prefix.
- **Multi-Part Upload**: recommended > 100 MB, required > 5 GB; parallelizes uploads.
- **Transfer Acceleration**: routes through AWS Edge Locations for faster inter-region transfers.
- **Byte-Range Fetches**: parallelize GETs by requesting specific byte ranges; also retrieves partial files.

> [!TIP]
> The request limits are **per prefix**, so spreading objects across more prefixes multiplies throughput. This is why partitioned S3 layouts (`year=/month=/day=`) help both cost and performance.

### Encryption

| Method | Key Owner | Notes |
|---|---|---|
| SSE-S3 | AWS | AES-256; default for new objects; header `x-amz-server-side-encryption: AES256` |
| SSE-KMS | AWS KMS | Audit via CloudTrail; header `aws:kms`; subject to KMS quota limits |
| SSE-C | Customer | HTTPS required; key sent in every HTTP request header |
| Client-Side | Customer | Encrypted before upload; customer manages full key lifecycle |

- Bucket Policies are evaluated **before** default encryption settings.
- Force HTTPS via `aws:SecureTransport` bucket policy condition.

> [!WARNING]
> SSE-KMS is subject to **KMS API quotas**. A high-throughput training job reading millions of small objects can hit `ThrottlingException` — a classic exam scenario where the fix is SSE-S3 or fewer, larger objects.

### Access Points & Object Lambda

- **Access Points**: per-prefix DNS endpoints with their own IAM-like policies; simplify multi-team bucket access. Can be VPC-restricted.
- **S3 Object Lambda**: attach a Lambda function to transform objects on the fly before they reach the caller (e.g., redact PII, convert XML to JSON).

> [!TIP]
> "Different teams need different views of the same bucket" → Access Points. "Same object, different content per caller (redacted vs. full)" → S3 Object Lambda.

---

## EC2 Instance Storage

When SageMaker or an EC2-based training job needs a filesystem rather than object storage, the choice is between block storage (EBS), managed NFS (EFS), and purpose-built high-performance filesystems (FSx). The discriminators are **how many instances mount it**, **how many AZs it spans**, and **what latency the workload needs**.

<img width="2752" height="1536" alt="Image" src="https://github.com/user-attachments/assets/f754ad02-f07f-4459-844f-a2cc0206a337" />

### EBS (Elastic Block Store)

- Network drive attached to one instance (io1/io2 support multi-attach).
- Bound to a single AZ; migrate across AZ via snapshot.
- Elastic Volumes: increase size or change type without restart (can only increase, not decrease).
- Delete on Termination: root volume deleted by default; other volumes are not.

### EFS (Elastic File System)

- Managed NFS mountable on hundreds of EC2 instances across multiple AZs.
- Linux only (POSIX), NFSv4.1, encryption at rest with KMS.
- Pay-per-use; scales automatically to petabyte scale.
- Performance modes: **General Purpose** (low latency) · **Max I/O** (high parallelism, big data).
- Throughput modes: **Bursting** · **Provisioned** · **Elastic** (auto-scales, best for unpredictable workloads).
- Storage tiers: Standard → Infrequent Access (EFS-IA) → Archive (50% cheaper).

### EBS vs. EFS

| | EBS | EFS |
|---|---|---|
| Instances | 1 (except io1/io2 multi-attach) | Hundreds across AZs |
| AZ scope | Single AZ | Multi-AZ |
| OS | Any | Linux only (POSIX) |
| Cost | Lower | Higher (~3× gp2) |

> [!WARNING]
> EBS volumes are **locked to a single AZ**. A distributed training job spread across AZs cannot share one EBS volume — that scenario needs EFS or FSx for Lustre.

### Amazon FSx

Amazon FSx provides fully managed file systems optimized for specific workloads. It supports multiple file system types, each with its own protocol and best use cases.

| Variant | Protocol | Best For |
|---|---|---|
| FSx for Windows | SMB, NTFS | Windows workloads, Active Directory, DFS |
| FSx for Lustre | POSIX | HPC, ML, video processing; S3 integration; sub-ms latency |
| FSx for NetApp ONTAP | NFS, SMB, iSCSI | Lift-and-shift ONTAP/NAS workloads; multi-OS |
| FSx for OpenZFS | NFS | ZFS migration; up to 1M IOPS, < 0.5ms latency |

> [!IMPORTANT]
> **FSx for Lustre is the ML answer.** It links directly to an S3 bucket and presents objects as files with sub-millisecond latency, which is why it appears as a SageMaker training input mode alongside File, Fast File, and Pipe.

### FSx for Lustre Deployment Options

FSx for Lustre is a high-performance file system optimized for workloads that require fast storage and processing. It can be deployed in two main configurations:

- **Scratch**: temporary, high burst (6× faster), no replication — short-term processing.
- **Persistent**: replicated within AZ, replace failed files in minutes — long-term sensitive data.

---

## Amazon Kinesis

Amazon Kinesis is a platform for real-time data streaming and analytics. It allows you to collect, process, and analyze streaming data at scale. The family has three members that get confused constantly on the exam, and the distinction is almost always **latency plus who manages scaling**: Data Streams is real-time and you manage shards, Firehose is near-real-time and fully managed, and Managed Service for Apache Flink queries the stream in flight.

<img width="2752" height="1536" alt="Image" src="https://github.com/user-attachments/assets/db0174e7-b180-4f23-9516-8368ee6972f3" />

### Kinesis Data Streams (KDS)

- Collect and store real-time streaming data.
- Retention up to **365 days**; replay capability.
- Data up to **1 MB**; ordering guaranteed per Partition Key.
- Encryption: KMS at-rest, HTTPS in-flight.
- Capacity modes:
  - **Provisioned**: choose shards manually (1 MB/s in, 2 MB/s out per shard); pay per shard-hour.
  - **On-Demand**: auto-scales based on 30-day peak; pay per GB in/out.

### Amazon Data Firehose (ADF)

Formerly Kinesis Data Firehose.

- Fully managed, serverless, near-real-time (buffered delivery).
- Destinations: S3, Redshift, OpenSearch, Splunk, Datadog, MongoDB, custom HTTP.
- Supports CSV, JSON, Parquet, Avro, Raw Text, Binary; converts to Parquet/ORC; compresses with gzip/snappy.
- Custom transformations via Lambda.

### Data Streams vs. Firehose

| | Kinesis Data Streams | Amazon Data Firehose |
|---|---|---|
| Latency | Real-time | Near real-time (buffered) |
| Management | Manual scaling | Automatic scaling |
| Storage | Up to 365 days | No storage |
| Replay | Yes | No |
| Use case | Custom producers & consumers | Load to S3/Redshift/OpenSearch/3rd party |

> [!IMPORTANT]
> **Firehose does not store data and cannot replay it.** Any scenario mentioning reprocessing historical records, multiple independent consumers, or sub-second latency rules Firehose out and points to Data Streams.

### Producer Troubleshooting

- Slow writes → check throttling limits; use better partition key distribution; use KPL + `PutRecords` for batching.
- 500/503 errors → error rate > 1%; implement retry with exponential backoff.
- Flink connection issues → check VPC config; adjust `RequestTimeout` and `#setQueueLimit`.

### Consumer Troubleshooting

- Skipped records → check unhandled exceptions in `processRecords`.
- Slow reads → increase shards; increase `maxRecords`; check for slow processing code.
- `ReadProvisionedThroughputExceeded` → reshard; use enhanced fan-out; exponential backoff.
- High latency → monitor `GetRecords.Latency` and `IteratorAge`; increase shards.

> [!TIP]
> A rising `IteratorAge` means consumers are falling behind producers. The fix is more shards or faster processing — not a bigger retention window.

### Managed Service for Apache Flink (MSAF)

Formerly Kinesis Data Analytics.

- Query streaming data with SQL or custom Flink applications (Java, Python, Scala).
- **Reference tables**: S3-backed JOIN data for cheap lookups (e.g., zip code → city mapping).
- Lambda as destination for post-processing (aggregate, transform, encrypt, route to other services).
- Pay per **Kinesis Processing Unit (KPU)** consumed: 1 KPU = 1 vCPU + 4 GB; serverless.
- **RANDOM_CUT_FOREST**: built-in SQL function for anomaly detection on numeric stream columns.

> [!IMPORTANT]
> `RANDOM_CUT_FOREST` in MSAF is the go-to answer for **real-time anomaly detection on a stream** without training a model. Do not confuse it with SageMaker's Random Cut Forest built-in algorithm, which operates on data at rest.

---

## Amazon MSK (Managed Streaming for Apache Kafka)

MSK exists for one reason on the exam: the scenario already uses Kafka, or needs a Kafka-compatible API. Functionally it overlaps heavily with Kinesis Data Streams, so questions hinge on message size limits, scaling model, and authentication options.

- Fully managed Apache Kafka — creates and manages Kafka brokers and ZooKeeper nodes.
- Deployed in VPC, multi-AZ (up to 3); data stored on EBS volumes.
- Default message size: **1 MB** (configurable up to 10 MB+).

### MSK vs. Kinesis Data Streams

| | Kinesis Data Streams | Amazon MSK |
|---|---|---|
| Message size | 1 MB | 1 MB default (configurable higher) |
| Scaling unit | Shards (split & merge) | Partitions (add only) |
| Encryption | TLS in-flight, KMS at-rest | PLAINTEXT or TLS in-flight, KMS at-rest |
| Auth | IAM | Mutual TLS + Kafka ACLs · SASL/SCRAM + Kafka ACLs · IAM |

> [!TIP]
> Two discriminators decide MSK vs. Kinesis: messages **larger than 1 MB** (only MSK supports it) and existing **Kafka tooling or migration** (MSK). Everything else favours Kinesis for its lower operational overhead.

> [!WARNING]
> MSK partitions can only be **added**, never removed. Kinesis shards support both split and merge. An answer claiming you can shrink an MSK cluster's partition count is wrong.

### MSK Security

- In-flight TLS between brokers and clients; KMS at-rest for EBS.
- Auth options: Mutual TLS (AuthN) + Kafka ACLs (AuthZ) · SASL/SCRAM + Kafka ACLs · IAM Access Control.

### MSK Monitoring

- CloudWatch: Basic (cluster/broker) → Enhanced (broker) → Topic-level.
- Prometheus: JMX Exporter (metrics) or Node Exporter (CPU/disk).
- Broker log delivery: CloudWatch Logs · S3 · Kinesis Data Streams.

### MSK Connect

Managed Kafka Connect workers with auto-scaling; deploy any connector (S3, Redshift, OpenSearch, Debezium…).

### MSK Serverless

Fully managed capacity; IAM access control; pay per cluster-hour, partition-hour, and GB stored/transferred.
