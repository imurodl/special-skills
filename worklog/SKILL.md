---
name: worklog
description: Generate daily worklogs from git commits. Use when asked to create a worklog, daily report, or summarize work done. Supports current project commits or all repos across user's GitHub organizations.
---

# Worklog Generator

Generate readable daily worklogs from git commit history.

## Fetching Commits

### Current Project Only

```bash
git log --since="YYYY-MM-DD 00:00" --until="YYYY-MM-DD 23:59" --author="$(git config user.email)" --format="=== COMMIT ===%n%s%n%b"
```

### All Organization Repos

**Step 1: Discover user's organizations**

```bash
gh api user/orgs --jq '.[].login'
```

**Step 2: Fetch commits from each org**

```bash
gh search commits --author="@me" --owner="ORG_NAME" --committer-date="YYYY-MM-DD..YYYY-MM-DD" --json repository,commit --jq '.[] | "\(.repository.name): \(.commit.message)"'
```

**If user belongs to multiple orgs:**
- Check which orgs have commits that day
- If only one org has commits, use it without asking
- If multiple orgs have commits, ask user which org(s) to include

## Writing the Worklog

Read the full commit messages and synthesize into a coherent worklog. Do NOT just list commit messages.

**Output directly as text. Do NOT create a file.**

### Multi-Repo Layout

When commits span multiple repos:

1. **Main repo** (most commits) - listed normally, no label
2. **Other repos** - listed after, prefixed with "On [label]," derived from repo name

Derive labels by dropping the common org/project prefix:
- `politech-frontend` → "On frontend, ..."
- `politech-admin` → "On admin, ..."
- `politech-landing` → "On landing, ..."
- If repo has no clear suffix, use the full repo name as label

### Rules

**For significant features:**
- 3-5 sentences, enough to show the substance and scope of the work
- Describe what it does so a non-developer can understand, but keep enough meaning to convey the actual work done
- Use plain language but don't strip context that gives meaning to the work
- Include key aspects like what was built, who it affects, and how it works at a high level

**For small fixes:**
- 1-2 sentences
- State what was fixed and briefly why it mattered

**Skip entirely:**
- Makefile, README, docs, configs
- CI/CD, lint, code style
- Test-only changes
- Developer tooling

### Style

- Write so a non-developer manager can understand the work
- Don't use raw code terms (endpoint names, model names, API paths, class names)
- But DO keep enough technical context so the meaning of the work isn't lost
- "Added discount code system with admin management and automatic application to subscriptions" - good (clear, meaningful)
- "Added DiscountCode model with CRUD endpoints under /discounts" - bad (raw code terms)
- "Fixed database issues" - bad (too compressed, lost the meaning)
- "Fixed issue where deleted articles could still cause conflicts when new articles with the same URL were added, preventing proper article tracking" - good (explains what and why)

### Format

Single repo:
```
YYYY-MM-DD

- [Feature description].

- Fixed [what was broken].
```

Multiple repos:
```
YYYY-MM-DD

- [Main repo feature].

- Fixed [main repo fix].

- On frontend, [frontend work].

- On landing, [landing work].
```

### Example

Commits across repos:
```
politech: feat(subscriptions): add discount code system
politech: fix(db): convert article url index to partial for soft delete
politech: fix(migrations): fix promo_request duplicate downgrade
politech-frontend: feat(ui): add discount code input to profile
politech-frontend: fix(mobile): subscription page layout
politech-landing: feat(promo): add campaign banner
```

Output:
```
2026-02-04

- Developed discount code system for promotional campaigns. Admins can create and manage discount codes with expiration dates and usage limits. Users can apply codes to preview reduced subscription prices before committing. When an admin activates a subscription, any applicable discount is automatically applied to the invoice.

- Fixed issue where deleted articles could cause conflicts when new articles with the same URL were added, preventing the system from properly tracking newly crawled articles.

- Fixed promotional request migration ordering issue that was causing errors during database updates.

- On frontend, added discount code input field to candidate profile page so users can apply promotional codes.

- On frontend, fixed mobile layout issue on subscription page that was causing elements to overlap.

- On landing, added promotional banner for the upcoming campaign launch.
```
