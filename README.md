
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
  
## Ingestion & Streaming:

- Raw arXiv metadata is validated at the ingestion boundary against an `ArxivPaperContract` **Pydantic** model (`paper_id`, `title`, `category`, `published_date`, `abstract`, `authors`), which rejects empty required fields and malformed dates.
- A local **Apache Kafka** broker (with Zookeeper) is stood up, and validated records are published to the `arxiv_raw_topic` topic by a **Kafka producer**.
- A **Kafka consumer** reads from that topic and routes any record that fails re-validation to a **Dead Letter Queue (DLQ)** topic, with the rejection reason recorded alongside the record.


## Quality Gates & Lineage:

- Every stage of the pipeline (ingestion, lakehouse, quality gate, RAG) is wrapped with a shared **OpenLineage** helper that emits **START / COMPLETE / FAIL** events to a single lineage log, giving a full audit trail of each run.
- A **Great Expectations** checkpoint (`paper_quality_suite`) enforces column-level expectations — for example, that `title` is never null and `abstract` meets a minimum length — before data is allowed to progress from Bronze into Silver. A failing checkpoint halts the pipeline rather than letting bad data flow downstream.


## Delta Lakehouse:

Built with **Delta Lake** (via `deltalake` / PyArrow) as three layers:

- **Bronze** — raw ingested data written with an enforced PyArrow schema (`paper_id`, `title`, `abstract`, `authors`, `category`, `published_date`); a write with a mismatched schema is demonstrably rejected.
- **Silver** — cleaned data merged into the Bronze table using a real Delta **MERGE (upsert)** keyed on `paper_id`, so re-ingesting a paper updates its existing row instead of duplicating it.
- **Gold** — genuine aggregates computed over Silver (e.g. paper counts and statistics by category), not a copy of the Silver table.

## AI & RAG Pipeline:

- **Chunking** — paper text is split into sentence-based, overlapping chunks (`chunk_size=2`, `overlap=1`).
- **Vector search** — chunks are embedded with `all-MiniLM-L6-v2` and stored in a **ChromaDB** collection.
- **Keyword search** — a parallel **BM25** (`rank_bm25`) index is built over the same chunks.
- **Hybrid search** — dense vector results and BM25 results are fused with **Reciprocal Rank Fusion (RRF)**.
- **Reranking** — the fused candidates are reranked with a **cross-encoder** (`cross-encoder/ms-marco-MiniLM-L-6-v2`) for relevance.
- **Grounded generation** — the top reranked chunks are assembled into a source-numbered context and passed to **LLaMA 3.1 (8B Instant) via the Groq API**, with a prompt that requires the model to answer only from the provided sources, cite claims by source number, and explicitly say when the sources don't support an answer.


## Prerequisites & Installation:

- Python 3.10+
- A [Kaggle](https://www.kaggle.com) account (for `kagglehub` dataset download) with API credentials configured
- A [Groq](https://console.groq.com) API key
- Java (required by local Kafka/Zookeeper)

Install all required libraries:

```bash
pip install -q kafka-python deltalake pyarrow great_expectations openlineage-python \
  chromadb sentence-transformers rank-bm25 numpy groq \
  apache-airflow apache-airflow-providers-standard kagglehub
```

The notebook also downloads and runs a local Kafka broker directly:

```bash
wget -q https://archive.apache.org/dist/kafka/3.6.1/kafka_2.12-3.6.1.tgz
tar -xzf kafka_2.12-3.6.1.tgz
./kafka_2.12-3.6.1/bin/zookeeper-server-start.sh -daemon ./kafka_2.12-3.6.1/config/zookeeper.properties
./kafka_2.12-3.6.1/bin/kafka-server-start.sh -daemon ./kafka_2.12-3.6.1/config/server.properties
```

---

## Configuration & Secrets:

The pipeline requires one secret: **`GROQ_API_KEY`**, used to authenticate with the Groq API for RAG answer generation.

**In Google Colab** (as used in this notebook), add it via Colab Secrets and it is loaded with:

```python
import os
from google.colab import userdata
os.environ["GROQ_API_KEY"] = userdata.get("GROQ_API_KEY")
```

**Running locally**, use a `.env` file instead (never commit this file — see `.gitignore`):

```
GROQ_API_KEY=your_groq_api_key_here
```

```python
import os
from dotenv import load_dotenv
load_dotenv()
os.environ["GROQ_API_KEY"] = os.getenv("GROQ_API_KEY")
```

Kaggle API credentials for `kagglehub` should similarly be kept out of version control (typically `~/.kaggle/kaggle.json` or Colab Secrets), never hard-coded in the notebook.

---

## How to Run:

1. **Open the notebook** (Google Colab recommended, since it uses `google.colab.userdata` for secrets).
2. **Install dependencies** — run the setup/pip-install cell first.
3. **Set your secrets** — add `GROQ_API_KEY` (and Kaggle credentials) via Colab Secrets, or a local `.env` file if running outside Colab.
4. **Run Section 1 (Setup)** to initialize the shared OpenLineage wrapper — every later stage depends on it.
5. **Run Section 2 (Ingestion)** in order: download/extract the arXiv sample → validate with the Pydantic contract → start the local Kafka broker → run the producer → run the consumer (malformed records land in the DLQ).
6. **Run Section 3 (Delta Lakehouse)**: build Bronze, run the Great Expectations quality gate, build Silver with the `paper_id` MERGE, then build Gold. Each "Proof" cell demonstrates a failure path (bad schema write, bad data rejected by the gate).
7. **Run Section 4 (Orchestration)** to define and test the `arxiv_research_pipeline` Airflow DAG, which chains every stage above with `t1 >> t2 >> ... >> t7` and halts downstream tasks if `quality_gate` fails.
8. **Run Section 5 (RAG Pipeline)**: chunk the papers, build the ChromaDB vector index and BM25 index, then run hybrid search + reranking.
9. **Test the RAG pipeline** by calling `generate_rag_answer_groq("<your question>")` (or the lineage-tracked wrapper shown in the notebook) and reviewing the cited, grounded answer along with its `Sources` section.

---

## References:

- [SDAIA Academy on GitHub](https://github.com/SDAIAAcademy)
- Dataset: [Cornell University arXiv Dataset (Kaggle)](https://www.kaggle.com/datasets/Cornell-University/arxiv)

---

