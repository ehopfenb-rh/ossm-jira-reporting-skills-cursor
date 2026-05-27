# OSSM Jira Reporting Skills for Cursor

Cursor Agent Skills for automated status reporting on the **Red Hat OpenShift Service Mesh (OSSM)** program.

**Author:** Emily Hopfenberg (ehopfenb@redhat.com)

## Skills

| Skill | Description |
|-------|-------------|
| [ossm-weekly-status](ossm-weekly-status/SKILL.md) | Generate a weekly status report from Jira and Product Pages, export to Google Docs and email |

## Setup

Clone this repo into your Cursor skills directory:

```bash
git clone https://github.com/ehopfenb-rh/ossm-jira-reporting-skills-cursor.git ~/.cursor/skills/ossm-jira-reporting-skills-cursor
```

All skills inside will be automatically available in every Cursor workspace.

### Prerequisites

The following MCP servers must be configured in Cursor:

- **Atlassian** (Jira access to the OSSM project)
- **Product Pages** (`productpages` MCP server)
- **Google Workspace** (for Google Docs export and email)

## Usage

Open any Cursor chat and say:

- "Run my OSSM status report"
- "Generate OSSM weekly status"
- "Pull my OSSM program update"

The skill will pull data from Jira and Product Pages, generate an interactive canvas, export a formatted Google Doc, and provide the link.

## Contributing

This is a team repo. To add or edit skills:

1. Clone the repo and open it in Cursor
2. Edit the `SKILL.md` files directly
3. Commit and push — changes take effect for everyone on their next `git pull`

When adding a new skill, create a new directory with a `SKILL.md` file following the [Cursor skill format](https://docs.cursor.com).
