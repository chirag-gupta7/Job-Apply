# Task 19 — Manual Lead Recording

**Date:** 2026-10-29  
**Outcome: PASS**  
**Inbox:** `%USERPROFILE%\career-ops-data\data\pipeline.md` (expanded path omitted)

## 1. Repository and private-workspace preflight

| Check | Result |
|---|---|
| Working directory | `D:\document\job apply` |
| Repository HEAD | `a6d7526c05861b1df9b70c011f8b1a84c44252ac` |
| Origin | `https://github.com/career-ops-hq/career-ops.git` |
| Tracked/full Git status before and after | Clean |
| `package.json` SHA-256 | `10ca52815f65e4ba14ea4025828f2f9b212d1b684a2ab495a173d3a461df1fac` |
| `package-lock.json` SHA-256 | `cb14c2cd24a3676514af291914db76ace250a8bbf2d6708006f02300028c22cc` |
| Private root, `data` directory and inbox | Present |
| Root, `data` and inbox ACLs | PASS; root protected, child entries inherit the restricted policy; no broad user-group access |
| Inbox before recording | Present; expected Pending and Processed sections; SHA-256 `6b8e304a5a8b6fad469527752acb3acb639f9862da6c47f5a2aca11503fffecf` |

No username, expanded profile path, or SID is recorded.

## 2. Leads recorded

The inbox contained none of the three exact URLs before this task. Added all three beneath `## Pending`; each appears once and remains unchecked. The `## Processed` section and existing content were preserved.

| Newly recorded lead | Pending status |
|---|---|
| [Amazon — Software Development Engineer Intern - May 2027 (2 month)](https://www.amazon.jobs/en/jobs/10513277/software-development-engineer-intern-may-2027-2-month-amazon-university-talent-acquisition) | `- [ ]` |
| [Trinetro Labs — Founding Backend Engineering Intern](https://trinetrolabs.com/careers/founding-backend-engineer/) | `- [ ]` |
| [Valura.AI — Python Backend](https://careers.valura.ai/6da7deb6-ad9e-465f-8ff0-c5c5852d2696) | `- [ ]` |

The supplied notes were recorded verbatim. No URLs were fetched or opened, and no posting content was added.

## 3. Read-back and separation

The inbox was read back and matched the expected initial skeleton plus exactly the three requested lines. All three exact URLs occur once in the Pending section, before Processed.

`data\applications.md` remains present and unchanged; it remains the only application tracker. The other four Task 13 scaffold files retain their accepted hashes:

| Existing file | SHA-256 | Result |
|---|---|---|
| `config\profile.yml` | `8f4b9250965c7cec9305d5bd15cd437555f96aa1c1bd16f793ebcde416ca6fb1` | PASS |
| `portals.yml` | `398a369c7ab91ee9df75bd98b6acff0ad28a02bbf88907f5c91a553b02da0fd3` | PASS |
| `data\applications.md` | `85f999182de6efc6adc8b343f3062b3c4016cdaba81b333f05cfc1c42fa53d0b` | PASS |
| `data\claim-source-map.template.md` | `39a6fb94f611fb8853a7aca9713811a2f947d32338274869bfcc533898a177db` | PASS |

**Inbox SHA-256 after recording:** `992e2629cab0c9b35482937f034a61a752b8a575edec22d080ac6044f038353d`.

## 4. Activity and final integrity

- No job URL was fetched or opened; no network, browser, model, scanner, validator, pipeline, evaluator, archive, or application-preparation workflow ran.
- No application record, résumé, PDF, email, or application draft was created.
- Candidate JSON contents were not read. No environment/system settings were changed.
- No file was deleted. The only data-root change was the three requested Pending lines in `data\pipeline.md`; the only report created is this file.
- Final repository pin/origin, clean Git status, manifest hashes, existing scaffold hashes and private ACLs passed recheck.

**Unresolved issues:** none for recording the leads. They remain Pending and unevaluated; any later fetching or model use requires separate authorization.
