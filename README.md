# JAVA-VECTOR-LIBS

A Java library project providing support and implementations for vector search, embeddings, and AI/ML — mirroring the functionality of popular Python libraries used in real-world AI/ML applications.

---

## Python Libraries (Real-World Usage) & Java Equivalents

---

### 1. Numerical Computing & Data Manipulation

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **NumPy** | Foundational n-dimensional array computing. Core dependency for all ML/DL stacks. | **ND4J** |
| **Pandas** | DataFrames for tabular data manipulation and analysis. | **Tablesaw** |
| **Polars** | Fast, memory-efficient DataFrame library written in Rust. Modern Pandas alternative. | **Tablesaw** / **Smile** |
| **SciPy** | Scientific computing — optimization, linear algebra, statistics, signal processing. | **Apache Commons Math** / **Ojalgo** |
| **Dask** | Distributed parallel computing with Pandas-like API for large datasets. | **Apache Spark Java API** |
| **CuPy** | GPU-accelerated NumPy-compatible array library for NVIDIA GPUs. | **ND4J (CUDA backend)** |
| **JAX** | NumPy + automatic differentiation + JIT compilation. Google's ML research framework. | **DL4J / DJL** |

---

### 2. Machine Learning

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **scikit-learn** | Industry-standard ML — classification, regression, clustering, dimensionality reduction. | **Smile** / **Weka** |
| **XGBoost** | Optimized gradient boosting for tabular data. | **Smile** / **H2O** |
| **LightGBM** | Fast, low-memory gradient boosting from Microsoft. | **H2O** / **Spark MLlib** |
| **CatBoost** | Gradient boosting with native categorical feature support from Yandex. | **H2O** |
| **Optuna** | Bayesian hyperparameter optimization with pruning and parallelization. | **H2O AutoML** |
| **H2O AutoML** | Automated ML — algorithm selection, hyperparameter tuning. | **H2O AutoML (Java)** |
| **Apache Spark MLlib** | Distributed ML for classification, regression, clustering at scale. | **Apache Spark MLlib (Java API)** |

---

### 3. Deep Learning Frameworks

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **TensorFlow** | Google's production deep learning framework. TF Lite, TF.js ecosystem. | **TensorFlow Java API** / **DJL** |
| **PyTorch** | Facebook's deep learning framework with dynamic graphs. Research and production. | **DL4J** / **DJL (PyTorch engine)** |
| **Keras** | High-level neural network API integrated into TensorFlow. | **DL4J Keras import** |
| **JAX** | NumPy-like automatic differentiation and JIT compilation for ML research. | **DJL** |
| **PyTorch Lightning** | Structured training loops and distributed training wrapper for PyTorch. | **DL4J** |
| **MXNet** | Multi-language scalable deep learning framework from Apache. | **DJL (MXNet engine)** |
| **Flax** | Functional neural network library built on JAX for research. | **DJL** |

---

### 4. Computer Vision

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **OpenCV** | Industry-standard computer vision. Image processing, video analysis, feature detection. | **OpenCV Java bindings** |
| **Pillow (PIL)** | Image loading, manipulation, and processing. | **ImageJ / BoofCV** |
| **torchvision** | PyTorch vision library — pre-trained models, transforms, datasets. | **DJL Vision module** |
| **Albumentations** | Fast image augmentation library (90+ techniques) for deep learning. | **DJL** |
| **scikit-image** | Image processing algorithms on NumPy/SciPy. | **BoofCV** |
| **YOLOv8** | State-of-the-art real-time object detection by Ultralytics. | **DJL Model Zoo** |
| **Detectron2** | Facebook's object detection and segmentation framework on PyTorch. | **DJL** |
| **timm** | 1000+ pre-trained vision models — EfficientNet, ViT, ResNet, DenseNet. | **DJL Model Zoo** |

---

### 5. Natural Language Processing (NLP)

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **NLTK** | Comprehensive NLP toolkit — tokenization, stemming, tagging, parsing. | **OpenNLP** |
| **spaCy** | Industrial-strength NLP — fast tokenization, POS tagging, NER, dependency parsing. | **Stanford CoreNLP** |
| **Hugging Face Transformers** | 50,000+ pre-trained transformer models — BERT, GPT, T5, LLaMA, Mistral, etc. | **DJL** / **LangChain4j** |
| **Tokenizers** (Hugging Face) | Production-grade Rust-based tokenizers — BPE, WordPiece, SentencePiece. | **HuggingFace Tokenizers Java** / **DJL** |
| **Gensim** | Word2Vec, FastText, Doc2Vec, LDA — word embeddings and topic modeling. | **Word2vec Java** |
| **Stanza (StanfordNLP)** | Stanford's NLP toolkit — tokenization, POS, NER, dependency parsing. | **Stanford CoreNLP** |
| **SentencePiece** | Unsupervised tokenizer for modern LLMs and neural machine translation. | **DJL NLP** |
| **TextBlob** | Simplified NLP — sentiment analysis, lemmatization, n-grams. | **OpenNLP** |
| **FairSeq** | Facebook's sequence modeling toolkit for machine translation. | **DJL** |

---

### 6. Embeddings & Semantic Search

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **Sentence-Transformers** | Dense vector embeddings from text using transformer models. Standard for semantic similarity and retrieval. | **DJL Embeddings** / **LangChain4j** |
| **Hugging Face Embeddings** | Pre-trained embedding models from Hub — BERT, MPNet, E5, bge-m3. | **LangChain4j Embeddings** |
| **Universal Sentence Encoder** | Google's pre-trained multilingual sentence encoder (TF Hub). | **DJL** |
| **Gensim Word2Vec/FastText** | Classical word embedding algorithms. | **Word2vec Java** |
| **InstructOR** | Instruction-tuned text embedding model for few-shot generalization. | **DJL** |

---

### 7. Vector Search

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **FAISS** | Meta AI's industry-standard efficient similarity search and clustering. GPU-accelerated. | **JVector** / **Apache Lucene** |
| **Hnswlib** | Fast approximate nearest neighbor search via HNSW graphs. Lightweight and local. | **Hnswlib Java** / **Hnswlib-JNA** |
| **Annoy** | Spotify's static memory-mapped approximate nearest neighbor search. | **Voyager Java** |
| **Voyager** | Spotify's HNSW-based successor to Annoy. Python and Java bindings. | **Voyager (Java bindings)** |
| **ScaNN** | Google's Scalable Nearest Neighbors library. Optimized for large-scale cloud deployments. | **JVector** |

---

### 8. Vector Databases

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **Chroma** | Open-source embedded vector database for LLM applications. | **JVector** (embedded) |
| **LanceDB** | Embedded vector database on Lance columnar format. Zero-copy, in-process. | **JVector** |
| **Weaviate** (Python client) | Open-source vector DB — built-in embeddings, hybrid search, GraphQL API. | **Weaviate Java Client** |
| **Pinecone** (Python SDK) | Fully managed serverless vector database. | **Pinecone Java SDK** |
| **Qdrant** (Python client) | Rust-based vector DB with rich filtering. REST and gRPC. | **Qdrant Java Client** |
| **Milvus** (`pymilvus`) | Distributed vector database for billions of vectors at enterprise scale. | **Milvus Java SDK** |
| **pgvector** | PostgreSQL extension adding vector types and IVF/HNSW indices. | **PostgreSQL JDBC + pgvector** |
| **OpenSearch** (Python client) | Open-source search with native k-NN vector search. | **OpenSearch Java Client** |
| **Redis + RedisSearch** | In-memory database with vector search extension. Sub-millisecond latency. | **Jedis (Redis Java Client)** |
| **MongoDB Atlas Search** | MongoDB native vector search on Apache Lucene. | **MongoDB Java Driver** |

---

### 9. Large Language Models (LLM) & Generative AI

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **OpenAI SDK** | Official SDK for GPT-4, GPT-4o, embeddings, and other OpenAI APIs. | **LangChain4j (OpenAI provider)** / **Spring AI** |
| **Anthropic SDK** | Official SDK for Claude models with extended context windows. | **LangChain4j (Anthropic provider)** |
| **Google Generative AI SDK** | Official SDK for Gemini models and Google embeddings. | **LangChain4j (Vertex AI provider)** |
| **Hugging Face Transformers** | Run any transformer/LLM from Hugging Face Hub locally or via API. | **DJL** / **LangChain4j (HF provider)** |
| **vLLM** | Production LLM serving engine — high throughput, PagedAttention. | **LangChain4j (OpenAI-compatible)** |
| **Ollama** (Python bindings) | Run LLMs locally with simple API. CPU and GPU support. | **Ollama Java SDK** / **LangChain4j (Ollama)** |
| **llama-cpp-python** | Python bindings for llama.cpp. Run quantized LLMs efficiently. | **LLaMA Java bindings** |

---

### 10. LLM Frameworks & RAG (Retrieval-Augmented Generation)

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **LangChain** | Industry-standard framework for LLM apps — chains, memory, RAG, agents. | **LangChain4j** |
| **LlamaIndex** | Data framework for LLM apps — document loading, indexing, retrieval for RAG. | **LangChain4j** / **Spring AI** |
| **Semantic Kernel** (Microsoft) | Framework combining LLMs with plugins and native code. | **Semantic Kernel Java** |
| **AutoGen** | Multi-agent conversational systems with LLMs. | **LangChain4j Agents** |
| **LiteLLM** | Unified interface for 100+ LLM providers with consistent API. | **LangChain4j** |
| **DSPy** | Modular LLM programming with automatic prompt optimization. | **LangChain4j** |

---

### 11. Search Platforms with Vector Support

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **Elasticsearch** (Python client) | Search platform with dense vector and semantic search. Powered by Apache Lucene. | **Elasticsearch Java API Client** |
| **Apache Solr** | Enterprise search platform with vector search on Lucene. | **Apache Solr Java Client** |
| **Vespa** | Yahoo's engine combining vector search, traditional search, and ML ranking at scale. | **Vespa (Java-native)** |

---

### 12. Data Visualization

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **Matplotlib** | Foundational plotting library — 2D charts, histograms, scatter plots. | **JFreeChart** |
| **Seaborn** | Statistical data visualization built on Matplotlib — heatmaps, distributions. | **Smile Visualization** |
| **Plotly** | Interactive web-based visualization — dashboards, 3D plots. | **Apache ECharts Java** |
| **Bokeh** | Interactive visualization for large datasets with server-side streaming. | **XChart** |
| **Altair** | Declarative visualization based on Vega/Vega-Lite. Grammar of graphics. | **XChart** |

---

### 13. MLOps, Experiment Tracking & Model Management

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **MLflow** | Industry standard for experiment tracking, model versioning, and model registry. | **MLflow Java API** |
| **Weights & Biases (W&B)** | Cloud-based experiment tracking, visualization, hyperparameter sweeps. | **W&B REST API** |
| **DVC** | Data Version Control — version data, models, and ML pipelines. | **DVC REST API** |
| **Apache Airflow** | DAG-based workflow orchestration and scheduling. | **Airflow REST API** / **Quartz Scheduler** |
| **Prefect** | Modern Python-first workflow orchestration. Dynamic DAGs, error handling. | **Spring Batch** |
| **Kubeflow Pipelines** | Kubernetes-native ML pipelines with container-based workflows. | **Kubernetes Java Client** |

---

### 14. Model Serving & Inference

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **ONNX Runtime** | Universal inference engine for models from any framework. Supports CPU and GPU. | **ONNX Runtime Java** |
| **TensorFlow Serving** | Production serving for TensorFlow models. Batching, versioning, REST/gRPC. | **TensorFlow Serving Java Client** |
| **TorchServe** | PyTorch's model serving framework. Production deployment of PyTorch models. | **DJL Serving** |
| **BentoML** | Package, containerize, and deploy ML models. Multi-model serving. | **Spring Boot + DJL** |
| **Triton Inference Server** | NVIDIA's multi-framework model serving with GPU support. | **Triton Java Client** |
| **Ray Serve** | Scalable ML model serving with Ray. Dynamic composition. | **Spring Boot** |

---

### 15. Distributed Computing & Big Data

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **Apache Spark** (PySpark) | Distributed computing — DataFrames, SQL, ML pipelines at scale. | **Apache Spark Java API** |
| **Dask** | Parallel arrays and dataframes. Scales Pandas and NumPy workflows. | **Apache Flink** |
| **Ray** | Distributed computing for ML — training, tuning, serving. | **Apache Flink** / **Storm** |
| **Modin** | Drop-in Pandas replacement with Dask/Ray parallelization. | **Spark Java API** |
| **Hadoop** | Distributed file system and MapReduce. Big data foundation. | **Apache Hadoop Java API** |

---

### 16. Statistical & Probability Programming

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **Statsmodels** | Statistical modeling — ARIMA, GLM, time series, hypothesis tests. | **Smile Statistics** / **Commons Math** |
| **PyMC** | Probabilistic programming — Bayesian modeling, MCMC sampling. | **Apache Commons Math** |
| **SciPy.stats** | Statistical distributions and tests. Part of SciPy. | **Commons Math** |
| **Prophet** | Facebook's forecasting tool — seasonality, trends, holidays. | **Smile Time Series** |

---

### 17. NLP-Specific Search & Indexing

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **BM25 (rank-bm25)** | Classic BM25 ranking algorithm for keyword-based search. | **Apache Lucene BM25** |
| **Whoosh** | Pure Python full-text indexing and search. | **Apache Lucene** |
| **Elasticsearch** (NLP) | Hybrid keyword + vector search. | **Elasticsearch Java API Client** |

---

### 18. Graph Neural Networks & Graph ML

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **PyTorch Geometric** | GNNs — GCN, GAT, GraphSAGE. Deep learning on graphs. | **JGraphT** |
| **DGL** | Framework-agnostic graph neural network library. | **DL4J + JGraphT** |
| **NetworkX** | Graph analysis and manipulation — algorithms, metrics, generators. | **JGraphT** / **JUNG** |

---

### 19. Reinforcement Learning

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **Stable Baselines3** | Reliable RL implementations — PPO, A2C, DQN, SAC. | **RL4J (DL4J module)** |
| **Ray RLlib** | Scalable distributed RL. Multi-agent, Kubernetes-native. | **RL4J** / **MASON** |
| **OpenAI Gym** | Standard environment interface for developing RL algorithms. | **RL4J environments** |

---

### 20. Feature Engineering & Feature Stores

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **Featuretools** | Automated feature engineering — deep feature synthesis. | **Spark MLlib Feature Engineering** |
| **Feast** | Open-source feature store — management and serving. | **Feast Java SDK** |
| **tsfresh** | Automatic time series feature extraction (100+ features). | **Smile** |

---

### 21. Data Quality & Validation

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **Great Expectations** | Data quality framework — validations, documentation, profiling. | **Apache Commons Validator** |
| **Pydantic** | Data validation using Python type hints. | **Bean Validation (JSR 380)** |
| **Pandera** | DataFrame schema validation using type hints. | **Tablesaw + Bean Validation** |
| **Evidently** | ML and data drift monitoring. | **Apache Flink (monitoring)** |

---

### 22. Model Compression & Quantization

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **ONNX Quantization** | Post-training quantization — Int8, dynamic, static. | **ONNX Runtime Java (quantized)** |
| **TensorFlow Lite** | Convert and quantize TensorFlow models for mobile/edge. | **TensorFlow Lite Java** |
| **PyTorch Quantization** | Built-in post-training and quantization-aware training. | **DJL** |
| **NNI** | AutoML and model compression — pruning, quantization, distillation. | **ONNX Runtime Java** |

---

### 23. Knowledge Graphs & Semantic Networks

| Python Library | Description | Java Equivalent |
|----------------|-------------|-----------------|
| **RDFlib** | RDF/OWL graph library with SPARQL support. | **Apache Jena** / **RDF4J** |
| **Neo4j Python Driver** | Python driver for Neo4j graph database. | **Neo4j Java Driver** |

---

## Quick Reference: Python → Java

| Category | Python | Java |
|----------|--------|------|
| Arrays | NumPy | ND4J |
| DataFrames | Pandas / Polars | Tablesaw / Smile |
| ML Algorithms | scikit-learn | Smile / Weka |
| Gradient Boosting | XGBoost / LightGBM / CatBoost | H2O / Spark MLlib |
| Deep Learning | TensorFlow / PyTorch | DL4J / DJL / TF Java API |
| Computer Vision | OpenCV / torchvision | OpenCV Java / DJL Vision |
| NLP | spaCy / NLTK / Transformers | Stanford CoreNLP / OpenNLP / DJL |
| Embeddings | Sentence-Transformers | LangChain4j / DJL |
| Vector Search | FAISS / Hnswlib / Voyager | JVector / Hnswlib Java / Voyager Java |
| Vector DB | Chroma / Weaviate / Qdrant / Pinecone / Milvus | Official Java clients |
| LLMs | OpenAI SDK / Anthropic SDK / HF Transformers | LangChain4j / Spring AI |
| RAG / Agents | LangChain / LlamaIndex | LangChain4j / Spring AI |
| Experiment Tracking | MLflow / W&B | MLflow Java API |
| Model Serving | ONNX Runtime / TorchServe | ONNX Runtime Java / DJL Serving |
| Distributed | Spark / Dask / Ray | Spark Java / Flink / Storm |
| AutoML | Optuna / H2O | H2O AutoML Java |
| Time Series | Statsmodels / Prophet | Smile / Commons Math |
| RL | Stable Baselines3 / RLlib | RL4J |
| Graph ML | PyTorch Geometric / DGL | JGraphT / DL4J |
| Knowledge Graphs | RDFlib / Neo4j | Jena / RDF4J / Neo4j Java |
