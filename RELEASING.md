# Releasing flexaudio

Releases are cut by pushing a version tag (e.g. `v0.3.0`), which triggers the
three workflows in `.github/workflows/release-*.yml`.

```bash
git tag v0.3.0
git push origin v0.3.0
```

## 0.3.0 — not yet released

0.3.0 is not published. Tag `v0.3.0` only when the remaining work is done; the
0.2.0 registry-status table below stays as the last published snapshot until
then.

## Registry status — 0.2.0

| Registry | Status | Notes |
|---|---|---|
| **crates.io** | ✅ published | All nine crates. |
| **PyPI** | ✅ published | Wheels (Linux x64/arm64, macOS arm64, Windows x64) + sdist. |
| **npm** | ⏳ **pending** | Blocked by an npm-side bug — see below. Re-run `release-npm.yml` to finish. |

Prebuilt npm binaries follow `release-npm.yml`'s build matrix, which now covers
Linux x64/arm64, macOS arm64 (Apple Silicon) **and x64 (Intel)**, and Windows
x64 **and arm64** — six targets. (The PyPI wheel set in the table above is the
0.2.0 snapshot and is narrower.)

## npm is not published yet — how to finish it

The npm packages (`@studio-sadola/flexaudio` + the per-platform packages) build
correctly in CI but cannot be published from CI right now. This is an npm
platform issue, not a problem with this repo:

- Publishing a package requires **2FA or a granular access token with "Bypass
  2FA" enabled**. The "Bypass 2FA" token feature is currently broken
  (npm/cli [#8869](https://github.com/npm/cli/issues/8869),
  [#9268](https://github.com/npm/cli/issues/9268) — both open).
- **Trusted publishing (OIDC) cannot bootstrap a brand-new package** — npm has
  no "pending publisher" equivalent yet (npm/cli
  [#8544](https://github.com/npm/cli/issues/8544)), so the first version can't
  be published over OIDC.

**To finish once npm ships a fix:** the `NPM_TOKEN` secret and the workflow are
already in place. When "Bypass 2FA" tokens work, re-create `NPM_TOKEN` as a
granular token with Bypass 2FA enabled and re-run `release-npm.yml`. After the
first successful publish, switch to OIDC trusted publishing (configure it per
package at `npmjs.com/package/<name>/access`) for subsequent releases.

Alternatively, the first version can be published interactively from a machine
with 2FA, using the `.node` binaries produced by the `release-npm.yml` build job.

## lightningrodlabs fork releases

`lightningrodlabs/flexaudio` (this fork) publishes to npm under the
`@lightningrodlabs` scope while upstream's own npm publication is blocked (see
above). Fork releases are versioned `<upstream-version>-lrl.N` — e.g.
`0.3.0-lrl.1` is the first fork release built from upstream's unreleased
`0.3.0` — so a fork version always sorts after the upstream version it tracks
and `N` increments for a fork-only respin without waiting on upstream.

**`0.3.0-lrl.N` is a SemVer prerelease — consumers must pin it exactly.** A
range does not match prereleases: `"^0.3.0"` and `"~0.3.0"` both resolve to
nothing here. Depend on it as

```json
"@lightningrodlabs/flexaudio": "0.3.0-lrl.1"
```

**Dist-tag:** npm refuses to publish a prerelease without an explicit
`--tag` (the first dry run failed on exactly that), so the workflow passes
`--tag latest` on every publish. A fork release therefore lands on **`latest`**
for `@lightningrodlabs/flexaudio` and its platform packages: `npm install
@lightningrodlabs/flexaudio` resolves to it (npm follows the dist-tag), and a
separate `lrl` tag would leave `latest` empty. `--tag lrl` is NOT used.

Before tagging a release, validate the workflow with a dry run (builds every
platform, skips the actual `npm publish`):

```bash
gh workflow run "Release (npm)" --repo lightningrodlabs/flexaudio --ref lightningrodlabs/publish -f dry_run=true
```

After a real publish, verify the registry directly — never trust the workflow
log:

```bash
npm view @lightningrodlabs/flexaudio version
```

| Version | Date | Notes |
|---|---|---|

