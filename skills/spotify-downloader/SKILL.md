---
name: "spotify-downloader"
description: "Resolves public Spotify tracks into dataset rows with metadata and MP3 download links, from track URLs or search terms, via the Apify Actor maximedupre/spotify-downloader. Use when asked to get an MP3 link for a Spotify track, export Spotify track metadata (artist, album, ISRC, duration), search Spotify for tracks, or save Spotify MP3 files to Apify storage."
---

# Spotify Downloader

Runs the Apify Actor [maximedupre/spotify-downloader](https://apify.com/maximedupre/spotify-downloader) (Actor id `maximedupre~spotify-downloader`). It handles public tracks only: no playlists, albums, artist pages, podcasts, or private content. Spotify cookies, login, and API key are not needed.

Every use case below shares one run protocol, then differs in input and in how the rows are used. Read [the input schema](https://apify.com/maximedupre/spotify-downloader/input-schema) for every field, and [pricing](https://apify.com/maximedupre/spotify-downloader/pricing) for cost questions.

## Run protocol

`APIFY_TOKEN` must be set in the environment. Plain curl needs no other setup and has no time limit.

1. Start the run. The response holds `data.id` (run id) and `data.defaultDatasetId`.

```bash
curl -s -X POST "https://api.apify.com/v2/acts/maximedupre~spotify-downloader/runs" \
  -H "Authorization: Bearer $APIFY_TOKEN" -H "Content-Type: application/json" \
  -d '{"trackUrls":["https://open.spotify.com/track/1dGr1c8CrMLDpV6mPbImSI"]}'
```

2. Poll until `data.status` is terminal (`SUCCEEDED`, `FAILED`, `ABORTED`, `TIMED-OUT`). Wait a few seconds between polls.

```bash
curl -s "https://api.apify.com/v2/actor-runs/<runId>" -H "Authorization: Bearer $APIFY_TOKEN"
```

3. Fetch the rows.

```bash
curl -s "https://api.apify.com/v2/datasets/<defaultDatasetId>/items?format=json" \
  -H "Authorization: Bearer $APIFY_TOKEN"
```

4. Check the outcome before reporting: compare the row count with the input count, and when rows are missing or the dataset is empty, follow the first two Gotchas.

Row fields: `trackId`, `trackName`, `artistNames`, `albumName`, `durationMs`, `explicit`, `releaseDate`, `coverImageUrl`, `isrc`, `popularity`, `spotifyUrl`, `media.downloadUrl` (always present), `media.audioPreviewUrl` (only with `includeAudioPreview`), `savedFile` (only with `includeSavedMp3File`). Optional fields are missing when Spotify gives no value.

## Use case 1: MP3 links and metadata for known tracks

Use when the user has Spotify track links, URIs, or 22-character IDs and needs the MP3 link and details for each.

1. Collect the tracks into `trackUrls` (links, `spotify:track:` URIs, or IDs, up to 1000).
2. Set `maxTracks` to `0` to get every track, since a lower value stops the run early.
3. Run the protocol.
4. Report `trackName`, `artistNames`, and `media.downloadUrl` per row. Name each missing track with its reason.

```json
{
  "trackUrls": [
    "https://open.spotify.com/track/1dGr1c8CrMLDpV6mPbImSI",
    "spotify:track:7qiZfU4dY1lWllzX7mPBI3"
  ],
  "maxTracks": 0
}
```

## Use case 2: Find tracks by search and filter them

Use when the user describes songs by title, artist, album, or genre, or wants a clean/explicit or length-bounded list.

1. Put each search in `keywords`. Each term returns up to `maxTracksPerSearch` tracks (1 to 100, default 5).
2. Set `maxTracks` as the overall cap (it spans all searches and URLs; `0` means no cap). A cap protects the budget.
3. Add filters when asked: `explicitContent` (`all`, `explicit_only`, `non_explicit_only`), `minDurationSeconds`, `maxDurationSeconds`. Filtered tracks do not count toward the limits, so the run keeps looking until the limits fill.
4. Run the protocol and summarize rows per search term using `trackName` and `artistNames`. An empty result is possible: follow the first two Gotchas.

```json
{
  "keywords": ["lofi hip hop", "Daft Punk Discovery"],
  "maxTracksPerSearch": 10,
  "maxTracks": 20,
  "explicitContent": "non_explicit_only",
  "maxDurationSeconds": 360
}
```

## Use case 3: Archive MP3 files and previews in Apify storage

Use when the user wants the audio files themselves kept, or a 30-second preview link.

1. Set `includeSavedMp3File` to `true` for stored MP3 files, and `includeAudioPreview` to `true` for preview links. Each adds its own charge, so enable only what was asked.
2. Run the protocol.
3. For each row, `savedFile.key` is a key in the run's default key-value store. Fetch the file:

```bash
curl -s -o "<trackId>.mp3" -H "Authorization: Bearer $APIFY_TOKEN" \
  "https://api.apify.com/v2/key-value-stores/<defaultKeyValueStoreId>/records/<savedFile.key>"
```

`defaultKeyValueStoreId` is in the run response from step 2 of the protocol. If a row has no `savedFile`, the log carries a warning that the save failed after several tries; the row still has `media.downloadUrl`.

## Gotchas

- Zero rows with status `SUCCEEDED` is possible, with nothing charged for rows. It happens when every input is an invalid link, a track Spotify does not have, or removed by your filters. Only failures that exhaust their retries with no row saved end the run `FAILED`. So check the row count even on `SUCCEEDED`.
- Skipped tracks are not saved as failure rows. Read the reason in the log: `GET https://api.apify.com/v2/actor-runs/<runId>/log` (with the bearer header). Each submitted URL, URI, or ID that is invalid, missing on Spotify, removed by your filters, or failed after retries should have a `Skipped ...` warning. Tracks found by search that fail get one summary warning, and duplicates are skipped silently. When a missing track has no `Skipped` line, report that the reason is unknown instead of guessing; offer a re-run with that single URL, or point the user to the [issues page](https://apify.com/maximedupre/spotify-downloader/issues/open).
- Only track links work. A playlist, album, or artist link is reported as not a Spotify track link.
- Charges apply to saved rows, plus the add-on events (audio preview, saved MP3 file) and a run-start event; the [pricing tab](https://apify.com/maximedupre/spotify-downloader/pricing) has the amounts. To cap spend, set `maxTracks` and the `maxTotalChargeUsd` query parameter on the start-run URL.
- API details for starting runs with other options: [API tab](https://apify.com/maximedupre/spotify-downloader/api).
