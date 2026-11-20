# Output fields

Each dataset row describes one video. Always present: `videoUrl`, `sourceUrl`. All others appear only when Snapchat shows them. Story highlight rows have no duration or counts. Anonymous creators have no creator name.

| Field | Type | Meaning |
| --- | --- | --- |
| `videoUrl` | string | Direct video file link, no watermark |
| `sourceUrl` | string | The Snapchat link the row came from |
| `title` | string | Title or caption |
| `thumbnailUrl` | string | Thumbnail image link |
| `videoHeight` | integer | Height in pixels, such as 720 |
| `durationSeconds` | number | Length in seconds (Spotlight only) |
| `uploadedAt` | string | Upload time, ISO 8601 |
| `creatorUsername` | string | Creator's Snapchat username |
| `creatorDisplayName` | string | Creator's display name |
| `viewCount`, `likeCount`, `commentCount`, `shareCount` | integer | Counts Snapchat shows (Spotlight only) |
| `hashtags` | array of strings | Hashtags on the video |
