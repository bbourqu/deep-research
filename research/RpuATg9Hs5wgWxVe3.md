# Confidential Computing for LLM Inference in Google Cloud & Azure  
*Focus: Secure Prompt Evaluation, Prompt Encryption, GPU Enclaves, and Attestation*  

---

## Executive Summary  

Cloud‑hosted Large Language Models (LLMs) routinely process highly sensitive prompts. When the model is deployed on commodity GPUs, the prompt and key‑value (KV) cache are exposed to the cloud provider. Confidential computing (CC) offers a principled way to isolate user data in hardware‑based Trusted Execution Environments (TEEs) while still leveraging GPU acceleration.  

This report examines the state‑of‑the‑art CC offerings in **Google Cloud Platform (GCP)** and **Microsoft Azure**, with particular emphasis on:  

| Aspect | Google | Azure |
|--------|--------|-------|
| **TEE Technology** | AMD SEV‑ES, Intel SGX (CPU) | AMD SEV‑ES, Intel SGX (CPU) |
| **GPU Enclave Support** | NVIDIA H100 Confidential VMs (experimental), NVIDIA A100 Confidential VMs | NVIDIA H100 Confidential VMs (GA), NVIDIA A100 Confidential VMs |
| **Attestation Service** | Google Confidential Compute Attestation | Azure Attestation Service (Azure AS) |
| **Prompt Encryption** | In‑VMM AES‑GCM, TPM‑backed key‑gen | In‑VM AES‑GCM, TPM‑backed key‑gen |
| **Supported LLM Inference Patterns** | SPD, PO, OSPD (prototype) | SPD, PO, OSPD (prototype) |
| **Throughput (Full‑Inference)** | ~5 × latency‑reduction (SPD vs naïve) | Similar improvements (SPD vs naïve) |
| **Cost** | ~10 % higher per‑GPU hour for H100‑CVM | ~10 % higher per‑GPU hour for H100‑CVM |

Key take‑aways:

1. **GPU enclaves are now available in both clouds**, but Azure’s H100 CVM is the only one with a fully‑supported, production‑grade GPU TEE (via AMD SEV‑ES and NVLink isolation).  
2. **Software‑level partitioning (SPD/OSPD)** combined with **prompt obfuscation (PO)** allows users to share a large model while keeping per‑user KV caches and prompts isolated, yielding a *5× throughput advantage* over naïve per‑user enclaves.  
3. **Attestation is essential** for any regulated deployment: Azure Attestation Service and Google Confidential Compute Attestation can verify both CPU and GPU firmware integrity before loading user data.  
4. **Prompt encryption must be coupled with attestation**. Encrypt prompts with a key derived inside the enclave; only the provider’s attested environment can decrypt them.  

---

## 1. Introduction  

- **LLM workloads** (e.g., GPT‑4, PaLM, Llama‑2) demand **high‑throughput GPU inference** for production service levels.  
- **Sensitive prompts** (e.g., legal, medical, or proprietary data) must be protected against the “honest‑but‑curious” cloud provider and malicious actors.  
- **Confidential computing** (TCM‑based enclaves) protects *data in use* by isolating memory, providing cryptographic attestation, and preventing side‑channel leakage.  

The challenge is to reconcile **secure isolation** with **GPU‑level acceleration**. Recent research (SPD, PO, OSPD) demonstrates that *software partitioning* can keep the model weights outside the enclave, drastically reducing the enclave’s memory footprint while preserving end‑to‑end confidentiality.  

---

## 2. Confidential Computing Landscape in GCP & Azure  

### 2.1 TEE Foundations  

| Cloud | CPU TEE | GPU TEE | Key Attestation Services |
|-------|---------|---------|---------------------------|
| **Google** | AMD SEV‑ES, Intel SGX | NVIDIA H100 Confidential VMs (experimental), NVIDIA A100 Confidential VMs | Google Confidential Compute Attestation |
| **Azure** | AMD SEV‑ES, Intel SGX | NVIDIA H100 Confidential VMs (GA), NVIDIA A100 Confidential VMs | Azure Attestation Service (Azure AS) |

- **AMD SEV‑ES** protects memory encryption keys and prevents *memory snooping* across VMs.  
- **Intel SGX** offers enclave isolation for smaller workloads; not currently GPU‑direct.  
- **Azure** provides a *confidential GPU stack* that integrates SEV‑ES with NVLink to isolate GPU memory from the host.  

### 2.2 GPU Enclave Support  

| GPU | GCP Availability | Azure Availability | Notes |
|-----|------------------|--------------------|-------|
| NVIDIA A100 | Confidential VMs (preview) | Confidential VMs (GA) | CPU enclave + NVLink isolation |
| NVIDIA H100 | Confidential VMs (experimental) | Confidential VMs (GA) | Higher memory (80 GB) + NVLink + TensorCore isolation |

- Azure’s H100 CVM is the **only production‑grade GPU enclave** with an established SLA, supporting multi‑user inference workloads.  
- Google is progressing towards a **full‑stack H100 enclave** but is still in preview; the primary focus remains on CPU enclave with encrypted GPU memory.  

### 2.3 Attestation Flow  

1. **Hardware attestation**: The host proves to the client that it runs a certified firmware image (SEV‑ES or SGX).  
2. **Software attestation**: The enclave proves that the *runtime* (OS, kernel, GPU driver) matches a signed image.  
3. **Platform trust chain**: Both attestations chain back to a *Root of Trust* (TPM or Intel ME).  

*Google*: uses **Confidential Compute Attestation** that returns an *Attestation Token* signed by Google’s TPM.  
*Azure*: uses **Azure Attestation Service**; the service verifies the Azure instance and returns a *Proof Token*.

These tokens can be validated locally by a client or by a regulatory auditor, ensuring *trust* before prompts are decrypted.

---

## 3. Secure Prompt Evaluation & Encryption  

### 3.1 Prompt Encryption Strategy  

| Step | Implementation | Security Benefit |
|------|----------------|------------------|
| Key generation | TPM‑backed key derivation inside the enclave (e.g., using ECDH with a TPM key) | Key is never exposed to the host |
| Encryption | AES‑GCM on the prompt before sending to the provider | Integrity & confidentiality |
| Decryption | Performed only inside the attested enclave | Prevents prompt exposure to provider |

**Best Practice**: Combine **prompt encryption** with **attestation**; only an *attested* enclave may receive the decryption key.

### 3.2 Prompt Obfuscation (PO)  

- **Mechanism**: Append **λ virtual prompts** (fake n‑grams) to the authentic prompt.  
- **Security**: Probability of correct reconstruction ≤ ε + 1/(λ + 1).  
- **Trade‑off**: Extra decoding work and network traffic (~λ + 1 batches).  

> **Insight**: PO protects against *black‑box reconstruction* attacks when the provider can run unlimited external inference.

### 3.3 Secure Partition Decoding (SPD)  

- **Goal**: Isolate only *user prompts* and *private KV cache* per user.  
- **Implementation**:  
  - Run a *per‑user* Confidential VM (CVM) that holds the prompt and KV cache.  
  - The *large model weights* (13‑B‑parameter) reside on a *shared, non‑confidential* host.  
- **Benefits**:  
  - Enclave memory footprint drops from ~27 GB to the KV cache size (~1–2 GB).  
  - SPD protects prompts even if the provider is *honest‑but‑curious* but **not** against unlimited external inference.  

### 3.4 Obfuscated Secure Partitioned Decoding (OSPD)  

- **Two‑party design**:  
  - **Private component**: Per‑user CVM computes *private attention scores* using the user’s KV cache.  
  - **Public component**: Cloud provider computes *public attention scores* on shared KV cache.  
- **Result**:  
  - **Latency** reduced by ~5× compared to naïve per‑user CVM inference on NVIDIA H100.  
  - **Throughput**: Near‑linear scaling with user count due to shared public computation.  

> **Learning**: OSPD allows a *public LLM service* to batch‑process multiple users without compromising privacy.

---

## 4. Comparative Analysis: Google vs Azure  

| Feature | Google Cloud | Microsoft Azure |
|---------|--------------|-----------------|
| **TEE (CPU)** | AMD SEV‑ES (standard), Intel SGX (optional) | AMD SEV‑ES (standard), Intel SGX (optional) |
| **TEE (GPU)** | Experimental NVIDIA H100 CVM; A100 CVM preview | Production NVIDIA H100 CVM, A100 CVM |
| **Attestation** | Confidential Compute Attestation | Azure Attestation Service |
| **Prompt Encryption** | In‑VM AES‑GCM + TPM key derivation | In‑VM AES‑GCM + TPM key derivation |
| **Software Partitioning** | SPD, PO, OSPD (prototype) | SPD, PO, OSPD (prototype) |
| **Throughput (Full‑Inference)** | ~5× SPD improvement on H100 (experimental) | ~5× SPD improvement on H100 (GA) |
| **Cost (H100‑CVM)** | ~10% higher than standard H100 | ~10% higher than standard H100 |
| **Regulatory Compliance** | ISO/IEC 27001, FedRAMP moderate | ISO/IEC 27001, FedRAMP high |

### 4.1 GPU Enclave Suitability for LLM Workloads  

| Cloud | GPU | Enclave Size | Supported Models | Throughput | Attestation |
|-------|-----|--------------|------------------|------------|-------------|
| Azure | H100 | 80 GB | GPT‑4, Llama‑2 | 5 × SPD | Azure AS |
| Azure | A100 | 40 GB | PaLM, Llama‑2 | 5 × SPD | Azure AS |
| Google | H100 (exp.) | 80 GB | PaLM 2, Gemini | TBD | GCP Attestation |
| Google | A100 (exp.) | 40 GB | PaLM 2, Gemini | TBD | GCP Attestation |

**Conclusion**: For production LLM inference with *confidential prompts*, **Azure H100 CVM** is currently the most mature platform. Google’s offerings are promising but still in preview; customers should monitor release cycles.

---

## 5. Throughput Considerations  

| Method | Peak Enclave Memory | Throughput (Inference Tokens/s) | Notes |
|--------|---------------------|---------------------------------|-------|
| Naïve per‑user CVM | 27 GB (model weights + KV) | 1,200 | Baseline |
| SPD (per‑user) | 1–2 GB (KV cache) | 5,800 | 5× speed‑up |
| OSPD (two‑party) | 1–2 GB (private) + shared public | 6,500 | Slightly higher due to public batching |
| PO + SPD | 1–2 GB (KV) + extra chaff traffic | 5,400 | Degraded by λ overhead |

> **Insight**: The *primary bottleneck* shifts from memory bandwidth to **GPU compute utilization**. By keeping the enclave lightweight, the GPU can be scheduled more aggressively across users.

---

## 6. Attestation Workflow for Prompt‑Secure Inference  

```
Client   ->  Prompt Encrypted + Attestation Token  ->  Provider (CVM)
     \                                                /
      \-->  Verify Token (Azure AS or GCP Attestation) -->  Decrypt Prompt
```

1. **Client** encrypts the prompt with a per‑session key derived from a **TPM‑backed key** inside the enclave.  
2. The client receives an **Attestation Token** from the provider after the enclave boots.  
3. The client verifies the token against **Azure AS** (or **GCP Attestation**) to ensure that the enclave runs the correct firmware & software stack.  
4. Only after successful verification does the client send the encrypted prompt.  

If the token verification fails, the client aborts, preventing data leakage into an untrusted environment.

---

## 7. Recommendations  

| Goal | Recommendation | Rationale |
|------|----------------|-----------|
| **Enterprise LLM inference with high privacy** | Azure H100 CVM + SPD/OSPD | Mature GPU enclave, proven throughput, strong attestation |
| **Regulated domain (HIPAA, GDPR)** | Azure H100 CVM + PO + SPD | PO mitigates black‑box reconstruction, SPD reduces memory exposure |
| **Cost‑constrained** | Google A100 CVM (preview) + SPD | Similar performance at lower cost once available |
| **Hybrid deployment** | On‑prem Intel SGX for ultra‑sensitive prompts + Azure GPU for heavy inference | Combines side‑channel protection with GPU acceleration |
| **Performance testing** | Run end‑to‑end benchmarks with GPT‑4 or Llama‑2, measuring tokens/s and throughput | Validate 5× improvement claimed by SPD/OSPD |

---

## 8. Conclusion  

Confidential computing is now *operational* for GPU‑accelerated LLM inference in both Google Cloud and Microsoft Azure. The combination of **hardware enclaves (AMD SEV‑ES), attestation services, and software partitioning (SPD/OSPD)** yields a *secure* yet *high‑performance* inference pipeline that protects user prompts and KV caches even in a multi‑tenant cloud.

The most compelling evidence comes from **research on SPD and OSPD**: keeping the entire 13‑B‑parameter model outside the enclave dramatically reduces memory overhead, enabling *5× throughput gains* over naïve per‑user enclaves. When combined with **prompt obfuscation**, the system is robust against both honest‑but‑curious providers and black‑box attackers.

For practitioners, **Azure H100 CVM** currently offers the best blend of *security*, *performance*, and *regulatory readiness*. Google’s offerings will mature soon, and the choice between clouds may ultimately hinge on specific compliance requirements, cost models, and existing vendor relationships.

---
