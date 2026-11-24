---
name: "bilibili-video-downloader"
description: "Downloads public Bilibili videos as MP4 plus a separate audio file and extracts video details (title, uploader, views, likes, tags, chapters, comments) with the Bilibili Video Downloader Apify Actor. Use when the user wants to download a Bilibili video, get a bilibili.com or b23.tv link as MP4 or audio, collect Bilibili video metadata, or pull Bilibili comments."
---

# Bilibili Video Downloader

Runs the Apify Actor `maximedupre~bilibili-video-downloader` through the Apify REST API with curl. It needs the `APIFY_TOKEN` environment variable. Store page: https://apify.com/maximedupre/bilibili-video-downloader?fpr=qn4m5w

Every input field is documented at https://apify.com/maximedupre/bilibili-video-downloader/input-schema?fpr=qn4m5w. Prices are at https://apify.com/maximedupre/bilibili-video-downloader/pricing?fpr=qn4m5w.

## Run protocol (all use cases)

1. Start the run. The response holds `data.id` (the run id) and `data.defaultDatasetId`:
   ```bash
   curl -s -X POST "https://api.apify.com/v2/acts/maximedupre~bilibili-video-downloader/runs" \
     -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
     -d '<input JSON>'
   ```
2. Poll until `data.status` is `SUCCEEDED`, `FAILED`, `ABORTED`, or `TIMED-OUT` (wait a few seconds between polls):
   ```bash
   curl -s "https://api.apify.com/v2/actor-runs/<runId>" -H "Authorization: Bearer $APIFY_TOKEN"
   ```
3. Only on `SUCCEEDED`, fetch the rows. One row per video link:
   ```bash
   curl -s "https://api.apify.com/v2/datasets/<defaultDatasetId>/items?format=json" -H "Authorization: Bearer $APIFY_TOKEN"
   ```

If the status is not `SUCCEEDED`, report the status to the user rather than retrying blindly.

## Use case 1: Download videos as MP4 and audio

Use for "download this Bilibili video", or to get the audio track.

1. Collect the links. Full `https://www.bilibili.com/video/BV...` links and `https://b23.tv/...` short links both work; each must start with `https://`.
2. Pick `quality`: `low` (lowest offered), `medium` (default), `high` (highest). Most videos offer only two qualities, so `medium` and `high` often match. Check the `videoVersions` array in the result to see what exists.
3. Run the protocol with:
   ```json
   {"videoUrls": [{"url": "https://www.bilibili.com/video/BV1xx411c7mD"}], "quality": "high"}
   ```
4. Read `videoFileUrl` (MP4) and `audioFileUrl` from each row, then download both files with `curl -L -o <name>.mp4 "<videoFileUrl>"`. The file links expire soon after the run, so download them in the same session, and rerun the Actor for fresh links.

## Use case 2: Collect video details only

Use for research, monitoring or catalogs where no file is needed. It is cheaper per video than a download.

1. Run the protocol with `"quality": "none"`:
   ```json
   {"videoUrls": [{"url": "https://b23.tv/example"}], "quality": "none"}
   ```
2. Each row has `title`, `description`, `duration` (seconds), `publishedAt`, `category`, `tags`, `chapters`, `uploader` (name, profileUrl, followerCount, isVerified), `viewCount`, `likeCount`, `commentCount`, `shareCount`, `favoriteCount`. No file links are set.

## Use case 3: Collect video comments

Use for sentiment analysis or audience research.

1. Run the protocol with comments on. `maxComments` caps comments per video (default 20; 0 means all available). Each comment is billed as its own event, so set a cap before using 0:
   ```json
   {"videoUrls": [{"url": "https://www.bilibili.com/video/BV1xx411c7mD"}], "quality": "none", "includeComments": true, "maxComments": 100}
   ```
2. Read the `comments` array of each row: `text`, `author`, `likeCount`, `publishedAt`.

## Gotchas

- Use `maxItems` to cap how many videos run; empty or 0 processes every link. Set it when testing with a long list, because each video is billed.
- Only public videos work; the Actor needs no Bilibili account or cookie.
- For one part of a multi-part video, add `?p=<part number>` to the link. A part that does not exist gives no row, so a run can return fewer rows than links.
- The audio file is separate; it is not merged into the MP4.
- `videoUrls` is a list of objects with a `url` key, not plain strings.
- For cost questions read the pricing tab; for programmatic details read https://apify.com/maximedupre/bilibili-video-downloader/api?fpr=qn4m5w.
