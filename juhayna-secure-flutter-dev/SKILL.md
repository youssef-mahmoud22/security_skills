---
name: juhayna-secure-flutter-dev
description: Use when writing, modifying or reviewing Flutter or Dart code for Juhayna applications (Android, iOS, Flutter web), especially login and tokens, secure storage, HTTP/Dio calls, TLS and certificate pinning, WebViews, deep links, local databases, logging, permissions, app release builds or pub packages.
---

# Juhayna Secure Flutter Development

## Overview
Juhayna Secure Coding Standard (IT-01 v1.0) is mandatory. A MUST deviation needs an approved exception: a GitHub issue labeled `security-exception` with policy ID, risk, compensating controls, owner and expiry (12 months max). A SHOULD deviation is explained in the merge request. The Security Reviewer evaluates every release; nothing ships without approval (SCS-TEST-04).

## When to use
Any Flutter/Dart change touching data, identity, network, device storage, builds or dependencies. Skip only for pure styling with no data flow.

## Workflow
1. Read `references/index.md`.
2. Map the task to domains (table below).
3. Open only those `references/<CODE>.md` files; apply the "Implementation (Flutter)" lines.
4. Check `references/prohibited.md`. Every item is a hard stop.

## Quick reference
| Task | Domains and key cards |
|---|---|
| Login, tokens, sessions, logout | AUTH (AUTH-01, AUTH-07), MOB-01 |
| Biometrics, local auth | MOB-05 |
| Showing or editing a record by ID, roles | AUTHZ-01 (server enforces; UI hiding is not security) |
| Calling an API, Dio/http | API-07, CRY-01, MOB-04 (pinning), INP-01 |
| Storing data on device | MOB-01, MOB-02, DATA-01, DATA-02 |
| Screenshots, clipboard, keyboard | MOB-03 |
| WebView, deep links, platform channels | MOB-06, AUTH-01 (system browser for login) |
| Release build, flavors, signing, obfuscation | MOB-07, MOB-08, DEP-03, SEC-01 |
| Keys, config, secrets | SEC-01, SEC-03, CRY-03 |
| Crypto, hashing, randomness | CRY-02 |
| Logging, crash reporting, errors | LOG-02, ERR-01, ERR-02 |
| Input, URLs, paths, web output | INP-01, INP-03, INP-04 |
| File pick/upload | FILE-01 |
| Adding or updating a package | SUP-01, SUP-02, SUP-03 |
| New feature or design | GOV-02, GOV-03 |
| Merge and release | DEP-01, DEP-02, TEST-01, TEST-02 |

## Iron rule
Never weaken a control to make something work. Fix the cause: install a trusted certificate, move the key to the backend.
- No `badCertificateCallback` returning true.
- No secrets in `--dart-define`, bundled `.env`/asset files or source.
- No tokens in SharedPreferences; use platform secure storage (MOB-01).
- No client-side check as the only control.

If a trade-off is truly unavoidable: a Development-only build flag that release builds cannot reach, plus an exception.

## Common mistakes and rationalizations
| Excuse | Reality |
|---|---|
| "It's only for QA" | QA builds are release builds. |
| "It's obfuscated" | Strings and keys survive obfuscation. |
| "The API checks it anyway" | Verify that it does, then still validate. |
| "We'll add pinning later" | Pin to the CA key with a backup pin now (MOB-04). |
| "Just a debug log" | Logs leak tokens and personal data (LOG-02). |
| "Hide the button" | Server must refuse the call (AUTHZ-01). |

## Red flags
`badCertificateCallback`, `SharedPreferences` + token, `print(` of a response, `String.fromEnvironment` holding a key, `.env` in `pubspec.yaml` assets, `javascriptMode: unrestricted` with remote URLs, empty `catch`, unpinned `any` package versions. Stop and fix.

## Before done
- [ ] Appendix A items for touched domains (`references/checklist.md`).
- [ ] SCS-TEST-02 self-check; attach results and tool versions to the MR:
  - `gitleaks git -v`
  - `osv-scanner scan source -r .`
  - `flutter analyze` (or `dart analyze`) with flutter_lints and `avoid_print`; fix all warnings
  - `dart pub outdated`
  - MobSF scan of the APK/IPA for High-risk app release builds
- [ ] MR description cites the SCS IDs addressed.
- [ ] Security tests written for the change (SCS-TEST-01 scenarios).
