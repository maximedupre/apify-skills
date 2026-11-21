---
name: amazon-keywords-discovery-tool
description: "Expands seed terms into Amazon autocomplete keyword suggestions (marketplace, locale, query variant, source position, thumbnail) by running the Amazon Keywords Discovery Tool Apify Actor. Use when the user wants Amazon keyword research, listing SEO or PPC keyword ideas, long-tail Amazon search phrases, buyer-intent or question keywords, or to compare suggestions across Amazon marketplaces or departments."
metadata:
  actor: "maximedupre/amazon-keywords-discovery-tool"
---

# Amazon Keywords Discovery Tool

Runs the Apify Actor `maximedupre~amazon-keywords-discovery-tool` with curl. Input is seed keywords plus a marketplace and a discovery depth. Output is one dataset row per unique Amazon autocomplete suggestion. Store page: https://apify.com/maximedupre/amazon-keywords-discovery-tool

Requires the `APIFY_TOKEN` environment variable. Every input field is listed at https://apify.com/maximedupre/amazon-keywords-discovery-tool/input-schema. Cost questions: https://apify.com/maximedupre/amazon-keywords-discovery-tool/pricing.

## Run the Actor

1. Start the run with the use case's input. Save `data.id` and `data.defaultDatasetId` from the response.
   ```bash
   curl -s -X POST "https://api.apify.com/v2/acts/maximedupre~amazon-keywords-discovery-tool/runs" \
     -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
     -d '{"keywords":["laptop stand"],"expansionMode":"alphabet","maxTotalResults":50}'
   ```
2. Poll until `data.status` is `SUCCEEDED`, `FAILED`, `ABORTED`, or `TIMED-OUT`. Wait 10 seconds between polls.
   ```bash
   curl -s "https://api.apify.com/v2/actor-runs/<runId>" -H "Authorization: Bearer $APIFY_TOKEN"
   ```
3. On `SUCCEEDED`, fetch the rows.
   ```bash
   curl -s "https://api.apify.com/v2/datasets/<defaultDatasetId>/items?format=json" -H "Authorization: Bearer $APIFY_TOKEN"
   ```
4. Done when the status is terminal and the dataset is fetched. On any other status, see Gotchas.

Row fields: `suggestion`, `seedKeyword`, `marketplace`, `locale`, `query`, `expansionMode`, `expansionToken` (absent for seed-only rows), `sourcePosition` (1 = Amazon's top suggestion), `thumbnailImageUrl` (often absent).

## Use case 1: Listing SEO keyword list for a product

Goal: find the phrases shoppers type for one product.

1. Take 1-5 seeds that name the product (for example `laptop stand`).
2. Run with `expansionMode` `full` and `maxTotalResults` 500. `full` covers alphabet, numbers, and buyer modifiers.
3. Sort rows by `sourcePosition` ascending, then deduplicate on `suggestion`. Low positions are the phrases Amazon ranks as most popular.
4. Report the top phrases per seed with the query that produced each.

```json
{"keywords":["laptop stand"],"expansionMode":"full","maxTotalResults":500}
```

## Use case 2: Long-tail PPC keyword ideas

Goal: specific, buyer-intent phrases for sponsored ad campaigns.

1. Use `buyerModifiers` for intent words, or `custom` with your own suffixes (such as `best`, `for kids`, `under 50`) in `customExpansions`.
2. Set `minWords` to 3 to drop head terms. Use `excludeTerms` for brands or words you do not sell.
3. Add `recursiveLevels` 1 to treat each found suggestion as a new seed. This finds deeper phrases but multiplies the run size, so keep `maxTotalResults` set.
4. Group the rows by `expansionToken` to see which modifier produced each phrase.

```json
{"keywords":["yoga mat"],"expansionMode":"custom","customExpansions":["best","for kids","under 50"],"minWords":3,"maxTotalResults":300}
```

## Use case 3: Customer-question and comparison research

Goal: the questions and comparisons shoppers ask, for listing bullets and A+ content.

1. Run with `expansionMode` `questions` (adds what, why, how, which, can, plus comparison and preposition suffixes).
2. Optionally set `includeTerms` (for example `vs`, `for`, `with`) to keep only comparison-style phrases.
3. Summarize the recurring themes from `suggestion`.

```json
{"keywords":["air fryer"],"expansionMode":"questions","maxTotalResults":200}
```

## Use case 4: Marketplace or department comparison

Goal: see how demand wording differs by country or category.

1. Run once per marketplace (`amazon.com`, `amazon.co.uk`, `amazon.de`, ...) with the same seeds and `expansionMode`. For non-English marketplaces, write the seeds in that language, since Amazon returns local-language suggestions.
2. To narrow by category, set `department` (such as `pets`, `kitchen`, `electronics`). Leave empty for all departments.
3. Compare the runs by `marketplace` and `locale`; list phrases present in one marketplace and absent in another.

```json
{"keywords":["hundebett"],"marketplace":"amazon.de","department":"pets","expansionMode":"alphabet","maxTotalResults":100}
```

## Gotchas

- `maxTotalResults` 0 or empty means unlimited, and `full` with `recursiveLevels` above 0 can produce thousands of rows, and the Actor charges per saved suggestion. Set a cap on the first run.
- `maxSuggestionsPerKeyword` limits each seed; `maxTotalResults` limits the whole run. Use the per-seed limit when comparing many seeds so one seed does not use the whole budget.
- `customExpansions` is ignored unless `expansionMode` is `custom`.
- `seedOnly` returns only the seed's own suggestions: few rows, not a broad expansion.
- A run that ends `SUCCEEDED` with an empty dataset means Amazon returned no suggestions for those seeds, filters, or department. Loosen `includeTerms`, `minWords`, or `department`, or try a more common seed.
- On `FAILED`, `ABORTED`, or `TIMED-OUT`, fetch the run log with `GET https://api.apify.com/v2/actor-runs/<runId>/log` for the cause. Partial rows may still be in the dataset; fetch them before retrying.
- Input errors (such as an empty `keywords` array) fail the run immediately. Fix the field named in the log and start a new run.
- Suggestion data is Amazon's public autocomplete; it has no search volume. Do not present `sourcePosition` as volume, only as relative ranking.
- API reference for run options: https://apify.com/maximedupre/amazon-keywords-discovery-tool/api
