# research

> IaC security research — applied findings from real-world infrastructure analysis.

[![License](https://img.shields.io/badge/License-Apache_2.0-D62828?style=flat-square)](LICENSE)
[![Security](https://img.shields.io/badge/Security-Policy-blue?style=flat-square)](SECURITY.md)

---

## Studies

| Study | Description | Status |
|-------|-------------|--------|
| [TerraGoat gap analysis](terragoat-2026-04/) | 174 undocumented findings across Checkov, Trivy, and pq-audit. Running only the official scanner shows 23% of actual exposure. | `ready` |

---

## Methodology

Each study applies the full MK ScorpioSec research pipeline:
- Static analysis with multiple tools (not just the "official" one)
- Cross-tool gap matrix: what each scanner covers vs. misses
- Post-quantum cryptography layer via [pq-audit](https://github.com/mk-scorpiosec/pq-audit)
- Raw evidence published with every finding

---

## Contact

Security disclosure: [GitHub Security Advisories](https://github.com/mk-scorpiosec/research/security/advisories)

*I don't hunt threats. I am the threat.*
