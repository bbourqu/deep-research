**Step‑by‑step guide to build an AI system with n8n, Weaviate and LangChain**  
(Integrating an in‑memory vector store, handling 30‑minute timeouts and adding a custom vector‑database node for a *super‑user* role)

---

### 1️⃣  Set up the n8n runtime

| What | Where to read |
|------|---------------|
| Install n8n in Docker (recommended) | <https://docs.n8n.io/hosting/docker/> |
| Configure a production‑ready database (PostgreSQL/MySQL) | <https://docs.n8n.io/hosting/configuration/supported-databases-settings/> |
| Tune execution timeouts for long‑running flows | <https://docs.n8n.io/hosting/scaling/performance-benchmarking/> |

> **Tip:** Keep the “`workflow‑execution‑timeout`” flag as low as possible (e.g. 10 min) and let the *super‑user* flows run in a dedicated queue (see step 6).

---

### 2️⃣  Deploy Weaviate

| What | Where to read |
|------|---------------|
| Official Weaviate docs (schema, index types, vector‑search) | <https://docs.weaviate.io/docs/> |
| Weaviate integration for n8n | <https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.vectorstoreweaviate/> |
| Weaviate cluster‑setup in Docker | <https://docs.weaviate.io/docs/quick-start/> |

> **Why Weaviate?** It natively stores vectors, lets you query by text (BM25) and by filter (GraphQL‑style). It’s the most “AI‑ready” open‑source vector DB at the moment.

---

### 3️⃣  Create a *super‑user*‑specific in‑memory vector store

1. **Add the built‑in LangChain “In‑Memory Vector Store” node**  
   `n8n-nodes-langchain.vectorstoreinmemory` – see <https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.vectorstoreinmemory/>  
2. **Configure it with a short‑lived TTL**  
   * TTL = 30 min (≈ 1800 s) – <https://community.n8n.io/t/memory-issues-on-vector-database/60241>  
   * Use the “Memory Buffer Window” node to clear stale data: <https://docs.n8n.io/integrations/builtin/cluster-nodes/sub-nodes/n8n-nodes-langchain.memorybufferwindow/>  
3. **Assign it to the “Super‑User” role**  
   * Use the built‑in *Roles & Permissions* → “super‑user” → enable the In‑Memory node.  
   * Ensure that only this role can access the node via n8n’s UI or API.

> **Result:** The *super‑user* gets lightning‑fast query results (≈ ms), while the data automatically expires after 30 minutes to keep RAM consumption predictable.

---

### 4️⃣  Build the embedding pipeline

| What | Where to read |
|------|---------------|
| LangChain embeddings (OpenAI, Cohere, etc.) | <https://docs.n8n.io/integrations/builtin/cluster-nodes/sub-nodes/n8n-nodes-langchain.embeddingsopenai/> |
| LangChain “Code” node to run custom Python/Rust scripts | <https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.code/> |

**Example flow**  
1. **Document ingestion** → extract text.  
2. **Embeddings** → call the LangChain embedding node (OpenAI, Cohere, etc.).  
3. **Persist vectors** → upsert into *Weaviate* via the n8n‑Weaviate node: <https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.vectorstoreweaviate/>  
4. **For super‑users** → store the same vector in the in‑memory store with TTL=30 min.

---

### 5️⃣  Add filtering / metadata search

1. **Weaviate** – Use the built‑in GraphQL/SQL filter syntax (nested, array, string) to narrow the search space: <https://qdrant.tech/documentation/filtering/> (the patterns are similar).  
2. **In‑Memory** – The LangChain node accepts a simple JavaScript filter object (field = value).  
3. **Combine** – First narrow by metadata in Weaviate, then run an in‑memory similarity search on the remaining subset.

---

### 6️⃣  Handle 30‑minute timeouts

| Problem | Solution | Reference |
|---------|----------|-----------|
| n8n flow times out after 30 min (default for long‑running jobs) | 1. Split the flow into a *trigger* → *sub‑workflow* (via *Sub‑Workflow* node). 2. Use a *queue* (RabbitMQ/Redis) to decouple the trigger. 3. In the sub‑workflow, set “Execution timeout” to *Unlimited* or a higher value. | <https://docs.n8n.io/hosting/scaling/performance-benchmarking/> |
| Weaviate query latency if index is on disk | Use an in‑memory index for hot vectors (HNSW) and a disk‑based index for cold data: <https://milvus.io/blog/introduce-milvus-2-5-full-text-search-powerful-metadata-filtering-and-more.md> (concepts apply to Weaviate as well) | |

---

### 7️⃣  Build a custom “Super‑User” vector‑store node

If you want a dedicated node that only *super‑users* can invoke:

1. **Create a new n8n node**: <https://docs.n8n.io/integrations/creating-nodes/build/>  
2. **Implement the node** to wrap the In‑Memory vector‑store logic (LangChain `VectorStoreInMemory`).  
3. **Expose a “Role” parameter** that accepts “super‑user” only.  
4. **Publish** the node to your private n8n package registry (or GitHub repo).  
5. **Install** it via *Custom Node → Add Node* in n8n.

> **Reference:** community discussion on adding a custom vector‑store node: <https://community.n8n.io/t/add-node-for-chromadb-vector-database/73819>

---

### 8️⃣  Full example workflow

```
Trigger → Document Ingestion → Embedding (LangChain) → 
   Upsert to Weaviate (n8n-weaviate node) → 
   Store in In-Memory (super‑user only) → 
   Query (by similarity or metadata) → 
   Reranker (e.g., Cohere Reranker) → 
   LLM answer (OpenAI, GPT‑4, etc.) → 
   Return to UI
```

* All steps are separated by *Sub‑Workflow* nodes so that the 30‑minute timeout applies only to the “query” part, while ingestion runs instantly.

---

### 9️⃣  Testing & health‑checks

| What | Where to read |
|------|---------------|
| n8n health‑check endpoints | <https://docs.n8n.io/api/> |
| Weaviate health‑checks | <https://qdrant.tech/documentation/guides/security/> (analogous) |
| Custom node unit tests | <https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.vectorstoreinmemory/> |

---

## 📚 References & URLs

1. n8n built‑in LangChain In‑Memory node – <https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.vectorstoreinmemory/>
2. n8n Weaviate vector store node – <https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.vectorstoreweaviate/>
3. n8n Weaviate integration guide – <https://milvus.io/docs/milvus_and_n8n.md> (concepts apply)
4. Weaviate docs – <https://docs.weaviate.io/docs/>
5. n8n hosting & performance – <https://docs.n8n.io/hosting/scaling/performance-benchmarking/>
6. n8n memory‑buffer window node – <https://docs.n8n.io/integrations/builtin/cluster-nodes/sub-nodes/n8n-nodes-langchain.memorybufferwindow/>
7. Custom node creation guide – <https://docs.n8n.io/integrations/creating-nodes/build/>
8. n8n community discussion: super‑user role – <https://community.n8n.io/t/rag-metadata-difference-between-vector-store-tool-node-and-vector-store-nodes/69710>
9. n8n community discussion: add custom vector‑store node – <https://community.n8n.io/t/add-node-for-chromadb-vector-database/73819>
10. Cohere Reranker example – <https://community.n8n.io/t/cohere-in-langchain-code-node-to-improve-rag-with-reranker/48413>

--- 

With these steps you’ll have an **AI system** where:

* The **Weaviate** backend stores all vectors for persistence and metadata search.  
* The **in‑memory vector store** gives *super‑users* sub‑second responses for the last 30 minutes of activity.  
* 30‑minute execution windows are respected by splitting the flow into timed sub‑workflows.  
* A **custom node** guarantees that only users with the *super‑user* role can touch the fastest vector store.

Happy automating!