---
name: ossm-operator-initiatives
description: >-
  Analyze the OCP Operator Portfolio Alignment Google Doc against OSSM's tracking
  Jiras and update the OSSM OCP Operator Initiatives Tracker spreadsheet. Use when
  the user asks to update operator initiatives, check OCP operator requirements,
  refresh the OSSM tracking sheet, or review what OSSM needs to work on for OCP releases.
---

# OSSM OCP Operator Initiatives Tracker

## Purpose

Analyze the "RH Operator Portfolio Alignment Call" Google Doc, cross-reference
with OSSM Jira epics, Slack channel updates, and program email communications,
then update the tracking spreadsheet with current status.

## Key References

- **Source Doc**: `1fQSkbliOD6p51UFLLDz8HUmJhxnYZaACy1mxmdRX1ow` (RH Operator Portfolio Alignment Call)
- **Tracking Sheet**: `1eluM4NuZIPuEvDH1XPOkJKNX0xSsb1-_Vt_8DHzvyS4` (OSSM OCP Operator Initiatives Tracker)
- **Jira Project**: OSSM (search entire project, not limited to any single epic)
- **Known Tracking Epics**: OSSM-13174 (OCP 4.23/5.0), OSSM-11620 (OCP 4.22, Closed)
- **User Email**: ehopfenb@redhat.com
- **Jira Cloud**: https://redhat.atlassian.net
- **Slack Channels**:
  - `C09NEFBFZSA` — #team-program-managers-operator (Eugenia Gibson's PgM channel)
  - `C09E3S730MN` — #forum-operator-fw-program (broad operator program discussions)
- **Email Sources**: `operatorframework-pgm@redhat.com`, Eugenia Gibson (`eugenia.gibson@redhat.com`)

## OSSM Context (Critical Filters)

OSSM is a **Platform Agnostic** operator. Apply these rules:

1. **SKIP** items targeted only at "Core Platform" or "Platform Aligned" operators
2. **INCLUDE** items for "All OLM-managed operators", "Platform Agnostic", or "All operators"
3. **ALREADY DONE** (do not re-add to Active):
   - UBI9-minimal migration (OCPSTRAT-2553)
   - OLMv1 Adoption Phase 1 & 2 (OCPSTRAT-3025 / OPGM-1 / OSSM-9045)
4. For OCPSTRAT-2361 (PQC ML-KEM): now MANDATORY for Platform Agnostic (changed from link-only as of June 2026). New tracking placeholder: OCPSTRAT-3303.

## Workflow

### Step 1: Fetch the Source Document

Use the Google Workspace MCP `get_doc_content` tool:
```
document_id: 1fQSkbliOD6p51UFLLDz8HUmJhxnYZaACy1mxmdRX1ow
user_google_email: ehopfenb@redhat.com
```

Focus on these tabs:
- **Prioritization** — the ranked initiative table
- **Action Items** — current action items with timelines
- **Meeting Agenda** — new topics and context

### Step 2: Fetch Current Sheet State

Use `read_sheet_values` on the tracking spreadsheet:
```
spreadsheet_id: 1eluM4NuZIPuEvDH1XPOkJKNX0xSsb1-_Vt_8DHzvyS4
range_name: Active Initiatives!A1:K50
```

Also read the Completed sheet to avoid duplicates:
```
range_name: Completed!A1:H50
```

### Step 3: Fetch Slack Channel Updates

Use the Slack MCP `get_channel_history` tool to pull recent messages from both
program channels. Look for announcements, deadlines, and action items relevant
to OSSM's tracked initiatives.

```
# PgM channel (Eugenia's announcements, deadlines, program updates)
channel_id: C09NEFBFZSA
limit: 50
oldest: <30-60 days ago, ISO 8601>
include_threads: true

# Broad operator program channel (cross-team discussions, policy changes)
channel_id: C09E3S730MN
limit: 50
oldest: <30-60 days ago, ISO 8601>
include_threads: true
```

Key things to extract from Slack:
- New deadlines or enforcement dates (e.g., Konflux pipeline gating)
- Program-wide announcements affecting OSSM
- Clarifications on initiative scope or requirements
- Action items called out by Eugenia Gibson or program leads

### Step 4: Fetch Program Email Communications

Use the Google Workspace MCP `search_gmail_messages` tool to find recent emails
from the operator program mailing list and Eugenia Gibson:

```
query: from:eugenia.gibson@redhat.com OR from:operatorframework-pgm@redhat.com newer_than:60d
user_google_email: ehopfenb@redhat.com
page_size: 10
```

Then use `get_gmail_messages_content_batch` to read the full content of found messages.

Key things to extract from emails:
- Immediate asks or action items for operator teams
- New initiative announcements or scope changes
- Timeline shifts or shipping freezes
- Program-level status updates and kudos (for context)

### Step 5: Fetch OSSM Jira Status

Search the **entire OSSM project** for issues related to each initiative. Do NOT limit
to a single epic — issues may exist anywhere in the project.

Run one JQL query per active initiative using keywords and OCPSTRAT/HPSTRAT references:

```
# TLS Compliance (OCPSTRAT-2611)
project = OSSM AND (summary ~ "OCPSTRAT-2611" OR summary ~ "TLS Profile" OR summary ~ "TLS compliance" OR summary ~ "TLS scanner" OR description ~ "OCPSTRAT-2611") ORDER BY key DESC

# PF6 / Compass (OCPSTRAT-2962)
project = OSSM AND (summary ~ "OCPSTRAT-2962" OR summary ~ "PatternFly" OR summary ~ "PF6" OR summary ~ "Compass" OR description ~ "OCPSTRAT-2962") ORDER BY key DESC

# Network Policies - Operands (HPSTRAT-104)
project = OSSM AND (summary ~ "HPSTRAT-104" OR summary ~ "network polic" OR description ~ "HPSTRAT-104") ORDER BY key DESC

# Network Policies - Operators (HPSTRAT-278)
project = OSSM AND (summary ~ "HPSTRAT-278" OR description ~ "HPSTRAT-278") ORDER BY key DESC

# Operator Compatibility (HPSTRAT-127)
project = OSSM AND (summary ~ "HPSTRAT-127" OR description ~ "HPSTRAT-127") ORDER BY key DESC

# RHCOS Dual Compatibility (HPSTRAT-105)
project = OSSM AND (summary ~ "HPSTRAT-105" OR summary ~ "RHCOS" OR summary ~ "RHEL 10" OR summary ~ "UBI 10" OR description ~ "HPSTRAT-105") ORDER BY key DESC

# PQC ML-KEM (OCPSTRAT-2361)
project = OSSM AND (summary ~ "OCPSTRAT-2361" OR summary ~ "PQC" OR summary ~ "ML-KEM" OR description ~ "OCPSTRAT-2361") ORDER BY key DESC

# Default PQ Crypto Policy (OCPSTRAT-3113)
project = OSSM AND (summary ~ "OCPSTRAT-3113" OR summary ~ "crypto policy" OR description ~ "OCPSTRAT-3113") ORDER BY key DESC
```

Also search for any newly linked issues:
```
project = OSSM AND (summary ~ "OCPSTRAT" OR summary ~ "HPSTRAT") AND status != Closed ORDER BY key DESC
```

Use fields: `summary, status, issuelinks, labels, issuetype`
Set `maxResults: 50` and `responseContentFormat: markdown`

### Step 6: Analyze and Diff

Compare ALL data sources (source doc, Slack, email, Jira) against the current sheet:

1. **New initiatives**: Items in the Prioritization/Action Items tabs that are NOT in the sheet yet and apply to OSSM (per filter rules above)
2. **Status changes**: OSSM Jira statuses that have changed since last update
3. **Timeline changes**: Target dates that shifted in the source doc or announced via Slack/email
4. **New deadlines**: Enforcement dates or gating checks announced in Slack/email (e.g., Konflux pipeline deadlines)
5. **Scope changes**: Requirements that expanded or narrowed (e.g., mandatory vs. link-only)
6. **Completed items**: Initiatives where all OSSM Jiras are Closed — move to Completed sheet

When incorporating Slack/email intelligence, note the source in the Context/Details column
(e.g., "Source: Slack #team-program-managers-operator" or "Source: operatorframework-pgm email").

### Step 7: Update the Sheet

For each change found:

- **New row**: Append to the next empty row in "Active Initiatives"
- **Status update**: Update columns I (OSSM Status) and K (Last Updated)
- **Moved to completed**: Remove from Active, add to Completed sheet
- **New line for updates**: If there is a new update on the source doc, Slack, or email for an existing initiative, add the update details to the Context/Details or Next Steps column and update Last Updated

Use `modify_sheet_values` to write changes. Always update the "Last Updated" column with today's date.

### Step 8: Report Summary

After updating, provide the user a summary:
- New initiatives added (if any)
- Status changes detected
- Items moved to Completed
- Items needing attention (no OSSM Jira yet, or approaching deadline)
- Key intel from Slack/email that informed updates

## Sheet Columns Reference

### Active Initiatives
| Col | Header | Description |
|-----|--------|-------------|
| A | ID / Strategy Key | OCPSTRAT-XXXX or HPSTRAT-XXX |
| B | Initiative Name | Short name from the doc |
| C | Priority | From the doc (High Blocking, High Non-blocking, Medium, etc.) |
| D | Context / Details | Condensed description of what's needed |
| E | Target Timeline | OCP version or date |
| F | Target Group | Who it applies to |
| G | Support Channel | Slack channel |
| H | OSSM Jira | OSSM-XXXX issue(s) tracking this |
| I | OSSM Status | Current status of OSSM's work |
| J | Next Steps | What OSSM needs to do next |
| K | Last Updated | Date of last refresh |

### Completed
| Col | Header |
|-----|--------|
| A | ID / Strategy Key |
| B | Initiative Name |
| C | Context / Details |
| D | OSSM Jira |
| E | OSSM Status |
| F | Completion Date |
| G | Notes |
| H | Last Updated |
