# rclone commands

## rclone copy

```bash
rclone --verbose --cache-workers 64 --transfers 64 --retries 32 -P —filter-from ~/.config/rclone/rclone_filters.txt copy ~/source OCI:bucket
```
