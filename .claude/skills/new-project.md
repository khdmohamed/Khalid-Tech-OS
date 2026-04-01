# Skill: New Project

## Trigger
User says: "new project", "create project", "start project", or describes work that needs a project folder.

## What to collect (ask once, all at once)
- Project name (short, descriptive)
- Client (existing folder name, or "internal")
- Objective (one sentence)
- Stack (technologies involved)
- Any known deliverables or tasks

## What to do
1. Determine next project number by checking existing `Projects/PRJ-XXX-*` folders
2. Create folder: `Projects/PRJ-[NNN]-[project-name]/outputs/`
3. Create and fill `brief.md`, `tasks.md`, `context.md` using templates below
4. If client provided, add project reference to `Clients/[client]/client-context.md` under Active Projects
5. Confirm: "Project PRJ-[NNN]-[name] created."

## Template: brief.md

```markdown
# Project Brief — [PROJECT NAME]

**ID:** PRJ-[NNN]
**Client:** [client]
**Start Date:** [current date]
**Status:** Active

## Objective
[objective]

## Scope
**In:** [derive from objective and stack]
**Out:** TBD

## Stack
[stack]

## Deliverables
[list if provided, otherwise placeholder]

## Key Contacts
| Role | Name | Contact |
|---|---|---|
|  |  |  |
```

## Template: tasks.md

```markdown
# Tasks — [PROJECT NAME]

## Active
- [ ] [first logical task derived from objective]

## In Review


## Done


## Blocked

```

## Template: context.md

```markdown
# Context — [PROJECT NAME]

## Decisions Log
| Date | Decision | Rationale |
|---|---|---|

## Key Notes


## References

```

## Rules
- Collect all info in one prompt — no back and forth
- Auto-increment PRJ number — check the folder, don't guess
- Always derive at least one starter task from the objective
- Always update the linked client file if a real client is named
- Never scatter files — everything under the PRJ folder
