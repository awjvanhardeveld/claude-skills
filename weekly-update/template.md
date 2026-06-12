# Weekly Update Output Template

## Sending via Slack MCP

When sending via `mcp__slack__slack_send_message`, use the message text directly (no code block wrapper). When producing output for copy-paste, wrap in a fenced code block.

## Formatting Rules

All formatting must be compatible with Slack's block renderer:

**Bold text** — use Unicode bold characters for all bold levels (team header, section titles, initiative names). Never use `*markers*`.
Examples: 𝗧𝗛𝗜𝗦 𝗪𝗘𝗘𝗞, 𝗜𝗡𝗧 𝗦𝗘𝗢 𝗰𝗹𝗲𝗮𝗻𝘂𝗽, 𝗧𝗲𝗮𝗺 𝗠𝗮𝗻𝗴𝗿𝗼𝘃𝗲

**Bullet points** — use `- ` (hyphen + space) so Slack renders a real bulleted list block. Always leave a blank line BEFORE and AFTER each bullet block to keep the next header from being pulled into list indentation.

**Bullet status prefix** — every bullet starts with a status emoji (no text label, no colon), then the sentence. Map:
- `Done` / `Completed` → `✅`
- `QA` / `Code Review` / `In Review` → `🔜`
- `In Progress` → `⏳`
- `Selected for Development` → `⏩`

**Findings as sub-bullets** — when a finding/metric/result applies to a specific ticket, place it as an indented sub-bullet (4 spaces) directly below that ticket's bullet, ending with `📊`. Do NOT use a separate `Findings/Results:` section line at the initiative level for ticket-specific findings. Project-level findings that can't be tied to a ticket may still appear as a `Findings/Results:` line below the bullet block.

**Blank lines between sections** — use a plain empty line (double newline `\n\n`). Do NOT use `&#8203;` (renders as literal text when posted via bot MCP). Do NOT use `---` (Slack block renderer rejects horizontal rules).

**Links** — use `<url|caption>` Slack mrkdwn format:
- Linear issue: `<https://linear.app/rebuy/issue/MNG-123|MNG-123>`
- Linear project: `<https://linear.app/rebuy/project/...|Project>`
- Linear milestone: `<https://linear.app/rebuy/...|Milestone>`
- Slack message: `<https://rebuy.slack.com/archives/C.../p...|Slack>`

**Context line placement** — always place `Context:` BEFORE the bullet list for an initiative, never after. Trailing non-bullet lines after a bullet block get pulled into list indentation.

**Findings/Results** — omit entirely if no data is available after asking the user.

**No `Progress:` label** — bullets speak for themselves, drop the label.

## Structure

```
[Unicode bold team header + date]
[Unicode bold section header]
[Unicode bold initiative name]
Context: <url|Project or Milestone> 🔗        ← only if applicable; always BEFORE bullets

- ✅ [sentence] — <issue-url|OCN-NNNN>
    - [ticket-specific finding or metric] 📊       ← optional sub-bullet
- 🔜 [sentence] — <issue-url|OCN-NNNN>
- ⏳ [sentence] — <issue-url|OCN-NNNN>
- ⏩ [sentence] — <issue-url|OCN-NNNN>

Findings/Results: [project-level data only] 📊  ← omit if none or if attached as sub-bullet
Blocker/Delay: [reason] ⚠️                       ← omit if none

[Unicode bold initiative name]

- ✅ [sentence] — <issue-url|OCN-NNNN> · <slack-permalink|Slack>
    - [finding] 📊

[Unicode bold NEWS & FYI header]

- [item] — <slack-permalink|Slack>
```

## Example (Team Mangrove)

```
𝗧𝗲𝗮𝗺 𝗠𝗮𝗻𝗴𝗿𝗼𝘃𝗲 🌿 — Mar 26, 2026

✅ 𝗧𝗛𝗜𝗦 𝗪𝗘𝗘𝗞: 𝗣𝗿𝗼𝗴𝗿𝗲𝘀𝘀 & 𝗥𝗲𝘀𝘂𝗹𝘁𝘀

𝗜𝗡𝗧 𝗦𝗘𝗢 𝗰𝗹𝗲𝗮𝗻𝘂𝗽 𝗮𝗻𝗱 𝗼𝗽𝘁𝗶𝗺𝗶𝘇𝗮𝘁𝗶𝗼𝗻
Context: <https://linear.app/rebuy/project/...|Project> 🔗

- ✅ FR market SEO audits — iPhone flow (<https://linear.app/rebuy/issue/MNG-1016|MNG-1016>) and Android phone flow (<https://linear.app/rebuy/issue/MNG-1018|MNG-1018>)
    - Indexed page count up 12% week-over-week 📊
- ✅ NL initial audit deployed — <https://linear.app/rebuy/issue/MNG-936|MNG-936>

𝗜𝗡𝗧 𝗧𝗿𝗮𝗻𝘀𝗹𝗮𝘁𝗶𝗼𝗻 𝗔𝘂𝘁𝗼𝗺𝗮𝘁𝗶𝗼𝗻

- ✅ Distinguish critical and non-critical translations deployed — <https://linear.app/rebuy/issue/MNG-988|MNG-988>

💡 𝗡𝗘𝗪𝗦 & 𝗙𝗬𝗜

- Valeria on vacation Mar 23 – Apr 20 — <https://rebuy.slack.com/archives/C055SRHRV4M/p1774282577745639|Slack>
```
