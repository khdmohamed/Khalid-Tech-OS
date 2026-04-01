# Skill: New Client

## Trigger
User says: "new client", "add client", "create client", or gives a client name with no existing folder.

## What to collect (ask once, all at once)
- Client name (used for folder: lowercase-hyphenated)
- Industry
- Systems in use (ERP, POS, e-commerce, etc.)
- Contract type (Fixed / Retainer / T&M)
- Any active projects to link

## What to do
1. Create folder: `Clients/[client-name]/outputs/`
2. Create `Clients/[client-name]/client-context.md` using the template below
3. Confirm: "Client [name] created at Clients/[client-name]/"

## Template: client-context.md

```markdown
# Client Context — [CLIENT NAME]

**Industry:** [industry]
**Primary Contact:**
**Contract Type:** [contract type]
**Since:** [current date]

## Systems in Use
[list systems provided]

## Active Projects
[list projects if provided, otherwise: "None yet"]

## Notes
<!-- Important client quirks, preferences, pain points -->

---

## Issues Log

| Date | Issue | Status |
|---|---|---|
| [current date] | [first issue if mentioned, otherwise leave empty row] | Open |
```

## Rules
- Never ask more than once — collect everything in one prompt
- Folder name: lowercase, hyphens only, no spaces
- Always create the outputs/ subfolder
- Do not open or edit any other file
