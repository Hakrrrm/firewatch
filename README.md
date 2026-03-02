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
- First `Emergency` detection creates an urgent emergency notification card with stronger animation and a 30s countdown.
- If no decision is made in 30s, an auto-timeout placeholder is triggered:
  - Status set to `auto_call_triggered_module_missing`
  - Actual authority call remains intentionally empty until teammate module is integrated.

### Emergency decision API

`POST /api/events/<event_id>/emergency/decision/`

Body:
- `{\"action\":\"call_now\"}` (manual call placeholder)
- `{\"action\":\"cancel\"}` (do not call)
- `{\"action\":\"auto_timeout\"}` (used by countdown auto-trigger)
