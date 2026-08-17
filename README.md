# 📞 Plus Dial — Call Center Analytics Dashboard

An end-to-end data analytics project for a call center operations department: synthetic data generation in **Python**, a relational data model in **PostgreSQL**, and an interactive **Power BI** dashboard used by team leaders and managers to track productivity, quality, and attendance across Calls, Chats, and Emails.

🔗 **Live Dashboard:** [Dashboard](https://app.powerbi.com/view?r=eyJrIjoiMWU2NDM5ZWItNThlYS00ZTllLThhZWUtZDNiNTE3MDJkNTY1IiwidCI6IjQ4MjkzMjgyLTgzMmQtNGQwYi05ZTBmLTVmMmFmYTg5YTFlNCIsImMiOjJ9)

---

## 📊 What the Dashboard Shows

The report is built for team leads and managers who need answers fast, not a data science lecture. It has five pages:

| Page | What it's for |
|---|---|
| **Summary Overview** | The "state of the department" at a glance: total calls/chats/emails, average handling time, overall rating %, contacts by category, contacts by day of week, an attendance donut, and a QA score donut. Filterable by month. |
| **Email / Chat / Call Department** | One page per channel. Top KPI cards (rating, QA score, on-time attendance, volume, yes/no ratings) plus an agent-level table so a lead can scan every agent's numbers for the week and immediately see who needs a check-in. |
| **Agent Activity** | Individual agent drill-down (hire date, line of business, status, historical trend). |
| **QA Department** | Quality scores sliced by team (All / Email / Chat / Phone), an agent leaderboard, a week-over-week QA trend line, a month-over-month QA average, and how many evaluations were completed per month — so managers know both *how well* agents are doing and *how much* auditing is actually happening. |

A **Notification Center** on the left nav also flags anomalies automatically (e.g. "We went down on the CSAT this week"), so leads don't have to go hunting for the bad news.

---

## 🧱 How It's Built — Architecture

```
Python (synthetic data)  →  CSV files  →  PostgreSQL (schema, views, triggers)  →  Power BI (report)
```

This is a classic **star-schema-style warehouse**: a handful of dimension tables (who, what, when) feeding fact tables (what actually happened), with SQL views doing the weekly roll-ups so Power BI doesn't have to.

### 1. Python — Data Generation (`all_viz1_python_codes.ipynb`)

Since this is a portfolio project, there's no real call center data — so the notebook *generates* a realistic one from scratch. In plain terms, each cell builds one CSV file:

- **Calendar** — builds a full daily calendar (year, month, week, quarter, week-start/end, offsets) from a start and end date. This becomes the single source of truth for "what week/month/quarter does this date belong to," so every other table can join to it instead of recalculating dates itself.
- **Categories** — a fixed list of contact reasons (Refunds, Account, Password, etc.).
- **Account Status** — the possible states an agent's account can be in (Active, Supervisor, Vacation, Deactivated, Inactive).
- **Agents** — generates ~115–130 fake employees (using the `Faker` library for realistic names) with a random line of business (Chat, Email, or Phone) and a hire date pulled from the first Monday of each quarter.
- **Attendance** — for every working day (Mon–Fri) each agent was employed, randomly assigns 1–3 absences per month, and labels the rest as "on time" or "late." This becomes the base for who was actually available to take contacts.
- **Production (wk_emails / wk_calls / wk_chats)** — takes only the agents who showed up to work and generates their day's contacts: how many, which category, whether it was rated "yes/no," and — for calls/chats — a random handling time.
- **QA Scores** — every other Friday, generates two quality scores (50–100) per agent, simulating a biweekly quality audit.

The result is a set of CSVs that behave like real call center data: seasonal patterns, absentee days, and messy-but-plausible metrics — without using anyone's real information.

### 2. PostgreSQL — Database (`viz1_sql_code.sql`)

This script takes those CSVs and gives them a proper home. In plain terms:

**Dimension tables** (the "lookup" data):
- `calendar` — the date table described above, now with a primary key and indexes for fast joins.
- `category` — contact reason lookup.
- `status` — agent account status lookup.
- `agents` — the employee roster, linked to `calendar` (hire date) and `status` (account status) via foreign keys.

**Fact tables** (the "what happened" data):
- `attendances` — one row per agent per day: on time / late / absent.
- `qa_score` — one row per agent per audit date, with the two quality scores.
- `wk_chats`, `wk_phone`, `wk_email` — one row per contact handled, linked to the agent, the date, and the category.

**Why foreign keys everywhere?** They keep the data honest — you can't log a chat for an agent who doesn't exist, or file it under a category that isn't real.

**Triggers** (automatic safety checks that run on every insert):
- One blocks an email record from being logged unless that agent is actually in the "Email" department.
- Another blocks *any* production record (calls, chats, etc.) from being logged for an agent whose account is deactivated.

Think of triggers as the database saying "wait — that shouldn't be allowed" before bad data ever gets saved, instead of catching it later in a report.

**Views** (pre-built, reusable queries — the layer Power BI actually reads from):
- `daily_summary` — combines calls, chats, and emails into one table with daily yes/no rating counts per category.
- `calendar_week_help` — a helper that maps every date to the *last day of its work week*, so weekly numbers always roll up to a consistent "week ending" date.
- `week_attendance`, `week_calls`, `week_chats`, `week_emails`, `week_qa_score` — the weekly rollups per agent (average handling time, CSAT/DSAT counts, QA scores) that feed the dashboard's agent tables.
- `work_calendar` — a trimmed version of the calendar that only covers the actual date range present in the data, so filters in Power BI don't show months with no data.

### 3. Power BI — Reporting Layer

Power BI connects directly to the PostgreSQL views (not the raw fact tables), which keeps the report fast and keeps all the business logic — like "what counts as a week" — in one place instead of duplicated inside DAX measures.

On top of that, a dedicated `Calcs` table holds all the DAX measures that power the visuals. A few things worth calling out for anyone reading the model:

- **Reusable calculation "functions."** Instead of writing the CSAT-rate formula three separate times for calls, chats, and emails, the model uses shared helper measures (e.g. `udf_yes_rate`, `udf_status`, `udf_color_variance`, `udf_full_variance`, `udf_last_result`, `udf_avg_aht`) that each channel's measures call with their own numbers. One formula, reused everywhere — so a fix or tweak only has to happen once.
- **Automatic status badges.** The "Good" / "Review" tags in the agent tables (see the Email Department page) aren't manually flagged — they're small SVG images generated on the fly by a measure, colored green or yellow based on each agent's yes/no rating.
- **Built-in week-over-week and month-over-month variance.** Every KPI card (Total Emails, Rating, QA Score, etc.) has a matching "variance" measure that pulls the prior period's number and calculates the change automatically, plus a color measure that turns the number red or green depending on whether it's trending the right way.
- **Smart max/min highlighting.** On the Total Contact trend chart, the highest and lowest points are automatically detected and highlighted (the white dots you see in the Summary Overview) — no manual annotation needed.
- **A dynamic Notification Center.** This is the standout piece: a small lookup table cycles through the department's key rates (overall, email, calls, chats, QA score) and compares each one to its prior period. Whichever metric is underperforming gets automatically surfaced as an alert card — which is exactly what produces the "We went down on the CSAT this week" and "We got lower score from last week" messages you see on the dashboard. Leads don't have to go looking for the bad news; the report finds it for them.
---

## 🛠️ Tech Stack

- **Python** (pandas, numpy, Faker) — synthetic data generation
- **PostgreSQL** — schema design, views, triggers, indexing
- **Power BI** — dashboard and report layer
---

## 💡 Ideas for Future Enhancements

A few directions this project could grow in:

- **Attrition / risk alerts** — extend the existing Notification Center pattern to individual agents, flagging anyone with a declining QA + attendance trend before it becomes a retention problem.
- **Forecasting** — use the `calendar` table's seasonality to forecast next month's contact volume for staffing/scheduling.
- **SLA / service-level tracking** — add a metric for contacts resolved within a target time window per category.
- **Drill-through coaching page** — click an agent in any table to jump to a one-page "coaching card" combining their QA, attendance, and rating history.
- **Row-level security in Power BI** — so each team lead only sees their own agents by default.
- **Automated refresh pipeline** — replace the manual CSV export step with a scheduled Python job (or dbt) that loads straight into PostgreSQL.
---

## 📁 Repo Structure

```
├── all_viz1_python_codes.ipynb   # Synthetic data generation (Calendar, Agents, Attendance, Production, QA)
├── viz1_sql_codes.sql                  # Schema, indexes, triggers, and views (PostgreSQL)
└── README.md
```

## ⚠️ Known Limitations

This is a portfolio/demo project, so a few things are simplified on purpose:
- All data is synthetic (via `Faker`) — no real customer or employee information is used.
- The SQL script assumes a bulk `COPY` step to load the generated CSVs (path left as a placeholder) — swap in your own file paths before running it.
- A couple of column-name references in the raw script need to match the final `calendar` table's column names (`months_name`, `days_week`) if you're running it top to bottom yourself.
---

### 🙋 About This Project

Built as an end-to-end analytics exercise: generate the data, model it properly in a relational database, and turn it into a report a manager can actually use — the same lifecycle a real call center analytics project follows.
