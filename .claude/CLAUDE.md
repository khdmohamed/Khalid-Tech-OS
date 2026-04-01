# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an **Obsidian Knowledge Vault** for Khalid's technical consulting practice — a personal knowledge management system using Markdown files. It is not a traditional software project with build steps, tests, or deployment.

---

## Who I Am

**Khalid** — Technical Project Manager at Techasus Arabia (Jeddah/Riyadh), and independent software consultant + SaaS developer.

Core identity: software engineer first, PM by role. I think in architecture, systems, and sequences.

---

## My Stack

| Domain | Technologies |
|---|---|
| ERP | Microsoft Dynamics Business Central, LS Central / LS Retail |
| Compliance | ZATCA Phase 2 (CSIDs, CSID renewal, sandbox + prod integration) |
| E-Commerce | Shopify (GraphQL API, theme dev, Arabic RTL, integrations) |
| POS | WPF / .NET 8, SQLite, SignalR, EFT DLL, multi-merchant |
| Mobile | Flutter |
| Backend | ASP.NET Core (C#), Node.js |
| DevOps | Oracle Cloud (me-jeddah-1), nginx, Linux VPS (France) |
| Integrations | Torod logistics, Shopify–LS Central, BC REST APIs |

---

## Vault Structure

```
Khalid-Tech-OS/
├── .claude/
│   ├── CLAUDE.md              # This file
│   ├── commands/              # Reusable slash command definitions
│   │   ├── new-project.md     # /new-project scaffolding
│   │   ├── bc-api-call.md     # /bc-api-call scaffolding
│   │   └── zatca-checklist.md # /zatca-checklist
│   └── skills/                # Domain-specific skill prompts
│       ├── shopify-graphql.md
│       └── zatca-invoice-builder.md
├── Business/                  # Contracts, invoices, proposals, positioning
├── Clients/                   # One folder per client
│   ├── _template/             # Client template
│   ├── al-hamra/
│   ├── alsafwa-konooz/
│   └── dragon-world/
├── Domains/                   # Reusable domain reference material
│   ├── bc-ls-central/
│   ├── flutter/
│   ├── infrastructure/
│   ├── shopify/
│   └── zatca/
├── Knowledge/                 # Decisions, snippets, standards
│   ├── decisions/             # Architecture decisions log
│   ├── snippets/              # Code snippets
│   └── standards/             # e.g., zatca-reference.md
├── Projects/                  # One folder per project
│   ├── _template/             # Project template (brief.md, tasks.md, context.md, outputs/)
│   ├── PRJ-001-central-pos/
│   ├── PRJ-002-zatca-portal/
│   └── PRJ-003-oracle-proxy/
└── Archive/                   # Completed/archived work
```

---

## Project Conventions

- Every client has a folder under `Clients/` using `client-context.md` template
- Every project has a folder under `Projects/` with a number prefix: `PRJ-XXX-name/`
- Every project folder contains: `brief.md`, `tasks.md`, `context.md`, `outputs/`
- Outputs go in the project's `outputs/` folder — never in the root
- Domain reference material lives in `Domains/` — reusable across projects
- Architecture decisions get logged in `Knowledge/decisions/decisions-log.md`

---

## Active Context

### Clients (Techasus Arabia)
- **Konooz / Alsafwa Alarabia** — Shopify + LS Central integration, ZATCA pricing, Arabic RTL, Torod, returns/cancel flows
- **Al Hamra Al Bukhari** — BC implementation, UAT, go-live, training
- **Dragon World** — Freshdesk support portal

### Independent Projects
- **Central POS** — WPF/.NET 8, multi-merchant, ZATCA-compliant, BC integration
- **ZATCA Compliance Portal** — ASP.NET Core 6 MVC, multi-tenant, JWT, EGS device registration
- **Oracle VM Proxy** — nginx reverse proxy on me-jeddah-1 routing France VPS traffic through Saudi IP for ZATCA API

---

## How I Work

- **Think in systems** — give full architectural context, not piecemeal answers
- **No hand-holding** — skip intros, skip re-explaining what I just said
- **Format matters** — use bullets, hierarchy, code blocks. Be terse and clear
- **Trade-offs explicit** — when there are options, show trade-offs clearly
- **Decisions documented** — when a technical decision is made, log it in `Knowledge/decisions/`

---

## Output Defaults

- Code: always include language identifier in code fences
- SQL: format with uppercase keywords
- API calls: show full request/response structure
- When writing docs: Markdown, clear headings, no filler text
- When writing specs: include purpose, inputs, outputs, edge cases

---

## Language

- Default: English
- Switch to Arabic if I write in Arabic
- Technical terms stay in English even in Arabic context

---

## Git Conventions

**Repository:** https://github.com/khdmohamed/Khalid-Tech-OS

**Commit Messages:**
- NEVER include `Co-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>` or any AI attribution
- This is a company repository — commits should appear to be from Khalid only
- Keep messages concise, summarize what changed

**Workflow:**
```
git add .
git commit -m "Brief description"
git push
```
