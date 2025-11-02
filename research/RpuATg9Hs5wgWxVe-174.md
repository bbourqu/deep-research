**Quick low‑code guide to the n8n In‑Memory Vector Store & a custom LLM**

1. **What the in‑memory store actually is**  
   * `n8n‑nodes‑langchain.vectorstoreinmemory` is a **root** node that keeps your vectors in the node‑process RAM.  
   * It’s great for prototyping: you can push embeddings in‑flight, retrieve them instantly, and experiment with LangChain‑style RAG pipelines.  
   * **Critical caveats**  
     - No persistence – everything vanishes on restart or memory pressure.  
     - Keys are global, not workflow‑scoped. Anyone who knows the key can pull your vectors (so avoid confidential docs).  
     - Not usable in queue mode (workers don’t share the same memory).  
   * For production you’ll want a persistent store (Pinecone, Qdrant, PGVector, etc.) or a custom node that wraps one of those back‑ends.

2. **Hooking it up to a custom LLM**  
   * Use any LangChain‑compatible LLM (OpenAI, Anthropic, local Llama‑CPP, etc.). In n8n you can tap a `Chat` node or a custom “LLM” node that exposes the `prompt` → `response` API.  
   * **Typical flow**  
     1. **Generate embeddings** – a `Python Code` or `Function` node calls your LLM’s embeddings endpoint.  
     2. **Store** – feed the vector + metadata into `InMemoryVectorStore` (set a key, e.g. `my‑docs`).  
     3. **Retrieve** – a `Retriever` sub‑node (`n8n‑nodes‑langchain.retrievervectorstore`) queries the store for the nearest neighbors.  
     4. **Pass to LLM** – concatenate the retrieved snippets with the user prompt and send that string to your custom LLM node.  
   * If you’re building a custom LLM node yourself, just implement the `execute` method to accept a prompt, forward it to your LLM endpoint, and return the response. The In‑Memory store stays in the same workflow, so your custom node can read/write it directly via the global key.

**Bottom line:** Use the in‑memory store for rapid prototyping; be mindful of its volatility and lack of isolation; for anything production‑grade, swap in a persistent vector DB and keep your LLM behind a secure, custom node. Happy automating!