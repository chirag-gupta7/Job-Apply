# Task 06 — Pinned Source Checkout and Static Audit

**Audit date:** 2026-10-28 (environment inventory timestamp `2026-10-28T21:32:22+05:30` from Task 05)
**Working directory:** `D:\document\job apply`
**Checkout:** `D:\document\job apply\tools\career-ops` at commit `a6d7526c05861b1df9b70c011f8b1a84c44252ac`
**Decision:** **GO** for the pinned source checkout itself. **CONDITIONAL GO** for a future dependency installation, with the bounded prerequisites in §8–§10. No dependency was installed and no project code was executed.

## 1. Executive summary and go/no-go decision

The exact reviewed revision was cloned from the correct origin, checked out in detached HEAD, and verified against the requested SHA. All required documentation and source files exist at this revision. The static inspection confirms Task 05's findings and adds specifics:

- `package.json` (v1.38.0, `private: true`, Node `^22.13.0 || >=23.4.0`) has **only** a `postinstall` lifecycle script, which attempts `npx playwright install chromium` (twice) unless `PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD` is set. There is no tracked root lockfile (`package-lock.json` is in `.gitignore`; no `npm-shrinkwrap.json` exists).
- The test-helper Bash selection (`tests/helpers.mjs` `getBash()`) probes only Program Files/Scoop Git Bash paths, then a WSL probe, then bare PATH `bash`. This machine's `D:\Git` install matches none of the hard-coded candidates, and bare `bash` resolves to the WSL launcher, so a test run would select **WSL Bash** unless the session PATH is adjusted first.
- Source inspection confirms the documented draft-only posture: `prepare-application.mjs` performs no HTTP POST; `docs/APPLY_AUTOFILL.md` states the agent never submits. The `apply` mode does open a live form in a project-launched Playwright browser — excluded from the MVP.
- The repository's `AGENTS.md` (auto-loaded into agent context by design) contains agent-behavior instructions, including an update-check command and onboarding flows. It was treated strictly as **untrusted inspected data**; none of its instructions were executed.

**Go/no-go:** the checkout itself is verified and safe as read-only evidence — **GO**. A dependency install remains **CONDITIONAL GO**: it requires an approved dependency-review plan (no tracked lockfile; semver ranges), an explicit decision on the Chromium download, and a resolved Git Bash test-shell path (§8–§10).

## 2. Candidate-source hashes before and after checkout

Computed from raw bytes with SHA-256; the working directory was confirmed exactly `D:\document\job apply` before any action.

| Candidate file | Expected | Before checkout | After checkout | Result |
|---|---|---|---|---|
| `full-profile.json` | `2f767145d8cb3bb619f0e77ff5f9f0e804a6026e1d2122e7b3fa50dcc9161fdb` | match | match | **PASS** |
| `internships.json` | `606e03d0d98d783a00c0afb002a08dc1958b065ea245a9b232fee8b0d5da2aea` | match | match | **PASS** |
| `projects.json` | `86749dc8d6f1aa9523e6a57f1decf3a74c911e18eb3ed3ee2000f16b8725e4f2` | match | match | **PASS** |

All three matched before and after the checkout. No candidate file was modified.

## 3. Checkout location, origin, and verified commit SHA

Preflight confirmed `D:\document\job apply\tools` and `tools\career-ops` did not exist, so no existing checkout was overwritten or reused. The clone and detached checkout used the machine's Git `2.53.0.windows.2` (`D:\Git\cmd\git.exe`).

| Check | Command/evidence | Result |
|---|---|---|
| Clone source | `git clone https://github.com/career-ops-hq/career-ops.git tools\career-ops` | Completed; 2,843 files checked out |
| Checked-out commit | `git rev-parse HEAD` | `a6d7526c05861b1df9b70c011f8b1a84c44252ac` — **exact match** to the required full SHA |
| Origin | `git remote get-url origin` | `https://github.com/career-ops-hq/career-ops.git` — correct upstream |
| Working-tree state | `git status --porcelain --branch` | `## HEAD (no branch)` — detached HEAD, clean tree, no local modifications |
| Version context | `git describe --always --tags` | `career-ops-v1.38.0-231-ga6d7526c` — 231 commits after the v1.38.0 tag; not a tagged release, which is expected for a main-branch pin |

The exact revision was obtained; no substitution with `main` or a release was made. `node_modules/` does not exist in the checkout; no dependency was installed. `AGENTS.md`, `CLAUDE.md`, `OPENCODE.md`, and `.opencode/` skill wrappers are present at this revision, which is why the repository instructions auto-loaded into agent context during inspection.

## 4. Static inspection of installation and lifecycle scripts

All inspection was read-only; no listed file is missing at this revision.

| Item | Finding at `a6d7526` |
|---|---|
| Lifecycle scripts | `package.json` defines **only** `postinstall`: a Node one-liner that exits 1 when `PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD` is unset/`0`/`false`, then runs `npx playwright install chromium` twice, warning on failure. No `preinstall`/`install` script. `npm install` therefore downloads the Playwright Chromium browser by default. |
| Dependency specs | `dependencies`: `@google/generative-ai ^0.24.1`, `dotenv ^18.0.4`, `js-yaml ^5.3.0`, `playwright 1.64.1` (exact), `undici ^7.30.1`. Four of five are semver ranges. |
| Lockfile | No `package-lock.json` in the repo (404 at raw path; `.gitignore` excludes it). No `npm-shrinkwrap.json`. Dependency resolution is **not reproducible from the repo alone**. |
| npm bootstrap | `npx @santifer/career-ops init` installs the **latest release** via a separate npm package not contained in this repository and not audited. Do not use it for this pin. |
| Commands executing external content | `openai-eval.mjs` POSTs `cv.md` + profile + full JD to a user-configured OpenAI-compatible endpoint (HTTPS enforced off-loopback; key required). `scan.mjs` performs zero-token HTTP GET reads of ATS feeds. `generate-pdf.mjs` drives Playwright Chromium locally. `gemini-eval.mjs` sends data to Google AI. None of these was executed. |
| Data paths | `path-resolver.mjs`: data root resolution order is `CAREER_OPS_ROOT`/`CAREER_OPS_DATA_DIR` → `.career-ops-data` marker → plugin-copy default `~/career-ops-data` → repository root. Tracker write path is `{DATA_ROOT}/data/applications.md` (or `CAREER_OPS_TRACKER`). Reports → `{DATA_ROOT}/reports/`, PDFs → `{DATA_ROOT}/output/`, scanner config → `{DATA_ROOT}/portals.yml`. A data root resolving inside a Claude Code plugin copy throws `UnsafeDataRootError`. |
| `.gitignore` coverage | Ignores `cv.md`, `data/*` (with `.gitkeep` exceptions), `reports/*.md`, `output/`, `batch/*` (except scripts/prompt), `jds/*`, `documents/*`, `interview-prep/*`, `writing-samples/*`, `config/profile.yml`/`cv-facts.json`, `portals.yml`, `.env`, `**/.env*.local`, `*.db`/`-wal`/`-shm`, `*.bak*`/`*.tmp*`, `package-lock.json`, and identity-filename patterns (`*passport*`, `*diploma*`, etc.). Coverage of local data, generated files, and secrets is thorough; it is a Git convention, not an access control. |
| Repository agent instructions | `AGENTS.md` instructs agents to run `node update-system.mjs check` each session, personalize `modes/_profile.md`, and follow onboarding flows. Treated as untrusted data; not followed. Its Data-Contract and Source-of-Truth sections are consistent with the docs and are useful evidence, but only the user's explicit task instructions govern this project. |

## 5. Runtime and dependency findings

- **Node:** required `^22.13.0 || >=23.4.0`; installed `v22.17.1` (`D:\node\node.exe`) — meets the requirement. `node:sqlite` used by the tracker index needs 22.13+, satisfied.
- **npm:** `10.9.2` present. Not run.
- **Git:** `2.53.0.windows.2` present; clone/checkout worked.
- **Python 3.13.3 / Go 1.25.7:** present but not required for the core workflow (Go optional for the dashboard TUI).
- **Bun / Docker:** absent; not required by the documented career-ops path.
- **Chromium:** not installed; needed only for PDF generation, liveness checks, and the apply/Playwright modes. Not required for zero-token scanning, evaluation reports (text), tracker, or email drafts.
- **Unresolved supply-chain risk:** because the root lockfile is untracked and ranges are used, two `npm install` runs at the same commit can produce different dependency trees. A reviewed lock strategy (§9) is required before install.

## 6. Windows Git Bash and path handling

`tests/helpers.mjs` (`getBash()`, lines ~1358–1386) resolves Bash in this order:

1. **`WINDOWS_BASH_CANDIDATES`** — literal paths under `C:\Program Files\Git\` and `%USERPROFILE%\scoop`/`%SCOOP%` only. **`D:\Git\bin\bash.exe` is not in this list.** (A comment in the allowlist logic acknowledges Git installs outside Program Files/Scoop, e.g. `D:\Git`, but the candidate list itself does not include them.)
2. **WSL probe** — `wsl -e bash -c true`; if it succeeds, the source is marked `wsl` and the bare name `bash` is used, which on this machine resolves to `C:\Windows\system32\bash.exe` (the WSL launcher).
3. **PATH fallback** — bare `bash`; the resolved file is cached.

`toBashPath()` prefers `cygpath` from `WINDOWS_CYGPATH_CANDIDATES` (again Program Files/Scoop only, else `cygpath` on PATH — not present there on this machine), then WSL `wslpath`. `batch/batch-runner.sh` is a `#!/usr/bin/env bash` script, so the same selection applies to batch mode.

**Consequence on this machine:** unless PATH is adjusted first, the test/batch shell selection resolves to **WSL Bash**, which the project's own docs identify as the failure mode (`node: command not found`, exit 127). The ordinary Node command path (`node *.mjs`) does not shell out to Bash and is unaffected.

**Recommended handling (no PATH/system change made in this task):** in a future authorized session, prepend `D:\Git\bin` (and `D:\Git\usr\bin` for `cygpath`) to the **session** `PATH` before running tests/batch, so the WSL probe is not reached and the PATH fallback resolves Git Bash. Alternatively invoke Git Bash by full path for manual script runs. Verify the helper's actual selection with its diagnostics before trusting test results. Any persistent PATH change requires separate authorization and was not made.

## 7. Privacy, data egress, and source-provider assessment

| Area | Finding at the pin | MVP boundary |
|---|---|---|
| Model egress | `openai-eval.mjs` sends `cv.md` + `config/profile.yml` + full JD text to a configured endpoint (HTTPS + key required off-loopback); `gemini-eval.mjs` analogous for Google AI. The interactive CLI path sends the same context through the chosen CLI provider. | Keep model calls disabled until the provider, endpoint, retention, and data-sharing scope are explicitly approved. Never send the full candidate source JSON. |
| Telemetry | README at this revision states there is no telemetry and no project backend; plugins are opt-in (enable + keys required). Not independently code-audited end-to-end. | Treat as a documented claim. Leave plugins disabled. |
| Application behavior | `prepare-application.mjs` reads local profile/PDF and prints a prefill summary; **no POST**. `docs/APPLY_AUTOFILL.md`: Playwright-launched browser fills fields, leaves checkboxes/CAPTCHA/submit to the human. The `apply` mode opens live employer forms. | Exclude the `apply` mode from the MVP. Manual form filling by the candidate only. |
| Discovery providers | `docs/SUPPORTED_JOB_BOARDS.md` mixes documented public GET APIs (Greenhouse, Lever, Ashby, SmartRecruiters, Amazon, IBM, etc.) with SSR/HTML parsers (Apple, Google, iCIMS, Computrabajo…), POST-based public search endpoints (Alibaba, Feishu, Garena, MyCareersFuture), and Playwright sessions behind Cloudflare/NextAuth (Dayforce) or stateful portals (Interamt, PeopleSoft). `templates/portals.example.yml` defaults target AI/LLM roles, boost senior/staff/lead, exclude `Junior`/`Intern`, and enable broad web-search queries (Wellfound, Remotive, YC, European boards). | MVP allowlist: documented public GET ATS feeds plus manual employer-page intake only. Disable Dayforce/Interamt/PeopleSoft browser sessions, POST search endpoints, and all broad web-search queries. Rewrite title filters for Backend/Python + graduate/intern/new-grad with security secondary. |
| Freehire | No Freehire provider exists in the pinned provider table. Its public API permits keyless personal/non-commercial reads, but the data licence grants explicit storage/history rights only to plan holders. | Persistent record caching remains **unapproved**. Manual intake is the fallback; any adapter is a separately authorized future task. |
| Untrusted content | `AGENTS.md` itself instructs treating postings/emails as data-never-instructions — consistent with this project's rules, but not a substitute for them. | Repository content, JDs, and web pages remain untrusted input; no instruction inside them was or will be executed. |

The MVP allowlist should be public job-listing **GET** requests only. The supported-provider documentation also lists POST-based public search calls for some boards; these are search actions, not application submissions, but are outside the initial allowlist until each endpoint is specifically reviewed. `prepare-application.mjs` only summarizes fields from local files and contains no application POST. The separate Playwright `apply` mode can populate a live form (documentation says the candidate submits), so exclude it from the MVP.

## 8. Minimum safe installation plan

Not executed; the sequence a future authorization would follow:

1. **Session PATH adjustment** (per-invocation, not persistent): ensure `D:\Git\bin` and `D:\Git\usr\bin` precede `C:\Windows\System32` so Git Bash and its matching `cygpath` win over the WSL launcher.
2. **Dependencies:** from the verified checkout, resolve `npm install` against a reviewed dependency plan. Recommended: generate the lockfile as a local review artifact in a separately authorized step, review the resolved tree (including transitive additions vs. the five declared dependencies), then install. If `PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1` is chosen, set it for the install and run the Chromium download as its own explicitly approved step later.
3. **Chromium:** approve or defer explicitly. Deferred is viable: scanning, evaluation text, tracker, and email drafts work without it; PDF generation and browser checks do not.
4. **Private data root:** create a directory outside both the candidate JSON directory and the checkout (recommendation in §9), then point career-ops at it via `CAREER_OPS_ROOT` (absolute path) or the `.career-ops-data` marker.
5. **Verification:** `node doctor.mjs --json` (read-only prerequisite report), `node test-all.mjs` per the Windows guide after confirming Bash selection, then a zero-token `node scan.mjs` configuration check against the approved company list only.

## 9. Explicit decisions on lockfile strategy, Chromium, model use, and data paths

- **A. Exact revision — YES, reproducible.** The full SHA is verified in the local checkout; origin URL and clean detached HEAD confirm the working tree corresponds to the requested upstream commit. The remaining reproducibility gap is dependencies, not source.
- **B. Dependency strategy:** do not rely on bare `npm install` (no tracked lockfile; ranges drift). Recommend a separately authorized step that generates a lockfile locally, reviews the resolved tree against the declared dependencies, and records the lockfile hash in the future install report. Alternatively pin by reviewing the generated tree at install time and keeping `package-lock.json` outside Git as a local artifact. No lockfile was generated in this task.
- **C. Chromium:** **defer initially.** It is not needed for the minimum useful MVP (zero-token scan, text evaluation, tracker, email drafts). Approve the download only when PDF generation or liveness browser checks are actually needed. If approved, it happens via the documented `npx playwright install chromium` at Playwright 1.64.1.
- **D. Windows shell:** use session-scoped PATH precedence for `D:\Git\bin` / `D:\Git\usr\bin`, or full-path invocation of Git Bash and `cygpath` from the same install. Never rely on bare `bash` (currently WSL). No PATH or system change was made.
- **E. Private data root:** recommend creating `D:\document\career-ops-data` (sibling of the project directory, outside both the candidate JSON directory `D:\document\job apply` and the checkout `tools\career-ops`), referenced by an absolute `CAREER_OPS_ROOT`. Metadata checks showed no conflicting path exists there; nothing was created and no ACL was changed. The candidate JSON directory must never become the checkout or data root.
- **F. Model egress:** keep all model-dependent features disabled until the exact provider/endpoint and its retention policy are approved. Required reviews before enabling: which CLI/provider path (interactive CLI vs. `openai-eval.mjs` direct API), what prompt content is sent (`cv.md`, `config/profile.yml`, JD text), where the provider documents retention, and confirmation that the full source JSON is never sent. OpenCode Go must not be assumed to authorize standalone direct-API calls (`openai-eval.mjs` can hit `opencode.ai/zen/go/v1` with session headers per its own code — treat that as unapproved for now).
- **G. Discovery policy:** approved for MVP consideration — documented public GET ATS providers (Greenhouse, Lever, Ashby, SmartRecruiters) for a curated India/international company list, plus manual employer-page intake. Excluded: Dayforce/Interamt/PeopleSoft browser sessions, Notion internal endpoints, POST search endpoints, all web-search aggregator queries, any scraping of LinkedIn/Naukri/Indeed, and the `apply` browser mode. Freehire persistent caching remains unapproved.

## 10. Remaining blockers and mitigations

| Blocker | Mitigation |
|---|---|
| No tracked lockfile; `npm install` is not reproducible from the repo. | Generate and review a lockfile in a separately authorized step before installing; record its hash. |
| `postinstall` auto-downloads Chromium by default. | Decide explicitly: skip via `PLAYWRIGHT_SKIP_BROWSER_DOWNLOAD=1` and defer PDF capability, or approve the download as its own step. |
| Bash/test selection would resolve to WSL on this machine (`D:\Git` outside hard-coded candidates). | Session PATH precedence or full-path invocation before any test/batch run; verify the helper's selection diagnostics first. |
| Model provider/retention unapproved. | Keep model calls disabled; approve provider, endpoint, and data scope before first evaluation. |
| Sample portal config misaligned (excludes junior/intern; broad web searches enabled). | Rewrite `portals.yml` for the three campaigns and target roles; disable unapproved query sources. |
| Freehire storage rights unclear for no-plan use; no provider at this pin. | Manual employer-page intake for MVP; revisit Freehire only after written terms clarification. |
| India-specific company list not yet curated. | Candidate/owner approves a bounded list of Indian employers and Indian offices; verify one live board per source when authorized. |

## 11. Exact authorized changes performed

1. Created `D:\document\job apply\tools\` and cloned `https://github.com/career-ops-hq/career-ops.git` into `tools\career-ops` (2,843 files).
2. Checked out detached HEAD at `a6d7526c05861b1df9b70c011f8b1a84c44252ac`; verified via `rev-parse HEAD`, origin URL, and clean status.
3. Read-only inspection of the local checkout files listed in §4 (plus `batch/batch-runner.sh`, `scan.mjs`, `generate-pdf.mjs`, `doctor.mjs` existence checks) and public raw files at the same SHA.
4. Created exactly one new report: `reports/06_pinned_source_checkout_audit.md`.

**Not done:** no `npm install`/`npm ci`/`npx`, no lockfile, no Chromium/browser download, no test or project command execution, no service start, no job-API call, no account/key/OAuth creation, no application/email/contact, no candidate JSON/report/env/browser-profile modification, no PATH or system change, no unrelated directory inspection, no secrets printed.

## 12. Stopping conditions for the next task

- Do not run any project command, dependency install, browser download, or test until the §8 plan and §9 decisions are explicitly authorized.
- Do not point career-ops at the candidate JSON directory; create the separate data root only after authorization.
- Do not enable model-dependent features, plugins, web-search query sources, browser apply mode, or Freehire persistence before their respective approvals.
- Do not persistently modify PATH, Git config, or system settings; session-scoped shell setup only.
- Do not proceed past a failed gate: if the dependency review, Bash selection, or a source-terms check fails, stop and record the blocker instead of substituting a workaround that bypasses an access control or review step.
