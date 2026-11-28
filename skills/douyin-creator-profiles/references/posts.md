# Post filters and output

- Use `postType: "photos"` for photo posts or `"all"` for both videos and photo posts. Omitting the field also selects both. The Actor checks only the first page of each creator's posts for photos, so treat a photo review as a sample even with no date filter.
- Set `recentPostsSince` to an inclusive `YYYY-MM-DD` date at midnight UTC or a whole number of days, weeks, months, or years before the run's UTC date. Omit it for no date filter.
- Set `excludePinnedPosts: true` to skip posts marked as pinned. Rows with unknown pin status can still appear. If the review requires posts confirmed as unpinned, keep only rows with `isPinned: false`.
- `maxPostsPerCreator` applies after filters and counts both post formats. Omit it or set it to `0` for no per-creator cap. Videos appear newest first for each creator and use the shared post cap before photo posts.
- To collect profiles and posts together, set `resultType: "profilesAndPosts"`. Allow room for both in `maxItems` because a profile consumes one row. Separate `profile` and `post` rows using the output `resultType` and match `secUid` on profile rows to `author.secUid` on post rows when present.
- Use the output `type` field to distinguish `video` from `photo`. Use `imageUrls` for photo image links or `video.downloads[].fileUrl` for video downloads. Check `video.downloads[].fileUrlExpiresAt` when present before using a download link. Keep the post `url` for citation.
