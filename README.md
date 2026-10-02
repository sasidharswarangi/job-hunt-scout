# Job hunt

Tracks Software Engineer openings found on LinkedIn. Two Claude Code skills scan your
own logged-in LinkedIn session (through the Claude in Chrome extension) and write what
they find into Markdown files in this folder.

## What's in here

| File | What it is | Written by |
|---|---|---|
| `linkedin_jobs.md` | Main list of SWE openings from your LinkedIn **home feed**. Split into "Leads with email", "Openings without email" and a "Details" section per role. | `linkedin-job-scout` skill |
| `linkedin_email_leads.md` | Posts from LinkedIn **content search** that include a direct contact email, with scam flags. | `linkedin-job-scan` skill |
| `.claude/skills/linkedin-job-scan/` | Project-level skill definition for the email-leads scan. | — |

The `linkedin-job-scout` skill lives in your user-level skills folder
(`~/.claude/skills/linkedin-job-scout/`) along with its helper `update_jobs_readme.py`.

## Setup

1. Install [Claude Code](https://claude.com/claude-code).
2. Install the **Claude in Chrome** extension and connect it to Claude Code.
3. Log in to LinkedIn in that Chrome profile yourself. The skills never type a password or
   fill in login / 2FA fields.
4. Make sure `python3` is on your `PATH` (the scout's helper script uses only the standard
   library).
5. Open Claude Code in this folder:

   ```bash
   cd ~/WORKSPACE/"job hunt"
   claude
   ```

## Usage

### Scan your home feed → `linkedin_jobs.md`

Ask Claude something like:

- "scan my LinkedIn feed for jobs"
- "find SWE openings on LinkedIn, last 100 posts"
- "update my LinkedIn job list"

What happens:

- Scrolls your feed (default **500 posts**, or the number you give) and keeps text/image
  posts about SWE, SDE, Backend/Frontend/Full Stack, MTS and similar roles. Video posts,
  #OpenToWork posts, career advice, and SDET/QA roles are skipped.
- Pulls role, company, location, experience, HR email (only if literally in the post),
  poster and poster type (Hiring Manager / Recruiter/TA / Employee referral / Other).
- Gets a short `lnkd.in` link for each post via "Copy link to post".
- For hiring-manager posts, opens the manager's profile to record their title.
- Runs `update_jobs_readme.py`, which adds new rows to the top of the right table, adds a
  details section, refreshes the counts, and skips posts already in the file.
- Closes its tab and prints a summary (posts scanned, new leads, duplicates, videos skipped).

### Search posts with emails → `linkedin_email_leads.md`

Ask Claude something like:

- "check LinkedIn for job posts with an email"
- "find email-based applications on LinkedIn"

What happens:

- Runs LinkedIn content search (e.g. "software engineer hiring", "send resume Bangalore")
  sorted by date, and reads a few screens of results.
- Keeps posts that are both SWE-related **and** contain an email address.
- Flags likely scams (personal Gmail/Yahoo, dozens of unrelated roles bundled together,
  no named company, "urgent / limited slots" language) instead of dropping them.
- Lists near-misses under "Not included" so you can see what was filtered and why.

## Rules the skills follow

- **Read only.** No likes, comments, reposts, connects, follows, messages, applies or emails.
- **Manual runs only.** Run when you ask, never on a schedule. Repeated automated scraping
  breaks LinkedIn's terms and can get your account flagged.
- **Stops on checks.** A login page, CAPTCHA, "unusual activity", rate limit or
  `/checkpoint/` page ends the scan. Results collected so far are still saved.
- **No guessing.** Blank fields stay blank, and emails are never made up from name patterns.
- **Minimal data.** Only the table fields are stored. No full post text or phone numbers.

## Before you apply

- Treat personal-webmail "send your resume" posts with caution. Check the recruiter and
  agency on LinkedIn before sharing your resume or personal details.
- Prefer the company's own careers page or a named recruiter you can verify.
- Direct-email SWE posts are rare. Most real openings go through an apply link, so the
  "Openings without email" list is worth working through too.
