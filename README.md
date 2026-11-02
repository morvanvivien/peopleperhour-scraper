# How to Scrape PeoplePerHour: Freelance Jobs (Budget, Skills, Experience Level)

> **Run it now, no code: [peopleperhour-scraper on the Apify Store](https://apify.com/bibian/peopleperhour-scraper)** &nbsp;|&nbsp; $1.20 per 1,000 jobs &nbsp;|&nbsp; no browser, no proxy

PeoplePerHour is a UK-based freelance marketplace used worldwide, with open jobs posted by clients across design, development, writing, marketing and more. It has no public export and no official API for third parties. This guide shows how to get those jobs as clean structured data: budget, skills, experience level and full description, with a paid Apify Actor, from the console or from code.

Searches this page answers: peopleperhour scraper, peopleperhour api, freelance jobs data, freelance marketplace scraper.

## What you get from the PeoplePerHour scraper

- Every open job on PeoplePerHour, newest first, with title, category, subcategory, skills, experience level and exact posting/expiry dates.
- Budget parsed from PeoplePerHour's own structured job data into `salaryMin`, `salaryMax`, `salaryCurrency` and `salaryPeriod` (the client's own currency and figure, not the site's approximate GBP conversion), plus PeoplePerHour's own `"fixed"`/`"hourly"` label.
- The same unified field names (`datePosted`, `salaryMin`, `employmentType`, `skills`...) shared by every job and freelance-mission Actor from this author, so PeoplePerHour drops straight into a pipeline already built for one of the others.
- Optional full job description as plain text, with any e-mail address or phone number removed.
- No client name, ever: PeoplePerHour always truncates a client's display name to a first name and a last-name initial, for an individual and a registered business alike, with no reliable way to tell them apart — so no client name is read into the dataset at all.

Export the dataset as JSON, CSV or Excel, or read it through the Apify API. A real sample (five results, no personal data, long descriptions truncated) is in [`sample-output.json`](sample-output.json).

## Output fields

| Field | What it is |
|---|---|
| `id`, `url`, `title` | `peopleperhour.com:<job id>`, the public job page, and its title. |
| `companyName`, `companyUrl`, `confidentialCompany` | Always `null` / `true`. See "Good to know" below. |
| `locationLabel`, `city`, `region`, `postalCode`, `country` | The country the client restricted applicants to, when the job has that restriction (most jobs are open to "Any" country, so these stay `null`). PeoplePerHour never publishes a city or region for a job. |
| `remote` | `"remote"` when PeoplePerHour's own data marks the job as telecommute (the vast majority), `"unknown"` otherwise. |
| `employmentType` | Always `"freelance"`. |
| `salaryMin`, `salaryMax`, `salaryCurrency`, `salaryPeriod` | The client's own budget figure and currency. PeoplePerHour shows a single figure, never a range, so `salaryMin` and `salaryMax` are equal. `salaryPeriod` is `"hour"` for hourly jobs, `null` for a fixed budget. |
| `datePosted`, `validThrough` | The job's own posting and expiry timestamps. |
| `experienceLevel` | The level the client selected ("Entry", "Intermediate", "Expert"), when set. |
| `industry`, `subcategory` | PeoplePerHour's own category and subcategory, read from the job's URL (for example `"Design"` / `"Web Design"`). |
| `skills` | Skills listed by the client, as PeoplePerHour's own tags. |
| `source`, `sourceId`, `scrapedAt` | `"peopleperhour.com"`, the job's numeric id, and when this run read it. |
| `budgetType` | `"fixed"` or `"hourly"`, PeoplePerHour's own label for the budget above. |
| `description` | Full job description as plain text (only when **Include full descriptions** is on). Any e-mail address or phone number is removed before saving. |

## Use cases

- **Find freelance work.** Filter by keyword or category and check new jobs every morning.
- **Monitor demand for a skill.** How many jobs mention "wordpress" or "logo design" this week, and at what budget.
- **Feed a lead list.** A client posting a job for a skill you sell is a lead; filter by category and experience level.
- **Study the freelance market.** Budgets, in-demand skills and experience levels requested over time.

## Input

| Field | What it does |
|---|---|
| `keywords` | Job title or skill, using PeoplePerHour's own search, for example `wordpress`. Empty means every open job. |
| `category` | Optional, PeoplePerHour's own category slug (for example `technology-programming`, `design`, `writing-translation`), copied from a filtered `peopleperhour.com/freelance-jobs/` URL. |
| `publishedWithinDays` | Keep only jobs posted in the last N days, computed from the job's own exact posting date. Use `1` for a daily monitoring run. |
| `maxItems` | The run stops once this many jobs are saved (default 50). |
| `onlyNew` | For scheduled runs: skip the jobs an earlier run of this same search already delivered. |
| `includeDescriptions` | Adds the full job description as plain text, with any e-mail address or phone number removed. |

Example input:

```json
{
  "keywords": "wordpress",
  "publishedWithinDays": 7,
  "maxItems": 50,
  "onlyNew": false,
  "includeDescriptions": false
}
```

## Four ways to use it

### 1. Apify console, no code

1. Open [the Actor page](https://apify.com/bibian/peopleperhour-scraper) and click **Try for free** (a free Apify account is enough).
2. Fill in the input form (keywords, filters and the maximum number of results) and click **Start**.
3. When the run finishes, open the **Output** tab, check the table, then **Export** as CSV, Excel or JSON.
4. For a daily alert, open **Schedules**, create a daily schedule for the Actor and, optionally, add an integration that sends new results to Slack, e-mail or Google Sheets.

### 2. Apify API with JavaScript

```bash
npm install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
node examples/run.js
```

```js
import { ApifyClient } from 'apify-client';

const client = new ApifyClient({ token: process.env.APIFY_TOKEN });

const run = await client.actor('bibian/peopleperhour-scraper').call({
    "keywords": "wordpress",
    "publishedWithinDays": 7,
    "maxItems": 50,
    "onlyNew": false,
    "includeDescriptions": false
});

const { items } = await client.dataset(run.defaultDatasetId).listItems();
for (const item of items) {
    console.log(`${item.title} | ${item.salaryMin ?? '-'} ${item.salaryCurrency ?? ''} | ${item.industry ?? '-'}`);
}
```

The complete file is [`examples/run.js`](examples/run.js) (ES module, Node 18 or later).

### 3. Apify API with Python

```bash
pip install apify-client
export APIFY_TOKEN="your token from Apify console, Settings, Integrations"
python examples/run.py
```

```python
import os
from apify_client import ApifyClient

client = ApifyClient(os.environ["APIFY_TOKEN"])

run = client.actor("bibian/peopleperhour-scraper").call(run_input={
    "keywords": "wordpress",
    "publishedWithinDays": 7,
    "maxItems": 50,
    "onlyNew": False,
    "includeDescriptions": False,
})

for item in client.dataset(run["defaultDatasetId"]).iterate_items():
    print(f"{item['title']} | {item.get('salaryMin')} {item.get('salaryCurrency') or ''} | {item.get('industry')}")
```

The complete file is [`examples/run.py`](examples/run.py). Never put your token in the code: read it from the `APIFY_TOKEN` environment variable as above.

### 4. Schedule it and connect your tools

1. **Schedule it.** Save a task with your keywords and schedule it daily with Apify Schedules. Tick **Only new results** (`onlyNew: true`) and each run returns, and charges, only the jobs earlier runs of the same search did not deliver.
2. **Push the results where you work.** Add an Apify integration or a webhook to the task: Google Sheets, Slack, Make, Zapier, n8n or your own endpoint, triggered at the end of every run.
3. **Let an AI agent use it.** Through the [Apify MCP server](https://docs.apify.com/platform/integrations/mcp), assistants such as Claude or ChatGPT can search the Store, read this page and run the Actor as a tool.

## Pricing

Pay per result: **$1.20 per 1,000 jobs** (that is $0.0012 per job), plus a small start fee. For example, a 50-job test run costs about $0.06. You pay only for the results saved to the dataset. Apify's free plan includes monthly credits, enough to try it on a few hundred results. Prices are those shown on the [Store page](https://apify.com/bibian/peopleperhour-scraper) when this guide was written; the Store page is the reference.

## Limits and honest notes

- No login, proxy or browser needed for a default run: the Actor reads PeoplePerHour's own public job pages directly, respecting `robots.txt` (pagination always requests `?page=` first, the only query form the site's own `robots.txt` allows).
- `datePosted` and `validThrough` come from the job page's own timestamp, which carries no timezone marker; treated as UTC, the same documented best-effort approximation used by this author's other date-only sources.
- No personal data is collected. PeoplePerHour always truncates a client's name to a first name and a last-name initial, whether the account behind it is an individual or a registered business, and the page gives no way to tell the two apart (its own structured data names the employer "PeoplePerHour" itself, never the real client) — so the client's name, avatar and profile link are never read into this dataset at all, for any job (visible in `sample-output.json`, where all five sample records have `companyName: null`). The list page also shows a second name and price next to each job card; that pair is a different freelancer PeoplePerHour suggests for the job, not its client, and is likewise never read.
- Any e-mail address or phone number a client typed into the description is removed before saving.
- A job with no explicit country restriction (the large majority) keeps `country` and `locationLabel` as `null`: PeoplePerHour does not require one, by the nature of a freelance marketplace.
- This is an unofficial tool, not affiliated with PeoplePerHour. Check the site terms and the data protection rules that apply to your use.

## FAQ

**Do I need an account or a proxy for PeoplePerHour?**
No. The Actor reads publicly visible jobs only.

**Why is `companyName` always empty?**
Because PeoplePerHour shows every client's name the same truncated way, whether they are an individual or a business, with no way to tell which is which. See "Limits and honest notes" above.

**Why is `country` almost always empty?**
Most PeoplePerHour jobs are open to applicants from any country; `country` is only filled when the client explicitly restricted the job to one.

**Which formats can I export?**
JSON, CSV, Excel, XML, HTML table or RSS, from the dataset page or through the API.

**Is it legal to scrape?**
The Actor collects publicly visible jobs and respects the robots.txt rules of the site. How you use the data is your responsibility: check the site's terms and the laws that apply to you, for example the GDPR.

**Something does not work?**
Open an issue in the Issues tab of the Actor's Apify Store page, or on this repository. Every public Actor is test-run daily and the broken ones are repaired.

## Try it

**[Open peopleperhour-scraper on the Apify Store](https://apify.com/bibian/peopleperhour-scraper)**

The code in `examples/` is released under the MIT license (see [LICENSE](LICENSE)). The data belongs to its original publishers.
