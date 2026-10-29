# Task 15 — Source Adapter and Discovery Configuration Audit

**Date:** 2026-10-29  
**Outcome: PASS WITH LIMITATIONS**  
**Repository:** `D:\document\job apply\tools\career-ops`  
**Private data root:** `%USERPROFILE%\career-ops-data` (expanded path intentionally omitted)

## 1. Executive summary

The pinned checkout has a real provider framework: `scan.mjs` loads **124 non-helper `providers/*.mjs` modules** and routes configured `tracked_companies` and `job_boards` entries to them. Greenhouse, Lever, Ashby, and SmartRecruiters all have working code paths for public, unauthenticated job-posting APIs. Provider results become a normalized `Job[]`; the normal scanner applies filters and can append leads to `data/pipeline.md` and dedup/health history. It does **not** write application rows to `data/applications.md`.

The current `%USERPROFILE%\career-ops-data\portals.yml` is inactive: `tracked_companies`, `job_boards`, and `search_queries` are empty; `skip_tiers` is empty. No provider is enabled. No discovery was run.

**Best first setup:** after the user approves a short list of actual employers and their career URLs, add only a few employer-owned, public ATS boards under `tracked_companies`. Keep web-search queries and broad aggregators off. Do not infer eligibility from “Remote”; international remote roles require a manual check that the employer permits work from India. The repository has no reliable positive filter for that fact.

Limitations: this is local source evidence only. No vendor documentation, terms, robots files, board ownership, URLs, or live job data were externally checked. The 124 adapters are broad and differ in endpoint stability, pagination, and provenance; their individual catalog entries must be reviewed before any additional source is enabled.

## 2. Repository integrity and privacy preconditions

| Check | Result |
|---|---|
| Working directory | `D:\document\job apply` |
| HEAD | `a6d7526c05861b1df9b70c011f8b1a84c44252ac` |
| Origin | `https://github.com/career-ops-hq/career-ops.git` |
| Tracked/full Git status before inspection | Clean |
| `package.json` SHA-256 | `10ca52815f65e4ba14ea4025828f2f9b212d1b684a2ab495a173d3a461df1fac` |
| `package-lock.json` SHA-256 | `cb14c2cd24a3676514af291914db76ace250a8bbf2d6708006f02300028c22cc` |
| Private root | Present; expanded profile path not logged |
| `portals.yml` | Present at `%USERPROFILE%\career-ops-data\portals.yml` |
| Private-root ACL | Rechecked against Task 13 policy; PASS; no usernames or SIDs recorded |

Candidate JSON contents were not read. No source/config/manifest/lockfile, profile, portals file, or tracker was changed.

## 3. How discovery actually routes

### Core provider registry

Evidence: `providers/_registry.mjs`, `providers/README.md`, `providers/_types.js`, and `scan.mjs`.

- `loadProviders()` imports each `providers/*.mjs` file that **does not** start with `_`, in alphabetical order. It accepts a default export with a unique `id` and `fetch(entry, ctx)`; `detect(entry)` is optional.
- Routing precedence is: explicit `provider:` ID; then `local-parser` when `parser.command` and `parser.script` are configured; then the first provider whose `detect()` matches.
- `tracked_companies` is one entry per employer. `job_boards` is one entry per multi-employer board/feed. They share the entry schema and dedup namespace.
- Provider output is the normalized contract in `_types.js`: required `title`, absolute posting `url`, `company`, and `location`; optional `description`, `postedAt`, salary, IDs, language and trust metadata. The scanner adds source attribution (`<provider-id>-api`, or `local-parser`) before pipeline/history serialization.
- A module existing is not proof its source is useful or current. `echojobs` and `torre` remain loadable compatibility stubs, but their fetch functions report the source retired/unavailable.

### Requested ATS adapters

All four below use HTTPS, public/no-auth list endpoints in the code, pass redirects as errors, and return normalized jobs. The repository describes these as public ATS APIs. No external vendor documentation was fetched, so “documented” below means the repository has an explicit endpoint contract; current vendor documentation and terms still need a later source-approval check.

| Source | Implementation and minimum entry data | Endpoint / method / paging | Output and documentation status |
|---|---|---|---|
| **Greenhouse** | `providers/greenhouse.mjs`: `resolveApiUrl()`, `detect()`, `fetch()`. Generic enabled-entry `name` is required. Set `careers_url` to a real Greenhouse board URL, or pin an allowlisted `api:` URL. Auto-detection understands `job-boards[.eu].greenhouse.io`, legacy `boards[.eu].greenhouse.io`, and the embed `?for=` form. | GET to `https://boards-api.greenhouse.io/v1/boards/{slug}/jobs?content=true`. One complete board-list response; no pagination loop. `content=true` returns descriptions in the list. A conditional second GET to `/v1/boards/{slug}/offices` occurs only when the list reports work-model-only locations. Redirects are rejected. | `Job[]` includes title, `absolute_url`, configured employer name, location, description, publish date and available IDs. Repository labels the Board API public. The endpoint is a public ATS surface, not a login/session scraper. |
| **Lever** | `providers/lever.mjs`: `resolveApiUrl()`, `detect()`, `fetch()`. Enabled-entry `name` plus `careers_url` on `jobs.lever.co` / `jobs.eu.lever.co`, or an allowlisted API URL. `api:` takes precedence. | GET `https://api[.eu].lever.co/v0/postings/{slug}`. One board-wide array; no page loop. The list includes descriptions. EU boards use the EU API host. Redirects are rejected. | `Job[]` includes title, hosted posting URL, configured company, all exposed locations, description, date and posting ID. Repository labels this the public postings endpoint. |
| **Ashby** | `providers/ashby.mjs`: `resolveApiUrl()`, `detect()`, `fetch()`. Enabled-entry `name` plus a `jobs.ashbyhq.com/{slug}` careers URL or allowlisted `api:` URL. Board slugs are case-sensitive. | Default: GET `https://api.ashbyhq.com/posting-api/job-board/{slug}?includeCompensation=true`, one board-wide response, no pagination. Retries transient/rate-limit failures with bounded delay. Optional `ashby: { embed: true }` instead GETs the public embed page `https://jobs.ashbyhq.com/{slug}?embed=js`; it can use `ashby.board` when `careers_url` is branded. That HTML/hydration fallback is opt-in and more markup-dependent. | API path emits title, job URL, company, structured/secondary location plus work model, description, date and parsed compensation. Embed path emits a smaller result (no full description/date and no normalized salary). Repository describes the posting API and its documented embed surface as public. |
| **SmartRecruiters** | `providers/smartrecruiters.mjs`: `resolveSlug()`, `buildPostingsUrl()`, `fetch()`. Enabled-entry `name` and a `careers.smartrecruiters.com/{slug}` or `jobs.smartrecruiters.com/{slug}` URL, or `api:` containing such a SmartRecruiters careers URL. `provider: smartrecruiters` can force routing for a branded careers page, but the slug still must be supplied via the `api:` value. | GET `https://api.smartrecruiters.com/v1/companies/{slug}/postings?limit=100&offset={n}&status=PUBLIC`; offset pages of 100, capped at 50 pages. The health probe can cap this at one page. Optional per-entry `smartrecruiters: { fetchDetails: true, detailLimit: N }` adds bounded detail GETs (default 25, max 100), skipped during probes. | List results yield title, public posting URL, configured company, location/remote label, IDs and language; description is absent unless details are explicitly requested. Repository describes the postings API as public. |

“Public/zero-auth” does not prove that an employer’s slug belongs to that employer or that a role allows work from India. A human must verify both before enabling a board.

### Manually curated employers and links

- A manually approved employer belongs in `tracked_companies`, with a display `name` and a real `careers_url`; `api:` and `provider:` are optional routing aids. The generic validator permits either URL field to be absent, but the chosen provider may then fail to resolve the entry. `enabled` defaults on; only boolean `false` disables an entry.
- A single job URL is **not** a `tracked_companies` feed. A job link supplied by a person belongs in the `data/pipeline.md` URL inbox for a separately authorized manual pipeline workflow. That workflow may fetch the posting and use a model; it is not a local-only discovery action.
- `local-parser` is a custom escape hatch, not an ATS API. `providers/local-parser.mjs` runs a configured in-repository script via `execFile`; its output is parsed as JSON and normalized. Although the module restricts the command/script path, it still executes code and may itself access the network. Do not use it for the first setup.
- `search_queries` are not feeds consumed by `scan.mjs`. The agent-facing `modes/scan.md` uses enabled query entries for WebSearch; this is broad, potentially stale external discovery. The core CLI only emits a web-search handoff for an unmatched entry explicitly marked `scan_method: websearch`. Keep both paths off.

## 4. Other discovery sources present in this commit

### Core provider catalog

The core directory contains 124 non-helper modules; each returns the same normalized `Job[]` shape. `docs/SUPPORTED_JOB_BOARDS.md` is the repository’s per-source catalog: each row identifies API/parser/RSS/Atom/XML type and usually gives endpoint construction, provider key, page cap and notable requirements. `providers/ADDING_A_PROVIDER.md` says core providers are intended for public, no-auth sources and requires source eligibility/robots review. That convention is not a guarantee that every endpoint is an official, stable vendor API.

The loaded module inventory (filename stem; see the corresponding catalog row for that adapter’s exact endpoint/config/page behavior):

```text
4dayweek, a16z-speedrun-talent, adp-workforcenow, agentic-jobs, alibaba, amazon,
apple, applitrack, arbeitnow, arbeitsagentur, ashby, atlassian, avature, bamboohr,
beesite, breezy, builtin, bwi, careerviet, clearcompany, collage, comeet,
computrabajo, consider, cryptocurrencyjobs, csod, dassault, deutschebahn, dover,
echojobs, eightfold, eploy, feishu-jobs, flowxtra, garena, gem, generalist-world,
getonbrd, getro, glints, google, greenhouse, gupy, hackernews, hecklerkoch,
higheredjobs, himalayas, hiringroom, homerun, ibm, icims, inrecruiting, itviec,
jazzhr, jibeapply, jobbankca, jobicy, jobscz, jobspresso, jobstreet, jobvite,
join, joinup, justjoin, kalibrr, landingjobs, larajobs, lever, local-parser,
magneto, manfred, meituan, mokahr, mycareersfuture, neogov, nodesk, nofluffjobs,
notion, occ, oraclecloud, pageup, paylocity, peoplesoft, personio, phenom,
pinpoint, platsbanken, pracecz, prevueaps, pythonorg, radancy, recruitee,
redrover, remoteok, remotive, remotli, rheinmetall, rippling, schoolspring,
senjob, smartrecruiters, softgarden, solidjobs, startup-jobs, successfactors,
taleo, teamtailor, techinasia, telegram-channel, tencent, thehub, themuse, tkms,
torre, ultipro, vdab, weworkremotely, workable, workday, workingnomads,
wp-job-manager, wttj, yourator, zoho-recruit
```

This catalog mixes employer ATSes, national/regional public job services, board-wide aggregators, public RSS/Atom feeds, and HTML parsers. Many adapters use ordinary GETs; some public sources use POST/GraphQL or site-owned endpoints. All produce `Job[]`. Some consume endpoint-specific fields such as `max_pages`, keywords, tenant/board identifiers, locale or provider blocks. Do not copy a provider-specific example without reading that provider’s exact module and catalog row.

Important exceptions and endpoint caveats from the catalog/source:

- `echojobs` and `torre` are retired stubs, not usable sources; their IDs remain for old-config compatibility and `fetch()` fails explicitly.
- `notion` uses the site’s **undocumented** `/api/v3/loadPageChunk` and `/api/v3/queryCollection` POST endpoints. It is not an official-token API and is not recommended for the first setup.
- Some providers parse server-rendered pages or use site-internal GraphQL/JSON endpoints. “Public” does not mean vendor-documented, stable, or approved for this candidate; check the specific catalog row and source-indexing record before enabling one.
- Several worldwide-remote feeds exist (e.g. RemoteOK, Remotive, Jobicy, Himalayas, Working Nomads, We Work Remotely). A `Remote` label in those feeds is not evidence that India is an eligible hiring country.

### Other discovery entrypoints

| Entry point | What source it uses | Material behavior / reason not proposed now |
|---|---|---|
| `scan-ats-full.mjs` | Reverse sweep of public company directories for Greenhouse, Lever, Ashby, Workday, iCIMS, BambooHR, Paylocity, and Join; source code names `SOURCES` and `DATASET_BASE`. | Network-heavy broad discovery, fetches a mutable external company dataset (24-hour local cache), checkpoints/caches board data, can write pipeline/history, and has optional Playwright liveness. Not a small approved-employer list. |
| `scan-interamt.mjs` | Interamt search pages via Playwright/Wicket; `interamt_searches` or built-in German IT keywords. | Launches Chromium, searches externally and can write pipeline/history; unrelated to India-first ATS APIs. |
| `scan-dayforce.mjs` | Dayforce tenant boards via Playwright, a CSRF-protected API and per-entry `dayforce_boards` config. | Launches Chromium and uses a fresh session. The source explicitly stops at an interactive challenge; it does not solve/retry a CAPTCHA. Not proposed. |
| `scan-hn.mjs` | Hacker News “Who is hiring?” via the `hackernews` provider. | Separate script, not the plain core scanner. If `GEMINI_API_KEY` is present it calls Gemini to extract fields; without the key it uses keyword matching. It writes pipeline/history. Excluded. |
| Agent-facing `modes/scan.md` | `search_queries` via external WebSearch; may then use browser/API steps and liveness verification. | Queries can be stale and go to a search service. Not the local zero-token `scan.mjs` and not enabled in the current config. |
| `discover-ats.mjs` | Probes candidate company names against a bounded set of ATS hosts/providers. | Makes live HTTP probes. Preview mode writes nothing, but still contacts sources; `--write` edits `portals.yml`. Not run. |
| `discover-new-companies.mjs` | Reads existing local scan history, subtracts tracked/blacklisted employers, and emits a candidate YAML list. | No network in this step; `--out` writes the named candidate-list file. It only prepares input for the networked `discover-ats.mjs`; neither command was run. |
| Optional provider plugins | `plugins/_engine.mjs` provider hooks, enabled only through `config/plugins.yml` plus required environment. | Designed for integrations that may use keys or external services; plugin execution is trusted code, not a sandbox. No `config/plugins.yml` exists in this workspace. |

### Unsupported or excluded sources/behaviors

- No direct provider adapter for LinkedIn, Indeed, or Naukri was found. `linkedin-join.mjs` is a local join against a user-provided connections export, not a job-listing scraper. The Telegram parser may extract a link appearing in a public post; it does not crawl LinkedIn/Indeed/Naukri itself. Do not use such links as a substitute for an approved employer posting.
- The catalog explicitly marks Glassdoor and Dice as unsupported where reference scraping requires `playwright-stealth`; TopDev/CareerLink are excluded for robots restrictions, and JobsGo/ViecOi for challenge barriers. The source policy says not to route around applicable robots restrictions or bot controls.
- There is no supported CAPTCHA solver, MFA bypass, stealth browser, or login-session scraping path recommended here. Dayforce and Interamt are browser-driven special cases; do not enable them for this MVP. Core `scan.mjs --verify` also has optional headed Playwright fallback flags; do not use those in a first discovery run.
- Unsupported/unverified third-party, private, login-gated or undocumented endpoints are not an acceptable reason to add a guessed `api:` URL. No external probing was done in this task.

## 5. Configuration contract and filters

### What is required versus optional

`validate-portals.mjs` and `scan.mjs` confirm the following:

| Field | Contract / behavior |
|---|---|
| `tracked_companies` | Optional array. Each enabled entry must have a non-empty `name`; it represents one employer. `careers_url`, `api`, and `provider` are structurally optional, but the selected adapter may require enough of them to derive a board. |
| `job_boards` | Optional array with the same entry contract; one entry represents a multi-employer board/feed. |
| `enabled` | Optional; defaults enabled. Only boolean `false` disables. A string such as `"false"` is still active and is an error in the validator. |
| `careers_url`, `api` | Optional generic URL strings; validator checks only `http(s)` syntax. Provider code may require a specific HTTPS host and slug and apply its own host/redirect guard. |
| `provider` | Optional explicit provider ID; bypasses detection. Validator checks the ID exists, but does not prove it belongs to the named employer. |
| `parser` | Optional local-parser object; validator checks command/script/args/limits, not what the script does. Running it executes local code. |
| `search_queries` | Optional array; validator checks only that it is an array, not each query’s shape or its safety. Agent-facing scan modes send enabled queries to WebSearch. `scan.mjs` does not use this field as provider targets. |
| `title_filter` | Optional object with positive/negative keyword arrays. Positive entries are ORed; negative entries veto. Bare terms are mostly case-insensitive substrings; `word:`/`stem:` and ` + ` have special semantics. A malformed bare-string keyword is a warning because it may be ignored. |
| `skip_tiers` | Optional global array; per-entry `skip_tiers` replaces it. Empty means no seniority-tier exclusion. This key is not comprehensively schema-checked by `validate-portals.mjs`; `scan.mjs` validates per-entry values and treats the global non-array as empty. |
| `location_filter`, `country_eligibility_filter`, `content_filter`, `visa_filter` | Optional. Their presence does not make their evidence complete; see geography notes below. |

The current file’s `tracked_companies: []`, `job_boards: []`, and `search_queries: []` are valid empty lists. `scan.mjs` converts absent lists to `[]`, creates zero targets, and therefore makes no provider fetch. `tests/scan-json-receipt.test.mjs` has a local fixture asserting zero targets/results for empty lists. **But an actual non-dry-run `scan.mjs` is not a no-op:** the main flow can still append a scan-run record (and other run metadata) even with no offers. Do not run it merely to test empty configuration.

The shipped example is not an approved profile: `templates/portals.example.yml` contains a large, broad catalog and even has an enabled Wellfound WebSearch example. Do not copy it wholesale into the private configuration.

### Role and seniority filtering

- The existing configured `title_filter.positive` contains Backend/Python/Software Engineer, internship, graduate, trainee, junior, entry-level, Cloud Security and Cybersecurity keywords. Its negative list and `skip_tiers` are empty. This audit recommends no edits.
- Title matches are exact keyword semantics (with optional `word:` and `stem:` boundaries), not semantic role classification. A matching title reaches downstream filters; a keyword miss is silently filtered.
- `classify-tier.mjs` labels `intern`, `entry`, `mid`, or `senior`; plain/unrecognized titles default to `mid`. `skip_tiers: [mid]` would therefore remove ordinary titles such as “Backend Engineer”, not just explicit mid-level postings. Keep it empty. `tests/scan-skip-tiers-per-entry.test.mjs` covers the override/empty behavior.
- A plain “Graduate Engineer” is not necessarily classified as intern; the classifier’s graduate special case requires “program” or “scheme”. With `skip_tiers: []`, that default does not exclude it.

### Geography and work eligibility

- `location_filter` operates on provider-supplied location strings plus supported URL hints. With no filter, all locations pass. `allow` is an allow-list; `block` rejects; `always_allow` can rescue multi-location entries; `block_hard` cannot be overridden; missing location passes unless `strict: true` applies to an active restriction.
- `remote_title_fallback` defaults true. When a restrictive allow-list fails but the title explicitly says Remote, it can pass. This is a discovery rescue, **not** country eligibility. Turning it off can also drop valid India-eligible remote listings that expose only a generic Remote location.
- Provider work-model labels differ: some expose country/cities and Remote/Hybrid separately; some provide only “Remote”, an office/HQ, or no location. A generic Remote label does not mean work from India is permitted.
- `country_eligibility_filter` is an optional description-text heuristic: exclusionary phrases can reject; inclusive phrases or the candidate-country string anywhere in the description can make it pass; missing descriptions pass. It is not a structured employer country allow-list and is not proof of India eligibility.
- `visa_filter` is about sponsorship wording, not general permission to work from India. Neither it nor a missing eligibility statement proves eligibility.
- Consequently, the code can help narrow explicit India/city locations, but **international remote eligibility and ambiguous/missing locations require manual review against the employer’s own current posting**. Keep an explicit distinction between India-based roles and international remote roles; do not mark “Remote” as “India eligible.”

### Deduplication, attribution, and persisted data

- Source identity is carried as a provider/source label. Scan history records `portal` (the source), URL, first-seen date, title, company, status, location, posting date and IDs. The scanner deduplicates by normalized URL/listing identity and company+role, with optional location/language rules; aggregators primarily deduplicate by URL.
- Provider descriptions may be fetched/received in memory for content/eligibility filters and cross-listing detection. `formatPipelineOffer()` writes URL, company, title, location, compensation/date and optional trust/note fields—not the full description. `formatScanHistoryRow()` stores a short JD fingerprint, not the description body.
- Persistent state can include `data/pipeline.md`, `data/scan-history.tsv`, `data/portal-health.tsv`, `data/scan-runs.tsv`, and `data/cache/provider-ids/*.json`. The provider-ID cache stores lookup identifiers, not job descriptions. `scan-ats-full.mjs` also caches the external company directory (24 hours) and checkpoints large runs. Some provider-specific `fetchDetails` options add bounded per-posting detail requests; leave them unset initially.
- The core scanner reads the application tracker for deduplication but the normal scan path does not write `data/applications.md`. It writes a pending pipeline inbox; downstream pipeline/application preparation is a separate workflow and is not triggered automatically by a scan.

## 6. Side effects and validation boundary

| Component | Static side-effect findings |
|---|---|
| `scan.mjs` ordinary run | Loads provider modules, reads configuration/tracker/filters and sends HTTP requests through provider fetches. Can read public job descriptions, apply filters, and persist pipeline/history/health/run/cache files. Does not call an AI model in its normal provider path and does not write the application tracker. `--dry-run` still fetches sources; it only suppresses writes. |
| `scan.mjs --verify` | Adds liveness requests for posting URLs and can use Playwright fallback when API checks are inconclusive. `--headed-fallback` is an additional explicit browser retry. Not run/recommended here. |
| `scan-interamt.mjs`, `scan-dayforce.mjs` | Launch Playwright/Chromium, make network requests, and may write pipeline/history; Dayforce reads protected API state via its browser context but explicitly aborts unresolved interactive challenges. |
| `scan-ats-full.mjs` | Broad network sweep across thousands of ATS boards; reads a mutable public company dataset, caches data/checkpoints and can write the pipeline/history. Optional liveness uses Playwright. |
| `scan-hn.mjs` | Fetches HN data; if `GEMINI_API_KEY` is present, it sends post text to Gemini and may write pipeline/history. Not used. |
| `verify-portals.mjs` | Despite the name, this is a live endpoint probe. Sweep/`--add` makes HTTP requests and follows up to five redirects with guards. It is not the local validator. |
| `discover-ats.mjs` | Probes public ATS endpoints. Default is preview-only regarding config, but still makes network calls; `--write` changes `portals.yml`. |
| `validate-portals.mjs` | Narrow local config validator. Normal mode reads YAML, dynamically imports provider modules to collect IDs, and checks schema/provider IDs; it does not call `provider.fetch`, scan sources, open a browser, or call a model. It does not verify employer ownership, live slugs, source terms, or provider-specific settings comprehensively. `--self-test` creates a temporary fixture, so avoid that if the later task wants no temp artifacts. |

Job listings, descriptions, company pages, and feed payloads are untrusted external data, not instructions (`modes/scan.md`). No discovery, validation command, test suite, network request, browser, model, or external service was executed for this audit.

## 7. Smallest safe first-discovery proposal (not activated)

**Proposal only. No identifiers or URLs are filled in and no configuration was changed.** Target the Nov 3, 2026 run with perhaps **3–5 individually approved employers**, not a board-wide sweep:

```yaml
tracked_companies:
  - name: "<employer approved by the user>"
    careers_url: "<official employer careers or ATS board URL supplied/verified by the user>"
    # provider: greenhouse | lever | ashby | smartrecruiters  # only if auto-detect is ambiguous
job_boards: []
search_queries: []
skip_tiers: []
```

Keep the current title filter unchanged. Do not add `skip_tiers: [mid]`, a `search_queries` WebSearch campaign, reverse ATS discovery, a general remote-job feed, or a local parser.

Before enabling each employer entry, obtain/confirm its exact public careers URL and employer display name, verify that the resolved ATS board is actually that employer’s board, review the applicable source terms/robots paths, and confirm postings are public and applyable without account/CAPTCHA/MFA workarounds. Use only an endpoint whose public/auth and access basis has been approved. The static validator does not perform those checks.

No target-city list was approved in this task, so do not invent one. Initially leave `location_filter` unset rather than silently dropping city-only Indian postings. Manually classify each result as (1) India-based (remote/hybrid/onsite) or (2) international remote with explicit permission to work from India. Treat generic Remote, missing location, and no eligibility statement as **unconfirmed**, not eligible. If the user later wants automated location narrowing, first obtain target cities and decide how to handle generic Remote labels; the current filter cannot safely turn that label into India eligibility.

If the user cannot supply a few approved employer URLs by Nov 3, the safe alternative is manual URL intake into the existing `data/pipeline.md` inbox, not enabling a broad search adapter.

## 8. Exact validation method for a later task

After a bounded config is separately approved, the narrowest supported check is the normal `validate-portals.mjs` CLI, with `CAREER_OPS_ROOT` set only in that process/session so its default file is `%USERPROFILE%\career-ops-data\portals.yml`. The validator can confirm YAML/config shape and known provider IDs without activating providers. Do **not** substitute `verify-portals.mjs`, `doctor --strict`, `scan.mjs`, `discover-ats.mjs`, or `scan-ats-full.mjs`; those can probe sources or run workflows. Do not pass `--self-test` when avoiding temporary files.

The validator is structural, not a live-source or eligibility test. A later source-approval task must separately check employer-to-board identity and access policy; a later runtime task may run only the specifically authorized discovery procedure.

## 9. Final integrity and unresolved uncertainties

- Final HEAD/origin remain the pinned values in §2; tracked/full Git status remained clean during inspection.
- Manifest hashes and private-root ACL remained verified; all four Task 13 scaffold files remain in the private root.
- No application configuration, source code, manifests, lockfile, profile, tracker or candidate JSON was changed/read for content. This report is the only authorized new artifact.
- No external service, network, browser, model, scanner, verifier, validator, or discovery command was invoked.
- No source-specific employer IDs, board tokens, tenants, or URLs were guessed.
- Unresolved: approved employer list; each employer’s current ATS ownership/slug; current source terms/robots status; target cities; exact India work eligibility for international remote roles; and independent current vendor documentation for endpoint stability. These require a later, explicitly authorized, source-by-source review.
