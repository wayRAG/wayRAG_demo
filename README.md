# wayRAG Cloud - Phase 1

A cloud-native migration of wayRAG_Local to Azure using Foundry IQ for intelligent document querying with document-level access control and citation-based responses.

## Overview

wayRAG Cloud maintains all capabilities of wayRAG_Local while leveraging Azure's managed services for scalability, security, and enterprise deployment. Phase 1 focuses on core RAG functionality with RBAC and citations, setting the foundation for advanced orchestration in future phases.

## Motivation

**Why Cloud-Native?**
- Deploy to client Azure subscriptions with isolated environments
- Leverage enterprise-grade security (Azure AD, RBAC, encryption)
- Scale automatically based on query volume
- Reduce infrastructure management overhead
- Enable multi-region deployment for global clients

**Why Foundry IQ?**
- Handles hybrid search (BM25 + vector) natively - replaces custom retrieval logic
- Built-in semantic reranking - eliminates manual reranker implementation
- Multi-source RAG capability - connects to Blob, SharePoint, OneDrive simultaneously
- Automatic citation extraction - eliminates custom parsing logic
- Query planning and decomposition - handles complex queries intelligently
- Document-level ACL support - native permission inheritance from sources

## Design Philosophy

### Principle 1: Retrieval First, Governance Later

Phase 1 answers one critical question:

> **Can we deliver a secure, enterprise-acceptable internal document assistant on Azure using managed services — without prematurely building a full knowledge management system?**

The goal is **credibility and correctness of plumbing**, not sophistication of knowledge governance.

Most RAG systems fail not because governance is missing, but because:
- Identity is weak
- Access control is bolted on
- Retrieval is unreliable

**Phase 1 prioritizes retrieval correctness and security.**

### Principle 2: Managed Services Wherever Possible

Phase 1 prefers Azure managed services over custom implementations:
- Knowledge & Retrieval → **Azure AI Foundry IQ**
- Identity → Azure Entra ID
- Models → Azure OpenAI
- Storage → Azure Blob Storage

Foundry IQ is used to avoid reinventing:
- Data-source connectors
- Indexing pipelines
- Hybrid (vector + keyword) retrieval
- Security trimming at source

**Custom code is reserved for orchestration and policy enforcement.**

### Principle 3: Phase-2-Safe by Construction

Every Phase 1 decision must leave a clean seam for Phase 2. This means:

**Critical Design Constraints:**
- ✅ **No logic hidden in prompts** - Business rules must live in orchestration code, not system prompts
- ✅ **No irreversible data modeling** - doc_id must remain stable across phases
- ✅ **No business rules embedded inside search indexes** - Foundry IQ retrieves, doesn't decide what's authoritative
- ✅ **Keep orchestration logic outside Foundry** - The orchestrator is the long-term backbone, not Foundry IQ

**What must remain stable for Phase 2:**
1. `doc_id` stability - IDs cannot change when adding metadata
2. Orchestration outside Foundry - Governance decisions happen in our code
3. Separation of retrieval and policy - Foundry retrieves what user can see, orchestrator decides what's correct

**Violating these constraints makes Phase 2 infeasible.**

## Phase 1 Goals

### What Phase 1 Delivers

| Feature | Local Implementation | Cloud Implementation |
|---------|---------------------|---------------------|
| **Access Control** | CSV-based user/group filtering | Azure AD + Foundry IQ document-level ACL |
| **Retrieval** | Custom BM25 + Semantic search | Foundry IQ hybrid retrieval |
| **Reranking** | LLM-based relevance scoring | Foundry IQ semantic reranking |
| **Citations** | Custom extraction with page numbers | Foundry IQ automatic citations |
| **Data Sources** | Local files | Azure Blob, SharePoint, OneDrive |
| **User Auth** | CSV user_map.csv | Azure AD authentication |
| **Metadata** | CSV metadata.csv | Azure Cosmos DB (future filtering) |
| **UI** | Gradio interface | React web app (Azure Static Web Apps) |
| **API** | Python local server | Azure Functions serverless API |
| **Monitoring** | Local logs | Azure Application Insights |

### What Phase 1 INCLUDES ✅

1. **Core RAG Pipeline**
   - Query → Foundry IQ retrieval → Azure OpenAI generation → Response with citations
   - Document-level access control (user sees only authorized documents)
   - Citation-based responses with source attribution

2. **Multi-Source Connectivity**
   - Azure Blob Storage (primary document repository)
   - SharePoint Online (optional)
   - OneDrive for Business (optional)

3. **User Interface**
   - React chat interface
   - Citation display with source links
   - Session management

4. **Backend API**
   - Azure Functions (Python) REST API
   - `/api/query` - main query endpoint
   - `/api/health` - health check

5. **Infrastructure**
   - Infrastructure as Code (Bicep templates)
   - Managed Identity (zero secrets architecture)
   - Cosmos DB for query logs and metadata
   - Application Insights for monitoring

6. **Deployment**
   - One-click client deployment script
   - Client-specific configuration
   - Sample data setup

### What Phase 1 EXCLUDES ❌

Phase 1 provides a **working demo** for clients. Advanced orchestration comes in later phases.

**Capabilities Explicitly Excluded:**
- ❌ Content moderation (Phase 2)
- ❌ Query classification/decomposition (Phase 2)
- ❌ Claim validation (Phase 2)
- ❌ Contextual query enhancement (Phase 2)
- ❌ Conflict resolution (Phase 2)
- ❌ Multi-turn conversation memory (Phase 3)
- ❌ Advanced metadata filtering UI (Phase 3)
- ❌ Custom reranking logic (Phase 3 if needed)

**Important: Phase 1 does NOT guarantee:**
- ❌ Authoritative answers (no doc_authority enforcement)
- ❌ Latest-version enforcement (no version selection)
- ❌ Document conflict resolution (no authority precedence)
- ❌ Lifecycle correctness (draft vs active vs deprecated not enforced)
- ❌ Controlled non-answers (may retrieve deprecated or conflicting docs)

**These are intentional exclusions.** Phase 1 focuses on retrieval correctness and security, not knowledge governance.

## Phase Roadmap

### Phase 1: Core RAG + RBAC (Current)
**Goal**: Working demo with citations and user-level access control

**Capabilities**:
- Query with citations
- Document-level RBAC
- Multi-source data ingestion
- Basic metadata storage (provision for future filtering)

**Timeline**: 4-6 weeks

---

### Phase 2: Orchestration Layer
**Goal**: Add intelligent query processing pipeline

**Capabilities**:
- Content moderation
- Query classification (FACT/OPINION/CLAIM)
- Query decomposition for complex queries
- Claim validation
- Contextual query enhancement
- Routing engine

**Timeline**: 4-6 weeks

---

### Phase 3: Advanced Features
**Goal**: Conversation memory and metadata-based filtering

**Capabilities**:
- Multi-turn conversation with context
- Metadata filter UI (doc_authority, stage, valid_from/till)
- Conflict resolution with authority-based ranking
- Enhanced analytics

**Timeline**: 4-6 weeks

---

### Phase 4: Enterprise Integration
**Goal**: Production hardening and client-specific integrations

**Capabilities**:
- Custom connectors for client systems
- Advanced compliance features
- Performance optimization
- White-labeling

**Timeline**: Ongoing

## Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      USER LAYER                              │
│  ┌────────────────────────────────────────────────────┐     │
│  │  React Web App (Azure Static Web Apps)            │     │
│  │  • Chat interface                                  │     │
│  │  • Citation display                                │     │
│  │  • Session management                              │     │
│  └───────────────────┬────────────────────────────────┘     │
└─────────────────────┼──────────────────────────────────────┘
                       │ HTTPS
┌──────────────────────┼──────────────────────────────────────┐
│              ORCHESTRATOR LAYER                              │
│               (Long-term backbone)                           │
│  ┌────────────────────▼─────────────────────────────────┐   │
│  │  Azure Functions (Python 3.11)                       │   │
│  │  wayRAG Orchestrator - Custom Logic                  │   │
│  │                                                       │   │
│  │  POST /api/query                                     │   │
│  │   1. Authenticate user (Azure AD)                    │   │
│  │   2. Resolve user groups                             │   │
│  │   3. Apply access control filters                    │   │
│  │   4. Query Foundry IQ KB with user context          │   │
│  │   5. Generate response (Azure OpenAI)                │   │
│  │   6. Assemble citations                              │   │
│  │   7. Log to Cosmos DB                                │   │
│  │   8. Return response                                 │   │
│  │                                                       │   │
│  │  **This orchestrator is the system's backbone**      │   │
│  │  Phase 2 adds governance logic HERE, not in Foundry │   │
│  └──────────────┬────────────────────┬──────────────────┘   │
└─────────────────┼────────────────────┼──────────────────────┘
                  │                    │
         ┌────────▼──────────┐  ┌─────▼──────────┐
         │ Foundry IQ        │  │ Azure OpenAI   │
         │ Knowledge Base    │  │ (GPT-4)        │
         │                   │  │                │
         │ Retrieves what    │  │ Models are     │
         │ user CAN see      │  │ stateless      │
         │                   │  │ No business    │
         │ Does NOT decide   │  │ rules in       │
         │ what's correct    │  │ prompts        │
         │                   │  └────────────────┘
         │ • Query Planning  │
         │ • Hybrid Search   │
         │ • Reranking       │
         │ • Citations       │
         │ • Document ACL    │
         └────────┬──────────┘
                  │
    ┌─────────────┴─────────────┐
    │                           │
┌───▼───────────┐   ┌───────────▼─────────┐
│ Azure Blob    │   │ SharePoint/OneDrive │
│ Storage       │   │ (Optional)          │
└───────────────┘   └─────────────────────┘

┌───────────────────────────────────────────┐
│           STORAGE & LOGGING               │
│  ┌──────────────┐  ┌──────────────────┐  │
│  │ Cosmos DB    │  │ App Insights     │  │
│  │ • query_logs │  │ • Monitoring     │  │
│  │ • metadata   │  │ • Telemetry      │  │
│  │ • sessions   │  └──────────────────┘  │
│  │ • users      │                         │
│  └──────────────┘                         │
└───────────────────────────────────────────┘
```

**Key Architectural Principle:**
> The orchestrator is the long-term backbone. Foundry IQ is a managed retrieval service that we call, not a system we embed logic into.

### Data Flow

```
User Query: "How many days can I work remotely?"
    ↓
[1] Azure Function receives request
    • Extract user_id from Azure AD token
    • Get user's access_groups
    ↓
[2] Query Foundry IQ Knowledge Base
    • Foundry IQ applies document-level ACL
    • Query planning and decomposition
    • Hybrid search (BM25 + vector) across sources
    • Semantic reranking
    • Extract citations with page numbers
    • Return context + citations
    ↓
[3] Generate Response (Azure OpenAI)
    • Build prompt with retrieved context
    • Generate answer with inline citations [1][2]
    • Format response
    ↓
[4] Log to Cosmos DB
    • Save query, response, citations, user_id
    ↓
[5] Return to User
    Answer: "The company allows remote work up to 3 days per week [1]..."
    Citations: [1] Remote Work Policy, Page 5
```

## Data Models

### 1. Cosmos DB - query_logs Container

```json
{
  "id": "uuid",
  "user_id": "user@company.com",
  "session_id": "session-uuid",
  "query": "How many days can I work remotely?",
  "answer": "The company allows remote work up to 3 days per week [1]...",
  "citations": [
    {
      "id": 1,
      "title": "Remote Work Policy",
      "source": "policies/remote-work.pdf",
      "page": 5,
      "chunk": "Eligible employees may work remotely...",
      "confidence": 0.95
    }
  ],
  "metadata": {
    "kb_id": "wayrag-kb",
    "model": "gpt-4",
    "retrieval_latency_ms": 850,
    "generation_latency_ms": 1420,
    "total_latency_ms": 2270,
    "tokens_used": 1430
  },
  "timestamp": "2026-01-21T14:23:45Z",
  "_partition_key": "user@company.com"
}
```

### 2. Cosmos DB - metadata Container

**Purpose**: Store document metadata for future filtering (Phase 3)

```json
{
  "id": "doc-uuid",
  "doc_id": "doc-uuid",
  "logical_doc_id": "logical-001",
  "document_name": "Employee Handbook 2026",
  "document_link": "documents/handbook-2026.pdf",
  "file_type": "pdf",
  "last_modified": "2026-01-15T10:00:00Z",
  "created_by": "user@company.com",
  "author": "HR Department",
  "doc_authority": 1,
  "stage": "active",
  "valid_from": "2026-01-01T00:00:00Z",
  "valid_till": "2026-12-31T23:59:59Z",
  "access_policy": {
    "allowed": ["Internal", "Secret"],
    "denied": []
  },
  "version": 1,
  "created_at": "2026-01-10T09:00:00Z",
  "_partition_key": "doc-uuid"
}
```

**Note**: Phase 1 stores metadata for future use. Phase 3 will add UI filters.

### 3. Cosmos DB - users Container

```json
{
  "id": "user@company.com",
  "user_id": "user@company.com",
  "employee_id": "EMP001",
  "name": "John Doe",
  "access_groups": ["Internal", "Secret"],
  "created_at": "2026-01-15T09:00:00Z",
  "is_active": true,
  "_partition_key": "user@company.com"
}
```

### 4. Foundry IQ Knowledge Base Configuration

```json
{
  "knowledge_base_id": "wayrag-kb",
  "name": "wayRAG Knowledge Base",
  "sources": [
    {
      "type": "azure_blob_storage",
      "container": "documents",
      "sync_schedule": "hourly"
    },
    {
      "type": "sharepoint",
      "site_url": "https://client.sharepoint.com/sites/docs",
      "sync_schedule": "hourly"
    }
  ],
  "retrieval_config": {
    "mode": "hybrid",
    "reasoning_effort": "medium",
    "top_k": 10,
    "enable_semantic_ranking": true,
    "citation_extraction": "automatic"
  },
  "security": {
    "enable_document_acl": true,
    "permission_model": "azure_ad",
    "inherit_source_permissions": true
  },
  "indexing_config": {
    "chunk_size": 1000,
    "chunk_overlap": 200,
    "enable_image_extraction": true,
    "enable_table_extraction": true
  }
}
```

## Access Control Model (Phase 1)

### How Access Control Works

**Model:**
Access control in Phase 1 is:
- **Document-level** (not field-level, not paragraph-level)
- **Group-based** (user belongs to groups like "Finance", "HR", "Internal", "Secret")
- **Static** (permissions don't change based on context or time)

**Implementation Flow:**

1. **User Authentication**: Azure AD provides user identity
2. **Group Resolution**: Retrieve user's group memberships from Azure AD
3. **Query Filtering**: Pass user groups to Foundry IQ
4. **Document Filtering**: Foundry IQ applies security trimming:
   ```
   filter = document.allowed_groups intersects user.access_groups
   ```
5. **Retrieval**: Only documents matching the filter are retrieved

**Example:**
```
User: john.doe@company.com
User Groups: ["Internal", "Finance"]

Document A:
  allowed_groups: ["Internal"]
  Result: ✅ Retrieved (user has "Internal")

Document B:
  allowed_groups: ["Secret"]
  Result: ❌ Filtered out (user lacks "Secret")

Document C:
  allowed_groups: ["Finance", "Secret"]
  Result: ✅ Retrieved (user has "Finance")
```

### Explicit Limitations

**Phase 1 access control does NOT account for:**

- ❌ **Document lifecycle**: draft/active/deprecated all treated equally
  - A deprecated document with matching groups WILL be retrieved
  
- ❌ **Authority precedence**: no `doc_authority` enforcement
  - Lower-authority documents appear alongside high-authority ones
  
- ❌ **Version selection**: may return old and new versions
  - If both v1.0 and v2.0 match group filter, both may appear
  
- ❌ **Time-based validity**: `valid_from`/`valid_till` ignored
  - Documents outside their validity window still appear
  
- ❌ **Contextual permissions**: access doesn't change based on query intent
  - Same user gets same documents regardless of what they ask

### Why These Limitations Are Acceptable

Phase 1 focuses on **retrieval correctness and security**, not knowledge governance.

The goal is to prove:
- ✅ Users only see documents they're authorized to access (security)
- ✅ Citations are accurate and traceable (retrieval correctness)
- ✅ The system works reliably at scale (plumbing works)

**Knowledge governance** (which document is authoritative, which version is correct, which documents are still valid) is **Phase 2's responsibility**.

## Technology Stack

### Azure Services

| Service | Purpose | Why |
|---------|---------|-----|
| Azure AI Foundry | AI project hub | Centralized AI service management |
| Foundry IQ | Knowledge base & retrieval | Replaces custom hybrid search + reranking |
| Azure OpenAI | GPT-4 deployment | Response generation |
| Azure AD / Entra ID | User authentication | Enterprise SSO, user identity |
| Azure Blob Storage | Document storage | Scalable, cost-effective storage |
| Cosmos DB | NoSQL database | Query logs, metadata, user data |
| Azure Functions | Serverless API | Auto-scaling, pay-per-use |
| Static Web Apps | Frontend hosting | Global CDN, automatic HTTPS |
| Application Insights | Monitoring | Telemetry, logging, alerting |
| Key Vault | Secrets management | Secure credential storage |

### Application Stack

| Component | Technology | Version |
|-----------|-----------|---------|
| Frontend | React + TypeScript | 18.x |
| Backend | Python | 3.11 |
| API Framework | Azure Functions | v4 |
| AI SDK | azure-ai-projects | Latest |
| Auth | azure-identity | Latest |

## Local vs Cloud Mapping

### Module Migration

| Local Module | Cloud Implementation | Notes |
|--------------|---------------------|-------|
| kb_builder | Azure Blob upload + Foundry IQ auto-indexing | No manual chunking needed |
| chunker | Foundry IQ automatic chunking | Built-in with configurable size/overlap |
| doc_filterer | Foundry IQ document-level ACL | Native permission filtering |
| secure_user_login | Azure AD authentication | Enterprise SSO |
| chat_interface | React UI + Cosmos DB sessions | Web-based, scalable |
| query_enhancer | **Phase 2** | Will use Azure OpenAI |
| orchestrator | **Phase 2** | Will use Azure Functions |
| retriever (BM25 + Semantic) | Foundry IQ hybrid search | Native implementation |
| reranker | Foundry IQ semantic reranking | Native implementation |
| generator | Azure Functions + Azure OpenAI | Custom citation formatting |
| metadata storage | Cosmos DB metadata container | Queryable, scalable |
| conversation logger | Cosmos DB query_logs container | Persistent, partitioned by user |

### Feature Parity

| Feature | Local | Cloud Phase 1 | Future Phase |
|---------|-------|--------------|--------------|
| Document-level access control | ✅ CSV | ✅ Azure AD + Foundry IQ | - |
| Hybrid retrieval (BM25 + vector) | ✅ Custom | ✅ Foundry IQ | - |
| Semantic reranking | ✅ LLM | ✅ Foundry IQ | - |
| Citation with page numbers | ✅ Custom | ✅ Foundry IQ | - |
| Multi-source ingestion | ❌ Local only | ✅ Blob/SharePoint/OneDrive | - |
| Contextual query enhancement | ✅ | ❌ | Phase 2 |
| Content moderation | ✅ | ❌ | Phase 2 |
| Query classification | ✅ | ❌ | Phase 2 |
| Query decomposition | ✅ | ❌ | Phase 2 |
| Claim validation | ✅ | ❌ | Phase 2 |
| Conflict resolution | ✅ Authority-based | ❌ | Phase 3 |
| Metadata filtering UI | ❌ | ❌ | Phase 3 |
| Multi-turn memory | ✅ History | ❌ | Phase 3 |

## Metadata Module - Future Provision

Phase 1 stores document metadata in Cosmos DB but doesn't expose filtering in the UI. This sets the foundation for Phase 3 advanced filtering.

**Stored Metadata Fields**:
- `doc_authority` (1=highest, 2=medium, 3=least)
- `stage` (draft/active/deprecated)
- `valid_from` / `valid_till` timestamps
- `access_policy` (allowed/denied groups)
- `created_by`, `last_modified`

**Phase 3 Will Add**:
- UI filters for authority level
- Time-based validity filtering
- Stage-based document filtering
- Search within specific metadata criteria
- Conflict resolution using doc_authority

## Success Criteria

| Metric | Target | Measurement |
|--------|--------|-------------|
| Query Success Rate | 95%+ | Successful responses / total queries |
| Citation Accuracy | 100% | All answers must have citations |
| User Access Control | 100% | Users only see authorized documents |
| Response Latency | <3 seconds | End-to-end query to response |
| Deployment Time | <30 minutes | From code to running system |
| Multi-Source Support | 2+ sources | Blob + SharePoint/OneDrive |

## Next Steps

1. ✅ **Infrastructure Setup** - Deploy Azure resources using Bicep
2. ✅ **Foundry IQ Configuration** - Create knowledge base, connect sources
3. ✅ **User Data Migration** - Import users from CSV to Cosmos DB
4. ✅ **Document Upload** - Upload documents to Azure Blob Storage
5. ✅ **Frontend Deployment** - Deploy React UI to Static Web Apps
6. ✅ **Backend Deployment** - Deploy Azure Functions API
7. ✅ **Testing** - Verify RBAC and citations
8. ✅ **Client Demo** - Showcase working system

---

## Summary

**Phase 1 is a focused, deployable product** that demonstrates:
- ✅ Core RAG with citations
- ✅ Document-level access control
- ✅ Multi-source data ingestion
- ✅ Production-ready UI and API
- ✅ Foundation for advanced features in Phase 2+

**What Phase 1 IS:**
- An Azure-native RAG foundation
- Secure and demonstrable (credibility and correctness of plumbing)
- Suitable for pilots and early client adoption
- A retrieval system that respects user permissions

**What Phase 1 IS NOT:**
- ❌ A knowledge management system
- ❌ A source-of-truth engine
- ❌ A compliance solution
- ❌ An authoritative answer system

**Those claims belong to Phase 2 and beyond.**

**Not trying to replicate all 11 stages of local wayRAG in Phase 1**. We're building incrementally with each phase adding orchestration capabilities while maintaining a working product at every step.

Phase 1 deliberately optimizes for:
- Speed of delivery
- Azure alignment
- Architectural correctness

By constraining scope and preserving clean seams, it creates a stable foundation on which **real knowledge governance** can be layered in Phase 2 — without rewrites, migrations, or loss of credibility.

**This is a foundation, not the destination.**
