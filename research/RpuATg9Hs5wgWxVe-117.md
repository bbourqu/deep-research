**GPU‑CC (GPU‑Based Confidential Computing)** is NVIDIA’s framework for extending the trusted computing base (TCB) of a system from the CPU to discrete GPUs. It provides a set of security features that let a GPU operate in a *confidential* mode—protected from the host, hypervisor, or other VMs—while still delivering the raw performance of the device.  

### 1.  Core building blocks

| Feature | What it does | How it’s implemented |
|---------|--------------|----------------------|
| **Secure boot** | Guarantees that only authenticated firmware runs on the GPU. | A signed boot‑loader checks the ROM or eFuse‑stored public key against the GPU’s firmware hash at the highest privilege level (EL 3). |
| **Device attestation** | Lets a host or verifier prove that the GPU’s firmware and configuration are in a known‑good state before enabling confidential mode. | The GPU produces a signed SPDM attestation report that includes firmware measurements, monotonic counter values, and current security settings. The host verifies this report against a trusted root‑certificate authority. |
| **Firewall / memory isolation** | Prevents a malicious driver or hypervisor from reading or writing GPU control registers or memory that belong to other VMs. | The secure processor engine exposes a *control‑register firewall* that limits access to BAR 0 and *GPU memory protection registers* (CPR). Only authenticated, privileged operations are allowed. |
| **Staging‑buffer encryption** | Protects data in flight between the host and GPU while GPU‑CC itself does not encrypt memory at run‑time. | SPDM is used to exchange encryption keys; the host encrypts the staging buffers that feed the GPU, and the GPU decrypts them before use. |
| **Memory scrubbing** | Removes residual data that might remain in GPU memory after a VM is released. | The Secure Processor engine performs a full wipe (scrub) of GPU memory whenever a new virtual machine takes ownership of the GPU, ensuring no data persists between tenants. |

### 2.  How it works

1. **Boot‑time check** – At EL 3 the firmware’s public key is read from ROM/eFuse and used to verify the GPU firmware hash.  
2. **SPDM key exchange** – The host and GPU negotiate a symmetric key via SPDM, establishing the encryption keys for staging buffers.  
3. **Attestation** – The GPU creates a signed attestation report that the host (or external verifier) can use to confirm the GPU’s secure‑boot status before turning on confidential computing.  
4. **Firewall activation** – Once the report is verified, the GPU’s firewall restricts access to BAR 0 (control registers) and GPU memory (CPR), preventing the host from snooping or tampering.  
5. **VM‑level isolation** – When a VM claims a GPU, the Secure Processor scrubs the device’s memory to eliminate any stale data.  
6. **Confidential mode** – The VM can now use the GPU in an isolated “confidential” mode, where its instructions and data are protected from the host.

### 3.  Benefits

* **Hardware‑level isolation** that covers the entire GPU, not just the CPU.  
* **Device‑specific attestation** that assures the host of the GPU’s integrity.  
* **Fine‑grained memory protection** without runtime memory encryption (thus preserving performance).  
* **Scalable for multi‑tenant workloads**—each VM gets a clean, zero‑conflict GPU instance.

### 4.  Key take‑away

GPU‑CC is essentially a *GPU‑centric* extension of the classic trusted‑compute model. It brings secure boot, attestation, and a firewall to discrete GPUs, leveraging SPDM for key management and encryption of the only stage that requires it—staging buffers. The secure processor’s memory‑scrubbing engine ensures that data does not linger in GPU memory when ownership changes, while the firewall protects the GPU’s internal registers and control paths. Together, these mechanisms create a minimal yet robust TCB that allows confidential GPUs to safely accelerate AI and other workloads.