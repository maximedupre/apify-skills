---
name: handshake-jobs-scraper
description: "Collects public Handshake job listings (title, employer, pay, locations, workplace, dates, full descriptions) by keyword, location, search URL, job URL, or job ID by running the Handshake Jobs Scraper Apify Actor. Use when the user wants Handshake internships or entry-level jobs, remote Handshake jobs, Handshake salary data, hiring research on named employers, or a lookup of specific Handshake job IDs."
metadata:
  actor: "maximedupre/handshake-jobs-scraper"
---

# Handshake Jobs Scraper

Runs the Apify Actor `maximedupre~handshake-jobs-scraper` with curl. Input is keywords, locations, Handshake links, or job IDs plus filters. Output is one dataset row per unique public job. No Handshake login is needed. Store page: https://apify.com/maximedupre/handshake-jobs-scraper

Requires the `APIFY_TOKEN` environment variable. All use cases share the run steps below.

## Run the Actor

1. Start the run with the use case's input. Save `data.id` and `data.defaultDatasetId` from the response.
   ```bash
   curl -s -X POST "https://api.apify.com/v2/acts/maximedupre~handshake-jobs-scraper/runs" \
     -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
     -d '{"keywords":["data analyst"],"maxDiscoveryItems":50}'
   ```
2. Poll until `data.status` is `SUCCEEDED`, `FAILED`, `ABORTED`, or `TIMED-OUT`. Wait 10 seconds between polls.
   ```bash
   curl -s "https://api.apify.com/v2/actor-runs/<runId>" -H "Authorization: Bearer $APIFY_TOKEN"
   ```
3. On `SUCCEEDED`, fetch the rows.
   ```bash
   curl -s "https://api.apify.com/v2/datasets/<defaultDatasetId>/items?format=json" -H "Authorization: Bearer $APIFY_TOKEN"
   ```
4. Done when the status is terminal and the dataset is fetched. On any other terminal status, follow the failed-run entry in Gotchas.

Every input field: https://apify.com/maximedupre/handshake-jobs-scraper/input-schema. Always set `maxDiscoveryItems`; unset or `0` means every matching job, which can be a very long, costly run.

## Use case 1: Search jobs by keyword, location, or role

For job lists, internship and co-op searches, or remote-job lists.

1. Turn the request into input: `keywords` (any term can match), `locations` (full labels such as `Austin, Texas, United States`), `roleCategories` (exact Handshake names), and `maxDiscoveryItems`.
2. Add filters the user asked for: `employmentTypes` (`internship`, `full-time`, `part-time`, `co-op`), `workplace` (`remote`, `hybrid`, `on-site`), `postedWithinDays`, `countries`, `workTypes`.
   ```json
   {"keywords":["marketing"],"employmentTypes":["internship"],"workplace":"remote","postedWithinDays":30,"maxDiscoveryItems":100}
   ```
3. Run the Actor (steps above).
4. Present the rows as the user asks, such as a table of `title`, `employer.name`, `locations`, `workplace`, `postedAt`, `url`. For field meanings, read [references/output-fields.md](references/output-fields.md).
5. If the result is small or empty, loosen the filters before concluding there are no jobs (see Gotchas).

## Use case 2: Salary and pay analysis

For comparing published pay across roles, employers, or places.

1. Run a search as in use case 1. Add `compensationFilter` for a minimum, for example `{"amount":25,"currency":"USD","cadence":"hour"}`, or `payPeriods` to keep only one pay period.
2. Read `compensation.minAmount`, `maxAmount`, `currency`, and `cadence` from each row. Many jobs publish no pay, so report how many rows have pay and how many do not.
3. Group by `cadence` and `currency` before computing ranges or averages. The Actor does not convert between them.

## Use case 3: Employer hiring research

For seeing what named employers hire for, and optionally their public contact data.

1. Put the employer names in `employers` and set `maxDiscoveryItems`. Add `keywords` or `locations` to narrow, because Handshake search starts from those, not from the employer name.
2. For contact data, set `includeEmployerEmails`, `includeEmployerPhones`, `includeEmployerSocialProfiles`, or `includeEmployerPaymentTechnologies` to `true`. Each adds a charge per saved value, so enable only what the user needs and keep the matching `maxEmployer...` limit low. Default memory rises to 512 MB when any is on.
3. Run the Actor, then group rows by `employer.name`. Count jobs, list `roleTypes` and `locations`, and include the employer fields requested.

## Use case 4: Specific jobs from links or IDs

For refreshing known jobs, or collecting jobs from a Handshake search page the user already has.

1. Put job IDs in `jobIds` (one per item) and Handshake links in `startUrls` as `{"url":"..."}` objects. Search links `https://joinhandshake.com/find-jobs/<city>/` and job links `https://app.joinhandshake.com/public/jobs/<id>` both work.
   ```json
   {"jobIds":["11529531"],"startUrls":[{"url":"https://joinhandshake.com/find-jobs/san-antonio-tx/"}],"maxDiscoveryItems":20}
   ```
2. Run the Actor. Duplicates across all sources are returned once.
3. Match each requested ID against the rows' `id` and tell the user which ones are missing. Keyword, location, and job filters also apply to links and IDs, so leave them empty when the user wants every listed job.

## Gotchas

- Run log: `curl -s "https://api.apify.com/v2/actor-runs/<runId>/log" -H "Authorization: Bearer $APIFY_TOKEN"`. Lines such as `Could not read <link>. Processing continues.` name a link or value that gave no jobs.
- A failed run: a run can end `FAILED` with 0 rows and the message `Could not finish reading Handshake jobs. Please try again.`, and the log gives no reason. Retry once. If it fails again, report the run's `statusMessage` to the user and suggest the Actor issues tab: https://apify.com/maximedupre/handshake-jobs-scraper/issues
- A filter excludes any job that lacks the value it checks (for example `postedWithinDays` drops jobs with no posting date, and `compensationFilter` drops jobs without pay). Many filters together can give 0 rows.
- A public search page can expose fewer jobs than its displayed total. A search link can return only what the page makes public.
- Fields Handshake does not publish are absent from the row, not `null`. Check a field exists before using it.
- Cost: a small per-run start fee plus a price per saved job, plus extra per saved employer value when those options are on. For current prices, read https://apify.com/maximedupre/handshake-jobs-scraper/pricing before large runs.
- API details beyond this skill: https://apify.com/maximedupre/handshake-jobs-scraper/api
