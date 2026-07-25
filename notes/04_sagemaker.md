# 04 — Amazon SageMaker & Built-In Algorithms <!-- omit in toc -->

> Source: [`04_MLA-C01_sagemaker.pdf`](../udemy_notes/04_MLA-C01_sagemaker.pdf)

---

## Table of Contents <!-- omit in toc -->

- [Why This Topic Matters](#why-this-topic-matters)
- [SageMaker Workflow Overview](#sagemaker-workflow-overview)
  - [Data Preparation on SageMaker](#data-preparation-on-sagemaker)
  - [SageMaker Processing](#sagemaker-processing)
  - [Training on SageMaker](#training-on-sagemaker)
  - [Deploying Trained Models](#deploying-trained-models)
  - [SageMaker Input Modes](#sagemaker-input-modes)
- [Built-In Algorithms](#built-in-algorithms)
  - [Linear Learner](#linear-learner)
  - [XGBoost](#xgboost)
  - [LightGBM](#lightgbm)
  - [Seq2Seq](#seq2seq)
  - [DeepAR](#deepar)
  - [BlazingText](#blazingtext)
  - [Object2Vec](#object2vec)
  - [Object Detection](#object-detection)
  - [Image Classification](#image-classification)
  - [Semantic Segmentation](#semantic-segmentation)
  - [Random Cut Forest](#random-cut-forest)
  - [Neural Topic Model (NTM)](#neural-topic-model-ntm)
  - [LDA (Latent Dirichlet Allocation)](#lda-latent-dirichlet-allocation)
  - [KNN (K-Nearest-Neighbors)](#knn-k-nearest-neighbors)
  - [K-Means](#k-means)
  - [PCA (Principal Component Analysis)](#pca-principal-component-analysis)
  - [Factorization Machines](#factorization-machines)
  - [IP Insights](#ip-insights)
- [Algorithm Quick-Reference](#algorithm-quick-reference)
- [Input Format Cheat Sheet](#input-format-cheat-sheet)
- [Instance Type Cheat Sheet](#instance-type-cheat-sheet)

---

## Why This Topic Matters

Amazon SageMaker is AWS's fully managed platform built to handle the **entire machine learning workflow**: 
1. fetch and prepare data
2. train and evaluate a model
3. deploy and monitor it in production. 
Rather than assembling infrastructure yourself, SageMaker provides managed building blocks for each stage.

**SageMaker carries the largest share of MLA-C01 exam questions.** Expect scenarios that test:
- Which built-in algorithm fits a given problem (classification, forecasting, clustering, anomaly detection).
- What input format an algorithm expects (RecordIO-protobuf vs. CSV vs. JSON Lines).
- Which instance type is appropriate (CPU vs. GPU, single vs. multi-machine).
- Which input mode to choose for a given data size and access pattern.
- How to deploy: real-time endpoint vs. Batch Transform.

> [!TIP]
> You do not need to memorize every hyperparameter. 
> Focus on:
>  1. **what each algorithm solves**
>  2. **Its expected input format**
>  3. **Whether it needs GPU** 
> These three axes cover most exam questions.

---

## SageMaker Workflow Overview

The SageMaker lifecycle is a continuous loop:

1. **Fetch, clean, and prepare data** — usually from S3.
2. **Train and evaluate a model** — using built-in algorithms or your own code.
3. **Deploy the model and evaluate results in production** — then feed learnings back into the next iteration.

<img width="2752" alt="Image" src="https://github.com/user-attachments/assets/58d3dea3-0e34-4298-9296-a6555d0f26db" />

**Core architecture:**

| Component | Role |
|---|---|
| **S3 training data** | Input data source for training jobs. |
| **Model training** | Runs a training container pulled from ECR on ML compute instances. |
| **S3 model artifacts** | Where the trained model is saved. |
| **Model deployment / hosting** | Loads model artifacts + inference image to serve predictions. |
| **Endpoint** | HTTPS interface client applications call for predictions. |
| **Amazon ECR** | Stores both the training code image and the inference code image. |

**How you drive SageMaker:**
- **Notebook Instances** (EC2-backed): spin up from the console with S3 access, Scikit-learn, Spark, TensorFlow pre-installed; can launch training jobs and deploy models.
- **SageMaker console**: create and monitor training jobs, tuning jobs, models, and endpoints directly.
- **SDK / API**: programmatic control for automation and pipelines.

---

### Data Preparation on SageMaker

Data usually arrives from **Amazon S3**, and the ideal format varies by algorithm — most often **RecordIO / Protobuf** for the built-in algorithms.

**Supported data sources:**
- Amazon S3 (primary).
- Amazon Athena.
- Amazon EMR.
- Amazon Redshift.
- Amazon Keyspaces DB.

**Tooling available:**
- **Apache Spark** integrates directly with SageMaker for large-scale preparation.
- **Scikit-learn, NumPy, Pandas** are all available inside a notebook for smaller-scale work.

---

### SageMaker Processing

**Processing jobs** run arbitrary data processing workloads on managed infrastructure — useful for preprocessing, postprocessing, and model evaluation at scale.

**How it works:**
1. Copy input data from S3 to `/opt/ml/processing/input` inside the container.
2. Spin up a **processing container** — either a SageMaker built-in image or your own.
3. Write results to `/opt/ml/processing/output`, which is copied back to S3.

> [!TIP]
> On the exam, "run a custom preprocessing script at scale before training" → SageMaker Processing. It is the general-purpose compute escape hatch for data work inside SageMaker.

---

### Training on SageMaker

To train, you create a **training job** specifying four things:

| Requirement | Description |
|---|---|
| **S3 input URL** | Location of the training data. |
| **ML compute resources** | Instance type and count. |
| **S3 output URL** | Where model artifacts are written. |
| **ECR path** | Container image holding the training code. |

**Training options available:**
- Built-in training algorithms (covered below).
- Spark MLLib.
- Custom Python TensorFlow / MXNet code.
- PyTorch, Scikit-Learn, RLEstimator.
- XGBoost, Hugging Face, Chainer.
- Your own Docker image.
- An algorithm purchased from the AWS Marketplace.

---

### Deploying Trained Models

Once trained, the model is saved to S3 and can be deployed in **two main ways**:

| Method | Use Case |
|---|---|
| **Persistent endpoint** | Real-time, individual predictions on demand. |
| **Batch Transform** | Predictions across an entire dataset, offline. |

**Additional deployment options:**

| Option | Purpose |
|---|---|
| **Inference Pipelines** | Chain multiple containers for complex multi-step processing. |
| **SageMaker Neo** | Compile and deploy models to edge devices. |
| **Elastic Inference** | Attach fractional GPU acceleration to reduce deep learning inference cost. |
| **Automatic scaling** | Increase or decrease endpoint instances based on load. |
| **Shadow Testing** | Route a copy of live traffic to a new model to catch regressions before promoting it. |

> [!TIP]
> On the exam: "predictions on an entire dataset overnight" → **Batch Transform**. "Low-latency predictions from an app" → **persistent endpoint**. "Validate a new model against the live one without user impact" → **Shadow Testing**.

---

### SageMaker Input Modes

Input mode determines **how training data gets from storage into the training container** — a frequent exam topic because it drives both start-up latency and cost.

| Mode | Behavior | Best For |
|---|---|---|
| **S3 File Mode** | Default; **copies** the full dataset from S3 to a local directory in the container before training starts. | Smaller datasets that fit on local disk. |
| **S3 Fast File Mode** | Streams from S3 on demand via a FUSE mount; training begins **without waiting for the download**. | Large datasets; works best with sequential access. |
| **Pipe Mode** | Streams data directly from S3 into the algorithm. Largely **replaced by Fast File Mode**. | Legacy streaming use cases. |
| **S3 Express One Zone** | High-performance single-AZ S3 storage class; works with file, fast file, and pipe modes. | Latency-sensitive, high-throughput training. |
| **Amazon FSx for Lustre** | Scales to hundreds of GB/s throughput and millions of IOPS with low latency. Single AZ, **requires VPC**. | Very large datasets and distributed training. |
| **Amazon EFS** | Mounts an existing EFS file system. Data must **already be in EFS**; requires VPC. | Data already living in EFS shared across teams. |

> [!IMPORTANT]
> **File Mode** waits for the full download before training begins; **Fast File Mode** starts immediately. For massive datasets where download time dominates, Fast File Mode or FSx for Lustre is the answer.

---

## Built-In Algorithms

SageMaker ships with a catalog of pre-implemented, optimized algorithms. For the exam, learn each one's **purpose**, **input format**, and **hardware requirement**.

---

### Linear Learner

**What it's for:**
- Fits a linear model to your training data and predicts from that line.
- Handles both **regression** (numeric predictions) and **classification**.
- For classification it applies a linear threshold function; supports **binary or multi-class**.

**Training input:**
- **RecordIO-wrapped protobuf** — **Float32 data only**.
- **CSV** — first column is assumed to be the label.
- File or Pipe mode both supported.

**How it's used:**
- **Preprocessing**: training data must be **normalized** so all features are weighted equally — Linear Learner can do this automatically. Input data should also be **shuffled**.
- **Training**: uses stochastic gradient descent; choose an optimizer (Adam, AdaGrad, SGD). Multiple models are optimized in parallel.
- **Validation**: the most optimal model is automatically selected.

**Important hyperparameters:**

| Hyperparameter | Purpose |
|---|---|
| `balance_multiclass_weights` | Gives each class equal importance in the loss function — useful for imbalanced data. |
| `learning_rate`, `mini_batch_size` | Standard training controls. |
| `L1` | L1 regularization. |
| `wd` | Weight decay (L2 regularization). |
| `target_precision` | Used with `binary_classifier_model_selection_criteria = recall_at_target_precision`; holds precision fixed while maximizing recall. |
| `target_recall` | Used with `precision_at_target_recall`; holds recall fixed while maximizing precision. |

**Instance types:** single or multi-machine, CPU or GPU. **Multi-GPU does not help.**

---

### XGBoost

**What it's for:**
- **eXtreme Gradient Boosting** — a boosted ensemble of decision trees where each new tree corrects the errors of previous trees, using gradient descent to minimize loss.
- Handles **classification** and **regression** (via regression trees).
- Consistently wins Kaggle competitions and is fast.

**Training input:**
- **CSV or libsvm** (it is open-source XGBoost, not built natively for SageMaker).
- AWS extended it to also accept **recordIO-protobuf and Parquet**.

**How it's used:**
- Models are serialized/deserialized with **Pickle**.
- Can be used as a framework within notebooks (`sagemaker.xgboost`) or as a built-in SageMaker algorithm.

**Important hyperparameters:**

| Hyperparameter | Purpose |
|---|---|
| `subsample` | Prevents overfitting. |
| `eta` | Step size shrinkage; prevents overfitting. |
| `gamma` | Minimum loss reduction to create a partition; larger = more conservative. |
| `alpha` | L1 regularization term; larger = more conservative. |
| `lambda` | L2 regularization term; larger = more conservative. |
| `eval_metric` | Optimize on AUC, error, RMSE. Use AUC if false positives matter more than raw accuracy. |
| `scale_pos_weight` | Balances positive/negative weights — helpful for **unbalanced classes**; often set to `sum(negative) / sum(positive)`. |
| `max_depth` | Max tree depth; too high leads to overfitting. |

**Instance types:**
- **Memory-bound, not compute-bound** → **M5 is a good choice**.
- XGBoost 1.2+: single-instance GPU training available (P2, P3) — must set `tree_method = gpu_hist`.
- XGBoost 1.2-2: P2, P3, G4dn, G5.
- XGBoost 1.5+: distributed GPU training — set `use_dask_gpu_training = true` and `distribution = fully_replicated`; only works with CSV or Parquet input.

> [!IMPORTANT]
> XGBoost is **memory-bound**. On the exam, choose **M5 (general purpose)** over C5 (compute optimized) for XGBoost training.

---

### LightGBM

**What it's for:**
- A **Gradient Boosting Decision Tree** similar to XGBoost and CatBoost.
- Extended with **Gradient-based One-Side Sampling (GOSS)** and **Exclusive Feature Bundling (EFB)**.
- Supports **classification, regression, or ranking**.

**Training input:** requires **txt/csv** for both training and inference. Training and optional validation channels may be provided.

**Important hyperparameters:**

| Hyperparameter | Purpose |
|---|---|
| `learning_rate` | Standard training control. |
| `num_leaves` | Max leaves per tree. |
| `feature_fraction` | Subset of features used per tree. |
| `bagging_fraction` | Similar to feature_fraction but randomly sampled. |
| `bagging_freq` | How often bagging is performed. |
| `max_depth` | Max tree depth. |
| `min_data_in_leaf` | Minimum data in one leaf; helps address overfitting. |

**Instance types:** single or multi-instance CPU training (set via `instance_count`). **Memory-bound** → choose general purpose (**M5, not C5**) and ensure enough memory.

---

### Seq2Seq

**What it's for:**
- Input is a **sequence of tokens**, output is a **sequence of tokens**.
- Machine translation, text summarization, speech-to-text.
- Implemented with RNNs and CNNs with attention.

**Training input:**
- **RecordIO-Protobuf** where **tokens must be integers** (unusual — most algorithms want floating point).
- Start with tokenized text files, convert to protobuf packing into integer tensors with vocabulary files.
- Must provide **training data, validation data, and vocabulary files**.

**How it's used:**
- Training for machine translation can take **days**, even on SageMaker.
- Pre-trained models and public training datasets are available for specific translation tasks.

**Important hyperparameters:** `batch_size`, `optimizer_type` (adam, sgd, rmsprop), `learning_rate`, `num_layers_encoder`, `num_layers_decoder`.

**Optimization metrics:**
- **Accuracy** — vs. a provided validation dataset.
- **BLEU score** — compares against multiple reference translations.
- **Perplexity** — cross-entropy.

**Instance types:** **GPU only** (e.g., P3). **Single machine only**, but multi-GPU on one machine is supported.

---

### DeepAR

**What it's for:**
- Forecasting **one-dimensional time series** data using RNNs.
- Allows training a **single model across several related time series**.
- Automatically finds frequencies and seasonality.

**Training input:**
- **JSON Lines** format (Gzip or Parquet).
- Each record **must contain**: `start` (starting timestamp) and `target` (the time series values).
- Each record **can contain**: `dynamic_feat` (dynamic features, e.g., whether a promotion was applied) and `cat` (categorical features).

**How it's used:**
- Always include the **entire time series** for training, testing, and inference.
- Use the entire dataset as the training set, remove the last time points for testing, and evaluate on the withheld values.
- **Don't use very large values for prediction length (> 400).**
- Train on **many time series** rather than just one whenever possible.

**Important hyperparameters:** `context_length` (number of time points seen before predicting — can be smaller than seasonality since the model lags one year anyway), `epochs`, `mini_batch_size`, `learning_rate`, `num_cells`.

**Instance types:**
- CPU or GPU, single or multi-machine.
- **Start with CPU** (`ml.c4.2xlarge`, `ml.c4.4xlarge`); move to GPU only for larger models or mini-batch sizes > 512.
- **CPU-only for inference.**
- May need larger instances for tuning.

---

### BlazingText

**What it's for — two distinct modes:**

1. **Text classification** (supervised): predicts labels for a *sentence*. Useful in web search and information retrieval.
2. **Word2vec** (unsupervised): creates a **word embedding** — a vector representation where semantically similar words sit close together. Useful *for* NLP tasks (machine translation, sentiment analysis) but is not an NLP algorithm itself.

> [!WARNING]
> Word2vec works on **individual words only** — not sentences or documents. If a question describes embedding whole sentences or arbitrary objects, the answer is **Object2Vec**, not BlazingText.

**Training input:**
- **Supervised mode**: one sentence per line, with the first "word" being the string `__label__` followed by the label. Also supports "augmented manifest text format".
- **Word2vec**: a plain text file with one training sentence per line.

**How it's used — Word2vec modes:**
- **Cbow** (Continuous Bag of Words).
- **Skip-gram**.
- **Batch skip-gram** — distributed computation over many CPU nodes.

**Important hyperparameters:**
- Word2vec: `mode` (batch_skipgram, skipgram, cbow), `learning_rate`, `window_size`, `vector_dim`, `negative_samples`.
- Text classification: `epochs`, `learning_rate`, `word_ngrams`, `vector_dim`.

**Instance types:**
- cbow and skipgram: recommend a single `ml.p3.2xlarge` (any single CPU or single GPU instance works).
- batch_skipgram: single or multiple CPU instances.
- Text classification: **C5** if less than 2GB training data; for larger datasets use a single GPU instance (`ml.p2.xlarge` or `ml.p3.2xlarge`).

---

### Object2Vec

**What it's for:**
- Like Word2vec but generalized to **arbitrary objects**, not just words.
- Creates **low-dimensional dense embeddings of high-dimensional objects**.
- Compute nearest neighbors of objects, visualize clusters, genre prediction, recommendations (similar items or users).

**Training input:**
- Data must be **tokenized into integers**.
- Training data consists of **pairs** of tokens and/or sequences of tokens: sentence–sentence, labels–sequence, customer–customer, product–product, user–item.

**How it's used:**
- Process data into JSON Lines and **shuffle it**.
- Train with **two input channels, two encoders, and a comparator**.
- Encoder choices: average-pooled embeddings, CNNs, bidirectional LSTM.
- The comparator is followed by a feed-forward neural network.

**Important hyperparameters:**
- Standard deep learning ones: dropout, early stopping, epochs, learning rate, batch size, layers, activation function, optimizer, weight decay.
- `enc1_network`, `enc2_network` — choose `hcnn`, `bilstm`, `pooled_embedding`.

**Instance types:**
- **Single machine only** (CPU or GPU, multi-GPU OK).
- `ml.m5.2xlarge`, `ml.p2.xlarge`; scale up to `ml.m5.4xlarge` or `ml.m5.12xlarge` if needed.
- GPU options: P2, P3, G4dn, G5.
- **Inference**: use `ml.p3.2xlarge`; set the `INFERENCE_PREFERRED_MODE` environment variable to optimize for encoder embeddings rather than classification/regression.

---

### Object Detection

**What it's for:**
- Identify **all objects in an image with bounding boxes**.
- Detects and classifies objects with a single deep neural network; classes come with confidence scores.
- Can train from scratch or use pre-trained models based on ImageNet.

**How it's used — two variants:**
- **MXNet**: uses a CNN with the **Single Shot multibox Detector (SSD)** algorithm; base CNN can be **VGG-16 or ResNet-50**. Supports transfer learning / incremental training using pre-trained base network weights. Uses flip, rescale, and jitter internally to avoid overfitting.
- **TensorFlow**: uses ResNet, EfficientNet, MobileNet models from the TensorFlow Model Garden.

**Training input:**
- MXNet: **RecordIO or image format** (jpg or png).
- With image format, supply a **JSON file for annotation data** for each image (image size, bounding box annotations with class_id/left/top/width/height, and categories).

**Important hyperparameters:** `mini_batch_size`, `learning_rate`, `optimizer` (sgd, adam, rmsprop, adadelta).

**Instance types:**
- **GPU for training** (multi-GPU and multi-machine OK): `ml.p2.xlarge`, `ml.p2.16xlarge`, `ml.p3.2xlarge`, `ml.p3.16xlarge`, G4dn, G5.
- **CPU or GPU for inference**: M5, P2, P3, G4dn all OK.

---

### Image Classification

**What it's for:**
- Assign **one or more labels to an entire image**.
- Does **not** tell you where objects are — only what objects are present.

**How it's used — separate algorithms for MXNet and TensorFlow:**
- **MXNet**:
  - **Full training mode**: network initialized with random weights.
  - **Transfer learning mode**: initialized with pre-trained weights; the top fully-connected layer gets random weights and the network is fine-tuned with new training data.
  - Default image size is **3-channel 224×224** (ImageNet's dataset).
- **TensorFlow**: uses TensorFlow Hub models (MobileNet, Inception, ResNet, EfficientNet); top classification layer available for fine tuning.

**Important hyperparameters:** the usual deep learning suspects — batch size, learning rate, optimizer. Optimizer-specific parameters: weight decay, beta 1, beta 2, eps, gamma (slightly different between MXNet and TensorFlow versions).

**Instance types:**
- **GPU for training** (`ml.p2`, `p3`, `g4dn`, `g5`); multi-GPU and multi-machine OK.
- **CPU or GPU for inference** (m5, p2, p3, g4dn, g5).

---

### Semantic Segmentation

**What it's for:**
- **Pixel-level object classification** — produces a **segmentation mask**.
- Different from image classification (labels whole images) and from object detection (labels bounding boxes).
- Useful for self-driving vehicles, medical imaging diagnostics, robot sensing.

**Training input:**
- **JPG images and PNG annotations**, for both training and validation.
- **Label maps** to describe annotations.
- Augmented manifest image format supported for Pipe mode.
- JPG images accepted for inference.

**How it's used:**
- Built on **MXNet Gluon and Gluon CV**.
- Choice of 3 algorithms: **Fully-Convolutional Network (FCN)**, **Pyramid Scene Parsing (PSP)**, **DeepLabV3**.
- Choice of backbones: **ResNet50** or **ResNet101**, both trained on ImageNet.
- Incremental training or training from scratch both supported.

**Important hyperparameters:** epochs, learning rate, batch size, optimizer, plus `algorithm` and `backbone`.

**Instance types:** GPU for training (`ml.p2`, `p3`, `g4dn`, `g5`); CPU or GPU for inference.

> [!TIP]
> **The three vision algorithms compared**: Image Classification = "what is in this image?" · Object Detection = "where is each object (bounding box)?" · Semantic Segmentation = "which class does each pixel belong to?".

---

### Random Cut Forest

**What it's for:**
- **Anomaly detection**, unsupervised.
- Detects unexpected spikes in time series data, breaks in periodicity, and unclassifiable data points.
- Assigns an **anomaly score** to each data point.
- Based on an algorithm developed by Amazon.

**Training input:**
- **RecordIO-protobuf or CSV**; File or Pipe mode on either.
- Optional **test channel** for computing accuracy, precision, recall, and F1 on labeled data (anomaly or not).

**How it's used:**
- Creates a forest of trees where each tree is a partition of the training data; looks at the expected change in tree complexity when adding a point.
- Data is sampled randomly, then trained.
- **RCF also appears in Kinesis Analytics** — it can work on streaming data too.

**Important hyperparameters:**
- `num_trees` — increasing reduces noise.
- `num_samples_per_tree` — should be chosen so that `1/num_samples_per_tree` approximates the ratio of anomalous to normal data.

**Instance types:** **does not take advantage of GPUs.** Use **M4, C4, or C5** for training; `ml.c5.xl` for inference.

---

### Neural Topic Model (NTM)

**What it's for:**
- Organize documents into **topics**; classify or summarize documents based on topics.
- Goes beyond TF/IDF — e.g., "bike", "car", "train", "mileage", "speed" might group a document under a transportation-like topic (though it won't name it).
- **Unsupervised**; the algorithm is **Neural Variational Inference**.

**Training input:**
- **Four data channels**: `train` is **required**; `validation`, `test`, and `auxiliary` are optional.
- **recordIO-protobuf or CSV**.
- Words must be **tokenized into integers**; every document must contain a count for every word in the vocabulary in CSV.
- The **auxiliary channel is for the vocabulary**.
- File or Pipe mode.

**How it's used:**
- You define **how many topics** you want.
- Topics are a **latent representation** based on top-ranking words.
- One of **two topic modeling algorithms** in SageMaker — you can try both.

**Important hyperparameters:** lowering `mini_batch_size` and `learning_rate` can reduce validation loss, at the expense of training time. Also `num_topics`.

**Instance types:** GPU or CPU. **GPU recommended for training**; CPU OK for inference and is cheaper.

---

### LDA (Latent Dirichlet Allocation)

**What it's for:**
- **Latent Dirichlet Allocation** — another topic modeling algorithm, but **not deep learning**.
- **Unsupervised**; topics themselves are unlabeled — they are just groupings of documents with a shared subset of words.
- Can be used for things other than words: clustering customers based on purchases, harmonic analysis in music.

**Training input:**
- Train channel, optional test channel.
- **recordIO-protobuf or CSV**.
- Each document has counts for every word in the vocabulary (in CSV format).
- **Pipe mode only supported with recordIO.**

**How it's used:**
- Unsupervised; generates however many topics you specify.
- Optional test channel can be used for scoring results (per-word log likelihood).
- **Functionally similar to NTM, but CPU-based** — therefore potentially cheaper and more efficient.

**Important hyperparameters:**
- `num_topics`.
- `alpha0` — initial guess for the concentration parameter. Smaller values generate **sparse** topic mixtures; larger values (> 1.0) produce **uniform** mixtures.

**Instance types:** **single-instance CPU training only.**

> [!IMPORTANT]
> **NTM vs. LDA**: both do topic modeling. NTM is neural/GPU-based; LDA is CPU-only and often cheaper. If the exam emphasizes cost efficiency for topic modeling, LDA is the answer.

---

### KNN (K-Nearest-Neighbors)

**What it's for:**
- Simple **classification or regression** algorithm.
- **Classification**: find the K closest points to a sample and return the most frequent label.
- **Regression**: find the K closest points and return the average value.

**Training input:**
- Train channel contains your data; test channel emits accuracy or MSE.
- **recordIO-protobuf or CSV** — first column is the label.
- File or Pipe mode on either.

**How it's used:**
- Data is first **sampled**.
- SageMaker includes a **dimensionality reduction stage** to avoid the curse of dimensionality, at the cost of noise/accuracy — using "sign" or "fjlt" methods.
- Builds an index for looking up neighbors, serializes the model, then queries the model for a given K.

**Important hyperparameters:** `K` and `sample_size`.

**Instance types:**
- Training on CPU or GPU: `ml.m5.2xlarge`, `ml.p2.xlarge`.
- Inference: **CPU for lower latency**, **GPU for higher throughput on large batches**.

---

### K-Means

**What it's for:**
- **Unsupervised clustering.**
- Divides data into K groups where members of a group are as similar as possible to each other.
- You define what "similar" means; measured by **Euclidean distance**.
- Web-scale K-Means clustering.

**Training input:**
- Train channel, optional test — train uses `ShardedByS3Key`, test uses `FullyReplicated`.
- **recordIO-protobuf or CSV**; File or Pipe on either.

**How it's used:**
- Every observation is mapped to n-dimensional space (n = number of features).
- Works to optimize the center of K clusters.
- **"Extra cluster centers"** may be specified to improve accuracy, which get reduced down to k (`K = k * x`).
- Algorithm: determine initial cluster centers (random or **k-means++**, which tries to make initial clusters far apart) → iterate over training data and calculate cluster centers → reduce clusters from K to k using Lloyd's method with kmeans++.

**Important hyperparameters:**
- `K` — choosing K is tricky. Plot **within-cluster sum of squares** as a function of K and use the **"elbow method"**; essentially optimize for tightness of clusters.
- `mini_batch_size`, `extra_center_factor`, `init_method`.

**Instance types:** CPU or GPU, but **CPU recommended**. Only one GPU per instance is used on GPU, so use `ml.g4dn.xlarge` if using GPU. P2, P3, G4dn, G4 supported.

---

### PCA (Principal Component Analysis)

**What it's for:**
- **Dimensionality reduction** — project higher-dimensional data into lower dimensions (like a 2D plot) while minimizing loss of information.
- Reduced dimensions are called **components**: the first component has the largest possible variability, the second the next largest, and so on.
- **Unsupervised.**

**Training input:** recordIO-protobuf or CSV; File or Pipe on either.

**How it's used:**
- A **covariance matrix** is created, then **singular value decomposition (SVD)** is applied.
- **Two modes**:
  - **Regular** — for sparse data and a moderate number of observations and features.
  - **Randomized** — for a large number of observations and features; uses an approximation algorithm.

**Important hyperparameters:** `algorithm_mode` and `subtract_mean` (unbias the data).

**Instance types:** GPU or CPU — "it depends on the specifics of the input data".

---

### Factorization Machines

**What it's for:**
- Dealing with **sparse data**: click prediction, item recommendations.
- Since an individual user doesn't interact with most pages/products, the data is inherently sparse.
- **Supervised** — classification or regression.
- **Limited to pair-wise interactions** (e.g., user → item).

**Training input:** **recordIO-protobuf with Float32 only** — sparse data means CSV isn't practical.

**How it's used:**
- Finds factors used to predict a classification (click or not? purchase or not?) or a value (predicted rating?) given a matrix representing some pair of things (users & items).
- Usually used in the context of **recommender systems**.

**Important hyperparameters:** initialization methods for bias, factors, and linear terms — uniform, normal, or constant; properties of each method can be tuned.

**Instance types:** CPU or GPU — **CPU recommended**; GPU only works with dense data.

> [!TIP]
> On the exam, "recommendations with sparse user-item data" → **Factorization Machines**. Note the input must be **recordIO-protobuf**, never CSV.

---

### IP Insights

**What it's for:**
- **Unsupervised learning of IP address usage patterns.**
- Identifies suspicious behavior from IP addresses: logins from anomalous IPs, accounts creating resources from anomalous IPs.

**Training input:**
- User names and account IDs can be fed in **directly — no pre-processing needed**.
- Training channel, optional validation channel (computes AUC score).
- **CSV only**: entity, IP.

**How it's used:**
- Uses a neural network to learn **latent vector representations** of entities and IP addresses.
- Entities are hashed and embedded — needs a sufficiently large hash size.
- Automatically generates **negative samples** during training by randomly pairing entities and IPs.

**Important hyperparameters:**
- `num_entity_vectors` — hash size; set to **twice the number of unique entity identifiers**.
- `vector_dim` — size of embedding vectors; scales model size, too large results in overfitting.
- Epochs, learning rate, batch size, etc.

**Instance types:** CPU or GPU — **GPU recommended** (`ml.p3.2xlarge` or higher); can use multiple GPUs. Size of CPU instance depends on `vector_dim` and `num_entity_vectors`.

---

## Algorithm Quick-Reference

Match the scenario to the algorithm — this is the highest-value table for the exam.

| Scenario | Algorithm |
|---|---|
| Linear regression or linear classification | **Linear Learner** |
| Tabular classification/regression, best general accuracy | **XGBoost** |
| Gradient boosting with ranking support | **LightGBM** |
| Machine translation, text summarization, speech-to-text | **Seq2Seq** |
| Time series forecasting | **DeepAR** |
| Text classification or word embeddings (single words) | **BlazingText** |
| Embeddings of arbitrary objects / sentences / user-item pairs | **Object2Vec** |
| Find objects in an image **with bounding boxes** | **Object Detection** |
| Assign labels to a whole image | **Image Classification** |
| Classify **every pixel** in an image | **Semantic Segmentation** |
| Anomaly detection / unexpected spikes | **Random Cut Forest** |
| Topic modeling (neural, GPU) | **Neural Topic Model (NTM)** |
| Topic modeling (CPU, cheaper) | **LDA** |
| Simple classification/regression by nearest neighbors | **KNN** |
| Unsupervised clustering into K groups | **K-Means** |
| Dimensionality reduction | **PCA** |
| Recommendations on sparse user-item data | **Factorization Machines** |
| Detect suspicious IP address behavior | **IP Insights** |

---

## Input Format Cheat Sheet

| Algorithm | Accepted Input |
|---|---|
| Linear Learner | RecordIO-protobuf (Float32 only), CSV (label first column) |
| XGBoost | CSV, libsvm, recordIO-protobuf, Parquet |
| LightGBM | txt / CSV only |
| Seq2Seq | RecordIO-protobuf (**integer** tokens) + vocabulary files |
| DeepAR | **JSON Lines** (Gzip or Parquet) |
| BlazingText | Text file, one sentence per line (`__label__` prefix for supervised) |
| Object2Vec | JSON Lines, tokenized into integers, in pairs |
| Object Detection | RecordIO or image (jpg/png) + JSON annotations |
| Image Classification | RecordIO or image formats |
| Semantic Segmentation | JPG images + PNG annotations + label maps |
| Random Cut Forest | RecordIO-protobuf or CSV |
| NTM | recordIO-protobuf or CSV (4 channels; auxiliary = vocabulary) |
| LDA | recordIO-protobuf or CSV (Pipe mode only with recordIO) |
| KNN | recordIO-protobuf or CSV (label first column) |
| K-Means | recordIO-protobuf or CSV |
| PCA | recordIO-protobuf or CSV |
| Factorization Machines | **recordIO-protobuf with Float32 only** |
| IP Insights | **CSV only** (entity, IP) |

---

## Instance Type Cheat Sheet

| Algorithm | Training | Notes |
|---|---|---|
| Linear Learner | CPU or GPU, single/multi-machine | Multi-GPU does **not** help. |
| XGBoost | **M5 (memory-bound)**; GPU via `gpu_hist` | Not compute-bound — avoid C5. |
| LightGBM | CPU, single/multi-instance | Memory-bound → M5, not C5. |
| Seq2Seq | **GPU only, single machine** | Multi-GPU on one machine OK. |
| DeepAR | Start CPU (c4), GPU for large models | **CPU-only for inference.** |
| BlazingText | p3.2xlarge (cbow/skipgram); C5 for small text classification | batch_skipgram supports multi-CPU. |
| Object2Vec | **Single machine only** (CPU/GPU) | Inference: ml.p3.2xlarge. |
| Object Detection | **GPU** (multi-GPU & multi-machine OK) | CPU or GPU for inference. |
| Image Classification | **GPU** (multi-GPU & multi-machine OK) | CPU or GPU for inference. |
| Semantic Segmentation | **GPU** | CPU or GPU for inference. |
| Random Cut Forest | **No GPU support** — M4, C4, C5 | ml.c5.xl for inference. |
| NTM | GPU recommended | CPU OK (cheaper) for inference. |
| LDA | **Single-instance CPU only** | No GPU. |
| KNN | CPU or GPU | CPU = lower latency; GPU = higher throughput. |
| K-Means | **CPU recommended** | Only 1 GPU per instance used. |
| PCA | GPU or CPU | Depends on input data. |
| Factorization Machines | **CPU recommended** | GPU only works with dense data. |
| IP Insights | **GPU recommended** (p3.2xlarge+) | Multiple GPUs supported. |

> [!IMPORTANT]
> Algorithms that **do not benefit from GPU**: Random Cut Forest (no GPU support), LDA (CPU-only), K-Means and Factorization Machines (CPU recommended). Algorithms that **require GPU**: Seq2Seq. The vision algorithms (Object Detection, Image Classification, Semantic Segmentation) all want GPU for training.
