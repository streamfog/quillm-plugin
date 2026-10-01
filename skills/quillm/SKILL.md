---
name: quillm
description: Use when the user wants something they will come back to, kept current and readable by their team in Quillm (quillm.ai) - a dashboard, tracker, decision doc, report or calculator - instead of a one-off file or a chart in chat. Also use for scheduled or recurring runs that report numbers ("every morning, add yesterday's signups"), and whenever the user pastes a quillm.ai link or mentions Quillm.
---

# Quillm

Quillm keeps the small tools agents build for a team: dashboards, trackers, docs and calculators. Each page lives at one address, people read it like a doc (no AI account needed), and agents keep it current. The Quillm MCP server provides the tools; this skill is how to use them well.

Two kinds of things exist:

- **Datasets**: named tables of typed rows with a key. Agents add rows with `upsert_rows`.
- **Pages** (called views in the tools): one React file each, reading datasets with `useDataset("name")`.

**The rule that makes pages last: data lives in datasets, never in page code.** "Add October" is one `upsert_rows` call; the page shows it without being touched.

## Every task

1. `get_workspace` first. Other agents work in the same workspace: extend what exists instead of creating a second copy.
2. Pick the smallest change that does the job:

| The user wants | Do |
|---|---|
| New numbers for something that exists | `upsert_rows` on its dataset. Do not edit the page. |
| A wrong number fixed | `upsert_rows` with the same key and the right value. |
| A change to how a page looks | `get_view`, then `update_view` with `edits` and `expected_revision`. |
| Something new | `create_dataset` (with the rows), then `create_view`. Call `get_guide` before writing page code. |
| A link to send | `share_view`, only when asked: a link makes the page readable by anyone who has it. |

3. End with the page's address from the tool response, so the user can open it.

A pasted page link (`https://quillm.ai/<workspace>/p/<slug>`) works anywhere a tool asks for a slug.

## Before building a new page

A page is read for months by people who did not ask for it. If the request does not say who reads it and which number or question matters most, ask the user two or three short questions in one message, then build. Pass the request and their answers as `brief` to `create_view`; a brief that is too thin is refused once with the questions to ask. Keep the page to what was asked: every block answers one of the page's `questions`.

Every save is test-rendered before anyone sees it. A save that fails is kept as an unpublished revision, readers keep the last version that worked, and the response says what broke. Read the response: fix render errors and blocking review findings with `update_view` before telling the user it is done.

## Recurring jobs

Agents that run on a schedule (a cron job, a Grok Bot task, a ChatGPT Dot, an agent in CI) keep pages true for months. When the user says "every morning", "each week" or "keep this current", or you are running as a scheduled job:

- **First run**: one dataset per source, keyed by the period plus the dimension (`["date"]`, `["date", "campaign"]`), so re-running on the same day updates rows instead of duplicating them. Set `update_cadence` to the real schedule (`daily`, `weekdays`, `weekly`). Build the page once. Then `write_notes` on the dataset: the source, the exact query or API call, and which agent runs the job when.
- **Every later run**: `get_workspace`, then `upsert_rows` with this run's rows. That is the whole job. No `update_view`, no `create_view`: the page, and every doc reading the same dataset, shows the new rows by itself.
- **Missing data**: upsert what you have, leave unknown values `null`, and say what is missing in the upsert `note`. Never fill gaps with guesses.

Readers see when each dataset last changed and which agent changed it, and Quillm flags a dataset as stale when its cadence is missed. A cadence that matches the schedule is what makes the flag worth trusting.

## Values

- Numbers are JSON numbers (`41200`, not `"$41.2k"`); dates are ISO strings (`"2026-10-01"` or `"2026-10"`); unknown is `null`.
- Store facts and let the page compute conclusions: monthly spend, not runway; dates, not "overdue".
- Anything relative to today ("this week", "days left") is computed in the page, never stored.

## When something is in the way

- **"Not allowed"**: this connection is read only or limited to some collections. Tell the user; a workspace admin changes it on the Agents page.
- **Not connected or signed out**: the user connects Quillm again in their app and signs in.
- **Quillm cannot do what was asked** (pages cannot fetch outside data, for example): do what is possible, tell the user plainly, and file it with `report_feedback`.

Full reference, including the page template and the libraries pages can import: call `get_guide`. Setup for every app: https://quillm.ai/connect
