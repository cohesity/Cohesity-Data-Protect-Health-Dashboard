# Cohesity Data Protect Health Agent For Microsoft Copilot Studio

Backup health and failure triage for the last 24 hours, as a Copilot Studio agent. This is the same logic as the Claude skill in `../claude/`, rewritten as agent instructions and knowledge files.

## Sample output

**Prompt:** `Show backup health for the last 24 hours for cluster sac01-pm-haswell8-p1`

**Model:** GPT-5 Chat. Lab data. Captured in the Copilot Studio test pane in Light mode.

![Data Protect Health agent answer in Copilot Studio](../docs/screenshots/data-protect-health-dashboard-copilot.png)

### Notes

| Topic | Note |
|---|---|
| Format | The answer is chat text with tables. The Claude skill gives a live, filterable page. See the sample in [claude/README.md](../claude/README.md). |
| Scope | Name one cluster for the fastest answer. |
| Accuracy | The agent follows the instructions but is a language model. Compare a few answers with Helios before you share the agent. |
| Your data | Cluster and group names in the image are lab data. Your answers will differ. |

## What is in this folder

| File | Where it goes in Copilot Studio |
|---|---|
| `instructions.md` | Agent, Overview, Instructions (under the 8,000 character limit) |
| `knowledge/failure-triage-methodology.md` | Agent, Knowledge, Add knowledge, Upload file |
| `knowledge/helios-mcp-tool-notes.md` | Agent, Knowledge, Add knowledge, Upload file |
| `starters.md` | Example prompts to try in the test pane |

## Agent settings

| Setting | Value |
|---|---|
| Agent name | Cohesity Data Protect Health |
| Model | GPT-5 Chat |
| Instructions | The content of `instructions.md` |
| Knowledge | Both files in `knowledge/` |
| Web Search | Disabled |
| Tool | Helios MCP server (Model Context Protocol), read-only tools only |

## Setup

1. In Copilot Studio, choose **Agents**, then **Create blank agent**, and wait until the agent shows **Provisioned**.
2. On **Overview**, under **Select your agent's model**, select **GPT-5 Chat**.
3. Under **Instructions**, choose **Edit**, paste the full content of `instructions.md`, and choose **Save**.
4. Under **Knowledge**, choose **Add knowledge**, **Upload file**, upload both files from `knowledge/`, and choose **Add to agent**. Keep **Web Search** disabled.
5. Open the **Tools** tab, choose **Add a tool**, filter on **Model Context Protocol**, select your Helios MCP server, and choose **Add and configure**. Enable only the tools in **Tools to enable** below.
6. Choose **Test** and try a prompt from `starters.md`. Then choose **Publish** and add the channels you need on the **Channels** page.

Example prompts:

- Show backup health for the last 24 hours
- Why are backups failing? Give me the triage queue
- Which protection groups have had no successful backup?
- What tickets should I open for the failures?
- Why did backups fail on <cluster name> this week?

## Tools to enable

This agent needs these read-only Helios MCP tools. Enable them and keep every other tool disabled, including any write, acknowledge, resolve or edit tool that your connector exposes.

| Tool |
|---|
| `dataprotect-get-clusters` |
| `dataprotect-get-protection-groups` |
| `dataprotect-get-protection-group-runs` |
| `dataprotect-get-alerts` |
| `reporting-get-all-reports` |

`knowledge/helios-mcp-tool-notes.md` is shared reference text for the whole family of agents and lists more tools than this agent needs; listing a tool there does not enable it. If a test prompt fails because a tool is missing, enable only that read-only tool. Before you publish, open the Tools page and confirm that only read-only tools are on.

## Differences from the Claude skill

| Area | Claude skill | Copilot Studio agent |
|---|---|---|
| Output | Live page that refreshes every 4 hours | Chat answer with tables, on demand |
| Cluster choice | Multi-select on the page | Name the cluster in your question |
| Schedule | Scheduled task in Claude | Power Automate flow with a schedule trigger (not part of this folder) |
| Large Helios responses | Processed with code | Can be slow or hit tool limits on big estates. Ask for one cluster at a time. |
| CSV | Copy and download buttons | Ask for a CSV block in chat and copy it |
| Access | Read-only | Read-only |

## Check before you publish

Share the agent only with people who may see backup data for the connected clusters. The agent follows the instructions but is a language model, not a fixed report. Ask a few known questions first and compare the answers with Helios.

**Authentication.** Choose the connection mode of the Helios MCP tool deliberately. With a shared (maker) connection, every person who can use the published agent sees backup data for all clusters that connection can reach. Prefer per-user authentication where your tenant supports it. Otherwise restrict the agent's audience to people who may see that data, and treat the answers as visible to all of them.

**Untrusted text.** Names, alert text and error text come from the systems being backed up and can contain text written by others. The instructions tell the agent to treat them as data, but a language model can still be influenced, so check unexpected output against Helios.

**CSV.** If you paste an agent's CSV block into a spreadsheet, check that cells starting with `=`, `+`, `-` or `@` show as text, not as formulas.
