# Task 05 — Windows 11 Foundation Feasibility Audit

**Audit date/time:** 2026-10-28, environment checks at `2026-10-28T21:32:22+05:30`  
**Working directory:** `D:\document\job apply`  
**Primary:** `career-ops` commit `a6d7526c05861b1df9b70c011f8b1a84c44252ac`  
**Decision:** **CONDITIONAL GO** for a separately authorized installation; **not installed or tested** in this audit.

## 1. Executive decision

The pinned commit exists in `career-ops-hq/career-ops`, its README and Windows setup guide describe Windows operation, its license is MIT, and the machine has the documented Node.js minimum, Git, Git Bash, and ample free disk. The basic workflow is Node-based; Python, Bun, and Docker are not required. The project documents local files, draft-only email, and no application submission.

This is **not an unconditional GO**. Before installation:

1. Use a manual checkout of the exact commit, **not** `npx @santifer/career-ops init`, which the README says installs the latest release rather than the reviewed commit. The separate npm bootstrap package was not present in the pinned repository and was not audited.
2. Review and authorize npm dependency installation. The pinned root has no tracked `package-lock.json`; `.gitignore` excludes it and package dependencies include version ranges. Its `postinstall` script attempts to download Playwright Chromium automatically. The resolved dependency graph and browser download must be explicitly reviewed/approved before installing.
3. Resolve Git Bash explicitly. Bare `bash` currently resolves to the Windows WSL launcher, while the actual Git Bash is at `D:\Git\bin\bash.exe`. The Windows test guide lists Program Files/Scoop layouts, not this D: installation. Ordinary Node workflows may work, but the full test/batch path is not established without testing or explicit path handling.
4. Replace the example portal/title configuration before any scan. It is not suitable for this candidate: its title filters exclude junior/intern roles and it has broad enabled web-search queries, including third-party/remote boards. Allowlist only sources whose current access method and terms have been approved.
5. Resolve model-data egress. career-ops sends CV/profile and job text to the configured model provider; no model-provider data-sharing choice is authorized here. Do not submit the original candidate JSON to a model.
6. Treat Freehire as **not approved for persistent full-record caching** until its current terms explicitly clarify storage for a no-plan personal-use integration. Its docs permit personal/non-commercial reads, but the separate data-license paragraph expressly grants long-term storage/history rights to plan holders. Manual employer-page intake is the safe interim source.

**Fallback:** AI Job Hunter v0.6.3 is **not warranted yet**. Task 03 documents a Windows-oriented fallback, but it was not re-inspected or tested in Task 05; use it only if the career-ops Windows/source-policy gate fails, and audit it before use. If this primary remains blocked, manual employer-page searches plus one local Markdown ledger are safer than installing an unreviewed alternative.

## 2. Candidate source-file integrity

Preflight confirmed the working directory exactly as requested before any report, environment, or upstream inspection. Hashes were computed from each file’s raw bytes.

| Candidate file | Expected SHA-256 | Actual SHA-256 | Result |
|---|---|---|---|
| `full-profile.json` | `2f767145d8cb3bb619f0e77ff5f9f0e804a6026e1d2122e7b3fa50dcc9161fdb` | `2f767145d8cb3bb619f0e77ff5f9f0e804a6026e1d2122e7b3fa50dcc9161fdb` | **PASS** |
| `internships.json` | `606e03d0d98d783a00c0afb002a08dc1958b065ea245a9b232fee8b0d5da2aea` | `606e03d0d98d783a00c0afb002a08dc1958b065ea245a9b232fee8b0d5da2aea` | **PASS** |
| `projects.json` | `86749dc8d6f1aa9523e6a57f1decf3a74c911e18eb3ed3ee2000f16b8725e4f2` | `86749dc8d6f1aa9523e6a57f1decf3a74c911e18eb3ed3ee2000f16b8725e4f2` | **PASS** |

All matched. No candidate file was modified. Candidate facts and conflicts remain governed by Reports 01/02; in particular, do not infer exact PrimeTrade dates, Labmentix service/completion, individual project contributions, or country-specific work authorization.

## 3. Windows environment inventory

Read-only environment inventory collected **2026-10-28T21:32:22+05:30**. Version commands queried tool versions only; no project code was invoked. Environment-variable values and credentials were not read or printed.

| Tool/environment | Observed state | Relevance |
|---|---|---|
| Windows | Windows 11 Home Single Language, version `10.0.26200`, build `26200`, 64-bit | Matches the target platform. The upstream Windows support statement remains a documentation claim until installed/tested on this machine. |
| PowerShell | `5.1.26100.9711` | Present; suitable for read-only audit and documented PowerShell commands. |
| Node.js | `v22.17.1`, `D:\node\node.exe` | **Meets** the pinned package engine `^22.13.0 \|\| >=23.4.0`. |
| npm | `10.9.2`, `D:\node\npm.ps1` | Present. No package was installed. |
| Git | `2.53.0.windows.2`, `D:\Git\cmd\git.exe` | Present. No clone or Git operation was performed. |
| Git Bash / `cygpath` | `D:\Git\bin\bash.exe` and `D:\Git\usr\bin\cygpath.exe` exist. Bare `bash` resolves to `C:\Windows\system32\bash.exe` (the WSL launcher). | Git Bash exists but is not the PATH-selected Bash. Do not use bare `bash` for this project; use the matching Git Bash and `cygpath` from the same installation if later authorized. |
| Python | `3.13.3`, `C:\Users\Dell\AppData\Local\Programs\Python\Python313\python.exe`; `py` launcher also resolves to 3.13.3 | Present but **not required** by the documented core career-ops workflow. |
| Go | `go1.25.7 windows/amd64`, `D:\Go\bin\go.exe` | Optional dashboard TUI only; setup docs list Go 1.21+. |
| Bun | Not found on PATH | Not required by the documented Node/npm install. |
| Docker | `docker` not found on PATH; Docker Desktop absent at the two checked standard executable paths; `com.docker.service` stopped; no Docker Desktop process running | Not required by career-ops. Reactive Resume is not selected, so this is not a primary-path blocker. No whole-machine search was done. |
| OpenCode CLI | `opencode.ps1` found at `C:\Users\Dell\AppData\Roaming\npm\opencode.ps1`; version not queried | CLI command is discoverable. The exact model/provider entitlement and current account configuration were not inspected. |
| D: disk | `180,778,082,304` bytes free (about `168.3 GiB`) | Ample by capacity for a checkout/dependencies/browser download; actual install size and browser cache were not tested. |

The current working directory is not a Git repository per the workspace metadata. No `career-ops` checkout or `career-data` folder exists at the exact paths checked. Career-ops documents `CAREER_OPS_ROOT`/`CAREER_OPS_DATA_DIR` and a data-root marker, so a separate absolute data directory can be selected later; no directory was created and no ACL for a future location was assessed. Do not use the current candidate JSON directory as the career-ops checkout or data root.

## 4. Pinned repository and licence verification

- The public commit page at [`career-ops@a6d7526c05861b1df9b70c011f8b1a84c44252ac`](https://github.com/career-ops-hq/career-ops/commit/a6d7526c05861b1df9b70c011f8b1a84c44252ac) loads under the requested repository and shows the full SHA. GitHub’s page identifies it as a tracker-path error-message fix, committed **2026-10-28 11:58 UTC**, changing `set-status.mjs` and its missing-tracker test. This is the requested pin; no newer branch was substituted.
- Read the [README at the pinned SHA](https://raw.githubusercontent.com/career-ops-hq/career-ops/a6d7526c05861b1df9b70c011f8b1a84c44252ac/README.md), [Windows guide](https://raw.githubusercontent.com/career-ops-hq/career-ops/a6d7526c05861b1df9b70c011f8b1a84c44252ac/docs/WINDOWS.md), [setup guide](https://raw.githubusercontent.com/career-ops-hq/career-ops/a6d7526c05861b1df9b70c011f8b1a84c44252ac/docs/SETUP.md), [package.json](https://raw.githubusercontent.com/career-ops-hq/career-ops/a6d7526c05861b1df9b70c011f8b1a84c44252ac/package.json), [.gitignore](https://raw.githubusercontent.com/career-ops-hq/career-ops/a6d7526c05861b1df9b70c011f8b1a84c44252ac/.gitignore), and [LICENSE](https://raw.githubusercontent.com/career-ops-hq/career-ops/a6d7526c05861b1df9b70c011f8b1a84c44252ac/LICENSE). The source declares **MIT**. The README’s Windows claim is “without a VM or WSL”; it calls out Git Bash for scripts that shell out to Bash.
- The pinned README’s project facts are documentation claims, not runtime verification. It states that the tool has no project telemetry/backend; candidate files go to the model provider selected by the user. The plugin docs say plugins are opt-in and require both an enable/consent step and configured keys. These are useful stated controls, not a full security audit.
- The standard `npx @santifer/career-ops init` path is **not a pinned install**: the README/setup docs say it runs an installer that clones the latest release and installs dependencies. The `@santifer/career-ops` bootstrap is a separate npm package, not code contained in this pinned repository, and its implementation was not inspected. Do not use this path for the reviewed SHA.
- The documented manual setup is a Git clone followed by `npm install`; to preserve this exact pin later, an authorized checkout would need to select the SHA explicitly rather than follow `main`/latest release. No clone occurred in this audit.

## 5. Runtime and installation requirements

### Runtime/dependency evidence

At the pinned revision, `package.json` declares package version `1.38.0`, `private: true`, Node engine `^22.13.0 || >=23.4.0`, and dependencies including Playwright `1.64.1`, `undici`, `dotenv`, and `js-yaml`. The installed Node `22.17.1` meets the declared engine. Setup docs list Node.js 22.13+ and Git as prerequisites; Go is optional for the dashboard. Python, Bun, and Docker are not listed as core prerequisites.

No root `package-lock.json` or `npm-shrinkwrap.json` exists at the inspected pin; the root `.gitignore` excludes `package-lock.json`. Several dependency declarations use semver ranges, so an `npm install` at a later date may resolve a different dependency tree even when source commit is fixed. A reproducible dependency lock/approval approach is unresolved. Do not silently generate or commit a lockfile as part of this audit.

The pinned root `package.json` has a `postinstall` script. Unless its skip flag is explicitly set, it tries `npx playwright install chromium` (retries once) and warns if installation fails. That downloads a browser binary during dependency installation, despite the separate docs presenting Chromium as needed for PDF/browser features. Chromium is not installed in this task. Approve the download separately or decide whether the supported install can safely defer it; do not infer from the docs alone that `npm install` is network/package-only.

### Windows method and path risks

- The Windows guide says routine code is Node-based and does not require WSL. `batch/batch-runner.sh` and some test fixtures do invoke Bash.
- The guide’s documented Bash candidates are Git-for-Windows under Program Files or Scoop. This machine’s Git is at `D:\Git`; `D:\Git\bin\bash.exe` exists, but bare `bash` resolves first to `C:\Windows\system32\bash.exe` (WSL launcher). The matching `D:\Git\usr\bin\cygpath.exe` is present.
- The pinned test helper enumerates Program Files/Scoop Git Bash paths and also probes PATH. Because PATH currently returns WSL Bash and D:\Git is outside the named paths, the Bash-dependent full test path is **not established**. The helper’s exact behavior has not been run; do not assume tests will select the correct Bash. Before relying on the full suite, verify its selection or use a separately authorized environment/path adjustment that selects Git Bash and its matching `cygpath`.
- PDF generation uses Playwright Chromium. The CLI commands and static discovery do not require Docker. Go is available for the optional TUI. Docker Desktop is absent/stopped in checked locations and is not needed for this primary.

### Installer safety and pinning

The pinned source’s `npm install` is documented, but its postinstall downloads Chromium and the dependency graph is not locked. The one-command npm bootstrap is separate/unpinned and fetches the latest release. Therefore the safe future route is **not approved yet**: before installation, the user must approve a manual exact-SHA checkout, the dependency resolution/lock plan, and whether Chromium download is needed. Do not execute these commands during Task 05.

## 6. Source/provider permissions and coverage

### What career-ops documents

- README claims 100+ preconfigured companies, 35+ search queries, and 55+ provider modules. The pinned `SUPPORTED_JOB_BOARDS.md` labels providers as API, Atom/RSS/XML, or parser; the repository therefore mixes public structured feeds with page parsers and aggregator search. These breadth figures are project documentation, not coverage measured in this audit.
- The official/public GET API providers include Greenhouse, Lever, and Ashby; SmartRecruiters is also in the provider catalogue. These are per-company/tenant sources, not universal vacancy databases. Preserve the employer’s original posting URL and validate that the endpoint and employer’s terms remain suitable.
- The sample `templates/portals.example.yml` is **not suitable as the candidate’s config**. Its title filter is an AI/LLM/agent/automation example, boosts senior/staff/head/director, explicitly excludes `Junior`, `Intern`, `Interns`, and `Internship`, and has no active location filter by default. Several web-search queries are enabled for Wellfound, remote aggregators, YC, and European/Spanish boards. Replacing role/seniority filters and manually reviewing/disable-by-default source queries is a prerequisite.
- The provider docs describe methods that should be excluded from this system unless a future source-specific authorization review allows them. Examples at the pin include **Notion** endpoints explicitly called undocumented/internal, **Dayforce** flow involving Cloudflare cookies/NextAuth CSRF handling, and numerous SSR/parser sources. The project’s own docs mark EchoJobs retired because of HTTP 429/robots restrictions and say it does not work around bot protection. Do not enable challenge/session, login-wall, undocumented, or anti-bot-dependent connectors.
- The opt-in plugin documentation says plugins can add network, ingest, search, export, or notification behavior; it also states plain ESM plugins are “sandboxed-by-convention,” not a hard sandbox. Leave all plugins disabled in the MVP. The source list also contains search queries that depend on a web-search provider; do not assume that source, terms, quota, or cost are included in OpenCode Go.

### Freehire decision

Current [API docs](https://freehire.me/docs/api), [terms](https://freehire.me/terms), and [plans/data-license page](https://freehire.me/jobs-api) were read on **2026-10-28**; no job endpoint was called and no account/key was created.

- **Personal use:** current terms/API docs permit keyless public read endpoints for personal, non-commercial job search/research/tool use. No key is needed for public reads; current plan page describes per-address rate limits (including a 600-request/minute no-key figure) and API responses carry rate-limit headers. Stop on `429`; never rotate IP/proxy to evade it.
- **Filters:** the docs expose `countries`, `regions`, `cities`, `work_mode`, `seniority` (`intern`, `junior`, etc.), `employment_type`, experience bounds, `relocation`, and `visa_sponsorship`; the filter fields are not all employer-originated. Country and applicant-location restrictions are useful discovery fields; sponsorship/seniority/experience may be enrichment, not a quoted employer requirement. `posted_at`, `created_at`, `updated_at`, `closed_at`, the original posting `url`, and source type are documented, but freshness still needs employer verification.
- **Local storage:** terms expressly grant storage/history/model use to API-plan holders. Without a plan they allow a personal/non-commercial search or tool for oneself, but do not state a clear retention period or unequivocal right to cache full posting text. The job data remains employer-owned. Per the task’s ambiguity rule, **persistent Freehire records/full descriptions are not approved** until this is clarified. A transient query or manual, candidate-authored tracker entry containing an employer link is the interim path; do not mirror/redistribute the raw catalogue.
- **Attribution/redistribution:** the paid product license prohibits raw-data resale/redistribution and stops display when the plan ends. Regardless of plan, preserve source and employer links; no public feed/product is contemplated.
- **Coverage:** the schema can distinguish country, work mode, seniority, experience, relocation, and visa-related signals better than a remote-only feed. It cannot by itself prove the employer currently accepts a candidate in India. Missing/unknown values remain unknown. The candidate’s campaign eligibility needs source text and candidate confirmation.
- **Integration status:** no Freehire provider was found in the pinned supported-board table inspected. A custom read-only adapter would be required if retained; it is **not approved** until the no-plan local-storage terms are clarified. This does not block the manual/direct-company MVP.

## 7. Privacy and security assessment

| Data/action | Evidence at pinned revision | Boundary / unknown |
|---|---|---|
| Candidate profile/resume | README and `openai-eval.mjs` say `cv.md`, profile, and full JD are sent to the configured model endpoint; remote endpoints require HTTPS and an API key in the direct-script path. | No candidate data was sent in this audit. If using an external CLI provider later, obtain explicit candidate/provider approval and minimize the prompt. Go is not assumed to authorize a separate direct API call. |
| Local files and generated materials | README/path resolver documents `cv.md`, `config/profile.yml`, `portals.yml`, `data/`, `reports/`, `output/`; `CAREER_OPS_ROOT`/`CAREER_OPS_DATA_DIR` support a separate data root. `.gitignore` ignores user data, reports/output, `.env`, documents, tracker DB, and backups. | `.gitignore` is not access control. Keep the immutable JSON in its existing directory and do not copy it into the checkout. A future absolute external data root can separate working profile/outputs from code; no root was created and ACLs of a future path were not checked. |
| Secrets | `.env` and local CLI settings are ignored; direct evaluator reads provider keys from environment or CLI arguments and sends them to the chosen HTTPS endpoint. | Do not put keys in arguments, `cv.md`, reports, or screenshots. Secret-manager/Windows Credential Manager integration was not established for the core workflow. Do not inspect current shell variables. |
| Telemetry/sharing | Pinned README asserts no telemetry or project backend; it says the only data egress is the selected AI provider. Plugins are opt-in and disabled by default. | These are maintainer statements; no comprehensive source/dependency telemetry audit was performed. Hired-story/issue contributions are public/voluntary; never submit candidate data to them. |
| Browser/apply path | `prepare-application.mjs` source reads a local profile/PDF, validates supported HTTPS ATS hosts, prints suggested fields, and contains no HTTP application POST. `docs/APPLY_AUTOFILL.md` says Playwright opens a project-launched browser, fills fields, leaves CAPTCHA/check-box decisions to the user, and never submits. | This is not a complete audit of all browser code or third-party page behavior. Filling a live form is still a consequential external action; do not use in MVP. Never handle CAPTCHA/MFA/attestations automatically. |
| Plugins/search | `docs/PLUGINS.md` says integrations are opt-in; the plugin mechanism is not a hard sandbox. The sample portal config uses enabled web-search queries and providers beyond documented ATS GET feeds. | Leave plugins disabled and disable broad web-search/aggregator queries until each provider, terms, and data egress are approved. No source scan or API call was run. |

The MVP allowlist should be public job-listing **GET** requests only. The supported-provider documentation also lists POST-based public search calls for some boards; these are search actions, not application submissions, but are outside the initial allowlist until each endpoint is specifically reviewed. `prepare-application.mjs` only summarizes fields from local files and contains no application POST. The separate Playwright `apply` mode can populate a live form (documentation says the candidate submits), so exclude it from the MVP.

## 8. Blockers and mitigations

| Finding | Severity / resolution before install |
|---|---|
| `npx ... init` fetches latest release, not the proposed pin; installer is separate and uninspected. | **Block that installer path.** If authorized later, use a manual exact-SHA checkout and review each command before execution. |
| Root dependency install has no tracked lockfile and `postinstall` downloads Chromium. | **Bounded install gate.** Decide whether to create/review a dependency lock in a separately scoped task; explicitly approve or defer browser download. The dependency graph and install hook were not executed. |
| Bare `bash` is WSL launcher; Git Bash is in a nonstandard D: path. The test guide lists Program Files/Scoop; tests were not run. | **Test gate.** Use a deliberately resolved Git Bash path and matching `cygpath` or separately configure PATH after authorization; verify test-helper selection before relying on test results. |
| Sample query/title config is misaligned and broad. | **Configuration gate.** Replace it with a reviewed Backend/Python + secondary security profile, include graduate/intern/new-grad terms, remove negative filters that exclude those roles, and disable broad web-search queries until source approval. |
| Mixed public APIs, SSR parsers, undocumented endpoints, and challenge-dependent sources. | **Source gate.** Allowlist only current documented public GET/feeds and explicitly permitted company pages. Disable Dayforce, Notion internal APIs, challenge/session sources, and all social-network scraping. |
| Freehire retention rights for no-plan persistent records are not explicit; its provider is absent from the pinned catalogue. | **Integration not approved.** Use manual company ATS paths until written terms clarification; if later allowed, a narrow adapter must preserve source and freshness fields and check ignored filters. |
| CV/JD are sent to configured AI service; no local Ollama command was found. | **Privacy/consent gate.** Use only an explicitly approved CLI provider; otherwise operate manually/deterministically without model calls. Never submit the source JSON wholesale. |
| Actual India target-company list not present/verified in Task 05. | **Coverage gate.** Candidate or project owner must approve a bounded list of Indian employers/offices; verify board URL and one live posting when installation is separately authorized. Do not claim India-wide completeness. |

## 9. Primary/fallback recommendation

### Primary: conditional use of pinned career-ops

The commit and MIT license are verified; Node 22.17.1 meets the declared runtime, and the repository documents a Windows/no-WSL path. The project is technically plausible on this machine. The recommendation is **CONDITIONAL GO**, not GO, because the pin-safe install, lock/dependency resolution, postinstall browser download, Bash-dependent test selection, model-provider egress, source-template reset, and Freehire retention must be resolved before a separately authorized install.

When those gates pass, use only documented/approved public ATS GET providers plus manual employer-page intake. Do not copy the sample portal configuration unchanged. Use a separate absolute `CAREER_OPS_DATA_DIR`; keep candidate JSON outside the checkout; first use a verified minimal `cv.md`; use no opt-in plugins or browser apply flow.

### Fallback: not warranted yet

AI Job Hunter v0.6.3 was compared in Task 03, but **not inspected or installed in this Task 05 audit** because career-ops is not yet genuinely blocked: its documented Node/Windows requirements are met and its direct-feeds/manual route remains possible. If the primary fails Task 05’s install/safety gate, a separately authorized fallback audit must verify AI Job Hunter’s pinned release, current source permissions, default telemetry/submit settings, and Windows setup before use. Never run both trackers. If both tools fail, use manual public employer pages and one local Markdown ledger.

## 10. Exact changes or authorizations required before installation

No changes are authorized or made here. Before a future installation, the owner must explicitly approve:

1. **Exact-source checkout:** manual checkout at SHA `a6d7526c05861b1df9b70c011f8b1a84c44252ac`; do not use the latest-release `npx` bootstrap.
2. **Dependency resolution:** a reviewed install plan for the unpinned root dependency graph. If a lockfile is needed, create/review it only in a separately authorized task.
3. **Browser download:** whether the automatic Playwright Chromium `postinstall` download is allowed or should be safely deferred; PDF generation will not be ready without Chromium.
4. **Windows test shell:** how the project/test helper will select `D:\Git\bin\bash.exe` and its matching `cygpath`, without accidentally invoking `C:\Windows\System32\bash.exe`/WSL.
5. **Private data root:** choose and, if needed, create an absolute directory outside the source checkout; inspect its ACL before placing candidate working data there. Do not set it to the candidate JSON directory.
6. **Model egress:** select an already-authorized CLI/model endpoint or approve no model use. Confirm the model provider receives resume/JD context; do not treat OpenCode Go as a separate application API allowance.
7. **Source configuration:** approve exact company URLs and API terms; replace the example role/location filters. Do not persist Freehire job descriptions/records unless its no-plan storage permission is clarified.

## 11. Completion and no-modification confirmation

- **Status:** CONDITIONAL GO, with the bounded pre-install gates above.
- **Pinned revision:** commit exists in the requested repository and matches the proposed full SHA; pinned README, Windows/setup docs, `package.json`, `.gitignore`, `LICENSE`, provider catalog, portal template, plugin docs, and relevant apply/evaluator source were read read-only.
- **Candidate hashes:** all three matched before and after the audit; exact results appear in §2.
- **Files changed:** exactly one new report, `reports/05_windows_foundation_feasibility.md`. Candidate JSON, reports 01–04, application code/configuration, and environment settings were not modified.
- **Actions not taken:** no package/tool install, clone, project command/test, local service start, Freehire/job API call, credential access, API-key creation, account change, application submission, email/message, unrelated directory scan, or fallback inspection.
- **Limits:** no runtime installation or test was performed; npm resolution/browser install behavior, Git Bash test selection, actual candidate-specific company coverage, model-provider quotas/retention, and Freehire persistence permission remain unresolved. The report makes no runtime or coverage guarantee.

### Read-only sources accessed 2026-10-28

- [Pinned commit](https://github.com/career-ops-hq/career-ops/commit/a6d7526c05861b1df9b70c011f8b1a84c44252ac), [pinned README](https://raw.githubusercontent.com/career-ops-hq/career-ops/a6d7526c05861b1df9b70c011f8b1a84c44252ac/README.md), [Windows guide](https://raw.githubusercontent.com/career-ops-hq/career-ops/a6d7526c05861b1df9b70c011f8b1a84c44252ac/docs/WINDOWS.md), [setup guide](https://raw.githubusercontent.com/career-ops-hq/career-ops/a6d7526c05861b1df9b70c011f8b1a84c44252ac/docs/SETUP.md), [package manifest](https://raw.githubusercontent.com/career-ops-hq/career-ops/a6d7526c05861b1df9b70c011f8b1a84c44252ac/package.json), [license](https://raw.githubusercontent.com/career-ops-hq/career-ops/a6d7526c05861b1df9b70c011f8b1a84c44252ac/LICENSE).
- Pinned source files: [.gitignore](https://raw.githubusercontent.com/career-ops-hq/career-ops/a6d7526c05861b1df9b70c011f8b1a84c44252ac/.gitignore), [example portal config](https://raw.githubusercontent.com/career-ops-hq/career-ops/a6d7526c05861b1df9b70c011f8b1a84c44252ac/templates/portals.example.yml), [supported boards](https://raw.githubusercontent.com/career-ops-hq/career-ops/a6d7526c05861b1df9b70c011f8b1a84c44252ac/docs/SUPPORTED_JOB_BOARDS.md), [plugin model](https://raw.githubusercontent.com/career-ops-hq/career-ops/a6d7526c05861b1df9b70c011f8b1a84c44252ac/docs/PLUGINS.md), [apply flow docs](https://raw.githubusercontent.com/career-ops-hq/career-ops/a6d7526c05861b1df9b70c011f8b1a84c44252ac/docs/APPLY_AUTOFILL.md), [prepare-application source](https://raw.githubusercontent.com/career-ops-hq/career-ops/a6d7526c05861b1df9b70c011f8b1a84c44252ac/prepare-application.mjs), [OpenAI-compatible evaluator](https://raw.githubusercontent.com/career-ops-hq/career-ops/a6d7526c05861b1df9b70c011f8b1a84c44252ac/openai-eval.mjs), [path resolver](https://raw.githubusercontent.com/career-ops-hq/career-ops/a6d7526c05861b1df9b70c011f8b1a84c44252ac/path-resolver.mjs), and [test helpers](https://raw.githubusercontent.com/career-ops-hq/career-ops/a6d7526c05861b1df9b70c011f8b1a84c44252ac/tests/helpers.mjs).
- [Freehire API docs](https://freehire.me/docs/api), [Freehire terms](https://freehire.me/terms), and [Freehire data-plan terms](https://freehire.me/jobs-api).
