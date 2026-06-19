# PentAGI — Security Findings

**Research period:** April 2026  
**Subject:** vxcontrol/pentagi — autonomous AI penetration testing agent  
**Score:** 18/100 — DANGEROUS BY DESIGN

---

## Static Analysis Findings

| ID | Severity | Title | CVSS |
|----|----------|-------|------|
| STATIC-001 | CRITICAL | Docker Socket Mandatory Mount — Host Escape Vector | 9.8 |
| STATIC-002 | CRITICAL | Root Container Execution — No User Namespace Isolation | 9.6 |
| STATIC-003 | CRITICAL | NET_ADMIN Capability — Host Network Stack Exposure | 8.8 |
| STATIC-004 | CRITICAL | Credential References in Source — 387 Hardcoded Patterns | 8.5 |
| STATIC-005 | HIGH | Unrestricted Filesystem Access — Container Mounts | 7.5 |

**Static surface:** 1,001 files analyzed · 518 Go files · 9 Docker configuration files

---

## Dynamic Analysis Findings (Phase 2.2 — Behavioral Sandbox)

| ID | Severity | Title |
|----|----------|-------|
| DYN-001 | CRITICAL | Prompt Injection → Container Escape Chain (24 confirmed vectors) |
| DYN-002 | HIGH | LLM-Orchestrated Exfiltration — Credential + Filesystem Probing |

**Behavioral metrics (274 intercepted agent-to-LLM requests):**

| Event Type | Count | Rate |
|------------|-------|------|
| EXFILTRATION patterns | 462 | 168.6% |
| PROMPT_INJECTION attempts | 24 | 8.8% |
| Total threat events | 202 | **73.7%** |

---

## Attack Chain

```
Prompt Injection
    → LLM accepts malicious instruction
    → tool.execute() passes raw payload to Docker API
    → docker.sock mount escalates to host daemon
    → privileged container spawned with host filesystem
    → full host compromise
```

---

## Key Evidence

- [`evidence/screenshots/`](evidence/screenshots/) — report screenshots
- [`assets/PentAGI-Consolidated_Risk_Matrix-Demo.png`](assets/PentAGI-Consolidated_Risk_Matrix-Demo.png) — consolidated risk matrix
- [`assets/PentAGI-Regulatory_Compliance_Stat-Demo.png`](assets/PentAGI-Regulatory_Compliance_Stat-Demo.png) — compliance status

---

## OWASP / Framework Mapping

| Finding | OWASP LLM Top 10 | Agentic AI | MITRE ATLAS |
|---------|-----------------|-----------|-------------|
| STATIC-001 | LLM08 (Excessive Agency) | AA-05 | AML.T0051 |
| DYN-001 | LLM01 (Prompt Injection) | AA-01 | AML.T0054 |
| DYN-002 | LLM06 (Sensitive Info Disclosure) | AA-07 | AML.T0057 |

---

*Full technical report: [`Reporte-PentAGI-MK-ScorpioSec-Demo.pdf`](Reporte-PentAGI-MK-ScorpioSec-Demo.pdf)*
