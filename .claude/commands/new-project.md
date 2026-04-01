# Command: /new-project

Create a new project folder under Projects/ using the _template structure.

## Steps

1. **Ask for input** (if not provided): project name, client, objective, stack, key contacts
2. **Generate PRJ-XXX number**:
   - List `Projects/` folders
   - Extract existing PRJ numbers
   - Next = max(existing) + 1
   - Verify `PRJ-XXX` folder does NOT exist before proceeding
3. **Create folder structure**:
   ```
   Projects/PRJ-XXX-{slug}/
   ├── brief.md
   ├── tasks.md
   ├── context.md
   └── outputs/
   ```
4. **Fill brief.md** with:
   - ID, Client, Start Date (today), Status (Planning/Active)
   - Objective (what + why)
   - Scope (in/out)
   - Stack (derive from context if not explicit)
   - Deliverables (derive from objective — at least 3 concrete items)
   - Key Contacts table (Role | Name | Contact)
5. **Fill tasks.md** with:
   - Active tasks (immediate next steps)
   - Done: "[x] Project registered"
6. **Fill context.md** with:
   - Decisions Log table (empty, ready)
   - Key Notes (project-specific context)
   - References (links, doc paths, API endpoints)
7. **Update client-context.md**:
   - Add `PRJ-XXX — [Project Name]` to Active Projects
   - Create client folder if missing
   - Update Primary Contact if provided
8. **Confirm creation** — show project path and brief summary

## PRJ Collision Rules
- ALWAYS scan `Projects/` first
- If `PRJ-XXX` folder exists, increment until finding gap
- Never assume sequential without verification

## Contacts Enforcement
- If contacts provided, write FULL Key Contacts table into brief.md
- If updating client, write Primary Contact into client-context.md
- Never drop contact info — capture all emails mentioned

## Deliverables Derivation
- If not explicitly provided, derive from objective:
  - Analysis → "Root cause analysis", "Investigation report"
  - Implementation → "Feature deployed", "UAT passed"
  - Integration → "Data mapping", "End-to-end test", "Documentation"
  - Meeting/Discovery → "Meeting notes", "Requirements document", "Timeline proposal"
