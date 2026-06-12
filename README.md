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

1. Open the skill folder, e.g. `weekly-update/SKILL.md`
2. Edit the instructions directly — they're plain English
3. Save the file
4. Copy the updated skill to your local Claude setup:
   ```
   cp -r weekly-update ~/.claude/skills/
   ```
5. Push the change back to GitHub so your colleagues get it too (see section 3 below)

### Tips

- The `description:` line at the top of `SKILL.md` is what Claude uses to decide when to trigger the skill. Keep it accurate.
- Other files in the folder (like `template.md` or `config.md`) are supporting documents the skill refers to — you can edit those too.
- Test your change by running the skill in Claude Code before pushing.

---

## 3. How to add a new skill

### Step 1 — Create a folder

Pick a short, lowercase name with hyphens (e.g. `sprint-planning`). Create a folder for it:

```
mkdir ~/.claude/skills/sprint-planning
```

### Step 2 — Create the SKILL.md file

Create a file called `SKILL.md` inside the folder. Use this template:

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

Open Claude Code and type `/<your-skill-name>`. Make sure it behaves as expected before sharing.

### Step 4 — Add it to this repo

```bash
# Clone the repo if you haven't already
gh repo clone awjvanhardeveld/claude-skills /tmp/claude-skills

# Copy your new skill into the repo
cp -r ~/.claude/skills/sprint-planning /tmp/claude-skills/

# Commit and push
cd /tmp/claude-skills
git add sprint-planning
git commit -m "Add sprint-planning skill"
git push
```

Done — your colleagues can now pull it down and use it.

---

## Pulling the latest skills from this repo

When a colleague adds or updates a skill, run this to get the latest version:

```
gh repo clone awjvanhardeveld/claude-skills /tmp/claude-skills-update 2>/dev/null || git -C /tmp/claude-skills-update pull
cp -r /tmp/claude-skills-update/* ~/.claude/skills/
```

---

## Questions?

Ask Job van Hardeveld or open an issue in this repo.
