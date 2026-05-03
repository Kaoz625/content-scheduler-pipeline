# Content Scheduler Pipeline — Replit Agent Build Spec

Build from scratch. Commit with "replit: " prefix.

## Purpose
Takes outputs from all other pipelines (images, videos, captions) and queues them for posting to social media platforms on a schedule. Supports Instagram, TikTok, Twitter/X, and buffer-style scheduling.

## Stack
- Python 3.11+, anthropic, requests, schedule, sqlite3 (built-in), python-dotenv

## Tasks

### 1. db.py — SQLite queue
```python
# Table: scheduled_posts
# id, platform, media_path, caption, scheduled_at, status (pending|posted|failed), created_at
def add_post(platform, media_path, caption, scheduled_at) -> int
def get_pending() -> list[dict]
def mark_posted(post_id) -> None
def mark_failed(post_id, error) -> None
```
Single `queue.db` file. No external DB needed.

### 2. platforms/buffer_provider.py
Buffer API integration:
- Auth: `BUFFER_ACCESS_TOKEN`
- `schedule_post(profile_id, text, media_url, scheduled_at) -> str`
- Supports image + video posts
- `get_profiles() -> list[dict]` — list connected accounts

### 3. platforms/instagram_provider.py
Instagram Graph API (Meta Business):
- Auth: `INSTAGRAM_ACCESS_TOKEN` + `INSTAGRAM_ACCOUNT_ID`
- `post_image(image_path, caption) -> str` — immediate post
- `post_reel(video_path, caption) -> str`
- Note: scheduling requires Creator Studio or Buffer

### 4. platforms/twitter_provider.py
Twitter/X API v2:
- Auth: `TWITTER_BEARER_TOKEN`, `TWITTER_API_KEY`, `TWITTER_API_SECRET`, `TWITTER_ACCESS_TOKEN`, `TWITTER_ACCESS_SECRET`
- `post_tweet(text, media_path=None) -> str`

### 5. content_picker.py
Watches output folders from other pipelines and auto-queues new content:
```python
WATCH_DIRS = {
    "images": "../infinite-image-pipeline/outputs/",
    "videos": "../infinite-video-pipeline/outputs/",
    "captions": "../infinite-caption-pipeline/outputs/",
}
def scan_and_queue(platform="instagram", schedule_gap_minutes=60) -> int:
    """Queues new unposted files found in watched dirs. Returns count queued."""
```

### 6. scheduler.py
Background scheduler loop:
- Checks `queue.db` every minute for `scheduled_at <= now AND status = pending`
- Posts via appropriate platform provider
- Marks posted/failed
- Daily summary report via Claude

### 7. main.py CLI
```
python main.py add --platform instagram --media outputs/image.jpg --caption "NYC vibes" --when "2026-05-04 18:00"
python main.py scan --platform instagram  # auto-queue from pipeline outputs
python main.py run  # start scheduler daemon
python main.py list  # show pending posts
python main.py report  # Claude-generated daily summary
```

### 8. requirements.txt
```
anthropic>=0.39.0
requests>=2.31.0
schedule>=1.2.0
python-dotenv>=1.0.0
tqdm>=4.66.0
```

### 9. .env.example
```
ANTHROPIC_API_KEY=
BUFFER_ACCESS_TOKEN=
INSTAGRAM_ACCESS_TOKEN=
INSTAGRAM_ACCOUNT_ID=
TWITTER_API_KEY=
TWITTER_API_SECRET=
TWITTER_ACCESS_TOKEN=
TWITTER_ACCESS_SECRET=
```
