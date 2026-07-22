# Research Atlas:

An end-to-end AI platform that transforms scientific publications into a searchable research atlas using modern data engineering and Retrieval-Augmented Generation (RAG).

## Overview:

Research Atlas continuously ingests scientific papers, validates incoming data, organizes research within a Delta Lakehouse, and enables intelligent exploration through semantic search and citation-backed AI responses.

## Features:

- Real-time data ingestion with Apache Kafka
- Pydantic schema validation and Dead Letter Queue (DLQ)
- Delta Lakehouse (Bronze, Silver, Gold)
- Data quality validation with Great Expectations
- Embeddings and vector database
- Hybrid Search (BM25 + Vector Search)
- Cross-Encoder reranking
- Retrieval-Augmented Generation (RAG)
- Apache Airflow orchestration

## Tech Stack:

- Python
- Apache Kafka
- Apache Spark
- Delta Lake
- Great Expectations
- Apache Airflow

## Pipeline:

```
Dataset
   ↓
Kafka
   ↓
Validation
   ↓
Bronze → Silver → Gold
   ↓
Embeddings
   ↓
Vector Database
   ↓
Hybrid Search
   ↓
RAG
```

## Example Questions:

- Which authors publish most frequently in computer vision?
- Which papers discuss reinforcement learning?
- Find recent lightweight LLM research.


## Training Program:

Developed as part of the "Modern Data Engineering for AI Systems" program by Sdaia Academy.

https://github.com/SDAIAAcademy

## Team:

- Sarah Abdulaziz Alkhudhiri
- Rehaf Alfaleh
- Basma Alhindi
