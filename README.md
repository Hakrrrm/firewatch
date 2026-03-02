# Fire Detection Project

This repo has two parts:

- `model_training/` → train/fine-tune YOLO for `controlled_fire`, `fire`, `smoke`
- `classification/` → run-time video risk classification

## Install

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Classification quick start (single-window)

Each command run analyzes **one selected time window** and outputs **one JSON + one JPG**.

```bash
python classification/analyze_video.py \
  --video /path/to/video.mp4 \
  --weights classification_model.pt \
  --start-seconds 0 \
  --analyze-seconds 10 \
  --sample-fps 2 \
  --conf 0.25 \
  --results-dir results \
  --run-label incident_cam01_0000_0010 \
  --camera-id cam_01 \
  --location-type warehouse
```

Outputs:

- `results/<run-label>/metrics.json`
- `results/<run-label>/top_fire.jpg`

For full flag/docs, see `classification/README.md`.

## OpenAI setup (optional)

```bash
cp .env.example .env
# fallback if dotfiles are hidden: cp env.example .env
```

Set key in `.env`:

```env
OPENAI_API_KEY=your_openai_api_key_here
TELEGRAM_BOT_TOKEN=8610841747:AAHEbtxFdZ28VTlN0t5VoTZn5vxUXS9j1VU
TELEGRAM_CHAT_ID=1334522798
TELEGRAM_USERNAME=your_telegram_username
```

If no key is provided, classifier still runs in local-only mode.
Use `--demo-mode` to force local-only.

## Training docs

All model training/fine-tuning documentation remains in:

- `model_training/README.md`

## Django Dashboard Integration (Video Mode)

Current integration in this workspace runs classification from `video.mp4`/`video.MP4` (camera stream disabled for now).

### Main URLs

- Dashboard: `http://127.0.0.1:8000/`
- Event detail: `http://127.0.0.1:8000/events/<event_id>/`
- Event footage endpoint: `http://127.0.0.1:8000/events/<event_id>/footage/`

### Run one classification window via API

`POST /api/events/<event_id>/classification/run/`

Body example:

```json
{
  "start_seconds": 0,
  "analyze_seconds": 10,
  "sample_fps": 2,
  "conf": 0.25
}
```

This executes `classification/analyze_video.py` using:
- `classification_model.pt`
- `video.mp4` or `video.MP4`

### Notification behavior

- First `Hazard` detection creates a distinct hazard notification card with footage/details.
- First `Emergency` detection creates an urgent emergency notification card with stronger animation and a 20s countdown.
- Escalation to authorities now sends Telegram notifications containing:
  - Detection details text summary.
  - 5-second video clip from the detected interval.
  - Grid map image with A* shortest path from entrance to fire location.
- Telegram send is triggered when:
  - User clicks **Escalate To Authorities** (general cases).
  - Emergency countdown reaches zero (auto-timeout case).

### Telegram receiver setup (simulate authority notification)

If a user wants to receive the Telegram escalation message (simulating notification to authorities), they must do this once in Telegram:

1. Open chat with `FIREWATCH_BOTBOT`.
2. Send `/start`.
3. Send any random message (for example: `hello`).

Without this first interaction, Telegram may block the bot from delivering the escalation notification to that user/chat.

### Telegram username -> chat id resolution

To receive Telegram alerts in your own chat:

1. Message the bot `FIREWATCH_BOTBOT`.
2. Send `/start`.
3. Send a test message to the bot.
4. Set your username in code/config using `TELEGRAM_USERNAME` (for example in `.env`).

Firewatch then resolves your chat id using bot updates (same logic as below, with your username substituted):

```python
import requests

# Replace with your bot token
BOT_TOKEN = "8610841747:AAHEbtxFdZ28VTlN0t5VoTZn5vxUXS9j1VU"
URL = f"https://api.telegram.org/bot{BOT_TOKEN}/getUpdates"

response = requests.get(URL)
data = response.json()

telegram_username = "your_telegram_username"
chat_id = None

# Loop through updates to find the user with this username
for update in data.get("result", []):
    message = update.get("message", {})
    chat = message.get("chat", {})
    if chat.get("username") == telegram_username:
        chat_id = chat.get("id")
        break

if chat_id:
    print(f"Chat ID for {telegram_username}: {chat_id}")
else:
    print(f"User '{telegram_username}' not found in updates.")
```

### Emergency decision API

`POST /api/events/<event_id>/emergency/decision/`

Body:
- `{\"action\":\"call_now\"}` (manual call placeholder)
- `{\"action\":\"cancel\"}` (do not call)
- `{\"action\":\"auto_timeout\"}` (legacy placeholder action)
