---
name: security-review
description: "Use when a change touches authentication, authorization, input handling, secrets, data access, file or network access, or when reviewing a design for security."
---
# Security review
Check relevant conventions under `docs/` and existing Core abstractions first; apply these as findings (severity, file:line, evidence).
- **AuthN/AuthZ:** every endpoint and Blazor page has an explicit policy or is intentionally public; deny by default; check resource ownership (no IDOR); OpenIddict scopes/claims match the policy.
- **Input:** validate at the boundary (validator in the CQRS pipeline); reject unexpected fields; limit sizes and lengths.
- **Injection:** parameterized SQL only (DbUp scripts and queries); no string-built queries, commands or paths.
- **Output/UI:** Blazor encodes by default; no `MarkupString` or raw HTML from user input; forms use antiforgery.
- **Secrets:** none in code, config files, logs, tests or reports; keys via `ConfigKeyBuilder` and the configured secret store.
- **Logging/errors:** no PII or tokens in logs; ProblemDetails does not leak stack traces or internals.
- **Outbound calls:** typed HttpClient with allow-listed hosts (SSRF), timeouts, no blind redirect following.
- **Files/serialization:** validate type and size of uploads; no unsafe deserialization of untrusted data.
- **Transport/browser:** CORS minimal, security headers, rate limiting on sensitive endpoints.
- **Data isolation:** if multi-tenant, every query filters by tenant.
- **Dependencies:** `dotnet list package --vulnerable` when packages change.
Report unverified concerns separately; never present a guess as a vulnerability.
