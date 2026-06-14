# Rclone → Telegram Bot

A Telegram bot that pulls media from any rclone remote (Dropbox, Google Drive, etc.)
and uploads it to a Telegram chat or channel with live progress, automatic
thumbnails, large-file splitting, and optional source cleanup.

## Features

- ⚡ Concurrent uploads (1–5 files at once)
- 📊 Live download & upload progress with percentage, speed, and ETA
- 🎬 Video or 📄 Document mode chosen per job
- 🖼 Automatic video thumbnails via ffmpeg
- ✂️ Auto-split files larger than the Telegram size limit (~1.99 GB)
- 🗑 Optional auto-delete from the remote after a successful upload
- 👥 Multi-user authorization
- 🔁 `/retry` for files that failed in the last job
- 🚦 Runtime bandwidth limiting
- 🌐 Built-in HTTP health check endpoint
- 💾 Persisted stats across restarts

## Requirements

- Python 3.11+
- [`rclone`](https://rclone.org) on the PATH with a configured remote
- [`ffmpeg`](https://ffmpeg.org) / `ffprobe` on the PATH
- Telegram API credentials and a bot token

The provided `Dockerfile` installs `rclone` and `ffmpeg` for you.

## Configuration

Copy the template and fill in your values:

```bash
cp config.env.example config.env
```

| Variable | Required | Description |
| --- | --- | --- |
| `API_ID` / `API_HASH` | yes | From https://my.telegram.org/apps |
| `BOT_TOKEN` | yes | From @BotFather |
| `OWNER_ID` | yes | Your numeric Telegram user ID |
| `DUMP_CHAT_ID` | yes | Target chat/channel ID (bot must be admin) |
| `RCLONE_CONF_URL` / `RCLONE_CONF` | one | rclone config via URL or inline; or mount the file |
| `AUTHORIZED_USERS` | no | Extra user IDs (comma/semicolon separated) |
| `CONCURRENT_JOBS` | no | Files per job, 1–5 (default 1) |
| `SPLIT_SIZE` | no | Split threshold in bytes |
| `BW_LIMIT` | no | rclone bandwidth limit, e.g. `8M` |
| `HEALTH_PORT` | no | Health endpoint port (default 8080) |
| `BOARD_REFRESH_INTERVAL` | no | Seconds between status edits (2–30) |
| `STATS_FILE` | no | Path for persisted stats |

See `config.env.example` for the full list.

## Running

### Docker Compose (recommended)

Place your `rclone.conf` next to `docker-compose.yml`, then:

```bash
docker compose up -d --build
```

### Docker

```bash
docker build -t rclone-tg-bot .
docker run -d --env-file config.env -p 8080:8080 \
  -v $(pwd)/rclone.conf:/root/.config/rclone/rclone.conf:ro \
  rclone-tg-bot
```

### Local

```bash
pip install -r requirements.txt
python bot.py
```

## Commands

| Command | Description |
| --- | --- |
| `/dl <remote>:<path>` | Download & upload everything at that path |
| `/retry` | Re-run files that failed in the last job |
| `/queue` | Show files currently being processed |
| `/setdelete on\|off` | Toggle auto-delete from the remote |
| `/concurrent <1-5>` | Set concurrent files (next job) |
| `/setbwlimit <8M\|off>` | Limit rclone download bandwidth |
| `/setrefresh <2-30>` | Status board refresh interval |
| `/status` | Bot stats & health |
| `/logs` | Last 30 log lines |
| `/cancel` | Stop the current job |
| `/restart` | Restart the bot process |
| `/stop` | Shut the bot down |

## Health check

```bash
curl http://localhost:8080/health
```

Returns uptime, active job count, and upload counters as plain text.

## License

[MIT](LICENSE)
