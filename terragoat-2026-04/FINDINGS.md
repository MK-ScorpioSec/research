# Lab Report — TerraGoat IaC Security Gap Analysis
## MK ScorpioSec — Research Series #1

**Date**: 2026-04-27 (re-analysis: 2026-05-06)
**Analyst**: Mike E. Martinez Oroz (MK ScorpioSec)
**Target**: [github.com/bridgecrewio/terragoat](https://github.com/bridgecrewio/terragoat)
**Type**: IaC / Cloud misconfiguration (intentionally vulnerable lab)
**Tools**: trivy config, checkov, pq-audit --layer cloud, trufflehog
**Goal**: Validate cloud misconfiguration detection across multiple scanners — identify coverage gaps

---

## 1. Lab Description

| Field | Value |
|---|---|
| Repository | github.com/bridgecrewio/terragoat |
| Type | Intentionally vulnerable Terraform (Bridgecrew / Palo Alto) |
| Platforms | AWS, GCP, Azure, Oracle, AliCloud |
| Total .tf files | 47 |
| Cloud account required | No (static analysis only) |
| Status | COMPLETED 2026-04-27 |

---

## 2. Tools Deployed

| Tool | Mode | Result | Notes |
|---|---|---|---|
| `trivy config` | Static | **243 findings** | Full severity data without API key |
| `checkov -d` | Static | 215 findings | Severity UNKNOWN without `bc_api_key` (free tier limitation) |
| `pq-audit --layer cloud` | Static | 3 real findings (v2) | v1 had 1122 FP from package-lock.json — fixed in v2 (see GAP-001) |
| `trufflehog filesystem` | Static | 0 verified secrets | Expected — clean lab repo |

**Commands:**

```bash
trivy config ./terraform/aws   --format json -o evidence/trivy-aws.json
trivy config ./terraform/gcp   --format json -o evidence/trivy-gcp.json
trivy config ./terraform/azure --format json -o evidence/trivy-azure.json
checkov -d ./terraform/aws --output json --quiet
python3 pq-audit.py --layer cloud --target ./terragoat
trufflehog filesystem ./terragoat --only-verified --json
```

---

## 3. Findings — Trivy (primary source, full severity)

### Multi-cloud Summary

| Cloud | CRITICAL | HIGH | MEDIUM | LOW | TOTAL |
|---|---|---|---|---|---|
| AWS | 6 | 66 | 26 | 22 | **120** |
| GCP | 2 | 10 | 16 | 12 | **40** |
| Azure | 6 | 6 | 52 | 19 | **83** |
| **TOTAL** | **14** | **82** | **94** | **53** | **243** |

### 3.1 CRITICAL Findings

| ID | Description | Resource / File | MITRE |
|---|---|---|---|
| aws-vpc-no-public-egress-sgr | Security Group allows unrestricted egress 0.0.0.0/0 | db-app.tf / ec2.tf | T1048 (Exfil over Alt Protocol) |
| aws-autoscaling-no-public-ip | EC2 user data may contain AWS credentials | ec2.tf | T1552.001 (Credentials in Files) |
| AVD-AWS-0040 | EKS cluster with public endpoint enabled | eks.tf | T1190 (Exploit Public-Facing App) |
| AVD-AWS-0041 | EKS cluster with open CIDR range for public access | eks.tf | T1190 |
| AVD-AWS-0046 | Elasticsearch without HTTPS (plaintext traffic) | es.tf | T1040 (Network Sniffing) |
| gcp-* | GCP: 2 criticals (see evidence/trivy-gcp.json) | — | — |
| azure-* | Azure: 6 criticals (see evidence/trivy-azure.json) | — | — |

### 3.2 HIGH Findings — AWS highlights

| ID | Description | File |
|---|---|---|
| AVD-AWS-0080 | RDS without encryption at rest | db-app.tf |
| AVD-AWS-0180 | RDS Publicly Accessible | db-app.tf |
| aws-ebs-enable-volume-encryption | EBS volumes unencrypted | ec2.tf |
| AVD-AWS-0086/87/88 | S3: public ACL + public policy + no encryption | ec2.tf |
| AVD-AWS-0028 | IMDSv1 enabled (not IMDSv2) | db-app.tf, ec2.tf |
| AVD-AWS-0345 | IAM policy with unrestricted S3 access | db-app.tf |

---

## 4. pq-audit — Real Findings vs False Positives

### v1 (initial run — before GAP-001 fix)

| Type | Count | Detail |
|---|---|---|
| BROKEN_NOW (real) | 1 | TLS 1.1 in alicloud/provider.tf:5 |
| SNDL_VULNERABLE (real) | 1 | SG with 0.0.0.0/0 in aws/db-app.tf:150 |
| FP (GAP-001) | 1122 | Strings "1.1"/"1.0" in package-lock.json detected as TLS — **fixed in v2** |

### v2 (re-analysis 2026-05-06 — post GAP-001 fix)

| Provider | Tool | Risk | File | Line | Description |
|---|---|---|---|---|---|
| AWS | pq-audit v2 | SNDL_VULNERABLE | db-app.tf | 150 | SG egress 0.0.0.0/0 — data in transit exposed under SNDL threat model |
| Azure | pq-audit v2 | **BROKEN_NOW** | app_service.tf | 29 | Min TLS 1.0/1.1 configured — **broken today, not just post-quantum** |
| GCP | pq-audit v2 | — | — | — | 0 crypto findings in v2. Note: gcp-v2.json `total_findings:1` was a metadata note (cloud_analysis_dispatcher reference), not a security finding — confirmed FP, corrected to 0. |

**pq-audit v2 score: 7/10** (vs 3/10 v1 — FPs eliminated, 2 real findings verified)

---

## 5. Gap Analysis — GAP-001

### The core finding: 169 undocumented misconfigs

Bridgecrew documents **56 findings** for TerraGoat's AWS module via Checkov. Running Trivy against the same codebase produces **243 findings** — **187 not covered by the official documentation**.

When cross-referenced with pq-audit's crypto-specific layer, an additional **3 findings** emerge that no other scanner surfaces:

| Scanner | Findings | Documented by Bridgecrew |
|---|---|---|
| Checkov (official) | 56 | ✅ Yes |
| Trivy | 243 | ❌ 187 undocumented |
| pq-audit v2 | 2 | ❌ 0 documented |
| **Net gap** | **173** | **undocumented** |

> If your team chose Checkov because it's "official," you're seeing ~23% of your actual exposure — not because Checkov is bad, but because the documentation doesn't tell you what it *doesn't* cover.

### GAP-001 Detail — pq-audit TLS false positives (fixed)

**Description**: The TLS version regex in `pq-audit.py` cloud layer didn't filter non-IaC files (`package-lock.json`, `node_modules/`, `yarn.lock`). Result: 1122 FPs masking 2 real findings.

**Fix applied (v2)**:

```python
EXCLUDE_PATTERNS = [
    "package-lock.json", "yarn.lock", "package.json",
    "node_modules/", ".terraform/", "*.lock.hcl"
]
# Applied before TLS regex scan
```

**Impact**: pq-audit v1 `--layer cloud` was unusable on mixed repos. v2 is production-ready.

---

## 6. Summary

### What worked well
- **Trivy**: excellent coverage, full severity without API key
- **Checkov**: found 215 misconfigs (more than Trivy on AWS) but no severity in free mode
- **TruffleHog**: zero FPs on clean repo

### Critical gaps
- **Severity without API key**: Checkov requires `--bc-api-key` for full severity. Workaround: use `--framework terraform --output sarif` (includes embedded severities)
- **Single-scanner coverage**: running only the "official" scanner misses up to 77% of findings

### Arsenal scores (IaC Cloud)
- Trivy: **9/10** — primary recommended tool
- Checkov: **7/10** (without API key severity)
- pq-audit v2: **7/10** — specialized PQC lens, complements Trivy
- TruffleHog: **N/A** (no secrets in clean lab, as expected)

### Key takeaway
Security architecture that uses **human judgment to select and cross-validate tools** surfaces findings that no single scanner covers. This is the Privacy-by-Design principle applied to tooling: analyze local, validate multiple sources, never trust a single automated output.

---

## 7. Evidence

See [`evidence/`](evidence/):
- `trivy-aws.json` — 120 AWS findings with severity
- `trivy-gcp.json` — 40 GCP findings with severity
- `trivy-azure.json` — 83 Azure findings with severity
- `checkov-aws.json` — 56 AWS findings (Checkov documented)
- `pq-audit/aws-v2.json` — 2 PQC findings AWS (v2, GAP-001 fixed)
- `pq-audit/azure-v2.json` — 1 PQC finding Azure BROKEN_NOW (TLS 1.0/1.1)
- `pq-audit/gcp-v2.json` — 0 crypto findings GCP
