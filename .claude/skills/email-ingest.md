# Skill: Email Ingest

Parse an email and extract: contacts, project, meeting, reply draft — then archive and update vault.

## When to Use
User pastes an email (headers, body, thread) that contains:
- Project discussion or request
- Meeting scheduling
- Issue report
- Client communication

## Steps

### 1. Parse Email
Extract from raw email:
- **Sender**: name + email
- **CC/To**: all recipients (name + email)
- **Subject**: email title
- **Date**: sent date
- **Body**: key content, action items, decisions
- **Platform**: if mentioned (Teams, Zoom, etc.)

### 2. Identify Client
Match sender domain or company name to existing `Clients/{slug}/` folder.
- If no match: ask user to confirm client slug
- Create client folder from `_template` if missing

### 3. Archive Raw Email
Save to `Clients/{client}/outputs/YYYY-MM-DD-{subject-slug}.md`
```markdown
# Email: {Subject}

**From:** {Name} <{email}>
**To:** {recipients}
**CC:** {recipients}
**Date:** {original date}

## Body

{raw email body}
```

### 4. Update Client Contacts
Update `Clients/{client}/client-context.md`:
- Set Primary Contact if missing
- Add all captured emails to a Contacts table (create if doesn't exist):
  ```markdown
  ## Contacts
  | Role | Name | Email |
  |---|---|---|
  | {derive role} | {name} | {email} |
  ```

### 5. Detect Project
Scan email for:
- Project keywords (integration, implementation, issue, setup, etc.)
- Named systems (Power BI, BC, ZATCA, etc.)
- Urgency language (pending, priority, follow-up, etc.)

If project discussed:
- Check if `PRJ-XXX-{slug}` exists for this client
- If yes: add note to `context.md` Key Notes
- If no: run `/new-project` flow with extracted info

### 6. Extract Meeting
If email contains:
- Meeting request (propose time, schedule, confirm slot)
- Platform mentioned (Teams, Zoom, etc.)
- Agenda items

Create/update `Projects/PRJ-XXX-{slug}/context.md`:
```markdown
## Meetings

| Date | Time | Platform | Attendees | Agenda | Status |
|---|---|---|---|---|---|
| {date} | {slot} | {platform} | {list} | {agenda} | Scheduled/Pending |
```

### 7. Draft Reply
Generate reply draft addressed to sender:
- Confirm meeting slot (if options provided — ask which or suggest first)
- Confirm receipt + next steps
- Professional tone, concise

Present reply in code block for user to copy/send.

### 8. Summary Output
Show what was done:
```
✓ Archived to: Clients/{client}/outputs/{file}.md
✓ Updated: Clients/{client}/client-context.md
✓ Project: PRJ-XXX-{name} (created or updated)
✓ Meeting logged: {date} {time}
✓ Reply drafted (see below)
```

## Example Input
```
From: muhammad.s@newdragonksa.com
Subject: Power BI Integration
...
I propose a virtual meeting for April 2, 2026.
Slot 1: 10:00 AM
Slot 2: 01:30 PM
...
```

## Example Output Actions
1. Archive email to `Clients/dragon-world/outputs/2026-04-01-powerbi-integration.md`
2. Update `Clients/dragon-world/client-context.md` with Muhammad Sheeraz as Primary Contact
3. Create `Projects/PRJ-005-dragon-world-powerbi/` with full brief
4. Add meeting to `context.md` meetings table
5. Draft reply confirming slot ask
6. Show summary checklist

## Role Derivation Hints
- Sender requesting → Client Lead / Stakeholder
- Technical questions → Technical Contact
- Financial/approval → Finance / Decision Maker
- Generic → leave blank or "Team Member"
