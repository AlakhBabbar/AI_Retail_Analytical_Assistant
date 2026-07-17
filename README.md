# AI Retail Analytics Assistant

An AI-powered retail analytics assistant that answers natural language business questions by combining **SQL generation**, **Retrieval-Augmented Generation (RAG)**, and **LLM-based response synthesis**.

Instead of asking users to write SQL queries, the assistant converts business questions into SQL, retrieves relevant business knowledge when required, executes the query on a structured retail dataset, and produces grounded analytical responses.

---

## Features

- Natural language business query interface
- Dynamic SQL generation using an LLM
- Automatic schema inspection (no hardcoded schema)
- SQLite-backed analytical engine
- Retrieval-Augmented Generation (RAG) using ChromaDB
- Business knowledge base for KPI definitions and aggregation rules
- Multi-stage planning pipeline
- JSON-based communication between pipeline components
- Open-source LLM through Hugging Face Inference API

---

## Architecture

The assistant follows a modular pipeline rather than a single prompt.

```
                    User Query
                         │
                         ▼
                 Query Planner
                         │
         ┌───────────────┴───────────────┐
         │                               │
         ▼                               ▼
   SQL Pipeline                    RAG Pipeline
         │                               │
         ▼                               ▼
    SQL Result                 Retrieved Knowledge
         └───────────────┬───────────────┘
                         ▼
               Response Generator
                         │
                         ▼
               Final Business Answer
```

---

## Project Structure

```
AI-Retail-Analytics/
│
├── notebook.ipynb              # Complete implementation
├── sales_data.csv              # Retail analytics dataset
├── knowledge.md                # Business knowledge base
├── requirements.txt
├── README.md
├── .env
└── .gitignore
```

---

## Technology Stack

| Component | Technology |
|------------|------------|
| Language | Python |
| Environment | Jupyter Notebook |
| Database | SQLite |
| Data Processing | Pandas, NumPy |
| Embeddings | Sentence Transformers (all-MiniLM-L6-v2) |
| Vector Database | ChromaDB |
| LLM | Qwen2.5-7B-Instruct |
| Inference | Hugging Face Inference API |
| Configuration | python-dotenv |

---

## Dataset

The analytical dataset consists of approximately **12,000 retail sales records** containing information such as:

- Region
- State
- City
- Product
- Category
- Brand
- Promotion
- Revenue
- Units Sold
- Discount Amount
- Inventory Before
- Inventory After
- Customer Rating

The CSV is automatically imported into SQLite during notebook initialization.

---

## Knowledge Base

A separate markdown knowledge base (`knowledge.md`) stores:

- Retail KPIs
- Promotion Lift
- Baseline Revenue
- Inventory concepts
- Aggregation rules
- Business terminology

The knowledge base is embedded once during startup and stored in ChromaDB for semantic retrieval.

---

## Pipeline Components

### 1. Query Planner

Determines:

- Whether SQL is required
- Whether RAG is required
- SQL task
- Retrieval task
- Final answer objective

Example output:

```json
{
    "needs_sql": true,
    "needs_rag": true,
    "sql_task": "...",
    "rag_task": "...",
    "answer_goal": "..."
}
```

---

### 2. SQL Generator

Uses:

- User task
- Dynamic database schema
- Business aggregation rules

Returns:

```json
{
    "sql":"SELECT ...",
    "summary":"..."
}
```

---

### 3. SQL Executor

- Executes generated SQL
- Returns Pandas DataFrame
- Converts output into markdown table

---

### 4. Knowledge Retriever

- Embeds retrieval query
- Performs ChromaDB similarity search
- Returns top-k business knowledge chunks

---

### 5. Response Generator

Combines:

- User question
- Planner objective
- SQL results
- Retrieved knowledge

Produces the final business response.

---

## Dynamic Schema Inspection

Instead of manually writing the database schema inside prompts, the assistant inspects SQLite during startup.

It automatically retrieves:

- Tables
- Columns
- Data types
- Sample values

The discovered schema is injected into the SQL generation prompt, making the system adaptable to similar datasets without modifying prompt templates.

---

## Running the Project

### 1. Clone the repository

```bash
git clone <repository-url>
cd AI-Retail-Analytics
```

---

### 2. Create virtual environment

Windows

```bash
python -m venv venv
```

Activate

```bash
venv\Scripts\activate
```

Linux / macOS

```bash
python3 -m venv venv
source venv/bin/activate
```

---

### 3. Install dependencies

```bash
pip install -r requirements.txt
```

---

### 4. Configure Hugging Face API

Create a `.env` file.

```env
HF_API_KEY=your_huggingface_api_key
HF_API_KEY2=your_huggingface_api_key2
```

---

### 5. Launch Jupyter

```bash
jupyter notebook
```

Open

```
notebook.ipynb
```

Run all cells.

---

## Example Questions

### SQL

- Which region generated the highest revenue?
- Show top 10 products by sales.
- Which category sold the most units?

### RAG

- What is Promotion Lift?
- Explain Baseline Revenue.
- What does Inventory Turnover mean?

### Hybrid

- Did promotions improve sales in the South region?
- Explain why the North region performed better than the East.
- Compare revenue and interpret the results using Promotion Lift.

---

## Design Decisions

- Dynamic schema inspection instead of hardcoded schema
- SQLite instead of Pandas filtering
- Separate planner from SQL generation
- Separate computation (SQL) from explanation (RAG)
- Open-source LLM instead of proprietary APIs
- Low-temperature inference for deterministic analytical outputs

---

## Current Limitations

- Planner generates only one SQL task and one retrieval task.
- Only a single SQL query is produced for each request.
- No SQL validation before execution.
- No automatic SQL self-correction.
- No multi-step reasoning or replanning.
- Knowledge is limited to the contents of `knowledge.md`.
- Designed specifically for structured retail analytics datasets.

---

## Future Improvements

- Multi-step planning
- SQL self-repair
- Query verification
- Conversation memory
- Interactive dashboards
- Automatic visualization generation
- Confidence estimation
- Support for multiple datasets

---

## License

This project was developed as part of an AI Engineering assessment and is intended for educational and demonstration purposes.