# SACHIN DEV X FX — Free Fire Guild Glory Shop

Telegram bot **and** neon web shop in one process. Same accounts, wallets, orders, stock and prices on both.
Customers pick a region, enter their Guild ID, pay by QR or wallet, and the glory bot request is sent
automatically. Admins get a Telegram alert for every paid order and every problem — nobody has to approve anything.

> **Demo / dummy build.** Every key, payment, phone number and link in this repository is a placeholder.
> `PROVIDER_MODE=mock` and `PAYMENT_MODE=mock`, so no real money is charged and no real glory order is sent.
> See [Going live](#going-live-checklist) when you are ready to connect your own accounts.

---

## Table of contents

- [What you get](#what-you-get)
- [Quick start (local)](#quick-start-local)
- [How to use — Telegram bot](#how-to-use--telegram-bot)
- [How to use — web shop](#how-to-use--web-shop)
- [Admin panel](#admin-panel)
- [Run on Pterodactyl](#run-on-pterodactyl)
- [Configuration](#configuration)
- [Dummy links and values](#dummy-links-and-values)
- [Going live checklist](#going-live-checklist)
- [Safety rules](#safety-rules)
- [Tests](#tests)
- [Project structure](#project-structure)

---

## What you get

**Customer side**

- 🛒 **SHOP NOW** — 5 steps: Region → Guild ID → Check → Squads → Pay.
- 🔍 **Check SQD available** — live squad count per region, with refresh.
- ⚔️ **My Guilds** — live glory with `▲ +N`, the 8-hour timer, glory info.
- 🧾 **My Orders** — every order as `SDX-1001`, `SDX-1002`, … with guild name and 12-hour Nepal time.
- 💰 **Wallet** — add balance by QR (demo gateway), auto-detected and auto-credited.
- 👤 **Profile**, 📜 **Rules**, 🛟 **Support**.
- 🌐 **Web shop** — the same wallet and orders in the browser; log in with username/password or a
  one-time Telegram code.

**Admin side**

- 🔴 Live orders with live glory and controls.
- 📦 Stock & credits: website credits, tap a region to mark it out of stock, pause all orders.
- 💰 Prices: ± buttons or an exact amount.
- 🎟 Redeem code: redeems a coupon on the website account (adds credits).
- 💳 Payments · 👥 Users (wallet adjust, suspend) · 🏆 Groups (restart/stop/delete).
- 📢 Broadcast · 📊 Reports + CSV · ⚙️ Settings (service window, API status, provider notices,
  premium emoji IDs, audit).

---

## Quick start (local)

Requirements: **Python 3.11+**.

```bash
# 1. Install
python -m venv .venv
# Windows: .venv\Scripts\activate
# Linux/macOS:
source .venv/bin/activate
pip install -r requirements.txt

# 2. Create your settings
cp .env.example .env      # Windows: copy .env.example .env

# 3. Start
python main.py
```

`main.py` also installs missing packages into `.pylibs/` on first start, so it works on hosts that only
run one startup file. `app.py` is a thin alias for the same entry point.

At startup the bot logs in, checks the provider and payment gateway, upgrades the database if needed,
starts the web shop, and sends every admin a **🟢 Bot started** message with a self-check.

The database is created automatically at `glory.db`. Delete it to start with an empty demo.

---

## How to use — Telegram bot

1. Open your bot and send `/start`.
2. Set your **name** and **country** (no password).
3. The home menu shows your wallet and whether orders are open:

   | Button | What it does |
   |---|---|
   | 🛒 SHOP NOW | Start a glory order |
   | 🔍 Check SQD available | Live stock per region |
   | ⚔️ My Guilds | Glory, change and the 8-hour clock |
   | 🧾 My Orders | Order history |
   | 💰 Wallet | Add balance |
   | 👤 Profile | Name, country, photo |
   | 📜 Rules | Service rules |
   | 🛟 Support | Contact the admin |

4. **Ordering:** Region → Guild ID → Check → Squads → Pay.
   Pay by **QR** or **wallet**. After payment the order is placed automatically and the customer sees
   **BOT REQUEST HAVE BEEN SEND — PLEASE ACCEPT BOT REQUEST IN GUILD** with the details.
5. In **My Guilds**, use **Refresh** to see glory grow and check the timer.

Useful commands: `/start`, `/stock`, `/orders`, `/wallet`, `/profile`, `/support`,
`/weblogin` (website code), `/help`, `/cancel`, `/admin`.

If a payment arrives but the order cannot be placed, the money goes back to the wallet. If only some
squads can be started, only those are charged and the rest returns to the wallet.

---

## How to use — web shop

The shop runs inside the bot process on the panel port (`WEB_PORT`, or `SERVER_PORT`).

### Sign in

Three ways:

1. **Username + password** — classic login.
2. **Create account** — name, username and password (8+ characters).
3. **Telegram code** — send `/weblogin` to the bot, get an 8-digit one-time code, enter it on the site.
   You land in the **same account**: same wallet, same orders. Telegram users can then set a
   username/password on the **Profile** page.

### Buy glory (5 steps)

1. **Region** — stock is shown per region.
2. **Guild ID**.
3. **Check** — validates the Guild ID.
4. **Squads** — limited to available stock.
5. **Pay** — QR or wallet. The QR disappears by itself once paid, and the order is placed automatically.

### Add balance

**Wallet → Add balance** → choose a preset or custom amount → scan the QR.
The demo gateway confirms the payment and credits the wallet automatically.

### Everything else

**My Guilds** (live glory with `▲` change, 8-hour clock, glory info) · **My Orders** · **Stock** ·
**Rules** · **Support** · **Profile**.

---

## Admin panel

Only Telegram IDs listed in `ADMIN_IDS` / `SUPER_ADMIN_IDS` can use it. On the website, log in with
`/weblogin`.

- **Overview** — users, orders, sales, credits, price ±, pause/resume.
- **Stock & Credits** — credits, region on/off, redeem coupon.
- **Live orders** — status, glory, controls.
- **Users** — suspend, wallet adjust.
- **Rules** — edit what customers see.
- **Broadcast** — message all users.
- **Reports** — totals and CSV.

Admins are notified for every paid order and every problem.

### Service window

Closed from **Sunday 9 PM to Monday 12 AM (Nepal time)** by default. Customers see a notice.
Turn it off with `SERVICE_WINDOW=false` or from the admin settings.

### Colored buttons and premium emoji

Buttons use Telegram's `style` API (green = go, blue = info, red = stop) on current apps; older apps show
plain buttons. For premium emoji: admin → Settings → **Get premium emoji IDs**, then set
`PREMIUM_EMOJI_BUTTONS=true` and `PREMIUM_EMOJI_IDS=buy=ID,stock=ID,...`
(names: buy, stock, guild, orders, wallet, profile, support, refresh, cancel, pay, ok, home, fire, trophy).

---

## Run on Pterodactyl

1. Upload **the contents** of this folder so `main.py` sits next to `requirements.txt` (not in a sub-folder).
2. Set the startup file to **`main.py`** and the requirements file to **`requirements.txt`**.
   Missing packages install themselves into `.pylibs/`.
3. Put your settings in `.env` (copy from `.env.example`). Start the server — admins get
   **🟢 Bot started** plus a self-check.
4. **Web shop port:** add a second **allocation/port** to the server in Pterodactyl, then point your
   domain/proxy (Cloudflare, nginx, …) to `IP:port`. Use **HTTPS** in front of it.
5. Set `WEB_ENABLED=false` if you only want the Telegram bot.

**Updating from an older build:** upload over the old files. The database upgrades itself on start
(old wallets become NPR wallets; old users keep their name and get currency NPR). The old `glory.db`
is kept — back it up first anyway.

---

## Configuration

All settings come from `.env` (or panel variables). The defaults below are **dummy demo values**.

| Variable | Default | Notes |
|---|---|---|
| `BOT_TOKEN` | `123456789:AAHdummy...` | **Required.** Token from @BotFather |
| `ADMIN_IDS` / `SUPER_ADMIN_IDS` | `123456789` | Numeric Telegram IDs, comma-separated |
| `DATABASE_URL` | `sqlite+aiosqlite:///./glory.db` | SQLite by default |
| `DEFAULT_CURRENCY` | `NPR` | Nepali Rupee |
| `DEFAULT_PRICE_PER_SQUAD` | `375` | Admin can change it live |
| `MAX_SQUADS_PER_ORDER` | `10` | 1–10 |
| `PROVIDER_MODE` | `mock` | `mock` or `live` |
| `GLORY_API_BASE_URL` | `https://glory-api.example.com` | Dummy |
| `GLORY_API_KEY` / `GLORY_API_MASTER_KEY` | empty | Your provider keys |
| `PAYMENT_MODE` | `mock` | Must match `PROVIDER_MODE` |
| `APINEPAL_BASE_URL` | `https://payments.example.com` | Dummy |
| `APINEPAL_SECRET_KEY` / `APINEPAL_PUBLIC_KEY` | empty | Your gateway keys |
| `SUPPORT_USERNAME` | `example_support` | Dummy Telegram handle |
| `WEBSITE_URL` | `shop.example.com` | Dummy shop address |
| `PUBLIC_URL` | empty | Public shop URL used in `/weblogin` |
| `WEB_ENABLED` | `true` | Run the web shop inside the bot |
| `WEB_PORT` | empty | Empty = panel `SERVER_PORT`, else `8080` |
| `SERVICE_WINDOW` | `true` | Sunday 9 PM → Monday 12 AM (Nepal) |
| `BUTTON_COLORS` | `true` | Telegram colored buttons |

`PAYMENT_MODE` and `PROVIDER_MODE` must both be `mock` or both `live`. This prevents real money from
buying fake orders and fake payments from spending real credits.

---

## Dummy links and values

Every old production link and credential was replaced. These are placeholders — replace them with your own:

| Where | Dummy value |
|---|---|
| Support handle | `@example_support` (`https://t.me/example_support`) |
| Shop website | `https://shop.example.com` |
| Glory provider API | `https://glory-api.example.com` |
| Payment gateway API | `https://payments.example.com` |
| eSewa / Khalti demo number | `9800000000` / `Demo User` |
| Provider keys | `ak_dummy_provider_key`, `dummy_master_key` |
| Gateway keys | `sk_test_dummy_key`, `pk_test_dummy_key` |
| Bot token | `123456789:AAHdummyTokenReplaceMe_xxxxxxxxxxxxxxxxx` |
| Admin Telegram ID | `123456789` |

Order numbers use the **`SDX-`** prefix (`SDX-1001`). The website session cookie is `sdx_session`.

---

## Going live checklist

1. Copy `.env.example` to `.env`.
2. Set a real `BOT_TOKEN` and your numeric `ADMIN_IDS`.
3. Replace `SUPPORT_USERNAME`, `WEBSITE_URL` and `PUBLIC_URL` with your real addresses.
4. Put your own glory provider keys in `GLORY_API_*` and set `GLORY_API_BASE_URL`.
5. Put your own payment gateway keys in `APINEPAL_*` and set `APINEPAL_BASE_URL`.
6. When **both** accounts are ready, switch together:
   ```env
   PROVIDER_MODE=live
   GLORY_API_ENABLED=true
   PAYMENT_MODE=live
   ```
7. Start once and read the admin self-check (provider credits, payment connection, region mapping).
8. Back up `glory.db` before real orders.
9. Put HTTPS (Cloudflare, nginx, Caddy) in front of the web port.

> Keep provider and payment keys on the server. Never put them in the browser, a public repo, or a screenshot.

---

## Safety rules

- The same payment never creates two orders.
- A wrong amount creates no order and alerts the admin.
- The provider has no idempotency key, so a create call is never blindly re-sent. Unclear answers are
  checked against the group list (before/after); “busy” answers are retried; anything still unclear
  becomes **Needs a look** for the admin.
- Passwords are scrypt-hashed; sessions are random tokens (only the hash is stored).
- CSRF header + SameSite cookie; login rate-limit; security headers (CSP).
- Provider and payment keys never leave the server; the browser only talks to `/api/*`.

---

## Tests

```bash
pip install -r requirements-dev.txt
pytest
```

Tests run in mock mode and never touch real providers or payments.

---

## Project structure

```
main.py                  startup file (Pterodactyl)
app.py                   alias for main.py
requirements.txt         runtime packages
.env.example             dummy configuration template
glorybot/
  bot.py                 Telegram application, startup self-check
  config.py              settings from .env
  handlers/              user and admin Telegram UI
  services/              orders, payments, wallet, stock, glory, runner
  providers/             glory provider adapter + mock
  payments/              QR gateway adapter + mock
  web/                   Starlette API + single-page shop
tests/                   mock-mode test suite
```

---

## License / use

Demo project for learning and internal testing. Replace every dummy value before production.
Free Fire is a trademark of its owner; this project is not affiliated with or endorsed by them.
