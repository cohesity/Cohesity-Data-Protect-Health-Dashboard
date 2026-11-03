# Failure Triage Methodology

## Window and counting
- Default window: last 24 hours, UTC. The user may name another period.
- Backup runs = finished runs in the window (Succeeded, SucceededWithWarning, Failed). Failed runs = local backup status Failed.
- Success rate = (runs - failed) / runs.
- Paused groups are skipped for run collection. Groups that are paused are not counted as failures.

## Cause classification
Look at the first message of each failed run, case-insensitive. The first matching rule wins, in this order.

| Order | Cause | Match any of |
|---|---|---|
| 1 | unreach (source unreachable) | connect, unreachable, not reachable, cannot be reached, resolve, network, no route, agent |
| 2 | auth (authentication) | token, AADSTS, authentication, credential, unauthorized, access denied, permission, certificate |
| 3 | key (encryption or key management) | KMS, the word key, encrypt (also encrypted, encryption) |
| 4 | notfound (entity missing) | not found, doesn't exist, missing from EH, deleted, no eligible sources |
| 5 | timeout | timed out, timeout |
| 6 | other | everything else |

A message such as "source not reachable ... credentials ... timed out" counts as unreach because it matches first.

## Suggested action per cause (generic guidance, not a diagnosis)
| Cause | Suggested action |
|---|---|
| unreach | Check the source host or vCenter is reachable from the cluster: DNS, firewall ports and the agent or service state. |
| auth | Renew or re-enter the source credentials or token, and check the account's permissions and certificate expiry. |
| key | Check the key management service (KMS) connection and the encryption settings of the source. |
| notfound | The protected entity is missing from the source inventory. Refresh the source registration and remove or re-add the entity. |
| timeout | Check load and network latency on the source and cluster, and whether schedules overlap. |
| other | Read the full error in Helios and open a support case if it is unclear. |

## Triage queue
- One row per protection group with at least one failed run in the window, ranked by failed runs. Show the top 8.
- Dominant cause = the most frequent cause among that group's failed runs.
- Error text = the first message of the most recent failed run whose cause equals the dominant cause, so the cause and the text agree. Collapse whitespace, remove email addresses and full GUIDs, cut to 160 characters.
- Last success = hours since the last Succeeded or SucceededWithWarning run. Search back to 72 hours, else "> 72 h". Use "ok < 24 h" if it also succeeded in the window.

## Ticket candidates
One row per cluster and cause, so one cause opens one ticket. Columns: affected groups (distinct), failed runs, first and last failure (UTC), one line of evidence (most recent failure of that cause, cut to 140 characters), and a matching open alert if one fits. Candidates only. Do not create tickets.

## Numbers must tie
For any cluster selection, the failures by cause sum to the failed runs. The triage and ticket failed counts for a cluster each equal that cluster's failed runs.

## Snapshots past retention (optional)
Normalise policy retention to days (Weeks x 7, Months x 30, Years x 365) and take the longest of base and extended retention. Flag groups whose snapshots are older than that. State how many groups were sampled. If not done, say "n/a: not assessed".
