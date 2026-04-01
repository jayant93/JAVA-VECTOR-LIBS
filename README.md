# JAVA-VECTOR-LIBS

A Java library project providing support and implementations for vector search, embeddings, and similarity search — mirroring the functionality of popular Python vector libraries used in real-world AI/ML applications.

---

## Python Vector Libraries (Real-World Usage)

The following Python libraries are widely used in production for vector search, embeddings, and similarity search:

### Core Vector Search

| Library | Description |
|---------|-------------|
| **FAISS** | Meta AI's industry-standard library for efficient similarity search and clustering of dense vectors. GPU-accelerated. |
| **Hnswlib** | Fast approximate nearest neighbor search using Hierarchical Navigable Small World (HNSW) graphs. Lightweight and local. |
| **Annoy** | Spotify's library for approximate nearest neighbor search. Static index files, memory-mapped for fast serving. |
| **Voyager** | Spotify's successor to Annoy. HNSW-based, production-ready, with both Python and Java bindings. |
| **ScaNN** | Google's Scalable Nearest Neighbors library. Optimized for large-scale cloud deployments. |

### Vector Database Clients & Embedded Databases

| Library | Description |
|---------|-------------|
| **Chroma** | Open-source embedded vector database for LLM applications. Handles text-to-embedding conversion and similarity search. |
| **LanceDB** | Embedded vector database built on Lance columnar format. Zero-copy, runs in-process, ideal for local workloads. |
| **Weaviate** (Python client) | Client for Weaviate open-source vector database. Built-in embedding generation and hybrid search. |
| **Pinecone** (Python SDK) | Client for Pinecone fully managed vector database service. |
| **Qdrant** (Python client) | Client for Qdrant vector database (written in Rust). REST and gRPC interfaces, rich filtering. |
| **Milvus** (`pymilvus`) | Client for Milvus distributed vector database. Supports billions of vectors at enterprise scale. |
| **pgvector** | PostgreSQL extension adding vector data types and IVF/HNSW index support directly in SQL. |
| **OpenSearch** (Python client) | Client for OpenSearch with native k-NN vector search via the k-NN plugin. |

### Embedding Generation

| Library | Description |
|---------|-------------|
| **Sentence-Transformers** | Generates dense vector embeddings from text using transformer models. Standard for semantic similarity and retrieval. |
| **Transformers** (Hugging Face) | Core library for working with pre-trained transformer and embedding models. |
| **Gensim** | Word2Vec, FastText, and Doc2Vec embedding algorithms. Classical NLP embedding toolkit. |
| **spaCy** | Industrial-strength NLP library with built-in word vectors and deep learning support. |

---

## Java Vector Libraries (Implemented / Planned in This Repo)

This repository provides Java implementations and integrations for the following vector search and embedding libraries:

### Core Vector Search Engines

| Library | Description |
|---------|-------------|
| **JVector** | Pure Java, zero-dependency embedded vector search engine. Based on DiskANN algorithm. Used by DataStax Astra DB and Apache Cassandra. Apache 2.0 licensed. |
| **Apache Lucene** | Foundation of Elasticsearch, Solr, OpenSearch, and MongoDB Atlas Search. Dense vector fields with HNSW algorithm, bulk scoring, and early termination optimizations. |
| **Hnswlib Java** (`jelmerk/hnswlib`) | Pure Java implementation of the HNSW algorithm for approximate nearest neighbor search. |
| **Hnswlib-JNA** (`stepstone-tech/hnswlib-jna`) | JNA bindings for hnswlib offering native-like performance with pre-built libraries for Windows, Linux, and macOS. |
| **Voyager** (Java bindings) | Spotify's production-ready nearest-neighbor search library with official Java bindings. Successor to Annoy. |

### Vector Database Java Clients

| Library | Description |
|---------|-------------|
| **Weaviate Java Client** | Official Java client for Weaviate. Collection creation, data import, and vector search queries. |
| **Elasticsearch Java API Client** | Official Java client for Elasticsearch with vector/semantic search support. |
| **OpenSearch Java Client** | Official Java client for OpenSearch. Native k-NN vector search with GPU acceleration (OpenSearch 3.0+). |
| **Qdrant Java Client** | Official Java client for Qdrant. REST and gRPC interfaces for vector search. |
| **Milvus Java SDK** | Official Java SDK for Milvus distributed vector database. Enterprise-scale vector storage and retrieval. |
| **Pinecone Java SDK** | Official Java SDK for Pinecone managed vector database service. |

### Search Platforms with Vector Support

| Library | Description |
|---------|-------------|
| **Vespa** | Yahoo's open-source data serving engine combining vector search, traditional search, and ML ranking at massive scale. |
| **Apache Solr** | Enterprise search platform built on Lucene with vector search capabilities. |

### ML & Embedding Integration

| Library | Description |
|---------|-------------|
| **DJL** (Deep Java Library) | Engine-agnostic Java deep learning framework for embedding generation, tokenization, and model inference. |
| **LangChain4j** | Java library for LLM integration. Supports vector embeddings, semantic search, and RAG pipelines. |

---

## Goal

This project aims to provide Java developers with first-class access to the same vector search and embedding capabilities that the Python ecosystem offers — enabling production-grade AI/ML applications in Java.
