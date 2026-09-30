# ControlClaw skills

Agent skills for [ControlClaw](https://controlclaw.com), the hosting platform for OpenClaw AI
assistants. Each skill walks a coding agent (Claude Code or any agent that reads `SKILL.md`) through
a setup task around ControlClaw.

| Skill | What it does |
| --- | --- |
| [`controlclaw-google-oauth`](skills/controlclaw-google-oauth/SKILL.md) | Creates a Google Cloud project and OAuth client in your Google account, ready to drop into ControlClaw's Integrations, Google, Set up. You sign in to Google yourself; the agent does the rest. |
| [`controlclaw-google-drive`](skills/controlclaw-google-drive/SKILL.md) | Creates a Google service account and key, and shares your chosen Drive folders with it, so ControlClaw can mount them on your agents. |

## Install

With the [skills.sh](https://skills.sh) CLI:

```bash
npx skills add madarco/controlclaw-skills
```

It asks which skills to install and which agents to add them to. To install just one, name it:

```bash
npx skills add madarco/controlclaw-skills --skill controlclaw-google-oauth
```

Then run `/controlclaw-google-oauth` or `/controlclaw-google-drive` in Claude Code, or ask your agent
to do the task.

### Manual install (Claude Code)

```bash
git clone https://github.com/madarco/controlclaw-skills.git
cd controlclaw-skills
for d in skills/*/; do ln -sfn "$PWD/$d" ~/.claude/skills/"$(basename "$d")"; done
```

Then run `/controlclaw-google-oauth` or `/controlclaw-google-drive` in Claude Code.
