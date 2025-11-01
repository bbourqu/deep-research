**Memory‑usage limits for the Simple Vector Store node**

| Setting | Default (self‑host) | Default (n8n Cloud) | Approx. docs that fit |
|---------|---------------------|---------------------|-----------------------|
| `N8N_VECTOR_STORE_MAX_MEMORY` (bytes) | –1 (no hard cap) | 100 MiB | ~8 000 embeddings (≈100‑200 B each) |
| `N8N_VECTOR_STORE_TTL_HOURS` | –1 (no automatic purge) | 7 days | – |

The node keeps all vectors in the process’s RAM, so the limits above govern how much data can stay in‑memory. The Cloud defaults are intentionally conservative to protect shared resources.  

**What happens when the limits are exceeded**

1. **Automatic cleanup** – When the `MAX_MEMORY` cap is crossed, the system automatically removes the oldest or least‑used vector stores until the total usage falls below the limit.  
2. **TTL‑based purge** – If a vector store has been idle for longer than the configured `TTL_HOURS`, it is purged to free space.  
3. **Process‑level eviction** – On low‑memory conditions, the node may be terminated and its data discarded (because the store is non‑persistent).  

These mechanisms keep the node from starving the host or other workflows of memory while ensuring that the store remains usable.  

**References**  
- Memory limits and defaults for the Simple Vector Store node are controlled by the environment variables `N8N_VECTOR_STORE_MAX_MEMORY` and `N8N_VECTOR_STORE_TTL_HOURS`.  
- Exceeding the limits triggers automatic cleanup of the oldest or inactive vector stores.  [source]