# Targets

## How packages are chosen

Each candidate gets a priority score:

```
priority = reach × open vulnerabilities × critical dependents
```

- **Reach:** registry downloads over a recent window (npm weekly downloads, PyPI downloads, crates.io downloads), log-scaled so one giant package doesn't drown out everything else.
- **Open vulnerabilities:** published advisories (GitHub Advisory Database, OSV, NVD) with no patched version, weighted by severity.
- **Critical dependents:** how many widely used or security-sensitive packages depend on it, directly or transitively.

A package only qualifies if it is verifiably unmaintained: officially deprecated by its maintainers, archived, or with no release and no maintainer response to security reports for an extended period. Every listed target links to the evidence.

The score sets order, not adoption. The human steward makes the final call.

## Queue

| # | Package | Ecosystem | Status | Evidence |
|---|---------|-----------|--------|----------|
| 1 | `request` | npm | **In progress** | Deprecated 2020 ([request#3142](https://github.com/request/request/issues/3142)); last release 2.88.2 (Feb 2020); [CVE-2023-28155](https://nvd.nist.gov/vuln/detail/CVE-2023-28155) affects ≤ 2.88.2 with no patched release ([GHSA-p8p7-x288-28g6](https://github.com/advisories/GHSA-p8p7-x288-28g6)); ~13.6M downloads/week (npm, Nov 16–22, 2026) |

Further targets are added only after their maintenance status and open advisories are verified, with sources linked here.
