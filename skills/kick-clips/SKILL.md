---
name: "kick-clips"
description: "Finds and ranks public Kick clips by channel, category, or globally, and enriches one Kick clip URL with metadata, media details, a transcript, and an optional translation. Use when the user wants top or newest Kick clips, new clips from a Kick channel since the last run, a Kick clip transcript or translation, or a source video link for a Kick clip."
---

# Kick Clips

Runs the Apify Actor `maximedupre~kick-clips` with curl. It needs only the `APIFY_TOKEN` environment variable. Store page: https://apify.com/maximedupre/kick-clips. Every input field is listed at https://apify.com/maximedupre/kick-clips/input-schema. Costs are at https://apify.com/maximedupre/kick-clips/pricing.

## Run protocol (all use cases)

1. Start the run. Save `data.id` and `data.defaultDatasetId` from the response:
   ```bash
   curl -s -X POST "https://api.apify.com/v2/acts/maximedupre~kick-clips/runs" \
     -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
     -d '<input JSON>'
   ```
2. Poll `GET https://api.apify.com/v2/actor-runs/<runId>` (same header) every 5 to 10 seconds until `data.status` is `SUCCEEDED`, `FAILED`, `ABORTED`, or `TIMED-OUT`.
3. Fetch rows: `GET https://api.apify.com/v2/datasets/<defaultDatasetId>/items?format=json` (same header).
4. If the status is not `SUCCEEDED`, or the dataset is empty, read the run log at `GET https://api.apify.com/v2/actor-runs/<runId>/log`. An unknown channel or category, or filters that match nothing, end the run with zero rows and a log warning, not an error.

Always set `maxItems` on discovery. Without it the run returns every matching clip, which costs more and takes longer. Check the pricing tab before large runs.

## Use case 1: Find the top or newest clips of a channel, category, or the whole site

1. Pick `discoveryScope`: `channel` (set `channel` to the slug after `kick.com/`), `category` (set `category` to the exact name or slug), or `global`.
2. Pick `recencyWindow` (`lastHour`, `last24Hours`, `last7Days`, `last30Days`, `allTime`) and `rankBy` (`totalViews` or `newest`).
3. Add optional filters: `minimumViews`, `minimumViewsPerHour`, `maxDurationSeconds`, `excludeMature`.
4. Run the protocol. Rows come back ordered by `rank`.

```json
{"findClipsBy":"discover","discoveryScope":"channel","channel":"blame","recencyWindow":"last7Days","rankBy":"totalViews","maxItems":20}
```

Each row has `clipId`, `clipUrl`, `title`, `channel`, `creator`, `category`, `publishedAt`, `views`, `viewsPerHour`, `isMature`, `durationSeconds`, `videoUrl`, `thumbnailUrl`, `rank`. Discovery rows have no transcript or media details; use case 3 adds them for one clip.

## Use case 2: Monitor a channel or category for new clips

Use on a schedule, so each run reports only clips not seen before.

1. Run once with `onlyNewClips: true` and a `maxItems` cap. The first run can return all matching clips and sets the baseline.
2. Repeat the same input on each later run. Each run returns only clips earlier runs did not deliver.
3. An empty dataset on a later run means no new clips, not a failure.

```json
{"findClipsBy":"discover","discoveryScope":"channel","channel":"blame","recencyWindow":"last24Hours","rankBy":"newest","onlyNewClips":true,"maxItems":50}
```

Keep the input identical between runs. Changing the scope or filters changes which clips count as new.

## Use case 3: Get a transcript, translation, media details, or video link for one clip

1. Take the full clip URL (`https://kick.com/<channel>/clips/<clipId>`; it must be on the `kick.com` domain).
2. Run with `findClipsBy: "enrich"` and `clipUrl`. Add `translationLanguage` (for example `English`) to also get a translated transcript.
3. Read `transcript.segments[]` (`startSeconds`, `endSeconds`, `text`), `transcript.translation.segments[]`, `media` (resolution, codecs, bitrate, file size), and `videoUrl`.

```json
{"findClipsBy":"enrich","clipUrl":"https://kick.com/blame/clips/clip_01M7VXE5584VFZ9CQZH7DCXJNF","translationLanguage":"Spanish"}
```

This mode handles one URL per run. For many clips, discover them first (use case 1), then start one enrich run per `clipUrl` you need. Read [references/output-fields.md](references/output-fields.md) when you need the full shape of `media` and `transcript`.

## Gotchas

- `creator` is the Kick user who clipped the clip, not the stream owner. It holds only `username`. The channel the clip came from is `channel` (`channelId` and `slug`). Many clips of one channel show one `channel.channelId` and many different `creator.username` values; that is correct. Group by channel with `channel.channelId`, never by `creator`.
- `rankBy` accepts only `totalViews` and `newest`. Other values fail input validation.
- Fields for the other mode are ignored (`clipUrl` in discover mode, `channel` in enrich mode), so a wrong `findClipsBy` silently runs the wrong mode.
- `channel` and `category` must match exactly. Use the slug from the Kick link when unsure.
- `videoUrl` is an HLS playlist (`.m3u8`), not a single mp4. It and `media` can be null when Kick provides no video, and `transcript` is null when no speech is found.
- An unsupported translation language returns the original transcript only.
- Treat `APIFY_TOKEN` as secret: never print it or write it to files.
