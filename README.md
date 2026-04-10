# SureContact Skills

A collection of Claude Code skills for the [SureContact](https://surecontact.io) platform.

## Install

```bash
npx skills add brainstormforce/surecontact-skills
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
