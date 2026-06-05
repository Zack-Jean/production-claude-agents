---
name: daily-eod-summary
description: Searches Slack, Gmail, Monday, and Confluence for Zack's work activity each day and drafts an HTML priorities email in Gmail with the new-feedo-signature applied.
---

# Daily EOD Summary — HTML Priorities Email for Melissa

**Run this task every weekday at 3:30 PM Arizona time.**

Your job is to gather Zack's work-related activity from today, organize it into a short numbered priorities list, and save it as an HTML Gmail draft addressed to [Manager]. The draft must use a real numbered list (rendered via `<ol>`) and embed the `new-feedo-signature` block as inline HTML so the email renders properly in Gmail.

---

## Step 1 — Establish Today's Date

Get the current date. Use it to scope all searches to today only. Format the date as MM.DD.YY for the subject line and MM/DD/YYYY in the body greeting if needed.

---

## Step 2 — Gather Activity

Run searches in parallel. Capture work-related activity only — exclude personal messages, off-topic threads, and automated system notifications with no action taken.

**Slack:** Search messages Zack sent today using `slack_search_public_and_private` with `from:zack.jean on:YYYY-MM-DD`. Capture substantive work conversations: decisions communicated, questions answered, updates shared. Skip #np, #basketball-atx, and other clearly off-topic channels.

**Gmail:** Search threads sent or received today using `search_threads` with `after:YYYY/MM/DD`. Capture subject lines and what happened. Exclude automated ticket reminders, marketing, and system notifications.

**Monday.com board activity:** Get board activity for `YOUR_BOARD_ID` using `get_board_activity` scoped to today. Capture status changes, new items, and column value updates on existing items.

**Monday.com item updates:** Use `get_updates` on board `YOUR_BOARD_ID` to pull update threads (comments) posted on items today. Filter to items where the status column is not closed/done OR the Backlog column (`color_mm2ce0vd`) is set. For each relevant update, capture the item name and the **full detail** of what was posted — quotes, decisions, blockers, action items. Do not summarize away substance; these threads are often the most relevant context for the day.

**Confluence:** Search for pages contributed to today using `searchConfluenceUsingCql`. Note if none found and continue.

**Cowork Sessions:** Use `list_sessions` to identify sessions that ran today. Read the most recent 3–4 using `read_transcript` to understand what was produced or worked on.

**Local Claude folder:** Check `/Users/zack.jean/Documents/Claude` for files created or modified today. Use filesystem access to list files with today's modification date (including subfolders). Capture file names and a brief note on what was added or changed. Skip hidden files and system files (`.DS_Store`, etc.). If the folder is empty for today or inaccessible, note it and continue.

---

## Step 3 — Organize into Priorities

Group all activity into 4–8 top-level numbered priorities. Each priority is a named area of work (e.g., "Weekly Status Updates", "Carlton One Scope", "AI Champions"). Under each, add 1–3 short sub-items describing what specifically happened or what the status is.

Keep it high-level — one concise sentence per sub-item. No jargon, no filler. Write it the way Zack would type it himself.

---

## Step 4 — Create the Gmail Draft

Use `create_draft` with:

- **To:** melissa.cantrelle@commerce.com
- **Subject:** `D | Zack Priorities MM.DD.YY` (use today's actual date in this format)
- **body:** Plain-text fallback only — see template below.
- **htmlBody:** REQUIRED. The rich-text version of the email. Use a real `<ol>` numbered list with nested `<ol type="a">` for sub-items — never type "1." "a." characters in the HTML body. Embed the new-feedo-signature block exactly as defined below at the end of the email.

### Why HTML is required

Gmail signatures cannot be referenced by name through the API and will not auto-attach to API-created drafts. The `new-feedo-signature` block must be embedded inline in the HTML body so the email renders correctly when opened in Gmail. Plain text alone produces a "1. 2. 3." typed list and no signature, which is not acceptable.

### HTML body template (use exactly this structure)

```html
<div style="font-family:'DM Sans', Arial, sans-serif; font-size:14px; color:#000000; line-height:1.5;">
  <p>Hi Melissa,</p>
  <p>Please see priorities from today:</p>
  <ol style="padding-left:24px; margin:0 0 16px 0;">
    <li style="margin-bottom:8px;">
      <strong>[Priority Name]</strong>
      <ol type="a" style="padding-left:24px; margin:6px 0 0 0;">
        <li>[Sub-item]</li>
        <li>[Sub-item]</li>
      </ol>
    </li>
    <li style="margin-bottom:8px;">
      <strong>[Priority Name]</strong>
      <ol type="a" style="padding-left:24px; margin:6px 0 0 0;">
        <li>[Sub-item]</li>
      </ol>
    </li>
    <!-- repeat for each priority -->
  </ol>
  <p style="margin:16px 0;">Please reach out with any questions/issues/concerns.</p>
  <br>
  <!-- new-feedo-signature start -->
  <div style="font-family:'DM Sans', Arial, sans-serif; font-size:13px; color:#000000; line-height:1.5;">
    <div>Thanks, <strong>[Author]</strong> &nbsp;/&nbsp; Technical Delivery Manager</div>
    <br>
    <div>Visit us at &nbsp;&nbsp;<a href="https://[your-company].com" style="color:#6058FF; text-decoration:none;">[your-company].com</a></div>
    <div>Follow us &nbsp;&nbsp;&nbsp;<a href="https://www.linkedin.com/company/[your-company]/" style="color:#6058FF; text-decoration:none;">LinkedIn</a> &nbsp;/&nbsp; <a href="https://x.com/[your-company]" style="color:#6058FF; text-decoration:none;">X</a> &nbsp;/&nbsp; <a href="https://www.instagram.com/[your-company]" style="color:#6058FF; text-decoration:none;">Instagram</a></div>
    <br>
    <div style="font-size:11px; color:#666666; font-style:italic;">This email may contain confidential information. If you believe you may have received this message in error, please contact me and delete all copies.</div>
  </div>
  <!-- new-feedo-signature end -->
</div>
```

### Plain-text fallback (for the `body` field)

```
Hi Melissa,

Please see priorities from today:

1. [Priority Name]
   a. [Sub-item]
   b. [Sub-item]
2. [Priority Name]
   a. [Sub-item]
3. [Priority Name]
   a. [Sub-item]
   b. [Sub-item]

Please reach out with any questions/issues/concerns.

Thanks, [Author]  /  Technical Delivery Manager
```

### Formatting rules

- Top-level list: bolded priority name on its own line, then nested `<ol type="a">` for sub-items.
- Use real `<ol>` and `<li>` tags — never hard-code "1." "2." "a." "b." characters in the HTML.
- Keep sub-items to one concise sentence each.
- Keep the sign-off line ("Thanks, [Author] / Technical Delivery Manager") inside the signature block; do not duplicate it above.
- Do not modify the new-feedo-signature block other than swapping the priorities content above it.

### Critical: do not wrap the htmlBody value

Pass the HTML to `create_draft`'s `htmlBody` parameter as a raw HTML string. **Do not** wrap it in any of the following — they are XML/template syntax, not HTML, and will render as literal text in the email:

- `<![CDATA[ ... ]]>`
- Backticks, triple backticks, or fenced code blocks (```)
- `<html>`, `<head>`, `<body>`, or `<!DOCTYPE>` tags
- XML processing instructions like `<?xml ... ?>`

The `htmlBody` should start with `<div ...>` (the outer wrapper from the template) and end with the matching `</div>`. Nothing before, nothing after. Before submitting the draft, scan the htmlBody string for `]]>`, `<![CDATA`, or backticks and remove them.

---

## Step 5 — Confirm Completion

After creating the draft, output a brief summary:
- Date used in subject line
- Number of priorities included
- Confirmation the Gmail draft was created (with the new draft ID)
- Note any source that failed or returned no data