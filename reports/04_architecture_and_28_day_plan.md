# Task 04 — Architecture Decision and 28-Day Delivery Plan

**Plan date:** 2026-10-28  
**Delivery window:** 2026-10-28 through 2026-11-24 inclusive. Research is complete; **27 implementation days remain, 2026-10-29–2026-11-24**.  
**Status:** Decision and plan only. No implementation or installation was performed.

## 1. Executive decision

### Decision: career-ops + permitted public sources + one local Markdown tracker

Use **career-ops as the sole MVP workflow, discovery, job-evaluation, document-drafting, and application-tracking foundation**. Pin the reviewed code to commit [`a6d7526c05861b1df9b70c011f8b1a84c44252ac`](https://github.com/career-ops-hq/career-ops/commit/a6d7526c05861b1df9b70c011f8b1a84c44252ac) (inspected 2026-10-28), subject to Task 05 rechecking that commit, its Windows instructions, and license before any installation. Reuse its provider modules only where the employer/API source is documented and allowed. Configure a small, curated company-career list, not a crawler of every company.

Add **one narrow, read-only Freehire API source** for country-filtered India and remote-from-India discovery if a provider is not already present in the pinned career-ops source list. Its public API is documented for personal, non-commercial use without a key; current terms must be rechecked before use. Do not send candidate data to Freehire. Keep **career-ops `data/applications.md` as the only application tracker**. Do not run Reactive Resume or another tracker in parallel in the MVP.

The working candidate profile is a manually reviewed, private projection of the three immutable JSON source files. Every material resume claim remains traceable to a source JSON Pointer and current file hash in a local provenance note. Unresolved employment, dates, contributions, education, credential, and work-authorization facts are omitted from generated materials until the candidate confirms them. Use career-ops’ PDF/Markdown and draft-only features; do not add a second resume pipeline just to obtain DOCX. If a specific vacancy requires DOCX, create that one document manually from approved text using an already available editor, or ask for a separately scoped Reactive Resume decision.

**Fallback:** if career-ops fails the Windows/safety feasibility gate or cannot operate without prohibited sources, use **[AI Job Hunter v0.6.3](https://github.com/Insideofcub/ai-job-hunter/releases/tag/v0.6.3)** (the first tagged release, 2026-10-19) as a single local Windows workflow and SQLite tracker. Disable JobSpy, LinkedIn/Naukri/Indeed scraping, browser auto-submit, and any source whose terms are not established. Use manually curated company ATS links and the same source, eligibility, provenance, and human-approval rules. Do not run both systems or migrate two trackers.

This choice prioritizes relevant, reviewable opportunities and truthful applications over maximizing records or automation. It uses no paid API or subscription by default, leaves a usable search workflow early in the schedule, and limits custom work to source configuration and at most one read-only adapter. The README-level feature review is not a tested Windows installation; Task 05 is a no-install feasibility audit and a no-go gate, not authorization to install.

## 2. Inputs and constraints

- Read the Task 01 audit, Task 02 correction report, and Task 03 research report, including Task 03’s appended post-research hash verification.
- Before reading those reports, confirmed the working directory is exactly `D:\document\job apply` and recomputed the candidate-file hashes from raw bytes. All three matched; results are recorded in §11.
- Candidate evidence is limited to the JSON/report contents. The candidate’s stated 2027 graduation is an expected completion, not a completed degree. PrimeTrade dates, Labmentix service/completion and date format, seven Labmentix-project contributions, and other conflicts remain unresolved. The broad overseas work-authorization statement is not a country-specific legal answer.
- Main target: backend/Python engineering. Secondary: cloud security/cybersecurity. Prioritize internships, graduate/new-grad and early-career roles suitable for an expected 2027 graduate; do not silently treat a junior label or inferred seniority as source truth.
- Campaigns remain distinct: **A** India roles, **B** international remote explicitly permitting work from India, **C** relocation/sponsorship with role-specific evidence.
- No code or installation is authorized by this planning task. No paid API, account change, external contact, application, or message is authorized.
- Primary architecture evidence: Task 03 describes career-ops as MIT, local-file based, OpenCode-compatible, Windows-documented, and draft-only; its actual installation, dependencies, source terms, and provider list still need inspection at the pinned commit. Freehire is an optional source, not a reason to self-host its full stack. Reactive Resume’s DOCX/version/tracker advantages do not justify its additional PostgreSQL/upload-storage service for this MVP.

## 3. Selected architecture and rejected alternatives

### Primary: one career-ops workspace

**Reused unchanged, subject to Task 05 confirmation:**

- career-ops evaluation, source providers, company portal configuration, local Markdown/YAML/TSV workspace, PDF/Markdown generation, contact/company research drafts, follow-up helpers, and `data/applications.md` tracker.
- Official documented public GET endpoints already represented by providers, starting with employer-specific Greenhouse, Lever, Ashby, and SmartRecruiters boards. Company board URLs/tokens are curated and attributable.
- Candidate JSON only as read-only evidence. Do not put the original JSON, private identifiers, DOB, unreviewed work authorization, live operational URLs, or secrets into the agent workspace.

**MVP integrations:**

1. **Direct company career pages/ATS GET feeds** for Indian employers and Indian offices, plus a bounded list of international targets. They are the primary path for India office, hybrid, and onsite vacancies.
2. **Freehire read-only search** for an India-specific path (`countries=IN`) and international remote leads with applicant-country restrictions. Recheck its personal-use terms and current API behavior. If career-ops already has a working provider, reuse it; otherwise add only a small adapter that preserves the original URL, provider, fetched-at time, source-posted time when supplied, and response fields. If terms, freshness, or Windows integration fail the gate, use manual career-page/ATS searches rather than a paid or scraped substitute.
3. **Manual job intake** for employers not covered by the feeds. Record the original employer URL and observed date; no unsupported page crawler.

Himalayas is not required initially: it is remote-only, refreshed daily, requires visible attribution, and its supplied apply link routes through Himalayas. Adzuna needs an explicitly approved key/account and is not needed to establish the first India path. Recent Funding API terms conflict with its docs; funding discovery stays manual and source-linked. No browser form filling or Gmail integration is required.

### One canonical ledger and traceability

- **Tracker:** career-ops `data/applications.md`, and only that tracker. Keep its status vocabulary; add evidence/source links in supported notes rather than creating a parallel database or tracker.
- **State distinctions:** `discovered` → `reviewed` → `prepared` → `filled` → `submitted` → `confirmed` are not interchangeable. Also allow `closed`, `rejected`, `withdrawn`, and **`outcome unknown`**. A filled form is not submitted; a click without explicit employer confirmation is not confirmed. No blind retry from an unknown outcome.
- **Candidate facts:** retain original JSON as immutable baseline. Maintain a minimal working `cv.md` from only candidate-approved facts. A small local claim map records resume bullet/answer → source file + JSON Pointer + source hash + status (`candidate-reported`, `source-reviewed`, `conflicting`, or `unknown`). It is a review aid, not an assertion that a source artifact was independently verified.
- **Resume versions:** keep the base profile read-only after approval; generate a new job-specific file for each selected vacancy with company, role, date, canonical job URL, job-description snapshot/hash, and base-profile hash in its filename or manifest. Never overwrite the base profile. Human review is required before the file is used externally.
- **Outreach:** store public contact/source link, role/company context, draft, and next follow-up in the same application record. Drafts only; a separate candidate action is required to send.

### Rejected for MVP

- **Reactive Resume:** useful if DOCX, named document versions, or richer contacts/tracker become hard requirements. It adds a local multi-container app and overlapping features; no candidate need currently justifies that setup. Reconsider only if an application specifically requires DOCX or career-ops cannot meet document/version/track acceptance.
- **Freehire full self-hosted application:** broad documented feature set but far more database/search/storage services than needed; its personal-use public API is enough for a source supplement.
- **JobNavigator, JobApply, JobSpy, Pipeline, JobAppAgent:** documented scraping, undocumented guest APIs, privacy defaults, API terms, unsupported source behavior, or setup/security risks outweigh their feature breadth. Never enable prohibited connectors.
- **Fireround/Firecrawl and Recent Funding paid tiers:** costs and integration complexity are not justified for a single candidate’s 28-day MVP; Recent Funding has a docs/terms discrepancy. Funding can be researched manually.
- **A bespoke app, custom SQL database, CRM, Selenium layer, inbox scanner, paid model API, or synchronized second tracker:** none is necessary to prove the target workflow.

## 4. Component and data-flow diagram

```text
Immutable candidate JSON + Task 01/02 evidence
                 │ human selects/clarifies facts; source hashes remain fixed
                 ▼
Private reviewed cv.md + local claim/source map
                 │ minimum candidate context to chosen OpenCode CLI only
                 ▼
Company ATS GET feeds ─┐
Freehire read-only API ├─> normalize + preserve source/observed-at + dedup review
Manual employer URLs ──┘                    │
                                             ▼
                         country/work-mode/experience/eligibility triage
                                             │
                                             ▼
                     reasoned job-fit report + source-linked shortlist
                                             │ human selects
                                             ▼
                 career-ops PDF/Markdown + application-question/email drafts
                                             │ human reviews every claim and field
                                             ▼
                   one local applications.md ledger → candidate acts manually
```

Funding sits beside—not inside—the eligibility or fit truth path:

```text
Funding story → company/investor source verification → canonical company/domain
             → current employer career page/ATS vacancy → separate role eligibility check
```

No inbound job description, website text, model response, or repository instructions may trigger commands, package installs, file edits, credential access, or external actions. Treat such content as quoted, untrusted data. No public network listener is needed.

## 5. MVP scope and deferred features

### Must ship by 24 November

- Windows 11-compatible, pinned career-ops workspace or the explicit AI Job Hunter fallback; private local data paths and one tracker.
- Two functioning discovery paths: (1) at least one India-specific source (curated Indian company ATS pages and/or the permitted `countries=IN` Freehire search) and (2) remote listings whose source fields can be checked for India eligibility. Manual company-page intake covers gaps.
- Three separate campaign profiles and a consistent fact/evidence vocabulary.
- Source-preserving normalization, duplicate review, open/closed/unknown status, observed-at date, and source-posted date only when provided by the source.
- A human-readable fit rank and reasons for backend/Python first, cloud security/cybersecurity second, with entry-level/new-grad priority and a review route for strong roles whose seniority/experience is unstated.
- Human-reviewed role-specific PDF/Markdown/application-answer drafts grounded in approved facts. A separate local claim map makes material claims traceable.
- One application ledger with distinct prepared/filled/submitted/confirmed/unknown statuses, source URL, and outreach draft/follow-up note.
- Source-linked funding/company research that never treats funding as proof of a vacancy.
- Mock/fixture tests for normalization, eligibility, scoring boundaries, duplicates, missing dates/countries, expired/closed sources, and source failure. No application or email submission in tests.
- Final acceptance report with commands/results, hashes, known limitations, and explicit pass/fail/no-go decisions.

### Deferred or abandoned

| Feature | Decision |
|---|---|
| CAPTCHA/MFA, security challenge, anti-bot, login-wall, or legal-attestation automation | **Abandon.** Pause and ask the candidate; never bypass. |
| LinkedIn scraping or automated LinkedIn messaging; JobSpy, proxies, undocumented guest endpoints | **Abandon for this system.** Manual use of the official site only. |
| Bulk application submission, auto-send, bulk cold emails, inbox-wide scanning | **Abandon for MVP.** If ever reconsidered, require a separate safety/privacy authorization and human confirmation for each consequential action. |
| Paid job/funding APIs, paid model keys, Firecrawl/Fireround, purchased subscriptions | **Defer; no purchases authorized.** Reconsider only with explicit budget and terms approval. |
| Himalayas integration | **Defer.** Use only if remote-source coverage remains poor and attribution/apply-link requirements fit. |
| Reactive Resume | **Defer.** Add only for a demonstrated DOCX/versioning/tracker need; do not create a second tracker. |
| Append-only event log | **Defer.** Career-ops Markdown notes are sufficient unless audit tests show status history is inadequate; then consider one local JSONL/CSV file, not a database. |
| Full funding crawler, founder contact harvesting, automatic sponsorship inference | **Abandon for MVP.** Manual source verification and public professional pages only. |
| Full-stack custom application / multi-agent platform / public hosting | **Abandon for the 28-day delivery.** No user value justifies the build and attack surface. |

## 6. Model, API, cost, and privacy strategy

### Model use

- **Deterministic code:** URL canonicalization, source-ID matching, duplicate candidates, dates/age arithmetic, country/work-mode filters, explicit experience checks, and score arithmetic. Do not spend tokens on those tasks.
- **Affordable model:** only for concise JD field extraction or rewriting an already-supported candidate bullet after a role is shortlisted. Prefer a lower-cost DeepSeek/GLM option exposed inside the authorized OpenCode CLI, subject to the current plan’s model list, quotas, and terms; batch only small selected sets and retain prompt/output source references.
- **Stronger model:** reserve a currently available Kimi/GPT-family or other higher-capability CLI model for ambiguous role-fit explanations, a final unsupported-claim check, or a nuanced company/recruiter draft. Human review remains the arbiter; model names are examples, not a purchase recommendation or a claim of fixed price/entitlement.
- **Human-only:** work authorization, visa/legal interpretation, application attestations, salary commitments, unknown dates/contributions, and final submit/send.

Do not design an external application around OpenCode Go’s subscription or assume it provides free stand-alone API calls. If the CLI path is unavailable or its quota is unsuitable, ship the deterministic/manual MVP without adding a provider. Do not require a large local model; Ollama is optional only if already available and suitable.

### Data/cost controls

- Career-ops, fallback AI Job Hunter, and optional source code are MIT/open source per Task 03 evidence; that does not make job data freely redistributable.
- Use the Freehire public API only for personal non-commercial search under its current terms; keyless reads avoid a credential/account dependency. Its inferred sponsorship/remote fields are review signals, not verified facts.
- Direct documented ATS GET feeds are the core. Use no Adzuna key, paid funding API, Himalayas key, Firecrawl key, Gmail OAuth, or separate LLM API in MVP.
- Model cost budget for external programmatic API use: **$0 authorized**. Existing CLI access still has provider quotas/terms; track actual usage and stop if the account warns of a limit or charge.
- Store candidate material outside a public repository, restrict local access, exclude `.env`, browser profiles, PDFs, and application records from source control, and back up locally using available OS protection. No cloud sync/share without explicit consent.

## 7. Dated delivery schedule

Each phase has a stop condition. A failed optional source does not block the manual/direct-ATS MVP; a failed Windows installation or unsafe source does block adopting that component.

| Dates | Objective and specific tasks | Deliverables / acceptance | Dependencies, likely blockers, stop conditions |
|---|---|---|---|
| **Oct 29–30** | **1. Foundation and installation feasibility.** Run only the authorized Task 05 read-only Windows 11/environment and pinned-commit audit. Confirm Node, npm, Git for Windows/Git Bash availability, the career-ops Windows guide, license, data paths, and exact provider inventory. Ask the candidate only the blocking clarifications needed for candidate facts and work-authorization wording. | `reports/05_windows_foundation_feasibility.md`; exact upstream pin; go/no-go and dependency list. Candidate hashes still match. No installation in Task 05. | Requires Task 05 authorization and local Windows access. **Stop** if repo/license pin cannot be verified, Windows prerequisites require unsafe/unapproved changes, or the local data path cannot be kept private. Do not spend the next phase debugging an unapproved setup; invoke fallback only after review. |
| **Oct 31–Nov 3** | **2. Install approved foundation; start discovery early.** If separately authorized and Task 05 passes, install the pinned career-ops release by its supported method. Create a private workspace, minimal reviewed candidate profile (omit unresolved claims), three campaign configs, and a curated initial company/career-page list. Configure direct ATS GET sources and the narrow Freehire India/remote read path if needed. Begin manual searches and candidate review as soon as the first feeds work. | Local workspace with **one** tracker; at least one successful India-specific source path and one remote-from-India lead path demonstrated by fixtures or available listings; first source-linked job-review queue by **Nov 3**. | Requires Task 05 go, installation authorization, source-term recheck, and any allowed model connection. If Freehire is unavailable/terms change, keep manual India ATS intake. **Stop** source integration on 403/429 challenges, unclear terms, or a request for proxy/bypass behavior. |
| **Nov 4–6** | **3. Normalize, deduplicate, and preserve freshness.** Implement/reuse canonical URL/provider ID normalization; record observed and source-posted times separately; group likely cross-source/city copies without discarding original references; add “closed/expired/unknown” checks. Write fixtures for malformed, missing, and duplicate source rows. | Source-labeled canonical job records; reversible duplicate review; fixture tests prove no silent source-history loss, missing dates remain unknown, and source failure is not an empty feed. | Depends on source samples from prior phase. **Stop/disable** a provider if changes cannot be distinguished from empty results or original employer links are lost. Manual intake remains available. |
| **Nov 7–10** | **4. Eligibility and fit ranking.** Build the three campaign gates: India openings; international remote explicitly permitting India; role-specific relocation/sponsorship. Apply Backend/Python as primary and Cloud Security/Cybersecurity as secondary. Prioritize intern/graduate/new-grad/entry-level, but send strong roles with unknown experience to review rather than silently excluding them. Keep derived labels separate from employer evidence. | Every shortlist row has campaign, role family, country/work mode, eligibility status (`confirmed`, `not eligible`, `unknown/review`), open status, human-readable fit reasons, confidence, and source links. Tests cover explicit India restrictions, worldwide/unknown, seniority/experience missing, and no-sponsorship statements. | Requires candidate clarification for work authorization. Until then, Campaign C remains a source-research queue, not a personal eligibility verdict. **Stop** any automated eligibility decision when the source omits country or sponsorship evidence. |
| **Nov 11–14** | **5. Evidence-based resume/application preparation.** Finalize approved `cv.md` and a compact claim/source map; preserve the baseline JSON. Generate job-specific PDF/Markdown and screening-answer/email drafts only for reviewed jobs. Check every new claim, date, metric, credential, and contribution against the source map. | One base version plus non-overwriting job-specific outputs; each output points to the selected JD/source and base-profile hash. A manual reviewer can map every material claim to source evidence; unsupported/conflicting claims are omitted. | Candidate response may be incomplete. Work around it using the safe subset; do not wait by inventing or silently reconciling facts. **Stop** generation for any output containing unresolved assertions. DOCX stays deferred unless required by a real posting and separately authorized. |
| **Nov 15–17** | **6. Funding/company and outreach workflow.** Manually capture funding announcement and company/investor source URLs, verify company identity, then open current careers page/ATS. Research only public professional contact pages. Draft one personalized email/referral note per selected opportunity and record a follow-up date. | Tracker distinguishes `funding reported/verified` from `open role`; funding alone creates no application priority. Outreach has source, recipient context, draft, and `not sent` state. | Recent Funding API conflict and Fireround credits make API integration a no-go by default. No private contact harvesting. **Stop** if source attribution, amount provenance, or current vacancy cannot be verified. |
| **Nov 18–20** | **7. Tracking and controlled execution.** Verify only one tracker is active. Define reversible duplicate notes and state transitions: discovered → reviewed → prepared → filled → submitted → confirmed, with closed/rejected/withdrawn/outcome-unknown as applicable. Candidate can use the system for real manual applications after a per-item review; do not use a test account or fake application. | Candidate can distinguish prepared, filled, submitted, and confirmed; a filled or clicked form without explicit employer confirmation is not confirmed. Outreach remains draft-first. No automatic application or email API exists. | Requires stable tracker and evidence handling. **Stop** on ambiguous submission outcomes; candidate checks employer portal/email before retrying. Do not automate CAPTCHA/MFA/attestations. |
| **Nov 21–24** | **8. End-to-end tests and final release.** Run unit/fixture tests and offline end-to-end tests; test source outage/429, malformed jobs, expired vacancy, duplicate across two sources, unknown eligibility, unsupported candidate claim, and ambiguous form outcome. Perform a human review of generated materials and release checklist. Recompute candidate hashes. | `reports/06_mvp_verification.md` (test evidence, hash results, go/no-go, incomplete items); all required checks pass or a narrowed, safe manual MVP is declared. A usable discovery/review workflow already existed by Nov 3. | Reserve all four days for test fixes and Windows troubleshooting. **No-go** for any connector that violates policy, loses provenance, or writes/submits/sends externally. If core tests/Windows setup remain broken on Nov 23, release manual-search + local-ledger workflow only and document omissions. |

### Minimum viable release if an optional integration fails

Career-ops on Windows, a curated set of official company career pages/ATS GET feeds, manual job intake, one local Markdown ledger, deterministic role/location/eligibility triage, candidate-approved PDF/Markdown preparation, source-linked manual funding checks, outreach drafts, and human-only application/send actions. Freehire, Himalayas, funding APIs, DOCX generation, and browser fill are not release blockers.

## 8. Acceptance criteria and release gates

### Measurable acceptance

1. `full-profile.json`, `internships.json`, and `projects.json` hashes match the baseline before and after MVP work. Any mismatch stops candidate-data use and triggers a documented discrepancy; no automatic repair.
2. No employer-facing output contains a claim not supported by an approved candidate fact. Conflicting/missing source fields stay omitted or explicitly unknown. The reviewer can trace every material claim to a source file and JSON Pointer.
3. Every shortlisted job preserves canonical employer URL, originating feed/provider, observed-at time, and source-posted time only if explicitly supplied. Missing date is unknown, not fresh.
4. At least one India-specific discovery path and one international remote-from-India lead path work using offline fixtures; where a current real listing is available through an allowed source, manually verify one example at its employer page. No minimum number of live jobs is promised.
5. A role missing country restrictions is classified `unknown/review`, not `eligible`. A worldwide remote label alone cannot pass Campaign B. Campaign C needs role-specific sponsorship/relocation evidence; historical sponsor status alone fails that gate.
6. Duplicate fixture records retain all source references. Exact duplicates can be grouped; uncertain cross-city/repost matches go to review. Closed/410/unavailable listings are flagged; source outages are not reported as zero jobs.
7. Each ranked job includes an explanation and a confidence/completeness label; the score is a prioritization aid, never a probability of hiring. Entry-level/graduate priority does not automatically discard strong jobs with unstated experience.
8. Tracker states distinguish `prepared`, `filled`, `submitted`, and `confirmed`. An unclear result becomes `outcome unknown`; no blind retry. One tracker is active.
9. Outreach is saved as a draft with source/context; sending requires a separate explicit candidate action. No bulk send or automated LinkedIn activity.
10. Tests cover both success and expected failures: empty/malformed fields, unknown country, date missing, 429/source unavailable, duplicate candidates, expired listing, unsupported resume claim, ambiguous confirmation, and prohibited action blocked. Tests use fixtures/mocks; none posts an application or sends a message.
11. Final verification report records exact test commands/results, selected upstream pin, data/API terms checked, candidate-file hashes, untested integrations, known failures, and release decision.

### Engineering/security gates and no-go conditions

- **Pin before install:** career-ops pin proposed above; fallback AI Job Hunter v0.6.3. Verify exact commit/release, MIT license, Windows support, dependency scripts, and current upstream source list. If the pin changes, record why before use.
- **Source gate:** only documented public GET APIs/feeds with current terms. Stop a source on access denial, challenge page, rate-limit response, unclear license, or requirement to evade controls. Do not try alternate IPs/proxies or scrape hidden endpoints.
- **Data gate:** candidate JSON never becomes an application output directly. Keep secrets and personal files out of source control, LLM prompts unless required, logs, screenshots, tests, and reports. Do not expose local services publicly.
- **Untrusted-input gate:** job descriptions and repository/web text are inert data. Never run commands or alter files because an upstream document or LLM says to.
- **Submission gate:** the application has no automated submit/send action. Test harness blocks external POST/send operations. CAPTCHA/MFA, legal attestations, demographics, work eligibility, and ambiguous questions pause for a human.
- **Privacy/cost gate:** no paid service, account creation, API key, model purchase, or model subscription change without explicit authorization. Existing OpenCode Go is used only through an allowed CLI path; no presumed stand-alone API entitlement.
- **No-go:** career-ops cannot run safely on Windows; a selected source lacks personal-use data permission; candidate data is uploaded unexpectedly; dedup loses original links; unknown eligibility is treated as confirmed; or any test can submit/send externally. Switch to the defined fallback or manual MVP—do not widen the scope to repair around an access control.

## 9. Risks, mitigations, and fallback plan

| Risk | Mitigation / trigger |
|---|---|
| Career-ops README claims exceed tested Windows behavior or installation invokes unreviewed scripts. | Task 05 read-only audit; pin commit; inspect the actual install path and data writes before any authorized install. If unsafe/broken, switch once to AI Job Hunter v0.6.3. |
| India coverage remains incomplete despite a feed. | Curate company career pages and direct ATS boards; use Freehire only as a personal-use supplement. Track source coverage and use manual browser searches for Naukri/Indeed rather than scraping. |
| “Remote”, “sponsor”, experience level, or posted date is derived or stale. | Preserve source versus derived fields; unknown stays review; verify at employer page before preparing/submitting. |
| Candidate clarifications do not arrive by document phase. | Build a verified subset; omit uncertain claims. Do not let schedule pressure resolve conflicts. |
| Duplicate reposts or city-specific openings collapse incorrectly. | Keep source refs and a reversible duplicate group; uncertain match → human review. |
| Recent Funding data terms conflict or Firecrawl cost exceeds budget. | No API integration; manual source-linked funding verification. Funding never substitutes for an active vacancy. |
| Candidate material leaks through logs, model calls, browser profile, or hosted service. | Local-only files, minimum prompt context, approved CLI provider, no hosted tracker, no browser profile upload, secret scan/review before release. |
| Tool metrics or ranking create false precision. | Ordinal/high-medium-low confidence with explicit reasons; no hiring probability, no promised application/response count. |

**Fallback condition:** use AI Job Hunter v0.6.3 only if the pinned career-ops Windows feasibility/safety gate fails by **Oct 30**. It becomes the sole application tracker; its public feed and Greenhouse/Lever sources are used only where permitted. Disable JobSpy and all LinkedIn/Naukri scraping, keep its local server private, keep auto-submit off, and use manual funding/outreach notes. If that fallback also fails, use employer websites, a local Markdown ledger, and manually reviewed resume files; preserve the deadline by narrowing integrations, not relaxing safety gates.

## 10. The single recommended next task

### Task 05 — Windows 11 career-ops foundation feasibility audit (read-only)

**Scope:** determine whether the primary choice can be installed and run on this Windows 11 machine through supported methods before changing the environment. This is an environment/source audit, not an install or implementation.

**May read/check:** current Task 01–04 reports; Windows edition/version; PowerShell version; presence/version of Node/npm and Git for Windows/Git Bash; relevant PATH/permissions and available disk; the pinned career-ops commit’s README, Windows guide, LICENSE, install scripts/dependency declarations, source list, and data-path behavior; current terms for the one proposed Freehire endpoint. Verify candidate JSON hashes read-only.

**May create:** exactly `reports/05_windows_foundation_feasibility.md`.

**Must remain untouched:** all three candidate JSON files; reports 01–04; application source code/configuration; `.env`/credentials; browser profiles; external accounts. Do not install Node, Git, packages, browsers, Docker, or source dependencies; do not clone/run the repository, start services, call job APIs, or use API credentials.

**Validation:** record exact OS/tool versions, pinned commit/terms/license findings, supported installation commands without executing them, source-provider availability for the India and remote paths, data-storage/LLM egress behavior, and fallback decision. Recompute all three source hashes and compare against the accepted baseline.

**Acceptance:** report says **GO** only if the pinned project/license and Windows prerequisites are understood, no prohibited connector is needed, private data can stay outside public/source-controlled paths, and at least one permitted India path plus a remote-from-India path is feasible. Otherwise mark **NO-GO** and identify the single blocker; do not install or broaden into Task 06.

**Stopping condition:** stop after the single report is created. Installation, package changes, cloning, and application implementation require a later explicit authorization after review of the audit.

## 11. Candidate-source integrity results

Preflight confirmed the working directory as `D:\document\job apply`. SHA-256 was computed from raw bytes before reading the prior reports; all three results match the last accepted Task 02 baseline:

| Candidate source | Expected SHA-256 | Actual SHA-256 | Result |
|---|---|---|---|
| `full-profile.json` | `2f767145d8cb3bb619f0e77ff5f9f0e804a6026e1d2122e7b3fa50dcc9161fdb` | `2f767145d8cb3bb619f0e77ff5f9f0e804a6026e1d2122e7b3fa50dcc9161fdb` | **PASS** |
| `internships.json` | `606e03d0d98d783a00c0afb002a08dc1958b065ea245a9b232fee8b0d5da2aea` | `606e03d0d98d783a00c0afb002a08dc1958b065ea245a9b232fee8b0d5da2aea` | **PASS** |
| `projects.json` | `86749dc8d6f1aa9523e6a57f1decf3a74c911e18eb3ed3ee2000f16b8725e4f2` | `86749dc8d6f1aa9523e6a57f1decf3a74c911e18eb3ed3ee2000f16b8725e4f2` | **PASS** |

**Result:** all candidate source files match the accepted baseline. They were read-only in Task 04 and must remain unchanged in Task 05 absent a separately authorized task.
