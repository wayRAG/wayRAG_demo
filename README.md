# wayRAG – Azure Native Phase 1 (RAG Foundation)

## 1. Purpose & Motivation

Phase 1 exists to answer one narrow but critical question:

> **Can we deliver a secure, enterprise‑acceptable internal document assistant on Azure using mostly managed services — without prematurely building a full knowledge management system?**

The goal is *credibility and correctness of plumbing*, not sophistication of knowledge governance.

Phase 1 intentionally focuses on:

* Secure access to internal documents
* High‑quality Retrieval‑Augmented Generation (RAG)
* Citations and traceability
* Azure‑native deployment patterns

It explicitly **does not** attempt to solve advanced knowledge management problems such as authority resolution, lifecycle enforcement, or conflict arbitration. Those are reserved for Phase 2.

---

## 2. Design Philosophy

### 2.1 Principle: Retrieval First, Governance Later

Most RAG systems fail not because governance is missing, but because:

* identity is weak
* access control is bolted on
* retrieval is unreliable

Phase 1 prioritizes **retrieval correctness and security**.

### 2.2 Principle: Managed Services Wherever Possible

Phase 1 prefers Azure managed services over custom implementations:

* Search → Azure AI Search
* Identity → Azure Entra ID
* Models → Azure OpenAI (via Foundry / AI Studio)
* Storage → Azure Blob Storage

Custom code is reserved for **orchestration and policy enforcement**.

### 2.3 Principle: Phase‑2‑Safe by Construction

Every Phase 1 decision must leave a clean seam for Phase 2. This means:

* no logic hidden in prompts
* no irreversible data modeling
* no business rules embedded inside search indexes

---

## 3. Phase 1 Scope (What We Build)

### 3.1 Capabilities Included

Phase 1 delivers:

* Azure‑native RAG system
* Document‑level access control
* Hybrid search (vector + keyword)
* Answer generation using Azure OpenAI
* Inline citations (document + page/chunk)
* Basic audit logging

### 3.2 Capabilities Explicitly Excluded

Phase 1 **does not** guarantee:

* authoritative answers
* latest‑version enforcement
* document conflict resolution
* lifecycle correctness (draft vs active vs deprecated)
* controlled non‑answers

These are intentional exclusions.

---

## 4. High‑Level Architecture (Phase 1)

```
User (Azure Entra ID)
   ↓
UI / Client
   ↓
Orchestrator API (Azure App Service / Container Apps)
   ↓
Access Control Filter (AAD Groups)
   ↓
Azure AI Search (Hybrid Retrieval)
   ↓
Azure OpenAI (Foundry)
   ↓
Answer + Citations
```

---

## 5. Azure Services Used

### 5.1 Azure OpenAI (via Azure AI Studio / Foundry)

Used for:

* Answer generation
* (Optional) reranking
* (Optional) query rewriting

**Design rule:**

* Models are stateless
* No business rules in prompts

---

### 5.2 Azure AI Search

Used as the **retrieval execution engine**.

Capabilities leveraged:

* Vector search (embeddings)
* BM25 keyword search
* Hybrid scoring
* Metadata filtering

Each indexed chunk contains:

* chunk text
* document identifier
* page number
* access group metadata

**Important constraint:**

> AI Search filters only *enforce decisions*. They do not *make decisions*.

---

### 5.3 Azure Blob Storage

Used for:

* Raw document storage
* Version‑agnostic file persistence

Documents are immutable once uploaded in Phase 1.

---

### 5.4 Azure Entra ID (AAD)

Used for:

* User authentication
* Group membership resolution

Group membership is the sole access control signal in Phase 1.

---

### 5.5 Orchestrator Service (Custom)

A lightweight Python service responsible for:

* Receiving user queries
* Resolving user identity and groups
* Applying access filters
* Calling Azure AI Search
* Calling Azure OpenAI
* Assembling responses with citations
* Writing audit logs

**This service is the long‑term backbone of the system.**

---

## 6. Phase 1 Query Flow

```
1. User sends query
2. User identity resolved via AAD
3. User groups extracted
4. Query sent to Orchestrator
5. Orchestrator applies access filter
6. AI Search retrieves allowed chunks
7. LLM generates answer
8. Citations attached
9. Response returned
```

---

## 7. Access Control Model (Phase 1)

### 7.1 Model

Access control is:

* document‑level
* group‑based
* static

Each document / chunk contains:

```
allowed_groups: ["Finance", "HR"]
```

At query time:

```
filter = allowed_groups intersects user_groups
```

### 7.2 Explicit Limitation

Phase 1 access control does **not** account for:

* document lifecycle
* authority precedence
* version selection

---

## 8. Citations Model

Each response includes:

* document name
* page number (or chunk reference)

Example:

> "The remote work policy allows 2 days per week [HR_Policy_v1.pdf, Page 4]"

Citations are generated from retrieval metadata, not hallucinated.

---

## 9. Logging & Observability

Phase 1 logs:

* user ID
* query text
* retrieved document IDs
* response latency

Logs are written to:

* Azure Application Insights
* Azure Log Analytics

No compliance claims are made in Phase 1.

---

## 10. Phase 1 Data Model (Intentionally Minimal)

### 10.1 Document Fields

```
doc_id
document_name
blob_uri
allowed_groups
chunk_id
page_number
embedding
```

No authority, lifecycle, or ownership fields are used yet.

---

## 11. Phase 2 Forward Compatibility (CRITICAL)

Phase 1 is designed to enable Phase 2 without re‑architecture.

### 11.1 What Phase 2 Will Add

Phase 2 introduces a **Metadata & Knowledge Governance Module** that:

* evaluates queries
* selects valid documents
* enforces authority, lifecycle, and freshness

### 11.2 Phase 2 Interaction Model

```
User Query
   ↓
Metadata Module (Decision)
   ↓
List of Allowed Doc IDs
   ↓
Azure AI Search (filter: doc_id IN (...))
   ↓
LLM
```

### 11.3 Phase 1 Design Constraints to Preserve

To enable Phase 2, Phase 1 must:

* keep doc_id stable
* avoid embedding business rules in search index
* avoid prompt‑level governance
* keep orchestration logic outside Foundry

Violating these makes Phase 2 infeasible.

---

## 12. What Phase 1 Is — and Is Not

**Phase 1 is:**

* an Azure‑native RAG foundation
* secure and demonstrable
* suitable for pilots and early adoption

**Phase 1 is not:**

* a knowledge management system
* a source‑of‑truth engine
* a compliance solution

Those claims belong to Phase 2.

---

## 13. Summary

Phase 1 deliberately optimizes for:

* speed of delivery
* Azure alignment
* architectural correctness

By constraining scope and preserving seams, it creates a stable foundation on which **real knowledge governance** can be layered in Phase 2 — without rewrites, migrations, or loss of credibility.

This is a foundation, not the destination.
# wayRAG_demo
To demonstrate the need and capabilities of goveranance for enterprise AI
