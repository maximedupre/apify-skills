---
name: google-drive-downloader
description: "Downloads files from public Google Drive folders as ZIP archives or individual files. Use when archiving a shared folder, previewing its file list, exporting Google Workspace files, or collecting new or changed files on repeat runs."
---

# Google Drive Folder Downloader

Use public Google Drive folders that anyone with the link can view. Put each folder link or ID in `folders`. Keep the full link if it contains a resource key because Drive may need that key to open the folder. Replace the sample folder in the examples with the user's folder.

[Actor Store page](https://apify.com/maximedupre/google-drive-downloader?fpr=qn4m5w) · [Input fields](https://apify.com/maximedupre/google-drive-downloader/input-schema?fpr=qn4m5w) · [API tab](https://apify.com/maximedupre/google-drive-downloader/api?fpr=qn4m5w)

## Run the Actor

1. Save the chosen example with the user's values as `input.json`. Use the `APIFY_TOKEN` environment variable for authentication and keep its value out of logs and reports. For cost questions, check the run start and per-row charges on the [Pricing tab](https://apify.com/maximedupre/google-drive-downloader/pricing?fpr=qn4m5w).
2. Start the run with the JSON body and save `data.id` from the response as `runId`.

   ```sh
   curl -fsS -X POST \
     -H "Authorization: Bearer $APIFY_TOKEN" \
     -H 'Content-Type: application/json' \
     --data-binary @input.json \
     'https://api.apify.com/v2/acts/maximedupre~google-drive-downloader/runs'
   ```

3. Poll the run every few seconds until `data.status` is `SUCCEEDED`, `FAILED`, `ABORTED`, or `TIMED-OUT`. Keep the run ID so polling can resume across tool calls. If a poll fails temporarily, retry the GET for the same run. For a terminal failure, report the run ID, status, and any available `data.statusMessage`. Label any rows from that run as partial.

   ```sh
   curl -fsS -H "Authorization: Bearer $APIFY_TOKEN" \
     "https://api.apify.com/v2/actor-runs/$runId"
   ```

4. Set `defaultDatasetId` to `data.defaultDatasetId` from the terminal response. Fetch the manifest. Check the `X-Apify-Pagination-Total` response header and, if needed, append `&offset=N` with the number of rows already collected until you have the whole manifest.

   ```sh
   curl -fsS -D manifest-headers.txt -o manifest.json \
     -H "Authorization: Bearer $APIFY_TOKEN" \
     "https://api.apify.com/v2/datasets/$defaultDatasetId/items?format=json"
   ```

## Preview the files before downloading

1. Set `mode` to `preview`. This example lists PDFs in the folder and its immediate subfolders. Remove `maxDepth` to include deeper subfolders.

   ```json
   {"folders":["https://drive.google.com/drive/folders/1Xl5zaDP11_qmxslAXhjZENWDQncYtBla"],"mode":"preview","fileTypes":["pdf"],"maxDepth":1}
   ```

2. Follow [Run the Actor](#run-the-actor).
3. Report the matching `relativePath` values grouped by `inputFolder`. Include `sizeBytes` where present if the user needs to estimate download size. Preview rows describe source files and contain no archive or individual delivery links.

## Archive a folder for handoff

1. Set `downloadFormat` to `singleZip` and `zipLayout` to `folders` to preserve subfolders. Read [Split ZIPs](references/split-zips.md) if the user needs separate archives.

   ```json
   {"folders":["https://drive.google.com/drive/folders/1Xl5zaDP11_qmxslAXhjZENWDQncYtBla"],"mode":"full","downloadFormat":"singleZip","zipLayout":"folders"}
   ```

2. Follow [Run the Actor](#run-the-actor).
3. Return each distinct `zipDownloadUrl` once, since all rows in the same archive share a link. Use the manifest to show which files each archive contains. Keep the full link, including its query string, so the download remains accessible.

## Export files for a spreadsheet or another tool

1. Set `downloadFormat` to `individual` so each delivered file has its own link. This example selects spreadsheets and exports Google Sheets as CSV. Read [Export settings](references/exports.md) when choosing a format for another Google Workspace file type, selecting only Google Sheets CSV exports, or collecting only existing CSV files.

   ```json
   {"folders":["https://drive.google.com/drive/folders/1Xl5zaDP11_qmxslAXhjZENWDQncYtBla"],"mode":"full","downloadFormat":"individual","fileTypes":["spreadsheet"],"sheetsFormat":"CSV"}
   ```

2. Follow [Run the Actor](#run-the-actor).
3. Return `fileDownloadUrl` with `relativePath` and `exportFormat` when present. Use `fileUrl` when the user asks for the original Drive view link. Apply the matching row rule in the export reference when the user requests only Google Sheets CSV exports.

## Collect new or changed files on repeat runs

1. Set `mode` to `changes` and use the same Apify account and folder as before so the Actor can use its saved download history. On the first download for that folder, matching files are eligible for delivery.

   ```json
   {"folders":["https://drive.google.com/drive/folders/1Xl5zaDP11_qmxslAXhjZENWDQncYtBla"],"mode":"changes","downloadFormat":"individual"}
   ```

2. Follow [Run the Actor](#run-the-actor).
3. Return the new manifest rows. For `individual`, return `fileDownloadUrl` for each row. For `singleZip` or `splitZip`, return each distinct `zipDownloadUrl` once. Read [Download history](references/history.md) if the user needs to interpret an empty manifest or wants to change the output format on a repeat run.

## Gotchas

- The Store form starts with `maxItems: 1` for sampling. For an uncapped API run, omit `maxItems` or set it to `0`.
- An input folder may overlap another input folder. Expect at most one manifest row per Drive file ID in a run. Use `inputFolder` and `relativePath` together to identify the path the Actor used.
- A successful run can still skip folders or files. Check the run status message and log for skipped files, inaccessible folders, or a charge limit when the manifest is empty or expected files are missing. Report rows as delivered only when the requested delivery links are present.
