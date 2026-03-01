# Newsletter Distribution & CRM Workflow

> Automated multi-platform distribution for The Persistent Ghost newsletter.

---

## Platform Matrix

| Platform | Purpose | Automation Level | Account |
|----------|---------|-----------------|---------|
| **Beehiiv** | Email newsletter + CRM | Manual paste (API = Enterprise only) | The Persistent Ghost |
| **spookyjuice.ai** | Web-hosted archive + subscribe | Automated via deploy | Live |
| **X / Twitter** | Announcement thread | Semi-auto (API or manual) | @SpookyJuiceAI |
| **Discord** | Community announcement | Automated via webhook | SpookyJuice.AI |

---

## 1. CRM & Audience Growth (Beehiiv)

Beehiiv **is** the CRM. It handles:

- **Subscriber management** — list, segment, tag subscribers
- **Email deliverability** — SPF/DKIM/DMARC handled by Beehiiv
- **Analytics** — open rates, click rates, growth metrics
- **Unsubscribe handling** — CAN-SPAM compliant
- **Welcome sequences** — automated onboarding emails

### Email Capture Points

Every page should funnel to the Beehiiv subscribe API:

```
POST https://api.beehiiv.com/v2/publications/pub_98000f92-fc4d-42b6-aeb6-a1d21f2d7f82/subscriptions
Authorization: Bearer <BEEHIIV_API_KEY>

{
  "email": "subscriber@example.com",
  "send_welcome_email": true,
  "utm_source": "<page-name>",
  "utm_medium": "<context>"
}
```

**Where to capture:**
- Newsletter HTML (web version) — `utm_source=newsletter`
- Hub landing page — `utm_source=hub`
- spookyjuice.ai main site — `utm_source=website`
- Discord pinned message — `utm_source=discord`
- X/Twitter bio link — `utm_source=twitter`

### Growing the Audience Organically

1. **Content flywheel:** Newsletter deep dive → Twitter thread → Discord discussion → new subscribers
2. **Cross-promotion:** Every platform links back to subscribe
3. **SEO:** Host newsletters on spookyjuice.ai with proper meta tags
4. **Referral program:** Beehiiv has built-in referral tracking (enable in dashboard)
5. **Lead magnets:** Offer the ClawMart Memory Guide as a free download for subscribers

---

## 2. Beehiiv Publishing Workflow

### For Non-Enterprise Plans (Current)

1. Build HTML locally in `newsletters/issue-XX/`
2. Git tag the release: `git tag newsletter/vX.X.0`
3. Open Beehiiv dashboard → Create New Post
4. Switch to HTML mode → Paste newsletter HTML
5. Configure:
   - **Subject:** `The Persistent Ghost — Issue #X: [Title]`
   - **Preview text:** First sentence of the opening
   - **From:** SpookyJuice / ghost@spookyjuice.ai
   - **Audience:** All active subscribers
6. Send test email to yourself
7. Schedule or send immediately

### For Enterprise Plans (Future — API Automation)

```bash
# Publish via API (Enterprise only)
curl -X POST "https://api.beehiiv.com/v2/publications/pub_98000f92-fc4d-42b6-aeb6-a1d21f2d7f82/posts" \
  -H "Authorization: Bearer $BEEHIIV_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Issue #X: [Title]",
    "subtitle": "The Persistent Ghost — A SpookyJuice Dispatch",
    "body_content": "<html>...</html>",
    "status": "draft",
    "content_tags": ["newsletter", "persistent-ghost"]
  }'
```

---

## 3. Discord Announcement (Webhook)

### One-Time Setup

1. In Discord server → Server Settings → Integrations → Webhooks
2. Create webhook in #newsletter or #announcements channel
3. Name it "The Persistent Ghost"
4. Copy webhook URL
5. Store as env var: `DISCORD_WEBHOOK_URL`

### Automated Post (cURL or script)

```bash
#!/usr/bin/env bash
set -euo pipefail

ISSUE_NUM="$1"
ISSUE_TITLE="$2"
ISSUE_URL="$3"

curl -H "Content-Type: application/json" \
  -d "{
    \"username\": \"The Persistent Ghost\",
    \"avatar_url\": \"https://spookyjuice.ai/assets/logo-300.png\",
    \"embeds\": [{
      \"title\": \"The Persistent Ghost — Issue #${ISSUE_NUM}: ${ISSUE_TITLE}\",
      \"url\": \"${ISSUE_URL}\",
      \"description\": \"New newsletter issue is live. Read it now.\",
      \"color\": 3801876,
      \"footer\": {
        \"text\": \"Subscribe at spookyjuice.ai\"
      }
    }]
  }" \
  "$DISCORD_WEBHOOK_URL"
```

---

## 4. X / Twitter Announcement

### Thread Format (from RELEASE_WORKFLOW.md)

**Tweet 1 (Hook):**
```
The Persistent Ghost — Issue #X: [Title]

[1-sentence hook about the main topic]

Read → [link]

Thread below
```

**Tweet 2 (Deep dive):**
```
This week's deep dive: [topic]

[2-3 key insights]
```

**Tweet 3 (CTA):**
```
Subscribe to The Persistent Ghost at spookyjuice.ai

The ghost does not sleep. The ghost does not forget. The ghost persists.
```

### Automation Options

| Tool | Cost | Automation Level |
|------|------|-----------------|
| Manual post | Free | None |
| Buffer / Typefully | ~$6/mo | Schedule threads |
| X API (Basic) | $100/mo | Full API automation |
| GitHub Action + X API | $100/mo | Triggered on git tag |

**Recommendation:** Start manual or with Buffer. X API pricing is steep for the current stage.

---

## 5. spookyjuice.ai Newsletter Page

### Architecture

Add to the existing site (`infra/coming-soon/`):

```
infra/coming-soon/
├── index.html                    ← Main landing page (exists)
├── newsletter/
│   ├── index.html                ← Newsletter archive page (tiles)
│   └── issue-01/
│       ├── index.html            ← Issue #1 web version
│       └── assets/
│           └── logo-300.png      ← Shared logo
```

### Newsletter Archive Page Features

- Grid of issue tiles with title, date, topic tags, read time
- Beehiiv subscribe form at the top
- Each tile links to the full issue HTML
- RSS feed link for power users

---

## 6. Automated Release Pipeline

### Current (Manual + Scripts)

```
1. Write → newsletters/issue-XX/
2. Build HTML
3. git tag newsletter/vX.X.0
4. Paste into Beehiiv dashboard → Send
5. Run: ./scripts/discord-announce.sh X "Title" "URL"
6. Post X thread manually
7. Deploy newsletter to spookyjuice.ai
```

### Target (GitHub Actions)

```yaml
# .github/workflows/newsletter-release.yml
# Triggered on: push tag matching newsletter/v*
#
# Jobs:
#   1. deploy-web: Copy newsletter HTML to site, deploy
#   2. discord-notify: POST to Discord webhook
#   3. notify-beehiiv: Slack/email reminder to paste into Beehiiv
#   4. tweet-reminder: Slack/email reminder to post X thread
```

### Implementation Priority

1. **Now:** Discord webhook script (5 min setup)
2. **Now:** Deploy newsletter to spookyjuice.ai on push
3. **Later:** GitHub Action for tag-triggered pipeline
4. **Later:** X API automation (when subscriber count justifies $100/mo)
5. **Future:** Beehiiv API automation (when on Enterprise plan)

---

## 7. DNS / Email Deliverability

### Required DNS Records for spookyjuice.ai

If sending from `ghost@spookyjuice.ai` via Beehiiv:

1. **SPF:** Add Beehiiv's sending IPs to your SPF record
2. **DKIM:** Add the DKIM key Beehiiv provides
3. **DMARC:** Set up a DMARC policy
4. **Custom domain:** Configure in Beehiiv dashboard → Settings → Domains

Check Beehiiv's domain setup guide in the dashboard for exact records.

---

## 8. Security Notes

- Move API keys out of `BEEHIVE_CREDENTIALS.md` and into environment variables
- Add `BEEHIVE_CREDENTIALS.md` to `.gitignore`
- For production: proxy Beehiiv subscribe calls through a server-side endpoint
  to avoid exposing the API key in client-side JavaScript
- Current client-side approach is acceptable for launch but should be hardened

---

## Quick Reference

| Action | Command/URL |
|--------|-------------|
| Subscribe API | `POST /v2/publications/pub_.../subscriptions` |
| Create Post API | `POST /v2/publications/pub_.../posts` (Enterprise) |
| Beehiiv Dashboard | beehiiv.com → Publications → The Persistent Ghost |
| Discord Webhook | `./scripts/discord-announce.sh <num> <title> <url>` |
| Git tag release | `git tag -a newsletter/vX.X.0 -m "Issue #X: Title"` |
| Deploy to site | Push to main → auto-deploy (configure in hosting) |
