# Task 01 — Candidate Source-of-Truth Audit

Audit date: 2026-10-28  
Working directory: `D:\document\job apply`

## 1. Executive summary

All three named source files exist and parse as JSON when decoded with UTF-8 BOM support. Their byte sizes and SHA-256 hashes are recorded below as this audit's new baseline. The audit found no duplicate JSON object keys, duplicate primitive-array entries, or empty strings/arrays/objects. Explicit `null` values occur only in optional verification/link fields.

The files contain a substantial candidate profile: a named student, contact and public-profile links, education and marks, 7 experience records, 14 project records, 12 headline credential entries, 59 listed skills, preferences, and many quantified claims. These are **claims present in the supplied JSON**, not independently verified facts. Referenced PDFs, repositories, source trees, certificate pages, dashboards, and other documents were not opened; no internet or external service was used.

The main accuracy risks before employer-facing use are:

1. `internships.json` labels 7 records as internships, while profile bios describe 3 internships; several of the 7 records are explicitly assignments/projects or have no employment evidence/dates.
2. Important claims are incomplete or internally inconsistent: Grozoom dates are TBC; TypeMaster's lesson count is 200 versus about 260–300; SmartMeet's reported cache speedup does not align across descriptions; and the TaskFlow API name appears on two materially different project records.
3. The degree title, grades, credentials, employment, and project metrics have not been corroborated by the underlying records. Credential names, dates, and verification coverage are incomplete.
4. Work authorization wording is broad and jurisdiction-dependent; the exact legal status should be confirmed before application-form answers.
5. The data includes private contact/DOB information and a live webhook field. These should not be copied into public résumé content by default.

This report does not resolve these conflicts or certify any claim.

## 2. Audit scope and methodology

- Confirmed the current directory exactly matched `D:\document\job apply`; otherwise this audit would have stopped.
- Inspected only `full-profile.json`, `internships.json`, `projects.json`, and the filesystem metadata needed to identify those files and the requested report destination.
- Read each JSON file as UTF-8 with BOM support (`utf-8-sig`), parsed it, examined its structure and claims, and computed SHA-256 from its raw bytes before report creation.
- Did not inspect any referenced source code, PDFs, images, certificate pages, GitHub repositories, portfolio pages, or external services. A filename, URL, or evidence description inside a JSON file is a reference, not proof that the referenced item was checked.
- JSON Pointers below are RFC 6901-style paths relative to the named file's root; array indices are zero-based.
- “Supported by supplied data” means explicitly represented in these three files. It does **not** mean independently verified. “Derived” claims include their arithmetic or assumptions; plausible interpretations are not promoted to facts.

## 3. File integrity and SHA-256 baseline

Hashes below are the audit's local baseline; they were not compared with any prior project document.

| File | Exists / parse | Size | SHA-256 | Top-level structure and counts |
|---|---|---:|---|---|
| `full-profile.json` | Yes / valid JSON | 20,538 bytes | `1689b77d6970b6015062e02fb96618aaf2a7c775485480800901330979453c8a` | Object, 15 top-level keys; 12 headline credential records; 7 skill groups; 5 award/recognition strings. |
| `internships.json` | Yes / valid JSON | 77,587 bytes | `64c1d444dbb021f89a3bf3c99ed07ad5a8100756adf5622ea20bcdf9e21d2def` | Object, 4 top-level keys; 7 internship-array records; 3 free-text `experience_blocks`; 57 XYZ bullets. Declared count and bullet total match array counts. |
| `projects.json` | Yes / valid JSON | 172,794 bytes | `b16313fbefcd88ac36d4aa3528577dbb239293e7e27bf21181d16b292f988e08` | Object, 6 top-level keys; 4 flagship records with 80 XYZ bullets (declared total matches); 10 `more_projects` records. |

### Structure and data-quality observations

- `full-profile.json`: root sections include `identity`, `contact`, `education`, `skills`, `certifications_headline`, `certifications_note`, `awards_recognitions`, `bios`, `preferences`, `availability`, `languages`, `matching_job_titles`, and `open_items`. `education.sgpa` is an object; skills and coursework are arrays; most other profile values are strings or nested objects. Headline credential records use `date`, `id`, `issuer`, `name`, `type`, and `verify`.
- `internships.json`: `meta` contains declared counts; `experience_blocks` contains three narrative strings; `internships` contains seven objects with arrays for evidence, keywords, stack, and XYZ bullets, plus heterogeneous `metrics` objects. Some records have `links`/`repo_status`/`confidence`; others do not. Dates are unnormalized free text.
- `projects.json`: `pick_your_project` is a category-to-string object; `flagships` and `more_projects` are arrays; `meta` stores candidate/format/count; `universal_paragraph` and `how_to_use_this_file` are strings. Project records vary in fields and metric types. In `more_projects`, `repo` is used inconsistently (URL, source/repository names, or a description); the later records instead use `id` and generally omit a repository URL field.
- No duplicate JSON object keys, duplicate primitive-array values, or empty strings/arrays/objects were detected. There is no formal schema in the supplied files, so schema validity beyond JSON syntax cannot be established.
- Explicit nulls: 11 credential subfields in `full-profile.json` (missing credential date/ID/verification values); two `repo_url` values in `internships.json` (Axlerator and Cognifyz); and two `live_url` values in `projects.json` (SmartMeet-AI and TaskLogin-Manager). Null is distinguishable from a missing optional key, but there is no stated convention for either.
- Unicode punctuation such as em dashes, plus/minus, and multiplication signs is present and parses correctly. No malformed JSON or replacement-character issue was found.

## 4. Candidate profile inventory

### Identity, contact, location, and preferences

| Category | Explicit information in the files | Notes / omissions |
|---|---|---|
| Professional name | `Chirag Gupta`; certificate-name field says `Chirag`. (`full-profile.json` `/identity/name`, `/identity/name_on_certificates`) | Not necessarily a conflict, but the abbreviated credential display name could affect identity matching. Confirm how to explain it. |
| Contact | Email and phone are present (masked here); GitHub, LinkedIn, and portfolio links are present but not reproduced. (`/contact`) | Contact values were not tested. No street address is supplied. |
| Location / timezone | Jaipur, Rajasthan, India; `IST (UTC+5:30)`. (`/identity/city`, `/identity/timezone`) | City-level location only. Date of birth is present at `/identity/dob` but omitted from this report as private information. |
| Availability | Internship start “immediately”; full-time work “possible after graduation in June 2027.” (`/availability`) | Self-reported; no calendar/date or enrollment evidence supplied. |
| Preferences | Relocation/remote/hybrid/onsite stated as anywhere; any shift; starting expectation `6 LPA`, flexible by role band. (`/preferences`) | Preference, not a verified compensation requirement. Usually disclose salary only when asked. |
| Work authorization | Source says “India authorized; abroad-remote contractor roles need no sponsorship.” (`/preferences/work_auth`) | Broad and ambiguous; do not reuse as a universal legal answer without confirming the specific country, work arrangement, and status. |
| Languages | English and Hindi, each self-rated fluent in speaking, reading, and writing. (`/languages/0`, `/languages/1`) | No language credential or proficiency assessment supplied. |

### Education and academic record

- Degree as written: `Bachelor in computer science`; institution: Vivekananda Global University, Jaipur; university date string `(2023-2027)`; expected graduation `June 2027`. (`full-profile.json` `/education/degree`, `/education/university`, `/education/graduation`)
- CGPA is stated as `9.13/10 (Semesters 1-4; 93 credits)`. Semester SGPA values are 9.43, 8.24, 9.32, and 9.39. The awards list also says 9.13 CGPA through four semesters and lists O grades in DBMS, Data Science, ML, Networks, Business Intelligence, and Web Development. (`/education/cgpa`, `/education/sgpa`, `/awards_recognitions/2`)
- Coursework listed: DBMS; Operating Systems; Computer Networks; Cryptography & Network Security; Machine Learning; Principles of AI; Business Intelligence; Data Science; Cloud Web Services; Python; Java OOP; Data Structures in C. (`/education/coursework`)
- No official degree-title wording, transcript, school/board details, Class 10/12 marks, or entrance-exam result is present. The latter items are explicitly listed as open items at `/open_items`.

### Internships, employment, and internship-labelled projects

`internships.json` has 7 records. The profile narrative refers to 3 internships—Axlerator, Grozoom Ventures, and Cognifyz—while the array also includes an assignment, a project, and two PrimeTrade entries whose employment status/relationship needs clarification. The source labels are preserved below, not normalized.

| Record / source path | Dates, role, location, and stack as supplied | Responsibilities / reported outcomes |
|---|---|---|
| Axlerator `/internships/0` | Backend Engineer Intern; Jan–Jun 2026; remote (Mumbai company). Next.js 15, TypeScript, Prisma 6, Supabase/PostgreSQL, Zod, Twilio, Expo React Native, Render. | Database/schema work, query latency stated as 4–5s to about 1s, schema/workflow speedup stated as about 15%, migration, data verification, API routes, OTP, images, mobile app, and runbooks. `repo_url` is null; evidence array names local paths/certificate, which were not inspected. |
| Cognifyz `/internships/1` | Python Development Intern; Nov–Dec 2025; 8 weeks per source; Jaipur, Remote. Python, pandas, matplotlib, seaborn, Plotly, requests, BeautifulSoup, regex. | Scraper, data-cleaning pipeline, visualization tool; 13 programs and 6 chart types; dataset described as 9,551 × 21 with missing values reduced from 9 to 0. Narrative `experience_blocks` also claims about 40% less preprocessing, without a measurement method in the structured metrics. Certificate date is written `04/12/2025` and is ambiguous without a date-format convention. `repo_url` is null. |
| Grozoom Ventures `/internships/2` | Python/Backend Intern, security automation; remote; dates TBC. Python, requests, hashlib, pycryptodome, AES-CBC/PKCS7, SHA-256 proof-of-work, obfuscated-JavaScript reverse engineering. | Source claims a six-stage registration automation, AES interoperability, proof-of-work and telemetry handling. A completion flag is recorded, but its value is withheld here. Source explicitly says dates are not recorded and asks for proof before checks. Its repository is described as empty and “verified” in the JSON; that assertion was not independently checked. |
| Elvara Group `/internships/3` | Role says AI & Automation Engineering Intern; dated Sep 2026; type says `internship assignment (live tool, 2-4 hr brief)`; remote. n8n, OpenAI gpt-4o-mini, Groq gpt-oss-20b, Airtable, webhooks. | 11-node real-estate deal-analysis workflow and live acceptance test are claimed. The source identifies this as a short assignment, not clearly employment. A production webhook/dashboard link is present in `/links`; it is intentionally not reproduced and should not be placed in public materials without an access/security review. |
| Peblo `/internships/4` | `Peblo (internship project)`; 2026, 9 commits; no employment date range. Role says Full-Stack Developer (internship project, sole builder); remote. FastAPI, PostgreSQL, SQLAlchemy, Alembic, React, TypeScript, Docker Compose, GitHub Actions, pytest. | Internal publishing platform; source claims 19 endpoints, 5 tables, 14 tests, CI, and one-command boot. The record calls itself a project; employer/placement status and dates are not established by the JSON. |
| PrimeTrade full bot `/internships/5` | Python Developer Intern; Jan 2026; source dates refer to docs version 2026-01-22 and an assignment PDF. Remote. Python, Binance USDT-M Futures API, argparse, binance-connector, python-dotenv, requests. | 8-command CLI, 6 order/strategy types, validation, simulation mode, and security/logging work. The source does not provide a formal employment period or independently supplied employment evidence. |
| PrimeTrade simplified bot `/internships/6` | Python Developer Intern; Jan 2026; source dates refer to a commit on 2026-01-10 and call it a predecessor/sibling of the other bot. Remote. Python, python-binance, python-dotenv, Binance Futures Testnet. | Interactive testnet bot with 3 order types and layered files. Relationship to the other PrimeTrade record and to an actual internship is unresolved. |

### Projects, technical stacks, responsibilities, and outcomes

`projects.json` contains 4 flagship records and 10 additional project records. “Outcome” metrics below are claims in the JSON, not measurements independently reproduced in this audit.

| Project / source path | Stack and role as supplied | Selected claimed outcomes / evidence fields |
|---|---|---|
| SmartMeet-AI `/flagships/0` | Flask 3, React 18, Python, SQLAlchemy, Llama-3, ElevenLabs, Google Calendar/OAuth2, JWT; sole builder. | Voice calendar assistant; 16 intents, 16 API endpoints, 4 models, 103 backend + 19 frontend tests (122 by addition), timezone parser, caching, fallbacks. `repo_url` present; `live_url` null. Cache-speed claims conflict in scale; see §6. |
| Type-Master `/flagships/1` | Next.js 14, React 18, TypeScript, Express, Prisma, PostgreSQL, Redis, NextAuth, Gemini, Docker, GitHub Actions; sole builder. | 40 endpoints, 13 models, 626 commits, live URL, AI games, CI, curriculum listed as about 260–300 lessons here. Profile bios instead say 200 lessons. `repo_url` and `live_url` fields are present. |
| MCP-Website `/flagships/2` | HTML, Tailwind, vanilla JS, Chart.js, Canvas, Vercel; research/design/engineering role says sole builder. | 9 research sections, paper bundled, interactive demos and live URL claimed. Gemini proxy is described as documented/designed, not a shipped integration. Repository and live URL fields are present. |
| TaskLogin-Manager (TaskFlow API) `/flagships/3` | FastAPI, SQLAlchemy, SQLite, JWT/Bcrypt, Pydantic, pytest; sole backend builder. | 9 endpoints, 2 models, 8 tests; `live_url` null. A separate `TaskFlow-API` record in `more_projects/0` claims 10 endpoints, PostgreSQL, Docker, and React; relationship unresolved. |
| TaskFlow-API `/more_projects/0` | FastAPI, PostgreSQL, Docker, React, JWT/RBAC; role not separately specified. | 10 endpoints and permission matrix claimed. `repo` contains a GitHub-style URL string; no dates/metrics object. |
| AI-helper `/more_projects/1` | Flask, SocketIO, SQLAlchemy, ElevenLabs, Google Calendar/OAuth; role not separately specified. | Labeled SmartMeet predecessor; 14-table schema claim. Stack strings are split oddly (`Google`, `Calendar`, `14-table`, `schema`). |
| Portfolio `/more_projects/2` | HTML/CSS/JS, PDF.js, Vercel; role not stated. | Dual themes and 15 credential PDFs in a gallery claimed; “repo” field is a technology description rather than a repository URL. |
| Go curriculum + Little Lemon capstone `/more_projects/3` | Go/Gorilla Mux and Django REST Framework/Djoser. | 23 Go programs and a role-based ordering capstone claimed. Some stack terms are split across separate strings. |
| CS fundamentals (20 repos) `/more_projects/4` | C, Java, Python, unittest, discord.py. | C data structures, Java GUI, 15+ Python programs, and four unittest suites claimed. `repo` is a broad account/practice-repos reference, not a specific URL. |
| nothotdog `/more_projects/5` | Flask, HuggingFace Inference API, HTML/CSS/JS, Python dotenv; sole builder. | Image classifier proxy, six formats, 16 MB cap, 4 commits claimed; repository URL present. |
| Heart-disease-detection `/more_projects/6` | XGBoost, TensorFlow/Keras, scikit-learn, Optuna, pandas, CDC BRFSS 2020; sole builder (ML/paper/showcase). | AUC 0.9458, recall 0.9012, precision 0.7039, F1 0.79, and about 80K pseudo-labelled rows claimed. Repository URL present. These are research metrics, not evidence of clinical validation. |
| PHOTO-RECOGNIZER `/more_projects/7` | Flask 2, HuggingFace ViT, Pillow, NumPy, HTML/CSS/JS; sole builder. | 709 backend lines, six formats, metadata/forensics and natural-language insights claimed. Repository URL present. |
| Whiteboard-APP `/more_projects/8` | React 18, Canvas, hooks, Create React App; sole builder. | Six tools, 20-step history, touch support, 5 commits claimed. Repository URL present. |
| QR-Code-Generator `/more_projects/9` | React 18, TypeScript 4, qrcode, file-saver, PapaParse; sole builder. | Three sizes, four error-correction levels, CSV batch, 10-item history, one commit claimed. Repository URL present. |

### Skills, credentials, awards, and application targets

- **Skills (59 listed strings; no proficiency levels, years of use, or assessment):**
  - AI/data: pandas, NumPy, matplotlib, seaborn, Plotly, OpenAI GPT-4, Gemini, Llama-3, prompt engineering, web scraping (BeautifulSoup, Requests), ETL pipelines, data cleaning.
  - Backend: Python, Flask, Django REST Framework, FastAPI, Node.js, Express.js, REST API design, JWT, OAuth2, Go (Golang), TypeScript.
  - Data/databases: PostgreSQL, MySQL, SQLite, Redis, SQLAlchemy, Prisma ORM, query optimization, indexing.
  - DevOps: Docker, Docker Compose, Git/GitHub, GitHub Actions (CI/CD), Linux, Bash, Postman.
  - Frontend: React, Next.js, Tailwind CSS, HTML/CSS.
  - PM/soft: Agile, Scrum, stakeholder management, risk management, team leadership (hackathon).
  - Security: SIEM, IDS/IPS, network security, AES-CBC/PKCS7, SHA-256/MD5, reverse engineering (obfuscated JS), proof-of-work solving, vulnerability management, incident response, cloud security, NGFW, threat hunting.
- **12 headline credential entries** (`full-profile.json` `/certifications_headline`): Meta Back-End Developer; Google Cybersecurity; Google Cloud Cybersecurity; Google Project Management; Programming with Google Go (UC Irvine); Introduction to Cyber Security (NYU); Advanced Network Security (LearnQuest); Generative AI SQL Database Specialist with ChatGPT (Vanderbilt); AWS Academy Graduate—Cloud Foundations; AWS Solutions Architecture Job Simulation (Forage); Prompt Engineering (IBM); and a LinkedIn Learning “Microsoft Security Essentials (+14 more...)” bundle. The source has verification URLs and IDs for 8 entries; AWS has no verifier/ID, and Forage, IBM, and LinkedIn Learning lack date/ID/verification values. Credential IDs are withheld here. The separate claim of 77 verified credentials is a summary only; the other individual credentials and their links are not in these three files.
- **Awards/recognition (5 entries):** Code Red Hackathon team leader with an unspecified “placement-level result”; Meta Mind Hackathon 2025 participant; academic standing/grades as above; Axlerator certificate citation; and heart-disease model/MCP-paper accomplishments. The award text does not give a specific Code Red placement/rank or underlying certificates/paper records.
- **Matching job titles (13):** Backend Developer; Backend Engineer; Python Developer; Software Engineer (Backend); API Developer; Python Automation Engineer; Full Stack Developer (entry); Go Developer (junior); Cybersecurity Analyst (entry); SOC Analyst Tier 1; Cloud Security Analyst (entry); Information Security Analyst; Junior Project Coordinator.
- **Other explicit application information:** one-to-two references are listed as to be collected; Grozoom proof/dates and official CGPA cross-check are open; school/board marks and JEE scores are explicitly unknown/open. No other prior employment, publication venue, or formal job history is established in these three files.

## 5. Claim and evidence inventory

Evidence links/references in this table are only those represented in the JSON. A source-code path, certificate filename, paper, or URL was not opened during this audit.

| Claim / fact and source path | Claim form; supplied evidence reference | Evidence status | Employer-facing risk / minimum resolution |
|---|---|---|---|
| Candidate name `Chirag Gupta`; certificate name `Chirag`. `full-profile.json` `/identity/name`, `/identity/name_on_certificates` | Explicit. Contact/profile fields at `/contact`. | Supported by supplied data; contact links not tested. | Use a consistent professional name; confirm the shortened certificate name refers to the same candidate and what name appears on official records. Keep DOB out of résumé/application materials unless specifically required through a secure form. |
| Degree, institution, expected graduation, 9.13/10 CGPA, semester SGPAs. `full-profile.json` `/education` | Explicit. No transcript link/file in scope; `/open_items/2` itself asks for a transcript cross-check. | Incomplete; externally verifiable but not yet verified. | Wrong degree title/date/grade is material. Confirm exact official qualification wording and transcript-approved CGPA/credits before asserting them. |
| English/Hindi fluency; 59 skills; 13 target roles. `full-profile.json` `/languages`, `/skills`, `/matching_job_titles` | Explicit self-description; no assessment, proficiency scale, or dates. | Supported by supplied data only. | Avoid implying expert proficiency or production experience for every listed tool. Confirm practical proficiency and recency for roles requiring it. |
| 12 headline credential entries and “77 verified credentials total.” `full-profile.json` `/certifications_headline`, `/certifications_note` | 8 headline records include verification URLs; 4 records have no verification URL; full course export is only referenced by name, not supplied. | Incomplete; 77-total claim unsupported by the itemized data here. | Do not state “77 verified” or list unspecified bundle courses until a complete issuer/title/date/link list is available. Confirm active verification links and the AWS badge record. |
| Axlerator employment, Jan–Jun 2026, backend role; latency 4–5s to ~1s; ~15% workflow speedup; 100+ products. `internships.json` `/internships/0`, `/experience_blocks/axlerator_xp1` | Explicit. Evidence array lists certificate and source paths; `repo_url` is null. | Supported by supplied data; underlying evidence not supplied/inspected. Metrics incomplete. | “Production,” baseline/query selection, 15% methodology, and sample/product scope need support. The 4–5s-to-~1s range implies roughly 75–80% reduction depending on baseline; the `~80%` bullet is only exact at the 5s end. Confirm with mentor/certificate and before/after measurements. |
| Cognifyz internship, Nov–Dec 2025; eight weeks; 13 programs; 9 missing values to 0; 6 chart types; ~40% preprocessing reduction. `internships.json` `/internships/1`, `/experience_blocks/cognifyz_xp2` | Explicit. Evidence array names code, report, dataset, and certificate files, none inspected; `repo_url` is null. | Supported by supplied data for the narrative; ~40% metric lacks a measurement detail in the structured record. Certificate date is ambiguous. | Confirm exact dates/date format and the 40% calculation (baseline, timing, task). Avoid saying “any website” or implying production-scale scraping without scope. |
| Grozoom security-automation internship and six-stage flow. `internships.json` `/internships/2` | Technical narrative explicit; source itself says dates TBC and points to an empty repo / other profile documents outside scope. Repo URL is present but unvisited. | Incomplete; technical claims not corroborated by artifacts in scope; date explicitly missing. | Do not list as a dated employment entry or claim successful independent verification until dates and employer/mentor proof are supplied. Clarify authorization, scope, and whether the portal was an authorized test/challenge environment before describing anti-bot bypass/telemetry work. |
| Elvara “internship” and live deal analyzer. `internships.json` `/internships/3` | Explicit type is `internship assignment (live tool, 2-4 hr brief)`; assignment/workflow JSON, screenshot, repo, dashboard, and production webhook are named. | Conflicting classification (role says intern; type says assignment); external links not checked. | Calling a short assignment employment may misstate work history. Confirm whether there was an employment relationship and use “assignment/project” if not. Review/remove or restrict the live webhook/dashboard before sharing. |
| Peblo as an internship project. `internships.json` `/internships/4` | Explicit project, 2026/9 commits; repo URL and internal file references present; no dates or employment proof. | Incomplete classification and dates; links unvisited. | Do not imply employment or duration without confirmation; distinguish project work from a job. Confirm whether it was assigned by an employer and who owned the work. |
| Two PrimeTrade Python Developer Intern records. `internships.json` `/internships/5`, `/internships/6` | Both explicit role strings and Jan 2026 dates; records describe sibling/earlier bots and refer to an assignment specification. Repo URLs are present. | Conflicting/unclear relationship; no confirmed employment dates or proof. | Could double-count one assignment as two internships. Confirm employer relationship, whether these are iterations of one assignment, and whether both belong under one role/project. |
| SmartMeet-AI: 16 intents/endpoints, 122 combined tests, caching improvement. `projects.json` `/flagships/0/metrics`, `/flagships/0/llm_brief`, `/flagships/0/xyz_bullets/4` | Explicit. Repo URL present; `live_url` null. Derivation: 103 backend + 19 frontend tests = 122. | Test/feature counts externally verifiable but not checked; speedup conflicting/underspecified. | One bullet says about 1s to ~1ms (approximately 1,000× by division); elsewhere the claim is “up to 50×,” with a mocked 0.5s call returning in <0.01s (at least 50× in that mock). These are different scenarios/claims, not a reconciled benchmark. State the tested operation, environment, and measurement or omit the multiplier. |
| Type-Master: live platform, 40 endpoints, 13 models, curriculum, user impact. `projects.json` `/flagships/1`; `full-profile.json` `/bios/linkedin_about`, `/bios/video_intro_script_45s` | Explicit; repo and live URL fields present but unvisited. Curriculum conflict: project says about 260–300; profile bios say 200. | Conflicting lesson count; other metrics externally verifiable but not verified. | Confirm the count and whether it means lessons currently included versus planned. Do not state user adoption/“real users” as a measurable outcome without user/usage evidence. |
| MCP-Website research and AI integration. `projects.json` `/flagships/2` | Explicit; repo/live URL and bundled-paper path listed. One bullet says Gemini backend was “designed on paper”; architecture also calls it a documented proxy design. | Site/research claims externally verifiable but not checked; actual Gemini integration is explicitly not established as shipped. | Describe the AI proxy as documented/designed, not integrated/deployed. Confirm paper authorship, sources, and any research claims before saying “original research.” |
| TaskLogin-Manager / TaskFlow API. `projects.json` `/flagships/3`, `/more_projects/0`, `/universal_paragraph` | Two explicit project records; different stacks (SQLite vs PostgreSQL), counts (9 vs 10 endpoints), and names. Repositories are referenced separately. | Conflicting/ambiguous project identity; links not checked. | Do not merge metrics or describe as one project until the candidate confirms whether these are separate repositories, versions, or aliases. |
| Heart-disease model: AUC 0.9458, recall 0.9012, precision 0.7039, F1 0.79; profile says 0.946 AUC. `projects.json` `/more_projects/6/metrics`; `full-profile.json` `/awards_recognitions/4` | Explicit metrics; profile value is 0.9458 rounded to three decimal places (0.946). Repo/paper/showcase are referenced, not inspected. | Supplied-data values are internally consistent for AUC rounding; externally reproducible but unverified. | Present only as a project evaluation on the stated dataset/split. Do not imply diagnosis, clinical deployment, clinical validation, or patient outcomes; the project narrative itself lists further external validation/governance needs. |
| Other project claims (10 additional records). `projects.json` `/more_projects` | Explicit project/stack/bullet descriptions; some repo fields are present, some are non-URLs or absent; most lack dates and role/evidence details. | Supported by supplied data only; most claims not independently verifiable from this audit. | Keep only projects with a specific, explainable contribution. Confirm dates, repo visibility, ownership, and whether “sole builder” applies before using. |

## 6. Conflicts, gaps, and risk assessment

### Priority 1 — resolve before relying on affected claims

| Issue | Sources and exact conflict/gap | Risk and minimum clarification |
|---|---|---|
| Experience count and employment classification | `full-profile.json` bios/summary say 3 internships; `internships.json` `/meta/count` says 7 and array has seven records. Elvara is explicitly a 2–4-hour assignment; Peblo is an internship project; both PrimeTrade items are assignment/sibling-style projects with no formal employment period shown. | Misrepresenting assignments or duplicate projects as employment is a high factual risk. Confirm which records were actual paid/formal internships, the employer, title, and dates; keep project/assignment entries separate otherwise. |
| Missing Grozoom dates and supporting proof | `internships.json` `/internships/2/dates` says TBC; `/open_items/1` asks for dates/letter. | Background-check and chronology risk. Obtain exact dates and employer/mentor confirmation, or omit the role from employment history until established. |
| Education title/record | `full-profile.json` gives “Bachelor in computer science,” June 2027, and 9.13/10; `/open_items/2` asks for transcript cross-check. | Degree-name or grade mismatch can undermine applications. Confirm the official enrolled degree title, expected completion date, transcript marks/credits, and whether CGPA can be shared. |
| Work-authorization answer | `full-profile.json` `/preferences/work_auth` combines India authorization with an abroad-remote contractor statement. | A universal answer may be wrong for specific country, employee/contractor status, or application wording. Confirm country-specific eligibility/sponsorship facts before each form. |
| Credential inventory and identity name | `/certifications_note` says 77 verified credentials, but only 12 headline entries appear; several lack dates/IDs/verification. `/identity/name_on_certificates` is shorter than the professional name. | Unsupported credential count or verification/name mismatch can be checked by employers. Supply itemized records/links and confirm how to match the certificate name to the candidate. |

### Priority 2 — reconcile before using the affected metrics or project bullets

- **TypeMaster lessons:** 200 in `full-profile.json` `/bios/linkedin_about` and `/bios/video_intro_script_45s`; about 260–300 in `projects.json` `/flagships/1/metrics/curriculum` and bullets. Confirm actual published/complete count and use one definition.
- **SmartMeet cache speedup:** about 1s→~1ms (approximately 1,000× by arithmetic) versus “up to 50×” and a mocked 0.5s→<0.01s test. Identify the exact measurement, baseline, operation, and whether results are mocked or observed; otherwise omit the multiplier.
- **Axlerator latency reduction:** 4–5s→~1s corresponds to roughly 75–80%, but one bullet calls it about 80%. State exact samples/baseline or retain only the before/after range. The ~15% workflow improvement has no method stated in the structured metrics.
- **Cognifyz preprocessing:** `experience_blocks/cognifyz_xp2` says ~40% reduction; the structured record instead gives missing values 9→0 and does not identify how the 40% was measured. Confirm or omit that percentage.
- **TaskFlow naming/versions:** `TaskLogin-Manager (TaskFlow API)` has SQLite and 9 endpoints; `TaskFlow-API` has PostgreSQL/React and 10 endpoints. Confirm whether separate projects, versions, or aliasing; do not merge.
- **SmartMeet endpoint count:** metrics say 16 endpoints, while the evidence description says README has a 14-row API table. These might count different things; reconcile endpoint inventory and docs before using one number.
- **Cognifyz completion date:** `04/12/2025` has no stated format; the internship range says Nov–Dec 2025. Confirm whether day/month/year or month/day/year and use an unambiguous date.
- **Hackathon result:** “placement-level result” for Code Red has no rank/placement specified. Confirm exact event, result, and team role or use only the team-leader claim.
- **MCP project scope:** Gemini integration appears as a documented proxy design, not a shipped feature. Confirm paper authorship and avoid claiming deployment/integration unless separately supported.

### Priority 3 — incomplete data and presentation risks

- School/board, Class 10/12 results, and JEE details are not present; references are not collected. Include only if relevant to an application and after the candidate supplies exact details.
- Several records have no dates, roles, or evidence pointers (`more_projects`); `repo` values mix URLs, repository names, and descriptions. Many XYZ bullets contain absolute language (“zero,” “never,” “hacker-proof,” “any website”) that cannot be substantiated by the JSON alone. Prefer bounded, testable descriptions.
- No skill proficiency or years-of-experience scale exists. Listing an item does not establish a level or employment use.
- Claims of production use, real users, operational impact, and “verified” outcomes have no user counts or independently inspected evidence in scope.
- Do not normally include date of birth, salary expectation, private references, internal IDs, completion-flag value, vehicle-registration examples, or a live webhook endpoint in a public résumé. Contact details should be included only intentionally; the exact email and phone are masked in this report.
- The source files contain DOB and direct contact data, plus an operational webhook URL in the Elvara record. Keep those fields out of general-purpose/public résumé generation unless the candidate explicitly requests them and the destination is appropriate.

## 7. Recommended candidate-data contract

Recommendation only; no schema or source file was created or changed.

1. **Category ownership:** use `full-profile.json` for identity, contact, education, skills, credentials, preferences, availability, languages, and profile-level summaries; `internships.json` for experience/assignment records and their technical details; `projects.json` for project architecture, stacks, metrics, and project evidence references.
2. **No silent precedence:** when the same claim differs across files or records, retain each source claim separately and mark it `conflicting` until the candidate resolves it. Do not let a narrative bio override a structured record without an explicit decision.
3. **Trace every generated claim:** store the source filename, JSON Pointer, source-file SHA-256 snapshot, and any supporting link/reference for each internal claim. Preserve exact source wording for titles, dates, names, and metrics. For derived claims, store the formula, source pointers, rounding rule, and assumptions (for example, `103 + 19 = 122 tests`).
4. **Separate status dimensions:** distinguish `missing` (no key), `null` (key present with null), `unknown`, `conflicting`, `unverified`, and `verified against supplied artifact`. A URL/reference alone is not verified; verification requires actually reviewing the artifact, recording what was checked, and when.
5. **Keep provenance internally:** generated résumé bullets and application answers should have an internal claim-to-source map; do not expose JSON pointers to employers unless useful. Do not generate a factual number or achievement when its source status is incomplete/conflicting.
6. **Separate private and public data:** isolate DOB, direct contact, salary preference, work-authorization details, reference information, IDs, flags, and operational URLs from public project/ résumé content. Add contact fields only to an approved document; exclude DOB by default. Require explicit review before sharing links that lead to production tools or dashboards.
7. **Detect source changes read-only:** on each run, hash the three raw files and compare with the stored baseline; parse with UTF-8 BOM support; report changed-file hashes and a semantic JSON diff. Keep source files immutable. A hash change flags a new version but does not itself establish that a changed claim is true.

## 8. Prioritized clarification checklist

### 1. Blocking — before submitting materials that rely on these facts

1. Which of the seven records in `internships.json` were actual formal employment/internships, and which were assignments or personal/portfolio projects? In particular, what is the correct classification and date range for Elvara, Peblo, and PrimeTrade? Are the two PrimeTrade bots one assignment/role or two separate engagements?
2. What were the exact Grozoom start/end dates, and what employer-issued or mentor-issued evidence can support the role and completion? Was the security automation performed in an explicitly authorized test/challenge environment, and what wording accurately describes that scope?
3. What is the exact official degree title, expected graduation date, and transcript-approved CGPA/credits? Can the transcript or official academic record be used to validate 9.13/10?
4. Does the `Chirag` certificate-name entry refer to the same candidate as `Chirag Gupta`, and what name should be used consistently for credential checks?
5. For each target country and role type, what is the accurate work-authorization/sponsorship answer? The current broad statement should not be reused as a universal application response.

These are blocking for including the affected employment, education, credential, or eligibility claims. An evidence-limited draft can still omit unresolved items.

### 2. Important — materially improves matching and claim quality

1. Is TypeMaster’s correct completed curriculum count 200 or about 260–300, and what exactly is being counted?
2. What SmartMeet performance test supports the cache improvement, and which figure is correct for that specific test? Are the reported tests mocked or measured on a running service?
3. What baseline/sample supports Axlerator’s ~15% workflow improvement and Cognifyz’s ~40% preprocessing reduction? Can the latency range be stated with a measured starting value?
4. Are `TaskLogin-Manager (TaskFlow API)` and `TaskFlow-API` separate projects, versions, or aliases? Which endpoint counts and stack belong to each?
5. What does Cognifyz certificate date `04/12/2025` mean in an unambiguous format? What exact Code Red placement/rank, if any, was earned?
6. Can the 77-credential total be itemized with exact titles, issuers, dates, and active verification references? Which credentials should be prioritized for a résumé?
7. What is the correct endpoint count for SmartMeet (16 route/endpoints versus the README’s 14-row API table)? Was the MCP paper individually authored, co-authored, or adapted, and which Gemini features actually shipped?
8. Which project repositories are public/current, and are any of the project live URLs or operational endpoints meant to be shared with employers?

### 3. Optional — useful later, not required for a first MVP

1. What proficiency/recency levels should be attached to the skills list, and which tools have been used in internships versus only personal projects/coursework?
2. Are school/board results, JEE results, and one or two references available and relevant to target applications?
3. Are there preferred job titles, industries, cities, or role exclusions beyond the current broad list and relocation preferences?
4. Which of the 14 projects should be treated as portfolio highlights for backend, data/ML, security, and full-stack applications?

## 9. Read-only compliance confirmation

- Confirmed working directory was `D:\document\job apply`.
- The three source files were read and hashed; no source file or application code was modified, renamed, reformatted, moved, overwritten, or deleted.
- No internet, external service, credentials, or unrelated directory was accessed. No package was installed and Git was not initialized.
- This report is the only new deliverable created for Task 01: `reports/01_candidate_source_audit.md`.
- **Limitations:** underlying referenced documents, local paths, repositories, verification URLs, and live services were outside scope and were not inspected. Therefore all claim verification is limited to what the three JSON files state. The source-file hashes should be recomputed after any later edits; no comparison with earlier baselines was made.
