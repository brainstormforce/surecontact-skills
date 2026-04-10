---
name: email-template-designer
description: Design and generate SureContact email templates as valid JSON. Use this skill when the user asks to create, generate, design, or build an email template for the SureContact email builder. Accepts a description of the email purpose, content, and style preferences, then outputs a complete, valid template JSON ready to save via the API.
---

You are an expert email template designer for the SureContact email builder. When invoked, you will generate a complete, valid email template JSON based on the user's requirements.

The user provides: email purpose, content requirements, style preferences, brand colors, and any special layout needs.

## Your Job

1. **Understand the goal** — what is this email for? (newsletter, promo, transactional, welcome, etc.)
2. **Plan the layout** — decide sections, columns, element sequence
3. **Generate the full template JSON** — valid, Gmail-safe, ready to POST to the API
4. **Output the JSON** — present the complete template JSON the user can use directly

---

## Template Root Structure

Every template must have this root shape:

```json
{
  "name": "Template Name",
  "subject": "Email subject line",
  "preview_text": "Short preview text shown in inbox",
  "containerWidth": "600px",
  "backgroundColor": "transparent",
  "canvasBackgroundColor": "#f4f4f4",
  "padding": "0px",
  "fontFamily": "Arial, Helvetica, sans-serif",
  "fontSize": "16px",
  "lineHeight": "1.6",
  "color": "#333333",
  "textAlign": "left",
  "elements": [...],
  "typographyStyles": {
    "paragraph": { "fontSize": "16px", "color": "#333333", "fontWeight": "400", "lineHeight": "1.6" },
    "heading1":  { "fontSize": "32px", "color": "#111111", "fontWeight": "700", "lineHeight": "1.2" },
    "heading2":  { "fontSize": "24px", "color": "#111111", "fontWeight": "600", "lineHeight": "1.3" },
    "heading3":  { "fontSize": "20px", "color": "#222222", "fontWeight": "600", "lineHeight": "1.4" },
    "heading4":  { "fontSize": "18px", "color": "#222222", "fontWeight": "500", "lineHeight": "1.4" }
  },
  "buttonStyles": {
    "primary":   { "backgroundColor": "#8345DD", "color": "#ffffff", "borderColor": "#8345DD", "borderWidth": "0px", "borderRadius": "6px", "fontSize": "16px", "fontWeight": "600", "paddingX": "24px", "paddingY": "12px" },
    "secondary": { "backgroundColor": "#ffffff", "color": "#8345DD", "borderColor": "#8345DD", "borderWidth": "2px", "borderRadius": "6px", "fontSize": "16px", "fontWeight": "600", "paddingX": "24px", "paddingY": "12px" },
    "tertiary":  { "backgroundColor": "transparent", "color": "#8345DD", "borderColor": "transparent", "borderWidth": "0px", "borderRadius": "6px", "fontSize": "16px", "fontWeight": "600", "paddingX": "24px", "paddingY": "12px" }
  }
}
```

**Always set `containerWidth: "600px"`** — standard email width for all clients.

---

## Element Base Shape

Every element in the `elements` array:

```json
{
  "id": "unique-id-here",
  "type": "ElementType",
  "content": "text or image URL or null",
  "styles": {},
  "parentId": "parent-element-id or null"
}
```

**ID format**: `{type}-el-{random6chars}` — e.g., `section-el-a1b2c3`, `text-el-d4e5f6`

**Important**: The `elements` array is FLAT. All elements live at the top level — nesting is expressed via `parentId` references only.

---

## All Element Types

### `section` — Vertical container
Groups other elements stacked vertically. Root-level sections have `parentId: null`.

```json
{
  "id": "section-el-abc123",
  "type": "section",
  "content": null,
  "parentId": null,
  "styles": {
    "width": "100%",
    "padding": "40px 20px",
    "gap": "20px",
    "backgroundColor": "transparent",
    "margin": "0px",
    "borderRadius": "0"
  }
}
```

**`gap`**: spacing between child elements (e.g., `"16px"`, `"24px"`)
**`padding`**: inner whitespace — `"top right bottom left"` or shorthand
**`backgroundColor`**: can be hex, rgba, `"transparent"`
**`backgroundImage`**: image URL (wrapped in url() automatically)
**`backgroundGradient`**: gradient string e.g. `"linear-gradient(135deg, #667eea, #764ba2)"`

---

### `columns` — Multi-column layout
Must only contain `column` children.

```json
{
  "id": "columns-el-abc123",
  "type": "columns",
  "content": null,
  "parentId": "section-el-abc123",
  "styles": {
    "gap": "16px",
    "padding": "0",
    "margin": "0px",
    "columnPresetId": "equal-2"
  }
}
```

**`columnPresetId`** — required, controls column widths:
- `"single"` — 1 column (100%)
- `"equal-2"` — 2 equal columns (50%/50%)
- `"equal-3"` — 3 equal columns (33%/33%/33%)
- `"equal-4"` — 4 equal columns (25% each)
- `"narrow-wide-2"` — 33%/67%
- `"wide-narrow-2"` — 67%/33%
- `"alternating-narrow-wide"` — 17%/33%/17%/33%
- `"alternating-wide-narrow"` — 33%/17%/33%/17%
- `"sidebar-left"` — 25%/75%
- `"sidebar-right"` — 75%/25%

---

### `column` — Single column inside columns
Must have a `columns` element as parent. Contains content elements.

```json
{
  "id": "column-el-abc123",
  "type": "column",
  "content": null,
  "parentId": "columns-el-abc123",
  "styles": {
    "padding": "10px",
    "backgroundColor": "transparent",
    "verticalAlign": "top",
    "gap": "8px"
  }
}
```

**verticalAlign**: `"top"` | `"middle"` | `"bottom"`

---

### `text` — Paragraph/body text
Supports inline HTML: `<b>`, `<i>`, `<u>`, `<a>`, `<br>`, `<strong>`, `<em>`

```json
{
  "id": "text-el-abc123",
  "type": "text",
  "content": "Hey {{contact.first_name || \"there\"}},<br><br>Your body text here.",
  "parentId": "section-el-abc123",
  "styles": {
    "typographyStyle": "paragraph",
    "fontSize": "inherit",
    "color": "inherit",
    "fontFamily": "inherit",
    "fontWeight": "400",
    "lineHeight": "inherit",
    "textAlign": "left",
    "margin": "0px",
    "padding": "0",
    "backgroundColor": "transparent"
  }
}
```

**`typographyStyle`**: `"paragraph"` | `"heading1"` | `"heading2"` | `"heading3"` | `"heading4"` | `"inherit"`
— When set, the element inherits fontSize/color/fontWeight/lineHeight from `template.typographyStyles`

**Personalization variables**: Use `{{contact.first_name || "there"}}` for merge tags

---

### `header` — Heading text
Renders as `<h1>`.

```json
{
  "id": "header-el-abc123",
  "type": "header",
  "content": "Welcome to Our Newsletter",
  "parentId": "section-el-abc123",
  "styles": {
    "typographyStyle": "heading1",
    "textAlign": "center",
    "margin": "0px",
    "color": "inherit",
    "fontFamily": "inherit"
  }
}
```

---

### `button` — Call-to-action button

```json
{
  "id": "button-el-abc123",
  "type": "button",
  "content": "Shop Now",
  "parentId": "section-el-abc123",
  "styles": {
    "buttonVariant": "primary",
    "buttonSize": "medium",
    "buttonFit": "compact",
    "href": "https://example.com",
    "linkTarget": "_blank",
    "align": "center"
  }
}
```

**`buttonVariant`**: `"primary"` | `"secondary"` | `"tertiary"` | `"custom"`
— `primary`/`secondary`/`tertiary` inherit from `template.buttonStyles`
— `custom` uses element's own styles

**`buttonSize`**: `"small"` | `"medium"` | `"large"` | `"custom"`
— small: 8px 16px padding, 14px font
— medium: 12px 24px padding, 16px font
— large: 16px 32px padding, 18px font

**`buttonFit`**: `"compact"` (auto width) | `"full"` (100% width) | `"custom"`
**`align`**: `"left"` | `"center"` | `"right"` — alignment within the container

---

### `image` — Image element
`content` is the image URL.

```json
{
  "id": "image-el-abc123",
  "type": "image",
  "content": "https://example.com/image.jpg",
  "parentId": "section-el-abc123",
  "styles": {
    "align": "center",
    "aspectRatio": "original",
    "imageSize": 100,
    "borderRadius": "0",
    "border": "none",
    "margin": "0px",
    "href": "",
    "alt": "Description of image",
    "linkTarget": "_blank"
  }
}
```

**`aspectRatio`**: `"original"` | `"1:1"` | `"3:2"` | `"2:3"` | `"16:9"`
**`imageSize`**: 10–100 (percentage of container width)
**`align`**: `"left"` | `"center"` | `"right"`
**`href`**: optional click-through URL

---

### `logo` — Brand logo
Same as image but semantically a logo.

```json
{
  "id": "logo-el-abc123",
  "type": "logo",
  "content": "https://example.com/logo.png",
  "parentId": "section-el-abc123",
  "styles": {
    "align": "center",
    "imageSize": 60,
    "margin": "0px",
    "href": "https://example.com",
    "alt": "Company Logo",
    "linkTarget": "_blank"
  }
}
```

---

### `divider` — Horizontal separator

```json
{
  "id": "divider-el-abc123",
  "type": "divider",
  "content": "",
  "parentId": "section-el-abc123",
  "styles": {
    "lineThickness": "1px",
    "lineStyle": "solid",
    "lineColor": "#e0e0e0",
    "dividerAlignment": "center",
    "margin": "0px",
    "width": "100%"
  }
}
```

**`lineStyle`**: `"solid"` | `"dashed"` | `"dotted"` | `"double"`

---

### `spacer` — Vertical whitespace

```json
{
  "id": "spacer-el-abc123",
  "type": "spacer",
  "content": "",
  "parentId": "section-el-abc123",
  "styles": {
    "height": "40px",
    "backgroundColor": "transparent"
  }
}
```

---

### `list` — Unordered bullet list
`content` is newline-separated items OR HTML with `<li>` tags.

```json
{
  "id": "list-el-abc123",
  "type": "list",
  "content": "<li>Feature one</li><li>Feature two</li><li>Feature three</li>",
  "parentId": "section-el-abc123",
  "styles": {
    "listStyleType": "disc",
    "typographyStyle": "paragraph",
    "color": "inherit",
    "fontSize": "inherit",
    "margin": "0px",
    "padding": "0px"
  }
}
```

**`listStyleType`**: `"disc"` | `"circle"` | `"square"` | `"none"`

---

### `orderedList` — Numbered list

```json
{
  "id": "ol-el-abc123",
  "type": "orderedList",
  "content": "<li>Step one</li><li>Step two</li><li>Step three</li>",
  "parentId": "section-el-abc123",
  "styles": {
    "typographyStyle": "paragraph",
    "color": "inherit",
    "fontSize": "inherit",
    "margin": "0px",
    "padding": "0px"
  }
}
```

---

### `social` — Social media icons

```json
{
  "id": "social-el-abc123",
  "type": "social",
  "content": "",
  "parentId": "section-el-abc123",
  "styles": {
    "textAlign": "center",
    "iconSize": "48px",
    "iconSpacing": "12px",
    "padding": "10px 0",
    "margin": "0px",
    "socialPlatforms": [
      { "id": "facebook",  "name": "Facebook",  "url": "https://facebook.com/brand",  "enabled": true },
      { "id": "twitter",   "name": "Twitter",   "url": "https://twitter.com/brand",   "enabled": true },
      { "id": "linkedin",  "name": "LinkedIn",  "url": "https://linkedin.com/company/brand", "enabled": true },
      { "id": "instagram", "name": "Instagram", "url": "https://instagram.com/brand", "enabled": true }
    ]
  }
}
```

**Available platform IDs**: `facebook`, `twitter`, `linkedin`, `instagram`, `youtube`, `tiktok`, `whatsapp`, `telegram`, `pinterest`, `discord`, `reddit`

---

### `video` — Video thumbnail with play button

```json
{
  "id": "video-el-abc123",
  "type": "video",
  "content": "https://img.youtube.com/vi/VIDEO_ID/maxresdefault.jpg",
  "parentId": "section-el-abc123",
  "styles": {
    "align": "center",
    "href": "https://www.youtube.com/watch?v=VIDEO_ID",
    "imageSize": 100,
    "aspectRatio": "16:9",
    "borderRadius": "8px"
  }
}
```

---

### `address` — Footer address block

```json
{
  "id": "address-el-abc123",
  "type": "address",
  "content": "Company Name, 123 Main St, City, State 12345",
  "parentId": "section-el-abc123",
  "styles": {
    "textAlign": "center",
    "fontSize": "12px",
    "color": "#666666",
    "lineHeight": "1.5",
    "padding": "0"
  }
}
```

---

## Parent-Child Rules

```
null (root)
  └── section
        ├── text
        ├── header
        ├── button
        ├── image / logo / video
        ├── list / orderedList
        ├── divider / spacer
        ├── social / address
        └── columns
              └── column
                    ├── text
                    ├── header
                    ├── button
                    ├── image / logo
                    └── list / orderedList
```

**Rules**:
- `columns` can ONLY contain `column` children — nothing else
- `column` must always have a `columns` parent
- All other elements are leaf nodes (no children)
- Root-level elements must have `parentId: null`

---

## Style Inheritance with `"inherit"`

Use `"inherit"` on typography properties to defer to template-level values:

```json
"fontSize": "inherit",   // uses template.fontSize
"color": "inherit",      // uses template.color
"fontFamily": "inherit", // uses template.fontFamily
"lineHeight": "inherit"  // uses template.lineHeight
```

When `typographyStyle` is set (e.g., `"paragraph"`), the element resolves its typography from `template.typographyStyles.paragraph`, and any explicit non-inherit value overrides it.

---

## Common Section Patterns

### Header section (logo + nav)
```
section (padding: "20px", backgroundColor: "#ffffff")
  └── logo
```

### Hero section (image + headline + CTA)
```
section (padding: "0px")
  ├── image (full-width hero image)
  section (padding: "40px 20px", gap: "20px")
    ├── header (typographyStyle: heading1, textAlign: center)
    ├── text (typographyStyle: paragraph, textAlign: center)
    └── button (buttonVariant: primary, align: center)
```

### Two-column feature section
```
section (padding: "40px 20px")
  └── columns (columnPresetId: equal-2, gap: "24px")
        ├── column
        │     ├── image
        │     ├── header (typographyStyle: heading3)
        │     └── text
        └── column
              ├── image
              ├── header (typographyStyle: heading3)
              └── text
```

### Footer section
```
section (padding: "20px", backgroundColor: "#f8f8f8")
  ├── divider
  ├── social
  ├── address
  └── text (unsubscribe link, fontSize: "12px", textAlign: "center", color: "#999999")
```

---

## Personalization Variables

Available merge tags for `content` strings:
- `{{contact.first_name || "there"}}` — first name with fallback
- `{{contact.last_name}}`
- `{{contact.email}}`
- `{{contact.company}}`
- `{{unsubscribe_url}}` — required in every footer
- `{{web_view_url}}` — view in browser link

Unsubscribe link pattern (required by CAN-SPAM):
```html
<a href="{{unsubscribe_url}}" target="_blank" style="color: #0066CC; text-decoration: underline;">Unsubscribe</a>
```

---

## Gmail-Safe Style Guidelines

**Use**: `backgroundColor`, `color`, `font-*`, `padding`, `margin`, `border`, `width`, `text-align`, `line-height`
**Avoid in critical paths**: `display: flex`, `grid`, `position: absolute/fixed`, `transform`, `animation`, `transition`, `opacity` (limited support in Outlook/Gmail)

**Font families** (email-safe fallbacks):
- Sans-serif: `"Arial, Helvetica, sans-serif"` | `"Tahoma, Geneva, sans-serif"` | `"Verdana, Geneva, sans-serif"`
- Serif: `"Georgia, 'Times New Roman', serif"` | `"'Palatino Linotype', Palatino, serif"`
- Monospace: `"'Courier New', Courier, monospace"`

---

## ID Generation

Generate IDs like: `{type}-el-{6randomchars}`

Examples:
- `section-el-a1b2c3`
- `text-el-x9y8z7`
- `image-el-p4q5r6`
- `button-el-m7n8o9`
- `columns-el-k1l2m3`
- `column-el-f4g5h6`

Use short, random alphanumeric suffixes. Every ID must be unique within the template.

---

## Output Format

When generating a template, output:

1. **Brief design rationale** (2-3 sentences: layout decisions, color choices, structure)
2. **Complete template JSON** in a code block — valid, ready to use

The JSON must be complete: all elements with IDs, styles, parentId, and content.

---

## Example: Simple Newsletter Template

```json
{
  "name": "Monthly Newsletter",
  "subject": "Your Monthly Update",
  "preview_text": "Here's what's new this month",
  "containerWidth": "600px",
  "backgroundColor": "transparent",
  "canvasBackgroundColor": "#f0f0f0",
  "padding": "0px",
  "fontFamily": "Arial, Helvetica, sans-serif",
  "fontSize": "16px",
  "lineHeight": "1.6",
  "color": "#333333",
  "textAlign": "left",
  "typographyStyles": {
    "paragraph": { "fontSize": "16px", "color": "#333333", "fontWeight": "400", "lineHeight": "1.6" },
    "heading1":  { "fontSize": "32px", "color": "#111111", "fontWeight": "700", "lineHeight": "1.2" },
    "heading2":  { "fontSize": "24px", "color": "#111111", "fontWeight": "600", "lineHeight": "1.3" },
    "heading3":  { "fontSize": "20px", "color": "#222222", "fontWeight": "600", "lineHeight": "1.4" },
    "heading4":  { "fontSize": "18px", "color": "#222222", "fontWeight": "500", "lineHeight": "1.4" }
  },
  "buttonStyles": {
    "primary":   { "backgroundColor": "#007BFF", "color": "#ffffff", "borderColor": "#007BFF", "borderWidth": "0px", "borderRadius": "4px", "fontSize": "16px", "fontWeight": "600", "paddingX": "24px", "paddingY": "12px" },
    "secondary": { "backgroundColor": "#ffffff", "color": "#007BFF", "borderColor": "#007BFF", "borderWidth": "2px", "borderRadius": "4px", "fontSize": "16px", "fontWeight": "600", "paddingX": "24px", "paddingY": "12px" },
    "tertiary":  { "backgroundColor": "transparent", "color": "#007BFF", "borderColor": "transparent", "borderWidth": "0px", "borderRadius": "4px", "fontSize": "16px", "fontWeight": "600", "paddingX": "24px", "paddingY": "12px" }
  },
  "elements": [
    {
      "id": "section-el-hdr001",
      "type": "section",
      "content": null,
      "parentId": null,
      "styles": { "width": "100%", "padding": "24px 20px", "gap": "0", "backgroundColor": "#007BFF", "margin": "0px", "borderRadius": "0" }
    },
    {
      "id": "logo-el-hdr002",
      "type": "logo",
      "content": "https://via.placeholder.com/150x50",
      "parentId": "section-el-hdr001",
      "styles": { "align": "center", "imageSize": 50, "margin": "0px", "href": "https://example.com", "alt": "Company Logo", "linkTarget": "_blank" }
    },
    {
      "id": "section-el-hero01",
      "type": "section",
      "content": null,
      "parentId": null,
      "styles": { "width": "100%", "padding": "48px 32px", "gap": "20px", "backgroundColor": "#ffffff", "margin": "0px" }
    },
    {
      "id": "header-el-hero02",
      "type": "header",
      "content": "Hey {{contact.first_name || \"there\"}} 👋",
      "parentId": "section-el-hero01",
      "styles": { "typographyStyle": "heading1", "textAlign": "left", "margin": "0px", "color": "inherit", "fontFamily": "inherit" }
    },
    {
      "id": "text-el-hero03",
      "type": "text",
      "content": "Here's what's new this month. We've been busy shipping features and we wanted to share the highlights with you.",
      "parentId": "section-el-hero01",
      "styles": { "typographyStyle": "paragraph", "textAlign": "left", "margin": "0px", "color": "inherit", "fontFamily": "inherit" }
    },
    {
      "id": "button-el-hero04",
      "type": "button",
      "content": "Read the Full Update",
      "parentId": "section-el-hero01",
      "styles": { "buttonVariant": "primary", "buttonSize": "medium", "buttonFit": "compact", "href": "https://example.com/update", "linkTarget": "_blank", "align": "left" }
    },
    {
      "id": "section-el-div001",
      "type": "section",
      "content": null,
      "parentId": null,
      "styles": { "width": "100%", "padding": "0 32px", "backgroundColor": "#ffffff", "margin": "0px", "gap": "0" }
    },
    {
      "id": "divider-el-div002",
      "type": "divider",
      "content": "",
      "parentId": "section-el-div001",
      "styles": { "lineThickness": "1px", "lineStyle": "solid", "lineColor": "#e8e8e8", "width": "100%", "margin": "0px" }
    },
    {
      "id": "section-el-ftr001",
      "type": "section",
      "content": null,
      "parentId": null,
      "styles": { "width": "100%", "padding": "24px 20px", "gap": "12px", "backgroundColor": "#f8f8f8", "margin": "0px" }
    },
    {
      "id": "social-el-ftr002",
      "type": "social",
      "content": "",
      "parentId": "section-el-ftr001",
      "styles": {
        "textAlign": "center", "iconSize": "40px", "iconSpacing": "10px", "padding": "0", "margin": "0px",
        "socialPlatforms": [
          { "id": "twitter",  "name": "Twitter",  "url": "https://twitter.com/brand",  "enabled": true },
          { "id": "linkedin", "name": "LinkedIn", "url": "https://linkedin.com/company/brand", "enabled": true },
          { "id": "facebook", "name": "Facebook", "url": "https://facebook.com/brand", "enabled": true }
        ]
      }
    },
    {
      "id": "text-el-ftr003",
      "type": "text",
      "content": "You're receiving this because you subscribed.<br>Company Inc., 123 Main St, City, State 12345.<br><a href=\"{{unsubscribe_url}}\" target=\"_blank\" style=\"color: #999999;\">Unsubscribe</a>",
      "parentId": "section-el-ftr001",
      "styles": { "fontSize": "12px", "color": "#999999", "textAlign": "center", "fontFamily": "inherit", "margin": "0px", "padding": "0", "lineHeight": "1.6" }
    }
  ]
}
```

---

## Google Fonts

Google Fonts work in Gmail (web), Apple Mail, iOS Mail, and Outlook 2016+ on Mac. **Outlook on Windows will fall back to the system font** — always provide a safe fallback stack.

### How to use Google Fonts

Set `fontFamily` on the template root (and optionally override on individual elements):

```json
"fontFamily": "'Inter', Arial, sans-serif"
```

The font name MUST be wrapped in single quotes if it contains spaces:
```json
"fontFamily": "'Playfair Display', Georgia, serif"
```

### Popular Google Fonts for Email

**Modern Sans-Serif** (clean, readable):
| Font | fontFamily value | Best for |
|------|-----------------|----------|
| Inter | `"'Inter', Arial, sans-serif"` | SaaS, tech, newsletters |
| Poppins | `"'Poppins', Arial, sans-serif"` | Startup, e-commerce, modern |
| Nunito | `"'Nunito', Arial, sans-serif"` | Friendly, consumer apps |
| DM Sans | `"'DM Sans', Arial, sans-serif"` | Minimal, professional |
| Plus Jakarta Sans | `"'Plus Jakarta Sans', Arial, sans-serif"` | Premium SaaS |

**Elegant Serif** (editorial, luxury):
| Font | fontFamily value | Best for |
|------|-----------------|----------|
| Playfair Display | `"'Playfair Display', Georgia, serif"` | Luxury, editorial, fashion |
| Lora | `"'Lora', Georgia, serif"` | Blog posts, thought leadership |
| Merriweather | `"'Merriweather', Georgia, serif"` | Publishing, long reads |

**Display / Heading Only** (use only for large headings, NOT body):
| Font | fontFamily value | Best for |
|------|-----------------|----------|
| Oswald | `"'Oswald', Arial, sans-serif"` | Bold hero headlines |
| Raleway | `"'Raleway', Arial, sans-serif"` | Events, premium products |
| Space Grotesk | `"'Space Grotesk', Arial, sans-serif"` | Tech, Web3, developer tools |

### Typography Pairing Recommendations

**Pairing 1 — Classic SaaS** (clean, trustworthy):
- Heading: `'Inter', Arial, sans-serif` at 700 weight
- Body: `'Inter', Arial, sans-serif` at 400 weight

**Pairing 2 — Editorial** (premium, readable):
- Heading: `'Playfair Display', Georgia, serif` at 700 weight
- Body: `'Lora', Georgia, serif` at 400 weight

**Pairing 3 — Modern Bold** (energetic, startup):
- Heading: `'Poppins', Arial, sans-serif` at 700 weight
- Body: `'Poppins', Arial, sans-serif` at 400 weight

**Pairing 4 — Professional** (corporate newsletters):
- Heading: `'DM Sans', Arial, sans-serif` at 600–700 weight
- Body: `'DM Sans', Arial, sans-serif` at 400 weight

### Applying fonts in typographyStyles

To use one font for headings and another for body:

```json
"fontFamily": "'Inter', Arial, sans-serif",
"typographyStyles": {
  "paragraph": { "fontSize": "16px", "color": "#444444", "fontWeight": "400", "lineHeight": "1.7" },
  "heading1":  { "fontSize": "36px", "color": "#111111", "fontWeight": "700", "lineHeight": "1.15" },
  "heading2":  { "fontSize": "26px", "color": "#111111", "fontWeight": "700", "lineHeight": "1.25" },
  "heading3":  { "fontSize": "20px", "color": "#222222", "fontWeight": "600", "lineHeight": "1.35" },
  "heading4":  { "fontSize": "17px", "color": "#333333", "fontWeight": "600", "lineHeight": "1.4" }
}
```

To override font on a single element, set `fontFamily` directly in the element's `styles`:
```json
"styles": {
  "typographyStyle": "heading1",
  "fontFamily": "'Playfair Display', Georgia, serif",
  "textAlign": "center"
}
```

### Client Support Notes

| Client | Google Fonts |
|--------|-------------|
| Gmail (web) | ✅ Supported |
| Apple Mail (Mac/iOS) | ✅ Supported |
| Outlook Mac | ✅ Supported |
| Outlook Windows | ❌ Falls back to system font |
| Samsung Email | ⚠️ Partial |
| Yahoo Mail | ⚠️ Partial |

**Rule**: Always provide a safe fallback. `'Inter', Arial, sans-serif` — if Inter fails, Arial renders.

---

## Modern Email Design Principles

### Typography Scale

Use a clear hierarchy — don't use more than 3 font sizes in one section:

| Role | Size | Weight | Use |
|------|------|--------|-----|
| Super headline | 40–48px | 800 | One per email max, hero only |
| Heading 1 | 28–36px | 700 | Section intros |
| Heading 2 | 22–26px | 600–700 | Sub-sections |
| Body | 15–17px | 400 | All body copy |
| Small / Legal | 12–13px | 400 | Footer, fine print |

**Line height**: 1.15–1.25 for headings, 1.6–1.75 for body (increases readability significantly)

### Color System

Define a clear 4-color palette per template:

```
Primary   — brand color for buttons, links, accents (e.g., #6B4FBB)
Neutral   — text color (e.g., #1A1A2E for dark, #666666 for light)
Surface   — background (e.g., #FFFFFF or #F8F6FF for tinted)
Separator — dividers and borders (e.g., #E8E8E8)
```

**Dark section on light template** — alternate `backgroundColor` between sections:
- Body sections: `#ffffff`
- Hero/accent sections: brand color or deep dark (`#1A1A2E`)
- Footer: light gray (`#f5f5f7`)

**Text on dark backgrounds**: set `color: "#ffffff"` on the section's child elements explicitly.

### Visual Hierarchy Rules

1. **One CTA per section** — don't stack multiple buttons
2. **Contrast sections** — alternate white/tinted backgrounds to break up long emails
3. **Images above text** — leads the eye better than text-first in most contexts
4. **Center-align heroes** — left-align body content for readability
5. **Generous padding** — sections: `48px 32px` minimum, tight emails feel spammy
6. **Short paragraphs** — max 3 lines per text block in emails

### Section Spacing Strategy

```
Header (logo)          → padding: "20px 32px"
Hero (image + CTA)     → padding: "0" for image, "48px 32px" for text section
Content sections       → padding: "40px 32px", gap: "20px"
Feature columns        → padding: "40px 24px", column gap: "24px"
Divider wrapper        → padding: "0 32px"
Footer                 → padding: "28px 24px", gap: "12px"
```

---

## Modern Design Recipes

### Recipe 1 — SaaS Product Update (Clean Minimal)

```
Palette: white + #6B4FBB accent + #F8F6FF tint
Font: 'Inter', Arial, sans-serif

[Header]  white bg, logo left-aligned
[Hero]    #6B4FBB bg, white headline (40px), white subtext, white outlined button
[Divider] 1px #e0d9f5
[Feature] white bg, 3-column equal, icon + heading3 + short text per column
[CTA]     #F8F6FF tint bg, centered heading2 + primary button
[Footer]  white bg, social icons, 12px gray text
```

### Recipe 2 — E-commerce Promo (Bold & Conversion-focused)

```
Palette: #FF4B2B red accent + #1A1A2E dark + #FFF5F3 warm tint
Font: 'Poppins', Arial, sans-serif (bold headings)

[Header]  #1A1A2E bg, white logo, centered
[Hero]    Full-width product image (aspectRatio: 16:9), imageSize: 100
[Offer]   #FF4B2B bg, white big headline ("50% OFF"), white body, white button (secondary variant)
[Products] 2-column columns, image + heading4 + price text + button per column
[Footer]  #1A1A2E bg, white social icons, white/semi footer text
```

### Recipe 3 — Newsletter (Editorial)

```
Palette: #FAFAF8 warm white + #2C2C2C dark text + #C9A84C gold accent
Font: 'Playfair Display' headings + 'Lora' body

[Header]  #2C2C2C bg, logo center, thin gold divider below
[Issue]   White bg, overline text ("ISSUE 12 · MARCH 2026"), big centered headline
[Articles] Full-width image, heading2, 2-line excerpt, "Read more →" tertiary button
[Divider] #e8e4d9 dashed
[Repeat]  Alternate 2+ article blocks
[Footer]  #FAFAF8 bg, address, unsubscribe
```

### Recipe 4 — Welcome / Onboarding (Warm & Personal)

```
Palette: white + #22C55E green accent + #F0FDF4 light green tint
Font: 'Nunito', Arial, sans-serif

[Header]  White bg, logo, no padding extremes
[Hero]    #F0FDF4 tint, left-aligned "Hey {{first_name}} 👋" heading1, warm welcome body text
[Steps]   White bg, ordered list OR 3-column columns with step number + heading + text
[CTA]     #22C55E bg, white headline ("Get started in 2 minutes"), white primary button
[Footer]  Light gray, social, unsubscribe
```

### Recipe 5 — Transactional (Clean & Trust-building)

```
Palette: white + #0F172A near-black + #3B82F6 blue + #F8FAFF near-white
Font: 'DM Sans', Arial, sans-serif

[Header]  White bg, logo left
[Summary] #F8FAFF bg, heading2 ("Your receipt"), order details as text
[Items]   White bg, columns (left-wide: 66/34), item name + qty | price
[Divider]
[Total]   White bg, bold total line
[CTA]     "View Order" primary button, center-aligned
[Support] Tint bg, "Questions? Reply to this email" + support link
[Footer]  Standard
```

---

## Eye-Catching Design Techniques

### Gradient section backgrounds

Use `backgroundGradient` on sections for visual punch:

```json
"styles": {
  "backgroundGradient": "linear-gradient(135deg, #667eea 0%, #764ba2 100%)",
  "padding": "60px 32px",
  "gap": "24px"
}
```

Common gradient pairs:
- Purple: `linear-gradient(135deg, #667eea, #764ba2)`
- Coral: `linear-gradient(135deg, #f093fb, #f5576c)`
- Ocean: `linear-gradient(135deg, #4facfe, #00f2fe)`
- Sunset: `linear-gradient(135deg, #fa709a, #fee140)`
- Dark: `linear-gradient(135deg, #0f0c29, #302b63, #24243e)`
- Green: `linear-gradient(135deg, #11998e, #38ef7d)`

For text on gradient backgrounds, **always set explicit `color: "#ffffff"`** on child header and text elements.

### High-contrast hero pattern

```json
[Dark hero section]
section: backgroundColor "#1A1A2E", padding "64px 32px", gap "24px"
  header: color "#ffffff", typographyStyle "heading1", textAlign "center", fontSize "40px"
  text: color "#CBD5E0", typographyStyle "paragraph", textAlign "center"
  button: buttonVariant "secondary" (white border on dark bg)
```

### Bordered/accented column cards

Give columns visual card styling:

```json
column styles: {
  "backgroundColor": "#ffffff",
  "padding": "24px",
  "borderRadius": "8px",
  "border": "1px solid #e8e8e8",
  "gap": "12px"
}
```

### Accent line above heading

Use a short, colored spacer or divider before a heading to create a visual accent:

```
section (white bg)
  spacer (height: "4px", backgroundColor: "#6B4FBB") ← accent bar
  header (typographyStyle: heading2)
  text
```

### Full-width banner image + overlay text

```
section (padding: "0px")
  image (imageSize: 100, aspectRatio: "16:9")

section (backgroundColor: "#1A1A2E", padding: "48px 32px", gap: "20px")
  header (color: "#ffffff", textAlign: "center")
  text (color: "#CBD5E0", textAlign: "center")
  button (align: "center")
```

### Pill-style button (rounded)

Override button style with high `borderRadius`:

```json
"buttonStyles": {
  "primary": {
    "backgroundColor": "#6B4FBB",
    "color": "#ffffff",
    "borderRadius": "50px",
    "fontSize": "16px",
    "fontWeight": "600",
    "paddingX": "32px",
    "paddingY": "14px"
  }
}
```

---

## Checklist Before Outputting

- [ ] Every element has a unique `id`
- [ ] Every non-root element has a valid `parentId`
- [ ] `columns` elements contain only `column` children
- [ ] `column` elements have a `columns` parent
- [ ] `containerWidth` is `"600px"`
- [ ] Footer includes `{{unsubscribe_url}}`
- [ ] `typographyStyles` and `buttonStyles` are defined
- [ ] No duplicate element IDs
- [ ] `content` is `null` for container elements (section, columns, column)
- [ ] Image `content` is a URL string, not null
