# Output template (Confluence storage XHTML)

One block per Linear project. Append these blocks to the existing page body.

```html
<h2>📌 Workstream: Example Project Name</h2>
<p><strong>Workstream Status:</strong> 🟢 🟡 🔴 | <strong>Goal:</strong> Improve checkout CVR by 5% test over test | <a href="https://linear.app/rebuy/project/example-project-abc123">Linear project</a></p>
<table data-layout="full-width">
  <colgroup>
    <col style="width: 100.0px;" />
    <col style="width: 360.0px;" />
    <col style="width: 460.0px;" />
    <col style="width: 100.0px;" />
    <col style="width: 110.0px;" />
    <col style="width: 130.0px;" />
  </colgroup>
  <tbody>
    <tr>
      <th><p>Commit Phase</p></th>
      <th><p>What (Activity + Link)</p></th>
      <th><p>So What (How it advances the KR)</p></th>
      <th><p>Target</p></th>
      <th><p>Status</p></th>
      <th><p>More Info</p></th>
    </tr>
    <tr>
      <td><p>May</p></td>
      <td><p>Example issue title.</p></td>
      <td><p>Short sentence (≤25 words) saying why we believe this ticket contributes to the project goal — e.g. "Tests value-based warranty pricing — likely the biggest lever for the +25% CM1 target."</p></td>
      <td><p>Sprint 4</p></td>
      <td><p>In Progress</p></td>
      <td>
        <p><a href="https://linear.app/rebuy/issue/ENG-123/example-issue">Linear</a> // <a href="https://rebuy.slack.com/archives/CSJEUTK42/p1778224668654779">Slack</a> // <ac:image ac:thumbnail="true" ac:width="50"><ri:attachment ri:filename="ENG-123-slack.png" /></ac:image></p>
      </td>
    </tr>
    <!-- One <tr> per issue. Exclude issues with the hide-roadmap label, or status Rejected / Backlog, or archived.
         Sort rows ascending by the displayed sprint label (Sprint 1 → Sprint N) — primary key is the N parsed from cycle.title,
         NOT cycle.number absolute. Issues without a cycle at the bottom. Tiebreakers: cycle.number, then issue identifier.
         Omit the Slack <p> if no matched Slack post; omit the thumbnail <p> if no image was uploaded.
         Insert the check-in divider row (below) between the last row whose Commit Phase month <= current calendar month
         and the first row whose Commit Phase month > current calendar month. -->
    <tr><td colspan="6" data-highlight-colour="#deebff"><p style="text-align: center;">↑ Previous Commitments ↑ · <strong>Check-In Mark</strong> · ↓ Future Commitments ↓</p></td></tr>
  </tbody>
</table>
```

## Inclusion filters (apply before any column auto-fill)

An issue appears in the table only if **all** of these are true:

- `"hide-roadmap" not in issue.labels` (case-sensitive)
- `issue.status` is not `Rejected` and not `Backlog` (status name, case-sensitive)
- `issue.archivedAt` is null

On re-runs against an existing page, also delete `<tr>` rows whose primary OCN-ID now fails any of these checks but previously passed (labels and statuses evolve between checkpoints).

## Column auto-fill cheat-sheet

| Column                            | Source                                                              |
|-----------------------------------|---------------------------------------------------------------------|
| Commit Phase                      | The short month name the topic is/was delivered, derived from the cycle. Format `<MMM>` — no year (e.g. `May`, `Jun`). Parse the date suffix out of `cycle.title` (`"Sprint 4 - 18 May 2026"` → `May`); if the title has no date, fall back to `cycle.endsAt`'s month name. Blank if the issue has no cycle. |
| What (Activity + Link)            | Plain text sentence describing the activity — use `{issue.title}` verbatim (no hyperlink). Add a trailing period if missing. |
| So What (How it advances the KR)  | **≤25 words.** Single short sentence saying why we believe this ticket contributes to the project goal (the KR in the status line above). Pattern: 3–6 word activity, then one clause on contribution. For Shipped tickets, include the result number when concise. Blank if no signal. |
| Target                            | The sprint label parsed from `issue.cycle.title` — keep just `Sprint <N>` (split on ` - ` and strip). E.g. `"Sprint 4 - 18 May 2026"` → `Sprint 4`. **Do not** use `issue.cycle.number` (that's the absolute counter, e.g. 74). Blank if the issue isn't on a cycle. Do not fall back to dueDate or milestone targetDate. |
| Status                            | Four-level mapping on the Linear **status name** (`issue.status`, not `statusType`): `Done` → `Shipped ✅`; `QA` / `Code Review` / `Ready for Deployment` → `Almost done`; `In Progress` → `In Progress`; everything else (Backlog, Selected for Development, Unready, Rejected, Duplicate, Cancelled, …) → `Not Started`. |
| More Info                         | All on one line in a single `<p>`, separated by ` // ` (space-slash-slash-space). Linear // Slack // [50px thumbnail]: `<a href="{issue.url}">Linear</a>` (always), `<a href="{slack.permalink}">Slack</a>` (only if matched), `<ac:image ac:thumbnail="true" ac:width="50"><ri:attachment ri:filename="{filename}" /></ac:image>` (only if image exists — `ac:thumbnail="true"` makes click open full-size; **no `<a>` wrapper** on the thumbnail, that triggers a download instead). Drop the corresponding ` // ` separator when a piece is missing. |

## Row order rule

Primary sort key: the **displayed sprint label** number (parsed from `cycle.title` → `Sprint 1` through `Sprint 7`), ascending. Tiebreakers: `cycle.number` ascending (so Q1's Sprint 6 sorts above Q2's Sprint 6 within the Sprint 6 bucket), then `issue.identifier`'s numeric suffix. Issues without a cycle go to the bottom.

```python
def sort_key(issue):
    cycle_title = (issue.get("cycle") or {}).get("title") or ""
    cycle_number = (issue.get("cycle") or {}).get("number")
    m = re.match(r"Sprint (\d+)", cycle_title)
    sprint_label = int(m.group(1)) if m else 10**9
    return (
        sprint_label,                                          # primary
        cycle_number if cycle_number is not None else 10**9,   # secondary (Q1 vs Q2)
        int(issue["identifier"].split("-")[1]),                # tertiary
    )
```

**Do not** use `cycle.number` as the primary sort key — it's the workspace-absolute counter (e.g. 74 for Q2 Sprint 4) and produces a `6, 7, 1, 2, …` jump in the Target column when Q1 rows linger in the table.

## Heading rules

- Emoji = `project.icon` if it's an emoji glyph, else 📌
- Title = literally `Workstream: <project.name>`

## Status line rules

- Both labels `Workstream Status:` and `Goal:` are wrapped in `<strong>` for bold.
- `Workstream Status:` value is the three-ball picker `🟢 🟡 🔴` separated by spaces — the human deletes the two that don't apply. Never auto-pick a colour.
- `Goal:` value = the project's Key Result text from the Linear project page (the `**Key Result**` line or the `KR1`/`KR2` row). Do **not** include the Objective text. Literal `TBD` if Linear says TBD; blank if there's no description at all.
- Linear link uses `project.url` and link text `Linear project`.
