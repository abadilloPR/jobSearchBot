# LinkedIn Job Bot 🤖

Fetches LinkedIn job postings daily, scores them against your resume using Claude,
and emails you a digest of only the roles where you'd be a strong candidate.

---

## How It Works

1. **Fetch** — Calls LinkedIn Job Search via RapidAPI (~25 postings per run)
2. **Score** — Claude reads each job description against your resume and scores fit 1–10
3. **Filter** — Only jobs at or above your cutoff score (default: 7/10) are included
4. **Email** — Sends a clean HTML digest with match reasons, concerns, and apply links

---

## Setup (15 minutes)

### 1. Get Your API Keys

**RapidAPI (LinkedIn Jobs):**
1. Go to https://rapidapi.com/jaypat87/api/linkedin-jobs-search
2. Click **Subscribe to Test** → choose the free tier (100 req/month)
3. Copy your `X-RapidAPI-Key` from the code snippet panel

**Anthropic (Claude):**
1. Go to https://console.anthropic.com/settings/keys
2. Click **Create Key** and copy it
3. Make sure your account has credits (a month of daily runs costs ~$1–3)

### 2. Set Up Your Resume

Edit `resume.txt` with your actual experience. Plain text works best — include:
- Skills and technologies
- Job titles and years of experience
- Key accomplishments
- Salary expectations (optional but helps scoring)

### 3. Configure Environment Variables

```bash
cp .env.example .env
```

Open `.env` and fill in all values. For `EMAIL_PASSWORD`:
- Go to https://myaccount.google.com/apppasswords
- Create an App Password for "Mail"
- Use that 16-character password — **not** your real Gmail password

### 4. Install Dependencies

```bash
pip install anthropic requests python-dotenv
```

### 5. Run a Test

```bash
python job_bot.py
```

You should see output like:
```
==================================================
  LinkedIn Job Bot — June 09, 2025
==================================================
📄 Resume loaded (1243 chars)

🔍 Searching: 'Software Engineer' in 'New York'...
   Found 25 postings

🤖 Scoring with Claude (cutoff: 7/10, salary min: $120,000)...
   [1/25] Senior Engineer @ Stripe ... ✅ 9/10
   [2/25] Junior Developer @ StartupXYZ ... skip
   ...

📊 8 job(s) met your criteria
📧 Sending digest email...
✅ Email sent to you@gmail.com
```

---

## Schedule It Daily (Free via GitHub Actions)

1. Push this repo to a **private** GitHub repository
2. Go to **Settings → Secrets and variables → Actions**
3. Add each key from `.env.example` as a repository secret
4. The workflow in `.github/workflows/daily_job_bot.yml` will run Mon–Fri at 9 AM UTC

To change the time, edit the cron expression:
```yaml
- cron: "0 9 * * 1-5"   # min hour day month weekday
```
Use https://crontab.guru to build your schedule.

---

## Tuning Tips

| Setting | Effect |
|---|---|
| `RELEVANCE_CUTOFF=8` | Stricter — only very strong matches |
| `RELEVANCE_CUTOFF=6` | Looser — more results, more noise |
| `SALARY_MIN=0` | Ignore salary in scoring |
| Add more keywords | Separate multiple searches by running main() in a loop |

### Multiple Job Types
To search for multiple roles, duplicate the search/score loop in `main()`:

```python
for keywords in ["Software Engineer", "Backend Engineer", "Platform Engineer"]:
    jobs = fetch_jobs(keywords, JOB_LOCATION)
    # ... rest of the loop
```

---

## Cost Estimate

| Service | Free Tier | Paid |
|---|---|---|
| RapidAPI LinkedIn Jobs | 100 req/month = ~22 weekday runs | $10/mo for 1,000 |
| Anthropic Claude | — | ~$0.05–0.10/day (25 jobs × scoring) |
| GitHub Actions | 2,000 min/month free | Free for this use case |

Total estimated cost: **~$1–3/month**

---

## Troubleshooting

**`FileNotFoundError: resume.txt`** — Make sure `resume.txt` is in the same folder as `job_bot.py`

**`SMTPAuthenticationError`** — You're using your real Gmail password. Use an App Password instead (see Step 3).

**`RapidAPI 429`** — You've hit the free tier limit. Upgrade or reduce run frequency to 3x/week.

**Empty results** — Try a broader `JOB_KEYWORDS` or lower `RELEVANCE_CUTOFF`.
