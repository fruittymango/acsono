# Jobs v2: setup prompt for new users

A prompt a new user pastes into Claude to set up their own job search: a watchlist Google Sheet in the layout `/jobs` reads, two job-search skills, and two scheduled tasks. It goes with the [v2 multi-user plan](jobs-v2-multi-user.md).

## Before you start

- Attach your CV, and your cover letter and job preferences if you have them.
- Connect Google Drive and Google Sheets in Claude. Indeed and Gmail are optional.
- Step 7 uses "Use an existing sheet" on the dashboard, which arrives with v2 milestone 1. Until then the dashboard only opens the owner's sheet.

## The prompt

Copy everything in the block below.

```text
Set up a personal job search for me, the same way it works on acsono.co.za/jobs: a watchlist Google Sheet, two job-search skills, and two scheduled tasks that search for me and send me what's new.

I've attached my CV, and maybe a cover letter and a list of job preferences. Use only facts from these and from my answers. Never invent experience, numbers, employers or qualifications.

Work through these steps in order. Ask your questions in one round, using the question tool if you have it, and wait for my answers before building anything.

1. READ MY DOCUMENTS
- Read my CV, cover letter and preferences.
- Pull out: my name, phone, email, location; years of experience; skills and tools; each job with its main achievements and numbers; named projects or products; qualifications and studies in progress; languages; links (portfolio, LinkedIn, GitHub).
- Note anything that disagrees between the documents (dates, job titles, numbers). Ask me about those; don't pick one yourself.

2. ASK ME WHAT'S MISSING
Only ask what the documents don't already answer:
- Roles I'm looking for, in priority order, and roles I'd consider stretching into.
- Industries I want, and any I want to avoid.
- Where I'll work: cities, whether I'd relocate, remote, hybrid.
- Seniority I'm aiming for.
- Current and expected pay. This is for information only: never hide or rank roles by pay.
- Notice period and when I can start.
- Recruitment agencies: skip them, or show them separately?
- Deal-breakers (for example: no night shifts, no commission-only).
- My time zone, and what time I want my daily results.

3. CONFIRM MY PROFILE
- Show me a short profile: who I am, my strongest selling points with one fact each, my career direction in priority order, and my main gaps for the roles I want.
- Wait for me to correct it before going on.

4. SET UP MY WATCHLIST SHEET
Use the Google Drive and Google Sheets connectors. If they aren't connected, tell me which to connect and stop until I have.
- If I gave you a sheet link, use that sheet. Check its tabs and headers against the layout below and only add what's missing.
- Otherwise create a Google Sheet called "Job Watchlist - <my first name>" with these four tabs, header in row 1, header row frozen:
  - "Companies", columns A-M: Company, Priority, Status, Role seen, What they do, Found via, Post link, Posted, Contact, Careers page, Last checked, Applied on, Notes.
    - Priority is a dropdown: High, Medium, Low, Avoid.
    - Status is a dropdown: Not checked, No other openings, Has other openings, Applied, Not interested.
    - Turn on a filter for the header row.
  - "Run log", columns A-G: Date, Company, Type, Title, Link, Fit (0-10), Note. Type is one of: Job, Contract, Freelance, News, Added.
  - "Documents", columns A-H: Name, Kind, Link, Subject, Body, Updated, Notes, Industry. Kind is one of: CV, Cover letter, Email, Paragraph. Starter rows:
    - CV: leave Link empty, and note "Upload your CV PDF to Drive and paste its link".
    - Application email: a short email for when the CV and cover letter are attached. Use the placeholders [Name], [job title], [Company], [one concrete thing from the ad]. Write it from my documents.
    - Cover letter (text): one page, written from my real experience. Include an [industry paragraph] slot.
    - One Paragraph row per industry I'm targeting, put in the Industry column. Each paragraph uses my real experience for that industry.
  - "Profile", key/value pairs in columns A-B: Full name, Phone, Email, Signature, Cover letter file name ("{name} - Cover letter - {company}"), Schema version (2).
- Research employers that fit my direction and location. Aim for 40 to 100 real companies: industry leaders, mid-sized firms, and companies that sell to or supply my target industries. Add each one as a Companies row:
  - Found via = "Target list"; Status = Not checked; Role seen = "No role seen yet".
  - Priority: High for my top direction, Medium for the rest.
  - Careers page: only a URL you actually found on a real page; otherwise leave it empty.
- Read the sheet back and check that it's correct. Give me its link.

5. WRITE MY SKILLS
Use the skill-proposal tool to propose two new skills, named after me.

"<first name>-job-search" finds jobs posted directly by employers and ranks them for me. It must include:
- My confirmed profile: facts, named projects with numbers, education, direction in priority order, locations, pay for information only.
- A salary rule: never exclude or down-rank a role for pay. Show the stated salary or "not stated"; flag pay below my current pay as information only.
- Where to search, chosen for my field and country:
  - the Indeed connector if it's connected;
  - employer careers pages and their application systems (Greenhouse, Lever, Workable, SmartRecruiters, Workday, SuccessFactors, BambooHR);
  - the main local job boards for my field, for example PNet, Careers24 or LinkedIn Jobs in South Africa;
  - public LinkedIn hiring posts, searched as site:linkedin.com/posts hiring <role> <city>;
  - Google searches pairing a role with a city and "jobs" or "careers";
  - any community or niche boards for my field;
  - my target employers.
- How to sort out who posted a job:
  - direct employer;
  - agency with a named client (only if I want agency roles);
  - anonymous agency (excluded, but counted);
  - aggregators and reposts (leads only, until found on the employer's own site);
  - likely scams (personal email address posing as a company, pay that's too good to be true, someone else doing the work under my name, upfront fees).
- Freshness: drop anything closed, past its closing date, or dated more than 45 days ago.
- Fit score out of 10:
  - 0-3 for skills and tools match;
  - 0-3 for industry match to my direction;
  - 0-2 for seniority match;
  - 0-1 for location;
  - 0-1 for extras;
  - minus points for hard requirements I don't meet (for example a required degree or licence). Flag these, don't hide them.
  - Always name the main gap.
- How to spot the same job seen before, so I'm never told about a role twice:
  - the links match once tracking parameters are removed; or
  - company + job title + place match, ignoring case, punctuation and suffixes like (Pty) Ltd.
- The sheet: its ID; read the Run log first and skip roles already logged; skip companies marked Not interested and roles I've applied for.
- Logging: every new role with fit 5 or more goes in the Run log, and every new direct employer goes in Companies.
- Safe sheet writing:
  - re-read the tab right before writing;
  - find rows by the company name in column A, never by a remembered row number;
  - never overwrite my Status, Priority or Notes (add to Notes instead);
  - never change formatting or dropdowns.
- Output: a table with Fit, Role, Company, Location, Salary, Posted, Why it fits (linked to a real fact from my CV) and main gap, and Apply link. After the table: one line for anonymous agencies excluded, and a list of sources.
- Applying, only when I ask:
  - read the full post first;
  - draft the email from my Documents templates;
  - save it as a Gmail draft with my CV attached, under a clean file name;
  - re-read the draft before sending;
  - send only after I say "send" for that email;
  - afterwards set Status to Applied and fill in Applied on.
- Rules: read-only on every website. Never apply, message, comment or post for me without my explicit go-ahead.

"<first name>-company-watch" checks each watched company for new roles and news:
- Every High company, then the Medium ones checked longest ago, about 30 per run.
- Where it checks: careers page, Indeed, public LinkedIn, local boards, and news from the last 30 days (funding, contracts won, expansion, leadership changes, layoffs).
- In the sheet it updates: Last checked, Careers page (real URLs only), Status (never overwriting Applied or Not interested) and Notes (added to, not replaced).
- It logs new roles and news in the Run log and sends a short summary.

6. SCHEDULE THEM
Create two scheduled tasks with push notifications. Each gets a complete, standalone prompt that names its skill, the sheet ID, my profile in two lines, the read-only rules, and how to report back:
- "Daily job search": every day at the time I chose, in my time zone. It reports only new roles: the ranked table, or one line if nothing is new.
- "Company watch": twice a week, for example Tuesday and Friday mornings.
Tell me the exact times you set and whether runs need my approval.

7. CONNECT THE DASHBOARD AND FINISH
Tell me how to open my dashboard: go to https://www.acsono.co.za/jobs, sign in with my own Google account, and choose "Use an existing sheet" to pick my watchlist.

Then give me a short wrap-up:
- the sheet link;
- the skills to save from the review card;
- the schedule;
- what I still need to do (upload my CV PDF to Drive and paste its link into the Documents tab; connect Indeed and Gmail if I want them);
- one tip for my search.
```

## How it lines up with the dashboard

- The tabs and columns match the [sheet schema v2](jobs-v2-multi-user.md#sheet-schema-v2).
- `Role seen = "No role seen yet"` is what the dashboard's **No roles** filter looks for.
- `[industry paragraph]` is filled in the composer from the Paragraph row whose Industry matches.
- The skills follow the dashboard's write rules: re-read the tab and find rows by company name before every write.
