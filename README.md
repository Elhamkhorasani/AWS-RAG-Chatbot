# SAP BW → AWS RAG Chatbot

**Vector Search + Bedrock LLM grounded on BW Transformation Metadata**

---

## 1. Project Overview

This project builds an **end-to-end Retrieval-Augmented Generation (RAG) chatbot** that answers questions about **SAP BW transformation metadata** using:

* **AWS S3** – data storage
* **Amazon Bedrock (Embeddings + LLM)** – semantic understanding and answer generation
* **OpenSearch Serverless (k-NN vector index)** – semantic retrieval
* **SageMaker Notebook** – orchestration and experimentation

The chatbot provides **bilingual grounded answers (German/English)** based **only on the uploaded BW data**.

---

## 2. Architecture

```
SAP BW Export (CSV)
        ↓
     AWS S3
        ↓
  Bedrock Embeddings (1024-dim)
        ↓
OpenSearch Serverless (Vector Index)
        ↓
   k-NN Semantic Retrieval
        ↓
 Bedrock Chat Model (ON_DEMAND)
        ↓
  Grounded DE/EN Answer
```

This implements a **classical enterprise RAG pipeline**.

---

## 3. Dataset

The dataset contains **SAP BW transformation metadata**, including:

* Transformation ID
* Source object
* Target object
* Field mappings
* Owner
* Rule parameters

Each row is converted into a **natural-language document string (`doc_text`)** before embedding.

Example:

```
Transformation_ID=0DE3KLEV...
Source_Object=0BPARTNER_TEXT
Target_Object=ZBPARTNER
Field1=PARTNER
Field2=PARTNER
```

---

## 4. Embedding Generation

* Model: **Amazon Titan Text Embeddings**
* Vector dimension: **1024**
* Output stored together with:

  * `doc_text`
  * metadata fields

This enables **semantic similarity search** in OpenSearch.

---

## 5. Vector Database (OpenSearch Serverless)

### Index configuration

* Type: **k-NN vector index**
* Dimension: **1024**
* Fields:

  * `vector` → knn_vector
  * `doc_text` → text
  * `metadata` → object

### Result

Documents successfully indexed:

```
status: 200
"errors": false
"result": "created"
```

This confirms **correct ingestion**.

---

## 6. Semantic Retrieval

User questions are:

1. Embedded with the same **1024-dim embedding model**
2. Queried via **k-NN search**
3. Top-k relevant BW rows returned as **context**

Example query:

```
Which transformation maps PARTNER to ZBPARTNER?
```

OpenSearch returns the relevant transformation rows.

---

## 7. LLM Answer Generation (Bedrock)

Because some Bedrock models require **Inference Profiles**, the project uses an **ON_DEMAND chat model**:

* Example: `qwen.qwen3-235b-a22b-2507-v1:0`

The LLM is constrained with **strict grounding rules**:

* Use **only retrieved context**
* If information is missing → respond:

```
DE: Nicht in den Daten.
EN: Not in the data.
```

---

## 8. Example Result

**DE:**
Die Transformation mit der ID
`0DE3KLEVQQUL3YCP1FJRNDLOBZW530NL`
bildet **PARTNER → ZBPARTNER** ab.
Beteiligte Felder: **PARTNER, ZBPARTNER**.

**EN:**
The transformation with ID
`0DE3KLEVQQUL3YCP1FJRNDLOBZW530NL`
maps **PARTNER → ZBPARTNER**.
Fields involved: **PARTNER, ZBPARTNER**.

This proves:

* Correct **vector retrieval**
* Correct **grounded LLM reasoning**
* Successful **end-to-end RAG pipeline**

---

## 9. Key Learnings

### Technical

* Vector dimension must match **index mapping**
* Bedrock **ON_DEMAND vs Inference Profile** affects model usability
* OpenSearch `_bulk` can return **HTTP 200 with partial failures**
* Grounded prompts are essential for **enterprise trust**

### Architectural

This solution demonstrates a **production-grade enterprise AI pattern**:

* Secure cloud ingestion
* Semantic enterprise search
* LLM-powered knowledge access

---

## 10. Possible Extensions

* Aggregate **all field mappings per Transformation_ID**
* Build **Streamlit or web UI chatbot**
* Add **multi-file BW metadata ingestion**
* Implement **Bedrock Knowledge Bases comparison**
* Add **evaluation metrics for grounded accuracy**

---

## 11. Conclusion

The project successfully delivers a **fully functional SAP BW RAG chatbot on AWS** that:

* Understands BW metadata semantically
* Retrieves relevant transformations via vector search
* Produces **bilingual, grounded answers**
* Follows **enterprise AI architecture best practices**

This confirms the feasibility of applying **LLM-driven knowledge access** to **SAP BW transformation analysis** in a secure AWS environment.

---
