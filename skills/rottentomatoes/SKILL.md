---
name: rottentomatoes
description: "Collects Rotten Tomatoes movie, TV series, and season data (Tomatometer and audience scores, cast, credits) and critic or audience reviews as structured rows through the Apify Actor maximedupre~rottentomatoes. Use when an agent needs Rotten Tomatoes scores, title details, or review text for a list of titles or URLs, for example to compare critic and audience sentiment."
metadata:
  actor: "maximedupre~rottentomatoes"
---

# Rotten Tomatoes Scraper

Runs the Apify Actor `maximedupre~rottentomatoes` with plain `curl`. It needs only the `APIFY_TOKEN` environment variable. Store page: https://apify.com/maximedupre/rottentomatoes

The Actor returns two kinds of rows, chosen by `resultType`:
- `titles`: one row per movie, TV series, or season, with scores, cast, credits, and release facts.
- `reviews`: one row per critic or audience review of a movie or TV season.

## Run the Actor

Every use case below uses these three steps. Send the token as a header, never in the URL.

1. Start the run. The response holds `data.id` (run id) and `data.defaultDatasetId`.
   ```bash
   curl -s -X POST "https://api.apify.com/v2/acts/maximedupre~rottentomatoes/runs" \
     -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
     -d '{"resultType":"titles","keywords":["the matrix"],"maxItems":5}'
   ```
2. Poll until `data.status` is `SUCCEEDED`, `FAILED`, `ABORTED`, or `TIMED-OUT`. Wait 10 seconds between polls for titles and 30 seconds for reviews.
   ```bash
   curl -s "https://api.apify.com/v2/actor-runs/<runId>" -H "Authorization: Bearer $APIFY_TOKEN"
   ```
3. Fetch the rows.
   ```bash
   curl -s "https://api.apify.com/v2/datasets/<defaultDatasetId>/items?format=json" \
     -H "Authorization: Bearer $APIFY_TOKEN"
   ```

All input fields: https://apify.com/maximedupre/rottentomatoes/input-schema. Costs: https://apify.com/maximedupre/rottentomatoes/pricing. API details: https://apify.com/maximedupre/rottentomatoes/api.

## Use case 1: Look up scores and details for known titles

Use when the user has Rotten Tomatoes URLs and wants exact pages. URLs are exact; keyword search is not (see use case 2).

1. Build `startUrls` as objects: `[{"url":"https://www.rottentomatoes.com/m/the_matrix"}]`.
2. Set `resultType` to `titles` and `maxItems` to at least the number of URLs.
3. Run the Actor (steps above).
4. Read `criticRating.score` (Tomatometer), `audienceRating.score`, `cast`, `directors`, `releaseDates`, and `boxOffice`. Optional fields are absent when the source lacks them, so test for presence.

```json
{"resultType":"titles","startUrls":[{"url":"https://www.rottentomatoes.com/m/the_matrix"},{"url":"https://www.rottentomatoes.com/tv/breaking_bad"}],"maxItems":10}
```

## Use case 2: Find titles by keyword

Use when the user has names but no URLs.

1. Set `keywords` to one search term per entry and `resultType` to `titles`.
2. Set `mediaType` to `movies`, `tv`, or `both` to narrow the search.
3. Set `includeSearchAlternatives` to `false` to keep only the first match per keyword. Leave it `true` to collect every match.
4. After the run, compare each row's `title` and `releaseYear` with what the user asked for, because search does not guarantee an exact match. Switch to the row's `url` for later runs.

```json
{"resultType":"titles","keywords":["dune","the bear"],"mediaType":"both","includeSearchAlternatives":false,"maxItems":20}
```

## Use case 3: Compare a series score with its seasons

Use to see how a show's quality changes across seasons.

1. Run use case 1 with the series URL. The row's `seasons` array lists each season `name` and `url`.
2. Run the Actor again with those season URLs in `startUrls` and `resultType` `titles`.
3. Compare `criticRating.score` and `audienceRating.score` of the series row with each season row. Series and season scores come from their own pages.

```json
{"resultType":"titles","startUrls":[{"url":"https://www.rottentomatoes.com/tv/breaking_bad/s01"},{"url":"https://www.rottentomatoes.com/tv/breaking_bad/s02"}],"maxItems":10}
```

## Use case 4: Collect audience or critic reviews

Use to study viewer or critic feedback, run sentiment analysis, or pull quotes.

1. Choose URLs of movies or TV seasons. Reviews are for movies and TV seasons; for a series, use its season URLs from use case 3.
2. Set `resultType` to `reviews`, and `reviewType` to `critic`, `audience`, or `both`.
3. Optional filters: `topCriticsOnly` for critic reviews, `verifiedAudienceOnly` for audience reviews.
4. Set `maxItems` to cap the total rows across all URLs. Reviews are charged per saved row, so cap it for a first trial.
5. Run the Actor.
6. Read `reviewText`, `rating` (audience), `originalScore` (critic), `sentiment`, `createdAt`, and `isVerified` or `isTopCritic`. Field list and row examples: read [references/review-rows.md](references/review-rows.md) when you need exact review fields.

```json
{"resultType":"reviews","startUrls":[{"url":"https://www.rottentomatoes.com/m/top_gun_maverick"}],"reviewType":"audience","verifiedAudienceOnly":true,"maxItems":100}
```

## Gotchas

- `startUrls` entries use the documented form of objects with a `url` key; plain URL strings also work.
- Options for the other result type are ignored: `keywords` and `mediaType` do nothing for `reviews`, and review filters do nothing for `titles`.
- `maxItems` of `0` or an empty value means all available results, which can be thousands of review rows. Set a cap unless the user asked for everything.
- A `SUCCEEDED` run can end with a log warning such as `Could not collect <url>` and fewer rows than `maxItems`. The saved rows are kept. Compare the row count with `maxItems` and the warning before you treat the collection as complete. To retry that URL, run it again in a separate run.
- Each source item is saved once per run. A title matched by two keywords gives one row.
- When `data.status` is `FAILED`, `ABORTED`, or `TIMED-OUT`, the dataset can still hold rows saved before the stop. Fetch it before you decide the run produced nothing.
