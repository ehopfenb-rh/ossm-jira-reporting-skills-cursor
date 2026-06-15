---
name: ossm-release-progress
description: >-
  Track OSSM release completion percentage from a Jira Epic filter. Generates
  donut charts showing epic-level and all-issues completion with full Jira
  status breakdown, then uploads to Google Drive. Use when the user asks about
  release progress, completion percentage, or how far along a release is.
---

# OSSM Release Progress

Query a Jira Epic filter, calculate completion percentages by status, generate multi-status donut charts, and upload to Google Drive.

## Context

- **Jira project:** OSSM on `redhat.atlassian.net`
- **Cloud ID:** `redhat.atlassian.net`
- **Done statuses:** "Release Pending", "Closed"
- **Recipient:** Emily Hopfenberg (`ehopfenb@redhat.com`)

### Release Filters

Update the filter ID and version label when tracking a new release.

| Release | Filter ID | Filter URL |
|---------|-----------|------------|
| OSSM 3.4.0 | 104666 | https://redhat.atlassian.net/issues/?filter=104666 |

## Procedure

### Step 1: Query epics from the filter

Use `searchJiraIssuesUsingJql` on `plugin-atlassian-atlassian` MCP:

```
cloudId: "redhat.atlassian.net"
jql: "filter = 104666"
maxResults: 100
fields: ["summary", "status", "issuetype"]
```

Collect all epic keys and their `status.name` values.

### Step 2: Query children for each epic

For each epic key from Step 1, query its children:

```
cloudId: "redhat.atlassian.net"
jql: "\"Epic Link\" = <EPIC_KEY>"
maxResults: 100
fields: ["summary", "status", "issuetype"]
```

Run these queries in parallel where possible. Collect `status.name` for every child issue.

### Step 3: Calculate completion stats

Count issues by status for both groups (epics only, and all issues combined).

**Done** = status is "Release Pending" or "Closed".

Compute:
- **Epic completion:** done_epics / total_epics
- **All issues completion:** (done_epics + done_children) / (total_epics + total_children)

Also build a `Counter` (or dict) of status → count for each chart.

### Step 4: Generate charts

Run a Python script to create multi-status donut charts with matplotlib. Each wedge represents a distinct Jira status with its own color. The center shows the "done" percentage.

```python
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt
import matplotlib.patches as mpatches
from datetime import date

today = date.today().strftime('%B %d, %Y')

# Status color mapping — ordered from done → in progress → to do
status_colors = {
    'Closed': '#1b5e20',
    'Release Pending': '#4caf50',
    'ON_QA': '#ff9800',
    'Code Review': '#f57c00',
    'In Progress': '#2196f3',
    'Refinement': '#90caf9',
    'New': '#b0bec5',
    'Backlog': '#eceff1',
}

# Display order (done first, then in-progress, then to-do)
status_order = ['Closed', 'Release Pending', 'ON_QA', 'Code Review',
                'In Progress', 'Refinement', 'New', 'Backlog']

# Fill in from Step 3
epic_data = {<status: count, ...>}   # e.g. {'In Progress': 7, 'Release Pending': 4, ...}
all_data = {<status: count, ...>}    # combined epics + children

def get_ordered(data):
    sizes, colors, labels = [], [], []
    for s in status_order:
        if s in data and data[s] > 0:
            sizes.append(data[s])
            colors.append(status_colors[s])
            labels.append(s)
    return sizes, colors, labels

epic_sizes, epic_colors, epic_labels = get_ordered(epic_data)
all_sizes, all_colors, all_labels = get_ordered(all_data)

epic_total = sum(epic_data.values())
epic_done = epic_data.get('Closed', 0) + epic_data.get('Release Pending', 0)
epic_pct = epic_done / epic_total * 100

all_total = sum(all_data.values())
all_done = all_data.get('Closed', 0) + all_data.get('Release Pending', 0)
all_pct = all_done / all_total * 100

fig, axes = plt.subplots(1, 2, figsize=(13, 5.5))
fig.suptitle(f'OSSM 3.4.0 Release Progress — {today}', fontsize=14, fontweight='bold', y=0.98)

wedges1, _ = axes[0].pie(epic_sizes, colors=epic_colors,
                          startangle=90, wedgeprops=dict(width=0.4, edgecolor='white', linewidth=1.5))
axes[0].text(0, 0, f'{epic_pct:.0f}%', ha='center', va='center', fontsize=28, fontweight='bold', color='#1b5e20')
axes[0].set_title(f'Epics\n{epic_done} of {epic_total} Done', fontsize=12, pad=10)

wedges2, _ = axes[1].pie(all_sizes, colors=all_colors,
                          startangle=90, wedgeprops=dict(width=0.4, edgecolor='white', linewidth=1.5))
axes[1].text(0, 0, f'{all_pct:.0f}%', ha='center', va='center', fontsize=28, fontweight='bold', color='#1b5e20')
axes[1].set_title(f'All Issues (Epics + Children)\n{all_done} of {all_total} Done', fontsize=12, pad=10)

# Legend showing all statuses present in either chart
all_statuses_used = set(epic_labels + all_labels)
legend_patches = [mpatches.Patch(color=status_colors[s], label=s)
                  for s in status_order if s in all_statuses_used]
fig.legend(handles=legend_patches, loc='lower center', ncol=4, fontsize=9,
           frameon=False, bbox_to_anchor=(0.5, 0.0))

plt.tight_layout(rect=[0, 0.08, 1, 0.94])
plt.savefig('/tmp/ossm_release_progress.png', dpi=150, bbox_inches='tight', facecolor='white')
```

If new statuses appear that aren't in `status_colors`, assign them a neutral color and add to the map.

### Step 5: Upload chart to Google Drive

Copy the PNG to the MCP-allowed directory and upload:

```
cp /tmp/ossm_release_progress.png ~/.workspace-mcp/attachments/ossm_release_progress.png
```

Upload using `create_drive_file` on `user-google_workspace` MCP:
```
user_google_email: "ehopfenb@redhat.com"
file_name: "OSSM 3.4.0 Release Progress — <TODAY>.png"
mime_type: "image/png"
fileUrl: "file:///Users/ehopfenb/.workspace-mcp/attachments/ossm_release_progress.png"
```

Optionally create a Google Doc with `create_doc` containing a text summary and instruct the user to insert the image from Drive.

### Step 6: Summarize

Tell the user:
- Google Drive link to the chart PNG
- Epic completion percentage
- All-issues completion percentage
- Full status breakdown
- Instruction: insert image from Drive into their target Google Doc
