# 01 — Data Ingestion & Storage

> Source: [`01_MLA-C01_data_ingestion_storage.pdf`](../udemy_notes/01_MLA-C01_data_ingestion_storage.pdf)

---

## Table of Contents

- [01 — Data Ingestion \& Storage](#01--data-ingestion--storage)
  - [Table of Contents](#table-of-contents)
  - [Types of Data](#types-of-data)
  - [Properties of Data (The 3 V's)](#properties-of-data-the-3-vs)
  - [Data Storage Architectures](#data-storage-architectures)
  - [ETL Pipelines](#etl-pipelines)
  - [Data Formats](#data-formats)
  - [Amazon S3](#amazon-s3)
  - [EC2 Instance Storage](#ec2-instance-storage)
  - [Amazon Kinesis](#amazon-kinesis)
  - [Amazon MSK (Managed Streaming for Apache Kafka)](#amazon-msk-managed-streaming-for-apache-kafka)

---

## Types of Data

| Type | Definition | Characteristics | Examples |
|---|---|---|---|
| **Structured** | Organized in a defined schema, typically in relational databases | Easily queryable, rows & columns, consistent structure | Database tables, CSV files, Excel spreadsheets |
| **Unstructured** | No predefined structure or schema | Not easily queryable without preprocessing, various formats | Text files, videos, images, emails |
| **Semi-Structured** | Some structure via tags, hierarchies, or patterns | More flexible than structured, less chaotic than unstructured | XML/JSON files, email headers, log files |

---

## Properties of Data (The 3 V's)

- **Volume** — the amount or size of data (GB to petabytes). Challenges in storing, processing, and analyzing at scale.
- **Velocity** — the speed at which data is generated and processed. High velocity requires real-time or near-real-time processing (e.g., IoT sensors, high-frequency trading).
- **Variety** — the different types, structures, and sources of data (structured + unstructured + semi-structured from multiple origins).

---

## Data Storage Architectures

<img width="2752" height="1536" alt="Image" src="https://github.com/user-attachments/assets/90365e14-df40-4cf6-a7c5-3eee295d98f5" />
&nbsp;

**Data Warehouse**
- Centralized repository optimized for analysis with structured data
- Schema-on-write (ETL: Extract → Transform → Load)
- Star or snowflake schema; optimized for read-heavy complex queries
- Examples: Amazon Redshift, Google BigQuery, Azure SQL DW

**Data Lake**
- Stores vast amounts of raw data in native format (structured + semi + unstructured)
- Schema-on-read (ELT: Extract → Load → Transform)
- Supports batch, real-time, and stream processing
- Examples: S3 as a data lake, Azure Data Lake Storage, HDFS

**Comparison: Warehouse vs. Lake**

| Dimension | Data Warehouse | Data Lake |
|---|---|---|
| Schema | Schema-on-write (ETL) | Schema-on-read (ELT) |
| Data Types | Primarily structured | Structured + unstructured |
| Agility | Less agile (predefined schema) | More agile (raw data) |
| Cost | Higher (query optimization) | Lower storage, higher processing |
| Use Case | BI & analytics, complex queries | ML, data discovery, flexible future needs |

> Tip: Organizations often use both — raw data in a lake, refined data in a warehouse.

**Data Lakehouse**
- Hybrid combining the flexibility of a data lake with the performance of a data warehouse
- Supports schema-on-write AND schema-on-read
- ACID transactions via technologies like Delta Lake
- Examples: AWS Lake Formation (S3 + Redshift Spectrum), Delta Lake, Databricks, Azure Synapse

**Data Mesh**
- Governance/organizational concept (coined 2019), not a specific technology
- Individual teams own "data products" within a domain ("domain-based data management")
- Federated governance with central standards + self-service infrastructure

---

## ETL Pipelines

| Phase | Description |
|---|---|
| **Extract** | Retrieve raw data from DBs, CRMs, flat files, APIs, streams. Real-time or batch. |
| **Transform** | Cleanse (remove duplicates), enrich, reformat dates/strings, aggregate, handle missing values |
| **Load** | Move transformed data to target warehouse — batch or streaming |

**ETL Orchestration tools on AWS:** AWS Glue · EventBridge · Amazon MWAA · AWS Step Functions · Lambda · Glue Workflows

**Data Sources:** JDBC (platform-independent, language-dependent) · ODBC (platform-dependent, language-independent) · Raw logs · APIs · Streams

---

## Data Formats

| Format | Type | Best For | Systems |
|---|---|---|---|
| **CSV** | Text, tabular | Small–medium datasets, human-readable interchange | Databases, Excel, Pandas, ETL tools |
| **JSON** | Text, key-value | Web APIs, flexible/nested schemas, configs | REST APIs, NoSQL, web browsers |
| **Avro** | Binary + schema | Big data, schema evolution, efficient serialization | Kafka, Spark, Flink, Hadoop |
| **Parquet** | Columnar binary | Analytics on large datasets, column-selective reads | Spark, Hive, Impala, Redshift Spectrum |

---

## Amazon S3

<img width="2752" height="1536" alt="Image" src="https://github.com/user-attachments/assets/a97b471b-28d1-4a23-bf9e-d1ff48480921" />
&nbsp;

**Key concepts**
- Objects stored in globally-unique-named **buckets** (region-level)
- Object key = full path (prefix + object name); no real directories
- Max object size: **5 TB** (multi-part upload required for > 5 GB)
- Objects can have metadata, tags (up to 10), and version IDs

**Security**
- **IAM Policies** — user-based API access control
- **Bucket Policies** — resource-based (JSON), supports cross-account
- **ACLs** — object or bucket level (can be disabled)
- Access granted when: IAM allows OR resource policy allows, AND no explicit DENY

**Versioning**
- Enabled at bucket level; protects against accidental deletes and allows rollback
- Files not versioned before enabling get version `null`
- Suspending versioning does not delete previous versions

**Replication**
- Requires versioning enabled on source and destination
- **CRR** (Cross-Region): compliance, lower latency, cross-account replication
- **SRR** (Same-Region): log aggregation, prod/test sync
- Only new objects are replicated after activation; use S3 Batch Replication for existing objects
- Delete markers optionally replicated; version-ID deletions are NOT replicated (prevents malicious deletes)
- No chaining: bucket1 → bucket2 → bucket3 does NOT auto-replicate bucket1 to bucket3

**Storage Classes**

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

**Lifecycle Rules**
- **Transition actions** — move objects to cheaper class after N days (e.g., Standard → Standard-IA after 60 days)
- **Expiration actions** — delete objects after N days
- Rules can target by prefix or object tags
- Use **S3 Analytics** (updated daily, 24–48 h to start) to get recommendations on Standard → Standard-IA transitions

**Event Notifications**
- Triggers: `ObjectCreated`, `ObjectRemoved`, `ObjectRestore`, `Replication`, etc.
- Destinations: Lambda, SQS, SNS
- Via **EventBridge**: advanced JSON filtering, 18+ destinations (Step Functions, Kinesis, etc.), archive & replay

**Performance**
- Baseline: 3,500 PUT/COPY/POST/DELETE and 5,500 GET/HEAD requests/sec per prefix
- **Multi-Part Upload**: recommended > 100 MB, required > 5 GB; parallelizes uploads
- **Transfer Acceleration**: routes through AWS Edge Locations for faster inter-region transfers
- **Byte-Range Fetches**: parallelize GETs by requesting specific byte ranges; also retrieves partial files

**Encryption**

| Method | Key Owner | Notes |
|---|---|---|
| SSE-S3 | AWS | AES-256; default for new objects; header `x-amz-server-side-encryption: AES256` |
| SSE-KMS | AWS KMS | Audit via CloudTrail; header `aws:kms`; subject to KMS quota limits |
| SSE-C | Customer | HTTPS required; key sent in every HTTP request header |
| Client-Side | Customer | Encrypted before upload; customer manages full key lifecycle |

- Bucket Policies are evaluated **before** default encryption settings
- Force HTTPS via `aws:SecureTransport` bucket policy condition

**Access Points & Object Lambda**
- **Access Points**: per-prefix DNS endpoints with their own IAM-like policies; simplify multi-team bucket access. Can be VPC-restricted.
- **S3 Object Lambda**: attach a Lambda function to transform objects on the fly before they reach the caller (e.g., redact PII, convert XML to JSON)

---

## EC2 Instance Storage

**EBS (Elastic Block Store)**
- Network drive attached to one instance (io1/io2 support multi-attach)
- Bound to a single AZ; migrate across AZ via snapshot
- Elastic Volumes: increase size or change type without restart (can only increase, not decrease)
- Delete on Termination: root volume deleted by default; other volumes are not

**EFS (Elastic File System)**
- Managed NFS mountable on hundreds of EC2 instances across multiple AZs
- Linux only (POSIX), NFSv4.1, encryption at rest with KMS
- Pay-per-use; scales automatically to petabyte scale
- Performance modes: **General Purpose** (low latency) · **Max I/O** (high parallelism, big data)
- Throughput modes: **Bursting** · **Provisioned** · **Elastic** (auto-scales, best for unpredictable workloads)
- Storage tiers: Standard → Infrequent Access (EFS-IA) → Archive (50% cheaper)

**EBS vs EFS**

| | EBS | EFS |
|---|---|---|
| Instances | 1 (except io1/io2 multi-attach) | Hundreds across AZs |
| AZ scope | Single AZ | Multi-AZ |
| OS | Any | Linux only (POSIX) |
| Cost | Lower | Higher (~3× gp2) |

**Amazon FSx**
Amazon FSx provides fully managed file systems optimized for specific workloads. It supports multiple file system types, each with its own protocol and best use cases.

| Variant | Protocol | Best For |
|---|---|---|
| FSx for Windows | SMB, NTFS | Windows workloads, Active Directory, DFS |
| FSx for Lustre | POSIX | HPC, ML, video processing; S3 integration; sub-ms latency |
| FSx for NetApp ONTAP | NFS, SMB, iSCSI | Lift-and-shift ONTAP/NAS workloads; multi-OS |
| FSx for OpenZFS | NFS | ZFS migration; up to 1M IOPS, < 0.5ms latency |

**FSx for Lustre deployment options**
FSx for Lustre is a high-performance file system optimized for workloads that require fast storage and processing. It can be deployed in two main configurations:
- **Scratch**: temporary, high burst (6× faster), no replication — short-term processing
- **Persistent**: replicated within AZ, replace failed files in minutes — long-term sensitive data

---

## Amazon Kinesis
Amazon Kinesis is a platform for real-time data streaming and analytics. It allows you to collect, process, and analyze streaming data at scale.

**Kinesis Data Streams**
- Collect and store real-time streaming data
- Retention up to **365 days**; replay capability
- Data up to **1 MB**; ordering guaranteed per Partition Key
- Encryption: KMS at-rest, HTTPS in-flight
- Capacity modes:
  - **Provisioned**: choose shards manually (1 MB/s in, 2 MB/s out per shard); pay per shard-hour
  - **On-Demand**: auto-scales based on 30-day peak; pay per GB in/out

**Amazon Data Firehose** (formerly Kinesis Data Firehose)
- Fully managed, serverless, near-real-time (buffered delivery)
- Destinations: S3, Redshift, OpenSearch, Splunk, Datadog, MongoDB, custom HTTP
- Supports CSV, JSON, Parquet, Avro, Raw Text, Binary; converts to Parquet/ORC; compresses with gzip/snappy
- Custom transformations via Lambda

**Kinesis Data Streams vs Data Firehose**

| | Kinesis Data Streams | Amazon Data Firehose |
|---|---|---|
| Latency | Real-time | Near real-time (buffered) |
| Management | Manual scaling | Automatic scaling |
| Storage | Up to 365 days | No storage |
| Replay | Yes | No |
| Use case | Custom producers & consumers | Load to S3/Redshift/OpenSearch/3rd party |

**Kinesis Producer Troubleshooting**
- Slow writes → check throttling limits; use better partition key distribution; use KPL + `PutRecords` for batching
- 500/503 errors → error rate > 1%; implement retry with exponential backoff
- Flink connection issues → check VPC config; adjust `RequestTimeout` and `#setQueueLimit`

**Kinesis Consumer Troubleshooting**
- Skipped records → check unhandled exceptions in `processRecords`
- Slow reads → increase shards; increase `maxRecords`; check for slow processing code
- `ReadProvisionedThroughputExceeded` → reshard; use enhanced fan-out; exponential backoff
- High latency → monitor `GetRecords.Latency` and `IteratorAge`; increase shards

**Kinesis Data Analytics / Managed Service for Apache Flink (MSAF)**
- Query streaming data with SQL or custom Flink applications (Java, Python, Scala)
- **Reference tables**: S3-backed JOIN data for cheap lookups (e.g., zip code → city mapping)
- Lambda as destination for post-processing (aggregate, transform, encrypt, route to other services)
- Pay per **Kinesis Processing Unit (KPU)** consumed: 1 KPU = 1 vCPU + 4 GB; serverless
- **RANDOM_CUT_FOREST**: built-in SQL function for anomaly detection on numeric stream columns

---

## Amazon MSK (Managed Streaming for Apache Kafka)

- Fully managed Apache Kafka — creates and manages Kafka brokers and ZooKeeper nodes
- Deployed in VPC, multi-AZ (up to 3); data stored on EBS volumes
- Default message size: **1 MB** (configurable up to 10 MB+)

**MSK vs Kinesis Data Streams**

| | Kinesis Data Streams | Amazon MSK |
|---|---|---|
| Message size | 1 MB | 1 MB default (configurable higher) |
| Scaling unit | Shards (split & merge) | Partitions (add only) |
| Encryption | TLS in-flight, KMS at-rest | PLAINTEXT or TLS in-flight, KMS at-rest |
| Auth | IAM | Mutual TLS + Kafka ACLs · SASL/SCRAM + Kafka ACLs · IAM |

**MSK Security**
- In-flight TLS between brokers and clients; KMS at-rest for EBS
- Auth options: Mutual TLS (AuthN) + Kafka ACLs (AuthZ) · SASL/SCRAM + Kafka ACLs · IAM Access Control

**MSK Monitoring**
- CloudWatch: Basic (cluster/broker) → Enhanced (broker) → Topic-level
- Prometheus: JMX Exporter (metrics) or Node Exporter (CPU/disk)
- Broker log delivery: CloudWatch Logs · S3 · Kinesis Data Streams

**MSK Connect** — managed Kafka Connect workers with auto-scaling; deploy any connector (S3, Redshift, OpenSearch, Debezium…)

**MSK Serverless** — fully managed capacity; IAM access control; pay per cluster-hour, partition-hour, and GB stored/transferred
