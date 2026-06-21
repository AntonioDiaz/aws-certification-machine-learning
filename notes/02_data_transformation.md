# 02 — Data Transformation, Integrity & Feature Engineering <!-- omit in toc -->

> Source: [`02_MLA-C01_intro_data_transformation.pdf`](../udemy_notes/02_MLA-C01_intro_data_transformation.pdf)

---

## Table of Contents <!-- omit in toc -->

- [Why This Topic Matters](#why-this-topic-matters)
- [Amazon EMR](#amazon-emr)
- [Apache Spark on EMR](#apache-spark-on-emr)
- [Feature Engineering](#feature-engineering)
- [Handling Missing Data](#handling-missing-data)
- [Handling Unbalanced Data](#handling-unbalanced-data)
- [Handling Outliers](#handling-outliers)
- [Common Transformations](#common-transformations)
- [Amazon SageMaker — Data Preparation](#amazon-sagemaker--data-preparation)
  - [SageMaker Overview](#sagemaker-overview)
  - [SageMaker Ground Truth](#sagemaker-ground-truth)
  - [SageMaker Data Wrangler](#sagemaker-data-wrangler)
  - [SageMaker Model Monitor](#sagemaker-model-monitor)
  - [SageMaker Clarify](#sagemaker-clarify)
  - [SageMaker Feature Store](#sagemaker-feature-store)
  - [SageMaker Canvas](#sagemaker-canvas)
- [AWS Glue](#aws-glue)
- [Amazon Athena](#amazon-athena)

---

## Why This Topic Matters

**In machine learning**, data transformation and feature engineering are often the most impactful work you can do. Raw data is almost never model-ready: it contains missing values, outliers, imbalanced classes, and features at incompatible scales. The quality of your features determines the ceiling of your model's performance — no algorithm can compensate for poorly prepared data. As Andrew Ng famously noted, applied ML is basically feature engineering.

**On the MLA-C01 exam**, this domain carries significant weight. Expect scenario-based questions that test your ability to:
- Choose the right AWS service for a data preparation task (EMR vs. Glue vs. Data Wrangler vs. Athena)
- Select the appropriate imputation or balancing technique given constraints
- Identify which SageMaker tool detects bias, monitors drift, or explains predictions
- Recognize when transformations like normalization, log scaling, or one-hot encoding are required

> [!TIP]
> The exam frequently presents distractors that blur the lines between similar services (e.g., Glue DataBrew vs. SageMaker Data Wrangler, or Ground Truth vs. A2I). Focus on each service's unique strengths and anti-patterns.

---

## Amazon EMR

**What is EMR?**
Amazon EMR (Elastic MapReduce) is a managed cluster platform that simplifies running big data frameworks, such as Apache Spark, Hive, and Presto, to process and __analyze vast amounts of data__.

<img width="2752" height="1536" alt="Image" src="https://github.com/user-attachments/assets/59eca9b4-a52f-41f4-893c-ea89c6f0df63" />

- Elastic MapReduce — managed Hadoop framework on EC2 instances
- Includes Spark, HBase, Presto, Flink, Hive & more
- EMR Notebooks for interactive development
- Deep AWS integration (EC2, VPC, S3, CloudWatch, IAM, CloudTrail, Data Pipeline)

Key points for the exam:
* __Cluster Structure__: It consists of a Master node (manages the cluster), Core nodes (run tasks and store data), and Task nodes (run tasks only; ideal for cost-saving with Spot Instances)
* __EMR Serverless__: A deployment option that allows you to run applications without configuring or managing clusters, automatically scaling resources as needed
* __ML Role__: It is primarily used in the Data Preparation phase for massive-scale feature engineering and distributed data processing. It also supports Spark MLLib for distributed machine learning.
* __Storage__: It utilizes EMR_FS to access data in Amazon S3 as if it were a local HDFS file system.


**Cluster Node Types**

| Node | Role | Notes |
|---|---|---|
| **Master** | Manages the cluster | Single EC2 instance; m4.large (< 50 nodes), m4.xlarge (> 50 nodes) |
| **Core** | Hosts HDFS data and runs tasks | Can scale up/down with some risk of data loss |
| **Task** | Runs tasks only, no data hosting | Safe to remove; ideal for Spot instances |

**EMR Usage Patterns**

| Pattern | Description | Best For |
|---|---|---|
| **Transient cluster** | Spin up → run job → terminate automatically | Batch ETL, periodic ML training jobs; minimizes idle cost |
| **Long-running cluster** | Cluster stays alive between jobs | Interactive analytics, frequent ad-hoc queries; use Reserved Instances to offset cost |
| **EMR Serverless** | No cluster to manage; AWS auto-scales workers per job | Infrequent/unpredictable workloads where cluster management is unwanted overhead |

**How jobs are submitted:**
- SSH into the master node and run commands directly (development/debugging)
- Add ordered **Steps** in the console or via CLI — each step is a unit of work (e.g., a Spark submit command); steps execute sequentially and the cluster can auto-terminate when all steps complete
- Use **EMR Studio** / notebooks for interactive development without SSH

**Cost optimization tips:**
- Use **Spot Instances** on task nodes — safe to interrupt since task nodes hold no HDFS data
- Pair Spot task nodes with On-Demand core/master nodes for resilience
- For long-running clusters, commit to **Reserved Instances** (1- or 3-year) on core/master nodes
- Transient clusters with auto-termination eliminate idle charges entirely

> [!TIP]
> On the exam, "ETL job runs nightly" → transient cluster. "Data science team needs always-on notebooks" → long-running cluster with Reserved Instances. "No ops team to manage clusters" → EMR Serverless.

> [!WARNING]
> EMR Serverless application lifecycle is NOT fully automatic. You must call `CreateApplication`, `StartApplication`, `StopApplication`, and **`DeleteApplication`** explicitly to avoid excess charges.

**EMR Serverless: Pre-Initialized Capacity**
- Spark adds **10% overhead** to requested driver/executor memory
- Set initial capacity at least 10% above your job's actual requirements

**EMR Storage Options**

| Storage | Description |
|---|---|
| HDFS | Native Hadoop distributed file system |
| EMRFS | Access S3 as if it were HDFS; uses DynamoDB for consistency tracking |
| Local file system | Ephemeral, node-local |
| EBS for HDFS | Persistent EBS-backed HDFS |

**EMR on EKS**
- Submit Spark jobs on EKS without provisioning clusters
- Share resources between Spark and other Kubernetes workloads

**EMR Security**
- IAM policies, Kerberos, SSH, IAM roles
- Lake Formation integration; native Apache Ranger support (Hadoop/Hive data security)
- EMRFS: SSE or CSE at rest; TLS in transit between EMR nodes and S3
- Spark: encrypted driver ↔ executor communication; Hive ↔ Glue Metastore uses TLS

**Choosing Instance Types**

| Use Case | Instance Type |
|---|---|
| General / default | m4.large |
| Waiting on external I/O | t2.medium |
| Better performance | m4.xlarge |
| Computation-intensive | High CPU instances |
| Database / memory cache | High memory instances |
| NLP / ML (network + CPU) | Cluster compute instances |
| Accelerated AI/ML | GPU instances (g3, g4, p2, p3) |

> Spot instances are ideal for **task nodes**. Avoid on core/master unless testing or very cost-sensitive (risk of partial data loss).

---

## Apache Spark on EMR

**Hadoop stack:** HDFS + YARN + MapReduce → Spark replaces MapReduce for in-memory processing.

**Spark Architecture**
- **Driver Program** (with Spark Context) coordinates the job
- **Cluster Manager** (Spark or YARN) allocates resources
- **Executors** run tasks and cache data in memory across worker nodes

**Spark Components**

| Component | Purpose |
|---|---|
| Spark Core | Base engine |
| Spark SQL | SQL queries over structured data |
| Spark Streaming | Real-time stream processing |
| MLLib | Built-in ML algorithms |
| GraphX | Graph computation |

**Spark MLLib algorithms:** logistic regression, naïve Bayes, regression, decision trees, ALS (recommendation), K-Means clustering, LDA (topic modeling), SVD, PCA, ML pipelines

**Spark Structured Streaming**
- Read from S3, Kinesis, Kafka; write to JDBC, S3, etc.
- Kinesis integration via Kinesis Client Library (KCL)-backed Spark Dataset

**Interactive Tools**
- **Zeppelin**: run Spark interactively; execute SparkSQL; visualize results as charts
- **EMR Notebook**: like Zeppelin with more AWS integration; backed up to S3; VPC-hosted; console access only

---

## Feature Engineering

> "Applied machine learning is basically feature engineering." — Andrew Ng

Feature engineering applies domain knowledge to create better features for model training:
- Which features to use?
- How to transform features?
- How to handle missing data?
- Should new features be derived from existing ones?

**Curse of Dimensionality**
- Too many features → sparse data → poor model performance
- Solution: select the most relevant features (domain knowledge) or apply dimensionality reduction
- Techniques: **PCA**, **K-Means**

**TF-IDF (Term Frequency – Inverse Document Frequency)**

Measures how relevant a word is to a specific document relative to a corpus. A high score means the word appears often in this document but rarely across others — i.e., it is distinctive.

**Components:**

| Component | Formula | Intuition |
|---|---|---|
| **TF** (Term Frequency) | `count(term in doc) / total terms in doc` | How often the word appears in this document |
| **IDF** (Inverse Doc Frequency) | `log(total docs / docs containing term)` | How rare the word is across the whole corpus |
| **TF-IDF** | `TF × IDF` | High only when the word is both frequent here AND rare elsewhere |

**Why log for IDF?**
Word frequencies follow a power-law distribution (a few words appear exponentially more than others). Taking `log` compresses this range so common words like "the" don't drown out meaningful rare words.

**Bag-of-words assumption:**
TF-IDF treats a document as an unordered set of words — word position and grammar are ignored. This makes it fast and simple but loses semantic context (contrast with word embeddings/transformers).

**Hashing trick:**
Instead of building a full vocabulary index, words can be hashed to a fixed-size vector. Reduces memory and speeds up computation at the cost of potential (rare) hash collisions.

**N-grams:**
Extend TF-IDF beyond single words to capture multi-word phrases:
- Bigrams: "machine learning", "New York"
- Trigrams: "support vector machine"
- Improves relevance for compound concepts; vocabulary size grows fast

**Practical use cases:**
- Search engine ranking and document retrieval
- Text classification features (spam detection, topic classification)
- Information retrieval preprocessing before passing to ML models

**Limitations:**
- Ignores word order and semantics ("dog bites man" = "man bites dog")
- Struggles with synonyms (two documents about "car" vs. "automobile" score low similarity)
- For richer representations, prefer **word embeddings** (Word2Vec, GloVe) or **transformer models** (BERT)

> [!TIP]
> On the exam, TF-IDF is the go-to answer for "how do you turn text into numerical features for a classical ML model." Spark MLLib has built-in `HashingTF` and `IDF` transformers that make this practical at scale.

---

## Handling Missing Data

Missing data is one of the most common real-world data quality problems. How you handle it affects both model accuracy and the validity of your conclusions — a poor imputation strategy can silently introduce bias.

**Why data goes missing:**
- **MCAR** (Missing Completely At Random): absence is unrelated to any value — e.g., a sensor randomly drops packets
- **MAR** (Missing At Random): absence correlates with other observed variables — e.g., younger users skip an optional age field
- **MNAR** (Missing Not At Random): absence correlates with the missing value itself — e.g., high earners skip the income field; hardest to handle without domain knowledge

Understanding *why* data is missing guides which technique to apply.

| Technique | Description | When to Use |
|---|---|---|
| **Mean/Median Replacement** | Replace with column mean (median if outliers present) | Fast, simple; not for categorical data |
| **Dropping rows** | Remove rows with missing values | Only if few rows missing and no bias introduced; never the "best" answer |
| **KNN Imputation** | Average values of K nearest neighbors | Numerical data; can handle categorical via Hamming distance |
| **Deep Learning** | Train a model to predict missing values | Works well for categorical data |
| **Regression** | Fit linear/non-linear relationships between features | Structured numerical data |
| **MICE** | Multiple Imputation by Chained Equations | Most advanced; handles complex dependencies |
| **Get more data** | Collect real data instead of imputing | Always the best option when feasible |

> [!TIP]
> Median is preferred over mean when outliers are present. For categorical columns, imputing with the most frequent value is a common fallback.

---

## Handling Unbalanced Data

In a balanced dataset, classes are roughly equally represented. In an **unbalanced** (or imbalanced) dataset, one class — typically the event of interest — is far rarer than the other. This is the norm in many high-value ML problems.

**Why it matters:** 
A model trained on heavily skewed data can achieve high accuracy by simply predicting the majority class every time — yet completely fail at the task it was built for. Accuracy is a misleading metric here; prefer **precision, recall, F1, or AUC-ROC**.

**Common examples:**
- Fraud detection: ~0.1% of transactions are fraudulent
- Medical diagnosis: rare disease prevalence may be < 1%
- Anomaly detection in logs or sensor data

Unbalanced data: large discrepancy between positive (target) and negative cases — common in fraud detection, anomaly detection, etc. The table below compares the main techniques for correcting this imbalance, each with a different trade-off between data fidelity and model bias risk.

| Technique | Description | Notes |
|---|---|---|
| **Oversampling** | Duplicate samples from minority class | Simple, can cause overfitting |
| **Undersampling** | Remove samples from majority class | Loses data; avoid unless scaling is a concern |
| **SMOTE** | Synthetic Minority Over-sampling Technique | Generates new minority samples via KNN; also undersamples majority; generally best option |
| **Adjust threshold** | Raise classification probability threshold | Reduces false positives but increases false negatives |

**SMOTE process:**
1. For each minority sample, run K-nearest-neighbors
2. Create new synthetic sample from the mean of the KNN result

---

## Handling Outliers

Outliers are data points that differ significantly from the rest of the dataset. They can arise from measurement errors, data entry mistakes, or genuinely rare but real events. The key question is always: **does this outlier represent noise or signal?**

**Why outliers matter:**
- They can skew statistical measures (mean, variance) and distort model training
- Linear models and distance-based algorithms (KNN, K-Means) are especially sensitive
- Tree-based models (Random Forest, XGBoost) are more robust to outliers by nature

**Detection approaches:**
- **Standard deviation**: flag points beyond N×σ from the mean (common: 2σ or 3σ)
- **IQR (Interquartile Range)**: flag points below Q1 − 1.5×IQR or above Q3 + 1.5×IQR; more robust to skewed distributions than σ
- **Visual inspection**: box plots, scatter plots, histograms
- **Algorithmic**: AWS Random Cut Forest, Isolation Forest, Local Outlier Factor

**Standard Deviation approach:**
- Variance (σ²) = average of squared differences from the mean
- Standard Deviation (σ) = √variance
- Data points > 1σ from mean are "unusual"; use common sense to pick the multiple

**When to remove outliers:**
- Collaborative filtering: a single user rating thousands of movies may distort recommendations
- Web logs: outliers may represent bots

**When NOT to remove outliers:**
- When they represent real, meaningful data (e.g., billionaires in income surveys)

> [!IMPORTANT]
> AWS **Random Cut Forest** (available in QuickSight, Kinesis Analytics, SageMaker) is designed specifically for outlier detection at scale.

---

## Common Transformations

Raw features are rarely in the right form for a model to consume directly. Common transformations reshape, rescale, or re-encode features so that algorithms can learn effectively. Choosing the wrong transformation — or skipping one — is a frequent source of poor model performance.

| Transformation | Description | Use Case |
|---|---|---|
| **Binning** | Group values into ranges (buckets) | Reduce precision noise; ordinal encoding of continuous data |
| **Quantile binning** | Bin by distribution percentiles | Ensures even bin sizes |
| **Log transform** | Apply log to features with exponential trends | Income, word frequencies |
| **Polynomial features** | Add x², √x alongside x | Allows learning super/sub-linear functions (e.g., YouTube recommendations) |
| **One-hot encoding** | Binary column per category value | Categorical → numerical for neural nets |
| **Scaling / Normalization** | Normalize features to comparable ranges | Required by most models; use MinMaxScaler or StandardScaler |
| **Shuffling** | Randomize order of training samples | Prevents learning order-based residual signals |

---

## Amazon SageMaker — Data Preparation

SageMaker is AWS's fully managed ML platform, covering the entire lifecycle from data preparation through training to deployment. For the exam, the data preparation tooling is its own ecosystem — several distinct services that are easy to confuse because they overlap in capability. Understanding *which tool to reach for given a constraint* (no-code, scale, bias detection, labeling) is the core exam skill here.

<img width="2752" height="1536" alt="Image" src="https://github.com/user-attachments/assets/4213ae82-692f-4f10-b4b1-b3fc5045c364" />
&nbsp;

**SageMaker data prep tools at a glance:**

| Tool | Who uses it | Primary purpose |
|---|---|---|
| **Data Wrangler** | Data scientists | Visual, no-code data transformation in Studio |
| **Processing** | Engineers | Run custom containers for large-scale data prep |
| **Ground Truth** | ML teams | Human labeling workflows with active learning |
| **Feature Store** | ML platform teams | Centralized, reusable feature repository |
| **Clarify** | Responsible AI teams | Bias detection and explainability |
| **Model Monitor** | MLOps teams | Ongoing drift detection on deployed models |
| **Canvas** | Business analysts | No-code end-to-end ML (upload CSV → predictions) |

### SageMaker Overview

SageMaker handles the full ML workflow: data prep → training → deployment.

- **Notebook Instances** (EC2-backed): S3 access, Scikit-learn, Spark, TensorFlow; spin up training jobs and deploy models
- **Data sources**: S3 (primary), Athena, EMR, Redshift, Amazon Keyspaces
- **Preferred format**: RecordIO / Protobuf (varies by algorithm)
- **SageMaker Processing**: copy data from S3, run a processing container, output results back to S3

**SageMaker AI Domains**
- Organize users, apps, and resources; share an EFS volume
- User profiles → personal apps (Studio instances) with private EFS directory
- Shared spaces → communal IDE with shared EFS directory

**VPC configuration**
- Default: two VPCs (one SageMaker-managed for internet, one user-owned for EFS)
- "VPC Only" mode: all traffic routed through your own VPC

**Training on SageMaker**
- Create a training job with: S3 data URL, compute resources, output S3 URL, ECR training image
- Training options: built-in algorithms · Spark MLLib · TensorFlow/MXNet/PyTorch/XGBoost · Hugging Face · custom Docker image · AWS Marketplace algorithms

**Deploying Trained Models**
- **Persistent endpoint**: real-time individual predictions
- **Batch Transform**: predictions over entire datasets
- **Inference Pipelines**: complex multi-step processing
- **SageMaker Neo**: deploy to edge devices
- **Elastic Inference**: accelerate deep learning models
- **Automatic scaling**: increase endpoints under load
- **Shadow Testing**: evaluate new model vs. live model to catch regressions

---

### SageMaker Ground Truth

- Manages human labelers to generate training labels
- Builds its own model as images are labeled; routes only ambiguous samples to humans
- Can **reduce labeling cost by up to 70%**
- Labeler sources: Amazon Mechanical Turk · internal team · professional labeling companies
- **Ground Truth Plus**: fully managed turnkey solution; AWS experts manage labelers
- Alternatives for auto-labeling: Rekognition (images), Comprehend (text/sentiment)

**Amazon Mechanical Turk**
- Crowdsourcing marketplace for human labeling tasks
- Distributed virtual workforce; you set reward per item
- Integrates with SageMaker Ground Truth and Amazon A2I

---

### SageMaker Data Wrangler

Visual interface (in SageMaker Studio) for data preparation — no code required for most tasks.

**Capabilities:**
- Import, visualize, and transform data (300+ built-in transformations)
- Custom transforms with Pandas, PySpark, PySpark SQL
- "Quick Model" to rapidly validate data quality before full training
- Image data transformations (resize, enhance, corrupt)
- Balance data (oversampling, undersampling, SMOTE)
- Impute missing data, handle outliers, dimensionality reduction (PCA)

**Data sources:** S3 · Athena · Redshift · Lake Formation · SageMaker Feature Store · JDBC (Databricks, SaaS)

**Export options:** SageMaker Processing · Pipelines · Feature Store · Notebook

**Troubleshooting:**
- Ensure Studio user has `AmazonSageMakerFullAccess` IAM policy
- Check data source permissions
- For "instance type not available" errors → request quota increase via Service Quotas

---

### SageMaker Model Monitor

- Monitors deployed models for quality deviations; alerts via CloudWatch
- No code required

**Monitoring types:**

| Type | Description |
|---|---|
| Data quality drift | Statistical properties of input features vs. a baseline |
| Model quality drift | Accuracy / performance metrics vs. a baseline; integrates with Ground Truth labels |
| Bias drift | Detects emerging bias in predictions (via Clarify) |
| Feature attribution drift | Uses NDCG score to compare feature importance ranking between training and live data |

**Integration:** Tensorboard · QuickSight · Tableau · SageMaker Studio

---

### SageMaker Clarify

- Detects **pre-training bias** and explains model behavior
- Integrated with Model Monitor for ongoing bias monitoring via CloudWatch

**Pre-training bias metrics:**

| Metric | What it measures |
|---|---|
| Class Imbalance (CI) | One demographic group has fewer training samples |
| DPL | Imbalance of positive outcomes between groups |
| KL / JS Divergence | How much outcome distributions diverge between groups |
| Lp-norm (LP) | P-norm difference between outcome distributions |
| TVD | L1-norm difference between outcome distributions |
| Kolmogorov-Smirnov (KS) | Maximum divergence between group outcome distributions |
| CDD | Disparity of outcomes between groups as a whole and by subgroups |

**Explainability:**
- **Shapley Values (SHAP)**: measures each feature's contribution to predictions by simulating feature removal; originated in game theory
- **Asymmetric Shapley Values**: for time series — contribution of features at each time step
- **Partial Dependence Plots (PDPs)**: visualize how feature values influence predictions

---

### SageMaker Feature Store

Centralized repository for ML features — fast, secure access for training and inference.

**Organization:**
- **Feature Group**: collection of features (record identifier + feature name + event time)
- **Online Store**: low-latency access via `PutRecord` / `GetRecord` APIs (streaming)
- **Offline Store** (S3): batch access; automatically creates a Glue Data Catalog

**Ingestion:** Kinesis · MSK · Spark · Data Wrangler · batch jobs

**Security:** encrypted at rest and in transit · KMS customer master keys · IAM fine-grained access · AWS PrivateLink

---

### SageMaker Canvas

No-code ML for business analysts:
- Upload CSV, select prediction column, build model, make predictions
- Supports classification and regression
- Automatic data cleaning: missing values, outliers, duplicates
- Can join datasets; share models with SageMaker Studio
- Generative AI support via Bedrock or JumpStart foundation models

---

## AWS Glue

Serverless ETL and data catalog service.

**Glue Crawler / Data Catalog**
- Crawls S3 (and RDS, Redshift, DynamoDB, most SQL DBs) to infer schema
- Runs on a schedule or on demand
- Populates the **Glue Data Catalog** (stores only metadata; original data stays in S3)
- Once cataloged, unstructured data can be queried via Redshift Spectrum, Athena, EMR, QuickSight

> [!IMPORTANT]
> Plan your S3 partition structure **before** crawling. Glue extracts partitions based on your S3 folder hierarchy. Query pattern determines optimal layout: query by time → `yyyy/mm/dd/device`; query by device → `device/yyyy/mm/dd`.

**Glue Studio**
- Visual ETL workflow builder
- Create DAGs for complex pipelines
- Sources: S3, Kinesis, Kafka, JDBC
- Targets: S3, Glue Data Catalog
- Supports partitioning; visual job dashboard with status and run times

**Glue Data Quality**
- Define quality rules manually or use automatic recommendations
- Uses **Data Quality Definition Language (DQDL)**
- Results can fail the job or report to CloudWatch

**AWS Glue DataBrew**
- Visual data preparation tool; 250+ ready-made transformations
- Input: S3, data warehouse, database → Output: S3
- "Recipes" of transformations saved as reusable jobs
- Can create datasets with custom SQL from Redshift and Snowflake
- Security: KMS (customer master keys only) · SSL in transit · IAM · CloudWatch & CloudTrail

**DataBrew PII Handling:**

| Technique | DataBrew Operation |
|---|---|
| Substitution | `REPLACE_WITH_RANDOM…` |
| Shuffling | `SHUFFLE_ROWS` |
| Deterministic encryption | `DETERMINISTIC_ENCRYPT` |
| Probabilistic encryption | `ENCRYPT` |
| Decryption | `DECRYPT` |
| Nulling / deletion | `DELETE` |
| Masking | `MASK_CUSTOM`, `MASK_DATE`, `MASK_DELIMITER`, `MASK_RANGE` |
| Hashing | `CRYPTOGRAPHIC_HASH` |

---

## Amazon Athena

Serverless interactive SQL query service for S3 data — powered by Presto under the hood.

**Key facts:**
- No data loading required — data stays in S3
- Supports: CSV, TSV, JSON, ORC (columnar, splittable), Parquet (columnar, splittable), Avro (splittable)
- Compression: Snappy, Zlib, LZO, Gzip

**Common use cases:** ad-hoc web log queries · staging data validation before Redshift load · analyze CloudTrail/CloudFront/VPC/ELB logs · Jupyter/Zeppelin/RStudio notebooks · QuickSight integration

**Cost model:**
- $5 per TB scanned
- Successful and cancelled queries count; failed queries do not
- DDL (`CREATE`/`ALTER`/`DROP`) is free
- Use columnar formats (ORC, Parquet) to save 30–90% on query costs

**Athena Workgroups**
- Organize users/teams/workloads into isolated groups
- Per-workgroup: query history · data scan limits · IAM policies · encryption settings
- Integrates with IAM, CloudWatch, SNS

**Security:**
- IAM, ACLs, S3 bucket policies
- Encrypt results at rest: SSE-S3, SSE-KMS, CSE-KMS
- TLS in transit (Athena ↔ S3)
- Cross-account S3 access via bucket policy

**Anti-patterns:**
- Formatted reports/visualization → use **QuickSight** instead
- ETL pipelines → use **Glue** instead

**Performance optimization:**
- Use columnar formats (ORC, Parquet)
- Fewer large files > many small files
- Use partitions; run `MSCK REPAIR TABLE` to register new partitions added after the fact

**ACID transactions (Apache Iceberg)**
- Add `'table_type' = 'ICEBERG'` in `CREATE TABLE`
- Supports concurrent row-level modifications and time travel queries
- Compatible with EMR, Spark, and anything supporting Iceberg format
- Run `OPTIMIZE table REWRITE DATA USING BIN_PACK` periodically to maintain performance

**Fine-Grained Access to Glue Data Catalog:**
- IAM-based database and table-level security
- Map Athena operations (DROP TABLE, CREATE TABLE, etc.) to their IAM `glue:*` actions
- At minimum: policy granting access to the target database and Glue Data Catalog in each region

**CREATE TABLE AS SELECT (CTAS)**
- Creates a new table from query results
- Can convert data to a different format (e.g., convert CSV stored in S3 to Parquet or ORC)
```sql
CREATE TABLE new_table
WITH (format = 'Parquet', write_compression = 'SNAPPY')
AS SELECT * FROM old_table;
```
