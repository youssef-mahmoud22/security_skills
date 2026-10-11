---
name: juhayna-secure-dotnet-dev
description: Use when writing, modifying or reviewing ASP.NET Core or C# code for Juhayna applications, especially new endpoints, EF Core, SQL Server queries, stored procedures or logins, SAP integration, JWT or Entra ID auth, CORS, cookies, security headers, antiforgery, file uploads, connection strings, appsettings secrets, outbound HTTP or NuGet packages.
---

# Juhayna Secure .NET Development

## Overview
Juhayna Secure Coding Standard (IT-01 v1.0) is mandatory. A MUST deviation needs an approved exception: a GitHub issue labeled `security-exception` with policy ID, risk, compensating controls, owner and expiry (12 months max). A SHOULD deviation is explained in the merge request. The Security Reviewer evaluates every release; nothing ships without approval (SCS-TEST-04).

## When to use
Any C#/SQL change touching endpoints, data access, identity, configuration, integrations, uploads or dependencies.

## Workflow
1. Read `references/index.md`.
2. Map the task to domains (table below).
3. Open only those `references/<CODE>.md` files; apply the "Implementation (.NET / SQL Server)" lines.
4. Check `references/prohibited.md`. Every item is a hard stop.

## Quick reference
| Task | Domains and key cards |
|---|---|
| New endpoint | API-01, API-02, API-03, AUTHZ-01, AUTHZ-02, INP-01, ERR-01 |
| Load or change a record by ID | AUTHZ-02 (ownership check) |
| Query, stored procedure, EF Core | DB-01, INP-02 |
| SQL login, encryption, backups | DB-02, DB-03, DB-04, DB-05 |
| JWT, Entra ID, passwords, sessions | AUTH-01, AUTH-02, AUTH-03, AUTH-07 |
| Cookies, CORS, headers, antiforgery, HTTPS | WEB-01..04, API-04 |
| Outbound HTTP, vendor, LLM, URL fetch | API-07, API-08, CRY-01 |
| SAP RFC/BAPI | API-06 |
| Config, connection strings, keys | SEC-01, SEC-02, DB-02, CRY-03 |
| Upload or serve files | FILE-01, FILE-02 |
| Deserialization, paths, commands | INP-05, INP-04 |
| Errors, logging, monitoring | ERR-01, ERR-02, LOG-01, LOG-02, LOG-03 |
| NuGet package | SUP-01, SUP-02, SUP-03 |
| New feature or design | GOV-02, GOV-03 |
| Merge and release | DEP-01, DEP-02, DEP-03, TEST-01, TEST-02 |

## Iron rule
Never weaken a control to make something work. Fix the cause: install the CA certificate, grant the least privilege needed, add the right claim. Refuse these:
- `TrustServerCertificate=True`
- `AllowAnyOrigin`, `SetIsOriginAllowed(_ => true)`, wildcard origin with credentials
- `[AllowAnonymous]` added to fix a 401
- `ValidateIssuer`/`ValidateAudience`/`ValidateLifetime = false`
- returning exception messages to clients
- binding or returning EF entities
- string-built SQL
- `sa` or `db_owner` connections

If a trade-off is unavoidable: Development-only, unreachable from release builds, plus an exception.

## Common mistakes and rationalizations
| Excuse | Reality |
|---|---|
| "It's internal only" | Internal users and stolen tokens attack too. |
| "The frontend validates it" | Attackers skip the frontend (INP-01). |
| "Works on my machine with sa" | Create a least-privilege login (DB-02). |
| "Just return ex.Message for debugging" | Log it server-side; return a generic error. |
| "Auth middleware is global" | Verify order in `Program.cs` and ownership per object. |
| "EF parameterizes everything" | Not `FromSqlRaw`/`ExecuteSqlRaw` with interpolation. |

## Red flags
`FromSqlRaw($"...")`, `[AllowAnonymous]` on data endpoints, `Id` taken from route with no owner check, secrets in `appsettings.json`, Swagger open in production, empty `catch`, `UseDeveloperExceptionPage` outside Development, TypeNameHandling set. Stop and fix.

## Before done
- [ ] Appendix A items for touched domains (`references/checklist.md`).
- [ ] SCS-TEST-02 self-check; attach results and tool versions to the MR:
  - `gitleaks git -v`
  - `osv-scanner scan source -r .`
  - `dotnet build` with `<AnalysisMode>Recommended</AnalysisMode>` (CA5xxx, EF1002); never suppress without a written reason
  - `dotnet list package --vulnerable --include-transitive`
  - OWASP ZAP baseline scan on the test environment (web apps and APIs)
- [ ] MR description cites the SCS IDs addressed.
- [ ] Security tests written for the change (SCS-TEST-01 scenarios).
