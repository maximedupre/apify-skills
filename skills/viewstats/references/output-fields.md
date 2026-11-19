# Video fields in a channel row

Present only when `includeLongVideos` (`longVideos`) or `includeShorts` (`shorts`) is true. Both arrays hold video objects of the same shape.

Per video:
- `videoId`, `title`, `description`, `uploadedAt`, `durationSeconds`, `thumbnailUrl`
- `viewCount`, `likeCount`, `commentCount`
- `averageDailyViews`, `averageWeeklyViews`, plus daily and weekly revenue estimate ranges (`low`, `high`, US dollars)
- `expectedViewsByHoursAfterUpload`: array of `{hoursAfterUpload, lowViews, highViews}`, the typical view range Viewstats states. Compare a video's `viewCount` against the range for its age to tell over- from under-performers.

Channel-level fields beyond the basics: `averages` (`daily`, `weekly`, `monthly`, `yearly`, `uploadFrequency`), `projections.forecasts`, `projections.milestones`, `similarChannels`, `featuredVideo`, `longsVsShorts` (uploads and views split by period).
