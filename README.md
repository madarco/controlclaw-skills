# ControlClaw skills

Agent skills for [ControlClaw](https://controlclaw.com), the hosting platform for OpenClaw AI
assistants. Each skill walks a coding agent (Claude Code or any agent that reads `SKILL.md`) through
a setup task around ControlClaw.

| Skill | What it does |
| --- | --- |
| [`controlclaw-google-oauth`](skills/controlclaw-google-oauth/SKILL.md) | Creates a Google Cloud project and OAuth client in your Google account, ready to drop into ControlClaw's Integrations, Google, Set up. You sign in to Google yourself; the agent does the rest. |

## Install

```bash
git clone https://github.com/madarco/controlclaw-skills.git
cd controlclaw-skills
for d in skills/*/; do ln -sfn "$PWD/$d" ~/.claude/skills/"$(basename "$d")"; done
```

Then run `/controlclaw-google-oauth` in Claude Code.
