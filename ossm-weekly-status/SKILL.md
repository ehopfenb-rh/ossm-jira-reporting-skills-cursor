---
name: ossm-weekly-status
description: >-
  Generate a weekly status report for the OpenShift Service Mesh (OSSM) program
  by pulling data from Jira and Product Pages, rendering a canvas, and exporting
  to Google Docs. Use when the user asks to generate, run, or pull their OSSM
  status report, weekly status, program update, or portfolio summary.
---

# OSSM Weekly Status Report

Generate a comprehensive weekly status report for the Red Hat OpenShift Service Mesh program.

## Context

- **Program:** OpenShift Service Mesh (OSSM)
- **Jira project:** `OSSM` on `redhat.atlassian.net`
- **Product Pages product entity:** ID `158`
- **TPM:** Emily Hopfenberg (`ehopfenb@redhat.com`)
- **TPMAI tracker:** TPMAI-122

### Active Releases

Update these entity IDs when new releases appear. Use `search_entities` with `q: "OpenShift Service Mesh"` and `kind: "release"` to discover new ones.

| Release | Entity ID | Notes |
|---------|-----------|-------|
| OSSM 2.6 | 2717 | EOL June 30, 2026 |
| OSSM 3.3 | 3167 | Current GA (Mar 19, 2026) |
| OSSM 3.4 | 3319 | Next minor (target Jun 16, 2026) |

## Procedure

Run all steps, then produce both a canvas and a Google Doc.

### Step 1: Pull Jira Data

Use `searchJiraIssuesUsingJql` on `plugin-atlassian-atlassian` MCP:

```
cloudId: "redhat.atlassian.net"
jql: "project = OSSM ORDER BY updated DESC"
maxResults: 100
fields: summary, status, issuetype, priority, assignee, fixVersions, labels, updated
responseContentFormat: "markdown"
```

Parse results and compute:
- Counts by status (In Progress, New, Closed, etc.)
- Counts by issue type (Story, Task, Sub-task, Bug, Epic, Vulnerability, Ticket)
- Active fix versions
- CVE/vulnerability issues grouped by CVE ID with streams affected, assignee, and status
- Critical/blocker priority items
- Unassigned issue count

### Step 2: Pull Product Pages Data

Use `user-productpages` MCP. Run these calls in parallel:

1. `get_latest_status_posts` for each active release entity ID
2. `browse_schedule` for the next upcoming release (to get GA date, milestones)
3. `get_latest_status_posts` for entity `158` (product-level status)

Extract from each status post: summary, color/health (On Track / At Risk / Off Track), date, author, and any actions/mitigations.

### Step 3: Generate Canvas

Write a canvas to `canvases/OSSM-status-report.canvas.tsx` containing:

1. **Header** with report title, date, overall status pill
2. **Summary stats** row: active issues, in-progress, open CVEs, unassigned
3. **Release overview** cards: one per active release with status color, target date, RM, key blockers, schedule milestones
4. **CVE tracker** table: CVE ID, component, streams affected, assignee, status, with row tones
5. **Key active issues** table: critical/blocker items with key, summary, priority, assignee, status
6. **Strategic initiatives** cards: major epics and cross-program efforts
7. **Issue distribution** tables: by status and by type
8. **Footer** with data source attribution and last Product Pages post date

Read the canvas skill at `~/.cursor/skills-cursor/canvas/SKILL.md` and SDK types at `~/.cursor/skills-cursor/canvas/sdk/index.d.ts` before writing the canvas.

### Step 4: Export to Google Doc

Use `user-google_workspace` MCP via `import_to_google_doc`:

1. Write an HTML file to `~/.workspace-mcp/attachments/ossm-status-report.html` with the full report using proper HTML formatting: `<h1>` / `<h2>` / `<h3>` headings, `<table>` with `<th>` / `<td>`, `<ul>` / `<li>` bullet lists, `<strong>` for bold, and inline styles for colored status indicators and row highlighting.
2. Call `import_to_google_doc` with `file_path` pointing to the HTML file, `source_format: "html"`, and `user_google_email: "ehopfenb@redhat.com"`.
3. Google Drive auto-converts HTML to native Google Docs format, preserving tables, headings, lists, bold, and colors.
4. Return the Google Doc link to the user.

Style guidance for the HTML:
- Use colored status labels: green for On Track, yellow/amber for At Risk, red for Off Track
- Use row background colors in tables: `#fff3cd` for warning rows, `#fee2e2` for danger rows, `#d1fae5` for success rows
- Use a red bottom border on `<h2>` section headings for visual hierarchy
- Use a callout box (yellow background, left border) for important notices like EOL warnings

### Step 5: Summarize

Tell the user:
- Canvas is ready to open
- Google Doc link
- Key highlights: any status changes, new blockers, items needing assignment
- Date of the last Product Pages TPM status post (so they know if it needs updating)
