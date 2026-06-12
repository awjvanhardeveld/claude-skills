# Claude Skills

Custom slash commands for Claude Code used by the product team at Rebuy.

---

## What is a skill?

A skill is a custom slash command that you can type in Claude Code (e.g. `/weekly-update`). It triggers a pre-written set of instructions that tells Claude exactly what to do — which tools to use, what format to follow, and how to structure the output.

### Skills in this repo

| Skill | Command | What it does |
|---|---|---|
| Weekly update | `/weekly-update` | Pulls data from Linear and Slack to draft a structured weekly PM update |
| Weekly update (quick) | `/weekly-update-minimal` | Same as above but fully automatic — no follow-up questions, posts straight to Slack |
| Confluence workstreams | `/confluence-monthly-workstreams` | Syncs a Linear team's projects onto a Confluence roadmap page |

---

## Review policy

**All changes require a pull request and at least one approval before they go live.** This is enforced automatically — no one can push directly to `main`, including the repo owner.

This ensures skills stay reliable and colleagues aren't surprised by unreviewed changes.

---

## 1. How to use the skills

### Step 1 — Install Claude Code

If you don't have it yet, install the Claude Code CLI:

```
npm install -g @anthropic-ai/claude-code
```

Then open a terminal and run `claude` to get started.

### Step 2 — Copy the skills to your machine

Run this once in your terminal to download the skills:

```
gh repo clone awjvanhardeveld/claude-skills /tmp/claude-skills-setup
cp -r /tmp/claude-skills-setup/* ~/.claude/skills/
```

> If you don't have `gh` installed: `brew install gh`, then `gh auth login`.

### Step 3 — Use a skill in Claude Code

Open Claude Code and type the slash command:

```
/weekly-update
```

Claude will guide you through the rest.

---

## 2. How to adjust a skill

Each skill lives in its own folder. Inside you'll find a `SKILL.md` file — this is the main instruction file that tells Claude what to do.

### Making a change

1. Clone the repo (if you haven't already):
   ```
   gh repo clone awjvanhardeveld/claude-skills /tmp/claude-skills
   cd /tmp/claude-skills
   ```

2. Create a new branch for your change:
   ```
   git checkout -b improve-weekly-update
   ```

3. Edit the skill file, e.g. `weekly-update/SKILL.md` — instructions are plain English.

4. Test the change locally by copying it to your Claude setup:
   ```
   cp -r weekly-update ~/.claude/skills/
   ```
   Then open Claude Code and run the skill to verify it works.

5. Commit and push your branch:
   ```
   git add weekly-update
   git commit -m "Improve weekly-update: add blocker detection"
   git push -u origin improve-weekly-update
   ```

6. Open a pull request so a colleague can review it:
   ```
   gh pr create --title "Improve weekly-update" --body "Describe what you changed and why."
   ```

7. Once approved, merge via the GitHub website or:
   ```
   gh pr merge --squash
   ```

### Tips

- The `description:` line at the top of `SKILL.md` is what Claude uses to decide when to trigger the skill. Keep it accurate.
- Other files in the folder (like `template.md` or `config.md`) are supporting documents the skill refers to — you can edit those too.
- Always test locally before opening a PR.

---

## 3. How to add a new skill

### Step 1 — Create a branch

```bash
cd /tmp/claude-skills
git checkout main && git pull
git checkout -b add-sprint-planning
```

### Step 2 — Create the skill folder and SKILL.md

Pick a short, lowercase name with hyphens (e.g. `sprint-planning`):

```
mkdir sprint-planning
```

Create `sprint-planning/SKILL.md` using this template:

```markdown
---
name: sprint-planning
description: One sentence describing what this skill does and when Claude should use it.
---

[Write your instructions here in plain English. Tell Claude:
- What to ask the user
- Which tools or data sources to use (Linear, Slack, Confluence, etc.)
- What format to produce the output in]
```

The `name` must match the folder name exactly. The `description` is shown in the skill list and used by Claude to recognize when to trigger it.

### Step 3 — Test it locally

Copy the skill to your Claude setup and try it:

```
cp -r sprint-planning ~/.claude/skills/
```

Open Claude Code and type `/sprint-planning`. Make sure it behaves as expected.

### Step 4 — Open a pull request

```bash
git add sprint-planning
git commit -m "Add sprint-planning skill"
git push -u origin add-sprint-planning
gh pr create --title "Add sprint-planning skill" --body "Describe what it does and when to use it."
```

A colleague will review and approve it before it merges to `main`.

---

## Pulling the latest skills from this repo

When a colleague's PR is merged, run this to get the latest version:

```
git -C /tmp/claude-skills pull 2>/dev/null || gh repo clone awjvanhardeveld/claude-skills /tmp/claude-skills
cp -r /tmp/claude-skills/* ~/.claude/skills/
```

---

## Questions?

Ask Job van Hardeveld or open an issue in this repo.
