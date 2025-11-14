**Air Canada Digital Hub – 12‑Month Migration Blueprint**  
*(IBM + CAST Migration Factory, IBM Mono2Micro, ServiceNow CMDB/Now‑Assist, Docker/K8s, SBOM‑enabled CI/CD, Orca/Aqua security, Terraform IaC)*  

| Phase | Months | Key Deliverables | Milestones | Primary Risks | Mitigation Actions |
|-------|--------|------------------|------------|---------------|--------------------|
| **1 – Discovery & Inventory** | 1–2 | • Complete application‑portfolio audit with **CAST Highlight** (60 % faster discovery). <br>• Generate **SBOM patterns** in ServiceNow via **API‑Insights** and **SBOM‑Pattern‑Generator**. <br>• Build initial CMDB with **Now‑Assist** and **SGC** integrations. | • *Discovery kickoff* <br>• *SBOM baseline* <br>• *CMDB accuracy score > 95 %* | • Incomplete inventory; <br>• CMDB data drift | • Use **Now‑Assist** for duplicate CI detection. <br>• Automate daily **Metric‑Intelligence** sync to ServiceNow. |
| **2 – Risk & Impact Assessment** | 3 | • **Change‑Risk‑Conflict‑Analysis** in ServiceNow <br>• Security risk matrix (Orca/Aqua) <br>• Regulatory gap check (PCI‑3.1, GDPR, FedRAMP). | • *Risk register signed off* <br>• *Compliance checklist complete* | • Undetected legacy security flaws; <br>• Compliance gaps | • Run **Orca** container scans; patch critical CVEs (<20 % of total). <br>• Integrate **SBOM VPR** into CI/CD to flag high‑risk components. |
| **3 – Modernization Decision & Design** | 4–5 | • Prioritize apps: *Rehost*, *Refactor*, *Rebuild* <br>• Architecture blueprint (micro‑service vs. monolith) using **IBM Mono2Micro** <br>• Containerization strategy (Docker + K8s). | • *Decision‑matrix approved* <br>• *Architecture diagram* <br>• *Container policy* | • Wrong modernization path; <br>• Over‑engineering | • Leverage **CAST Highlight** to quantify tech debt. <br>• Adopt the **8 % approach** to surface critical debt. |
| **4 – Containerization & Packaging** | 6–7 | • Convert selected apps to micro‑services (Mono2Micro, FOSS) <br>• Build Docker images with **resource constraints** (CPU/Memory limits). <br>• Publish to **private registry**. | • *Container image baseline* <br>• *SBOM embedded* (SBOM‑generator) <br>• *Registry policy* | • Resource over‑commit; <br>• Image bloat; <br>• Security vetting lapses | • Enforce **Docker‑resource‑limits** per C‑group and K8s QoS classes. <br>• Run **Trivy/Orca** scans on every image. <br>• Tag images with SBOM metadata. |
| **5 – Integration & CI/CD Setup** | 8–9 | • Terraform IaC for K8s & registry <br>• Prefect 3 Orchestration pipelines <br>• Automate SBOM injection into pipeline (Sonatype/Harbor). | • *IaC repo ready* <br>• *CI/CD pipeline* <br>• *SBOM compliance gate* | • IaC drift; <br>• Pipeline failures | • Adopt **Terraform maturity model** (state file protection, drift detection). <br>• Use **Prefect 3** for workflow retries; <br>• Fail build if SBOM VPR > 2.5. |
| **6 – Testing & Validation** | 10 | • Functional + performance regression <br>• Security penetration test <br>• CMDB reconciliation | • *Test sign‑off* <br>• *Zero‑downtime cut‑over plan* | • Undetected regressions; <br>• Performance bottlenecks | • Run **CI/CD smoke** tests; <br>• Use **Prometheus + Grafana** for real‑time metrics; <br>• Auto‑rollback with Orca auto‑remediation. |
| **7 – Staging Deployment & Monitoring** | 11 | • Blue‑green deployment to staging <br>• Service‑level objectives (SLO) monitoring <br>• SBOM audit logs in ServiceNow | • *Staging sign‑off* <br>• *Monitoring dashboard live* | • Hidden production‑only issues; <br>• Monitoring gaps | • Deploy **Orchestrated K8s** with sidecar metrics; <br>• Verify SBOM audit trail; <br>• Run **Canary** releases. |
| **8 – Production Rollout & Cutover** | 12 | • Final cut‑over to production <br>• Data migration (if needed) <br>• Post‑go‑live monitoring | • *Production live* <br>• *Go‑live checklist* <br>• *Cost & performance baseline* | • Downtime; <br>• Post‑go‑live regressions | • Use **Orca auto‑remediation** for hot‑fixes <br>• Apply **ServiceNow Change‑Risk‑Conflict** during cutover <br>• Schedule **post‑go‑live 7‑day monitoring**. |

---

### Risk Matrix (Illustrative)

| Category | Risk | Likelihood | Impact | Total | Mitigation Priority |
|----------|------|------------|--------|-------|---------------------|
| Legacy App Compatibility | Runtime failures after refactor | Medium | High | 6 | *Code‑level unit tests + integration sandboxes* |
| SBOM Compliance | Missing component in SBOM → regulatory non‑compliance | Low | Critical | 4 | *Automated SBOM injection & audit* |
| Container Resources | CPU/Memory leaks causing OOM kills | Medium | Medium | 4 | *C‑group limits + K8s QoS* |
| Security Vulnerabilities | Unpatched CVEs in 3rd‑party libs | High | High | 9 | *Orca + SBOM VPR gating* |
| CMDB Integrity | Duplicate or stale CIs → incorrect cost modeling | Medium | Medium | 4 | *Now‑Assist duplicate CI detection* |
| Compliance Gaps (PCI, GDPR, FedRAMP) | Non‑compliant data flows | Low | High | 4 | *Compliance‑as‑code (Terraform, ServiceNow)* |
| Cost Overrun | Unexpected infra spend | Medium | Medium | 4 | *Cast AI cost‑adjustment engine* |

**Bottom line:** Use the IBM + CAST migration factory as the central orchestrator, coupled with ServiceNow CMDB/Now‑Assist for data fidelity, SBOM‑enabled CI/CD for security compliance, and Docker/K8s with resource constraints for stable, scalable deployments. The plan above gives you a 12‑month roadmap, clear milestones, and concrete risk mitigation tactics that are grounded in the latest IBM, ServiceNow, and industry‑standard tooling.