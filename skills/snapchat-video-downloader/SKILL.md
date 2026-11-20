---
name: snapchat-video-downloader
description: "Gets direct no-watermark download links and metadata for public Snapchat Spotlight videos and Story highlights by running the Snapchat Video Downloader Apify Actor. Use when the user wants to download or save Snapchat videos, back up Snapchat Spotlight clips, or collect creator names, titles, hashtags, and view, like, comment, share counts from Snapchat video links."
metadata:
  actor: "maximedupre/snapchat-video-downloader"
---

# Snapchat Video Downloader

Runs the Apify Actor `maximedupre~snapchat-video-downloader` with curl. Input is a list of public Snapchat video links. Output is one dataset row per video with a direct video file link and details. Store page: https://apify.com/maximedupre/snapchat-video-downloader

Requires the `APIFY_TOKEN` environment variable. Both use cases share the run steps below.

## Run the Actor

1. Start the run. Save `data.id` and `data.defaultDatasetId` from the response.
   ```bash
   curl -s -X POST "https://api.apify.com/v2/acts/maximedupre~snapchat-video-downloader/runs" \
     -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
     -d '{"startUrls":[{"url":"https://www.snapchat.com/@snapchat/spotlight/W7_EDlXWTBiXAEEniNoMPwAAYeHN6bGZ1dmVvAZ_ELfuUAZ_ELbWKAAAAAQ"}]}'
   ```
2. Poll until `data.status` is `SUCCEEDED`, `FAILED`, `ABORTED`, or `TIMED-OUT`. Wait 5 to 10 seconds between polls.
   ```bash
   curl -s "https://api.apify.com/v2/actor-runs/<runId>" -H "Authorization: Bearer $APIFY_TOKEN"
   ```
3. On `SUCCEEDED`, fetch the rows.
   ```bash
   curl -s "https://api.apify.com/v2/datasets/<defaultDatasetId>/items?format=json" -H "Authorization: Bearer $APIFY_TOKEN"
   ```
4. Done when the status is terminal and the dataset is fetched. On any other terminal status, report it to the user with the run's `statusMessage`.

Input has one field, `startUrls`: an array of `{"url": "https://..."}` objects. Spotlight links (`https://www.snapchat.com/@user/spotlight/ID` or `https://www.snapchat.com/spotlight/ID`) and Story highlight links (`https://www.snapchat.com/@user/highlight/ID`) work. Full schema: https://apify.com/maximedupre/snapchat-video-downloader/input-schema

## Use case 1: Save videos to disk

For offline copies, reuse, or backing up the user's own public videos.

1. Collect the links from the user. Put each in `startUrls`.
2. Run the Actor (steps above). For field meanings, read [references/output-fields.md](references/output-fields.md).
3. For each row, download `videoUrl` to a file: `curl -sL -o "<name>.mp4" "<videoUrl>"`. Name files from `creatorUsername` and `title` or the ID at the end of `sourceUrl`.
4. Match each `sourceUrl` against the submitted links. Read the run log warnings with `curl -s "https://api.apify.com/v2/actor-runs/<runId>/log" -H "Authorization: Bearer $APIFY_TOKEN"`, then tell the user which links produced no row and why (see the first Gotcha).

## Use case 2: Metrics and content research

For comparing Spotlight performance, ranking a creator's videos, or building a table of titles, hashtags, and thumbnails.

1. Run the Actor with all links in one run.
2. Read the fields the question needs. For field meanings, read [references/output-fields.md](references/output-fields.md).
3. Build the table or ranking as the user asks, such as a CSV with `sourceUrl`, `creatorUsername`, `title`, `hashtags`, `viewCount`. Compute rates such as likes per view yourself.
4. State which rows lack a field instead of treating it as zero.

## Gotchas

- Only public videos work. Links that are not `https://` or not public Snapchat videos are skipped with a warning. The run still succeeds unless every link fails to read, so a link can have no row because it was repeated, skipped, or failed to read.
- Fields Snapchat does not show are absent from the row. Check a field exists before using it (see [references/output-fields.md](references/output-fields.md)).
- Cost: pay per result row plus a small per-run start fee. For current prices, read https://apify.com/maximedupre/snapchat-video-downloader/pricing before large batches.
- API details beyond this skill: https://apify.com/maximedupre/snapchat-video-downloader/api
