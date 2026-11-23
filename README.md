# Evergreen

**A public lifeboat for the world's abandoned open-source software.**

Every modern app stands on thousands of packages, many maintained by one tired person or by no one at all. When a maintainer walks away, known vulnerabilities stay unpatched, builds rot, and everything downstream quietly inherits the risk. Humans can't keep up with that much maintenance. Agents can, if a human sets the rules and signs every release.

Evergreen adopts critical, unmaintained packages and brings them back to a known-good state, in public.

## How it works

For each adopted package, an agent works a rescue lane:

1. **Reproduce** the build from source, pinned and deterministic.
2. **Prove the problem** with a failing regression test for each known vulnerability.
3. **Fix** it with the smallest reviewable change that makes the test pass.
4. **Ship** the fix upstream as a PR where a maintainer is still reachable, or as a signed, clearly labeled fork where none is.
5. **Publish a migration path** so downstream projects can move to the fix, or off the package entirely, without guesswork.

Every change is a public, reviewable diff. Nothing is released without a human signature. See [TRUST_MODEL.md](TRUST_MODEL.md).

## First target: `request`

[`request`](https://github.com/request/request) is the classic npm HTTP client. It has been deprecated since 2020 ([request#3142](https://github.com/request/request/issues/3142)) and its last release, 2.88.2, shipped in February 2020. It still sees roughly 13.6 million downloads a week (npm, Nov 16–22, 2026), mostly as a transitive dependency.

It carries **[CVE-2023-28155](https://nvd.nist.gov/vuln/detail/CVE-2023-28155)** ([GHSA-p8p7-x288-28g6](https://github.com/advisories/GHSA-p8p7-x288-28g6), CVSS 3.1: 6.1 Medium): an attacker-controlled server can bypass SSRF mitigations with a cross-protocol redirect (HTTP to HTTPS, or HTTPS to HTTP). The advisory lists every version through 2.88.2 as affected, with no patched release. A fix was proposed upstream in [request#3444](https://github.com/request/request/pull/3444), but it has never been released.

**Root cause:** on a redirect that changes protocol, `lib/redirect.js` runs `delete request.agent`. SSRF defenses that work by passing a filtering `agent` are silently dropped, and the follow-up request goes out unfiltered.

The regression test lives in [`rescues/request/`](rescues/request/). It currently **fails on purpose**: it proves that request@2.88.2 reaches an "internal" service through a cross-protocol redirect even when the caller passed a guard agent. Making it pass is the job.

> **CI is red by design.** The `rescue / request` workflow stays red until the fix lands. A green run is the week-one deliverable.

## Roadmap

**Week 1: `request`, frozen to green**
- Reproducible build of request@2.88.2 with its original test suite running in CI.
- CVE-2023-28155 fixed, with the regression test in this repo passing.
- Fix shipped as a signed, clearly labeled fork release, plus a migration guide: how to pick up the fix, and how to move off `request` entirely.

**Weeks 2–8: parallel rescue lanes**
- Several agents work rescue lanes side by side across npm, PyPI, and crates.io.
- Targets ranked by the method in [TARGETS.md](TARGETS.md): reach × open vulnerabilities × critical dependents.
- Every lane follows the same playbook, and every release goes through the same human sign-off.
- A public changelog per package, explaining every line that changed and why.

## The human steward

Evergreen is agent-operated and human-governed. The steward:

- picks which packages are adopted,
- writes and maintains the safety rules in [TRUST_MODEL.md](TRUST_MODEL.md),
- reviews and signs every release,
- holds the signing keys. Agents never do.

## Repository layout

```
README.md          this file
TRUST_MODEL.md     what agents may do alone, and what needs a human
TARGETS.md         how packages are chosen; current queue
rescues/request/   rescue lane #1: regression test for CVE-2023-28155
.github/workflows/ CI (intentionally red until the fix lands)
```

## License

[MIT](LICENSE)
