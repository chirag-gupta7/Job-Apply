# Task 17 — Manual Job-Link Intake and Side-Effect Audit

**Date:** 2026-10-29  
**Outcome: PASS WITH LIMITATIONS**  
**Repository:** `D:\document\job apply\tools\career-ops`  
**Private data root:** `%USERPROFILE%\career-ops-data` (expanded path omitted)

## 1. Executive summary

There is a genuinely local-only way to **record** one candidate-supplied URL: manually add a Pending line to the existing `data/pipeline.md` URL inbox. This does not fetch the URL, invoke a model, open a browser, or touch the application tracker. The pipeline file is currently absent, so this would require creating its documented Markdown skeleton first. It is a persistent lead queue (including a Processed section), not the application tracker and not an ephemeral scratch file. No dedicated CLI command for adding one arbitrary URL plus notes without other work was found.

There is **no local-only job evaluator** for pasted text. Prompt-driven evaluation sends the JD to the configured model. A direct single-URL fetch exists (`fetch-jd.mjs`), but it makes a public ATS GET and outputs job text; `browser-extract.mjs` may launch Playwright for unsupported hosts. Both require separate network authorization. Neither was run.

**Recommended safe intake:** after a separate authorization to record a lead, manually add one line to the existing pipeline inbox and leave it pending. Do not run `/career-ops pipeline`, `auto-pipeline`, `fetch-jd`, `browser-extract`, or an archive command unless their individual side effects are explicitly approved.

## 2. Preflight integrity

| Check | Result |
|---|---|
| Working directory | `D:\document\job apply` |
| Repository HEAD | `a6d7526c05861b1df9b70c011f8b1a84c44252ac` |
| Origin | `https://github.com/career-ops-hq/career-ops.git` |
| Tracked/full Git status before inspection | Clean |
| `package.json` SHA-256 | `10ca52815f65e4ba14ea4025828f2f9b212d1b684a2ab495a173d3a461df1fac` |
| `package-lock.json` SHA-256 | `cb14c2cd24a3676514af291914db76ace250a8bbf2d6708006f02300028c22cc` |
| Private workspace and Task 13 scaffold | Present; ACL rechecked against the restricted policy without logging usernames/SIDs |
| `data/pipeline.md` | **Absent** |
| `data/applications.md` | Present; not read or modified |
| `config/plugins.yml` | Absent |

The candidate JSON files were not read. The existing application tracker remains the sole application tracker. No application record was created.

## 3. Actual individual-URL path

### URL recording: local, but manual

Evidence: `DATA_CONTRACT.md` identifies `data/pipeline.md` as the user’s URL inbox; `modes/pipeline.md` explicitly describes the user adding URLs there and accepts a bare `- [ ] {url}` line. It documents optional company, role, location, compensation and labeled `note:` segments. The current inbox file does not exist.

`scan.mjs` exports `appendToPipeline(offers)` and formats Pending lines, but this is an internal writer used by scanners/ingest paths—not a CLI for a user to add one arbitrary URL. The source call sites found are scanners and `plugins.mjs`; no standalone local-only “add URL” command was found. `appendToPipeline()` can create the standard skeleton, but invoking a caller that discovers offers is not a manual local intake method.

### Minimal manual input format (proposal only)

If separately authorized, a user could create the documented file and add one line:

```markdown
# Pipeline — Pending URLs

## Pending
- [ ] https://public.example/jobs/123 | Example Co | Backend Engineer | note: candidate-selected; eligibility unverified

## Processed
```

The URL is the only required field. Company, role and the one-line `note:` are optional; use `|` only as a column separator. This stores a **candidate lead** under `%USERPROFILE%\career-ops-data\data\pipeline.md`; it does not create an application, change `applications.md`, or fetch the page. Do not place secrets or sensitive personal notes there. The queue persists processed lines, so it is not ephemeral, but it remains distinct from the canonical application tracker.

One caveat: `openrouter-runner.mjs`’s `readPipeline()` extracts only URL, company and role from the first three positional cells; it does not forward a labeled `note:` to its model prompt. The agent-facing pipeline mode can inspect the line, but then proceeds into the external workflow below. Do not rely on candidate notes being honored by every consumer.

### Pasted job text

`modes/auto-pipeline.md` accepts pasted JD text without fetching the posting URL, but its next step is an A–G model evaluation and artifact generation. `openrouter-runner.mjs evaluate` also accepts pasted text interactively, then calls OpenRouter. Neither is a local-only text-processing path. A user could manually save copied text as a local file and use a `local:jds/{filename}` reference (documented by `modes/pipeline.md`), but that is archival storage only; subsequent evaluation still sends the text to a model. No dedicated local-only text-import/evaluation command was found.

## 4. Command and workflow side-effect matrix

| Entry point | Reads/writes | URL/network, redirects and browser | Model/data egress | Tracker, CV, application or email effects |
|---|---|---|---|---|
| **Manual edit of `data/pipeline.md`** | Human writes a Pending line to the local URL inbox. No script is needed. | None unless a later workflow is invoked. | None. | Does not touch `data/applications.md`, generate a CV, or create an application record. This is the only path suitable for local-only URL recording. |
| `node archive-posting.mjs --dry-run <url>` | Computes a proposed filename/reference from the URL; writes nothing. | No navigation/browser/network on the dry-run branch. | None. | Does not add a pipeline lead; therefore a local preview, not an intake method. |
| `node fetch-jd.mjs <url>` | `fetchJdViaKnownApi()` returns JD text on stdout (20,000-character cap); no file write in this CLI. | Makes a GET to a known ATS public API; redirect is rejected by the ATS fetchers. Unknown/unsupported hosts return a miss. **Ashby is org-level:** it fetches the entire board response and selects the requested job ID, so it may retrieve other descriptions too. No browser in this command. | No model call by the script. If stdout is passed to an AI tool, that is a separate egress. | No tracker, report, CV, or application record write. This is a one-URL *network fetch*, not local-only intake. |
| `node browser-extract.mjs <url>` | Emits `{url,title,text}` JSON to stdout; no persistent write. | For known ATS URLs it tries their JSON API first; otherwise/inconclusively it launches headless Playwright. The browser context filters private/invalid destinations and checks the settled URL; public redirects and page subresources may be fetched. | No model call by the script. | No tracker or artifact write. Network/browser authorization required. |
| `node check-liveness.mjs <url>` / pipeline liveness sweep | Reads URL(s); by default can append expired verdicts to `data/scan-history.tsv`. `--no-record` suppresses that write only. `--discard-expired --apply` can update tracker rows through `set-status.mjs`. | ATS API check first; Playwright fallback for inconclusive URLs, with optional headed fallback. Network request; not an intake command. | No model call. | May write scan history; with `--discard-expired --apply`, may change canonical tracker status. |
| `node archive-posting.mjs <url>` / `--pipeline` | Reads a supplied URL or all pending URLs; saves a rendered PDF under `jds/`. `--pipeline` archives every pending inbox URL. | Launches Chromium, navigates/follows public redirects, and routes every request through private-address/DNS egress guards. This is network/browser access. | No model call. | Persists a full-page posting capture (PDF); does not itself create a tracker row or submit an application. `tests/archive-posting-non-content.test.mjs` uses a fake browser to check login/challenge refusal and a real-posting control; `tests/archive-posting-collision.test.mjs` uses a fake browser and temporary files to test writes. Those tests were inspected, not run. |
| `/career-ops pipeline` mode | Reads Pending entries; can mark them Processed, write reports and CV artifacts, and create tracker-addition TSVs for later merge. Liveness/closed/discard paths can update pipeline/history/tracker state. | Runs a liveness sweep and per-URL extraction, normally Playwright then WebFetch/WebSearch fallbacks. One URL stays one URL; it does not activate the company-wide ATS feed, but it is not a local-only operation. | Sends JD and candidate context to the AI CLI/model used for the session. | Full evaluation and CV/PDF path; tracker additions may later be merged into the sole canonical tracker. Does not auto-submit. |
| `/career-ops auto-pipeline` or a pasted URL in the agent | The mode writes a report, CV output as configured, and tracker-addition data; it archives the JD verbatim in the report or a `jds/` capture. | Fetches the one supplied URL via Playwright/WebFetch/WebSearch according to mode rules; redirects/page content are part of the workflow. | Sends the JD and candidate context to the active AI model/provider; exact endpoint depends on the host CLI. | May generate CV/PDF and draft application answers; no automatic submission or email send. |
| `node openrouter-runner.mjs evaluate <url>` | Reads `cv.md`, `config/profile.yml`, modes and the URL; writes `reports/*.md` containing archived JD text and `batch/tracker-additions/*.tsv`. It does not directly merge `applications.md`. | Fetches the URL using Playwright when available, otherwise plain `fetch`; those clients follow their normal redirects. It checks the settled page for liveness/readability, but this fetch path does not install the per-request egress route guard used by `browser-extract.mjs`/`archive-posting.mjs`. | Explicit external egress: OpenRouter `GET /models` when no model is pinned, then `POST /chat/completions`; the prompt includes the JD **and** CV/profile/system context. | Writes report and tracker-addition TSV. No CV PDF in this command; no application submission/email. `openrouter-runner pipeline` additionally processes all Pending rows and marks successful entries done. |
| `node rank-pipeline.mjs` | Reads Pending URLs, company/title and up to 2,000 characters of `cv.md`; writes `rank:` annotations into `data/pipeline.md` unless `--dry-run`. | Does not open/fetch job URLs; CLI is launched in a temporary empty directory with tool restrictions. | Calls the installed agent CLI/model with CV excerpt and posting metadata; model egress is still possible. A `--dry-run` still performs the model call before suppressing writes. | Annotates the inbox, not the application tracker; not a no-model intake path. |
| `node generate-pdf.mjs` / PDF mode | Reads a prepared HTML artifact and local profile/CV inputs; writes a PDF and `data/pdf-index.tsv`. | Launches Chromium for local `file://` rendering. The renderer disables JavaScript and aborts non-`file:`/`data:` requests. | No model call in this renderer. | Creates a document artifact, not a tracker record or submission. It is downstream of preparation, not intake. |
| `/career-ops apply` mode | Reads the matched report/profile and may persist an `Application Answers` section in the report. | Launches a fresh Playwright browser to read/fill the employer form; external page requests occur. | Uses the active agent model for answer drafting; endpoint depends on the host CLI. | Can prepare/fill answers and persist them, but the mode explicitly stops before Submit/Send/Apply. No auto-submit was found. |
| `node openrouter-runner.mjs apply <report>` | Reads the report plus its context files; prints generated answers to stdout. | No browser/navigation in this command. | Sends the report/JD and system context (including CV/profile) to OpenRouter; may also fetch its model list if no model is pinned. | Draft output only in the direct runner; no form submission, email, or answer-file write in `cmdApply()`. |
| `/career-ops email` | Reads report/profile context and creates a draft for review. | No external send needed for drafting. | Uses the active model/agent. | `modes/email.md` explicitly says never submit/send/click send; draft only. |

### Observed implementation versus tests

- The source-confirmed pipeline format is in `modes/pipeline.md`; `scan.mjs` defines the same skeleton and `appendToPipeline()` writer. That writer is called by scanner/ingest code, not by a dedicated “add one manual URL” CLI.
- `openrouter-runner.readPipeline()` and `cmdPipeline()` implement a separate direct-runner consumer; `cmdEvaluate()` fetches, calls the model, saves a report and TSV, and `cmdPipeline()` marks the URL processed. The runner’s `buildSystemPrompt()` includes CV and profile data.
- `browser-extract.mjs` tests cover normalizers and stub `fetch`; the test header states the live Playwright navigation path is not exercised there. The archive tests use fake browser objects. `tests/pipeline-output-format-parity.test.mjs` checks mode prose, not execution. These tests were inspected but **not run**.
- No conclusion in this report is presented as live runtime validation. No workflow was executed.

## 5. Safest available manual-intake path

**Option 1 is available:** after separate authorization to record a lead, manually create the documented skeleton at `%USERPROFILE%\career-ops-data\data\pipeline.md` and add one Pending URL line. Example only:

```markdown
# Pipeline — Pending URLs

## Pending
- [ ] https://public.example/jobs/123 | Example Co | Backend Engineer | note: candidate-selected; eligibility unverified

## Processed
```

The URL is the only required input. Company, role and a short labeled `note:` are optional. The output is the private workspace’s URL inbox; it is **not** an application record. The existing `data/applications.md` remains the one canonical application tracker. The pipeline inbox persists its Pending/Processed history, so although it is separate from the tracker it is not an ephemeral scratch file. Avoid pipes/newlines inside a note because `|` separates fields.

**Current state:** `data/pipeline.md` is absent, so there is no existing line to edit and no URL has been recorded. Creating the standard skeleton plus a user-supplied Pending line later would be a local file edit only. Do not run a scanner to create it; scanners can fetch enabled boards and write several state files.

**Option 2 is not a local-only evaluation path.** Pasted JD text can be used by `auto-pipeline` or `openrouter-runner evaluate`, but those call a model. A `local:jds/{filename}` reference is documented for already-local text; it avoids fetching the employer page but does not avoid model egress when the evaluation mode runs. No local deterministic evaluator for arbitrary pasted JD text was found.

**Option 3 exists, but needs explicit per-URL network approval.** For a known ATS, `fetch-jd.mjs <url>` is the narrowest existing fetch CLI: API GET, no browser, no persistent write, JD text to stdout, and no model call. It is not fully narrow for Ashby because the Ashby API is org-level and the fetcher loads the board before selecting the one job. For unknown ATS sites, `browser-extract.mjs` may launch Playwright. Neither command was executed.

No implementation is required for **manual recording**; the existing inbox format suffices. If a dedicated command is desired later, the smallest addition would be a local-only single-lead appender that validates a URL string without resolving/fetching it and appends under the existing pipeline lock. It should write only `data/pipeline.md`, never `applications.md`, never fetch, and never call a model. This task did not implement it.

## 6. Privacy and safety conclusion

- A URL-only Pending line is the only path found that meets the requested “record without fetching/model” condition; it requires a manual edit because the inbox file is absent and no local CLI appender exists.
- Running a pipeline or direct evaluation is **not** the same as recording: it can fetch the posting, send the JD and candidate context to an external model, write reports/PDFs/TSVs, and alter pipeline state. `fetch-jd`/`browser-extract` also make external requests even though they do not call a model themselves.
- Job posting content must be treated as untrusted data, never as instructions. No candidate JSON, application data or job source was read or changed in this audit.
- No application submission or email-send behavior was triggered. The standard apply mode says stop before Submit/Send/Apply; email mode is draft-only.

## 7. Final integrity and activity checks

- Repository HEAD/origin, manifests and clean status remain as in §2.
- Private-root ACL remains PASS; all four Task 13 scaffold files remain present; `data/pipeline.md` remains absent.
- `data/applications.md` was not edited; no tracker row/application record was created.
- No scanner, liveness check, archive/fetch command, browser, model, application preparation, email, or external service was invoked. No network request was made.
- No persistent environment variable, repository file, profile, portals configuration, manifest or lockfile was changed. The report is the only new deliverable.
