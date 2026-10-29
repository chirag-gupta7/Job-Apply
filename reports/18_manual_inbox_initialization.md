# Task 18 — Manual Inbox Initialization

**Date:** 2026-10-29  
**Outcome: PASS**  
**Private workspace:** `%USERPROFILE%\career-ops-data` (expanded path intentionally omitted)

## 1. Preflight and repository integrity

| Check | Result |
|---|---|
| Working directory | `D:\document\job apply` |
| Repository HEAD | `a6d7526c05861b1df9b70c011f8b1a84c44252ac` |
| Origin | `https://github.com/career-ops-hq/career-ops.git` |
| Tracked/full Git status before creation | Clean |
| `package.json` SHA-256 | `10ca52815f65e4ba14ea4025828f2f9b212d1b684a2ab495a173d3a461df1fac` |
| `package-lock.json` SHA-256 | `cb14c2cd24a3676514af291914db76ace250a8bbf2d6708006f02300028c22cc` |
| Private root and `data` directory | Present |
| `data\pipeline.md` before creation | Absent |
| Report before creation | Absent |

The private root DACL was protected and matched the Task 13 allowed-principal policy. The `data` directory inherited only the restricted root entries. No broad `Everyone`, `Authenticated Users`, or general `Users` entry was found. The new file’s read-back ACL contains exactly the same three inherited Full Control entries; no ACL was modified.

## 2. Inbox created and verified

Created only `%USERPROFILE%\career-ops-data\data\pipeline.md` using UTF-8 without a BOM. Read-back matched the requested skeleton exactly:

```markdown
# Pipeline — Pending URLs

## Pending

## Processed
```

The file is an empty lead inbox: it contains no example URL, employer, job, contact detail, credential, candidate claim, or application status.

## 3. Separation and existing scaffold integrity

`data\applications.md` still exists and was not edited. It remains the sole application tracker; `data\pipeline.md` is the separate lead inbox. The other Task 13 scaffold files were hash-checked and remain unchanged:

| Existing file | SHA-256 | Result |
|---|---|---|
| `config\profile.yml` | `8f4b9250965c7cec9305d5bd15cd437555f96aa1c1bd16f793ebcde416ca6fb1` | PASS |
| `portals.yml` | `398a369c7ab91ee9df75bd98b6acff0ad28a02bbf88907f5c91a553b02da0fd3` | PASS |
| `data\applications.md` | `85f999182de6efc6adc8b343f3062b3c4016cdaba81b333f05cfc1c42fa53d0b` | PASS |
| `data\claim-source-map.template.md` | `39a6fb94f611fb8853a7aca9713821a2f947d32338274869bfcc533898a177db` | PASS |

Candidate JSON contents were not read.

## 4. Activity, limitations and final checks

- No URL was added and no application record was created.
- No network request, browser, model, scanner, validator, fetcher, archive, pipeline, or application-preparation workflow was run.
- No environment variable, system setting, repository source, manifest, lockfile, profile, or portals configuration was changed.
- Final HEAD/origin, clean Git status, manifest hashes, private ACL, inbox content, and scaffold hashes were verified.

**Unresolved:** none for this initialization. The inbox is intentionally empty. Processing a future URL remains a separate action with network/model side effects and requires its own authorization.
