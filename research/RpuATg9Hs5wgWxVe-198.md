```markdown
# OWASP Prompt Injection Prevention Cheat Sheet – Concise Summary

1. **Input Sanitization**  
   * Strip or neutralize known injection patterns (e.g., “ignore previous instructions”, “stop generating”).  
   * Use regular expressions, token blacklists, or context‑aware parsers to detect malicious phrasing before it reaches the LLM.

2. **Strict Prompt Construction**  
   * Separate system instructions from user data in a structured format (e.g., `SYSTEM: ... USER: ...`).  
   * Enforce clear boundaries so that user input cannot overwrite or merge with system prompts.

3. **Prompt Filtering Tools**  
   * Deploy prompt‑filter libraries or custom modules that block or rewrite suspicious content prior to model invocation.  
   * Integrate with existing frameworks (LangChain, Retrieval‑Augmented Generation) for automated filtering.

4. **Output Validation**  
   * Scan model responses for signs of system‑prompt leakage or instruction hijacking.  
   * Validate against expected formats (JSON, plain text) and flag anomalies.

5. **Human‑in‑the‑Loop (HITL)**  
   * Trigger manual review for high‑risk keywords (credentials, API keys) or when filters cannot determine safety.  
   * Maintain oversight over automated decisions to reduce false positives/negatives.

6. **Monitoring & Logging**  
   * Log every prompt, response, and tool call with timestamps and session metadata.  
   * Feed telemetry into SIEM or observability platforms to detect repeat or coordinated injection attempts.

7. **Continuous Testing & Tuning**  
   * Run regular jailbreak and injection tests; update regexes, filters, and safety layers accordingly.  
   * Adjust model prompt‑engineering or fine‑tuning strategies based on detected attack patterns.

*Implementing these layered controls—sanitization, strict formatting, filtering, validation, HITL, logging, and continuous testing—provides a robust defense against prompt‑injection attacks in LLM‑based applications.*
```