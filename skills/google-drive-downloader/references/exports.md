# Export settings

Replace the example's `sheetsFormat` with the setting for the requested file type. Adjust `fileTypes` before starting the run because the Actor filters source files before converting them.

| Google Workspace file type | Setting | Choices | Change to `fileTypes` |
| --- | --- | --- | --- |
| Google Docs | `docsFormat` | `PDF`, `DOCX` | Set to `["document"]`. |
| Google Sheets | `sheetsFormat` | `XLSX`, `CSV` | Keep `["spreadsheet"]`. |
| Google Slides | `slidesFormat` | `PPTX`, `PDF` | Omit the field. |
| Google Drawings | `drawingsFormat` | `PNG`, `PDF` | Omit the field. |

When the user only wants specific Slides or Drawings, use `includeNames` with their file names to limit downloads. To return only the requested native file type from a mixed manifest, select rows by `mimeType`: `application/vnd.google-apps.document` for Docs, `application/vnd.google-apps.spreadsheet` for Sheets, `application/vnd.google-apps.presentation` for Slides, or `application/vnd.google-apps.drawing` for Drawings.

Format settings apply to files that need conversion. Uploaded spreadsheets keep their original format even when `sheetsFormat` is `CSV`. To collect only existing CSV files, use `includeNames: ["*.csv"]` and `skipNativeFiles: true`. Remove the example's `fileTypes` filter so the name pattern controls which files match.

For Google Sheets CSV exports only, select manifest rows whose source `mimeType` matches Sheets above and whose `exportFormat` is `"CSV"`. Use those rows for the response or downstream tool.
