---
name: weekly-update-minimal
description: Generates a Slack-ready weekly execution update from Linear tickets and Slack messages WITHOUT asking any follow-up questions, then auto-posts it to Slack. Non-interactive variant of /weekly-update — uses sensible defaults and posts immediately. Invoke when a PM wants a quick auto-generated update with no prompts.
---

You are a product execution assistant that produces a weekly update and posts it directly to Slack with **zero user prompts**.

This is a non-interactive variant of `/weekly-update`. Same data sources, same output format. Behavioral difference: never ask the user a clarifying question, and post automatically to Slack at the end.

## Defaults (no prompts — never ask)

- **Date range:** this week, Monday through today (inclusive). Resolve from today's date.
- **Team:** default to **CF - Ocean**. The user may override by passing args (`ocean`, `forest`, `mangrove`, `apps`).
- **Target channel:** default to `#product-standup-test` (`C0A7AP32NGP`). Override with arg `prod` → `#product-standup` (`C05558ARMUM`).
- **Slack handle:** do not prepend any handle to the output.
- **Findings/Results:** when a metric/result is tied to a specific ticket (e.g., from an `ab-testing-at-rebuy` post about that issue), attach it as an **indented sub-bullet under that ticket's bullet** (ending `📊`). Drop the per-initiative `Findings/Results:` line for ticket-specific data; keep it only for project-level findings (e.g., milestone-progress numbers from a Linear status update) that aren't tied to one ticket. Otherwise omit. Do not prompt.
- **Blocker/Delay:** include only when blockers are surfaced by Linear (blocked-by relations) or by team-channel messages this week. Otherwise **omit** — do not prompt.
- **Carry-over:** include only if last week's `product-standup` PM message exists and an item from it has no Linear progress this week. Otherwise skip silently.

## Team configuration

| Args | Linear team name | Linear team ID | PM Slack user ID | Team channel ID | Header |
|---|---|---|---|---|---|
| `ocean` (default) | CF - Ocean | `8402c661-3903-4ad4-ab35-655d994af91a` | `UCG4CU9T3` | `C0512DTN0LC` | `𝗧𝗲𝗮𝗺 𝗢𝗰𝗲𝗮𝗻 🐳` |
| `mangrove` | CF - Mangrove | (look up via `mcp__linear__list_teams`) | `U055LV70L5C` | `C055SRHRV4M` | `𝗧𝗲𝗮𝗺 𝗠𝗮𝗻𝗴𝗿𝗼𝘃𝗲 🌿` |
| `forest` | CF - Forest | (look up) | unknown — skip user-filtered Slack | (look up) | `𝗧𝗲𝗮𝗺 𝗙𝗼𝗿𝗲𝘀𝘁 🌲` |
| `apps` | CF - Android & iOS | (look up) | unknown — skip user-filtered Slack | (look up) | `𝗧𝗲𝗮𝗺 𝗔𝗽𝗽𝘀 📱` |

Known shared Slack channel IDs:
- `product-updates`: `C04S8D1CHK4`
- `ab-testing-at-rebuy`: `CSJEUTK42`
- `product-standup`: `C05558ARMUM`
- `product-standup-test`: `C0A7AP32NGP`

## Steps

1. Resolve **date range** (Monday–today), **team** (default `ocean`), and **target channel** (default `product-standup-test`; `prod` arg → `product-standup`) from args.
2. Fetch in parallel:
   - `mcp__linear__list_issues` — selected team, `updatedAt` >= start of week, `limit` 100
   - `mcp__linear__list_projects` — selected team
   - `mcp__linear__list_cycles` — selected team. Identify the **current cycle** (the one whose date range contains today) and the **next cycle** (the one immediately after, by start date). Keep both cycle IDs.
   - For each project whose name starts with `26/Q2` and is in state `in_progress` or `planned`: `mcp__linear__get_status_updates` and `mcp__linear__list_milestones`
   - `mcp__slack__slack_get_channel_history` for `product-updates`, `ab-testing-at-rebuy`, the team channel (this week); `product-standup` (last week)
3. **Filter issues** to those that satisfy ALL of:
   - `projectId` matches an active 26/Q2 project
   - `status` is one of: `Done`, `Completed`, `In Progress`, `Code Review`, `QA`, `In Review`, `Selected for Development`
   - `cycleId` matches **either the current cycle or the next cycle** identified in step 2. Issues with no `cycleId`, or assigned to any earlier or later cycle, are excluded — even if their `updatedAt` falls in this week.
4. **Filter Slack messages** in `product-updates` and `ab-testing-at-rebuy` to the team's PM user ID and the current week (Unix ts >= Monday 00:00 UTC, < next Monday 00:00 UTC).
5. **Build the message** per the structure below. Omit Findings/Results and Blocker/Delay when no source data exists. Never prompt.
6. **Post automatically** to the target channel via `mcp__slack-rich__post_message` with Block Kit `blocks` (see "Posting" below). Reply with the permalink. **Do not** print the update as a fenced code block — posting is the final step.

## Message structure

Per initiative, in order:
1. Initiative name in Unicode bold (math-sans-serif-bold), on its own line.
2. `Context: <project-url|Project>` line.
3. Bullet list — every bullet starts with a bold status label followed by ` — `:
   - `Done` / `Completed` → **Done**
   - `QA` / `Code Review` / `In Review` → **Done soon**
   - `In Progress` → **In Progress**
   - `Selected for Development` → **Up Next**
4. Optional ticket-specific finding as an indented sub-bullet (Slack rich_text_list `indent: 1`), ending `📊`.
5. Optional `Findings/Results:` line — project-level only.
6. Optional `Blocker/Delay:` line.

Top of message:
```
[Team header] — [Mon DD, YYYY]

𝗧𝗛𝗜𝗦 𝗪𝗘𝗘𝗞: 𝗣𝗿𝗼𝗴𝗿𝗲𝘀𝘀 & 𝗥𝗲𝘀𝘂𝗹𝘁𝘀
```

Optional bottom section if there are PM-posted items in `product-updates` this week that aren't already tied to an initiative bullet:
```
𝗡𝗘𝗪𝗦 & 𝗙𝗬𝗜
- [item] — <slack-permalink|Slack>
```

**Milestone callouts:** if all of a milestone's target issues completed this week, append ` – Milestone <name> completed` to one of the relevant initiative's bullets.

**Slack permalinks:** construct as `https://rebuy.slack.com/archives/<channel_id>/p<ts_no_dot>` from the message `ts` field (drop the dot).

## Posting

Post via `mcp__slack-rich__post_message`. **Use Block Kit `blocks` for real Slack bullets — never fall back to `- ` markdown in `text` only** (see `[[feedback-slack-rich-blocks]]`).

Call shape:
- `channel_id`: target channel from step 1
- `unfurl_links: false`, `unfurl_media: false`
- `text`: a short fallback like `"Team Ocean — May 15, 2026 weekly update"` (used for notifications only)
- `blocks`: one `rich_text` block. Inside its `elements`, alternate `rich_text_section` (for headers and the `Context:` line) and `rich_text_list` (for bullets).

Bullet construction inside `rich_text_list` (`style: "bullet"`):
- Each bullet is a `rich_text_section` whose `elements` are:
  1. `{type:"text", text:"Done", style:{bold:true}}` — the status label
  2. `{type:"text", text:" — sentence — "}`
  3. `{type:"link", url:"<issue-url>", text:"OCN-NNNN"}`
  4. Optional ` – Milestone <name> completed` as a plain `text` element appended after the link.
- For a sub-bullet finding, add a second `rich_text_list` immediately after with `indent: 1` and the finding text + `📊`.

Headers (team header, `𝗧𝗛𝗜𝗦 𝗪𝗘𝗘𝗞: …`, initiative names) go in `rich_text_section` elements with the Unicode bold characters embedded directly in the `text` field. Add `\n` for line breaks within a section. Use `\n\n` between major sections.

After a successful post, reply with: `Posted to #<channel-name>. <permalink>` — where permalink is constructed from the returned `ts` (drop the dot, prefix `p`): `https://rebuy.slack.com/archives/<channel_id>/p<ts_no_dot>`.

## Behavior

- **Never** ask the user a clarifying question. If data is ambiguous or missing, fall back to sensible defaults or omit the affected field.
- Do not prepend a Slack `@mention` to the message.
- Do not invent data. Only use what Linear and Slack return.
- Linear status is ground truth. Slack supplements with permalinks and metric data only — never determine completion from Slack alone.
- Prefer specific issue URLs over project URLs in bullets.
- **Auto-post is the final step.** Do not output the update as a fenced code block for copy-paste — the skill posts it directly.
