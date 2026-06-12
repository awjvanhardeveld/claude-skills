---
name: confluence-monthly-workstreams
description: Generates Confluence workstream sections from a Linear team's projects. One section per project with a roadmap table populated from Linear issues. Invoke when the user wants to seed or refresh a Confluence roadmap/workstreams page from Linear.
---

You help the user push a Linear team's projects onto a Confluence page as "workstream" sections, one per project, each with a roadmap table seeded from the project's Linear issues.

Refer to `template.md` for the exact output structure (heading, status line, table columns).

## Step 1 — Select team

Ask the user which Linear team to pull projects from. Use `mcp__linear__list_teams` and present the names. Resolve to a team ID before continuing.

If the user says "Lake", treat that as the Apps team (CF - Android & iOS) — see the auto-memory entry `reference_lake_apps_alias`.

## Step 2 — Select Confluence page

Ask the user which Confluence page to append to. Accept any of:
- A page URL (extract the page ID from `/pages/<id>/`)
- A page title (use `mcp__confluence__search_pages` to find it; if multiple matches, ask the user to disambiguate)
- A page ID directly

Verify with `mcp__confluence__get_page` (request `body-format: "storage"`) and confirm the page title with the user before writing.

## Step 3 — Fetch Linear data

In parallel:

1. `mcp__linear__list_projects` scoped to the chosen team. Include only projects in state `planned`, `started`, or `in progress` (skip `completed`, `cancelled`, `paused`) — confirm scope with the user if unsure.
2. For each returned project, in parallel:
   - `mcp__linear__get_project` for the project's `icon`, `description`, `content`, `url`, and any goal/KR fields.
   - `mcp__linear__list_issues` filtered to the project. **Apply three exclusion filters** client-side; an issue is included only if it passes all three:
     1. **`hide-roadmap` label** — strictly exclude every issue carrying it (`"hide-roadmap" in issue.labels`, case-sensitive). The team uses the label as the deliberate "don't surface on the roadmap" signal. See feedback memory `feedback_workstreams_hide_roadmap`.
     2. **`Rejected` status name** — exclude any issue whose Linear status name is exactly `Rejected`. Rejected tickets are dead-end work that didn't ship; surfacing them on the roadmap creates noise about "failed" experiments that the team has already moved on from.
     3. **`Backlog` status name** — exclude any issue whose Linear status name is exactly `Backlog`. Backlog tickets aren't committed; including them would make the table look broader than what the team is actually working on this quarter.

     Note: also exclude `archived` issues automatically (the Linear MCP usually does this by default, but verify). Do **not** exclude `Duplicate` or `Cancelled` automatically unless the user asks — the user explicitly named only Rejected and Backlog as the new filters on 2026-05-27.
   - **On re-runs against an existing page, also remove rows for any issue that *now* carries `hide-roadmap` (or is now Rejected/Backlog/archived) but didn't before** — don't just refrain from adding new ones. Labels and statuses evolve between runs, and the existing page may have stale rows from a prior fetch. Identify them by scanning the existing table cells for `OCN-XXXX` (or equivalent identifiers), matching against the freshly-fetched exclusion sets, and deleting those `<tr>` blocks. Do this in both Mode A (before appending new sections) and Mode B (during the surgical-refresh pass).
3. For each non-`hide-roadmap` issue, resolve the **first image URL** (used in the More Info column). Source preference, in order:
   - **For AB-test tickets, prefer a Slack-sourced image** from the channels `#ab-testing-at-rebuy` (`CSJEUTK42`) and `#product-updates` (`C04S8D1CHK4`) — those posts include polished Amplitude charts / before-after mockups that read better on a stakeholder roadmap than the raw screenshots embedded in the Linear ticket. See feedback memory `feedback_workstreams_slack_images` for the matching heuristic, scope requirements, and the direct API path. Most tickets are **not** cited by `OCN-ID` in Slack, so match by title-keyword fuzzy overlap and ask the user to confirm ambiguous cases.
   - **Linear fallback** when no Slack post is found: prefer `mcp__linear__extract_images` if it returns ordered image URLs; otherwise parse `issue.description` for the first `![…](url)` or `<img src="url">`, then fall back to the first image-type attachment, then the first image in comments. Stop at the first hit.
   - If no image exists in any source, mark the issue as image-less; the table row will omit the thumbnail `<p>` and keep only the ticket-id link.

## Step 3.5 — Upload each image as a Confluence attachment

For every issue that has a resolved image URL, upload the image to the target Confluence page **before** building/updating the page body. This ensures the page can reference each image as a stable Confluence attachment (no Linear auth needed by readers).

For each image:
- **Linear-sourced image** (Confluence downloads server-side):
  - Call `mcp__confluence__attach_file` with:
    - `page_id`: the target Confluence page ID (from Step 2)
    - `source_url`: the resolved image URL (signed `uploads.linear.app` URLs work — Confluence fetches them server-side before the signature expires)
    - `filename`: `<issue.identifier>.<ext>` (e.g. `ENG-123.png`). Derive `<ext>` from the URL path; default to `.png` if it can't be inferred.
    - `comment`: optional, e.g. `"thumbnail for <issue.identifier>"`
- **Slack-sourced image** (download yourself with bot token, then upload binary):
  - Slack's `url_private` requires a bearer token with `files:read` scope. If the bot doesn't have it, you'll silently get an HTML sign-in page back instead of image bytes — verify by checking the first bytes start with the PNG/JPEG magic, not `<!DOCTYPE html>`. If missing, ask the user to grant `files:read` on the Slack app's OAuth & Permissions page and reinstall.
  - Download with `Authorization: Bearer <SLACK_BOT_TOKEN>` from `url_private_download`, then POST the bytes via Confluence multipart upload (v1 API: `POST /wiki/rest/api/content/{pageId}/child/attachment`) — `attach_file`'s `source_url` mode can't pass Slack auth.
  - Use filename `<issue.identifier>-slack.<ext>` to distinguish from any Linear-sourced image still on the page (and to keep both available if a user wants to swap back).
- If Confluence returns a 400 because the filename already exists on the page (re-running the skill), **treat that as success** and reuse the existing filename. Do not retry with a different name.
- Record the final `<filename>` for use in Step 4's More Info cell.

Run these uploads in parallel where possible. The page body must be updated *after* all uploads succeed, so it can reference each attachment by filename.

## Step 4 — Build one section per project

For each project, emit a section matching `template.md`:

**Heading (h2):** `<emoji> Workstream: <Project Name>`
- Emoji: use the project's Linear `icon` field if it resolves to an emoji. Otherwise fall back to 📌.

**Status line (paragraph directly under the heading):**

```
<strong>Workstream Status:</strong> 🟢 🟡 🔴 | <strong>Goal:</strong> <KR> | <a href="<project.url>">Linear project</a>
```

- **Bold both labels** with `<strong>…</strong>` — `Workstream Status:` and `Goal:`.
- After `Workstream Status:` always emit all three colored balls **🟢 🟡 🔴** separated by spaces. They're a picker — the human deletes the two that don't apply on review, leaving just one. Never pre-pick a colour automatically; the team owns that judgement.
- After `Goal:` emit **only the Key Result text** from the Linear project page's description (the `**Key Result**` line, or the `KR1`/`KR2` row in a markdown table when the project uses that format). **Do not write the Objective** — the Goal cell on this page is meant to surface just the metric we're chasing, not the broader narrative. If Linear's KR field is literally `TBD`, write `TBD`. If empty, leave the value blank.
- Linear project link is rendered as `<a href="<project.url>">Linear project</a>` exactly as before.

See feedback memory `feedback_workstreams_status_line`.

**Table** with these columns, in order:
1. Commit Phase
2. What (Activity + Link)
3. So What (How it advances the KR)
4. Target
5. Status
6. More Info (Linear + Slack text links + 50px inline thumbnail, all on one line separated by ` // `)

**Row order:** sort issues **ascending by the displayed sprint label** (the `N` in `Sprint <N>` parsed from `cycle.title` — so Sprint 1 → Sprint 7 reads top-to-bottom in the visible Target column). Issues without a cycle go at the **bottom** of the table. Within the same sprint label, break ties by `cycle.number` (the absolute counter — so Q1 Sprint 6 and Q2 Sprint 6 still bucket together with Q1 first) and then by `issue.identifier`'s numeric suffix for stable ordering. Do **not** use `cycle.number` as the primary key — that's the workspace-absolute counter and it produces a `6, 7, 1, 2, 3, …` jump in the visible Target column when Q1 sprints linger in the table. Do **not** sort by status, Commit Phase, or dueDate. See feedback memory `feedback_workstreams_sort_order`.

**Check-in divider row:** between past and future rows, insert a light-blue full-width divider:

```html
<tr><td colspan="6" data-highlight-colour="#deebff"><p style="text-align: center;">↑ Previous Commitments ↑ · <strong>Check-In Mark</strong> · ↓ Future Commitments ↓</p></td></tr>
```

Placement: right **before** the first row whose Commit Phase month is *strictly greater than the current calendar month* — i.e. between past/current-month rows and next-month rows. Examples (assuming check-in run in May 2026):
- A table with `Mar / Apr / May / Jun / Jul` rows → divider goes between the last `May` row and the first `Jun` row.
- A table with only `May` rows → divider goes after the last data row (nothing future yet).
- A table with only blank-Commit-Phase rows (no cycle) → divider goes before the first data row (everything is treated as future/unscheduled).

Compute the cutoff from today's date: month-number-of-today → divider sits between rows ≤ that month and rows > that month. The skill should re-derive the cutoff each run; don't hard-code `Jun` (e.g. a Q3 check-in run in August would shift the cutoff to between August and September).

See feedback memory `feedback_workstreams_checkin_divider`.

One row per non-`hide-roadmap` issue. Best-effort auto-fill:
- **Commit Phase:** the **short month name** the topic is done (for completed work) or planned to be done (for upcoming work), derived from the cycle the ticket is assigned to. Format: `<MMM>` — just the 3-letter month, **no year** (e.g. `May`, `Jun`, `Jul`). Derive by:
  1. Parsing the date suffix out of `cycle.title` when present (e.g. `"Sprint 4 - 18 May 2026"` → `May`). Look for the pattern ` - <DD> <MMM> <YYYY>` (or `<YY>`) after the sprint label and keep only the month abbreviation.
  2. Falling back to `cycle.endsAt` (the cycle end date) when the title has no date suffix (e.g. `"Sprint 7"` alone). Format the endsAt timestamp's month — `%b` in Python (`May`, `Jun`, …).
  3. If the issue isn't on a cycle, leave the cell **blank** — don't guess from `dueDate` or `completedAt`.

  Use `endsAt` (not `startsAt`) when falling back: sprints often start in one month and end in another; the month they *end* in is the one the work is delivered in, which is what "commit phase" reads as.

  History: this column used `Past`/`Next` binary until 2026-05-28, then `May 2026`/`Jun 2026` full-form for a few hours, then settled on month-only on user request the same day. The year is implied by the page's quarter context (e.g. a Q2 check-in page).
- **What (Activity + Link):** plain text — the `<issue.title>` written as a normal sentence (no `<a>` wrapper). Add a trailing period if the title doesn't already end with one. The clickable link lives in the **More Info** column (as the "Linear" text link), not here.
- **So What (How it advances the KR):** **≤25 words.** A single tight sentence (or two short ones) saying **why we believe this ticket contributes to the project goal** — the goal/KR being the one in the status line above the table (sourced from the Linear project page's KR field).
  - **Hard limit: 25 words.** Past versions ballooned to 80–110 words with results, hypotheses, and Direct/Indirect framing; the column became wall-of-text and stakeholders skipped it. Truncate ruthlessly.
  - **Pattern to follow:** lead with the activity in 3–6 words (e.g. "Lower the Green payout for sub-85% batteries"), then a single clause stating the contribution to the goal (e.g. "protects CM1 if we ship the 85% PDP threshold"). One em-dash or comma between the two halves. No "Direct contribution to the project goal — …" prefix; just say it.
  - **For Shipped tickets**, lean on the result number when it's concise (e.g. "proven +4.62% transactions for PLA — the PLA-CVR foundation").
  - **For supporting/tracking/cleanup work**, name what it unblocks: "Tracking for OCN-6383 — without it we couldn't attribute the +4.62% uplift cleanly."
  - **Source from Linear description + matched Slack post**, but distill aggressively. Strip markdown, links, image syntax. If the issue has no description AND no matched Slack post AND you can't reasonably infer the goal contribution from the title, leave the cell blank rather than guessing.
  - See feedback memory `feedback_workstreams_so_what`.
- **Target:** the sprint label parsed from the issue's Linear `cycle.title`, kept as just `Sprint <N>` (e.g. `Sprint 4` — drop any trailing date). Linear cycle titles look like `"Sprint 4 - 18 May 2026"`, `"Sprint 10"`, or sometimes just `"Sprint 7"`; take everything up to (but excluding) the first ` - ` (space-hyphen-space) and `.strip()` it. Do **not** use `issue.cycle.number` for the display value — that's the absolute counter (e.g. 74), not the within-quarter sprint number the team uses on the page. If the issue isn't on a cycle, leave the cell blank — do **not** fall back to `dueDate` or milestone `targetDate`. (See feedback memory `feedback_workstreams_target_sprint`.)
- **Status:** four-level mapping based on the **Linear status name** (`issue.status`, not `statusType`):
  - `Shipped ✅` — only when the status name is exactly `Done`.
  - `Almost done` — when the status name is `QA`, `Code Review`, or `Ready for Deployment`.
  - `In Progress` — when the status name is exactly `In Progress`.
  - `Not Started` — every other status (Backlog, Selected for Development, Unready, Triage, Rejected, Duplicate, Cancelled, Won't do, etc.). Yes — Rejected and Duplicate map to `Not Started` even though they're "completed" by `statusType`; the page is about what shipped to users, and Rejected/Duplicate work didn't.

  Match status names case-sensitively. Status name (e.g. `Code Review`) is distinct from `statusType` (e.g. `started`) — use the **name** for this mapping. See feedback memory `feedback_workstreams_status_shipped`.
- **More Info:** up to **two text links + a very small inline thumbnail**, all on **one line in a single `<p>`** separated by ` // ` (space-slash-slash-space). Order: Linear first, Slack second, thumbnail last.

  Build the cell as:

  ```html
  <td><p>{Linear-link} // {Slack-link} // {thumbnail}</p></td>
  ```

  Where:
  - `{Linear-link}` = `<a href="<issue.url>">Linear</a>` — always emit (every ticket has a Linear URL).
  - `{Slack-link}` = `<a href="<slack.permalink>">Slack</a>` — only if a matched Slack post exists (Step 3). The permalink format is `https://rebuy.slack.com/archives/<channel_id>/p<ts_without_dot>` — e.g. ts `1778224668.654779` in channel `CSJEUTK42` becomes `https://rebuy.slack.com/archives/CSJEUTK42/p1778224668654779`.
  - `{thumbnail}` = `<ac:image ac:thumbnail="true" ac:width="50"><ri:attachment ri:filename="<attachment-filename>" /></ac:image>` — only if a Step 3.5 image upload succeeded. Use `ac:width="50"` for a very small inline thumbnail; **`ac:thumbnail="true"` is required** for the click → full-size preview to engage reliably (without it, very small images sometimes fail to trigger Confluence's media viewer). **No `<a>` wrapper** — wrapping in `<a href="<download-url>">` triggers a file download instead of the lightbox.

  Omit the corresponding ` // ` separator when a piece is missing — don't leave dangling `//` or empty segments. So `Linear // Slack // [thumb]` becomes `Linear // [thumb]` when no Slack match, `Linear // Slack` when no image, or just `Linear` when neither exists.

  Humans can append additional notes after the auto-generated line if they want; only ever overwrite the auto-line, never anything else in the cell.

  See feedback memory `feedback_workstreams_more_info_links`.

## Step 5 — Write to the Confluence page

**Prerequisite:** Step 3.5 attachment uploads must already be complete. The page body references attachments by filename, so all uploads need to land first.

Choose one of two write modes based on the page state and the user's intent. **Ask if unclear** — bulk-appending a duplicate set of tables when the user wanted a targeted refresh is a noisy mistake.

### Mode A — Append fresh sections (first-time generation or user explicitly asked for a fresh block)

1. Get the current page body in `storage` representation (from Step 2).
2. Concatenate the new sections **after** the existing body — never replace it.
3. Write the body back (see "Writing the body" below).
4. `version_message`: e.g. `"team-workstreams: appended N project sections for <Team>"`.

### Mode B — Targeted refresh of existing workstream tables (re-run on a page that already has workstream tables for the same projects)

When the page already contains workstream tables for these projects, prefer **surgical edits** over appending a duplicate set.

**Core rule for re-runs: only the More Info column is auto-overwritten unconditionally.** Other columns (Commit Phase, What, So What, Target, Status) are **fill-when-empty only** — never overwrite existing content. The team uses those columns to capture judgment that the skill can't replicate (status calls, refined wording, sprint moves, etc.); silently overwriting them on every refresh would erase human work.

Concrete operations:

- **More Info (last column): always rebuild the cell from scratch.** Linear/Slack links and the inline thumbnail all come from Linear/Slack + Step 3.5 uploads; the skill is the source of truth. Replace the entire `<td>` content with the latest single-line `<p>Linear // Slack // [thumb]</p>` form (see Step 4's More Info spec). Don't preserve old More Info content — even if a human added a note, it'll regenerate on the next run anyway, and they can always re-add it.
- **Fill empty Target cells** — for each row whose primary OCN-ID has a `cycle.title`, replace the empty `<p local-id="…" />` in the 4th `<td>` with `<p local-id="…">Sprint <N></p>`. **If the cell already has text content (Sprint label or anything else), leave it alone.**
- **Fill empty So What cells** — same pattern. Replace the empty `<p local-id="…" />` in the 3rd `<td>` only when there's no text. **Preserve any human-written text.** Easy to miss: tables that someone else built often have empty So What cells; your appended sections look fine in isolation but the page reads half-filled top-to-bottom. **Always scan both pre-existing and appended sections.**
- **Do NOT** overwrite Status, Commit Phase, or What cells — even if Linear's data changed since the last refresh. If a row's Status went from `In Progress` to `Done` in Linear, the page won't update automatically; a human can edit it explicitly. The "drift" is a feature: the page is a manually-curated check-in, not a Linear mirror.
- **Remove rows for newly excluded issues** — delete the matching `<tr>` block when a ticket has gained the `hide-roadmap` label, switched to `Rejected`/`Backlog`, or been archived since the last run (see Step 3).
- **Add new rows for new tickets** — append at the appropriate sort position. New rows are auto-filled with the full set of columns (this is a first-fill, not an overwrite).

After all surgical edits, write the body back. `version_message` should describe what changed, e.g. `"team-workstreams: refresh More Info links; fill empty So What cells; remove rows newly tagged hide-roadmap"`.

See feedback memory `feedback_workstreams_overwrite_policy`.

### Writing the body

`mcp__confluence__update_page` works fine for bodies up to roughly 80–100 KB. **For larger bodies** (when the appended sections push past that), the tool call's `body` argument can exceed the Read tool's token limits when verifying — call the Confluence v2 API directly instead:

```python
PUT https://rebuyer.atlassian.net/wiki/api/v2/pages/{page_id}
Authorization: Basic base64(email:token)
Content-Type: application/json
Body: {"id": "...", "status": "current", "title": "...",
       "body": {"representation": "storage", "value": "..."},
       "version": {"number": <current+1>, "message": "..."}}
```

Credentials live in `~/.claude/settings.local.json` allow-list — reuse the same `j.vanhardeveld@rebuy.com` / `ATATT…` pair that the Confluence MCP uses.

**Confluence normalizes saved bodies.** It may rewrite your inserted markup (e.g. add `ri:version-at-save="1"` to attachment tags, decorate `<ac:image>` with `ac:align`/`ac:layout`/`ac:original-height`/etc., or — rarely — split paragraphs mid-prose if your `<p>` insert is positioned awkwardly inside flowing text). If a re-fetch shows broken HTML (orphan text fragments like `</p>ing-page thesis.` outside a paragraph), **rebuild the affected appended block from scratch** rather than trying to surgically patch it: truncate the body at the `<h2>` that starts your appended section and re-emit the entire block fresh.

Report back to the user with the page URL and a one-line summary of which projects/rows were affected.

## Notes

- Storage format is XHTML. Tables use `<table data-layout="full-width"><colgroup>…</colgroup><tbody><tr><th>…</th></tr><tr><td>…</td></tr></tbody></table>`. Empty cells should be `<td><p /></td>` (Confluence dislikes truly empty `<td>`s in some renderers — a paragraph keeps the cell editable).
- **Table must be full-width** with `data-layout="full-width"` on `<table>`, and a `<colgroup>` that sizes the 6 columns per `template.md`: narrow (~100–110px) for Commit Phase (100px to fit `May` / `Jun` / etc.), Target, Status; wide (~360–460px) for What and So What (So What now caps at ≤25 words but the column still benefits from the room); compact (~130px) for More Info — enough for the 50px inline thumbnail plus the two short text links inline with it on the same line. See feedback memory `feedback_workstreams_table_width`.
- Don't pre-fill columns the user said to leave blank. Half-filled "best-effort" guesses in those columns create noise.
- Issue label exclusion is case-sensitive in Linear: match `hide-roadmap` exactly.
- **Images are always uploaded as Confluence attachments**, never inlined via `<ri:url>` against `uploads.linear.app`. Linear's signed URLs require a Linear cookie to render and break for readers who aren't logged in. Step 3.5 hands each image to `mcp__confluence__attach_file` (Confluence downloads server-side), and Step 4 references the resulting filename via `<ri:attachment ri:filename="…" />`. This is heavier than `<ri:url>` but auth-free for readers.
- **Attachment filename convention:**
  - Linear-sourced image: `<issue.identifier>.<ext>` (e.g. `ENG-123.png`).
  - Slack-sourced image: `<issue.identifier>-slack.<ext>` (e.g. `ENG-123-slack.png`). The `-slack` suffix lets both variants coexist on the page if the source ever needs to swap back.

  One image per issue+source keeps names unique. On re-run, Confluence may 400 because the filename already exists — treat that as success and reuse the filename rather than renaming.
