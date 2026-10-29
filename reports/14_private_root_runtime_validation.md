# Task 14 — Private Root Runtime Validation

**Date:** 2026-10-29  
**Outcome: PASS WITH LIMITATIONS**  
**Expected private root:** `%USERPROFILE%\career-ops-data`

## Summary

The pinned checkout's actual resolver selected `%USERPROFILE%\career-ops-data` when `CAREER_OPS_ROOT` was set for a single Node process. The resolver's profile and tracker helpers returned paths in that root, and the root-relative portal and claim-map scaffold paths were present there. All runtime path assertions passed.

**Limitation:** this checkout does not select the private root automatically. With both root environment variables unset, the actual resolver selected the repository root: the checkout is not a plugin-install copy and no `.career-ops-data` marker exists. Set `CAREER_OPS_ROOT=%USERPROFILE%\career-ops-data` in each future process/session that should use the private workspace. No persistent environment setting was changed.

## Repository and workspace preflight

| Check | Result |
|---|---|
| Working directory | `D:\document\job apply` — exact match |
| Repository HEAD | `a6d7526c05861b1df9b70c011f8b1a84c44252ac` |
| Origin | `https://github.com/career-ops-hq/career-ops.git` |
| Tracked and full Git status before/after runtime check | Clean |
| Private root exists | Yes — expanded user-profile path intentionally omitted |
| `config\profile.yml` | Present; Task 13 hash unchanged |
| `portals.yml` | Present; Task 13 hash unchanged |
| `data\applications.md` | Present; Task 13 hash unchanged |
| `data\claim-source-map.template.md` | Present; Task 13 hash unchanged |
| Private-root ACL | PASS — checked against Task 13 policy without recording usernames or SIDs; no broad user-group access |
| `package.json` SHA-256 | `10ca52815f65e4ba14ea4025828f2f9b212d1b684a2ab495a173d3a461df1fac` — unchanged |
| `package-lock.json` SHA-256 | `cb14c2cd24a3676514af291914db76ace250a8bbf2d6708006f02300028c22cc` — unchanged |

## Resolver source and path-selection behavior

**Implementation:** `tools/career-ops/path-resolver.mjs`, `getCareerOpsRoot()` (lines 100–134). Its code root is the directory containing `path-resolver.mjs` (`__dirname`), not the process working directory.

Precedence, as implemented:

1. Non-empty `CAREER_OPS_ROOT`; otherwise non-empty `CAREER_OPS_DATA_DIR`. If both are non-empty, `CAREER_OPS_ROOT` wins. An absolute value remains absolute; a relative override is resolved against the code root, not the current working directory. Explicit roots inside a recognized Claude plugin-install copy are rejected with `UnsafeDataRootError`.
2. A non-empty `.career-ops-data` marker beside the resolver; an absolute marker path is used as given and a relative marker path is resolved against the code root. The marker is absent in this checkout.
3. For a Claude plugin-install copy only, `homedir()/career-ops-data`.
4. Otherwise, the repository/code root itself.

The `CAREER_OPS_ROOT` name and precedence are confirmed in source and in inspected tests: `tests/claude-plugin-install.test.mjs` (“CAREER_OPS_ROOT still wins over the plugin default”) and `tests/cli-tracker-path-resolution.test.mjs` (documents that it outranks `CAREER_OPS_DATA_DIR`). `tests/reconcile-pipeline-data-root.test.mjs` also covers both root variables and the marker through a pipeline entrypoint. These tests were inspected, not run; the broader files create fixtures, write files, or exercise workflow entrypoints.

### Paths beneath the selected root

- `getProfilePath(root)` → `{root}/config/profile.yml`. Passing `root` takes precedence over `CAREER_OPS_PROFILE`.
- `resolveTrackerPath(root)` reads `{root}/data/applications.md` when it exists, otherwise falls back to `{root}/applications.md`. `resolveTrackerPathForWrite(root)` uses `{root}/data/applications.md`. `CAREER_OPS_TRACKER` is a separate tracker-file override; it was cleared for this check.
- `portals.yml` is a User Layer file under the data root by default. Discovery entrypoints may have a separate `CAREER_OPS_PORTALS` override; no discovery entrypoint was run.
- The claim-source map is a Task 12 scaffold at `{root}/data/claim-source-map.template.md`, not a special path-resolver-managed file.
- `DATA_CONTRACT.md` defines the other User Layer directories, including `jds/`, `reports/`, `output/`, and `documents/`, relative to the chosen root. `getCareerOpsRoot()` itself does not create those directories or require their contents.

## Runtime validation

**Executed:** a narrow inline Node process from the specified working directory imported only `path-resolver.mjs`. Inside that child process, it set `CAREER_OPS_ROOT` to the expected profile-relative root and cleared `CAREER_OPS_DATA_DIR`, `CAREER_OPS_PROFILE`, and `CAREER_OPS_TRACKER`. It called the actual `getCareerOpsRoot()`, `getProfilePath(root)`, `resolveTrackerPath(root)`, and `resolveTrackerPathForWrite(root)` exports, then checked the root-relative portal and claim-map paths and all four file-existence checks. It printed pass/fail labels only; no expanded profile path was emitted.

| Runtime assertion | Result |
|---|---|
| Actual selected root equals `%USERPROFILE%\career-ops-data` | PASS |
| Profile path resolves to `config\profile.yml` inside that root | PASS |
| Tracker read path resolves to existing `data\applications.md` inside that root | PASS |
| Tracker write path resolves to `data\applications.md` inside that root | PASS |
| `portals.yml` and claim-map paths are inside the root | PASS |
| All four Task 13 scaffold files exist | PASS |

A second inline Node process cleared both root overrides and called the same actual resolver. It observed `isPluginInstallCopy() === false` and the repository root as the selected default, not the private root. This confirms the limitation above; the private root is used only when the per-process override (or a future approved marker) is supplied.

### Safety assessment

`path-resolver.mjs` imports only Node built-ins (`path`, `url`, `fs`, `os`). Importing it defines helpers; the invoked resolver performs path resolution, marker existence/read checks when applicable, and filesystem realpath checks for plugin-copy protection. It does not import an application workflow, create files, scan sources, make network requests, launch a browser, or call a model. The configured test's `CAREER_OPS_ROOT` existed, so the marker branch was not needed. The no-override check observed the marker absent.

## Privacy, activity, and integrity boundaries

- The private root and all four scaffold-file ACLs were checked; the report contains no expanded profile path, username, or SID.
- Candidate JSON content was not read, hashed, copied, moved, linked, or imported.
- No job source, scanner, application workflow, AI endpoint, browser, or external service was invoked; no network request was made.
- No temporary file was created, so no temporary-file cleanup was needed.
- The environment override was assigned only inside the child Node process. No persistent environment variable or system setting was changed.
- Final repository HEAD/origin and tracked working tree remained at the pinned clean state; manifests remained unchanged. No source, manifest, lockfile, application configuration, or scaffold file was edited.

## Remaining limitation / next task

The data root is verified when the real resolver runs with the process-local `CAREER_OPS_ROOT` override. A normal invocation without that override still defaults to the repository root in this checkout. Any later runtime task must explicitly set the override for its own process/session and must separately authorize any application workflow; this task did not enable discovery or validate scanner behavior.
