# Download history

For each folder, the Actor compares a file's last modified time and MD5 checksum with the values saved for its last delivery. If both values are missing, the file remains eligible for download. Preview leaves download history as it was.

An empty manifest can mean that no eligible files were new or changed. Follow the empty-manifest check in [Gotchas](../SKILL.md#gotchas) before reporting that result.

History tracks source file changes. To deliver files again after changing a ZIP layout or Google Workspace export format, use `mode: "full"` so previous deliveries do not filter out unchanged source files.
