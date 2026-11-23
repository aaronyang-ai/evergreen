# Rescue lane #1: `request`

Regression test for **CVE-2023-28155** ([GHSA-p8p7-x288-28g6](https://github.com/advisories/GHSA-p8p7-x288-28g6)) against `request@2.88.2`, the last published version.

## Run it

Requires Node 18 or newer.

```sh
cd rescues/request
npm ci
npm test
```

## What it checks

Everything runs on `127.0.0.1`; no network access is needed.

- Two "internal" servers (HTTP and HTTPS) stand in for services an app must never reach, and count every hit.
- An "attacker" HTTP server answers with a `302` pointing at one of them.
- The client calls `request` with a guard `agent` that refuses connections to the internal ports, the way SSRF-filtering agents work.

| Test | Expected on request@2.88.2 |
|------|----------------------------|
| control: same-protocol redirect (http → http) is blocked by the guard | **passes** |
| CVE-2023-28155: cross-protocol redirect (http → https) must not bypass the guard | **fails** |

The second test fails because `lib/redirect.js` runs `delete request.agent` whenever a redirect changes protocol. The guard agent is dropped, Node's default agent is used, and the internal HTTPS server answers with `200 internal-secret`.

**This failure is intentional.** It is the target for the fix.

## Definition of done

- The CVE test passes without weakening it.
- The control test still passes.
- Cross-protocol redirects no longer discard a caller-supplied agent by default. The behavior of upstream's proposed fix ([request#3444](https://github.com/request/request/pull/3444)) is the reference: an explicit `allowInsecureRedirect` opt-in. Locally, disabling the `delete request.agent` path makes both tests pass.
- request's original test suite still passes, with any intentional behavior change documented.

`test/fixtures/localhost.{key,crt}` is a self-signed certificate used **only** by the local test server. It is not a secret.
