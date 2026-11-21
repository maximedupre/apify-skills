# Enrich row fields

An enrich row has every discovery field except `rank` (null), plus:

- `media`: `fileType`, `fileSizeBytes`, `width`, `height`, `frameRate`, `videoCodec`, `audioCodec`, `bitrateKbps`, `dynamicRange`. Each value is null when Kick does not provide it.
- `transcript.detectedLanguage`: language of the original speech, such as `english`.
- `transcript.segments[]`: `startSeconds`, `endSeconds`, `text`, in order.
- `transcript.translation`: present only when `translationLanguage` was set and supported. It holds `language` and `segments[]` with the same timing as the original segments.

Join segment `text` values with a space to get a plain transcript.
