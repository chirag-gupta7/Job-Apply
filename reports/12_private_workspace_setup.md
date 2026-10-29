# Task 12 — Private Workspace and Safe Configuration Setup

**Date:** 2026-10-29  
**Working directory:** `D:\document\job apply`  
**Repository:** `D:\document\job apply\tools\career-ops` at `a6d7526c05861b1df9b70c011f8b1a84c44252ac`  
**Data root:** `D:\document\career-ops-data`  
**Status:** **SCAFFOLD CREATED — PRIVATE CANDIDATE DATA NOT YET APPROVED** (inherited ACL is broad; no runtime validation performed).

## 1. Executive summary

Created an isolated, non-personal career-ops scaffold at `D:\document\career-ops-data` with the required directories, a minimal profile, a source-disabled `portals.yml`, a blank canonical tracker, and a claim-source map template. Both YAML files parse successfully with the already-installed `js-yaml` dependency; no project code was imported or executed.

**Critical privacy finding:** `D:\document` inherits access for `NT AUTHORITY\Authenticated Users` with **Modify** and `BUILTIN\Users` with **ReadAndExecute**. The new data root inherits the same. It is therefore **not approved for candidate personal data**. Only non-personal scaffolding was created, and no candidate JSON was copied, moved, linked, or referenced by absolute path.

No source is enabled: `search_queries`, `tracked_companies`, and `job_boards` are empty. No model, job API, web search, browser, application, or message was triggered. Runtime schema validity was not proven because no application command was run.

## 2. Preflight integrity results

| Check | Result |
|---|---|
| Working directory | `D:\document\job apply` — exact match |
| Git HEAD | `a6d7526c05861b1df9b70c011f8b1a84c44252ac` |
| Origin | `https://github.com/career-ops-hq/career-ops.git` |
| Tracked working tree | Clean |
| `package.json` SHA-256 | `10ca52815f65e4ba14ea4025828f2f9b212d1b684a2ab495a173d3a461df1fac` |
| `package-lock.json` SHA-256 | `cb14c2cd24a3676514af291914db76ace250a8bbf2d6708006f02300028c22cc` |
| Candidate `full-profile.json` | `2f767145d8cb3bb619f0e77ff5f9f0e804a6026e1d2122e7b3fa50dcc9161fdb` — PASS |
| Candidate `internships.json` | `606e03d0d98d783a00c0afb002a08dc1958b065ea245a9b232fee8b0d5da2aea` — PASS |
| Candidate `projects.json` | `86749dc8d6f1aa9523e6a57f1decf3a74c911e18eb3ed3ee2000f16b8725e4f2` — PASS |
| Data root before creation | Absent — no conflict |

## 3. Private data-root creation and ACL assessment

Created `D:\document\career-ops-data` and the six documented directories. ACL metadata from the parent (`D:\document`):

| Group | Access | Inherited |
|---|---|---|
| `NT AUTHORITY\Authenticated Users` | **Modify** | Yes |
| `BUILTIN\Users` | **ReadAndExecute** | Yes |

No account names were printed. No ACL, environment variable, system setting, or Git config was modified.

**Decision:** because authenticated local users can modify files and ordinary users can read them, this directory is treated as **non-personal only**. Future candidate material (working CV, contact data, application records) must not be stored here until permissions are narrowed or a different private location is approved in a separate task. The scaffold contains no candidate facts.

## 4. Directories and configuration files created

```text
D:\document\career-ops-data\
├── config\profile.yml                  (non-personal role/location scaffold)
├── data\applications.md               (blank canonical tracker)
├── data\claim-source-map.template.md  (provenance template, no claims)
├── documents\                          (empty)
├── jds\                                (empty)
├── output\                             (empty)
├── reports\                            (empty)
└── portals.yml                         (inactive discovery policy)
```

Only these directories and files were created. No extra database, tracker, résumé, or company list.

### Profile scaffold (`config/profile.yml`)

- `target_roles.primary`: **Backend Engineer**, **Python Developer**.
- `target_roles.archetypes`: Backend Engineering and Python Engineering (`fit: primary`); Cloud Security and Cybersecurity (`fit: secondary`), all marked early-career (internship/graduate/new-grad/junior).
- `location.country`: **India** only.
- `language.output`: `en`.
- **Omitted deliberately:** name, email, phone, address, education, work history, skills, projects, salary, visa status/authorization, city/timezone.

### Portal scaffold (`portals.yml`)

- `title_filter.positive` includes Backend, Python Developer/Engineer, Software Engineer, `word:Intern`, Internship, Graduate, Trainee, `word:Junior`, Entry Level/Entry-Level, Cloud Security, Cybersecurity, Security Analyst.
- `title_filter.negative: []` — no default exclusion.
- `skip_tiers: []` — **ordinary and unknown titles are not excluded**.
- `search_queries: []`, `tracked_companies: []`, `job_boards: []` — **nothing enabled**.
- YAML comments state the India / India-eligible-remote / relocation discovery policy and the disabled-source policy. Geography is **not** encoded as an automatic filter because a location string cannot prove work eligibility.

### Blank tracker (`data/applications.md`)

Uses the canonical nine-column header with no rows. No invented jobs, companies, contacts, or statuses. This is the single canonical tracker; no second database was created.

### Claim-source template (`data/claim-source-map.template.md`)

Fields: claim ID, generated claim, source file, JSON Pointer, source-file SHA-256, evidence status, candidate approval status, notes/conflicts. It contains placeholder tokens only, no personal claims. Reads are referenced by filename in the template; no absolute candidate paths and no copies exist in the data root.

## 5. Schema evidence and unverified assumptions

| Item | Evidence at pinned revision | Status |
|---|---|---|
| Data root precedence/layout | `path-resolver.mjs`: `CAREER_OPS_ROOT`/`CAREER_OPS_DATA_DIR` > marker > plugin default > checkout; tracker at `{DATA_ROOT}/data/applications.md` | Verified by source reading |
| Profile keys | `config/profile.example.yml` (`target_roles.primary`, `archetypes`, `location`, `language.output`) | Keys are schema-supported; **candidate block intentionally absent** — runtime behavior unverified |
| Portal keys | `templates/portals.example.yml` and `validate-portals.mjs` (`title_filter.positive/negative`, `skip_tiers`, `search_queries`, `tracked_companies`, `job_boards`) | Structure is schema-supported; empty-source runtime behavior unverified |
| Canonical states | `templates/states.yml` | Verified by source reading |
| Tracker format | `AGENTS.md`/`DATA_CONTRACT.md` header shape plus `templates/states.yml` | Header format verified; runtime unverified |
| YAML syntax | Parsed with installed `js-yaml` | Both files parse |
| Full runtime validity | No `doctor.mjs`, `validate-portals.mjs`, scan, or application command was run | **Unverified** |

## 6. Search policy and source-disablement decisions

- No employer identifiers, board tokens, tenant names, or application URLs were guessed.
- No broad web-search query is enabled; no remote-job-board scraping.
- Not configured: LinkedIn, Indeed, Naukri, JobSpy, login-protected or CAPTCHA-dependent sources, browser-session providers, undocumented endpoints, POST-based search, Freehire caching, or any source that would send candidate data externally.
- Because no source is enabled, an executable scan would find nothing. That is intentional: **discovery is configured but inactive** until a bounded company list and its access terms are approved.
- `skip_tiers` is empty, and `negative` is empty, so ordinary Backend Engineer and unknown-seniority titles are not discarded by default.

## 7. Candidate-data privacy boundary

- Candidate JSON files remain at `D:\document\job apply` and were not read for content, copied, moved, linked, or duplicated.
- The data root contains no candidate name, contact data, education, employment, project claim, salary, or work-authorization statement.
- The inherited ACL makes the data root unsuitable for private candidate material. Do not place a working CV, tracker rows, or outreach records there until a narrower permission model or alternate private location is explicitly approved.
- No model/API keys, OAuth settings, or integration configuration were created. No files reference credentials.

## 8. YAML/configuration checks

| Check | Result |
|---|---|
| `config/profile.yml` parses | Yes — top-level keys: `target_roles`, `location`, `language` |
| `portals.yml` parses | Yes — top-level keys: `title_filter`, `skip_tiers`, `search_queries`, `tracked_companies`, `job_boards` |
| Invented keys | None observed relative to the example schema |
| Active source list | Empty |
| Broad web-search queries | None |
| `skip_tiers` excludes `mid` | No — empty |
| Candidate JSON copies in data root | 0 |
| Absolute candidate-path references | 0 |
| Likely secret patterns | 0 matches |
| Runtime validation | **Not performed** — no project command run |

## 9. Post-creation integrity results

| Check | Result |
|---|---|
| Candidate hashes | All three PASS (values in §2) |
| Repository HEAD / origin | Unchanged |
| `package.json` / `package-lock.json` | Unchanged (hashes in §2) |
| Application code / repository config | Not edited |
| New deliverables | Data-root scaffold + this report only |
| Browser / Chromium | Not launched or downloaded |
| Application / scanner / model / external API | Not invoked |
| Secrets printed | None |

**Files added:** the six directories and four scaffold files under `D:\document\career-ops-data`, plus `reports/12_private_workspace_setup.md`. No other file was created or modified.

## 10. Remaining blockers and next task

1. **ACL/privacy blocker:** inherited `Authenticated Users: Modify` and `BUILTIN\Users: ReadAndExecute`. Obtain a narrower ACL or an alternate approved private location before storing candidate data.
2. **No approved discovery source:** no company boards or permitted feeds are configured. A bounded India/international company list and source terms must be approved.
3. **Incomplete profile by design:** candidate-specific fields are absent; the scaffold cannot generate employer-facing material yet.
4. **Tracker status mismatch:** canonical states do not include `discovered`, `reviewed`, `prepared`, `filled`, `confirmed`, `closed`, or `outcome unknown`. Interim mapping:

| Desired state | Canonical state / handling |
|---|---|
| discovered / reviewed / prepared | `Evaluated` |
| filled | `Evaluated` + Notes (not submitted) |
| submitted | `Applied` |
| confirmed | `Applied` + Notes stating confirmation |
| closed | `Discarded` |
| rejected | `Rejected` |
| withdrawn | `Discarded` |
| outcome unknown | `Applied` + Notes "outcome unknown" |

A later task must decide whether richer states need a supported schema change or remain Notes-based.

**Recommended next task (not authorized):** resolve the private-data permission boundary (or select another location), then approve a small list of India/India-remote employer career pages and document their access terms. Do not populate candidate data, enable scanning, or run the application until then.
