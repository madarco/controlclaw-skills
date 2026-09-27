# ControlClaw skills

Agent skills (Claude Code `SKILL.md` format) for setting things up around ControlClaw.

| Skill | What it does |
| --- | --- |
| [`controlclaw-google-oauth`](skills/controlclaw-google-oauth/SKILL.md) | Creates a Google Cloud project and OAuth client in the user's Google account, ready to drop into ControlClaw's Integrations, Google, Set up. |

## Install

Link each skill into your skills folder:

```bash
for d in skills/*/; do ln -sfn "$PWD/$d" ~/.claude/skills/"$(basename "$d")"; done
```

Then run it as `/controlclaw-google-oauth`.
