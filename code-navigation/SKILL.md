---
name: code-navigation
description: "Use before reading or searching a large codebase: locate symbols, callers, callees, impact and flows with CodeGraph (MCP tool codegraph_explore or the codegraph CLI), then read only the needed line ranges. Also covers the limits of the index and the evidence rule for caller counts."
---
# Code navigation (large solution)
Goal: find the right code in 1-3 calls and read only what you need.

## Order of work
1. If `.codegraph/` exists at the repo root, start with CodeGraph: MCP tool `codegraph_explore` (symbol names or a question) or the shell command `codegraph explore "<symbols or question>"`. The returned source is current and line-numbered: treat it as already read and do not Read the same file again.
2. Focused follow-ups (read-only CLI): `codegraph callers <symbol>`, `codegraph callees <symbol>`, `codegraph impact <symbol>`, `codegraph node <symbol or file>`, `codegraph query <text>`, `codegraph files <path>`, `codegraph affected <files...>` (tests affected by a change). When a name has several definitions, add `--file <path>`.
3. No `.codegraph/` or tool missing: Grep to locate, then Read with offset/limit around the hit. Never read a whole large file to find one member. Mention the missing index once in your Handoff.
4. After edits, give auto-sync a moment before querying. If an answer carries a staleness notice, Read the file directly.

## Limits (observed on a small C# sample; verify on real code)
- A class registered in DI (`AddScoped<IFoo, Foo>`, `AddTransient`, `AddSingleton`, assembly scanning) can show zero callers.
- A call through an interface-typed field may not link to the interface method; caller lists can be empty or partial.
- `.razor` files are indexed as a component with few symbols; `@inject`, `@code` calls and child components can be missing. Code-behind `.razor.cs` is C# and is indexed.
- Reflection, attributes, configuration keys, string routing, source generators and runtime dispatch are not followed.
- `bin/`, `obj/`, `wwwroot/lib/` can be indexed unless excluded (`codegraph.json`).
Use CodeGraph to LOCATE and to get a first impact list. It is never proof of "no callers" or "dead code".

## Evidence rule for "unused", "0 callers", "safe to delete", "blast radius"
CONFIRMED only if you also checked with Grep: the name as a string; DI registrations (`AddScoped|AddTransient|AddSingleton|TryAdd|Scrutor|Decorate|Configure<`); `@inject` and `<Component` usage in `.razor`; `typeof(`/`nameof(`/reflection/attributes; `appsettings*.json` and route/endpoint registration; test projects. List what you checked in the Handoff, otherwise label the claim LIKELY.

## Context discipline
`explore` returns one dense payload that stays in your context. Ask narrow questions (name symbols), use `callers`/`impact` for follow-ups, and put only needed paths and line ranges in your Handoff, not the dump.

## Optional precise tool
If an `LSP` tool is available (plugin csharp-lsp), use go-to-definition, find-references and diagnostics for exact answers. It complements the steps above.
