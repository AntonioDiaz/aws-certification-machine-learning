# AWS Machine Learning Specialty — Study Notes

![AWS Certification](https://img.shields.io/badge/AWS-MLA--C01-FF9900?style=flat-square&logo=amazonaws&logoColor=white)
![Status](https://img.shields.io/badge/Status-In%20Progress-yellow?style=flat-square)
![License](https://img.shields.io/badge/License-Personal%20Use-blue?style=flat-square)

Personal study notes for the **AWS Certified Machine Learning Engineer – Associate (MLA-C01)** exam. Notes are compiled from the Udemy course and organized by topic domain.

---

## Resources

- [AWS Certified Machine Learning Engineer – Associate (MLA-C01)](https://aws.amazon.com/es/certification/certified-machine-learning-engineer-associate/) — official exam page (exam guide, sample questions, scheduling)
- [AWS Skill Builder](https://skillbuilder.aws/) — official learning platform with practice exams and courses
- [SageMaker Documentation](https://docs.aws.amazon.com/sagemaker/)
- [Amazon Bedrock Documentation](https://docs.aws.amazon.com/bedrock/)

---

## Table of Contents

- [Resources](#resources)
- [Notes](#notes)
    - [00 — Introduction \& Exam Overview](#00--introduction--exam-overview)
      - [Target Candidate](#target-candidate)
      - [Domains Covered](#domains-covered)
    - [01 — Data Ingestion \& Storage](#01--data-ingestion--storage)
      - [Types of Data](#types-of-data)
      - [Properties of Data (The 3 V's)](#properties-of-data-the-3-vs)
      - [Data Storage Architectures](#data-storage-architectures)
      - [ETL Pipelines](#etl-pipelines)
      - [Data Formats](#data-formats)
      - [Amazon S3](#amazon-s3)
      - [EC2 Instance Storage](#ec2-instance-storage)
      - [Amazon Kinesis](#amazon-kinesis)
      - [Amazon MSK (Managed Streaming for Apache Kafka)](#amazon-msk-managed-streaming-for-apache-kafka)
    - [02 — Data Transformation](#02--data-transformation)
    - [03 — AWS Managed ML Services](#03--aws-managed-ml-services)
    - [04 — Amazon SageMaker](#04--amazon-sagemaker)
    - [05 — Model Training, Tuning \& Evaluation](#05--model-training-tuning--evaluation)
    - [06 — Generative AI Fundamentals](#06--generative-ai-fundamentals)
    - [07 — Amazon Bedrock](#07--amazon-bedrock)
    - [08 — ML Operations (MLOps)](#08--ml-operations-mlops)
    - [09 — Security](#09--security)
    - [10 — Management \& Governance](#10--management--governance)
    - [11 — ML Best Practices](#11--ml-best-practices)
    - [12 — Exam Preparation](#12--exam-preparation)
  - [How to Use These Notes](#how-to-use-these-notes)
  - [Exam Details](#exam-details)

---

## Notes

All notes are stored as PDFs inside the [`udemy_notes/`](udemy_notes/) folder.

### 00 — Introduction & Exam Overview
[`00_MLA-C01_intro.pdf`](udemy_notes/00_MLA-C01_intro.pdf)

#### Target Candidate
The exam is aimed at engineers with:
- ~1 year of hands-on experience with SageMaker and other ML engineering services
- ~1 year of background in software development, DevOps, data engineering, or data science
- Working knowledge of ML algorithms, data engineering fundamentals, and CI/CD practices

> **Not required:** designing end-to-end ML solutions from scratch, or depth in NLP / Computer Vision.

#### Domains Covered

| # | Domain | Key AWS Services & Topics |
|---|---|---|
| 1 | **Data Ingestion & Storage** | Types & formats of data, data warehouses/lakes/lakehouses, ETL pipelines & orchestration — EBS, EFS, FSx, S3, Kinesis |
| 2 | **Data Transformation, Integrity & Feature Engineering** | Missing/unbalanced/outlier data, common transformations, SageMaker data processing — EMR, SageMaker, Glue |
| 3 | **AWS Managed AI Services** | Personalize, Polly, Rekognition, Comprehend, Forecast, Lex, Textract, Transcribe, Translate, Fraud Detector, Kendra, Lookout for Equipment, Amazon Q |
| 4 | **SageMaker Built-In Algorithms** | Pre-built algorithms available natively in SageMaker |
| 5 | **Model Training, Tuning & Evaluation** | Deep learning fundamentals, tuning techniques, model performance metrics, Automatic Model Tuning (AMT) |
| 6 | **Generative AI Fundamentals** | Transformer architecture, self-attention, how GPT works, SageMaker JumpStart |
| 7 | **Building Gen AI Apps with Bedrock** | Foundation models, RAG, Knowledge Bases, Vector Stores, Guardrails, LLM Agents |
| 8 | **ML Operations (MLOps)** | SageMaker in depth, ECS, ECR, CloudFormation, CDK, CodeDeploy, CodeBuild, CodePipeline, EventBridge, Step Functions, MWAA |
| 9 | **Security, Identity & Compliance** | Securing SageMaker, IAM, KMS, Macie, Secrets Manager, WAF, Shield, VPC, PrivateLink |
| 10 | **Management & Governance** | CloudWatch, CloudFormation, Config, CloudTrail, X-Ray, Trusted Advisor, Budgets, Cost Explorer |
| 11 | **ML Best Practices** | AWS Well-Architected Machine Learning Lens |

### 01 — Data Ingestion & Storage

[Full notes →](notes/01_data_ingestion_storage.md)

Topics: data types, 3 V's, warehouse vs lake vs lakehouse, ETL pipelines, data formats (CSV/JSON/Avro/Parquet), S3, EBS, EFS, FSx, Kinesis, Amazon MSK.

### 02 — Data Transformation
[`02_MLA-C01_intro_data_transformation.pdf`](udemy_notes/02_MLA-C01_intro_data_transformation.pdf)
Feature engineering, ETL pipelines, AWS Glue DataBrew, and data preprocessing techniques.

### 03 — AWS Managed ML Services
[`03_MLA-C01_aws_manage_services.pdf`](udemy_notes/03_MLA-C01_aws_manage_services.pdf)
Rekognition, Comprehend, Transcribe, Translate, Forecast, Personalize, and other AI/ML managed services.

### 04 — Amazon SageMaker
[`04_MLA-C01_sagemaker.pdf`](udemy_notes/04_MLA-C01_sagemaker.pdf)
SageMaker Studio, training jobs, built-in algorithms, pipelines, endpoints, and inference patterns.

### 05 — Model Training, Tuning & Evaluation
[`05_MLA-C01_model_training_tunning_evaluation.pdf`](udemy_notes/05_MLA-C01_model_training_tunning_evaluation.pdf)
Hyperparameter tuning, Automatic Model Tuning (AMT), evaluation metrics, bias detection, and Clarify.

### 06 — Generative AI Fundamentals
[`06_MLA-C01_gen_ai_fundamentals.pdf`](udemy_notes/06_MLA-C01_gen_ai_fundamentals.pdf)
Foundation models, prompt engineering, RAG, fine-tuning concepts, and responsible AI.

### 07 — Amazon Bedrock
[`07_MLA-C01_bedrock.pdf`](udemy_notes/07_MLA-C01_bedrock.pdf)
Bedrock APIs, model providers, Knowledge Bases, Agents, and Guardrails.

### 08 — ML Operations (MLOps)
[`08_MLA-C01_ml_operations.pdf`](udemy_notes/08_MLA-C01_ml_operations.pdf)
CI/CD for ML, SageMaker Pipelines, Model Registry, monitoring, and A/B testing.

### 09 — Security
[`09_MLA-C01_security.pdf`](udemy_notes/09_MLA-C01_security.pdf)
IAM, VPC, KMS encryption, data privacy, network isolation, and compliance for ML workloads.

### 10 — Management & Governance
[`10_MLA-C01_management_governance.pdf`](udemy_notes/10_MLA-C01_management_governance.pdf)
CloudWatch, CloudTrail, SageMaker Model Monitor, cost optimization, and tagging strategies.

### 11 — ML Best Practices
[`11_MLA-C01_ml_best_practice_.pdf`](udemy_notes/11_MLA-C01_ml_best_practice_.pdf)
Well-Architected Framework for ML, scalability patterns, and production readiness.

### 12 — Exam Preparation
[`12_MLA-C01_exam_preparation.pdf`](udemy_notes/12_MLA-C01_exam_preparation.pdf)
Practice questions, key service comparisons, common traps, and final review tips.

---

## How to Use These Notes

1. **Follow the order** — topics build on each other. Start with `00` (intro) and progress sequentially.
2. **Cross-reference the AWS docs** — use these notes as a quick reference; dive into official documentation for deeper understanding.
3. **Focus on SageMaker and Bedrock** — they represent the largest portion of exam questions.
4. **Revisit exam prep last** — `12_exam_preparation.pdf` consolidates the most exam-critical concepts.

---

## Exam Details

| Detail | Info |
|---|---|
| Exam code | MLA-C01 |
| Level | Associate |
| Duration | 130 minutes |
| Questions | ~65 questions |
| Passing score | 720 / 1000 |
| Format | Multiple choice & multiple response |
