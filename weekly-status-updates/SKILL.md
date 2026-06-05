---
name: weekly-tdm-status-updates
description: Generate weekly TDM project status update Gmail drafts from Monday.com, Gmail, and Drive
---

You are running the weekly TDM (Technical Delivery Manager) status update workflow for [Author], Technical Delivery Manager at [Your Company] (a Commerce company). Execute all steps autonomously — do not ask clarifying questions. Present a summary when complete.

---

## OBJECTIVE

For every open project on the Global Services Automation Projects Monday.com board:
1. Pull current status from Monday.com, Gmail, Google Drive, and Slack
2. Generate a branded HTML status update email
3. Save it as a Gmail draft addressed to the correct stakeholders
4. Surface a prioritized action item list

---

## STEP 1 — Pull open projects from Monday.com

Query board ID: `YOUR_BOARD_ID` (Global Services Automation Projects) using `get_board_items_page`.

If the response exceeds token limits, save it to a temp file and use a subagent to extract the data.

For each item capture: item name, status, due date, and any notes. Include all items where status is NOT Done / Complete / Closed. Flag any item past its due date with no recent update — these are action items.

---

## STEP 2 — Gather context for each open project

For each open project, search ALL of the following in parallel:

**Gmail:** Search threads containing the project name, last 30 days. Read the most recent thread(s) in full. Extract: latest status, decisions, open questions, blockers, stakeholder names and email addresses.

**Google Drive:** Search for project docs (Project Overview, Project Plan, Testing docs, WIP trackers) by project name. Read any doc modified in the last 14 days. Extract: current phase, milestone status, risks, open items. Key reference: Status Updates Template (Drive ID: `149rVoes3tWnT2W65b1EXZjWjybH_upzGmM3nKyINhCk`) — use the Large Project tab format for all emails.

**Slack:** Search the project name across public and private channels. Extract: decisions, blockers, escalations. If Slack returns no results, proceed with Gmail + Drive only.

**Source priority for narrative status:** Gmail (most recent) > Drive > Monday.com. Trust Monday.com for milestone dates.

---

## STEP 3 — Identify stakeholders

Determine To/CC recipients from Gmail threads, Project Overview docs, or the Monday.com item.

Standard pattern:
- **To:** Project requestor + Manager/Sponsor
- **CC:** melissa.cantrelle@commerce.com (always, unless already in To)

If stakeholders cannot be identified, do NOT create a draft. Flag as a 🔴 action item.

Known stakeholder mappings (verify against current Gmail threads — may have changed):
- Carlton One Spec Sheet Automation → To: eric.elliott@[your-company].com, jessica.pope@[your-company].com | CC: melissa.cantrelle@commerce.com
- Zapier to Make Migration → To: yesenia.monroy@[your-company].com, jessica.pope@[your-company].com | CC: melissa.cantrelle@commerce.com
- Toggl Overage Report Automation → To: jessica.pope@[your-company].com, peter.baldwin@[your-company].com | CC: yesenia.monroy@[your-company].com, melissa.cantrelle@commerce.com

---

## STEP 4 — Generate a branded HTML status update email for each project

Structure each email using the Large Project tab format from the Status Updates Template. Render as a complete HTML email with inline CSS.

**Brand styling:**
- Header background: `#170245` (Indigo), white text
- Meta bar background: `#DFDFFF` (Lilac), `#2D2D83` (Violet) text
- Section heading underline: `#6058FF` (Iris)
- Table headers: `#170245` fill, white text
- Alternating table rows: `#F5F3F2` (Pearl) for even rows, white for odd
- Callout/alert boxes: `#DFDFFF` background, `#6058FF` left border (4px)
- Font stack: `'DM Sans', Arial, sans-serif`
- Confidentiality footer: *[Your Company] proprietary and confidential. Unauthorized disclosure, reproduction, or other use is prohibited.*

**Required sections (in order):**
1. Header — project name, "Week Ending: [Friday of current week MM/DD/YYYY]", TDM: [Author], Sponsor, status indicator, phase, target go-live
2. Key Accomplishments — bullet list of completions this week
3. Immediate Next Steps — bullets with **owner name** bolded and target date
4. Project Overview table — Deliverable | Status | Notes
5. Key Risks / Issues table — Risk | Impact | Mitigation / Owner (if none: "No items currently")
6. Dependencies / Requests — anything blocked on a stakeholder action
7. Resources — Monday.com item link, relevant Drive doc links
8. Confidentiality footer

**Status indicators:**
- 🟢 On Track
- 🟡 At Risk / Testing — [qualifier] / Feedback Under Review
- 🔴 Blocked / Escalation Required

**Subject line:** `D | [Project Name] — Weekly Status Update | Week ending [Friday's date MM/DD/YYYY]`

Week-ending date = Friday of the current week(today).

---

## STEP 5 — Save as Gmail drafts

Use `create_draft` for each project:
- `to`: stakeholder emails
- `cc`: melissa.cantrelle@commerce.com (unless already in To)
- `subject`: per Step 4 format
- `htmlBody`: full branded HTML email

Do NOT send. Save as drafts only.

---

## STEP 6 — Surface action items

After all drafts are saved, output a consolidated action item list:

🔴 Must address before sending (blocks draft):
- Projects with no identifiable stakeholders (draft not created)
- Open questions in a draft that require stakeholder input before sending
- Drafts that may be inaccurate due to conflicting source data

🟡 Should address soon (doesn't block draft):
- Projects past their due date with no recent activity
- Monday.com items that appear stale or may need re-scoping or archiving

ℹ️ FYI / low urgency:
- Old/incorrect drafts from prior runs that should be manually deleted (include draft ID — drafts cannot be deleted programmatically)
- Minor inconsistencies between sources

---

## ERROR HANDLING

- **Monday.com response too large:** Save to temp file, use subagent to extract items.
- **Drive file too large:** Use a subagent with jq/python to extract relevant sections. Do not skip the file.
- **Slack returns no results:** Proceed with Gmail + Drive. Note in run summary.
- **Stakeholders unknown:** Do not create a draft. Flag 🔴.
- **Conflicting source data:** Trust Gmail > Drive > Monday.com for narrative; Monday.com for dates.
- **Old/incorrect draft found:** Create new corrected draft. Flag old draft ID as ℹ️ action item for manual deletion.

---

## RUN SUMMARY FORMAT

After completing all steps, present:

**Drafts Created:** [count] — list each project name and draft ID
**Drafts Skipped:** [count] — list reason for each
**Action Items:** [grouped by 🔴 / 🟡 / ℹ️]
**Sources checked:** Monday.com ✓, Gmail ✓, Drive ✓, Slack ✓/✗ (no results)