# Task 11 — Experience Filtering and Provider Normalization Test Report

**Date:** 2026-10-29  
**Repository:** `D:\document\job apply\tools\career-ops`  
**Pinned commit:** `a6d7526c05861b1df9b70c011f8b1a84c44252ac`  
**Status:** **TARGETED TESTS PASSED** — 49/49 selected tests passed across 8 files; material coverage gaps are disclosed below.

## 1. Executive summary and validation status

Eight source-reviewed `node:test` suites were executed sequentially: **49 tests passed, 0 failed, 0 skipped**, every command exit 0. They verify provider field normalization (Ashby compensation), invalid-date rejection, title normalization/filtering, Greenhouse URL/host handling, provider config bounds, provider-registry root resolution, and Ashby retry/permanent-failure behavior.

Two findings frame the result:

1. **Experience filtering is deterministic but title-based, not years-based.** The scanner has no numeric years-of-experience parser or filter. It applies configured title keywords and an optional four-tier title classifier (`intern`, `entry`, `mid`, `senior`). Actual years requirements are only assessed later in the LLM evaluation, not deterministically.
2. **Missing seniority defaults to `mid`.** A plain "Backend Engineer" and a role with unknown seniority both classify as `mid`; configuring `skip_tiers: [mid]` would silently drop them along with genuine mid-level roles. Entry-level campaigns should not skip `mid`.

Also, the most directly relevant experience/seniority suites (`classify-tier-position`, `scan-skip-tiers-per-entry`, `title-fit`, `jd-similarity-seniority`, provider `pass/fail` suites) are not `node:test` and do not call `finish()`, so direct execution cannot produce a trustworthy failure exit code. They were **not run**; that is a validation gap, not evidence of a defect.

## 2. Preflight integrity results

| Check | Result |
|---|---|
| Working directory | `D:\document\job apply` — exact match |
| Git HEAD | `a6d7526c05861b1df9b70c011f8b1a84c44252ac` |
| Origin | `https://github.com/career-ops-hq/career-ops.git` |
| Tracked working tree | Clean |
| `package.json` SHA-256 | `10ca52815f65e4ba14ea4025828f2f9b212d1b684a2ab495a173d3a461df1fac` |
| `package-lock.json` SHA-256 | `cb14c2cd24a3676514af291914db76ace250a8bbf2d6708006f02300028c22cc` |
| `node_modules/` | Present; `npm ls` consistent with manifest/lock |
| Candidate hashes | All three PASS before tests (§8) |

Reports 09 and 10 were reviewed: `doctor.mjs` remains excluded for launching Chromium; Bash-dependent suites remain blocked; the Task 10 timeout sensitivity informed the sequential execution used here.

## 3. Experience/seniority implementation review

### Deterministic path

- `scan.mjs` applies `title_filter.positive` / `.negative` (with `word:`/`stem:` modifiers) before any tier logic.
- If `skip_tiers` is configured, `scan.mjs` imports `classify-tier.mjs` and rejects titles whose classified tier is listed. Per-entry `skip_tiers` overrides the global list; `[]` disables it for that entry.
- `classify-tier.mjs` returns exactly `intern`, `entry`, `mid`, or `senior`; **default is `mid`** for unrecognized/plain titles.
- There is no parsing or filtering on numeric "years of experience" in `scan.mjs` or `classify-tier.mjs`. No `experience_years` filter exists there.

### Representative cases (code-derived behavior)

| Case | Title-filter behavior | Classified tier | Notes |
|---|---|---|---|
| Backend Engineer, 0–1 years | Passes if "Backend Engineer" configured as positive | `mid` (no level keyword) | No years filter; scanner keeps it. `skip_tiers:[mid]` would drop it. |
| Python Developer, fresh-graduate suitable | Passes if title keyword matches | `mid` | Fresh-graduate suitability is only visible in JD text; not deterministically enforced. LLM evaluation may score it, but does not block scan inclusion. |
| Software Engineering Intern | Passes only if intern titles are not in `negative` | `intern` | The shipped example template excludes `word:Intern`/`Internship`; a candidate config must include internship terms. `skip_tiers:[intern]` would also drop it. |
| Graduate Engineer Trainee | Passes if keyword matches | `intern` (`trainee` matcher) | The classifier's built-in cases include "Graduate Engineer Program" → `intern`. |
| Junior Backend Engineer, no years field | Passes if keyword matches | `entry` (`junior`) | No years data needed. |
| Backend Engineer, 3–5 years | Passes if keyword matches | `mid` | Years are not filtered deterministically; scanner keeps it. Evaluation-stage reasoning is the only check. |
| Senior Backend Engineer | Passes if keyword matches | `senior` | Excluded only if configured negative/skip. |
| Unknown seniority / missing experience | Passes if title matches a positive keyword | `mid` | Unknown becomes `mid` by design. A missing/non-string title also classifies `mid`, but the positive title filter normally rejects it first. |
| "Graduate Engineer" (no program/scheme) | Passes if keyword matches | `mid` | The `graduate` matcher requires `program`/`scheme`; bare "Graduate Engineer" is `mid` (pinned in the classifier's self-test data). |

### Product implication

The current design can preserve early-career roles with absent experience fields, provided the configuration does not skip `mid` and does include junior/intern/graduate keywords. It does **not** guarantee a 0–1-year role is accepted or a 3–5-year role rejected: that judgment is left to the evaluation stage. Configuring `skip_tiers: [mid]` is the main silent-exclusion risk for plain/unknown titles.

## 4. Provider-response normalization review

### What the code does (from inspected modules)

- Providers return a common job shape; Ashby's `parseCompensation` reads nested `compensationTiers[].components[]`, accepts the flat shape, and returns `null` when no readable salary exists rather than inventing a value.
- Date parsers for Softgarden and SuccessFactors reject impossible calendar dates (`02/31/2026`, `2026-06-31`) instead of rolling them over.
- `buildTitleFilter`/`matchedTitleKeywords` normalize whitespace (including NBSP/tab/doubled spaces) and fold accents on both keyword and title sides.
- `prepare-application.mjs` validates the ATS host and requires HTTPS; lookalike hosts (`job-boards.greenhouse.io.example.invalid`), embedded-URL query tricks, and `http://` are refused. Provider-returned `job-boards(.eu).greenhouse.io` URLs are accepted.
- `providers/_config-utils.mjs` clamps out-of-range page/page-size values and falls back on unreadable values, preventing empty or pathological queries.
- `audit-portals` loads the provider registry from the checkout while keeping `portals.yml` in the data root.
- Ashby retries only transient failures (429/5xx/network); 400/401/403/404/410 cost exactly one request. A successful parse preserves title, company, and posted timestamp.

### Not established by this batch

Missing/null `title`, `company`, `location`, `description`, and `applyUrl` normalization; remote/hybrid/onsite classification; country-eligibility/unknown handling; provider-specific ID preservation and canonical employer URL retention beyond Greenhouse; cross-source duplicate IDs; unknown optional-field changes across providers. These areas have tests, but the most relevant provider suites are non-`node:test` `pass/fail` files whose direct exit code is unreliable, so they were not executed.

## 5. Selected tests and safety evidence

| Test file | Area | Safety evidence |
|---|---|---|
| `tests/providers/ashby-compensation-tiers.test.mjs` | Provider compensation normalization, malformed/absent data | Pure function import; fixtures inline; no I/O/network |
| `tests/provider-date-rollover.test.mjs` | Invalid date normalization | Pure parsers; no I/O/network |
| `tests/title-filter-whitespace.test.mjs` | Title normalization/filtering | Pure imports; inline titles |
| `tests/title-filter-accent-folding.test.mjs` | Title filtering with accents | Pure imports; inline titles |
| `tests/prepare-greenhouse-hosts.test.mjs` | Provider URL/ID handling, host validation | Temp data root and fictional PDF/profile; injected `fetchJson`; no network |
| `tests/provider-config-utils.test.mjs` | Provider config bounds | Pure function; inline values |
| `tests/audit-portals-providers-root.test.mjs` | Provider registry vs data root | Temp data root; local provider registry; no network |
| `tests/providers/ashby-retry.test.mjs` | Provider failure handling/retry | Stubbed `ctx.fetchJson`/`ctx.sleep`; no real requests |

All are `node:test` suites with reliable exit codes, no Chromium, no Bash, no application/server start, no live API, and no candidate-file access. Each was executed in its own process/command to avoid the Task 10 concurrency timeout.

## 6. Exact commands and results

| Command | Exit | Tests | Pass | Fail | Duration |
|---|---:|---:|---:|---:|---:|
| `node --test tests/providers/ashby-compensation-tiers.test.mjs` | 0 | 15 | 15 | 0 | ~0.14 s |
| `node --test tests/provider-date-rollover.test.mjs` | 0 | 2 | 2 | 0 | ~0.11 s |
| `node --test tests/title-filter-whitespace.test.mjs` | 0 | 4 | 4 | 0 | ~0.32 s |
| `node --test tests/title-filter-accent-folding.test.mjs` | 0 | 6 | 6 | 0 | ~0.31 s |
| `node --test tests/prepare-greenhouse-hosts.test.mjs` | 0 | 7 | 7 | 0 | ~0.90 s |
| `node --test tests/provider-config-utils.test.mjs` | 0 | 8 | 8 | 0 | ~0.13 s |
| `node --test tests/audit-portals-providers-root.test.mjs` | 0 | 2 | 2 | 0 | ~0.52 s |
| `node --test tests/providers/ashby-retry.test.mjs` | 0 | 5 | 5 | 0 | ~0.16 s |
| **Total** | **0 (all)** | **49** | **49** | **0** | **~2.6 s** |

No warnings or error output affected results; the only console text was normal TAP output.

## 7. Failures, timeouts, and uncovered behavior

- **Failures/timeouts:** none in this task. The Task 10 10-second timeout did not recur; tests were run sequentially.
- **Not run (blocked by test harness shape):** `classify-tier-position`, `scan-skip-tiers-per-entry`, `title-fit`, `jd-similarity-seniority`, and provider `pass/fail` suites — no `finish()`/reliable exit code when run directly. They require `test-all.mjs` or a harness change, neither authorized here.
- **Not run (Bash/Windows):** all Bash-dependent suites, unchanged from Task 09.
- **Uncovered product behavior:** numeric years-of-experience filtering; internship/graduate keyword workflow end-to-end; missing/null job fields through each provider; work-mode and country-eligibility classification; cross-source dedup with distinct IDs; unknown/added optional provider fields; liveness/freshness; live ATS behavior.
- **Potential product risk (needs configuration care, not a proven bug):** `skip_tiers: [mid]` would drop plain/unknown titles, including ordinary Backend Engineer roles. Any candidate config should avoid skipping `mid` and should include intern/graduate/junior keywords.

## 8. Final integrity checks

| Check | Result |
|---|---|
| Git HEAD | `a6d7526c05861b1df9b70c011f8b1a84c44252ac` — unchanged |
| Origin | `https://github.com/career-ops-hq/career-ops.git` — unchanged |
| Tracked working tree | Clean |
| `package.json` SHA-256 | `10ca52815f65e4ba14ea4025828f2f9b212d1b684a2ab495a173d3a461df1fac` — unchanged |
| `package-lock.json` SHA-256 | `cb14c2cd24a3676514af291914db76ace250a8bbf2d6708006f02300028c22cc` — unchanged |
| Direct dependencies | Consistent; Playwright 1.64.1 |
| Chromium | Not launched/downloaded; repo-local browser dir absent |
| Temp artifacts | None for `prepare-greenhouse-*` / `career-ops-auditproviders-*` |

| Candidate file | Actual SHA-256 | Result |
|---|---|---|
| `full-profile.json` | `2f767145d8cb3bb619f0e77ff5f9f0e804a6026e1d2122e7b3fa50dcc9161fdb` | **PASS** |
| `internships.json` | `606e03d0d98d783a00c0afb002a08dc1958b065ea245a9b232fee8b0d5da2aea` | **PASS** |
| `projects.json` | `86749dc8d6f1aa9523e6a57f1decf3a74c911e18eb3ed3ee2000f16b8725e4f2` | **PASS** |

**Created:** `reports/11_experience_provider_test_report.md` only. No tests, source, configuration, manifest, environment, or candidate files were modified.

## 9. Recommended next task

A separately authorized, read-only coverage audit of the experience/seniority suites and provider `pass/fail` suites, deciding whether to run them through a reviewed harness or adapt them to `node:test` in a future code task. Until then, configure the MVP without `skip_tiers: [mid]`, include intern/graduate/junior title terms, and rely on evaluation-stage review for years-of-experience mismatches. Do not start the application or enable live providers/browsers.
