# 📘 Workshop: Knowledge Graphs for Retrieval-Augmented Generation (RAG)

This workshop introduces how to combine **Knowledge Graphs (KGs)** with **LLMs** for powerful question answering.
We’ll go from **raw documents** to a **graph-powered RAG system**, step by step.

---

## 🔹 Goals

* Understand the difference between **chunk-based retrieval** and **entity-based graphs**, and why hybrid approaches matter.
* Build a **Neo4j knowledge graph** from documents.
* Add **embeddings** for semantic retrieval.
* Use **entity extraction** to enrich the graph with structured facts.
* Query the graph using **Cypher** and **LLM-assisted query generation**.
* Enable **chat with the knowledge graph** using LangChain.

---

## 🔹 Prerequisites

* Python 3.10+
* [Neo4j Desktop or AuraDB](https://neo4j.com/cloud/platform/aura-graph-database/)
* An OpenAI API key (or alternative LLM provider supported by LangChain)
* Install dependencies:

  ```bash
  pip install -r requirements.txt
  ```

Create a `.env` file with:

```env
NEO4J_URI=bolt://localhost:7687
NEO4J_USERNAME=neo4j
NEO4J_PASSWORD=yourpassword
NEO4J_DATABASE=neo4j

OPENAI_API_KEY=sk-xxxx
```

---

## 🔹 Workshop Structure

We’ll follow the lifecycle of building a KG for RAG:

1. **L2 – Query with Cypher**

   * Learn Cypher basics to explore graphs manually.
   * Example: *Find all managers in California.*

2. **L3 – Prepare Text for RAG**

   * Split documents into **chunks**.
   * Add metadata (company, form ID, etc.).
   * Store chunks in Neo4j with embeddings.

3. **L4 – Construct KG from Text**

   * Run **entity & relationship extraction** with LLMs.
   * Create nodes for `Company`, `Manager`, `Address`, and connect them to chunks.

4. **L5 – Add Relationships**

   * Expand the graph with explicit edges (`OWNS_STOCK_IN`, `LOCATED_AT`).
   * Link entities back to supporting chunks.

5. **L6 – Expand the KG**

   * Load new documents.
   * Deduplicate entities across forms (e.g., same company in multiple filings).
   * Handle name variations.

6. **L7 – Chat with the KG**

   * Use LangChain’s `GraphCypherQAChain` to let an LLM **generate Cypher queries** from natural language.
   * Example: *“What does Palo Alto Networks do?” → Cypher query → Retrieve from graph*.

---

## 🔹 Example Workflow

```python
from langchain_community.graphs import Neo4jGraph
from langchain.chains import GraphCypherQAChain
from langchain_openai import ChatOpenAI

kg = Neo4jGraph(
    url=NEO4J_URI,
    username=NEO4J_USERNAME,
    password=NEO4J_PASSWORD,
    database=NEO4J_DATABASE
)

cypher_chain = GraphCypherQAChain.from_llm(
    llm=ChatOpenAI(model="gpt-4o-mini", temperature=0),
    graph=kg,
    allow_dangerous_requests=True
)

cypher_chain.run("What companies are in Santa Clara?")
```

---

## 🔹 Notes

* We use a **hybrid KG design**:

  * `Chunk` nodes with embeddings for semantic RAG.
  * `Entity` nodes (Company, Manager, Address) for structured queries.
* This redundancy is intentional – it gives **precision + recall + provenance**.
* You can experiment with **two LLMs**:

  * A powerful one for **Cypher generation**.
  * A smaller one for **answer synthesis**.

---

## 🔹 Next Steps

* Try extending with your own dataset (`.json`, `.csv`).
* Explore LangSmith for tracing & debugging queries.
* Add evaluation: measure if KG improves answer **precision & faithfulness** over plain RAG.

