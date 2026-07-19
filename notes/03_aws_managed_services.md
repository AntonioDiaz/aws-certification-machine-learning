# 03 — AWS Managed AI Services <!-- omit in toc -->

> Source: [`03_MLA-C01_aws_manage_services.pdf`](../udemy_notes/03_MLA-C01_aws_manage_services.pdf)

---

## Table of Contents <!-- omit in toc -->

- [Why This Topic Matters](#why-this-topic-matters)
- [Amazon Comprehend](#amazon-comprehend)
  - [Comprehend — Custom Classification](#comprehend--custom-classification)
  - [Comprehend — Named Entity Recognition (NER)](#comprehend--named-entity-recognition-ner)
  - [Comprehend — Custom Entity Recognition](#comprehend--custom-entity-recognition)
  - [Comprehend — Custom Models](#comprehend--custom-models)
- [Amazon Translate](#amazon-translate)
- [Amazon Transcribe](#amazon-transcribe)
  - [Transcribe — Toxicity Detection](#transcribe--toxicity-detection)
  - [Transcribe — Improving Accuracy](#transcribe--improving-accuracy)
- [Amazon Polly](#amazon-polly)
- [Amazon Rekognition](#amazon-rekognition)
  - [Rekognition — Custom Labels](#rekognition--custom-labels)
  - [Rekognition — Content Moderation](#rekognition--content-moderation)
- [Amazon Lex](#amazon-lex)
- [Amazon Personalize](#amazon-personalize)
  - [Personalize — Recipes](#personalize--recipes)
- [Amazon Textract](#amazon-textract)
- [Amazon Kendra](#amazon-kendra)
- [Amazon Augmented AI (A2I)](#amazon-augmented-ai-a2i)
- [Amazon EC2 for ML](#amazon-ec2-for-ml)
- [Amazon Hardware for AI](#amazon-hardware-for-ai)
- [Amazon Lookout — Anomaly Detection](#amazon-lookout--anomaly-detection)
- [Amazon Fraud Detector](#amazon-fraud-detector)
- [Service Quick-Reference](#service-quick-reference)

---

## Why This Topic Matters

AWS Managed AI Services are **pre-trained, fully managed ML capabilities** exposed as simple APIs. You call the API, AWS does the heavy lifting — no data science or ML infrastructure required. This makes them the fastest path from a business problem to a working AI feature.

**Why use them instead of building from scratch?**
- **Responsiveness and availability**: backed by AWS SLAs and deployed across multiple Availability Zones and regions.
- **Performance**: specialized CPU and GPU hardware optimized per use case, with lower cost than general-purpose compute.
- **Token-based pricing**: pay only for what you use; no idle infrastructure cost.
- **Provisioned throughput**: available for predictable workloads that need consistent performance guarantees.

**On the MLA-C01 exam**, this domain tests your ability to:
- Match a business scenario to the correct managed service.
- Distinguish between services with overlapping capabilities (e.g., Comprehend vs. Rekognition, Lex vs. Connect).
- Know when to use a service's built-in capability vs. its custom model extension.
- Understand A2I's role in adding human review to any AI pipeline.

> [!TIP]
> The exam loves scenario-based questions: "A company needs to extract entities from customer emails" → Comprehend; "detect inappropriate content in images" → Rekognition Content Moderation; "build a voice chatbot" → Lex + Transcribe + Polly.

<img width="2752" height="1536" alt="Image" src="https://github.com/user-attachments/assets/263eb86d-db7d-489f-bce0-5cb16b730756" />

---

## Amazon Comprehend

Amazon Comprehend is a fully managed, serverless **Natural Language Processing (NLP)** service. It uses machine learning to extract meaning and structure from unstructured text — without requiring you to train a model or manage any infrastructure.

<img width="2752" height="1536" alt="Image" src="https://github.com/user-attachments/assets/9f12c712-58d6-4276-8d55-9367f38c8737" />


**What Comprehend can detect from raw text:**
- **Language detection**: automatically identifies the language of the text.
- **Key phrase extraction**: finds the important phrases, people, places, brands, and events.
- **Sentiment analysis**: determines whether the text is positive, negative, neutral, or mixed.
- **Syntax analysis**: tokenizes text and identifies parts of speech (noun, verb, adjective, etc.).
- **Topic modeling**: automatically groups a collection of documents by topic.

**Use cases:**
- Analyze customer emails to determine what leads to a positive or negative experience.
- Create and group support articles by topic that Comprehend uncovers automatically.

> [!TIP]
> Comprehend is the go-to answer for any scenario involving "extract meaning from text" or "sentiment analysis" without building a custom NLP model.

---

### Comprehend — Custom Classification

Custom Classification lets you organize documents into **categories that you define**, going beyond the built-in sentiment/topic capabilities.

Think of it as training a classifier specifically for your business's document types — for example, routing customer emails to the right department based on the type of request.

**How it works:**
1. Provide labeled training data (documents + their category labels) stored in Amazon S3.
2. Comprehend trains a custom classifier on your data.
3. Deploy and classify new documents in real time or in batch.

**Key points:**
- Supports multiple document types: plain text, PDF, Word, images.
- **Real-time Analysis**: single document, synchronous — low latency for live applications.
- **Async Analysis**: multiple documents in batch, asynchronous — use for large volumes.

> [!IMPORTANT]
> Custom Classification requires labeled training data in S3. The workflow is: Training Data → S3 → Comprehend (trains) → Custom Classifier → classify new documents.

---

### Comprehend — Named Entity Recognition (NER)

NER automatically extracts **predefined, general-purpose entities** from text — the kind of structured information buried in unstructured prose.

**Built-in entity types:**
- Person, Organization, Location.
- Date, Quantity, Other (IDs, account numbers, etc.).

**Example**: given a financial letter, Comprehend NER can extract the customer name, company name, dollar amounts, and due dates — all with confidence scores.

---

### Comprehend — Custom Entity Recognition

When your business needs to extract **domain-specific entities** that the built-in NER doesn't cover, Custom Entity Recognition lets you teach Comprehend your own vocabulary.

**Examples of custom entities:**
- Policy numbers, internal product codes, customer IDs.
- Phrases that indicate a customer escalation in support tickets.
- Anything specific to your industry or business.

**How to train:**
- Provide a list of entity values (e.g., all your policy number formats) and/or annotated documents that contain them.
- Comprehend trains a Custom Entity Recognizer model.
- Run real-time or async analysis on new documents to extract the custom entities.

> [!TIP]
> Custom NER vs. Custom Classification: NER extracts *what things are* (entities); Classification decides *what bucket a document belongs to* (category).

---

### Comprehend — Custom Models

Custom models extend Comprehend for both entity recognition and document classification using your own training data.

**Key operational facts:**
- Comprehend manages **model versioning** automatically.
- Custom models can be **copied between AWS accounts**:
  - Attach an IAM policy to a model version, authorizing the target account.
  - The other account imports the model using the source ARN, region, and optional KMS key.
  - Both accounts must be in the **same region**.
  - The import can be done directly from the Comprehend console.

---

## Amazon Translate

Amazon Translate provides **natural and accurate machine translation** between languages. It is a fully managed neural machine translation service designed to translate large volumes of text at scale.

**Primary use:** localize content — websites, applications, user-generated content — for international audiences without building or maintaining translation models.

**Key characteristics:**
- Supports dozens of language pairs.
- Can auto-detect the source language (no need to specify it).
- Works with plain text; integrate with Textract or Comprehend to handle documents first.
- Commonly used alongside Transcribe (speech → text) and Polly (text → speech) to build multilingual voice pipelines.

> [!TIP]
> On the exam: if the scenario involves translating text or localizing an application, the answer is Amazon Translate. It is not used for extracting meaning — that's Comprehend.

---

## Amazon Transcribe

Amazon Transcribe converts **speech to text** using deep learning-based **Automatic Speech Recognition (ASR)**. It is fully managed and works on audio files or streaming audio.

**Core capabilities:**
- Transcribes audio files (MP3, WAV, FLAC, etc.) stored in S3, or streams audio in real time.
- **Automatic PII redaction**: detects and removes Personally Identifiable Information (names, phone numbers, SSNs) from transcripts using Redaction.
- **Automatic Language Identification**: detects the spoken language automatically, useful for multi-lingual audio content.
- Speaker identification (diarization): labels which speaker said which words.

**Use cases:**
- Transcribe customer service calls for QA and compliance.
- Automate closed captioning and subtitling for video content.
- Generate metadata for media assets to create a fully searchable archive.

> [!TIP]
> On the exam: "convert speech to text" or "transcribe call center audio" → Amazon Transcribe. "Remove PII from audio transcripts" → Transcribe + Redaction feature.

---

### Transcribe — Toxicity Detection

Transcribe includes an ML-powered **voice-based toxicity detection** capability that goes beyond just the words — it also analyzes speech cues like tone and pitch.

**Toxicity categories detected:**
- Sexual harassment, hate speech, threats, abuse, profanity, insults, graphic content.

**How it works:**
- Transcribe processes audio and outputs both the transcript and toxicity scores per segment.
- Scores range 0.0–1.0; higher = more toxic.
- You can filter by category and set custom thresholds.

**Use cases:** moderate live or recorded audio in gaming, social platforms, contact centers, and anywhere user-generated voice content is collected.

---

### Transcribe — Improving Accuracy

Out of the box, Transcribe may struggle with technical jargon, brand names, and acronyms. Two mechanisms address this:

| Mechanism | Scope | How it works | Best for |
|---|---|---|---|
| **Custom Vocabularies** | Words | Add specific words, phrases, brand names, acronyms with optional pronunciation hints | Improving recognition of individual terms |
| **Custom Language Models** | Context | Train Transcribe on your domain-specific text corpus to learn word context | Large volumes of domain-specific speech |

> [!IMPORTANT]
> Use **both** Custom Vocabularies and Custom Language Models together for the highest transcription accuracy on domain-specific content.

---

## Amazon Polly

Amazon Polly is the **inverse of Transcribe**: it converts **text to lifelike speech** using deep learning, enabling you to build applications that talk.

**Key features:**

| Feature | Description |
|---|---|
| **Lexicons** | Define custom pronunciations for specific words or abbreviations (e.g., "AWS" → "Amazon Web Services") |
| **SSML** | Speech Synthesis Markup Language — XML-like tags to control pauses, emphasis, pronunciation, and speaking style |
| **Voice engines** | Choose from generative, long-form, neural, or standard engines depending on quality and cost needs |
| **Speech marks** | Metadata encoding where each word starts/ends in the audio — useful for lip-sync or highlighting words as they're spoken |

**Common SSML tags:**
- `<break>` — adds a pause.
- `<emphasis>` — stresses a word.
- `<prosody>` — controls volume, rate, and pitch.
- `<phoneme>` — specifies exact phonetic pronunciation.
- `<say-as>` — controls how special content (dates, numbers) is spoken.

> [!TIP]
> On the exam: "generate audio from text" or "build a talking application" → Polly. "Transcribe audio to text" → Transcribe. They are opposites and should not be confused.

---

## Amazon Rekognition

Amazon Rekognition is a fully managed **computer vision** service that uses ML to analyze images and videos — no ML expertise required.

**Core capabilities:**
- **Object and scene detection**: identify thousands of objects, scenes, and activities in images and video.
- **Facial analysis**: detect faces and analyze attributes (age range, gender, emotions, accessories).
- **Facial search and verification**: compare faces against a database of known faces; verify identity or count people.
- **Celebrity recognition**: match faces against a database of public figures.
- **Text detection**: extract text visible in images or video frames.
- **Pathing**: track the movement of people across video frames (e.g., sports game analysis).

**Use cases:** access control, employee time-and-attendance, content moderation, media metadata, public safety.

> [!TIP]
> On the exam: any scenario involving **images or video analysis** points to Rekognition. For documents and scanned text, use Textract instead.

---

### Rekognition — Custom Labels

Custom Labels extends Rekognition to recognize **objects and scenes specific to your business** that the general model doesn't cover.

**How it works:**
1. Label your training images (only a few hundred images needed).
2. Upload labeled images to Amazon S3.
3. Rekognition trains a Custom Labels model on your dataset.
4. New images are automatically classified using your custom categories.

**Examples:**
- Detect your company logo in social media posts.
- Identify specific products on store shelves.
- The NFL uses Rekognition Custom Labels to find their logo in broadcast footage.

> [!TIP]
> Custom Labels requires far fewer labeled images than building a model from scratch — just a few hundred is enough for many use cases.

---

### Rekognition — Content Moderation

Content Moderation automatically detects **inappropriate, unwanted, or offensive content** in images and video at scale.

**How it works:**
- Rekognition analyzes content and returns moderation labels with confidence scores.
- A **Custom Moderation Adaptor** can be trained on your own labeled images to extend or tune the built-in model.
- Low-confidence detections are routed to **Amazon A2I** for optional human review (typically only 1–5% of content).

**Key benefit:** reduce human review costs dramatically — from reviewing 100% of content down to the 1–5% that is ambiguous.

**Integration flow:**
1. Image/video → `DetectModerationLabels` API → Rekognition returns moderation labels.
2. If confidence is below threshold → Amazon A2I human review queue.
3. Reviewed data can feed back to improve the model.

---

## Amazon Lex

Amazon Lex is a fully managed service for building **conversational interfaces (chatbots)** using both voice and text. It is the same technology that powers Amazon Alexa.

**How it works:**
- You define **Intents** (what the user wants to do, e.g., "BookHotel").
- For each intent, define **Slots** (input parameters the bot needs to collect, e.g., city, check-in date).
- Lex understands natural language, extracts slot values, and invokes an **AWS Lambda function** to fulfill the intent.

**Key integrations:**
- **AWS Lambda**: execute business logic when an intent is fulfilled.
- **Amazon Connect**: build AI-powered voice contact centers.
- **Amazon Comprehend**: add sentiment analysis to understand caller emotion.
- **Amazon Kendra**: answer knowledge-base questions within the bot.

**Use cases:** customer self-service (order pizza, book a hotel, check account balance), IT helpdesks, appointment scheduling.

> [!TIP]
> On the exam, if the question involves building a **chatbot or voice interface**, the answer is Amazon Lex. For a simple speech-to-text, it's Transcribe. Lex adds intent understanding and dialog management on top.

---

## Amazon Personalize

Amazon Personalize is a fully managed ML service that delivers **real-time, personalized recommendations** — the same technology used by Amazon.com itself.

**Why use it:**
- No ML expertise required; implement in days, not months.
- Ingests historical and real-time behavioral data to continuously improve recommendations.
- Integrates directly into websites, mobile apps, SMS, and email marketing platforms.

**Data flow:**
1. Historical interaction data → Amazon S3.
2. Real-time events → Amazon Personalize API (streamed directly).
3. Personalize trains and serves a recommendation model.
4. Applications call the Personalize API to get personalized recommendations per user.

**Use cases:** e-commerce product recommendations, media content suggestions, targeted marketing emails.

> [!TIP]
> On the exam: "recommend products to users based on past behavior" → Amazon Personalize. It is not a general ML service — it is purpose-built for recommendations.

---

### Personalize — Recipes

A **Recipe** is a pre-built algorithm configuration optimized for a specific recommendation use case. You choose the recipe that matches your goal, then provide training configuration on top.

| Recipe Type | Use Case | Example Recipe |
|---|---|---|
| `USER_PERSONALIZATION` | Recommend items for a specific user | User-Personalization-v2 |
| `PERSONALIZED_RANKING` | Re-rank a list of items for a user | Personalized-Ranking-v2 |
| `POPULAR_ITEMS` | Recommend trending or popular items | Trending-Now, Popularity-Count |
| `RELATED_ITEMS` | Recommend items similar to a given item | Similar-Items |
| `PERSONALIZED_ACTIONS` | Recommend the next best action for a user | Next-Best-Action |
| `USER_SEGMENTATION` | Group users by affinity for items | Item-Affinity |

> [!IMPORTANT]
> Recipes and Personalize are specifically for **recommendations** — not general prediction or classification tasks.

---

## Amazon Textract

Amazon Textract automatically **extracts text, handwriting, and structured data** (forms and tables) from scanned documents using AI and ML. It goes well beyond simple OCR.

**What makes it different from plain OCR:**
- Understands the **structure** of a document — it knows a field label belongs to its adjacent value.
- Extracts data from **forms** (key-value pairs) and **tables** (rows and columns) in their structured format.
- Returns results as JSON, preserving the document structure.

**Supported inputs:** PDFs, images (JPEG, PNG, TIFF), and more.

**Use cases:**

| Industry | Examples |
|---|---|
| Financial Services | Invoices, financial reports, tax forms |
| Healthcare | Medical records, insurance claims, prescriptions |
| Public Sector | ID documents, passports, government forms |

> [!TIP]
> On the exam: "extract structured data from scanned documents or forms" → Textract. "Analyze the text content/sentiment of a document" → Comprehend. They are complementary: Textract extracts text; Comprehend understands it.

---

## Amazon Kendra

Amazon Kendra is a fully managed **intelligent document search service** powered by machine learning, designed to answer natural-language questions against a corpus of documents.

**How it differs from keyword search:**
- Understands the intent behind a question and returns a specific answer extracted from documents, not just a list of links.
- Supports a wide range of document types: text, PDF, HTML, PowerPoint, Word, FAQs.

**Key features:**
- **Incremental Learning**: learns from user interactions and feedback to promote preferred results over time.
- **Manual fine-tuning**: adjust result relevance by configuring importance of data freshness, custom attributes, or business rules.
- **Multiple data source connectors**: Amazon S3, RDS, Google Drive, SharePoint, OneDrive, Salesforce, ServiceNow, and 3rd-party/custom sources.

**How it works:**
1. Connect data sources → Kendra indexes the content.
2. User asks a natural language question.
3. Kendra's ML-powered Knowledge Index returns a direct answer with source attribution.

> [!TIP]
> On the exam: "employees need to search internal documents and get direct answers" → Amazon Kendra. "Classify or extract entities from documents" → Comprehend or Textract.

---

## Amazon Augmented AI (A2I)

Amazon A2I adds a **human review layer** on top of any ML prediction pipeline. When a model's confidence falls below a threshold, A2I routes the prediction to a human reviewer before returning the final answer.

**Why it matters:** no ML model is 100% accurate; A2I provides a safety net for high-stakes decisions without requiring you to build your own review workflow.

**How it works:**
1. Input data passes through an AWS AI service or custom ML model.
2. **High-confidence predictions** are returned immediately to the application.
3. **Low-confidence predictions** are sent to human reviewers via A2I.
4. Human reviews are consolidated using weighted scores and stored in Amazon S3.
5. Reviewed data can be fed back into model training to improve future accuracy.

**Reviewer sources:**
- Your own internal employees.
- Over 500,000 contractors from AWS Mechanical Turk.
- Pre-screened vendors for confidentiality requirements.

**Integration:** the underlying ML model can be built on SageMaker, Rekognition, Textract, or any external service.

> [!TIP]
> On the exam: "add human review to ML predictions when confidence is low" → Amazon A2I. Distinguish from **SageMaker Ground Truth**, which is for labeling training data *before* model training — A2I is for reviewing predictions *after* deployment.

| | SageMaker Ground Truth | Amazon A2I |
|---|---|---|
| **When** | Before training | After deployment (production) |
| **Purpose** | Label training data | Review live ML predictions |
| **Output** | Labeled dataset | Reviewed prediction, feedback loop |

---

## Amazon EC2 for ML

Amazon EC2 (Elastic Compute Cloud) is the underlying Infrastructure-as-a-Service layer that powers much of AWS ML workloads. SageMaker, EMR, and other services run on EC2 instances under the hood.

**EC2 sizing options relevant to ML:**

| Resource | Options |
|---|---|
| Operating System | Linux, Windows, Mac OS |
| CPU | General-purpose to high-CPU; number of cores configurable |
| RAM | Standard to high-memory instances |
| Storage | Network-attached (EBS, EFS) or hardware-local (EC2 Instance Store) |
| Network | Configurable network card speed; Public IP address |
| Security | Security groups (firewall rules) |
| Startup | EC2 User Data bootstrap script runs at first launch |

**Key storage choices for ML workloads:**
- **EBS**: persistent block storage; survives instance stop/start.
- **EFS**: shared file system across multiple instances; useful for distributed training.
- **EC2 Instance Store**: ephemeral, high-speed hardware-local storage; lost on stop; best for temporary caching of training data or checkpoints.

**EC2 instance families relevant to ML:**

Choosing the right instance family is a common exam theme — match the workload to the hardware profile.

| Family | Type | Best For |
|---|---|---|
| **T, M** | General purpose | Notebooks, light preprocessing, small experiments |
| **C** | Compute optimized | CPU-bound feature engineering, classical ML training |
| **R, X** | Memory optimized | In-memory datasets, large feature stores, data prep |
| **P** (P3, P4, P5) | GPU — accelerated computing | Deep learning **training** on large models |
| **G** (G4, G5, G6) | GPU — accelerated computing | Deep learning **inference**, graphics, lighter training |
| **Trn** (Trn1) | AWS Trainium | Cost-efficient large-scale deep learning **training** |
| **Inf** (Inf1, Inf2) | AWS Inferentia | Cost-efficient high-throughput **inference** |

> [!TIP]
> On the exam: "train a large deep learning model" → P-family (GPU) or Trn (Trainium). "Serve predictions cheaply at scale" → Inf (Inferentia) or G-family. "CPU-bound classical ML / data prep" → C or M family.

**EC2 pricing models (cost optimization):**

Cost optimization scenarios appear frequently on the exam. Match the purchasing option to the workload's tolerance for interruption and commitment.

| Pricing Model | Description | Best For |
|---|---|---|
| **On-Demand** | Pay per second/hour, no commitment | Short-lived, unpredictable workloads; dev/test |
| **Reserved Instances** | 1- or 3-year commitment; up to ~72% discount | Steady-state, always-on workloads (e.g., persistent endpoints) |
| **Savings Plans** | Commit to $/hour spend for 1–3 years; flexible across families | Predictable spend with flexibility on instance type |
| **Spot Instances** | Use spare capacity; up to ~90% discount; can be interrupted | Fault-tolerant, interruptible training jobs |
| **Dedicated Hosts/Instances** | Physical server isolation | Compliance / licensing requirements |

> [!IMPORTANT]
> **Spot Instances for ML training**: because training can checkpoint and resume, it is an ideal fault-tolerant workload for Spot. SageMaker exposes this as **Managed Spot Training**, which can reduce training costs by up to **90%**. Spot instances can be reclaimed with a 2-minute warning, so always checkpoint to S3.

**Distributed training — networking:**

When training spans multiple GPU instances, inter-node network speed becomes the bottleneck. Two features address this:

- **Cluster Placement Group**: packs instances close together on the same high-bandwidth, low-latency network segment — essential for multi-node distributed training.
- **Elastic Fabric Adapter (EFA)**: a network interface for EC2 that accelerates inter-node communication for tightly-coupled HPC and distributed ML workloads (integrates with NCCL for GPU training).

> [!TIP]
> On the exam: "speed up multi-node distributed deep learning training" → Cluster Placement Group + Elastic Fabric Adapter (EFA).

---

## Amazon Hardware for AI

AWS offers specialized hardware chips purpose-built for ML workloads, reducing cost and energy footprint compared to standard GPUs.

**GPU-based instances:**
- P3, P4, P5 families (NVIDIA GPUs) — industry standard for deep learning training.
- G3–G6 families — GPU graphics and ML inference.

**AWS custom silicon:**

| Chip | Purpose | Key Fact |
|---|---|---|
| **AWS Trainium** (Trn1 instances) | Deep Learning **training** | Built for 100B+ parameter models; ~50% cost reduction vs. GPU training |
| **AWS Inferentia** (Inf1, Inf2 instances) | Deep Learning **inference** | Up to 4× throughput and 70% cost reduction vs. GPU inference |

> [!IMPORTANT]
> Trainium = **training** (Trn1). Inferentia = **inference** (Inf1/Inf2). Both have the lowest environmental footprint of any AWS ML hardware.

> [!TIP]
> On the exam: "reduce cost of training large models" → Trainium. "High-throughput, low-cost inference at scale" → Inferentia.

---

## Amazon Lookout — Anomaly Detection

Amazon Lookout is a family of ML-powered anomaly detection services that automatically learn what "normal" looks like for your data and alert you when something unusual occurs — without requiring ML expertise.

**Core concepts:**
- **Measure**: the metric or value you are trying to protect (e.g., revenue, defect rate, sensor reading).
- **Dimensions**: the features or attributes that influence the measure (e.g., product line, region, machine ID).
- **Detector**: monitors a dataset continuously to find anomalies in the measure across dimensions.

**Three specialized variants:**

| Service | Domain | What it monitors |
|---|---|---|
| **Lookout for Metrics** | Business & operational metrics | Revenue, ad campaigns, KPIs — any time-series metric |
| **Lookout for Equipment** | Industrial IoT | Sensor data from physical machines for predictive maintenance |
| **Lookout for Vision** | Manufacturing quality | Visual inspection of products using computer vision |

**Lookout for Metrics — integrations:**
- **Data sources**: Amazon S3, Athena, Redshift, CloudWatch, Salesforce, Marketo.
- **Alerts**: sends anomaly notifications via Amazon SNS or invokes AWS Lambda for automated response.
- **Feedback loop**: provide accuracy feedback or fine-tune from the AWS Console.

> [!TIP]
> On the exam: "detect anomalies in business metrics like sales or ad clicks" → Lookout for Metrics. "Detect equipment failures before they happen using sensor data" → Lookout for Equipment. "Automate visual quality inspection on a production line" → Lookout for Vision.

---

## Amazon Fraud Detector

Amazon Fraud Detector is a fully managed ML service purpose-built to **detect online fraud** — automatically creating and continuously improving fraud detection models from your own historical data.

**What it handles:**
- Online payment fraud.
- New account creation fraud.
- Trial program abuse.
- Account takeover attacks.

**How it works:**
1. Historical labeled data (fraudulent vs. legitimate) fed from S3 or via API.
2. Fraud Detector handles ETL, feature engineering, model selection, training, tuning, validation, and deployment automatically.
3. A **Fraud Detector Prediction API** returns fraud risk scores in real time.

**Key features:**
- **Automatic model creation and continuous learning**: the model improves over time as new data arrives.
- **Feature importance insights**: understand which variables (features) drive fraud scores.
- **Rule-based actions**: combine ML scores with business rules to customize fraud response.
- **SageMaker integration**: for teams that want to extend with custom models.

> [!TIP]
> On the exam: "detect fraudulent transactions or account creation" → Amazon Fraud Detector. It is distinct from Lookout for Metrics (general anomaly detection) — Fraud Detector is specifically trained on fraud patterns.

---

## Service Quick-Reference

Use this table for fast scenario-to-service matching on the exam.

| Scenario | Service |
|---|---|
| Extract sentiment, entities, or topics from text | Amazon Comprehend |
| Classify documents into custom business categories | Comprehend Custom Classification |
| Extract named entities specific to your domain | Comprehend Custom Entity Recognition |
| Translate text between languages | Amazon Translate |
| Convert speech/audio to text | Amazon Transcribe |
| Remove PII from audio transcripts | Transcribe (Redaction) |
| Detect toxicity in voice content | Transcribe Toxicity Detection |
| Convert text to lifelike speech | Amazon Polly |
| Analyze images or video for objects, faces, scenes | Amazon Rekognition |
| Detect inappropriate content in images/video | Rekognition Content Moderation |
| Recognize custom objects in images | Rekognition Custom Labels |
| Build a conversational chatbot (voice or text) | Amazon Lex |
| Personalized product or content recommendations | Amazon Personalize |
| Extract text and structured data from scanned documents | Amazon Textract |
| Natural language search across enterprise documents | Amazon Kendra |
| Add human review to low-confidence ML predictions | Amazon A2I |
| Label training data with human annotators | SageMaker Ground Truth |
| Detect anomalies in business metrics | Lookout for Metrics |
| Predictive maintenance for industrial equipment | Lookout for Equipment |
| Automated visual quality inspection | Lookout for Vision |
| Detect online payment fraud or account takeovers | Amazon Fraud Detector |
| Cost-efficient deep learning training at scale | AWS Trainium (Trn1) |
| Cost-efficient deep learning inference at scale | AWS Inferentia (Inf1/Inf2) |
