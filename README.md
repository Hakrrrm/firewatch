# Fire Detection Project

This repo has two parts:

- `model_training/` -> train/fine-tune YOLO for `controlled_fire`, `fire`, `smoke`
- `classification/` -> runtime video risk classification

## Python version

Use Python `3.10` to `3.12`.

Use a specific interpreter when creating the virtual environment:

- macOS/Linux (example 3.11): `python3.11 -m venv .venv`
- Windows PowerShell (example 3.11): `py -3.11 -m venv .venv`

## Install

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Environment setup

```bash
cp .env.example .env
```

Edit `.env`:

```env
OPENAI_API_KEY=your_openai_api_key_here
TELEGRAM_BOT_TOKEN=8610841747:AAHEbtxFdZ28VTlN0t5VoTZn5vxUXS9j1VU
TELEGRAM_CHAT_ID=optional_fallback_chat_id_here
TELEGRAM_USERNAME=your_telegram_username_without_at
```

Notes:

- Replace `OPENAI_API_KEY` with your own key if you want OpenAI context classification.
- Replace `TELEGRAM_CHAT_ID` only if you want a specific fallback chat.
- Replace `TELEGRAM_USERNAME` with your own Telegram username (without `@`) if you want escalation messages.
- If `OPENAI_API_KEY` is not set, the classifier runs in local-only mode.
- `TELEGRAM_USERNAME` is where escalation notifications are routed by username resolution.
- Telegram username configuration is read in `firewatch_project/settings.py` from `TELEGRAM_USERNAME`.

## Classification quick start (single-window)

Each run analyzes one selected time window and outputs one JSON + one JPG:

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
  --location-type warehouse \
  --device=cpu
```

Outputs:

- `results/<run-label>/metrics.json`
- `results/<run-label>/top_fire.jpg`

For full flags/docs, see `classification/README.md`.

## Dashboard integration (video mode)

The dashboard currently classifies from `video.mp4`/`video.MP4`.

Main URLs:

- Dashboard: `http://127.0.0.1:8000/`
- Event detail: `http://127.0.0.1:8000/events/<event_id>/`
- Event footage: `http://127.0.0.1:8000/events/<event_id>/footage/`

Notification behavior:

- First `Hazard` detection creates a hazard notification card.
- First `Emergency` detection creates an urgent card with a 20-second countdown.
- Clicking **Escalate To Authorities** (or emergency timeout) sends Telegram:
  - detection summary text
  - 5-second interval clip
  - A* route map image

Telegram receiver setup (one-time):

1. Open chat with `FIREWATCH_BOTBOT`.
2. Send `/start`.
3. Send any message (for example `hello`).
4. Set `TELEGRAM_USERNAME` in `.env` (without `@`).

## Testbench guide

For a full no-prior-knowledge tester workflow (Windows + macOS), see `testbench/README.md`.
