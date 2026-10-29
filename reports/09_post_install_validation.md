# Task 09 — Post-Install Diagnostics and Safe Test Validation

**Date:** 2026-10-29  
**Repository:** `D:\document\job apply\tools\career-ops`  
**Pinned revision:** `a6d7526c05861b1df9b70c011f8b1a84c44252ac`  
**Overall status:** **PARTIALLY VALIDATED** — selected safe tests passed; the full suite and `doctor.mjs` were deliberately not run because they do not meet this task's execution boundaries.

## 1. Executive summary and overall validation status

The installation is intact. `npm ls --depth=0 --json` exits 0 with the five direct dependencies at the exact versions in the lockfile and manifest, Playwright remains `1.64.1`, and `node_modules/` is present. No Chromium was downloaded or launched.

Static review changed the execution plan:

- **`doctor.mjs` was not run.** Although it is largely local, its Playwright check calls `chromium.launch({ headless: true })` (`doctor.mjs` lines ~487–541). That is a browser launch, which this task forbids regardless of whether a browser binary exists.
- **The full `test-all.mjs` suite was not run.** The runner is a ~1 MB file with 1,134 discovered test files plus inline sections; its full run includes a Go dashboard build, hundreds of spawned scripts, GitHub-related fixtures, and Bash-dependent suites. Its aggregate behavior cannot be established as network-free and Bash-free within this task's review scope. Running it would go beyond what the task permits until those suites are individually reviewed.
- **Two small `node:test` suites were reviewed and executed.** They are Node-only, fixture/temp-directory based, and have no network, Bash, Playwright, application-start, or live-job behavior. **8/8 tests passed** (exit 0).

All integrity checks after the run passed. No test left temp artifacts behind. No project source, manifest, lockfile, Git state, or candidate JSON changed.

## 2. Preflight hashes and dependency inventory

| Check | Result |
|---|---|
| Working directory | `D:\document\job apply` — exact match |
| Git HEAD | `a6d7526c05861b1df9b70c011f8b1a84c44252ac` — expected commit |
| Origin | `https://github.com/career-ops-hq/career-ops.git` |
| Tracked working tree | Clean |
| `node_modules/` | Present (from Task 08) |
| `package.json` SHA-256 | `10ca52815f65e4ba14ea4025828f2f9b212d1b684a2ab495a173d3a461df1fac` — expected |
| `package-lock.json` SHA-256 | `cb14c2cd24a3676514af291914db76ace250a8bbf2d6708006f02300028c22cc` — expected |
| Node / npm | `v22.17.1` / `10.9.2` |
| Installed direct dependencies | `@google/generative-ai` 0.24.1; `dotenv` 18.0.9; `js-yaml` 5.5.0; `playwright` 1.64.1; `undici` 7.30.1 |
| Candidate hashes | All three PASS (§8) |

`npm ls --depth=0 --json` exited 0 with no invalid, missing, or extraneous top-level entries. Installed versions match both `package.json` ranges and the lockfile's resolved versions.

## 3. Static safety review of diagnostics and tests

| Command/file | Network | Live jobs/model | Server/browser | Writes | Decision |
|---|---|---|---|---|---|
| `doctor.mjs --json` | No network by default (`--strict` enables portal reachability and was not used) | No model/job call | **Launches Chromium** in `checkPlaywright()` (`chromium.launch({headless:true})`) | Read-only without `--init-templates`, but it reads local config/MCP files and can emit repair suggestions | **Not executed.** Browser launch violates task boundary, so behavior is not validated here. |
| `test-all.mjs` (full) | Cannot be established as network-free across 1,134 suites and inline sections | Multiple suites spawn scripts; some fixtures model ATS/provider behavior | Inline dashboard section builds Go dashboard; other suites may spawn browsers/stubs | Designed to isolate data under a temp root, but full-run coverage is too broad to verify here | **Not executed.** Full-suite safety cannot be established without reviewing all suites. |
| `tests/helpers.mjs` | No direct network; spawns only allowlisted executables | Uses temp `CAREER_OPS_TEST_DATA_ROOT`, blanks tracker/root overrides, has real-root drift guard | `getBash()` can probe WSL; PDF tests may use Playwright | Creates/removes temp data root and scratch dirs | Safe pattern for Node-only suites; Bash selection remains an issue (§4). |
| `tests/ascii-fold-punctuation.test.mjs` | None | None | None | None (pure imports) | **Executed.** |
| `tests/set-status-missing-tracker.test.mjs` | None | None | Spawns Node only (no Bash/browser) | Temp directories under OS temp via `mkdtempSync`; `--dry-run` prevents tracker writes; `rmSync` in `finally` | **Executed.** |

The runner's own `--only` mode skips inline sections and discovers suites under `tests/`, but it runs the full harness setup and could still include suites not reviewed here. Direct `node --test` was used only for the two reviewed files.

## 4. Bash-selection diagnosis and supported remedy

`tests/helpers.mjs` `getBash()` resolution order (pinned revision, lines ~1358–1386):

1. **Hard-coded Git Bash candidates**: `C:\Program Files\Git\bin\bash.exe`, `C:\Program Files\Git\usr\bin\bash.exe`, and Scoop paths under `%USERPROFILE%`/`%SCOOP%`.
2. **WSL probe**: runs `wsl -e bash -c true`; on success, the source is recorded as `wsl` and the shell is the bare name `bash`, which resolves to `C:\Windows\System32\bash.exe`.
3. **PATH fallback**: probes bare `bash` and caches the resolved executable.

`toBashPath()` similarly prefers `cygpath` from Program Files/Scoop hard-coded candidates, otherwise `cygpath` on PATH, then WSL `wslpath`.

**Findings:**

- **No environment-variable override exists** in the helper for selecting a specific Bash or `cygpath`. There is no supported `CAREER_OPS_BASH`-style setting.
- **A session PATH prepend does not reliably avoid WSL**, because the WSL probe runs *before* the PATH probe and the hard-coded candidates do not include `D:\Git`. Since the WSL launcher exists on this machine, only a source patch, a persistent/system change, or making WSL unavailable would change that order — none is permitted here.
- **A matching `cygpath` is not picked up** from `D:\Git` unless it is also reachable on PATH; the hard-coded list lacks this install.
- **The test runner does not accept an explicit Git Bash path.** Only Node-only suites (which never call `getBash()`) are therefore safe to run under the current constraints.
- The ordinary Node command path is unaffected: `node *.mjs` does not shell out to Bash.

**Result:** Bash-dependent tests are **blocked** by tooling design, not by this task's refusal to fix it. Running them would require a separately authorized change (session-scoped PATH plus verified helper behavior, or a reviewed source/config change). No PATH or system change was made.

## 5. Commands executed and exact results

| Command | Working directory | Exit | Result |
|---|---|---:|---|
| `node --test tests/ascii-fold-punctuation.test.mjs tests/set-status-missing-tracker.test.mjs` | `tools\career-ops` | 0 | 8 tests, 8 pass, 0 fail, 0 skipped, 0 cancelled (TAP 13) |
| `npm ls --depth=0 --json` | `tools\career-ops` | 0 | Five direct dependencies at expected versions; no invalid/extraneous |
| Read-only hash/status checks | repo and candidate directory | 0 | All expected values matched (§2, §8) |

No other project command was run. `doctor.mjs`, `test-all.mjs`, `npm test`, scanners, `npx`, `update-system.mjs`, and any application/browser command were **not** executed.

## 6. Test results, limitations, and skipped checks

### Passed

- `ascii-fold-punctuation.test.mjs` — 5 tests:
  - punctuation-mode behavior (`space` vs `delete`)
  - default folding behavior and Unicode diacritics
  - tab handling under `delete`
  - trust-validator host matching (mismatch and match)
  - non-Latin names folding to empty
- `set-status-missing-tracker.test.mjs` — 3 tests:
  - missing-tracker error names canonical write path without creating it
  - legacy tracker at data root remains readable
  - `CAREER_OPS_TRACKER` override path is named when missing

### Not run / not validated

- **`doctor.mjs --json`**: blocked by browser-launch behavior.
- **Full `test-all.mjs`**: blocked by unverified aggregate behavior (network/Bash/browser/dashboard build) and the Bash-selection issue.
- **Bash-dependent suites** (batch runner, shell fixtures, PATH/cygpath tests): blocked by helper selection.
- **Playwright/PDF and browser suites**: blocked because Chromium is not installed and launching a browser is out of scope.
- **Scanner/provider live-network paths, model/provider calls, application flows, tracker writes beyond temp dry-runs**: untested.
- No claim is made that the full project test suite passes; only the two named suites were validated.

## 7. Browser/model/network behavior confirmed or still unknown

- **Chromium:** not installed or launched by this task. `node_modules/playwright-core/.local-browsers` remains absent. The global Playwright browser cache was not inspected, per the task boundary.
- **Model providers:** no provider was configured, no key was read or used, and no prompt/request was sent.
- **Job APIs/network:** the selected tests made no network calls. No scanner or live job source was contacted.
- **Application/email:** nothing was submitted or sent.
- Unknown: behavior of `doctor.mjs` beyond its static source; full-suite runtime; browser-dependent paths; Bash-dependent tests.

## 8. Post-validation integrity checks

| Check | Result |
|---|---|
| Git HEAD | `a6d7526c05861b1df9b70c011f8b1a84c44252ac` — unchanged |
| Origin | `https://github.com/career-ops-hq/career-ops.git` — unchanged |
| Tracked working tree | Clean |
| Non-ignored untracked files | None |
| `package.json` SHA-256 | `10ca52815f65e4ba14ea4025828f2f9b212d1b684a2ab495a173d3a461df1fac` — unchanged |
| `package-lock.json` SHA-256 | `cb14c2cd24a3676514af291914db76ace250a8bbf2d6708006f02300028c22cc` — unchanged |
| Playwright installed | 1.64.1 — unchanged |
| Node modules inventory | `npm ls` exit 0; five direct dependencies consistent |
| Test temp leftovers (`co-status-*`, `ats*`, etc.) | None found under OS temp for the selected suites |
| Candidate hashes | All three PASS (below) |

| Candidate file | Actual SHA-256 | Result |
|---|---|---|
| `full-profile.json` | `2f767145d8cb3bb619f0e77ff5f9f0e804a6026e1d2122e7b3fa50dcc9161fdb` | **PASS** |
| `internships.json` | `606e03d0d98d783a00c0afb002a08dc1958b065ea245a9b232fee8b0d5da2aea` | **PASS** |
| `projects.json` | `86749dc8d6f1aa9523e6a57f1decf3a74c911e18eb3ed3ee2000f16b8725e4f2` | **PASS** |

## 9. Files modified and artifacts created

- **Created:** `reports/09_post_install_validation.md` (this report) — the only persistent deliverable.
- **No changes** to candidate JSON, reports 01–08, `package.json`, `package-lock.json`, `.gitignore`, project source/tests, environment variables, system configuration, browser profiles, or Git history.
- Test-owned temp directories were created under the OS temp directory and removed by the tests; a follow-up check found no leftovers.
- The npm cache was not deliberately modified.

## 10. Remaining blockers and recommended next task

1. **Bash-dependent validation is blocked** by `getBash()` selecting WSL before any PATH fallback and lacking an override for `D:\Git`. A separate authorization must choose either a session-scoped PATH approach with verified helper behavior or a reviewed configuration/source change.
2. **Full-suite validation is not established.** If required, scope it in batches: review each batch's suites for network/browser/Bash/application behavior, then run only approved batches with the helper's isolation.
3. **`doctor.mjs` remains intentionally unvalidated** because of its Chromium launch. A version of that diagnostic that skips the Playwright probe, or an approved environment where browser launch is allowed, would be needed.
4. **Chromium and all browser-dependent features remain unavailable** pending separate download approval.
5. **Model providers remain unconfigured**; no candidate data egress occurred.

**Recommended next task (not authorized):** a scoped, source-reviewed test batch plan that explicitly resolves the Bash issue without patching project code, followed by a small approved batch run. Do not start the application or enable Chromium/model access in that task without separate approval.
