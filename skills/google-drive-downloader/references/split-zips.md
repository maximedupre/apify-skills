# Split ZIPs

In the archive example, set `downloadFormat` to `splitZip` and `zipPartSizeMb` to the target size in MB. Omit `zipPartSizeMb` to use the 100 MB default.

Each archive opens on its own. A file larger than the target can produce a larger archive, so the target does not enforce a file size limit.
