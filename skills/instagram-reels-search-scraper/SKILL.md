---
name: instagram-reels-search-scraper
description: "Searches public Instagram Reels by keyword phrase or hashtag and returns structured rows with captions, engagement counts, author details, media links, and transcripts. Use when researching trending Reels for a topic or hashtag, finding high-performing Reels by likes or plays, collecting recent Reels, or filtering Reels by caption words."
metadata:
  actor: "maximedupre/instagram-reels-search-scraper"
---

# Instagram Reels Search Scraper

Runs the Apify Actor [maximedupre/instagram-reels-search-scraper](https://apify.com/maximedupre/instagram-reels-search-scraper?fpr=qn4m5w) with curl. It searches the public, currently ranked Reels surface, so results reflect today's ranking and are not a historical archive. It needs the `APIFY_TOKEN` environment variable.

Every input field is documented at https://apify.com/maximedupre/instagram-reels-search-scraper/input-schema?fpr=qn4m5w. Costs are at https://apify.com/maximedupre/instagram-reels-search-scraper/pricing?fpr=qn4m5w (one charge per Reel row plus a small start fee). API details: https://apify.com/maximedupre/instagram-reels-search-scraper/api?fpr=qn4m5w.

## Run protocol (all use cases)

1. Start the run. Write the JSON input body from the use case below:
   ```bash
   curl -s -X POST "https://api.apify.com/v2/acts/maximedupre~instagram-reels-search-scraper/runs" \
     -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
     -d '{"searchTerms":["travel"],"maxItems":20}'
   ```
   Read `data.id` and `data.defaultDatasetId` from the response.
2. Poll until `data.status` is terminal (`SUCCEEDED`, `FAILED`, `ABORTED`, `TIMED-OUT`), waiting 10 seconds between polls:
   ```bash
   curl -s -H "Authorization: Bearer $APIFY_TOKEN" "https://api.apify.com/v2/actor-runs/<runId>"
   ```
3. On `SUCCEEDED`, fetch rows:
   ```bash
   curl -s -H "Authorization: Bearer $APIFY_TOKEN" "https://api.apify.com/v2/datasets/<defaultDatasetId>/items?format=json"
   ```
   On any other terminal status, report the status and stop; do not report partial rows as complete.

Row fields: `reelId`, `permalink`, `searchTerm`, `caption`, `hashtags`, `mentions`, `likes`, `comments`, `plays`, `durationSeconds`, `publishedAt`, `author` (`username`, `followerCount`, `isVerified`), `media` (thumbnail and video links), `transcript` (language and timed segments). Optional fields appear only when the source provides them, so read each defensively.

## Use case 1: Research what Reels exist for a topic

Goal: sample current Reels for a keyword phrase to study captions, formats, and creators.

1. Choose keyword phrases (for example `"travel"`, `"budget travel tips"`).
2. Set `maxItems` to the sample size wanted (20 to 100 is typical).
3. Run the protocol with:
   ```json
   {"searchTerms": ["budget travel tips"], "maxItems": 50}
   ```
4. Summarize from `caption`, `hashtags`, `author.username`, `likes`, and `plays`. Group by `searchTerm` when you submitted several phrases.
5. For spoken content, read `transcript.language` and `transcript.segments`. Transcripts exist only when speech could be transcribed, so skip rows without `transcript`.

## Use case 2: Collect Reels for hashtags

Goal: gather Reels under one or more hashtags with their engagement.

1. Start every hashtag with `#`.
2. Run the protocol with:
   ```json
   {"searchTerms": ["#fitness", "#homeworkout"], "maxItems": 100}
   ```
3. Each Reel is saved once. If two hashtags lead to the same Reel, `searchTerm` shows only the first one, so do not count rows per hashtag as complete per-hashtag totals. Run hashtags separately when per-hashtag counts matter.

## Use case 3: Find high-performing or recent Reels

Goal: keep only Reels that already perform well or were published recently.

1. Set `minLikes` and/or `minPlays` to the performance bar.
2. Set `publishedAfter` to a UTC date (`2026-01-31`) or a relative time (`7 days`, `1 month`).
3. Run the protocol with:
   ```json
   {"searchTerms": ["skincare routine"], "maxItems": 30, "minPlays": 100000, "publishedAfter": "7 days"}
   ```
4. Sort by `plays` or `likes` descending for a ranked list. Filters apply to every search value in the run; use separate runs for different thresholds.

## Use case 4: Filter Reels by caption words

Goal: keep on-topic Reels and drop unwanted ones.

1. Put wanted words, phrases, or hashtags in `includeCaptionWords` (a Reel needs at least one) and unwanted ones in `excludeCaptionWords` (a Reel with any is dropped).
2. Run the protocol with:
   ```json
   {"searchTerms": ["coffee"], "maxItems": 40, "includeCaptionWords": ["espresso", "#latteart"], "excludeCaptionWords": ["giveaway", "sponsored"]}
   ```

## Gotchas

- `maxItems` empty or `0` means all available results, which can be large and costly. Set it on every run.
- A run takes only keyword phrases or only hashtags; split mixed lists into two runs.
- Media links in `media` can expire (see the `...ExpiresAt` fields). Download videos promptly if you need them; the Actor does not host videos.
- A short or empty dataset on a `SUCCEEDED` run means few Reels matched the filters on the current surface; loosen the filters or change the term.
