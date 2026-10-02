---
name: linkedin-job-scan
description: Scan the user's LinkedIn feed (via Claude in Chrome, the user's own logged-in session) for Software Engineer posts that include a direct contact email, and write a leads file into the job hunt folder. Use when the user asks to check LinkedIn for job posts, find email-based applications, or update their job hunt leads list.
---

# LinkedIn job scan (email leads)

Finds LinkedIn **feed/post** content (not the Jobs board) that advertises a Software
Engineer-type role and includes a direct contact email, and appends results to
`linkedin_email_leads.md` in this folder.

## Preconditions

- This is a **one-time, user-triggered manual pull**, never a recurring/scheduled scrape.
  Automated, repeated scraping of LinkedIn violates its ToS and risks the user's account
  getting flagged. Only run this skill when the user explicitly asks in the current
  conversation.
- Requires the Claude in Chrome extension connected to the user's own logged-in LinkedIn
  session. If `tabs_context_mcp` reports the extension isn't connected, tell the user and
  stop — do not attempt any other scraping method.

## Steps

1. Load Chrome tools if not already loaded:
   `ToolSearch("select:mcp__claude-in-chrome__tabs_context_mcp,mcp__claude-in-chrome__navigate,mcp__claude-in-chrome__computer,mcp__claude-in-chrome__get_page_text,mcp__claude-in-chrome__browser_batch,mcp__claude-in-chrome__tabs_create_mcp,mcp__claude-in-chrome__tabs_close_mcp")`
2. `tabs_context_mcp` (createIfEmpty: true) to get a tab.
3. Navigate to LinkedIn's **content search** (not `/jobs/`), e.g.:
   `https://www.linkedin.com/search/results/content/?keywords=software%20engineer%20hiring&sortBy=%22date_posted%22`
   Vary keywords per the user's target roles/locations (e.g. add "Bangalore", "Hyderabad",
   "send resume").
4. Wait ~2s for the feed to render, then `get_page_text` to pull post text in bulk (cheaper
   and more reliable than screenshots). Scroll (`computer` scroll action) and re-run
   `get_page_text` to page through more results — a few screens is enough for one run,
   this does not need to be exhaustive.
5. From the extracted text, keep only posts where BOTH are true:
   - The post is about a Software Engineer / SWE-adjacent role (not unrelated fields).
   - The post body contains an email address (regex-style: `[\w.+-]+@[\w-]+\.[\w.-]+`).
6. For each kept post, flag scam indicators before presenting it as a real lead:
   - Personal webmail (gmail/yahoo/outlook) instead of a company domain
   - Role bundled with many unrelated job titles in one generic post
   - No named company, urgency language ("limited slots", "apply immediately")
   A post with several of these should be marked "likely scam" rather than dropped —
   the user should still see it and decide.
7. Write/append results to `linkedin_email_leads.md` in this folder using this table format:

   | Poster | Role mentioned | Email | Location | Notes |

   Include a short "Not included" list of near-misses (right role, no email; or email
   present, wrong role) so the user can see what was filtered and why.
8. Summarize findings to the user in chat: how many qualifying posts found, and call out
   any likely-scam entries explicitly — don't just list an email and let the user assume
   it's safe to use.

## What NOT to do

- Do not scrape the LinkedIn Jobs board pages (`/jobs/view/...`) for this — job posts
  there almost never contain raw emails (they use Easy Apply / external ATS links); the
  feed/post search surfaces recruiter posts, which is where emails actually show up.
- Do not click "Apply", "Easy Apply", "Follow", "Connect", "Send", or message anyone
  during this scan — read-only browsing only.
- Do not schedule this to run automatically. If the user wants recurring monitoring,
  point them back to the ToS-safe alternative (parsing their own inbox's LinkedIn job
  alert emails, or polling specific companies' ATS/career pages), not feed scraping.
