---
name: create-announcement
description: >-
  Create a new announcement for the mobile app and website announcement board.
  Use when adding company announcements, TLD launches, educational articles,
  feature announcements, partnership news, or operational updates.
user-invocable: true
---

# Create Announcement

Guide for adding new announcements to the domains-manager-announcements repo. The flutter app and website fetch these JSON files from GitHub raw content.

## Repository Structure

```
company.json   — Company announcements (TLD launches, features, education, partnerships, ops)
alerts.json    — Periodical recurring alerts (promotions, events on a schedule)
```

## Step 1: Determine Announcement Type

Ask the user which type unless obvious from context:

| Type | File | When to Use |
|------|------|-------------|
| **Company** | `company.json` | One-time announcements: TLD launches, blog posts, features, partnerships, operational updates |
| **Periodical Alert** | `alerts.json` | Recurring events: weekly promotions, monthly events, scheduled activities |

## Step 2: Gather Required Information

### For Company Announcements

Collect from user (ask for anything missing):

| Field | Required | Description |
|-------|----------|-------------|
| **title** | Yes | Headline for the announcement |
| **shortMessage** | Yes | 1-2 sentence teaser shown in card preview |
| **fullMessage** | Yes | Full announcement body. Use `\n` for line breaks within the JSON string |
| **link** | Yes | URL to blog post or relevant page (usually `https://unstoppabledomains.com/blog/...`) |
| **linkText** | Yes | CTA button label (e.g., "Learn More", "Get Your .X Domain", "Read the Guide") |
| **date** | No | Defaults to today. ISO 8601 format: `YYYY-MM-DDT00:00:00.000Z` |

### For Periodical Alerts

Collect all company fields above, plus:

| Field | Required | Description |
|-------|----------|-------------|
| **periodicity** | Yes | `daily`, `weekly`, `biweekly`, `monthly`, `bimonthly` |
| **dayOfWeek** | If weekly/biweekly | 1=Monday through 7=Sunday |
| **dayOfMonth** | If monthly | 1-31 |
| **weekOfMonth** | If bimonthly | 1-4 |
| **month** | If bimonthly | 1-12 |
| **eventTime** | No | `HH:MM` format (e.g., `14:30`) |
| **notificationDaysBefore** | Yes | Days before event to show the alert |
| **timezone** | Yes | IANA timezone (e.g., `America/New_York`) |

## Step 3: Generate the ID

### Company announcement IDs

Format: `company_YYYY-MM-DD[_descriptor]`

- Use the announcement date
- Add `_descriptor` suffix if multiple announcements share the same date, or to add clarity
- Descriptor: lowercase, hyphenated, brief (e.g., `_afternic`, `_dns-zone`, `_mobile-sunset`)

Examples:
- `company_2026-04-15` (single announcement on that date)
- `company_2026-04-15_new-tld` (multiple on same date, or for clarity)

### Alert IDs

Format: descriptive kebab-case summarizing the recurring event.

Examples:
- `every-friday-5-dollar-dot-com`
- `monthly-community-call`

## Step 4: Build the JSON Entry

### Company Template

```json
{
  "id": "company_YYYY-MM-DD[_descriptor]",
  "type": "company",
  "date": "YYYY-MM-DDT00:00:00.000Z",
  "title": "Announcement Title",
  "shortMessage": "Brief teaser for card preview.",
  "fullMessage": "Full announcement body.\\n\\nUse double-escaped newlines for line breaks in JSON.\\n\\nMultiple paragraphs supported.",
  "link": "https://unstoppabledomains.com/blog/categories/announcements/article/slug",
  "linkText": "Call to Action Label"
}
```

### Alert Template

```json
{
  "id": "descriptive-kebab-case-id",
  "type": "periodicalAlert",
  "title": "Alert Title",
  "shortMessage": "Brief teaser.",
  "fullMessage": "Full alert body with \\n for newlines.",
  "periodicity": "weekly",
  "dayOfWeek": 5,
  "dayOfMonth": null,
  "weekOfMonth": null,
  "month": null,
  "eventTime": "14:30",
  "notificationDaysBefore": 1,
  "timezone": "America/New_York"
}
```

## Step 5: Insert into the JSON File

### Company announcements (`company.json`)

- **Insert as the FIRST element** in the array (newest first, reverse chronological order)
- The file is a JSON array: `[ {...}, {...}, ... ]`
- Add the new object after the opening `[` bracket

### Periodical alerts (`alerts.json`)

- Add to the array (order less important, but append at end is fine)
- If file is empty array `[]`, replace with `[ { new entry } ]`

## Step 6: Validate

After editing, verify:

1. **Valid JSON** — Run `cat company.json | python3 -m json.tool > /dev/null` to check syntax
2. **Unique ID** — No duplicate IDs in the file
3. **Newlines in fullMessage** — Must be `\\n` (escaped) within the JSON string, NOT literal newlines
4. **Date format** — ISO 8601 with `.000Z` suffix
5. **Link is valid URL** — Starts with `https://`
6. **All required fields present** — No nulls for required fields

## Content Categories (for reference)

Common announcement categories seen in this repo:

- **TLD Launches** — New Web3 or partner TLD introductions
- **Educational Articles** — DNS guides, domain concepts, how-tos
- **Feature Announcements** — New product features or tools
- **Partnerships/Events** — Conference highlights, partner integrations
- **ICANN Applications** — TLD application milestones
- **Operational Updates** — App changes, platform migrations, sunset notices

## Step 7: Commit, Push, and Open PR

### Branch

Create a branch from latest `main`:

```
git fetch origin main && git checkout main && git pull --rebase
git checkout -b mike/<date>-<descriptor>
```

Branch name: `mike/YYYY-MM-DD-add-<brief-descriptor>-announcement` (e.g., `mike/2026-04-15-add-new-tld-launch`).

### Commit

Use conventional commits:

- New announcement: `feat: add [brief description] announcement`
- Fix copy/formatting: `fix: [what was fixed]`
- Remove announcement: `chore: remove [which] announcement`

Stage only the changed JSON file (`company.json` or `alerts.json`).

### Push and PR

```bash
git push -u origin <branch-name>
```

Create PR with `gh pr create`:

- **Title**: Same as commit message (e.g., `feat: add .example TLD launch announcement`)
- **Body**:

```markdown
## Summary
- Add [type] announcement: "[title]"
- Date: YYYY-MM-DD
- Links to: [URL]

## Preview
**Title:** [announcement title]
**Short:** [shortMessage]
**CTA:** [linkText] → [link]

## Notes
- Live once merged — flutter app fetches from `main` every 3 hours
- No build/deploy step needed
```

Set reviewers if user specifies them, otherwise leave default.

## How the App Consumes These

The flutter app fetches raw JSON from GitHub main branch every 3 hours:
- `https://raw.githubusercontent.com/unstoppabledomains/domains-manager-announcements/refs/heads/main/company.json`
- `https://raw.githubusercontent.com/unstoppabledomains/domains-manager-announcements/refs/heads/main/alerts.json`

Changes are live once merged to `main`. No build or deploy step needed.
