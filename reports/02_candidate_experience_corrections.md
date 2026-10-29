# Task 02 — Candidate Experience Corrections

Audit date: 2026-10-28  
Working directory: `D:\document\job apply`

## 1. Executive summary

Preflight passed: the working directory matched the requested path, and all three source-file SHA-256 hashes matched the Task 01 baseline before edits. The previous audit report and all three JSON files were read using UTF-8 BOM support.

Applied the requested corrections:

- Consolidated the two PrimeTrade entries into one internship relationship with two candidate-reported engagement blocks and two distinct nested project records.
- Removed Peblo from internship/employment history. No separate Peblo project record existed in `projects.json`, so no project record was removed.
- Added Labmentix using the candidate-supplied offer terms, without claiming actual completion, a service end date, or technical achievements.
- Added seven Labmentix-associated projects with project-level descriptions and stacks taken from read-only public GitHub README pages. Each entry separates candidate-reported participation, the coordinator's repository account, and the still-unknown individual contribution.
- Removed stale explicit internship-count/recency wording from three profile-summary strings without adding a new total.

All three JSON files parse successfully after editing. `internships.json` now has 6 array records and 52 XYZ bullets, counting the preserved PrimeTrade project bullets nested under the single employer record. `projects.json` now has 21 records total (4 flagships plus 17 additional projects).

## 2. Preflight hashes and source integrity

The current directory was confirmed as `D:\document\job apply`. Before modifying anything, all three current source hashes matched the Task 01 baseline exactly:

| Source | Preflight result |
|---|---|
| `full-profile.json` | Match: `1689b77d6970b6015062e02fb96618aaf2a7c775485480800901330979453c8a` |
| `internships.json` | Match: `64c1d444dbb021f89a3bf3c99ed07ad5a8100756adf5622ea20bcdf9e21d2def` |
| `projects.json` | Match: `b16313fbefcd88ac36d4aa3528577dbb239293e7e27bf21181d16b292f988e08` |

`reports/01_candidate_source_audit.md` was read and not edited. All source files had a UTF-8 BOM and CRLF line endings at preflight; those conventions were preserved. No Labmentix PDF was found among the top-level PDF files in the working directory. No recursive search was performed, so the offer letter was not locally inspected; its details below are only the candidate-supplied details in the task.

## 3. PrimeTrade consolidation: before/after structure and preserved project details

### Before

- `internships.json` `/internships/5` was a PrimeTrade `Python Developer Intern` record for the Binance Trading Bot, with 4 XYZ bullets and a January 2026 documentation/assignment date string.
- `/internships/6` was a second PrimeTrade `Python Developer Intern` record for the Simplified Trading Bot, with 2 XYZ bullets and a January 2026 commit date string.
- The two records had separate IDs, repository URLs, metrics, stacks, architecture notes, evidence references, and bullets. Their dates described project documentation/commits, not a confirmed full employment chronology.

### After

There is one PrimeTrade record at `internships.json` `/internships/4`, with role `Python Developer Intern`. It records one candidate-reported relationship consisting of two approximately three-month blocks. Both block `start_date` and `end_date` values are `null` because exact dates were not supplied. The source explicitly says that the existing January 2026 values are project-documentation dates, not internship dates.

The candidate reports being rehired for the second block after the first three months and attributes the rehire to the quality of their work. The source labels this candidate-reported, not employer-verified. The candidate did not say which project belongs to which block; the record does not infer that mapping.

| Nested project path | Project and exact repository URL | Distinct details retained |
|---|---|---|
| `/internships/4/projects/0` | Binance Trading Bot — `https://github.com/chirag-gupta7/Binance-trading-bot` | Original project ID, documentation-date text (now explicitly named `project_documentation_dates`), architecture, evidence list, keywords, brief, metrics, repo status, stack, and all 4 XYZ bullets. Includes its own 8-command CLI/6 order-strategy-type description, validation bounds, simulation mode, and security/logging details. |
| `/internships/4/projects/1` | Simplified Trading Bot — `https://github.com/chirag-gupta7/Simplified-Trading-Bot` | Original project ID, documentation-date text, architecture, evidence list, keywords, brief, metrics, repo status, stack, and both XYZ bullets. Its 3 order types, testnet scope, 5-file separation, server-time sync, and rotating-log details remain distinct from the other bot. |

The original project-specific metrics were not merged. The shared employer, role, and remote work-location are represented once on the parent relationship; both repository URLs and project-specific evidence remain on their corresponding nested projects.

## 4. Peblo removal and any preserved project record

- Removed the Peblo record that was formerly `internships.json` `/internships/4`. It had 5 XYZ bullets and was labeled an internship project.
- The three existing `experience_blocks` are Axlerator, Cognifyz, and Grozoom; none mentions Peblo or explicitly classifies it as employment. They were left unchanged.
- Checked the existing 14 project records in `projects.json`: no independent Peblo project record was present. Therefore no Peblo project was deleted or rewritten there; the seven new records were appended separately.
- The record count and bullet total were recalculated: `7 - 1 Peblo - 1 duplicate PrimeTrade entry + 1 Labmentix = 6 array records`; `57 - 5 Peblo bullets = 52 bullets`. All 6 PrimeTrade bullets remain in the nested project records; Labmentix has no XYZ bullets.

## 5. Labmentix record and offer-letter evidence limitations

Added `internships.json` `/internships/5` with:

- Organization: Labmentix.
- Position: Python Development Intern.
- Printed offer-letter date: `10/07/2026`, retained exactly as printed and marked format-ambiguous.
- Joining date stated: `15-Jul-2026`.
- Duration stated: six months.
- Signatory stated: Swaraj Phatangade, Founder & CEO.
- Letterhead location: Bengaluru, Karnataka, India; website shown: `www.labmentix.in`.

The record marks the internship identification as candidate-reported and the offer terms as candidate-supplied. `service_end_date` is `null`; `completion_status` is `not established`; no work location, responsibilities, stack, metrics, or achievement bullets were invented. The offer terms do not independently establish that service began or that any duration was completed. The PDF was not found among top-level files in the current working directory and was not inspected locally; no recursive search was performed.

## 6. Seven project records and contribution/evidence status

Each record is appended to `projects.json` `/more_projects/10` through `/more_projects/16`. Public repository README pages were fetched read-only on 2026-10-28. Descriptions and stacks below describe the **repository project**, not work personally performed by the candidate. Every record contains an `attribution` object stating that the Labmentix association and candidate participation are candidate-reported, that `nitendranagupta` is identified by the candidate as the project coordinator's GitHub account, and that the candidate's specific responsibilities are not supplied or verified.

| JSON Pointer / project | Exact repository URL | README-supported project purpose and documented stack | Contribution/evidence status |
|---|---|---|---|
| `/more_projects/10` — Digital Media Player | `https://github.com/nitendranagupta/digital_media_player` | Music/podcast streaming, uploads, playlists, search, queue, recent playback, and audio visualization. React, Tailwind CSS, Axios, React Icons, Web Audio API, Flask, Flask-JWT-Extended, Flask-SQLAlchemy, Flask-CORS, SQLite (development). | Candidate reports participation during Labmentix; README supports project description/stack only. No personal tasks identified. |
| `/more_projects/11` — Cloud Storage | `https://github.com/nitendranagupta/cloud_storage` | Authenticated file storage with folders, previews/downloads, search, sharing, and trash/restore. React 19, Vite, Tailwind CSS, React Router/Query, Axios, React Dropzone, Lucide Icons, FastAPI, SQLAlchemy, PostgreSQL, Supabase Storage, JWT libraries. | Candidate-reported participation; README does not establish individual authorship or responsibilities. |
| `/more_projects/12` — Document Signature | `https://github.com/nitendranagupta/seal_desk` | PDF upload/signing workflow, signature fields, signer invitations, typed signature capture, signed-PDF generation, status, and audit events. FastAPI, React, SQLAlchemy, PyMuPDF, JWT, Docker. | Candidate-reported participation. README sections conflict on drawn/uploaded signature capture; this conflict is preserved in `documentation_notes` rather than treating those features as confirmed shipped functionality. |
| `/more_projects/13` — 1-on-1 Mentor App | `https://github.com/nitendranagupta/1on1_mentoring_platform` | Real-time mentoring sessions with video, chat, collaborative Monaco editing, cursor presence, and code snapshots. Next.js 15, TypeScript, Tailwind CSS, FastAPI/Python, PostgreSQL, SQLAlchemy, WebSockets, Monaco, WebRTC, Zustand, JWT, Yjs. | Candidate-reported participation; no personal responsibilities or authorship established. |
| `/more_projects/14` — AI Data Analysis Dashboard | `https://github.com/nitendranagupta/AI_data_analysis_dashboard` | Dataset upload, natural-language analysis, generated insights/charts, dashboards, and exports. Next.js 15, TypeScript, Tailwind, Shadcn UI, Recharts, TanStack Query, Zustand, FastAPI, Pydantic, SQLAlchemy, Alembic, PostgreSQL, Redis, Celery, Docker, and documented AI-provider libraries. | Candidate-reported participation. Repository About text names Gemini, while README stack/architecture names OpenAI GPT-4 Turbo, LangChain, and Instructor; the conflict is recorded in `documentation_notes`, not resolved. |
| `/more_projects/15` — Social Media Platform | `https://github.com/nitendranagupta/postit` | Reddit-style communities, posts, comments/replies, voting, profiles, search, and image uploads. Next.js 15, TypeScript, Tailwind, Axios, React Query, React Hook Form, Zod, FastAPI, SQLAlchemy, PostgreSQL, Alembic, JWT, Passlib/bcrypt, Cloudinary. | Candidate-reported participation; no personal responsibilities or authorship established. |
| `/more_projects/16` — Phishing / Scam Detector | `https://github.com/nitendranagupta/Phishguard` | URL/email analysis, threat-intelligence lookups, AI classification, risk scoring, scan history, dashboards, and reports. Next.js 15, TypeScript, Tailwind, Recharts, FastAPI/Python, SQLAlchemy, Alembic, PostgreSQL, Redis, JWT/bcrypt, Gemini API, VirusTotal, Google Safe Browsing, Docker. | Candidate-reported participation; README supports project-level description/stack only. No personal tasks identified. |

The exact repository URL is stored in each record's `repo_url`. No repository was cloned, and no source code was run. Repository account ownership or visibility was not treated as proof of the candidate's contribution.

## 7. Schema, metadata, narrative, and count changes

- `internships.json` `/meta/count`: 7 → 6; added `count_note` to clarify this is the array-record count, not a count of independently verified employment relationships.
- `internships.json` `/meta/total_bullets`: 57 → 52. This counts all remaining XYZ bullets, including the six project bullets nested under PrimeTrade.
- Updated `internships.json` `/how_to_use_this_file` from “All 7 internships” to “All 6 experience records,” with wording that bullets/evidence exist where supplied.
- Replaced the two PrimeTrade array entries with one parent record, two `engagement_blocks`, and two nested project records. Original project evidence, metrics, stacks, IDs, and bullets are retained. Project documentation dates are now labeled as such.
- Removed only the Peblo experience record. No Peblo mention existed in `experience_blocks`; that object was not changed.
- Added the Labmentix record without technical claims or a completed-duration claim.
- `projects.json` `/more_projects`: 10 → 17 records; total projects: 14 → 21. No additional-project count field existed. The 4 flagship records and their declared 80 bullets were unchanged.
- New project records use existing `id`, `name`, `type`, `repo_url`, `description`, `stack`, and `evidence` fields, plus one optional `attribution` object to keep Labmentix association, repository account, and candidate participation distinct. Optional `documentation_notes` preserves the two README conflicts. No candidate `role`, XYZ bullet, metric, or sole-author claim was added.
- `full-profile.json` changed only three existing strings: `/bios/professional_summary` and `/bios/video_intro_script_45s` no longer claim exactly three internships and now mention the PrimeTrade experience; `/bios/linkedin_about` no longer calls Axlerator “most recent” while PrimeTrade dates remain unknown. No new internship total was asserted. All other profile fields were preserved.

## 8. Before/after SHA-256 hashes for every source file

“Before” is the verified Task 01 baseline. “After” is the final hash after all source edits and validation.

| Source | Before SHA-256 | After SHA-256 | Size before → after |
|---|---|---|---:|
| `full-profile.json` | `1689b77d6970b6015062e02fb96618aaf2a7c775485480800901330979453c8a` | `2f767145d8cb3bb619f0e77ff5f9f0e804a6026e1d2122e7b3fa50dcc9161fdb` | 20,538 → 20,579 bytes |
| `internships.json` | `64c1d444dbb021f89a3bf3c99ed07ad5a8100756adf5622ea20bcdf9e21d2def` | `606e03d0d98d783a00c0afb002a08dc1958b065ea245a9b232fee8b0d5da2aea` | 77,587 → 70,453 bytes |
| `projects.json` | `b16313fbefcd88ac36d4aa3528577dbb239293e7e27bf21181d16b292f988e08` | `86749dc8d6f1aa9523e6a57f1decf3a74c911e18eb3ed3ee2000f16b8725e4f2` | 172,794 → 188,316 bytes |

## 9. Validation results

- Parsed all three final JSON files with UTF-8 BOM support; each retained its BOM and CRLF line endings.
- Verified the PrimeTrade parent is one internship record, contains exactly two engagement blocks, and retains the two correct project URLs exactly once. Each project retains its own metrics and bullets; block dates remain unresolved and null.
- Verified Peblo is absent from the internship array and the three `experience_blocks`; no pre-existing Peblo project record was found or deleted.
- Verified one Labmentix record with the supplied role, joining date, stated duration, printed date, signatory, letterhead location, and website. No end date, completion, responsibilities, stack, or achievement was fabricated.
- Verified all seven requested Labmentix URLs occur exactly once, the seven records have unique IDs, and the existing 14 project records remain before the appended entries. `flagships` and the 80 flagship bullets are unchanged.
- Recalculated the internship record/bullet totals: 6 and 52. No internship record ID is duplicated.
- Verified the profile edits are limited to the three named summary strings; no new internship total was introduced.
- Rechecked final hashes after validation. The initial preflight hashes matched Task 01; the differences from those baselines are the authorized edits described here.

## 10. Unresolved questions and required candidate confirmation

1. What were the exact start/end dates for each PrimeTrade block? Was there a gap or overlap? Can any employment/re-hire documentation support the relationship and dates? The rehire reason remains candidate-attributed, not employer-verified.
2. Which of the two PrimeTrade projects belongs to which block, if either can be assigned to a particular block?
3. Did Labmentix service actually begin on `15-Jul-2026`, and what are the actual service dates/current status? The offer terms alone do not prove commencement or completed service. What date format does the printed `10/07/2026` use, if known?
4. For each of the seven Labmentix-associated projects, what specific components, tasks, or decisions did the candidate personally contribute? Are there candidate-specific PRs, commits, tickets, or other evidence that can be cited? Do not infer contribution from the coordinator's account.
5. Can the candidate clarify the conflicting repository documentation for drawn/uploaded signatures in Document Signature and the Gemini-versus-OpenAI provider descriptions in AI Data Analysis Dashboard?

## 11. Files modified and read-only compliance notes

**Modified authorized source files:** `full-profile.json`, `internships.json`, and `projects.json`.  
**New report:** `reports/02_candidate_experience_corrections.md`.  
`reports/01_candidate_source_audit.md` was not modified. No other report, backup, temporary file, or project artifact was created.

The seven public GitHub repository pages/README content were reviewed read-only as permitted. No source code was run, no repository was cloned, no dependencies or credentials were accessed, and no API service was contacted. No unrelated directory was searched and no one was contacted. The offer letter was not locally inspected; the limitation is recorded above. The report contains no candidate contact details, DOB, secrets, tokens, or private operational URLs. Task 03 and job-search ecosystem research were not started.
