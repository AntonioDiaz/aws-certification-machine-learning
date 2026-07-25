# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

Personal study notes for the **AWS Certified Machine Learning Engineer – Associate (MLA-C01)** exam. It is a documentation-only repository — no code, no build system, no tests.

## Repository Layout

```
notes/          — Markdown study notes (the primary deliverable; currently topics 01–04)
udemy_notes/    — Source PDFs from the Udemy course (read-only reference material)
README.md       — Index with exam overview, domain breakdown, and links to each notes file
TODO.md         — Living backlog: topics left to convert, formatting debt, fixes
```

Each note file maps 1-to-1 to a PDF in `udemy_notes/` and expands on it with structured Markdown.

## Notes Authoring Conventions

- **Naming**: `notes/NN_<slug>.md` matching the corresponding `udemy_notes/NN_MLA-C01_<slug>.pdf`
- **Structure per file**: YAML-omitted TOC → `## Why This Topic Matters` intro → topic sections → service-specific deep dives
- **Tables** are preferred for comparisons (service A vs B, technique trade-offs, exam heuristics)
- **Callout blocks** used consistently:
  - `> [!TIP]` — exam shortcuts and decision rules
  - `> [!IMPORTANT]` — high-weight exam facts
  - `> [!WARNING]` — common mistakes / gotchas
- **Section introductions**: each major section gets a prose paragraph explaining *why* the topic matters and the mental model before diving into bullet points or tables
- Topic sections that list techniques (missing data, outliers, unbalanced data) follow the pattern: intro + MCAR/MAR/MNAR-style taxonomy → comparison table → exam tip callout

## Exam Domain Map

| File | Domain |
|---|---|
| `01_data_ingestion_storage.md` | Data Ingestion & Storage (S3, Kinesis, MSK, EBS/EFS/FSx) |
| `02_data_transformation.md` | Data Transformation, Feature Engineering (EMR, Glue, SageMaker tooling, Athena) |
| `03_aws_managed_services.md` | AWS Managed AI Services (Comprehend, Rekognition, Transcribe, Textract, etc.) |
| `04_sagemaker.md` | Amazon SageMaker (workflow, training, deployment, built-in tooling) |
| Topics 05–12 | Still in PDF form — not yet converted to Markdown |

## Pending Work

El backlog vivo está en [TODO.md](TODO.md). Consúltalo antes de empezar una tarea nueva y actualízalo al completar un tema o un arreglo.

## Key Exam Facts to Keep in Mind

- **SageMaker and Bedrock** carry the largest share of exam questions
- The exam tests *service selection* given a scenario, not implementation details
- Common distractor pairs: Glue DataBrew vs. SageMaker Data Wrangler · Ground Truth vs. A2I · EMR vs. Glue for ETL
- Passing score: 720/1000 · 65 questions · 130 minutes


## Documentation Style
- Todas las frases en archivos .md deben terminar con punto (.)
- Aplica esto a notas de estudio, README, y cualquier markdown del repo