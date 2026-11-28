---
name: tiktok-live-search-scraper
description: "Finds public TikTok LIVE rooms and collects daily creator rankings. Use when a task calls for finding active streams by keyword or category, checking if a known creator or room is live, reviewing recent chat or top gifters, or comparing ranked creators by gift value."
---

# TikTok Live Search Scraper

Choose one workflow below and save its example as `input.json`. Adjust it using the [input schema](https://apify.com/maximedupre/tiktok-live-search-scraper/input-schema?fpr=qn4m5w). Set `maxItems` to a positive number to cap saved rows across the run. Check the [pricing tab](https://apify.com/maximedupre/tiktok-live-search-scraper/pricing?fpr=qn4m5w) for run and row charges.

## Run the Actor

Run this in a shell with `curl`, `jq`, and `APIFY_TOKEN` already set. Start one run and keep its ID so you can resume polling if interrupted. The loop waits for the Actor's terminal status. It has no local time limit.

```sh
(
  set -eu
  : "${APIFY_TOKEN:?Set APIFY_TOKEN before starting}"
  api=https://api.apify.com/v2
  apify() {
    curl --silent --show-error --fail \
      --header "Authorization: Bearer $APIFY_TOKEN" "$@"
  }
  run=$(apify --request POST --header 'Content-Type: application/json' \
    --data-binary @input.json "$api/acts/maximedupre~tiktok-live-search-scraper/runs")
  run_id=$(printf '%s' "$run" | jq -er '.data.id')
  printf 'Apify run ID: %s\n' "$run_id" >&2

  while :; do
    run=$(apify "$api/actor-runs/$run_id")
    status=$(printf '%s' "$run" | jq -er '.data.status')
    case "$status" in
      SUCCEEDED|FAILED|ABORTED|TIMED-OUT) break ;;
    esac
    sleep 10
  done

  printf 'Apify run status: %s\n' "$status" >&2
  dataset_id=$(printf '%s' "$run" | jq -er '.data.defaultDatasetId')
  apify "$api/datasets/$dataset_id/items?format=json" > results.json
)
```

Treat rows from `FAILED`, `ABORTED`, or `TIMED-OUT` runs as partial and report the status. If polling stops on an HTTP error, resume with the saved run ID. If the start request fails after it may have reached Apify, check recent runs before sending another start request, since each launch can incur charges. See the [Actor Store page](https://apify.com/maximedupre/tiktok-live-search-scraper?fpr=qn4m5w) and [API tab](https://apify.com/maximedupre/tiktok-live-search-scraper/api?fpr=qn4m5w).

### Read the run log

For empty or incomplete results, fetch the log. Replace `RUN_ID` with the run ID printed above.

```sh
run_id='RUN_ID'
curl --silent --show-error --fail \
  --header "Authorization: Bearer $APIFY_TOKEN" \
  "https://api.apify.com/v2/logs/$run_id" > run.log
```

## Find active rooms for a topic

1. Enter the topics in `keywords` and the search countries in `regions`. Read [Keyword results](references/keyword-results.md) for country selection, caps across countries, recent chat, or top gifters.

   ```json
   {"resultType":"rooms","findRoomsBy":"keyword","keywords":["football"],"regions":["GB","US"],"maxResultsPerKeyword":5,"maxItems":5}
   ```

2. Follow [Run the Actor](#run-the-actor).
3. Report the returned rooms with `keyword`, `creatorHandle`, `sourceUrl`, and `viewerCount` when available. If results are empty or incomplete, [fetch the run log](#read-the-run-log) and use [Keyword results](references/keyword-results.md) to interpret the warnings.

## Browse active rooms in a category

1. Use the category name as TikTok shows it. If two categories have the same name, use the full category page URL without its query or fragment. Set `maxResultsPerKeyword` to cap eligible rooms per category.

   ```json
   {"resultType":"rooms","findRoomsBy":"category","categories":["Gaming"],"maxResultsPerKeyword":5,"maxItems":5}
   ```

2. Follow [Run the Actor](#run-the-actor).
3. Report matching rooms with `category`, `title`, `sourceUrl`, and available `viewerCount` values. For empty or incomplete results, [fetch the run log](#read-the-run-log) and report what it says about category coverage.

## Check whether a known creator or room is live

1. Use `creators` for handles, profile URLs, or live page URLs.

   ```json
   {"resultType":"rooms","findRoomsBy":"creator","creators":["@tiktok"],"maxItems":1}
   ```

   For a specific broadcast, use `findRoomsBy: roomId` and replace the example ID with a `roomId` from a room row. Keep the ID as text to preserve all digits.

   ```json
   {"resultType":"rooms","findRoomsBy":"roomId","roomIds":["7713187734328380181"],"maxItems":1}
   ```

2. Follow [Run the Actor](#run-the-actor).
3. For a returned room, give its `roomId`, `sourceUrl`, and any requested viewer or stream details. For an empty dataset, report that the run saved no rooms. Use the run status, [run log](#read-the-run-log), and viewer filters to explain what the lookup established. Report failed checks as incomplete. Viewer filters can exclude a room even after the Actor verifies that it is live.

## Compare creators in the daily LIVE ranking

1. Choose `today` or `yesterday` in TikTok's United States pool (US+) ranking calendar.

   ```json
   {"resultType":"creatorRanking","rankingDay":"yesterday","maxItems":10}
   ```

2. Follow [Run the Actor](#run-the-actor).
3. Sort the returned entries by `rank`. Report `rankingDate`, `rank`, `creator.handle`, and available `giftValue` and `giftValueUnit` together. Keep gaps in the published ranks because the Actor skips entries whose creator profile it cannot verify.

## Gotchas

- A successful run may save zero rows or finish only some searches. Report the outcome for each input when the run log gives enough detail.
- Omit `minViewerCount` to keep rooms with missing viewer counts by default. Setting it to `0` keeps any known count but skips unknown counts under the default `includeRoomsWithoutViewerCount: auto`. Set `includeRoomsWithoutViewerCount` to `include` or `exclude` to decide explicitly.
- A matching room can count toward more than one search's cap, but the dataset saves each room once.
