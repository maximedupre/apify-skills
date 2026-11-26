---
name: twitter-scraper
description: "Collects public X posts, profiles, network members, and trends in an Apify dataset. Use when searching Twitter/X posts by topic or author, fetching known posts, conversations, or public source URLs, finding public accounts by name or handle, listing followers or retweeters, or comparing trending topics by region or city."
---

# Twitter (X) Scraper

Choose one workflow below and save its JSON input as `input.json`. Set `maxNbItemsToScrape` to the maximum number of rows needed across the run. Omitting it or using `0` collects available rows until the source or charge limit is reached. Use the live Pricing tab to estimate cost.

[Actor Store](https://apify.com/maximedupre/twitter-scraper?fpr=qn4m5w) · [Input schema](https://apify.com/maximedupre/twitter-scraper/input-schema?fpr=qn4m5w) · [Pricing](https://apify.com/maximedupre/twitter-scraper/pricing?fpr=qn4m5w) · [API](https://apify.com/maximedupre/twitter-scraper/api?fpr=qn4m5w)

## Run the Actor

Run the following in a shell with `curl`, `jq`, and `APIFY_TOKEN` set in the environment. The Actor collects public data without an X login. The commands start a run, print its ID, wait for a terminal status, and save the dataset as `results.json`.

```sh
(
  set -eu
  : "${APIFY_TOKEN:?Set APIFY_TOKEN in the environment}"
  api=https://api.apify.com/v2
  run=$(curl -fsS -X POST \
    -H "Authorization: Bearer $APIFY_TOKEN" \
    -H 'Content-Type: application/json' --data-binary @input.json \
    "$api/acts/maximedupre~twitter-scraper/runs")
  run_id=$(printf '%s\n' "$run" | jq -er '.data.id | select(length > 0)')
  printf 'Run ID %s\n' "$run_id" >&2

  while :; do
    run=$(curl -fsS -H "Authorization: Bearer $APIFY_TOKEN" "$api/actor-runs/$run_id")
    status=$(printf '%s\n' "$run" | jq -er '.data.status | select(length > 0)')
    case "$status" in
      SUCCEEDED|FAILED|ABORTED|TIMED-OUT) break ;;
    esac
    sleep 5
  done

  printf 'Run %s ended with status %s\n' "$run_id" "$status" >&2
  dataset_id=$(printf '%s\n' "$run" | jq -er '.data.defaultDatasetId | select(length > 0)')
  curl -fsS -H "Authorization: Bearer $APIFY_TOKEN" \
    "$api/datasets/$dataset_id/items?format=json" -o results.json
  test "$status" = SUCCEEDED
)
```

If polling is interrupted, resume the GET requests with the same run ID to avoid starting another paid run. If the initial POST loses its response, check the Actor's runs before submitting it again.

## Monitor public posts about a topic

1. Replace `OpenAI` with the requested topic or X search query. Set the row limit for the sample.

   ```json
   {"target":"searchPosts","postDiscoveryMethod":"searchQuery","searchQuery":["OpenAI"],"searchMode":"latest","maxNbItemsToScrape":20}
   ```

2. Read [post search filters](references/posts.md) if you need date bounds, media filters, or engagement filters. Run the Actor with the chosen input.
3. Return matching posts with `postText`, `postUrl`, and `postDateTime`. Use `postId` to identify the same post across runs.

## Fetch exact public posts

1. Replace the example URL in `directPostUrls` with the public post URLs or numeric post IDs as strings. Exact lookup ignores search filters.

   ```json
   {"target":"searchPosts","postDiscoveryMethod":"directPosts","directPostUrls":["https://x.com/NASA/status/2110798512795377741"],"maxNbItemsToScrape":1}
   ```

2. Set the row limit to at least the number of distinct requested posts and run the Actor. Match `postId` to the requested IDs, including the digits after `/status/` in each URL. Return the matching post details and list any requested IDs absent from the dataset.

## Find public posts in a conversation

1. Open the original post that started the conversation. Replace the example ID with the digits after `/status/` in that post's URL. Use the [post search filters](references/posts.md) to choose `startDate`.

   ```json
   {"target":"searchPosts","postDiscoveryMethod":"conversationIds","conversationIds":["2110798512795377741"],"startDate":"2026-10-15","searchMode":"latest","maxNbItemsToScrape":20}
   ```

2. Run the Actor. Return the matching public posts with their URLs and timestamps. If you supplied multiple conversation IDs, use `sourceInput` to identify which input first found each post. Report the row cap when presenting a sample.

## Collect posts from a public X source URL

1. Replace the example with the public profile, list, search, or conversation URL. For an X search URL, include `q`, omit `pf` and `lf`, and leave `f` blank or use `live` or `top`. Set result order with `searchMode`.

   ```json
   {"target":"searchPosts","postDiscoveryMethod":"sourceUrls","postSearchUrls":["https://x.com/NASA"],"searchMode":"latest","maxNbItemsToScrape":20}
   ```

2. If you need older posts from a source URL, including a conversation URL, use the [post search filters](references/posts.md) to choose `startDate`.
3. Run the Actor and return the matching post URLs, text, and timestamps. If you supplied multiple URLs, include `sourceInput` to identify the first source that found each post.

## Find public X profiles

1. For an exact account, replace `NASA` with the known handle, profile URL, or numeric user ID. Enter handles without `@`.

   ```json
   {"target":"searchProfiles","profileDiscoveryMethod":"directTargets","profileLookupTargets":["NASA"],"maxNbItemsToScrape":1}
   ```

2. For a name or topic search, use this input instead. Each search phrase can use up to 500 UTF-8 bytes. X treats a lone `@handle` or profile URL in a phrase as a handle search, so use the exact lookup input when identity matters.

   ```json
   {"target":"searchProfiles","profileDiscoveryMethod":"searchPhrases","profileSearchQueries":["space telescope"],"maxNbItemsToScrape":10}
   ```

3. Run the Actor. Return the matching `handle` and `displayName` with the requested public details. If a profile link is needed, build it from `https://x.com/` and the returned `handle`. For phrase searches, use `sourceInput` to identify the first query that found each profile.

## Sample a public account's network

1. Replace `NASA` with the account to sample. Use `followers` for accounts that follow it or `following` for accounts it follows. For a public post's retweeters, use `retweeters` and supply its URL or ID in `retweeterPostUrlsOrIds` instead of `profileLookupTargets`.

   ```json
   {"target":"searchProfiles","profileDiscoveryMethod":"followers","profileLookupTargets":["NASA"],"maxNbItemsToScrape":20}
   ```

2. Run the Actor. Report the returned handles with `networkType` and `sourceInput` so each relationship is clear. Describe capped results as a sample of the network X exposes.

## Compare trending topics by location

1. Use `trendRegionId` of `"1"` for Worldwide or choose a country ID from the input schema. For city trends, use `trendDiscoveryMethod: "city"` and supply `trendCities` with exact supported city and country pairs such as `{"city":"Tokyo","country":"Japan"}`.

   ```json
   {"target":"trendingTopics","trendDiscoveryMethod":"region","trendRegionId":"1","trendListScope":"newest","maxNbItemsToScrape":20}
   ```

2. For a region's recent history, use `trendListScope: "recent"` to request up to 24 unevenly spaced lists. The Actor returns the newest list first. Raise or omit `maxNbItemsToScrape` so the sample cap leaves room for older lists. City mode returns current trends. Run the Actor once per region or pass multiple cities in one city run. A batch can reach the shared row limit before the Actor processes later cities.
3. Identify region rows by `regionId` and label them with `regionName`. Identify city rows by the pair `cityName` and `regionCountry`. Check that every requested location appears before comparing. If a city is missing from a batch, run that city by itself. If any requested location still has no rows, identify it and report the comparison as incomplete.
4. Within each location, group rows by `trendListDateTime` when present. For recent history, report the distinct list timestamps returned for each location before comparing lists. If fewer than two dated lists appear or a requested period is missing, report the gap and limit the comparison to the lists available. Use `trendName` and `trendRank` to compare topics. Topic names can recur within and across lists.

## Gotchas

- If a run fails, times out, or is aborted, report its status and run ID. Label any saved rows as partial. If a run succeeds with no rows, inspect its warnings before reporting why the dataset is empty.
- Apify form prefills apply in the form. Send `target` and the matching source input explicitly in API requests.
- Public results depend on what X exposes. Preserve null fields as unknown and report coverage limits when the run warns that it could not finish a source.
