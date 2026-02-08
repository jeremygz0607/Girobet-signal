# 🧭 Aviator Scraper + Signal Engine [Project ID: P-782]

Scrapes payout data from the Aviator game, saves rounds to MongoDB, detects patterns, and sends Telegram signals with win/loss tracking and automated daily recaps in Portuguese (Brazil).

---

## 📚 Table of Contents

[About](#-about)  
[Features](#-features)  
[Tech Stack](#-tech-stack)  
[Installation](#️-installation)  
[Usage](#-usage)  
[Configuration](#-configuration)  
[Screenshots](#-screenshots)  
[Database Schema](#-database-schema)  
[Contact](#-contact)  
[Acknowledgements](#-acknowledgements)

---

## 🧩 About

This project provides an automated pipeline for the Aviator casino game: it scrapes round multipliers with a headless browser, stores them in MongoDB, and runs a signal engine that triggers when the last few rounds are below a threshold. It solves the need for real-time pattern detection and Telegram notifications (signals, wins, gales, losses, and daily/weekly recaps) so users can follow a simple “bet on next round” strategy with optional gale (recovery) levels.

**Key goals:**

- Reliable scraping of payouts and one-place storage of rounds.
- Pattern-based signal creation (e.g. last 3 rounds &lt; 2x) with gale escalation and cooldown.
- Telegram bot messages in Portuguese (BR) for signals, results, and recaps.

---

## ✨ Features

- **Headless scraper** – Logs in, opens the game iframe, and scrapes payout multipliers; logs only when the list changes to keep log size small.
- **Log monitor** – Tails the log, detects new payouts, saves them to MongoDB, and triggers the signal engine for each new round.
- **Signal engine** – Pattern trigger (e.g. 3 rounds &lt; 2x), signal creation, gale escalation (up to 2), win/loss tracking, and `today_wins` / `today_losses` for recaps.
- **Telegram service** – All message templates (Portuguese/Brazil): pattern monitoring, signal confirmed, win/recovery/loss, streak celebrations (5, 10, 15…), daily opener/close, mid-day and end-of-day recaps, weekly recap.
- **Scheduler** – BRT timezone: daily opener (08:00), mid-day recap (14:00), hourly scoreboard, end-of-day recap (22:30), daily close (23:00), weekly recap (Sunday 21:00).

---

## 🧠 Tech Stack

| Category   | Technologies |
|-----------|--------------|
| **Languages** | Python 3 |
| **Scraping**  | SeleniumBase, BeautifulSoup4 |
| **Database**  | MongoDB (PyMongo) |
| **Messaging** | Telegram Bot API (requests) |
| **Scheduling** | APScheduler (pytz for BRT) |
| **Tools**      | python-dotenv, log file tailing |

---

## ⚙️ Installation

```bash
# Clone the repository
git clone https://github.com/jeremygz0607/Girobet-signal.git

# Navigate to the project directory
cd Girobet-signal

# Create virtual environment (recommended)
python -m venv venv
source venv/bin/activate   # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

---

## 🚀 Usage

Run all scripts from the **project root** so `log.log` and `aviator.pid` are in the right place.

**1. Scraper (required)**

```bash
python aviator.py
```

**2. Log monitor (required)** – Saves payouts to MongoDB and runs the signal engine. Also starts the Telegram scheduler.

```bash
python log_monitor.py
```

For production deployment (e.g. Vultr), see **[DEPLOYMENT.md](DEPLOYMENT.md)** for systemd services and server setup.

---

## 🧾 Configuration

Create a `.env` file in the project root (do not commit real credentials):

| Variable | Required | Description |
|----------|----------|-------------|
| `AVIATOR_USERNAME` | Yes | Casino login username |
| `AVIATOR_PASSWORD` | Yes | Casino login password |
| `AVIATOR_GAME_URL` | No | Game URL (default in config) |
| `AVIATOR_LOGIN_URL` | No | Login URL (default in config) |
| `MONGODB_URI` | Yes (for DB) | MongoDB connection string |
| `MONGODB_DATABASE` | No | Database name (default: `casino`) |
| `MONGODB_COLLECTION` | No | Collection name (default: `rounds`) |
| `TELEGRAM_BOT_TOKEN` | Yes (for Telegram) | Bot token from @BotFather |
| `TELEGRAM_CHANNEL_ID` | Yes (for Telegram) | Channel ID (e.g. `@channel` or numeric) |
| `AFFILIATE_LINK` | No | Link for “JOGAR AGORA” buttons |

**config.py** – Signal engine and paths:

- `SEQUENCE_LENGTH` = 3 (consecutive rounds &lt; threshold to trigger)
- `THRESHOLD` = 2.0 (multiplier)
- `TARGET_CASHOUT` = 1.80, `MAX_GALE` = 2, `COOLDOWN_ROUNDS` = 3
- Collections: `rounds`, `signals`, `daily_stats`, `engine_state`

---

## 🖼 Screenshots
TG_channel.png

---

## 📜 Database Schema

**`rounds`** – `_id` (int), `multiplier` (float), `timestamp`, `created_at`  
**`signals`** – `_id`, `trigger_round_id`, `target`, `status` (active/won/gale1/gale2/lost), `result_round_id`, `result_multiplier`, `gale_depth`, `created_at`, `resolved_at`  
**`daily_stats`** – `_id` (date YYYY-MM-DD), `wins`, `losses`, `today_wins`, `today_losses`, `signals_sent`, `updated_at`  
**`engine_state`** – `_id` "state", `cooldown_until_round_id`

---

## 📬 Contact

**Author:** 2-1-SUJW 
**Email:** jeremygz0607@gmail.com  
**GitHub:** @jeremygz0607  

---

## 🌟 Acknowledgements

- SeleniumBase and BeautifulSoup for scraping.
- Telegram Bot API for notifications.
- MongoDB for round and signal storage.
