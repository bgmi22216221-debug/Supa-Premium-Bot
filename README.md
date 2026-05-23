# 💎 Supa Premium Bot

A Telegram bot that serves exclusive media (videos/photos) to approved premium users only. Features a full admin panel, 7-day subscription system with auto-expiry, ban/unban management, and DB-backed message auto-deletion.

---

## ✨ Features

**User Side**
- `/start` — Request access or resume watching from where you left off
- `/status` — View subscription expiry date and days remaining
- ⬅️ Previous / ▶️ Next inline buttons to navigate media
- Auto-delete media after 10 minutes (protect content enabled)
- Expiry warning notification 2 days before subscription ends

**Admin Side**
- Approve/Reject new users via inline buttons (no typing needed)
- `/approve <user_id>` — Manually approve a user (7-day subscription)
- `/reject <user_id>` — Reject a user
- `/ban <user_id> [reason]` — Ban a user with optional reason
- `/unban <user_id>` — Unban a user
- `/banned` — List all banned users
- `/pending` — View pending approval requests
- `/expiring` — View users expiring within 3 days
- `/stats` — Bot statistics (total, approved, pending, banned, media count)
- `/broadcast` — Broadcast messages to all active users (text, photo+caption, or exact copy)
- `/support on|off` — Toggle the Support button on media messages

**System**
- Auto-ban on subscription expiry (runs hourly)
- DB-backed scheduled deletion (survives bot restarts)
- Retry logic for missing source media (up to 10 retries)
- Per-user media history with 2% repeat chance for variety

---

## 🗄️ Database Schema

Uses **PostgreSQL** (Neon recommended) with these tables:

| Table | Purpose |
|---|---|
| `users` | User records, approval status, expiry |
| `media` | Source media message IDs |
| `user_history` | Per-user seen media log |
| `user_position` | Last sent media + bot message ID |
| `banned_users` | Banned user IDs + reason |
| `expiry_notified` | Tracks 2-day warning sends |
| `scheduled_deletes` | DB-persisted message deletion queue |

---

## ⚙️ Environment Variables

Set these as environment variables (or in a `.env` file):

| Variable | Description |
|---|---|
| `BOT_TOKEN` | Telegram bot token from @BotFather |
| `DATABASE_URL` | PostgreSQL connection string (with SSL) |
| `SOURCE_CHAT_ID` | ID of the private channel/group where media lives |
| `ADMIN_ID` | Your Telegram user ID |
| `ADMIN_USERNAME` | Admin username shown in messages (e.g. `@YourHandle`) |
| `SUPPORT_USERNAME` | Support contact username (e.g. `@Support`) |

---

## 🚀 Setup & Deployment

### 1. Clone the repo

```bash
git clone https://github.com/sxeditz78/Supa-Premium-Bot.git
cd Supa-Premium-Bot
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Set environment variables

```bash
export BOT_TOKEN="your_token"
export DATABASE_URL="postgresql://user:pass@host/dbname?sslmode=require"
export SOURCE_CHAT_ID="-100xxxxxxxxxx"
export ADMIN_ID="your_telegram_id"
export ADMIN_USERNAME="@YourHandle"
export SUPPORT_USERNAME="@SupportHandle"
```

### 4. Run the bot

```bash
python bot.py
```

### Deploying on AWS EC2 (systemd)

Create `/etc/systemd/system/bot.service`:

```ini
[Unit]
Description=Supa Premium Bot
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/Supa-Premium-Bot
EnvironmentFile=/home/ubuntu/Supa-Premium-Bot/.env
ExecStart=/usr/bin/python3 bot.py
Restart=always

[Install]
WantedBy=multi-user.target
```

Then:

```bash
sudo systemctl daemon-reload
sudo systemctl enable bot
sudo systemctl start bot
```

To update and restart:

```bash
git pull && sudo systemctl restart bot
```

---

## 📋 Commands Reference

### User Commands

| Command | Description |
|---|---|
| `/start` | Start bot / resume media |
| `/status` | Check subscription expiry |

### Admin Commands

| Command | Description |
|---|---|
| `/stats` | Bot statistics |
| `/pending` | Pending approval list |
| `/approve <id>` | Approve a user |
| `/reject <id>` | Reject a user |
| `/ban <id> [reason]` | Ban a user |
| `/unban <id>` | Unban a user |
| `/banned` | List all banned users |
| `/expiring` | Users expiring in 3 days |
| `/broadcast` | Broadcast to all active users |
| `/support on\|off` | Toggle support button |

---

## 📦 Requirements

```
python-telegram-bot==21.9
asyncpg
```

Python 3.10+ required.

---

## 🔒 Notes

- Media is sent with `protect_content=True` — forwarding and downloading is disabled
- Bot must be added as admin to the source channel/group to copy messages
- All admin reply messages auto-delete after 2 minutes to keep chat clean
- Broadcast messages auto-delete after 12 hours
