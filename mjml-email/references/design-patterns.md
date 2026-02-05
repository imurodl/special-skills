# MJML Email Design Patterns

## Base Structure

Every template follows this skeleton:

```xml
<mjml>
  <mj-head>
    <mj-attributes>
      <mj-all font-family="'Inter', 'Helvetica Neue', Arial, sans-serif" />
      <mj-text font-size="15px" color="#374151" line-height="1.6" />
      <mj-button
        background-color="#2563eb"
        border-radius="6px"
        font-size="16px"
        font-weight="600"
        inner-padding="14px 28px"
      />
    </mj-attributes>
  </mj-head>
  <mj-body background-color="#f3f4f6">

    <!-- Header: brand logo on colored background -->
    <mj-section background-color="#2563eb" padding="20px">
      <mj-column>
        <mj-image src="{{ logo_url }}" alt="{{ company_name }}" width="160px" padding="0" />
      </mj-column>
    </mj-section>

    <!-- Content: white card -->
    <mj-section background-color="#ffffff" padding="40px 30px 20px">
      <mj-column>
        <!-- title, greeting, message -->
      </mj-column>
    </mj-section>

    <!-- More content sections as needed -->

    <!-- Footer: light background -->
    <mj-section background-color="#f9fafb" padding="24px 20px 16px">
      <mj-column>
        <!-- logo, links, company info, unsubscribe -->
      </mj-column>
    </mj-section>

  </mj-body>
</mjml>
```

## Color System

Use a consistent Tailwind-inspired gray scale with one brand accent:

| Role | Default | Purpose |
|------|---------|---------|
| Brand/Primary | `#2563eb` | Header, buttons, links, accent |
| Text Dark | `#111827` | Headings |
| Text Body | `#374151` | Body text |
| Text Secondary | `#6b7280` | Labels, subtitles |
| Text Muted | `#9ca3af` | Footer, timestamps |
| Background | `#f3f4f6` | Body/page background |
| Card | `#ffffff` | Content sections |
| Box | `#f9fafb` | Info boxes, footer background |
| Highlight | `#eff6ff` | Special content boxes (amounts, alerts) |
| Alert/Urgency | `#dc2626` | Warnings, expiry, urgent info |

Adapt the brand color to the user's project. Everything else stays neutral.

## Component Library

### Title + Greeting

```xml
<mj-section background-color="#ffffff" padding="40px 30px 20px">
  <mj-column>
    <mj-text align="center" font-size="22px" font-weight="700" color="#111827" padding-bottom="8px">
      {{ title }}
    </mj-text>
    <mj-text align="center" color="#6b7280" padding-bottom="8px">
      Hi {{ name }},
    </mj-text>
    <mj-text align="center" color="#6b7280" padding-bottom="24px">
      {{ intro_message }}
    </mj-text>
  </mj-column>
</mj-section>
```

### Information Box (key-value pairs)

```xml
<mj-section background-color="#ffffff" padding="0 30px">
  <mj-column background-color="#f9fafb" border-radius="8px" padding="20px">
    <mj-text font-size="14px" color="#6b7280" padding="4px 0">
      <strong style="color: #374151">Label</strong>
    </mj-text>
    <mj-text font-size="16px" color="#111827" padding="0 0 12px 0">
      {{ value }}
    </mj-text>
    <!-- more pairs... -->
  </mj-column>
</mj-section>
```

### Highlight Box (amounts, totals)

```xml
<mj-section background-color="#ffffff" padding="16px 30px 0">
  <mj-column background-color="#eff6ff" border-radius="8px" padding="20px">
    <mj-text font-size="14px" color="#6b7280" padding="0 0 12px 0">
      Subtotal: {{ subtotal }}
    </mj-text>
    <mj-divider border-color="#2563eb" border-width="1px" padding="0 0 12px 0" />
    <mj-text font-size="20px" font-weight="700" color="#2563eb" padding="0">
      Total: {{ total }}
    </mj-text>
  </mj-column>
</mj-section>
```

### CTA Button

```xml
<mj-section background-color="#ffffff" padding="24px 30px 8px">
  <mj-column>
    <mj-button href="{{ action_url }}">
      {{ button_text }}
    </mj-button>
  </mj-column>
</mj-section>
```

### Secondary Button

```xml
<mj-button href="{{ url }}" background-color="#6b7280">
  {{ secondary_text }}
</mj-button>
```

### Link Fallback

```xml
<mj-section background-color="#ffffff" padding="0 30px">
  <mj-column background-color="#f9fafb" border-radius="8px" padding="16px">
    <mj-text font-size="13px" color="#6b7280" padding="0">
      If the button doesn't work, copy and paste this link into your browser:
    </mj-text>
    <mj-text font-size="13px" color="#2563eb" padding="8px 0 0 0">
      <a href="{{ link }}" style="color: #2563eb; word-break: break-all">{{ link }}</a>
    </mj-text>
  </mj-column>
</mj-section>
```

### Feature/Checklist Box

```xml
<mj-section background-color="#ffffff" padding="24px 30px 20px">
  <mj-column>
    <mj-text font-size="14px" font-weight="600" color="#111827" padding-bottom="12px">
      What's included:
    </mj-text>
    <mj-text font-size="13px" color="#6b7280" padding="0">
      ✓ Feature one<br/>
      ✓ Feature two<br/>
      ✓ Feature three
    </mj-text>
  </mj-column>
</mj-section>
```

### Footer

```xml
<mj-section background-color="#f9fafb" padding="24px 20px 16px">
  <mj-column>
    <mj-image src="{{ logo_url }}" alt="{{ company_name }}" width="100px" padding-bottom="16px" />
    <mj-text align="center" font-size="12px" color="#6b7280" padding-bottom="16px">
      <a href="{{ terms_url }}" style="color: #6b7280; text-decoration: none">Terms</a> ·
      <a href="{{ privacy_url }}" style="color: #6b7280; text-decoration: none">Privacy</a> ·
      <a href="{{ help_url }}" style="color: #6b7280; text-decoration: none">Help</a>
    </mj-text>
    <mj-text align="center" font-size="11px" color="#9ca3af" padding="0">
      {{ company_name }} | {{ company_address }}
    </mj-text>
    <mj-text align="center" font-size="11px" color="#9ca3af" padding="4px 0 16px 0">
      {{ support_info }}
    </mj-text>
    <mj-text align="center" font-size="11px" color="#9ca3af" padding="0">
      <a href="{{ unsubscribe_url }}" style="color: #9ca3af; text-decoration: underline">Unsubscribe</a>
    </mj-text>
    <mj-text align="center" font-size="11px" color="#9ca3af" padding="16px 0 0 0">
      © {{ year }} {{ company_name }}. All rights reserved.
    </mj-text>
  </mj-column>
</mj-section>
```

## Spacing Rules

- **Between sections**: Use padding on sections, not spacers
- **Header padding**: `20px`
- **First content section**: `40px 30px 20px` (generous top)
- **Subsequent sections**: `0 30px` or `16px 30px 0`
- **Button sections**: `24px 30px 8px` (or `24px 30px 20px` if last)
- **Final section before footer**: `20px 30px 30px`
- **Inside info boxes**: `20px` padding on column, `4px 0` / `0 0 12px 0` between items
- **Footer**: `24px 20px 16px`

## Typography Scale

| Element | Size | Weight | Color |
|---------|------|--------|-------|
| Title | 22px | 700 | #111827 |
| Subtitle/Box heading | 16px | 600 | #111827 |
| Body text | 15px | normal | #374151 |
| Labels | 14px | normal | #6b7280 |
| Values | 16px | normal | #111827 |
| Large amount | 20px | 700 | brand color |
| Small text/notices | 13px | normal | #9ca3af |
| Footer links | 12px | normal | #6b7280 |
| Footer info | 11px | normal | #9ca3af |

## Template Variables

Use Jinja2 syntax: `{{ variable_name }}` for values, `{% if condition %}...{% endif %}` for conditionals.

Name variables descriptively: `{{ user_name }}`, `{{ order_total }}`, `{{ expiry_date }}`, `{{ action_url }}`.
