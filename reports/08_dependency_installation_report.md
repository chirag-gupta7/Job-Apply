# Task 08 — Controlled Dependency Installation Report

**Date:** 2026-10-29  
**Working directory:** `D:\document\job apply`  
**Repository:** `D:\document\job apply\tools\career-ops`  
**Pinned revision:** `a6d7526c05861b1df9b70c011f8b1a84c44252ac`  
**Decision:** **INSTALLED AND VERIFIED**

## 1. Executive summary

Installed the reviewed dependency tree with `npm ci --ignore-scripts --no-audit --no-fund` from the verified checkout. npm reported `added 7 packages in 49s` with exit status 0. All five direct dependencies resolve to the versions recorded in the Task 07 lockfile, including Playwright `1.64.1`. Lifecycle scripts were suppressed, so the root `postinstall` did not run and no Chromium download was triggered. No project command, test, scanner, or application startup was executed.

Pre- and post-checks confirm: `package.json` and `package-lock.json` hashes unchanged; Git HEAD, origin, and tracked tree unchanged; no new non-ignored files; all three candidate JSON hashes unchanged. A repo-local Playwright browser directory was checked and is absent; the global Playwright browser cache was intentionally not inspected, per the task boundary.

## 2. Preflight results

| Check | Result |
|---|---|
| Working directory | `D:\document\job apply` — exact match |
| Repository HEAD | `a6d7526c05861b1df9b70c011f8b1a84c44252ac` — exact expected commit |
| Origin | `https://github.com/career-ops-hq/career-ops.git` |
| Tracked working tree | Clean |
| `package.json` SHA-256 | `10ca52815f65e4ba14ea4025828f2f9b212d1b684a2ab495a173d3a461df1fac` — matched Task 07 baseline |
| `package-lock.json` SHA-256 | `cb14c2cd24a3676514af291914db76ace250a8bbf2d6708006f02300028c22cc` — matched Task 07 value |
| Lockfile structure | `lockfileVersion: 3`, root name `career-ops`, Playwright `1.64.1` entry |
| `node_modules/` before install | Absent (no deletion or overwrite required) |
| Candidate hashes | All three PASS (exact values in §9) |
| Existing Task 08 report | Absent |

`npm ci` itself validates that the lockfile is consistent with `package.json`; it completed without an `ERESOLVE`/sync error, which is the runtime confirmation of lock–manifest agreement.

## 3. Installation command and exit status

**Command (run from `D:\document\job apply\tools\career-ops`):**

```text
npm ci --ignore-scripts --no-audit --no-fund
```

| Observation | Result |
|---|---|
| Exit status | 0 |
| npm output | `added 7 packages in 49s` |
| Lockfile regenerated? | No — SHA-256 unchanged |
| Packages beyond the lock? | None observed; npm `ci` installs strictly from the lock |
| Install location | `tools/career-ops/node_modules/` only |

## 4. Installed direct dependency versions

From `npm ls --depth=0 --json` (exit 0):

| Package | Manifest request | Lockfile resolved | Installed | Consistent |
|---|---|---|---|---|
| `@google/generative-ai` | `^0.24.1` | 0.24.1 | 0.24.1 | Yes |
| `dotenv` | `^18.0.4` | 18.0.9 | 18.0.9 | Yes |
| `js-yaml` | `^5.3.0` | 5.5.0 | 5.5.0 | Yes |
| `playwright` | `1.64.1` | 1.64.1 | 1.64.1 | Yes — exact match |
| `undici` | `^7.30.1` | 7.30.1 | 7.30.1 | Yes |

Transitive packages installed by the same tree: `playwright-core 1.64.1` and `argparse 3.0.4`, matching the lockfile. `node_modules/.package-lock.json` (npm's hidden install inventory) is present inside the authorized `node_modules/` directory.

## 5. Node/npm versions

| Tool | Version |
|---|---|
| Node.js | `v22.17.1` — satisfies the manifest engine `^22.13.0 \|\| >=23.4.0` |
| npm | `10.9.2` |

## 6. Lifecycle-script confirmation

`npm ci` ran with `--ignore-scripts`. The pinned root `package.json` declares a `postinstall` that would attempt `npx playwright install chromium`; with scripts disabled it did not run. No dependency lifecycle script, build step, `npx`, or repository script was invoked. Because `npm ci` installs strictly from the lockfile, no lockfile regeneration or dependency-version change occurred.

## 7. Chromium and application confirmation

- No browser-install command was issued by this task.
- `node_modules/playwright-core/.local-browsers` (the repo-local browser location Playwright uses when browsers are installed into the package) is **absent**.
- Chromium was not downloaded or installed by this task; browser-dependent functionality (PDF generation, liveness browser checks, apply mode) therefore remains unavailable.
- The global Playwright browser cache was **not inspected**, per the task boundary on unrelated browser caches and profiles. Confidence that this task did not install a browser rests on `--ignore-scripts` and the absence of any browser-install invocation, not on inspecting that cache.
- The career-ops application, scanners, tests, and `doctor.mjs` were not started.

## 8. Post-install validation and Git status

| Check | Result |
|---|---|
| `node_modules/` exists | Yes |
| `npm ls --depth=0 --json` | Exit 0; five direct dependencies, no invalid/missing entries |
| Installed vs. lock vs. manifest | Consistent for all five direct dependencies |
| Installed Playwright version | 1.64.1 |
| Lockfile SHA-256 | `cb14c2cd24a3676514af291914db76ace250a8bbf2d6708006f02300028c22cc` — unchanged |
| `package.json` SHA-256 | `10ca52815f65e4ba14ea4025828f2f9b212d1b684a2ab495a173d3a461df1fac` — unchanged |
| Git HEAD | `a6d7526c05861b1df9b70c011f8b1a84c44252ac` — unchanged |
| Origin | `https://github.com/career-ops-hq/career-ops.git` — unchanged |
| Tracked working tree | Clean (`git status --porcelain --untracked-files=no` empty) |
| Non-ignored untracked files | None (`--untracked-files=all` empty; `node_modules/` and the ignored lockfile are excluded by `.gitignore`) |
| Tracked source/manifest changes | None |

## 9. Candidate-file integrity

Recomputed after installation; all match the accepted baseline:

| Candidate file | Expected SHA-256 | Actual SHA-256 | Result |
|---|---|---|---|
| `full-profile.json` | `2f767145d8cb3bb619f0e77ff5f9f0e804a6026e1d2122e7b3fa50dcc9161fdb` | `2f767145d8cb3bb619f0e77ff5f9f0e804a6026e1d2122e7b3fa50dcc9161fdb` | **PASS** |
| `internships.json` | `606e03d0d98d783a00c0afb002a08dc1958b065ea245a9b232fee8b0d5da2aea` | `606e03d0d98d783a00c0afb002a08dc1958b065ea245a9b232fee8b0d5da2aea` | **PASS** |
| `projects.json` | `86749dc8d6f1aa9523e6a57f1decf3a74c911e18eb3ed3ee2000f16b8725e4f2` | `86749dc8d6f1aa9523e6a57f1decf3a74c911e18eb3ed3ee2000f16b8725e4f2` | **PASS** |

## 10. Warnings, failures, and remaining blockers

- No installation warnings or failures were emitted.
- **Chromium remains uninstalled.** PDF generation, Playwright-based liveness checks, and the apply mode cannot run until a separate, explicit browser-download approval.
- **The lockfile remains Git-ignored.** Installed versions are reproducible in this checkout only; a different clean clone needs an approved lock-preservation decision (Task 07, §8).
- **The Bash test-path issue is unresolved.** `tests/helpers.mjs` would select WSL Bash on this machine unless the session PATH is adjusted; no test was run in this task.
- **No model/provider configuration was made.** Model-dependent features remain disabled pending approval.
- `npm ci` reached the registry to fetch the locked tarballs; it did not run audits or lifecycle scripts, and no credentials were read or printed.

## 11. Exact changes made and next recommended task

**Created by this task:**

1. `tools/career-ops/node_modules/` — the authorized installation artifact (7 packages, including npm's hidden `node_modules/.package-lock.json`).
2. `reports/08_dependency_installation_report.md` — this report.

**Not changed:** `package.json`, `package-lock.json`, `.gitignore`, upstream Git history/commit, candidate JSON, prior reports, environment variables, system settings, and browser configuration. The npm cache may have been populated as a normal fetch side effect; it was not cleared or modified deliberately.

**Recommended next task (not authorized here):** a controlled, read-only post-install validation that (a) confirms the Bash selection issue and documents a session-scoped remedy, (b) runs the project's `doctor.mjs`/test suite only after explicit authorization, and (c) leaves Chromium uninstalled until a separate browser-download approval. Do not start the application or a scan in that task without a further authorization.
