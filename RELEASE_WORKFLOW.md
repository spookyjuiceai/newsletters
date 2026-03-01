# 🚀 The Persistent Ghost — Newsletter Release Workflow

> Treat every newsletter issue like a software release. This is the complete, step-by-step process.

---

## Overview

Every issue of *The Persistent Ghost* follows this release workflow:

```
Research → Draft → Review → Build → QA → Tag → Distribute → Announce → Track
```

This is a **versioned release process**. Every issue gets a Git tag. Every release has a changelog. No exceptions.

---

## Release Naming Convention

```
Newsletter issues are versioned like software:

v1.0.0  → Issue #1: The Beginning
v1.1.0  → Issue #2 (new feature/content)
v2.0.0  → Major format change or special edition

Git tags: newsletter/v1.0.0
Branches: newsletter/issue-001
```

---

## Phase 1 — Research & Content

### Step 1: Create Issue Branch

```bash
git checkout main
git pull origin main
git checkout -b newsletter/issue-XXX
```

### Step 2: Create Issue Folder

```
issues/
└── XXX-short-title/
    ├── DRAFT.md           ← Raw content draft
    ├── CHECKLIST.md       ← Issue-specific checklist
    └── assets/            ← Images, diagrams
```

### Step 3: Research Phase

- [ ] Identify 1–3 primary topics for the issue
- [ ] Pull from SpookyJuice engineering blog for "what shipped this week"
- [ ] Collect community links, discussions, notable OpenClaw activity
- [ ] Identify any benchmarks or metrics to include
- [ ] Draft key insights (max 3 per issue — readers remember 3 things)

### Step 4: Write DRAFT.md

Structure every issue the same way:

```markdown
# Issue #XXX: [Title]

## Opening (2–3 paragraphs)
- What happened this week
- Why it matters
- The ghost's take

## Deep Dive (main feature)
- One topic, explored thoroughly
- Include diagrams if relevant
- Link to full docs/research

## Platform Updates
- What shipped this week on SpookyJuice
- Engineering blog highlights

## Community & Ecosystem
- Notable community builds
- ClawMart featured listing (if applicable)

## Connect
- Twitter, Discord, email capture CTA

## Closing Quote
```

---

## Phase 2 — Build

### Step 5: Build HTML from Draft

1. Open `issues/XXX-short-title/DRAFT.md`
2. Port content into the newsletter HTML template
3. Update all variable fields:

```html
<!-- Fields to update every issue: -->
<title>The Persistent Ghost — Issue #XXX: [Title]</title>
<div class="issue-badge">Issue #XXX — [Title] — [Month Year]</div>
<div class="li-title">Issue #XXX: [Title]</div>
<!-- ... etc -->
```

4. Update the Hub landing page (`spookyjuice-hub.html`):
   - Change "Latest Issue" section to new issue
   - Add previous issue to archive section
   - Update engineering log entries

### Step 6: Generate QR Code

Use the API (already embedded in templates):

```
https://api.qrserver.com/v1/create-qr-code/
  ?size=150x150
  &data=https%3A%2F%2Fx.com%2FSpookyJuiceAI
  &color=000000
  &bgcolor=ffffff
```

For issue-specific QR codes (e.g., linking to a specific issue):

```
?data=https%3A%2F%2Fspookyjuice.ai%2Fissue-XXX
```

---

## Phase 3 — Quality Assurance

### Step 7: HTML QA Checklist

Before sending, verify all of these:

```
CONTENT
[ ] Title and issue number are correct
[ ] All dates are correct
[ ] All links work (test every <a href>)
[ ] Twitter handle @SpookyJuiceAI links correctly
[ ] ClawMart link works
[ ] Discord username correct (SpookyJuice.AI)
[ ] Engineering blog link works (spookyjuice.ai)
[ ] QR code resolves correctly (scan it)

VISUAL
[ ] Ghost logo renders
[ ] No broken images
[ ] Fonts load (Space Grotesk + Space Mono)
[ ] Dark theme renders correctly in browser
[ ] Mobile view is readable (375px width)
[ ] Scanline overlay appears
[ ] Particle animation runs (hub page)

EMAIL COMPATIBILITY
[ ] Test in Gmail (Chrome)
[ ] Test in Apple Mail
[ ] Plain-text version created (for Beehiiv)
[ ] Unsubscribe link present in footer
[ ] Sender domain (spookyjuice.ai) is correct

METADATA
[ ] <title> tag correct
[ ] OG title/description correct
[ ] Twitter card meta tags correct
```

### Step 8: Beehiiv Pre-Send Check

1. Log into Beehiiv dashboard
2. Create new post → paste HTML (or use Beehiiv editor)
3. Set audience: All subscribers
4. Schedule or send immediately
5. Verify preview email to yourself first

---

## Phase 4 — Release

### Step 9: Git Tag the Release

```bash
# Commit all issue files
git add issues/XXX-short-title/
git add spookyjuice-hub.html
git commit -m "feat(newsletter): Issue #XXX - [Title]"

# Tag the release
git tag -a newsletter/v1.X.0 -m "The Persistent Ghost Issue #XXX: [Title]

- [Key feature 1]
- [Key feature 2]
- [Key feature 3]

Subscribers: [count]
Open rate target: [X]%"

# Push
git push origin newsletter/issue-XXX
git push origin newsletter/v1.X.0
```

### Step 10: Create GitHub Release

1. Go to **Releases → Draft a new release**
2. Tag: `newsletter/v1.X.0`
3. Title: `The Persistent Ghost — Issue #XXX: [Title]`
4. Body (use template below):

```markdown
## 👻 The Persistent Ghost — Issue #XXX: [Title]

**Published:** [Month Day, Year]
**Subscribers:** [count]

### What's in this issue:
- [Bullet 1]
- [Bullet 2]
- [Bullet 3]

### Files included:
- `the-persistent-ghost-issue-XXX.html` — Newsletter HTML
- `spookyjuice-hub.html` — Updated hub page
- `DRAFT.md` — Raw content draft

### Links:
- 🌐 [spookyjuice.ai](https://spookyjuice.ai)
- 🐦 [@SpookyJuiceAI](https://x.com/SpookyJuiceAI)
- 🛒 [ClawMart Listing](https://www.shopclawmart.com/listings/...)
- 💬 Discord: SpookyJuice.AI
```

5. Attach: `the-persistent-ghost-issue-XXX.html` as release asset

---

## Phase 5 — Distribution

### Step 11: Send via Beehiiv

```
Beehiiv send checklist:
[ ] Subject line: "The Persistent Ghost — Issue #XXX: [Title]"
[ ] Preview text: "[First sentence of opening]"
[ ] From name: SpookyJuice
[ ] From email: ghost@spookyjuice.ai (or your verified domain)
[ ] Segment: All active subscribers
[ ] Send time: Tuesday or Thursday, 9–11am local
[ ] A/B test subject line if list > 500
```

### Step 12: Twitter / X Announcement

Post sequence (thread):

```
Tweet 1 (main):
👻 The Persistent Ghost — Issue #1: The Beginning

[1-sentence hook about the main topic]

Read → [link]

Thread below 🧵

---
Tweet 2:
This week's deep dive: [topic]

[2-3 key insights from the issue]

---
Tweet 3:
Also in this issue:
• [item 1]
• [item 2]
• [item 3]

---
Tweet 4 (CTA):
Subscribe to The Persistent Ghost at spookyjuice.ai

The ghost does not sleep. 👻
```

### Step 13: Discord Announcement

Post in #announcements (or #newsletter channel):

```
📰 **The Persistent Ghost — Issue #[X]: [Title]**

[Short description]

→ Read it here: [link]
→ Subscribe: spookyjuice.ai

React 👻 if you read it!
```

---

## Phase 6 — Track & Iterate

### Step 14: Record Metrics (48 hours post-send)

Update `docs/benchmarks/newsletter-metrics.md`:

```markdown
## Issue #XXX — [Title] — [Date]

| Metric | Value | vs. Previous |
|--------|-------|--------------|
| Subscribers | - | - |
| Open rate | - | - |
| Click rate | - | - |
| Unsubscribes | - | - |
| New subs (24h) | - | - |
| Twitter impressions | - | - |
| Twitter engagements | - | - |
| Discord reactions | - | - |
```

### Step 15: Retrospective Note

Add to `CHANGELOG.md`:

```markdown
## newsletter/v1.X.0 — [Date]
### Added
- [New content type or format feature]
### Changed
- [What we changed from last issue]
### Learned
- [What worked, what didn't]
```

---

## Release Schedule

| Week | Activity |
|------|----------|
| Mon–Tue | Research + Draft |
| Wed | Build HTML, QA |
| Thu | Final review, Beehiiv setup |
| **Fri** | **🚀 Send** |
| Mon | Record metrics, plan next issue |

---

## Emergency Fixes

If a bug is found after sending:

```bash
# For web version only (hub page link):
git checkout newsletter/vX.X.0
git checkout -b hotfix/issue-XXX-[description]
# Fix the issue
git commit -m "fix(newsletter): [description]"
git tag newsletter/vX.X.1
git push origin hotfix/issue-XXX-[description]
git push origin newsletter/vX.X.1
```

Note: **You cannot unsend emails.** For serious email errors, send a follow-up "correction" email within 24 hours.

---

> *The ghost ships on Fridays. The ghost does not miss Fridays.*
