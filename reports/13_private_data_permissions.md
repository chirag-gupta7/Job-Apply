# Task 13 — Secure the Private Career-Operations Workspace

**Date:** 2026-10-29  
**Status: PRIVATE ROOT VERIFIED**  
**Private data root:** `%USERPROFILE%\career-ops-data` (resolved from the current process environment; the expanded path is omitted to avoid exposing the account name)  
**Repository:** `D:\document\job apply\tools\career-ops`

## 1. Executive summary and security decision

Created a new data root at `%USERPROFILE%\career-ops-data`, disabled inherited permissions on the root, and verified its DACL contains exactly three explicit Full Control entries: the current user's SID, `NT AUTHORITY\SYSTEM`, and the built-in local Administrators group. No `Everyone`, `Authenticated Users`, or general `Users` access entry is present. The four known non-personal scaffold files were copied, hash-compared, checked for private ACL inheritance, and then removed from the old non-private root. The old root and its empty Task 12 directories were safely removed.

**Security decision:** the destination is verified against the requested Windows ACL policy for future private workspace use. As required, no candidate information was imported and the application was not started. Local Administrators and SYSTEM retain access by design.

## 2. Preflight integrity results

| Check | Result |
|---|---|
| Working directory | `D:\document\job apply` — exact match |
| Task 12 report | Read; confirmed the old root contained only non-personal scaffold files and had broad inherited access |
| New destination before creation | `%USERPROFILE%\career-ops-data` did not exist |
| Old root contents | Exactly the four expected files and six documented directories; no unexpected files |
| Git HEAD | `a6d7526c05861b1df9b70c011f8b1a84c44252ac` |
| Origin | `https://github.com/career-ops-hq/career-ops.git` |
| Tracked working tree before report | Clean |
| `package.json` SHA-256 | `10ca52815f65e4ba14ea4025828f2f9b212d1b684a2ab495a173d3a461df1fac` |
| `package-lock.json` SHA-256 | `cb14c2cd24a3676514af291914db76ace250a8bbf2d6708006f02300028c22cc` |

Candidate source files were checked by hash only; their contents were not read:

| File | SHA-256 | Result |
|---|---|---|
| `full-profile.json` | `2f767145d8cb3bb619f0e77ff5f9f0e804a6026e1d2122e7b3fa50dcc9161fdb` | PASS |
| `internships.json` | `606e03d0d98d783a00c0afb002a08dc1958b065ea245a9b232fee8b0d5da2aea` | PASS |
| `projects.json` | `86749dc8d6f1aa9523e6a57f1decf3a74c911e18eb3ed3ee2000f16b8725e4f2` | PASS |

## 3. New private-root path and ACL verification

The root was created using the current process's `%USERPROFILE%` value; no username or SID is recorded here. Inheritance was disabled on the new root only. The ACL was read back and verified before any scaffold files were copied.

| ACL check | Result |
|---|---|
| Root DACL protected from parent inheritance | Yes |
| Explicit Full Control principals | Current-user SID; SYSTEM; built-in local Administrators |
| Explicit root access-rule count | 3 |
| Broad `Everyone` / `Authenticated Users` / general `Users` entries | None |
| Root rules inherit to children | Yes — container and object inheritance enabled |
| ACL read-back verification | PASS |

Each migrated file's read-back DACL contains exactly the same three allowed principals as inherited Full Control entries. All four file ACL checks passed. No parent, profile-wide, repository, candidate-directory, or system ACL was modified.

## 4. Scaffold migration and hashes

The original and destination SHA-256 values match exactly for every migrated file:

| Relative path under data root | Original SHA-256 | Destination SHA-256 | Result |
|---|---|---|---|
| `config\profile.yml` | `8f4b9250965c7cec9305d5bd15cd437555f96aa1c1bd16f793ebcde416ca6fb1` | `8f4b9250965c7cec9305d5bd15cd437555f96aa1c1bd16f793ebcde416ca6fb1` | PASS |
| `portals.yml` | `398a369c7ab91ee9df75bd98b6acff0ad28a02bbf88907f5c91a553b02da0fd3` | `398a369c7ab91ee9df75bd98b6acff0ad28a02bbf88907f5c91a553b02da0fd3` | PASS |
| `data\applications.md` | `85f999182de6efc6adc8b343f3062b3c4016cdaba81b333f05cfc1c42fa53d0b` | `85f999182de6efc6adc8b343f3062b3c4016cdaba81b333f05cfc1c42fa53d0b` | PASS |
| `data\claim-source-map.template.md` | `39a6fb94f611fb8853a7aca9713821a2f947d32338274869bfcc533898a177db` | `39a6fb94f611fb8853a7aca9713821a2f947d32338274869bfcc533898a177db` | PASS |

The copied text was reviewed as non-personal scaffold, and value-pattern checks found no credential values or contact email addresses. Hash equality confirms the role/title filters and blank tracker were not changed. Both YAML files parsed with the already-installed `js-yaml` package without importing project code. `search_queries`, `tracked_companies`, `job_boards`, and `skip_tiers` remain empty; title filters have no negative entries.

## 5. Old-root cleanup results

Only after all four destination hashes, file ACLs, content checks, and YAML checks passed:

- Deleted the four exact original scaffold files from `D:\document\career-ops-data`.
- Removed only the six known Task 12 directories after confirming each was empty: `config`, `data`, `documents`, `jds`, `output`, and `reports`.
- Removed the old root after confirming it was empty.
- Confirmed `D:\document\career-ops-data` no longer exists.
- No unexpected file or directory was deleted; no ACL on `D:\document` was changed.

## 6. Tracker and privacy boundary

Exactly one `applications.md` tracker exists in the active workspace: `%USERPROFILE%\career-ops-data\data\applications.md`. It is the unchanged blank Task 12 tracker. The old root is absent. No second tracker or database was created.

No candidate files were copied, moved, linked, or read for content. The new root contains only the four unchanged non-personal scaffold files. No candidate claims, résumé, application records, contact details, or credentials were added.

## 7. Final integrity and activity checks

- All three candidate source hashes still match the accepted baseline (§2).
- Repository HEAD, origin, `package.json`, and `package-lock.json` are unchanged.
- No tracked repository source, manifest, or configuration was modified; the report is the only new repository deliverable.
- All four migrated files and the root have the intended verified ACLs.
- No application command, scanner, model endpoint, job source, browser, or external service was invoked.
- No persistent environment variable or system setting was changed.

## 8. Files and permissions changed

- Created `%USERPROFILE%\career-ops-data` with protected ACL and explicit Full Control for the three required principals.
- Created `config` and `data` beneath the private root; migrated the four files listed in §4. Child access inherits from the protected root.
- Deleted the four exact old copies and removed the now-empty old scaffold directories/root.
- Created `reports\13_private_data_permissions.md`.
- No other files or permissions were changed.

## 9. Remaining risks and unverified assumptions

- The security verification is a local Windows DACL check. Administrators and SYSTEM can access the data by design; this does not protect against a compromised account, malware running as the user, or device loss.
- Future files should be created beneath this root so they inherit its ACL. Any file copied with an explicit source ACL or any manually changed ACL must be rechecked.
- No career-ops runtime command was run, so runtime path resolution has not yet been confirmed.
- No sources are approved or enabled; discovery remains inactive.

## 10. Recommended next task

For a later, explicitly authorized runtime-validation task, set `CAREER_OPS_ROOT=%USERPROFILE%\career-ops-data` **only for that process/session**, verify that the pinned path resolver selects this root, and run only a non-network local validation. Do not point the application at the candidate JSON directory or repository checkout, and do not enable a scan until sources are separately approved.
