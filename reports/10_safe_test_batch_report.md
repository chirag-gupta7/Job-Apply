# Task 10 — Source-Reviewed Node-Only Test Batch Report

**Date:** 2026-10-29  
**Repository:** `D:\document\job apply\tools\career-ops`  
**Pinned revision:** `a6d7526c05861b1df9b70c011f8b1a84c44252ac`  
**Overall status:** **PARTIALLY VALIDATED** — 84/85 tests passed in the batch; the single failure was a hard-coded 10-second child-process timeout, and the same file passed 9/9 when rerun in isolation. Full-suite and Bash-dependent validation remain blocked.

## 1. Executive summary and validation status

Ten additional `node:test` suites were selected, source-reviewed, and executed. They cover location/work-mode filtering, stale/unreadable data gates, status normalization and transitions, URL/report handling, merge/dedup schema behavior, and source URL/ID normalization.

- **Batch run:** 85 tests, **84 passed, 1 failed**, 0 skipped. Exit code 1.
- **Failure:** `location-filter-remote-fallback.test.mjs` → "validator accepts omitted/boolean policy…" — the spawned `validate-portals.mjs` child hit the test's hard-coded **10,000 ms** timeout (`spawnSync` returned `status: null` after ~10.1 s). No assertion about application logic failed.
- **Isolated rerun:** the same file passed **9/9**; the previously timed-out test completed in **7.9 s**, under the limit.
- No source, test, manifest, lockfile, environment, or candidate file was modified. No browser or Chromium was launched or downloaded. No network, model, scanner, or application command was used.

**Interpretation:** functional assertions across the reviewed areas passed. One test is timing-sensitive on this Windows machine under concurrent `node --test` execution. Combined with the still-blocked full suite and Bash-dependent tests, overall status is **PARTIALLY VALIDATED**, not a claim that the whole project suite passes.

## 2. Preflight integrity checks

| Check | Result |
|---|---|
| Working directory | `D:\document\job apply` — exact match |
| Git HEAD | `a6d7526c05861b1df9b70c011f8b1a84c44252ac` |
| Origin | `https://github.com/career-ops-hq/career-ops.git` |
| Tracked working tree | Clean |
| `package.json` SHA-256 | `10ca52815f65e4ba14ea4025828f2f9b212d1b684a2ab495a173d3a461df1fac` |
| `package-lock.json` SHA-256 | `cb14c2cd24a3676514af291914db76ace250a8bbf2d6708006f02300028c22cc` |
| `node_modules/` | Present |
| Direct dependencies | `@google/generative-ai` 0.24.1, `dotenv` 18.0.9, `js-yaml` 5.5.0, `playwright` 1.64.1, `undici` 7.30.1 — consistent with lock/manifest |
| Candidate hashes | All three PASS (§7) |

`reports/09_post_install_validation.md` was read. Its findings (doctor excluded for `chromium.launch()`, Bash-selection issue, selected-test approach) were carried forward.

## 3. Test-file selection and safety-review rationale

| # | Test file | MVP area | Safety evidence | Executed |
|---|---|---|---|---|
| 1 | `tests/location-filter-strict.test.mjs` | Location/work-mode filter strictness | `node:test`; pure `buildLocationFilter` import; no fs/network/spawn | Yes |
| 2 | `tests/location-filter-remote-fallback.test.mjs` | Location + validator policy | `node:test`; temp `portals.yml`; spawns Node `validate-portals.mjs`; validator imports no network code | Yes |
| 3 | `tests/close-stale-unreadable-gate.test.mjs` | Stale/malformed data handling | `node:test`; temp data root; spawns Node `close-stale.mjs` (fs-only); asserts non-write on unreadable gates | Yes |
| 4 | `tests/normalize-tabs.test.mjs` | Status normalization | `node:test`; temp tracker; spawns Node `normalize-statuses.mjs` (fs-only) | Yes |
| 5 | `tests/merge-tracker-url-column-migration.test.mjs` | Source URL handling / schema migration | `node:test`; temp tracker; spawns Node `merge-tracker.mjs`; local fixtures only | Yes |
| 6 | `tests/set-status-report-link.test.mjs` | Status transitions / invalid transitions | `node:test`; temp tracker; spawns Node `set-status.mjs`; asserts no side effects on refusal | Yes |
| 7 | `tests/set-status-expect-tracker.test.mjs` | Concurrent/stale status writes | `node:test`; temp tracker; spawns Node `set-status.mjs`; digest gate, dry-run, concurrency | Yes |
| 8 | `tests/url-key-plugin-capability.test.mjs` | Source URL/ID normalization | `node:test`; pure `url-key`/plugin context; no network; reads local docs/source | Yes |
| 9 | `tests/merge-tracker-missing-report-warning.test.mjs` | Missing/stale report handling | `node:test`; temp data root; spawns Node `merge-tracker.mjs`/`verify-pipeline.mjs`; fictional fixtures | Yes |
| 10 | `tests/merge-tracker-test-isolation.test.mjs` | Local config/data-path isolation | `node:test`; temp tracker/additions; helper `run()` with Node only; no Bash | Yes |

Rejected for this batch (reviewed or screened, not run): `doctor.mjs` (launches Chromium); `test-all.mjs` (aggregate behavior not established; Bash-dependent and dashboard sections); `set-status.test.mjs` (large non-`node:test` suite without `finish()`, so direct execution would not produce a reliable exit code); `url-key.test.mjs`, `dedup-tracker-url-conflict.test.mjs`, `merge-tracker-url-dedup.test.mjs`, `scan-url-dedup.test.mjs`, `verify-cv-facts-missing-source.test.mjs` (same `pass/fail` pattern without `finish()`, so a direct run could exit 0 despite failures).

Three candidate tests were not run solely for exit-code reliability; they are not evidence of defects. Their coverage areas overlap with the selected suites, but gaps remain (§5).

## 4. Exact commands and results

### Batch command

```text
node --test tests/location-filter-strict.test.mjs tests/location-filter-remote-fallback.test.mjs tests/close-stale-unreadable-gate.test.mjs tests/normalize-tabs.test.mjs tests/merge-tracker-url-column-migration.test.mjs tests/set-status-report-link.test.mjs tests/set-status-expect-tracker.test.mjs tests/url-key-plugin-capability.test.mjs tests/merge-tracker-missing-report-warning.test.mjs tests/merge-tracker-test-isolation.test.mjs
```

| Metric | Value |
|---|---:|
| Exit code | 1 |
| Tests | 85 |
| Passed | 84 |
| Failed | 1 |
| Skipped / cancelled | 0 / 0 |
| Duration | ~26.1 s |

**Only failure:** `validator accepts omitted/boolean policy and rejects all non-boolean shapes by path` (`location-filter-remote-fallback.test.mjs:53`), `spawnSync` child `status: null` after ~10.1 s. The test allows 10 s for `validate-portals.mjs`; the child was terminated by that timeout.

Warnings emitted during merges were expected fixture messages (missing report links, URL-column inactive) and were asserted by their tests.

### Isolated rerun

```text
node --test tests/location-filter-remote-fallback.test.mjs
```

| Metric | Value |
|---|---:|
| Exit code | 0 |
| Tests | 9 |
| Passed | 9 |
| Failed | 0 |
| Duration | ~8.4 s |
| Slow test | ~7.9 s (under 10 s) |

### Other observations

- `merge-tracker-test-isolation` emitted its expected missing-report warning against temp paths.
- `merge-tracker-url-column-migration` emitted its expected "URL dedup INACTIVE" warning for a legacy tracker, and its missing-report warning; both are fixture expectations.
- No test wrote outside OS temp directories; no test touched the candidate JSON directory.

## 5. Failures, skips, and uncovered features

**Failure:** one timing failure, reproduced only under the concurrent batch (10.1 s vs. 10 s cap), passing at 7.9 s in isolation. This indicates a Windows performance/flakiness issue in the test harness, not a demonstrated functional bug in `validate-portals.mjs`. It should be recorded, not "fixed" by editing the test.

**Not executed / blocked:**

- Full `test-all.mjs` (aggregate safety not established; includes Bash and dashboard sections).
- `doctor.mjs` (browser launch).
- All Bash-dependent suites (helper selects WSL before PATH fallback).
- Browser/PDF/Playwright suites (no Chromium).
- Live provider/scanner network paths, model calls, application flows.

**Coverage still inadequate or untested:** real source freshness/liveness checks; provider fetches against live ATS endpoints; experience/seniority ranking logic; funding/company intelligence; cover-letter/fact-gate integration beyond unit-level checks; end-to-end scan → evaluate → track. Passing these tests does not prove a live provider works or that a job is currently open.

## 6. Bash/Windows test limitations

Unchanged from Task 09: `tests/helpers.mjs` `getBash()` tries Program Files/Scoop Git Bash paths, then a WSL probe, then bare PATH. `D:\Git` is not in the candidate list, and bare `bash` is the WSL launcher, so Bash-dependent tests are not authorized to run. No PATH, system, source, or test change was made. The selected suites avoid `getBash()` and shell execution.

## 7. Candidate and project-file integrity after testing

| Check | Result |
|---|---|
| Git HEAD | `a6d7526c05861b1df9b70c011f8b1a84c44252ac` — unchanged |
| Origin | `https://github.com/career-ops-hq/career-ops.git` — unchanged |
| Tracked working tree | Clean |
| Non-ignored untracked files | None |
| `package.json` SHA-256 | `10ca52815f65e4ba14ea4025828f2f9b212d1b684a2ab495a173d3a461df1fac` — unchanged |
| `package-lock.json` SHA-256 | `cb14c2cd24a3676514af291914db76ace250a8bbf2d6708006f02300028c22cc` — unchanged |
| Installed Playwright | 1.64.1 |
| Repo-local browser directory | Absent (`node_modules/playwright-core/.local-browsers` false) |
| Test temp leftovers | None for the reviewed suite prefixes (`close-stale-gate-*`, `normalize-tabs-*`, `url-column-migration-*`, `career-ops-report-link-*`, `career-ops-status-expect-*`, `career-ops-missing-report-*`, `career-ops-merge-isolation-*`, `co-remote-validation-*`) |
| `npm ls --depth=0 --json` | Exit 0; five direct dependencies consistent |

| Candidate file | Actual SHA-256 | Result |
|---|---|---|
| `full-profile.json` | `2f767145d8cb3bb619f0e77ff5f9f0e804a6026e1d2122e7b3fa50dcc9161fdb` | **PASS** |
| `internships.json` | `606e03d0d98d783a00c0afb002a08dc1958b065ea245a9b232fee8b0d5da2aea` | **PASS** |
| `projects.json` | `86749dc8d6f1aa9523e6a57f1decf3a74c911e18eb3ed3ee2000f16b8725e4f2` | **PASS** |

## 8. Overall confidence and limitations

**Higher confidence:** local URL normalization and dedup-key behavior, location/work-mode filter policy, status normalization/transitions and no-write-on-refusal, stale/unreadable gate handling, merge schema migration, missing-report warning behavior, and merge fixture isolation.

**Lower confidence / unverified:** live ATS/provider fetching and freshness, job availability, model/provider integration, browser/PDF generation, Bash-based batch execution, experience/seniority ranking, and full application workflow. One timing-sensitive test is noted, and the full suite remains unrun.

## 9. Files changed and recommended next task

**Created:** `reports/10_safe_test_batch_report.md` only. No test code, source, configuration, manifest, lockfile, environment, or candidate file was changed. Test temp artifacts were cleaned up.

**Recommended next task (not authorized here):** a small source-reviewed batch covering experience/seniority filtering and provider response normalization using local fixtures only, with the same safety screening. Keep the full suite and Bash-dependent tests out of scope until a separately approved shell-selection plan exists. Do not start the application, scan live sources, or enable Chromium/model access.
