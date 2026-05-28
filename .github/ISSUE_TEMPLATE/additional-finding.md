---
name: Additional Finding
about: You found a security issue in the research target that isn't covered in this
  study
title: "[FINDING]"
labels: enhancement, security
assignees: ''

---

```markdown
---
name: Additional Finding
about: You found a security issue in the research target that isn't covered in this study
title: '[FINDING] '
labels: enhancement, security
assignees: ''
---

## Study / Target
<!-- e.g., TerraGoat terragoat-2026-04 -->

## Finding Description
<!-- Describe the security issue -->

## Severity
- [ ] CRITICAL (CVSS 9.0+)
- [ ] HIGH (CVSS 7.0–8.9)
- [ ] MEDIUM (CVSS 4.0–6.9)
- [ ] LOW / INFO

## Scanner / Tool That Detected It
<!-- Trivy, Checkov, pq-audit, manual, other -->

## Evidence
<!-- Paste relevant scan output or config snippet (sanitized) -->

## File / Resource
<!-- e.g., terraform/aws/eks.tf:42 -->
```
