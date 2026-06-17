# Case Study #2 — PentAGI Autonomous AI Agent Security Analysis

**Research period:** April 2026  
**Subject:** [PentAGI](https://pentagi.com/) — autonomous AI-powered penetration testing agent  
**Classification:** RESEARCH — publicly available codebase  
**Author:** MK ScorpioSec Research Team

---

## Overview

PentAGI automates the full penetration testing lifecycle using LLM-orchestrated agents, MCP tools, and multi-agent workflows. This study examines the security posture of PentAGI itself when deployed in a containerized environment — the tool-testing-the-tool scenario.

**Key question:** If an AI security agent runs loose in your environment, what can it reach and what does it actually do?

---

## Findings Summary

### Static Analysis

| Severity | Count | Key Finding |
|----------|-------|-------------|
| CRITICAL  | 4     | docker.sock exposure, root execution, NET_ADMIN capability, 1,144 Docker API calls |
| HIGH      | 2     | Unrestricted filesystem access, host network exposure |
| MEDIUM    | 2     | Missing resource limits, debug interface exposure |

### Dynamic Analysis — Phase 2.2 (Behavioral)

| Metric | Value |
|--------|-------|
| Requests analyzed | 274 |
| Threat rate | **73.7%** |
| EXFILTRATION events | **462** (env vars, filesystem, credential probing) |
| PROMPT_INJECTION attempts | **24** |
| Docker API calls (1 session) | **1,144** |

---

## Critical Findings

### 1. Docker Socket Exposure (`/var/run/docker.sock`)
Mounting the Docker socket gives the container — and the LLM driving it — full control over the host Docker daemon. A successful prompt injection achieves host escape.

### 2. Root Container Execution
All PentAGI containers run as root with no user namespace isolation. Combined with the Docker socket, this is a direct path to full host compromise.

### 3. NET_ADMIN Capability
Grants the container full access to the host network stack: traffic interception, routing manipulation, and ARP spoofing against adjacent containers.

### 4. Prompt Injection → Container Escape Chain
Static analysis confirmed 24 injection-surface endpoints. A weaponized injection payload can direct the LLM to spin up a new privileged container mounting the host filesystem.

---

## Reports

| File | Description |
|------|-------------|
| [`PENTAGI_CASE_STUDY_BRANDING.html`](PENTAGI_CASE_STUDY_BRANDING.html) | Full branded report — print-ready, all charts and visualizations |
| [`PENTAGI_CASE_STUDY.html`](PENTAGI_CASE_STUDY.html) | Compact research report |

> Open in a modern browser. Reports use Chart.js for visualizations (loaded from CDN).

---

## Methodology

**Phase 1 — Static Analysis**
- Dockerfile + docker-compose.yml capability audit
- Go source code pattern analysis: `exec.Command` calls, Docker API usage, filesystem ops, secret references
- Dependency scanning with Trivy
- Container privilege matrix evaluation

**Phase 2 — Dynamic Analysis**
- Falco behavioral monitoring during live agent sessions
- API call pattern classification (Anthropic API, Docker API, filesystem)
- Behavioral threat event taxonomy: EXFILTRATION, PROMPT_INJECTION, PRIVILEGE_ESCALATION, LATERAL_MOVEMENT
- Prompt injection surface mapping (direct + indirect vectors)

---

## Responsible Disclosure

This research was conducted on the publicly available PentAGI codebase and default Docker Compose deployment. No production systems or live endpoints were targeted. Findings relate to the default configuration as shipped.

Disclosure timeline: findings documented April 2026.

---

## Tools Used

| Tool | Purpose |
|------|---------|
| [Trivy](https://github.com/aquasecurity/trivy) | Container + dependency scanning |
| [Falco](https://falco.org/) | Runtime behavioral monitoring |
| [pq-audit](https://github.com/mk-scorpiosec/pq-audit) | Post-quantum cryptography layer |
| Custom Falco rules | EXFILTRATION + PROMPT_INJECTION classification |

---

## Related

- [Case Study #1 — TerraGoat IaC Analysis](../terragoat-2026-04/)
- [MK ScorpioSec Research](https://github.com/mk-scorpiosec/research)
