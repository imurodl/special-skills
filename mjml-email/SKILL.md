---
name: mjml-email
description: Generate professional, responsive MJML email templates. Use when asked to create, design, or build email templates, HTML emails, or MJML files. Covers all email types including transactional (welcome, verification, password reset), billing (invoices, receipts), notifications (alerts, reminders, status updates), marketing (newsletters, promos, announcements), reports (digests, summaries), and e-commerce (order confirmation, shipping).
---

# MJML Email Template Generator

Generate clean, professional MJML email templates that render well across all email clients.

## Process

1. Ask what type of email template is needed if not clear
2. Read [references/design-patterns.md](references/design-patterns.md) for the component library and design system
3. Assemble the template using the components and patterns from the reference
4. Adapt the brand color to match the user's project if known
5. Use Jinja2 syntax (`{{ variable }}`) for dynamic content
6. Output the complete `.mjml` file

## Design Principles

- **One-column layout** - emails are read on mobile, keep it single column
- **White content card** on light gray background - clean and readable
- **Colored header** with logo - instant brand recognition
- **Generous whitespace** - don't crowd elements, let content breathe
- **Clear visual hierarchy** - title → greeting → content → action → footer
- **One primary CTA** per email - don't compete for attention
- **Link fallback** below buttons - some email clients block buttons
- **Info boxes** for structured data (key-value pairs, order details, account info)
- **Consistent typography** - size and weight convey importance, not decoration

## Quality Checklist

Before delivering a template, verify:
- Uses `mj-attributes` for base styles (font, text, button defaults)
- Has header with logo placeholder
- Has proper footer with company info, links, unsubscribe
- All dynamic content uses `{{ variable }}` syntax
- Spacing follows the pattern: generous top padding on first section, tighter between
- Buttons have readable contrast and sufficient padding
- No inline width on `mj-column` unless multi-column is intended
- Font stack includes system fallbacks
