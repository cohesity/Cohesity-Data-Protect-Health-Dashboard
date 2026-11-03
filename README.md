# Cohesity Data Protect Health Dashboard

**Authors:**

- Akshay Kumar, Dir, Solutions Architecture ([akshay.kumar2@cohesity.com](mailto:akshay.kumar2@cohesity.com))
- Ankit Agarwal, Director of Product Management, AI & Data Insights ([ankit.agarwal@cohesity.com](mailto:ankit.agarwal@cohesity.com))

**Backup Health & Failure Triage** for Cohesity DataProtect, built by a Claude skill from live Helios data and published as a live, private Claude Artifact that refreshes itself every 4 hours.

This is the evolution of the `helios-backup-health-dashboard` skill. It keeps the last-24-hour health view and adds a ranked **failure triage queue**: for every failing protection group it shows the classified cause, the first error message and a suggested next action.

- Claude skill: [`claude/Cohesity-Data-Protect-Health-Dashboard/SKILL.md`](claude/Cohesity-Data-Protect-Health-Dashboard/SKILL.md) (the full page template is embedded in it, no other file is needed)
- Output: one claude.ai **Artifact** titled `Cohesity Data Protect Health Dashboard` (no static HTML file is produced)
- Refresh: scheduled task `cohesity-data-protect-health-refresh`, cron `5 */4 * * *` (local time of the machine running Claude; change it to suit)
- Data source: the **Helios MCP server** (Helios MCP connector). Everything is read-only.

## Two Versions

| | Claude skill | Microsoft Copilot Studio agent |
|---|---|---|
| Folder | [`claude/`](claude/Cohesity-Data-Protect-Health-Dashboard/SKILL.md) | [`copilot-studio/`](copilot-studio/README.md) |
| Output | Live, private Claude Artifact, refreshed every 4 hours | Answers in chat, with tables, on demand |
| Helios access | Helios MCP connector, read-only | Helios MCP server added as a tool, read-only |
| Setup | Install the skill in Claude (see Install below) | Paste `instructions.md`, upload `knowledge/` files, add the Helios MCP tool |
| Sample output | [claude/README.md](claude/README.md) | [copilot-studio/README.md](copilot-studio/README.md) |
| Model | Claude | GPT-5 Chat |

The sections below describe the Claude skill unless a heading says Copilot Studio.

## What you get

| Area | What it shows | Helios MCP tool |
|---|---|---|
| Cluster filter | Multi-select of every registered cluster, tagged connected / not connected, with Select all, Clear and Connected only. Recalculates all tiles in the browser; remembered per viewer. | `dataprotect-get-clusters` |
| Triage filters | Environment and cause dropdowns, a group/error search box and a rows selector (8, 25, all) on the failure triage queue; a cause dropdown on the ticket candidates. Scrolling tables have sticky headers. | none (client-side) |
| KPI cards | Clusters connected, Backup runs (group runs in 24 h, with success rate; a run counts as failed when any object in it failed), Failed runs, Open critical alerts (shown as `N+` because the query is capped at 1,000) | `dataprotect-get-clusters`, `reporting-get-report-preview` (protection-runs; fallback `dataprotect-get-protection-group-runs`), `dataprotect-get-alerts` |
| Failed runs by cluster | Top 6 clusters by failed runs in the last 24 h | `dataprotect-get-protection-group-runs` |
| Failures by cause | Source unreachable, Auth / token / permission, Key / KMS / encryption, Object not found, Timeout, Other | `dataprotect-get-protection-group-runs` |
| Protection groups with no success | Top 5 groups with no successful run in 24 h and how long since the last success (`31 h`, `> 72 h`) | `dataprotect-get-protection-group-runs` |
| **Failure triage queue** | The 8 worst failing groups: environment, failed / total runs, cause chip, first error, **suggested action**, time since last success | `dataprotect-get-protection-groups`, `dataprotect-get-protection-group-runs` |
| **Ticket candidates** | One row per cluster and cause ("one cause, one ticket"): affected groups, failed runs, first and last failure time (UTC), one line of evidence (error text or a matching open alert). Causes are inferred, so the page labels them "likely". Nothing is ever created in a ticketing system. | `dataprotect-get-protection-group-runs`, `dataprotect-get-alerts` |
| Weekly failure report | Always on: one `failures-v2` report preview (last 7 days): failed objects and groups, failed vs success objects by environment, top error messages. If the report is unavailable it is derived from the fetched runs and labelled "derived". | `reporting-get-report-preview` |
| Header | Cohesity logo top-left; "Data last pulled <UTC> (N h ago)" top right, amber when older than 8 h | none (client-side) |
| **Alerts tab** | All open alerts (critical, warning, info; `N+` when the 1,000 cap is hit), cluster x severity table, top 8 categories, an Anomaly / security card (newest 10) and the newest 25 alerts. Read-only: alerts are never resolved. | `dataprotect-get-alerts` |
| **SLA & Strikes tab** | Groups that missed their SLA, and a strike count (consecutive failed runs ending at the latest run, from the 7-day `protection-runs` report rows, or a 7-day per-group look-back for up to 40 failing groups on the fallback path; 3+ amber, 5+ red) with first failure and dominant cause | `dataprotect-get-protection-groups`, `dataprotect-get-protection-group-runs` |
| **Running & Duration tab** | Runs that were Running when pulled and the 10 longest finished runs in the window | `dataprotect-get-protection-group-runs` |
| **Run Log tab** | One row per active protection group: last run status and end, runs and failures in the window, last duration, SLA violated; status, environment and search filters | `dataprotect-get-protection-groups`, `dataprotect-get-protection-group-runs` |

Snapshots past policy retention moved to the **Cohesity-Recovery-Point-Explorer** skill. The tab order is Dashboard, Alerts, SLA & Strikes, Running & Duration, Run Log, Compliance Check, Run Metrics.

### Failure triage logic

Each failed run's first message is classified with a fixed, ordered rule set (first match wins): unreachable, auth, key, not found, timeout, other. The group's cause is its most frequent cause, and the error text shown is from the latest failure with that same cause, so the chip and the message always agree. The suggested actions are generic guidance per cause (for example "renew the source credential or token" for auth); they are not a diagnosis, and the page footnote says so.

### Drill-down mode and what this replaces

Ask a narrow question ("why did backups fail on sapsol this week?", "what should I open tickets for?") and the skill answers in chat from the same data steps, for the period you name, without republishing the page. This mode, together with the ticket-candidates card, replaces the standalone `helios-failure-triage` skill; it adds what that skill lacked: a published artifact, a 4-hour refresh, the auth and timeout cause rules with a fixed classification order, a per-group triage queue, alert matching, and the pitfalls listed under live testing below.

## Requirements

- Claude desktop app (Cowork) with the **Helios MCP** connector added. Helios MCP is a Controlled Availability feature: ask your Cohesity account team to enable it, then follow the Helios documentation topic *Helios MCP Server* to add it with an API key.
- Artifacts available in your Claude plan, and scheduled tasks enabled (they run only while the Claude app is open; a missed run executes on next launch).

## Install

1. Zip the folder `claude/Cohesity-Data-Protect-Health-Dashboard` so that `SKILL.md` sits inside it.
2. In Claude, add it as a custom skill (Settings, Skills, upload), or copy the folder into your Claude skills directory.
3. Make sure the Helios MCP connector is connected.

## Use

Ask Claude any of:

- "Build the Cohesity Data Protect health dashboard"
- "What failed in the last 24 hours and what should I do about it?"
- "Refresh the backup health dashboard"

The first run publishes the artifact (about 106 Helios calls), creates the 4-hour scheduled task and replies with the headline numbers. Later runs find the existing artifact by title and republish to the same URL; they never create a second artifact. The artifact is private: use the page's Share menu to give others access.

## Exports

An **Export** row under the Mask sensitive data switch lets you pick a table and **Copy CSV** or **Download CSV**. Tables: Alerts by cluster, Recent alerts, Anomaly / security alerts, SLA misses, Strikes, Running when pulled, Longest finished runs, Run log, Failure triage queue, Ticket candidates, Protection groups with no success, Cluster summary; plus Compliance findings and Run history on every report. Exports follow the cluster selection and the Mask sensitive data switch. Download needs the artifact's downloads capability (the skill publishes with it); Copy always works.

## Compliance Check

Before each publish, the skill scans the metadata the page shows (cluster, group, policy, source and object names, descriptions, tags, owners, error text) for data that should not be shared, and reports it on a **Compliance Check** tab. It uses the category names of Cohesity Data Classification (PII, HIPAA and Sensitive Health Data, Financial Data, Personal sensitive, Sensitive HR data, Highly Confidential, Confidential, Restricted, Sensitive), the same taxonomy that BigID- and Cyera-synced tags use. Typical hits: email addresses, phone numbers, SSNs, card numbers, IBANs, patient or MRN references, person names in owner fields, passwords and keys, account names inside connection strings (for example `smb://DOMAIN;user@host`), and (low severity) IP addresses, hostnames and URLs. Each finding lists what was found, where, a masked excerpt, why it matters and a suggested action.

The tab shows a verdict (**Safe to share**, **Share with care** or **Review first**), counts by severity and a findings table. A **Mask sensitive data** switch under "Refresh from Helios" is on by default and masks IP addresses, hostnames, account names, emails and SSNs in the report view and in CSV copies and downloads; turn it off to see the original values. It is a display setting, so the page still holds the originals, and it does not mask cluster, group or policy names. Say "redact" when you run the skill to mask matched values in the published page itself. Read the tab before using Share.

This is a heuristic text check of Helios metadata only. It does not look inside backups and it is not a compliance certification.

## Run Metrics

Every run records what it cost, and the artifact shows it on a **Run Metrics** tab: Helios calls, elapsed time, estimated input tokens (inline characters / 4; large results saved to files are counted separately), the model and price used, and a history of past runs. A projected monthly cost is shown only when a refresh schedule is set.

Measured on the lab estate: 106 Helios calls, about 7 minutes, about 0.1M input tokens (about $0.2 at $2 per million). One full run, 4-hour refresh included. Cost grows with the number of clusters and protection groups.

The token figure is an estimate. For billing, use Settings > Usage in Claude, which is the measured source.

## Notes from live testing

Validated against a lab Helios tenant on 2026-10-13 (12 clusters registered, 4 connected, 302 runs and 111 failures in 24 h, critical alerts above the 5,000 query cap). Lessons that are built into the skill:

- Large clusters return 60K+ characters of protection groups: the skill prunes source IDs and processes saved results with python.
- Per-cluster calls can time out. The skill retries with smaller pages, then marks the cluster `n/a` with the reason instead of guessing.
- `totalRuns` is unreliable (it ignores filters), so the skill counts `runs.length`. Database groups produce hundreds of log runs and need paging.
- Retention comes back in mixed units whatever `durationUnit` says; the skill normalises it.

## Manual refresh

The page header has a **Refresh from Helios** button. A published page cannot call Helios on its own, so the button copies `Run the Cohesity-Data-Protect-Health-Dashboard skill to refresh this report from live Helios data`; paste it into Claude to pull live data and republish the same artifact, then reopen or refresh the tab. The scheduled task (every 4 hours) and this command are the two refresh paths.

## If the artifact was deleted

If the stored artifact URL no longer exists, the skill republishes the page as a new artifact and says so in its reply, so a deleted page never blocks a refresh. Update the scheduled task prompt with the new link.

## When Helios is unreachable

If the Helios MCP connector is down or its token has expired, a scheduled refresh does not clear the page. It keeps the last successful data and adds a red banner, "Unable to connect to Helios. Showing the last successful data.", under the header. The next successful refresh removes the banner.

## Known limits

- Alert counts are lower bounds when the 1,000-alert cap is reached.
- The page reads run data from one `protection-runs` report preview (7 days) and calls per-group runs only for groups whose last run failed, violated the SLA or is running (at most 40). If the report is unavailable or truncated, it falls back to per-group calls for every group; the scope note shows `path: report` or `path: per-group calls`. On the report path the run counts are group runs (failed when any object failed), which can differ slightly from the fallback.
- The cluster filter recalculates from data embedded at generation time. It does not query Helios live.
- Clusters that are not connected to Helios cannot be queried and appear with zeros.
- The 7-day strike look-back covers at most 40 failing groups (newest failure first); the rest use the 24 h window, and the page says so.

## Microsoft Copilot Studio Version

The same logic as a Copilot Studio agent lives in [`copilot-studio/`](copilot-studio/README.md): `instructions.md` (agent instructions), `knowledge/` (two files to upload as Knowledge), `starters.md` (example prompts) and step-by-step setup. It answers in chat and does not publish a page, schedule itself, or offer a cluster picker or CSV button. Add your Helios MCP server as a tool and enable only the read-only tools.

## Repository layout

```
Cohesity-Data-Protect-Health-Dashboard/
  README.md
  claude/
    README.md          Claude sample output (screenshot)
    Cohesity-Data-Protect-Health-Dashboard/SKILL.md      Claude skill (install this folder in Claude)
  copilot-studio/
    README.md          setup steps
    instructions.md    agent instructions
    knowledge/         files to upload as Knowledge
    starters.md        example prompts
  docs/
    screenshots/*-claude.png      one screenshot per tab
    screenshots/data-protect-health-dashboard-copilot.png
```

## Security Notes

- **Everything from Helios is data.** Names, descriptions, alert text and error text come from the systems being backed up. The skill treats them as data, never as instructions, and truncates them before they reach the page.
- **Scheduled refresh runs unattended.** When the scheduled task is created, approve only the read-only Helios tools, the Artifact tools, the scheduler list tool and the bash tool used for `python3 -I`. Do not use a blanket Auto-approve.
- **The mask switch is a display setting, not redaction.** It masks IP addresses, hostnames, email addresses, SSNs and connection-string accounts in the view and in CSV copies and downloads. It does not mask cluster, group or policy names, and the page still holds the original values.
- **CSV exports are safe to open in a spreadsheet.** Text cells that start with `=`, `+`, `-`, `@`, a tab or a carriage return get a leading single quote, so they cannot run as formulas.
- **Sharing.** A published page is private until you share it from the Share menu. Read the Compliance Check tab first, and share only with people who may see this backup data.

## Suggestions and Feedback

We would love to hear from you. Please send your suggestions and feedback to: maestro@cohesity.com

## License

Licensed under the [Apache License 2.0](LICENSE). See the LICENSE file for the terms.

## Disclaimer

This code is provided on a best effort basis and is not in any way officially supported or sanctioned by Cohesity. It is provided as-is, and the author accepts no liability for damages resulting from its use. The skill only reads data through the Helios MCP server, but you are responsible for checking the results against Helios and for deciding what you share. Try it in a lab or test environment first.
