# Firewatch Testbench Guide

This guide is for first-time testers with **no prior project knowledge**.

## 1) What you are testing

Firewatch is a dashboard that analyzes video windows for fire/smoke risk and shows alerts.

Core test flow:
1. Start the dashboard.
2. Click **Start Scenario**.
3. Wait for processing (can take up to 1 minute on slower devices).
4. Open the hazard/emergency footage view.
5. Optionally escalate to authorities (Telegram simulation).

---

## 2) System requirements

- **Python 3.10** (recommended for this repo)
- macOS, Windows, or Linux
- Internet access (for package installs + optional Telegram/OpenAI)

> Why Python 3.10? The project dependencies and current runtime are known to work with Python 3.10.

---

## 3) Clone and open project folder

```bash
git clone <your-repo-url>
cd firewatch
```

---

## 4) Create and activate virtual environment

### macOS/Linux (bash/zsh)

```bash
python3.10 -m venv .venv
source .venv/bin/activate
python --version
```

### Windows PowerShell

```powershell
py -3.10 -m venv .venv
.\.venv\Scripts\Activate.ps1
python --version
```

If activation is blocked in PowerShell, run once:

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

---

## 5) Install dependencies

```bash
pip install -r requirements.txt
```

---

## 6) Configure environment file

Copy the template:

### macOS/Linux

```bash
cp .env.example .env
```

### Windows PowerShell

```powershell
copy .env.example .env
```

Edit `.env` and set values:

```env
OPENAI_API_KEY=your_openai_api_key_here
TELEGRAM_BOT_TOKEN=8610841747:AAHEbtxFdZ28VTlN0t5VoTZn5vxUXS9j1VU
TELEGRAM_CHAT_ID=1334522798
TELEGRAM_USERNAME=your_telegram_username
```

### OpenAI key note

- You **can continue without an OpenAI key**.
- Without a key, use the project's local/demo path (no extra contextual OpenAI classification).

---

## 7) Telegram setup (authority simulation)

To receive simulated escalation messages:

1. Open Telegram and message `FIREWATCH_BOTBOT`.
2. Send `/start`.
3. Send a test message (e.g., `hello`).
4. Put your Telegram username in `.env` as `TELEGRAM_USERNAME=...`.

### Optional: verify your chat id manually

Run this script after sending a message to the bot:

```python
import requests

BOT_TOKEN = "8610841747:AAHEbtxFdZ28VTlN0t5VoTZn5vxUXS9j1VU"
URL = f"https://api.telegram.org/bot{BOT_TOKEN}/getUpdates"

response = requests.get(URL)
data = response.json()

telegram_username = "your_telegram_username"
chat_id = None

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

---

## 8) Start Firewatch

Apply migrations and run server:

```bash
python manage.py migrate
python manage.py runserver
```

Open dashboard:

- http://127.0.0.1:8000/

---

## 9) Dashboard test procedure (what to click and expect)

1. On Home Dashboard, click **Start Scenario**.
2. Wait while analysis runs.
   - Processing time is **device dependent**.
   - On slower devices, allow **up to 1 minute**.
   - This is expected because default runtime uses **CPU** for cross-device compatibility.
   - In real deployments, hardware-specific acceleration (GPU/inference optimizations) can be much faster.
3. Watch for a new alert card/bar (usually Hazard/Emergency style depending on classification output).
4. Click **View Footage** / open footage details from the alert card.
5. Confirm in footage page:
   - risk display,
   - camera and zone details,
   - escalation controls.
6. Click **Escalate To Authorities** to trigger Telegram notification workflow.

Expected Telegram payload includes:
- text summary,
- 5-second clip,
- map image with A* path to fire location.

---

## 10) Quick troubleshooting

- **`ModuleNotFoundError: django`**
  - Ensure venv is activated and run `pip install -r requirements.txt`.
- **No Telegram messages**
  - Ensure `/start` + test message were sent to bot.
  - Ensure `TELEGRAM_USERNAME` in `.env` is correct.
- **Slow processing**
  - Expected on CPU-only test machines.
- **No OpenAI key**
  - Continue in demo/local-only behavior.

---

## 11) Recommended smoke test checklist

- [ ] App starts at `http://127.0.0.1:8000/`
- [ ] Start Scenario works
- [ ] Alert appears
- [ ] Footage page opens
- [ ] Escalation endpoint responds
- [ ] Telegram message/clip/map received (if configured)

