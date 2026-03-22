# Personal Task Manager - Implementation Plan

## Context

You want a way to quickly capture tasks from your phone and manage them later. You already have a battle-tested Telegram intake system for your case — this reuses that exact pattern for personal task management.

## Approach: Telegram Bot + Simple Web UI

**Why this wins over alternatives:**
- Telegram is already on your phone — zero friction to type a task
- A web UI at `localhost:5002` handles sorting/completing/prioritizing
- Reuses your existing infrastructure patterns (launchd, Flask, SQLite, python-telegram-bot)
- No cloud dependency, no accounts, no app store

## What You'll Get

1. **Quick capture**: Open Telegram → type task → done
2. **Management UI**: Browser bookmark on phone/laptop at `http://<your-mac-ip>:5002`
3. **Bot commands**: `/list`, `/done 5`, `/hi 3` (set high priority), `/status`

## File Structure

```
/Users/iqbalhabib/claude/Personal/tasks/
├── task_bot.py           # Telegram bot (from telegram_bot.py pattern)
├── task_db.py            # SQLite helpers (from intake_db.py pattern)
├── task_app.py           # Flask web UI on port 5002
├── templates/
│   └── tasks.html        # Mobile-first single-page UI
├── .env                  # Bot token + chat ID
└── requirements.txt
```

## Implementation Steps

### Step 1: Database (`task_db.py`)
- Single `tasks` table: id, text, priority (0/1/2), status (todo/done/dropped), created_at, completed_at
- Pattern from `/Users/iqbalhabib/claude/case/message-browser/intake_db.py`

### Step 2: Telegram Bot (`task_bot.py`)
- Any text message → new task
- Commands: `/list`, `/done N`, `/drop N`, `/hi N`, `/status`
- Restricted to your chat ID via `.env`
- Pattern from `/Users/iqbalhabib/claude/case/message-browser/telegram_bot.py`

### Step 3: Web UI (`task_app.py` + `tasks.html`)
- Flask on port 5002, mobile-first vanilla HTML/JS
- API: GET/PATCH/DELETE /api/tasks, POST /api/tasks
- Features: filter by status, sort by priority, checkbox to complete, quick-add bar
- No JS frameworks — same approach as existing message browser

### Step 4: Bot Setup (manual — you do this)
1. Message @BotFather on Telegram → `/newbot` → name it "Personal Tasks"
2. Copy token into `.env` as `PERSONAL_TASK_BOT_TOKEN`
3. Send any message to the bot
4. Visit `https://api.telegram.org/bot<TOKEN>/getUpdates` to get your chat ID
5. Set `TASK_CHAT_ID` in `.env`

### Step 5: launchd Agents
- `com.personal.task-bot.plist` — keeps bot running, restarts on failure
- `com.personal.task-app.plist` — keeps web UI running on port 5002
- Pattern from `/Users/iqbalhabib/Library/LaunchAgents/com.case.telegram-bot.plist`

## What We're NOT Building
- No auth (local network only)
- No categories/tags/due dates (can add later)
- No cloud sync (SQLite on your Mac)

## Verification
1. Send a message to bot in Telegram → confirm it appears in `tasks.db`
2. Open `http://localhost:5002` → confirm task shows up
3. Complete a task via web UI → confirm status changes
4. Run `/list` in Telegram → confirm it shows open tasks
5. Restart Mac → confirm both services auto-start via launchd
