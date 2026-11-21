---
name: reddit-scraper
description: "Scrapes public Reddit posts, comments, subreddit info, and user profiles into structured rows with the Apify Reddit Scraper Actor. Use when the user needs Reddit search results, brand or topic mentions, subreddit feeds, comment threads from Reddit URLs, or a Reddit user's activity, with no Reddit login or API key."
---

# Reddit Scraper

Runs the Apify Actor `maximedupre~reddit-scraper` over plain curl. It needs only the `APIFY_TOKEN` environment variable. Store page: https://apify.com/maximedupre/reddit-scraper

The Actor reads public Reddit data only. It takes no Reddit login. Pick one `discoveryMethod` per run: `search`, `urls`, `subreddits`, or `users`. Fields of the other methods are ignored.

## Run protocol (all use cases)

1. Start the run. The response holds `data.id` and `data.defaultDatasetId`.
```bash
curl -s -X POST "https://api.apify.com/v2/acts/maximedupre~reddit-scraper/runs" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '<INPUT_JSON>'
```
2. Poll until `data.status` is `SUCCEEDED`, `FAILED`, `ABORTED`, or `TIMED-OUT`. Wait about 10 seconds between polls.
```bash
curl -s -H "Authorization: Bearer $APIFY_TOKEN" "https://api.apify.com/v2/actor-runs/<runId>"
```
3. Fetch rows.
```bash
curl -s -H "Authorization: Bearer $APIFY_TOKEN" "https://api.apify.com/v2/datasets/<defaultDatasetId>/items?format=json"
```
4. Tell the user the row count and the `rowType` mix. Rows with `rowType` of `post`, `comment`, `community`, or `user` have different fields. Read `rowType` before you read other fields.

Every field of the input is listed at https://apify.com/maximedupre/reddit-scraper/input-schema. Prices per saved row are at https://apify.com/maximedupre/reddit-scraper/pricing. API details: https://apify.com/maximedupre/reddit-scraper/api

## Use case 1: Track mentions of a topic or brand

Use when the user wants what Reddit says about a keyword.

1. Choose queries. Add `searchCommunityName` to limit to one subreddit.
2. Set `dateFrom` (for example `30 days`) for recent mentions only. Set `strictSearch` to `true` when the exact words must appear.
3. Set `contentType` to `posts`, `comments`, or `postsAndComments`.
4. Run the protocol. Group rows by `matchedInput` to see results per query.

```json
{"discoveryMethod":"search","searchQueries":["openai"],"sort":"new","dateFrom":"30 days","contentType":"posts","postRowLimit":50}
```

## Use case 2: Monitor a subreddit feed

Use when the user wants the latest or top posts of communities.

1. Set `subredditNames` (`r/name` or `name`).
2. Set `subredditSort` to `hot`, `new`, `top`, or `rising`. For `top`, set `time`.
3. Set `postRowLimit` per subreddit.
4. Run the protocol.

```json
{"discoveryMethod":"subreddits","subredditNames":["r/artificial"],"subredditSort":"top","time":"week","postRowLimit":25}
```

## Use case 3: Collect comment threads from known URLs

Use when the user gives Reddit post, comment, subreddit, user, or search URLs.

1. Put full URLs in `startUrls` as `{"url": "..."}` objects.
2. Set `commentRowLimit` for comment URLs. For posts found in listings, set `includeComments` to `true` and `commentsPerExpandedPost` to the depth needed.
3. Run the protocol. Each comment row has `parentPostId` and `depth` to rebuild the thread.

```json
{"discoveryMethod":"urls","startUrls":[{"url":"https://www.reddit.com/r/artificial/"}],"postRowLimit":10,"includeComments":true,"commentsPerExpandedPost":20}
```

## Use case 4: Profile a Reddit user's activity

Use when the user wants a person's profile and recent posts or comments.

1. Set `userNames` (`u/name` or `name`).
2. Set `contentType` to `posts`, `comments`, or `postsAndComments`.
3. Run the protocol. Profile rows hold karma and `createdAt`.

```json
{"discoveryMethod":"users","userNames":["spez"],"contentType":"postsAndComments","postRowLimit":20,"commentRowLimit":20}
```

## Gotchas

- Empty or `0` for `postRowLimit` or `commentRowLimit` means no limit. A forgotten limit makes a large, costly run. Always set both for a first run.
- `includeComments` adds comments under posts. `commentRowLimit` and `commentsPerExpandedPost` limit them differently, so set both.
- `includeNSFW` is `false` by default. NSFW results are dropped unless set to `true`.
- A run with status `SUCCEEDED` can have zero rows when inputs are skipped or not found. Check the run's `RUN-SUMMARY.json` in the key-value store (`GET https://api.apify.com/v2/key-value-stores/<defaultKeyValueStoreId>/records/RUN-SUMMARY.json`) for the reason.
- Charges apply per saved row only. Skipped, duplicate, and failed rows are free.
