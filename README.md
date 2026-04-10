# SureContact Skills

A collection of Claude Code skills for the [SureContact](https://surecontact.io) platform.

## Install

```bash
npx skills add brainstormforce/surecontact-skills
```

> **Claude Code users:** When the installer asks which agents to install to, make sure to select **Claude Code** from the list. Claude Code reads skills from `~/.claude/skills/` — other agents use `.agents/skills/` which Claude Code does not read.

### Manual install (Claude Code)

If you prefer to install directly without the CLI:

```bash
mkdir -p ~/.claude/skills/email-template-designer
curl -o ~/.claude/skills/email-template-designer/SKILL.md \
  https://raw.githubusercontent.com/brainstormforce/surecontact-skills/master/skills/email-template-designer/SKILL.md
```

## Available Skills

### `email-template-designer`

Design and generate SureContact email templates as valid JSON — ready to save via the API.

**Triggers when you say things like:**
- "create an email template"
- "design a welcome email"
- "build a newsletter template"
- "generate a promo email"

## Usage

After installing, just describe the email you want in Claude Code:

```
Create a welcome email template for new SureContact users with a hero image, headline, body text, and a CTA button.
```

Claude will output a complete, valid template JSON you can POST directly to the SureContact API.

## Skills in this repo

| Skill | Description |
|-------|-------------|
| [email-template-designer](./skills/email-template-designer/) | Generate SureContact email template JSON from a description |
