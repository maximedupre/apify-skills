# Output fields

Read this when a use case needs the meaning of a row field. A row is one job. Absent fields are not published.

| Field | Meaning |
| --- | --- |
| `id`, `url`, `title` | Job ID, public job link, job title. `id` identifies the job across runs. |
| `employer.id`, `name`, `url`, `logoUrl`, `industry`, `description`, `websiteUrl` | Employer profile data. Only `name` is always present. |
| `employer.emails`, `phones`, `socialUrls`, `paymentTechnologies` | Filled only when the matching `include...` input is `true`. Can be empty arrays. |
| `roleTypes` | Handshake role categories. Can be empty. |
| `employmentTypes` | Published classes such as `full-time`, `internship`, `co-op`, `job`. |
| `locations` | Work locations as full labels, for example `Honolulu, Hawaii, United States`. |
| `workplace` | Array of `remote`, `hybrid`, `on-site`. A job can list several. |
| `compensation.minAmount`, `maxAmount` | Lower and upper pay. With one published amount, only `minAmount`. |
| `compensation.currency`, `cadence`, `schedule` | ISO currency code, pay period (`hour`, `day`, `week`, `month`, `year`), and the published schedule label. |
| `benefitTypes` | Benefit and extra-pay labels, with no amounts or eligibility. |
| `postedAt`, `firstActiveAt`, `expiresAt` | ISO date-times: posting, first active, application expiry. |
| `descriptionHtml`, `descriptionText` | Full description as HTML and as plain text. |

Example row (descriptions shortened):

```json
{"id":"11529531","url":"https://app.joinhandshake.com/public/jobs/11529531","title":"Human Resources Assistant IV","employer":{"id":"239516","name":"Hawaii State Department of Education"},"employmentTypes":["full-time"],"locations":["Honolulu, Hawaii, United States"],"workplace":["on-site"],"compensation":{"minAmount":3606,"maxAmount":3900,"currency":"USD","cadence":"month"},"postedAt":"2026-09-29T02:37:06.000Z","descriptionText":"..."}
```
