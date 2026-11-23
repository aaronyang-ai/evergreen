# Trust model

Evergreen changes code that other people's software depends on. The rules below decide what an agent may do on its own and what needs a human. When in doubt, the answer is "needs a human."

## Agents may do autonomously

- Open branches and pull requests in Evergreen repositories.
- Write and run tests, including failing regression tests that reproduce a known vulnerability.
- Reproduce builds and pin toolchains.
- Run and fix CI configuration.
- Draft changelogs, migration guides, and security notes.
- Open issues or PRs on an upstream project, as long as they clearly identify themselves as coming from Evergreen and an agent.

## Requires human sign-off

- **Any release.** Tagging, building release artifacts, or cutting a version.
- **Publishing to a registry** (npm, PyPI, crates.io, or any other).
- **Dependency changes in the trusted base.** Adding, removing, or upgrading a runtime dependency of a rescued package, or anything in the build and release toolchain.
- **Anything touching signing keys or credentials.** Agents never hold or request signing keys. Registry and signing credentials belong to the human steward alone.
- **Adopting a new package** or retiring one.
- **Changes to this trust model.**

A sign-off is a signed commit or tag by the steward on the exact artifact being released. Approving a PR is not a release approval.

## Provenance and signing

- Every release is built in CI from a tagged commit, never on a developer machine.
- Release artifacts are signed, and build provenance is published alongside them (for npm, registry provenance attestations).
- Each release links to the PRs it contains, the advisories it fixes, and the regression tests that prove the fixes.

## No silent takeovers

- Evergreen does not request ownership of an existing package name unless the original maintainers agree in public, or the registry's own documented process for abandoned packages is followed.
- No name squatting and no look-alike package names.
- Upstream first: if a maintainer is reachable, the fix goes to them as a PR before any fork release.

## Fork labeling

When a fork release is necessary, it is unmistakable:

- Published under an Evergreen scope or clearly prefixed name, never the original name.
- The README opens with: what this fork is, which upstream version it is based on, exactly what changed, and the advisories it fixes.
- Behavior changes are listed explicitly. Where a security fix changes behavior (for example, refusing cross-protocol redirects by default), the opt-out is documented along with its risk.
- Every fork ships a migration guide, including how to leave the fork.
