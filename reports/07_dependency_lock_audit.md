# Task 07 — Dependency Lock Generation and Static Security Review

**Audit date/time:** 2026-10-28T22:15:54+05:30  
**Checkout:** `D:\document\job apply\tools\career-ops`  
**Upstream pin:** `a6d7526c05861b1df9b70c011f8b1a84c44252ac`  
**Decision:** **CONDITIONAL** — lock generation and the read-only npm advisory check completed, but installation remains gated on approval of the ignored lockfile/dependency policy and install lifecycle behavior.

## 1. Executive summary and decision

Generated the root `package-lock.json` using the exact authorized package-lock-only command with lifecycle scripts disabled. The lock records **7 resolved dependency packages** (5 direct, 2 transitive; 8 package entries including the root package), lockfile version 3, and only `registry.npmjs.org` tarball hosts. All resolved packages have integrity hashes. No `node_modules/` was created; Chromium was not invoked or installed; no project script was run.

`npm audit --package-lock-only --ignore-scripts --no-fund` reported **0 vulnerabilities** at audit time. This is not a supply-chain/security guarantee. The new lockfile is ignored by the repository’s existing `.gitignore`, so it is local-only and will not appear in normal Git status or follow another clone. The root `package.json` stayed byte-identical to the preflight hash; HEAD and origin stayed unchanged; candidate file hashes still match.

**Conditional blocker:** resolve whether this lockfile will remain a local install artifact or be preserved in a future authorized version-control change. Also explicitly approve or defer Chromium: a future ordinary `npm ci` runs the root `postinstall` unless `--ignore-scripts` is used, and that postinstall attempts a browser download. Do not install yet.

## 2. Preflight verification

Preflight ran in `D:\document\job apply` before dependency resolution.

| Check | Result |
|---|---|
| Working directory | Exact match: `D:\document\job apply` |
| Career-ops HEAD | `a6d7526c05861b1df9b70c011f8b1a84c44252ac` — exact expected commit |
| Origin | `https://github.com/career-ops-hq/career-ops.git` |
| Tracked working tree | Clean before resolution |
| `package.json` Git blob vs. checked-out file | Match (`10a67128f6f01b4c1baf396f50e79c965d162718`) |
| `package.json` SHA-256 before resolution | `10ca52815f65e4ba14ea4025828f2f9b212d1b684a2ab495a173d3a461df1fac` |
| `node_modules/` | Absent |
| `package-lock.json` | Absent |
| `npm-shrinkwrap.json` | Absent |
| Existing Task 07 report | Absent; no overwrite required |

Candidate source-file preflight hashes also matched before any resolution. Postflight results are in §10.

## 3. Dependency-resolution command and outcome

Run from the verified repository directory, with npm `10.9.2` and Node `v22.17.1`:

```text
npm install --package-lock-only --ignore-scripts --no-audit --no-fund --lockfile-version=3
```

**Outcome:** exit success; npm reported `up to date in 6s`. This was the sole dependency-resolution operation. No install script was run, no package was placed in `node_modules/`, and no Chromium install command was invoked. The operation necessarily contacted the effective npm registry for package metadata/resolution; the effective registry hostname was `registry.npmjs.org`. Registry URL userinfo, credentials, and secret values were neither requested nor printed. The later audit also used the registry; no packages were installed.

## 4. Lockfile statistics and SHA-256

| Statistic | Result |
|---|---:|
| Root package | `career-ops` |
| Root version | `1.38.0` |
| Lockfile format | v3 |
| Package entries | 8 total: root + 7 resolved packages |
| Direct dependencies | 5 |
| Transitive dependencies | 2 |
| Optional dependency entries | 0 |
| `hasInstallScript: true` markers | 1, on the root package entry |
| Resolved tarball hosts | `registry.npmjs.org` only (7 entries) |
| Git/local/file dependency sources | 0 |
| Resolved packages with integrity values | 7 of 7; all SHA-512 |
| Generated lockfile size | 3,732 bytes |
| Effective npm registry hostname | `registry.npmjs.org` (credentials/config-file contents not read or printed) |
| **`package-lock.json` SHA-256** | **`cb14c2cd24a3676514af291914db76ace250a8bbf2d6708006f02300028c22cc`** |

## 5. Direct and important transitive dependencies

| Direct dependency | Manifest request | Resolved version | Important dependency chain / note |
|---|---|---|---|
| `@google/generative-ai` | `^0.24.1` | `0.24.1` | No dependency chain recorded in this lock. |
| `dotenv` | `^18.0.4` | `18.0.9` | Reads environment/configuration when explicitly used; do not expose secret values. |
| `js-yaml` | `^5.3.0` | `5.5.0` | `js-yaml → argparse 3.0.4` (transitive CLI dependency). |
| `playwright` | `1.64.1` | `1.64.1` | Exact manifest match; `playwright → playwright-core 1.64.1`. Browser binaries are separate and are not represented as an installed package here. |
| `undici` | `^7.30.1` | `7.30.1` | No dependency chain recorded in this lock. |

**Version-range check:** all resolutions satisfy the pinned manifest ranges; Playwright is exactly the declared version. The lock is internally generated from the pinned `package.json`, whose hash is unchanged. This resolves dependency versions for this local checkout, but the lock is ignored by Git (see §8).

## 6. Registry/source and lifecycle-script findings

- All seven package tarball URLs resolve from `registry.npmjs.org` over HTTPS. No GitHub/Git dependency, local path/file dependency, alternate host, or unexpected registry appeared in the lock.
- The lock has no optional or platform-specific packages. No native-build dependency was identified among the seven resolved packages.
- The lock metadata flags only the **root package** with `hasInstallScript: true`; no installed dependency entry is flagged that way. The lock does not contain dependency lifecycle-script bodies, so this is not a source audit of third-party package scripts.
- The lockfile v3 root metadata matches the unchanged package manifest, so the lock is structurally suitable for a future `npm ci` in this checkout. `npm ci` was **not run**; by default it would execute the root postinstall and attempt Chromium installation. A separately approved `npm ci --ignore-scripts` would avoid lifecycle scripts but leave browser-dependent features unavailable until a distinct browser-download approval.
- The root `package.json`’s `postinstall` is known from Task 06: unless its skip flag is set, it attempts `npx playwright install chromium` twice, then warns on failure. The authorized command’s `--ignore-scripts` prevented that root lifecycle script from running in Task 07.
- `package-lock-only` produced metadata/lock resolution, not package files: postflight confirms `node_modules/` is still absent. No Chromium command was invoked, and no browser cache/profile was inspected. Thus this operation did not install Chromium; a future standard install may do so unless explicitly suppressed/approved.
- The current manifest has no root lockfile before this task and `.gitignore` ignores `package-lock.json`. This lock is generated output, not evidence that install scripts or runtime behavior are safe.

## 7. npm audit findings or limitation

Command run after lock generation:

```text
npm audit --package-lock-only --ignore-scripts --no-fund
```

**Result:** `found 0 vulnerabilities` (registry advisory data available at audit time). No fix, upgrade, or install operation was run. This only means the current audit service reported no matching advisories for this lockfile; it does not establish package provenance, absence of unknown vulnerabilities, safety of lifecycle scripts, or safety of the application.

## 8. Reproducibility and ignored-lockfile implications

`package-lock.json` remains ignored by the pinned `.gitignore`. `git status` is clean and does not display the lockfile; `git check-ignore` confirms the ignore rule. The lock exists locally and can support a later `npm ci` **in this checkout** after explicit authorization, but another clone will not receive it. Reproducibility across machines/clean checkouts therefore requires a separately authorized decision to preserve/version the lock (for example, changing the ignore policy or explicitly force-adding it). No `.gitignore` change or force-add was made here.

## 9. Files created or modified

- **Created:** `tools/career-ops/package-lock.json` — only authorized project artifact; ignored by Git.
- **Created:** `reports/07_dependency_lock_audit.md` — this report.
- **Unchanged:** `package.json` SHA-256 remains `10ca52815f65e4ba14ea4025828f2f9b212d1b684a2ab495a173d3a461df1fac`; `.gitignore`, upstream HEAD, and origin are unchanged.
- **No `node_modules/`**, no lockfile backup, no auxiliary script/report, and no other project file change was observed. Tracked Git status remains clean; the ignored lock will not appear in it.

## 10. Candidate-source integrity results

Postflight computed SHA-256 from raw bytes after lock generation and npm audit:

| Candidate file | Expected SHA-256 | Actual SHA-256 | Result |
|---|---|---|---|
| `full-profile.json` | `2f767145d8cb3bb619f0e77ff5f9f0e804a6026e1d2122e7b3fa50dcc9161fdb` | `2f767145d8cb3bb619f0e77ff5f9f0e804a6026e1d2122e7b3fa50dcc9161fdb` | **PASS** |
| `internships.json` | `606e03d0d98d783a00c0afb002a08dc1958b065ea245a9b232fee8b0d5da2aea` | `606e03d0d98d783a00c0afb002a08dc1958b065ea245a9b232fee8b0d5da2aea` | **PASS** |
| `projects.json` | `86749dc8d6f1aa9523e6a57f1decf3a74c911e18eb3ed3ee2000f16b8725e4f2` | `86749dc8d6f1aa9523e6a57f1decf3a74c911e18eb3ed3ee2000f16b8725e4f2` | **PASS** |

## 11. Remaining blockers and next authorization needed

**Readiness decision: CONDITIONAL.** The lockfile resolution and advisory query succeeded; no blocking vulnerability was reported. Before installing dependencies, the owner must decide:

1. Whether to preserve the ignored lockfile for future clean checkouts; if yes, authorize a separate `.gitignore`/version-control change. If no, treat this lockfile as a local-only artifact and do not claim clean-clone reproducibility.
2. Whether to authorize a future `npm ci` and its lifecycle scripts. A normal `npm ci` runs the root postinstall, which may download Playwright Chromium; if scripts are suppressed, PDF/browser checks remain unavailable until a separate browser-download approval.
3. Whether to run dependency advisory checks again later; advisories and registry metadata change over time.

**Stopping confirmation:** no dependency install, project/test command, browser download, application startup, external job API call, or candidate-data edit was performed. Stop here; Task 08 or any installation requires separate authorization.
