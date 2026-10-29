# Task 16 — Employer Source Verification and Approval Dossier

**Date:** 2026-10-29  
**Outcome: PASS WITH LIMITATIONS**  
**Scope:** five named employers; public pages, bounded excerpts, official ATS documentation and robots files only. No source was enabled.

## 1. Summary

One source is a reasonable **SUITABLE CANDIDATE** for a later, user-approved first configuration: Plane Software’s Ashby board. Its board identity is supported by the board title/company metadata and a reference to the company’s `plane.so` domain; the official Ashby public-job-posting documentation describes the endpoint used by the pinned adapter. Two India-located backend postings were visible. Their experience and graduate eligibility were not established, and source-use approval remains pending.

Hevo Data and Drivetrain have promising India-located engineering/intern listings, but their Lever board pages use a public `/v0/postings` feed that the official Lever developer guide retrieved here does not document; specific AI-crawler robots rules also need a source-policy decision. They are **DEFER**red, not approved.

The supplied Sardine and SupplyHouse postings are **REJECT**ed for this initial early-career shortlist: the observed roles require 7+ and 3+ years respectively. This is a decision about the supplied/current opportunities, not a permanent ban on either employer.

**Nothing was added to `portals.yml`.** Even the suitable-candidate status is not permission to scan. Official source terms and the user’s activation approval remain required.

## 2. Initial integrity and privacy checks

| Check | Result |
|---|---|
| Working directory | `D:\document\job apply` |
| Repository HEAD | `a6d7526c05861b1df9b70c011f8b1a84c44252ac` |
| Origin | `https://github.com/career-ops-hq/career-ops.git` |
| Tracked/full Git status before research | Clean |
| `package.json` SHA-256 | `10ca52815f65e4ba14ea4025828f2f9b212d1b684a2ab495a173d3a461df1fac` |
| `package-lock.json` SHA-256 | `cb14c2cd24a3676514af291914db76ace250a8bbf2d6708006f02300028c22cc` |
| Private workspace and inactive `portals.yml` | Present |
| Private-root ACL | Rechecked against Task 13 policy; PASS; no usernames or SIDs recorded |
| Existing `portals.yml` SHA-256 | `398a369c7ab91ee9df75bd98b6acff0ad28a02bbf88907f5c91a553b02da0fd3` |

The current file still has empty `tracked_companies`, `job_boards`, and `search_queries`, with `skip_tiers: []`. Its title filters were not changed. Candidate JSON content was not read.

## 3. Research method and boundaries

- Research was performed on 2026-10-29 using GETs only: the five named company/ATS URLs, the directly linked official careers page, the ATS board roots needed to establish ownership, relevant robots files, and official Greenhouse/Lever/Ashby documentation.
- Redirects were checked with auto-follow disabled. The Greenhouse documentation URL redirected to `docs.greenhouse.io`, an expected official documentation host, which was then read. The SupplyHouse board root redirected to its official `/careers` URL; that URL returned 403 and was not followed elsewhere.
- HTML reads were bounded; direct posting pages were read only as capped byte prefixes/ranges, and only short eligibility snippets were retained. No full description was saved or reproduced, and no provider API/feed was called. The total response size was not captured for every page, so omitted requirements remain unconfirmed.
- No general-purpose crawler, WebSearch, provider adapter, application code, scanner, verifier, browser, model endpoint, login, CAPTCHA, MFA, rate-limit retry, or external account was used.
- A returned page status of 200 establishes public reachability for that request only; it does not establish permission under terms or a work-eligibility guarantee.

## 4. Comparison and decision table

Observed-role dates below are **observation dates** (2026-10-29), not publisher dates. The inspected list cards did not expose publication dates. `UNCONFIRMED` means the retrieved evidence did not establish the fact; it is not treated as a positive result.

| Employer | Official company / careers page and ATS board | Ownership evidence / adapter | Current relevant public example(s) | Location and early-career evidence | Endpoint, policy, breadth and decision |
|---|---|---|---|---|---|
| **Hevo Data** | Official site `https://hevodata.com/` (HTTP 200; title “Hevo Data”). Board `https://jobs.lever.co/hevodata?department=Engineering` (HTTP 200; page title “Hevo Data”). | Lever. The board footer links back to `http://www.hevodata.com`; brand and slug match the official site. The bounded official-homepage read did not show a reciprocal careers link. | [Associate Software Development Engineer (SDE) Intern](https://jobs.lever.co/hevodata/993a3ed3-3014-432f-b0f4-28d300d58d9a), listed on-site/full-time in Bangalore, India; [SDE I](https://jobs.lever.co/hevodata/6cbbe304-e065-4711-bf3e-756795d2bc2a), on-site/full-time in Bangalore, India. | The intern title is relevant, but its full-time label, graduation dates, degree and experience requirements were not inspected. SDE I’s exact eligibility is unconfirmed; its title may not match the current title keyword list. | The board is publicly viewable. The pinned adapter derives `/v0/postings/hevodata`, ignores the browser `department=Engineering` query, and fetches the entire Lever board with descriptions. Official Lever docs retrieved here describe an authenticated `/v1` API, not this `/v0` endpoint; Lever robots allows `*` but blocks named AI crawlers, while the `career-ops/1.0` UA is not named. **DEFER** pending official v0/source-use clarification, policy approval and role eligibility. |
| **Drivetrain** | Official URL `https://drivetrain.ai/` returned HTTP 429; no retry or workaround. The public Lever board page is HTTP 200 and its footer links to `http://www.drivetrain.ai`. | Lever. The board title is “Drivetrain” and its footer links to the supplied corporate domain. The official HTTPS site itself could not be confirmed in this session because it returned 429. | [Backend Engineer](https://jobs.lever.co/drivetrain/e2aafad8-a785-4515-9e3f-2b4981b727b8), Remote—Full Time—India; [Engineering Intern – Gen AI for FP&A Platform](https://jobs.lever.co/drivetrain/bc1c17bc-86ac-4f00-a1e3-0eb85aae4fdc), Remote—Internship—India; [Quality Assurance Engineering Intern](https://jobs.lever.co/drivetrain/edb8a8f7-16ac-4177-bf2e-99c36ab47d5f), Remote—Internship—India. | Relevant role families and India-located remote/intern categories are visible on the board. Degree, graduation-date and years-of-experience requirements were not inspected; no candidate qualification is claimed. | Same Lever `/v0` documentation and robots caveats as Hevo. The `?team=Engineering` UI filter is not carried into the provider endpoint; it fetches the whole board, including full descriptions. **DEFER** pending official source-use/endpoint clarification and job-level eligibility review. |
| **Plane Software** | The Ashby board root `https://jobs.ashbyhq.com/plane` is HTTP 200 and titled “Plane Software, Inc. Jobs.” Its bounded HTML contains the `plane.so` host. `https://plane.so/` is HTTP 200 and titled “AI-native project management (Plane).” No reciprocal careers link was found in the bounded homepage excerpt. | Ashby. The company name on the board, `plane.so` reference in board metadata and matching official-site brand support the `plane` slug; the root board was checked rather than inferred from the slug alone. | [Software Engineer, Backend (Python)](https://jobs.ashbyhq.com/plane/00beeb42-56c0-48ce-9082-9fba93836b54) and [Software Engineer, Backend (Node.js)](https://jobs.ashbyhq.com/plane/188f905e-3f6f-4569-9a32-d8ec48dfe656). Bounded Ashby board fields show `locationName: India` and `employmentType: FullTime`; `workplaceType` was null. | Strong title-family match and India location. Full-time is shown, but work mode, years of experience, education and 2027-graduate eligibility remain unconfirmed from the bounded excerpts. No full JD was fetched. | Ashby’s official Job Postings API docs specify the public GET `/posting-api/job-board/{JOB_BOARD_NAME}` and describe title, location, employment type, description and publication fields. The adapter fetches the **whole board**, including descriptions, not only these two IDs. `jobs.ashbyhq.com/robots.txt` disallows `/api/` on that host; the provider uses the separate `api.ashbyhq.com` host, whose robots request returned 401. **SUITABLE CANDIDATE**, with source-use approval still **PENDING** and manual eligibility review required. |
| **Sardine** | Official site `https://www.sardine.ai/` is HTTP 200 and titled “Agentic Financial Crime Platform for Fraud Prevention & AML.” Ashby root `https://jobs.ashbyhq.com/sardine` is HTTP 200 and titled “Sardine Jobs.” A reciprocal careers link was not found in the bounded official-homepage excerpt. | Ashby. Root board title and the supplied posting title `@ Sardine` match the official company brand. | [Software Engineer – Backend](https://jobs.ashbyhq.com/sardine/2524d206-42cb-4202-8d85-59e47c72d704). The bounded posting excerpt states “7+ years” of backend experience and “Location: Remote - India.” | India-remote is explicitly stated for this role, but 7+ years is not an early-career/graduate role. This does not establish the candidate’s qualification. No other relevant entry-level listing was verified. | Same documented Ashby endpoint and whole-board/full-description breadth as Plane. Ashby API-host robots policy was inaccessible (401). **REJECT** for the supplied role in this initial early-career shortlist; a future graduate-level opening could be reviewed separately. |
| **SupplyHouse.com** | Official homepage `https://www.supplyhouse.com/` returned HTTP 403. The Greenhouse board root `https://job-boards.greenhouse.io/supplyhouse` returned 302 to `https://www.supplyhouse.com/careers`; that exact destination returned 403. The supplied Greenhouse posting page is HTTP 200 and links “Back to jobs” to the same careers URL. | Greenhouse. The board root’s redirect to the company careers path and the posting title “at SupplyHouse.com” support board ownership, although the official careers page could not be read. | [Backend Engineer](https://job-boards.greenhouse.io/supplyhouse/jobs/6208712004). Bounded excerpt states Remote from India, full-time, a Bachelor’s or Master’s degree, and 3+ years’ backend/full-stack experience. | Location is explicitly India-remote; the 3+ years requirement is a material mismatch for the intended early-career 2027-graduate search. No claim of candidate qualification. | Greenhouse’s official docs confirm public GET endpoints require no auth; `boards-api.greenhouse.io/robots.txt` disallows only `/embed/`, not the board API path. The Greenhouse adapter requests `?content=true`, retrieving full descriptions for every job on the board; the board size was not established. Company `/careers` and robots requests returned 403, so terms/policy review remains incomplete. **REJECT** for this initial early-career shortlist based on the supplied role; reconsider only if a suitable junior opening and access approval are established. |

## 5. ATS integration contract and risks

Static source review at the pinned commit confirms:

- **Lever** (`providers/lever.mjs`) derives the board slug from `jobs.lever.co/{slug}` and calls `GET https://api.lever.co/v0/postings/{slug}`; it does not pass the `department`/`team` query from the public page. It returns the board-wide list, including full descriptions, in one response. The provider does not perform an individual-posting detail request or pagination.
- **Ashby** (`providers/ashby.mjs`) derives the org slug from `jobs.ashbyhq.com/{org}` and calls `GET https://api.ashbyhq.com/posting-api/job-board/{org}?includeCompensation=true`; a job ID in the careers URL is not used to limit the feed. It returns all jobs and their `descriptionPlain` in one response, with retry/backoff. `ashby: { embed: true }` is a separate optional HTML/hydration fallback, not used here.
- **Greenhouse** (`providers/greenhouse.mjs`) takes the first board path segment from a posting URL and calls `GET https://boards-api.greenhouse.io/v1/boards/{slug}/jobs?content=true`; the posting ID is not a per-job filter. The official GET is public/no-auth and returns all postings; `content=true` includes the full description for each. The separate application POST documented by Greenhouse requires Basic Auth; career-ops’ source adapter does not call it.
- These are board feeds, not a way to scan only the single posting link supplied. The title filter is applied after the provider fetch, so it cannot prevent the source request from retrieving irrelevant rows/descriptions. For A/B, the browser query parameters do not constrain the Lever API request.
- `portals.yml` would use one `tracked_companies` entry per employer, with `name`, the verified ATS `careers_url`, and optionally explicit `provider: ashby` for Plane. `CAREER_OPS_ROOT` must be set only for the future process. No configuration was changed here.

### Documentation, robots, and policy findings

- **Greenhouse:** the official Job Board API documentation states GET endpoints are public and need no authentication; list jobs is `GET /v1/boards/{board_token}/jobs`, and `content=true` includes full job content. The application-submission POST is a distinct authenticated endpoint. The official Greenhouse API-host robots file returned `User-agent: *` with only `/embed/` disallowed; the public board-host robots response had no standard directives in the bounded response. This is not a terms-of-use determination.
- **Lever:** the official developer guide retrieved is for the `/v1` API, says those requests require authentication, and documents pagination/rate limits. It does **not** document the provider’s `/v0/postings/{slug}` path in the guide reviewed. Public Lever board pages are reachable; the API-host robots file allows `*` and says `Crawl-delay: 1`. The public board host allows `*` but blocks named AI crawlers such as ClaudeBot/anthropic-ai. The career-ops UA is a distinct identifier, but that distinction alone is not permission; formal authorization for the `/v0` feed remains pending.
- **Ashby:** official Ashby documentation describes the public job postings GET endpoint and its returned fields. The `jobs.ashbyhq.com` robots file disallows `/api/`, `/meeting/` and `/b/`; the inspected public job pages are not under those paths. The adapter’s API host is separate; its robots file returned HTTP 401, so no policy conclusion can be drawn from it. Source use remains pending approval rather than assumed.
- **Company policies/access:** Hevo and Sardine robots responses allow the general user-agent on public paths, with unrelated exclusions. Drivetrain’s official homepage returned 429 and its robots policy was not checked; no retry was made. SupplyHouse’s homepage, careers page and robots request returned 403. These results do not establish legal permission or imply a challenge should be bypassed.
- No login, CAPTCHA, MFA, browser session, anti-bot workaround or authenticated endpoint was used. No third-party job board was searched.

## 6. Recommended initial shortlist and exact later configuration fields

**Shortlist: Plane Software only, as a candidate pending approval.** Its board is employer-attributed, its official Ashby endpoint is documented, and the observed titles/location align with Backend and India-based discovery. The postings’ level/degree requirements are still unconfirmed, the work mode is not stated in the exposed fields, and the Ashby API-host policy/terms need approval before activation.

If the user approves after those checks, the only source-entry fields needed are:

```yaml
tracked_companies:
  - name: "Plane Software, Inc."
    careers_url: "https://jobs.ashbyhq.com/plane"
    provider: "ashby"
job_boards: []
search_queries: []
skip_tiers: []
```

**Proposal only; not written.** Keep current role/title filters unchanged. Do not invent cities. The existing `skip_tiers: []` is appropriate. Review India location and work eligibility manually for each listing; `India` location is not proof of a specific work authorization condition, and unknown experience requirements remain unconfirmed.

Hevo/Drivetrain are a second-tier deferred pair: their board listings are more directly aligned, but the Lever adapter’s exact public endpoint is not documented in the official guide retrieved and the boards’ AI-crawler policy raises an unresolved approval question. SupplyHouse and Sardine should not be part of this early-career shortlist based on the only supplied/current roles inspected.

## 7. Final integrity and activity checks

- Repository HEAD/origin remain the values in §2; tracked/full Git status is clean.
- `package.json`, `package-lock.json`, and the existing `portals.yml` hash remain unchanged.
- No source, profile, tracker, candidate JSON, or other configuration was edited. This report is the only new deliverable.
- Only bounded GET research of the named company/ATS pages, linked official pages, robots files, and official vendor documentation occurred. No ATS feed API or provider fetch was called. Direct posting pages were read as bounded prefixes/ranges; no full job description was saved or reproduced. No scanner, discovery command, verifier, browser, model endpoint, or application workflow was invoked.
- No credentials, account, expanded user-profile path, username, or Windows SID was used or recorded.

**Unresolved before activation:** explicit user approval; Lever `/v0` endpoint documentation/terms for Hevo and Drivetrain; Ashby API-host terms/policy and Plane listing experience requirements; Drivetrain official-site availability; SupplyHouse official-site 403; and any future listing’s explicit India work eligibility.
