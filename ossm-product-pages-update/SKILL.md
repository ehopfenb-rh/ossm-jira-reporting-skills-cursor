---
name: ossm-product-pages-update
description: >-
  Generate a weekly OSSM Product Pages update by pulling data from Jira, Product
  Pages, Gemini meeting notes, and the container vulnerability report, then
  produce a formatted Google Doc ready for copy-paste. Use when the user asks to
  generate, create, or draft their Product Pages update, PP update, or weekly
  OSSM status for Product Pages.
---

# OSSM Product Pages Update

Generate a formatted Product Pages update for OpenShift Service Mesh and export it as a Google Doc.

## Context

- **Program:** OpenShift Service Mesh (OSSM)
- **Jira project:** `OSSM` on `redhat.atlassian.net`
- **Product Pages product entity:** ID `158`
- **TPM:** Emily Hopfenberg (`ehopfenb@redhat.com`)
- **Main document:** Google Doc ID `1A-v7ozJ0DE1c9o9TdV4s5tahw3ChZqxlcMw5xUgAjNY` (tab `t.0`)

### Active Releases

Update these entity IDs when new releases appear. Use `search_entities` with `q: "OpenShift Service Mesh"` and `kind: "release"` to discover new ones.

| Release | Entity ID | Notes |
|---------|-----------|-------|
| OSSM 2.6 | 2717 | EOL June 30, 2026 |
| OSSM 3.3 | 3167 | Current GA (Mar 19, 2026) |
| OSSM 3.4 | 3319 | Next minor (target Jun 30, 2026) |

## Procedure

### Step 1: Pull Previous Entry

Use `user-google_workspace` MCP `get_doc_content` to read the main document:

```
document_id: "1A-v7ozJ0DE1c9o9TdV4s5tahw3ChZqxlcMw5xUgAjNY"
user_google_email: "ehopfenb@redhat.com"
```

Parse the most recent entry (first entry under "2026") to understand:
- What was reported last time
- What dates/targets were mentioned
- What action items were pending

### Step 2: Pull Data Sources (in parallel)

#### 2a: Product Pages Status Posts

Use `user-productpages` MCP `get_latest_status_posts` for each active release entity ID:
- Entity 3319 (OSSM 3.4)
- Entity 3167 (OSSM 3.3)
- Entity 2717 (OSSM 2.6)

Also call `browse_schedule` for each to get GA dates and milestones.

#### 2b: Jira Data

Use `plugin-atlassian-atlassian` MCP `searchJiraIssuesUsingJql`:

1. **Open issues for next minor release:**
   ```
   cloudId: "redhat.atlassian.net"
   jql: "project = OSSM AND fixVersion = \"OSSM 3.4.0\" AND status != Closed ORDER BY priority DESC"
   fields: summary, status, issuetype, priority, assignee, fixVersions, labels
   ```

2. **Open vulnerabilities:**
   ```
   jql: "project = OSSM AND issuetype = Vulnerability AND status != Closed ORDER BY priority DESC"
   ```

#### 2c: Gemini Meeting Notes

Use `user-google_workspace` MCP `search_gmail_messages` to find recent notes:

1. **Program Call notes:**
   ```
   query: "from:gemini-notes@google.com newer_than:7d \"Service Mesh\""
   ```

2. **Release & Maintenance notes:**
   ```
   query: "from:gemini-notes@google.com newer_than:7d \"Release & Maintenance\""
   ```

3. **Eng/Docs sync notes:**
   ```
   query: "from:gemini-notes@google.com newer_than:7d \"eng/docs\""
   ```

Fetch content with `get_gmail_messages_content_batch` for the most recent messages.

#### 2d: Container Vulnerability Report

Search for the latest daily report:

```
query: "subject:\"OSSM Container Vulnerability Report\" newer_than:3d"
```

Extract: total urgent/warning counts, specific container grades, downgrade dates, and affected CVEs.

### Step 3: Synthesize the Update

Compose the update following this structure. Refer to [html-template.md](html-template.md) for the HTML output format.

#### Summary (2 sentences max)

- Sentence 1: Status of the next minor release (version, target date, on track / at risk / off track)
- Sentence 2: Z-Stream status OR most notable other update

Base the risk assessment on:
- **On Track:** No blockers, milestones being met, builds progressing
- **At Risk:** Active blockers (proxy, rebasing, CI), date recently moved, unresolved dependencies
- **Off Track:** Date missed or will certainly miss, critical blockers with no remediation path

#### Minor Releases

For each active minor release (currently 3.4.0):
- Target date and any changes from last update
- Timeline dependencies (upstream releases, midstream branches, code freeze)
- Downstream build status
- Epic status summary

#### Z-Streams

For each active Z-Stream track:
- Target dates
- Container grade status (from vuln report): count of B/C grades, downgrade dates
- CVE details if relevant
- EOL reminders for 2.6

#### Strategic Program & Integration Updates

Include items from meeting notes covering:
- Documentation strategy
- AI collaborations
- ACM integration
- OCP compliance & security
- Any other cross-program initiatives

#### Action & Mitigation Plan

Only include if there are active risks. Format:

```
Target Release Date: [date] ([ON TRACK / AT RISK / OFF TRACK])
Owner: [Release Manager name]
Status Update: [Current blocker details and remediation]
Component Status:
- [Component]: [Status] ([date])
```

### Step 4: Generate HTML and Import as Google Doc

1. Write a complete HTML file to `~/.workspace-mcp/attachments/ossm-pp-update.html` following the template in [html-template.md](html-template.md).

2. Import using `user-google_workspace` MCP `import_to_google_doc`:
   ```
   user_google_email: "ehopfenb@redhat.com"
   file_name: "OSSM PP Update [DATE] (copy into main doc)"
   file_path: "~/.workspace-mcp/attachments/ossm-pp-update.html"
   source_format: "html"
   ```

3. Return the Google Doc link to the user.

### Step 5: Summarize

Tell the user:
- Google Doc link with the formatted update
- Instruction: "Copy-paste (Cmd+A, Cmd+C, Cmd+V) into your [main Product Pages doc](https://docs.google.com/document/d/1A-v7ozJ0DE1c9o9TdV4s5tahw3ChZqxlcMw5xUgAjNY/edit) under the 2026 header, above the previous entry."
- Key highlights: status changes, new blockers, items needing attention
- Date of the last Product Pages TPM status post (so they know if it needs updating on Product Pages too)
