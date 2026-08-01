# Research Atlas:

An end-to-end AI platform that transforms scientific publications into a searchable research atlas using modern data engineering and Retrieval-Augmented Generation (RAG).

## Program:

Developed as part of the "Modern Data Engineering for AI Systems" program by SDAIA Academy (DAICO).

*Trainer:* Mohammed Albeladi

*Session Dates:* July 19th 2026 - July 23rd 2026


## Team:

- Sarah Abdulaziz Alkhudhiri
- Rehaf Alfaleh
- Basma Alhindi

## Overview:

Research Atlas continuously ingests scientific papers, validates incoming data, organizes research within a Delta Lakehouse, and enables intelligent exploration through semantic search and citation-backed AI responses. Users can interactively ask their own questions about the research corpus and receive grounded, cited answers in return.

## Features:

- Real-time data ingestion with Apache Kafka
- Pydantic schema validation and Dead Letter Queue (DLQ)
- Delta Lakehouse (Bronze, Silver, Gold)
- Data quality validation with Great Expectations
- Embeddings and vector database
- Hybrid Search (BM25 + Vector Search)
- Cross-Encoder reranking
- Retrieval-Augmented Generation (RAG)
- Interactive question input for the RAG pipeline
- Apache Airflow orchestration

## Tech Stack:

- Python
- Apache Kafka
- Delta Lake
- Great Expectations
- Apache Airflow
- Pydantic (data contracts / schema validation)
- OpenLineage (pipeline lineage tracking)
- ChromaDB (vector store)
- Sentence-Transformers (embeddings + cross-encoder reranking)
- BM25 / rank_bm25 (keyword search)
- Groq API (LLaMA 3.1 — RAG answer generation)
- PyArrow (Delta Lake schema enforcement)
- Pandas

## Pipeline:
