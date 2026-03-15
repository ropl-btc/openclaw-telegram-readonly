# telegram-readonly

Read-only Telegram access for your **personal account** via **Telethon/MTProto**, packaged as a small CLI wrapper plus an OpenClaw-compatible `SKILL.md`.

This is for people who want their OpenClaw assistant to **read** Telegram chats from their own account without relying on the Telegram Bot API.

## What this is

- a small **read-only CLI wrapper** built on Telethon
- a matching **OpenClaw skill** (`SKILL.md`)
- opinionated defaults for inbox-style workflows

## What this is not

- not a Telegram bot
- not a write-capable Telegram automation client
- not a generic full Telegram CLI

The wrapper intentionally exposes only read operations.

## Features

- authenticate a real Telegram user account via MTProto
- list dialogs/chats
- token-based dialog search across name / username / title
- read recent messages from a chat
- search message text globally or inside one chat
- list unread chats
- list unread DMs only
- exclude muted and archived chats by default in unread views
- expose useful dialog metadata like `is_user`, `is_group`, `is_channel`, `is_bot`, `muted`, and `archived`

## Commands

- `help`
- `auth`
- `dialogs`
- `messages`
- `search`
- `unread-dialogs`
- `unread-dms`

Run built-in help:

```bash
python3 scripts/telegram_readonly.py help
```

## Requirements

- Python 3.10+
- a Telegram account
- Telegram `api_id` and `api_hash` from `https://my.telegram.org`
- Telethon

## Install

### Option 1: simple local venv

```bash
git clone https://github.com/YOUR-USER/telegram-readonly.git
cd telegram-readonly
python3 -m venv .venv
. .venv/bin/activate
pip install --upgrade pip telethon
```

Or with `requirements.txt`:

```bash
pip install -r requirements.txt
```

## Get Telegram API credentials

At `https://my.telegram.org`:

1. log in with your Telegram phone number
2. open **API Development Tools**
3. create an application
4. save your `api_id` and `api_hash`

## Authenticate once

You can pass credentials via env vars:

```bash
export TELEGRAM_API_ID='12345678'
export TELEGRAM_API_HASH='your_api_hash'
python3 scripts/telegram_readonly.py auth
```

The script will ask for:
- phone number
- login code
- Telegram two-step verification password, if enabled

It saves local config here:

```text
~/.config/telegram-readonly/config.json
```

That file contains a Telethon `StringSession`, which is **high-privilege**. Treat it like a password.

## Usage examples

### Show help

```bash
python3 scripts/telegram_readonly.py help
```

### List chats

```bash
python3 scripts/telegram_readonly.py dialogs --limit 50
```

### Search chats by query

```bash
python3 scripts/telegram_readonly.py dialogs --limit 200 --query 'petros skynet'
```

`dialogs --query` uses token-based matching, so queries like `petros skynet` work even when the exact full string is not present as one substring.

### Read recent messages from a chat

```bash
python3 scripts/telegram_readonly.py messages --chat @suuuupaman --limit 20 --reverse
```

### Search message text globally

```bash
python3 scripts/telegram_readonly.py search 'btc_txid_here' --limit 20
```

### Search message text inside one chat

```bash
python3 scripts/telegram_readonly.py search 'deadline' --chat @suuuupaman --limit 20
```

### List unread chats

By default, this excludes **muted** and **archived** chats:

```bash
python3 scripts/telegram_readonly.py unread-dialogs --limit 10
```

Include muted and archived when needed:

```bash
python3 scripts/telegram_readonly.py unread-dialogs --limit 10 --include-muted --include-archived
```

### List unread DMs only

```bash
python3 scripts/telegram_readonly.py unread-dms --limit 10
```

## OpenClaw usage

This repo includes a `SKILL.md` so it can be used as an OpenClaw skill/reusable workflow reference.

Relevant files:

- `SKILL.md`
- `scripts/telegram_readonly.py`
- `references/setup-and-safety.md`

## Safety notes

This wrapper is intentionally **read-only**.

It does **not** expose commands for:
- sending messages
- editing messages
- deleting messages
- marking chats read
- running an auto-reply daemon

Important nuance: the underlying Telethon session still has broad account access because it is a real Telegram login. The safety comes from keeping the wrapper surface area small and read-only.

## Unread-state caveat

The wrapper never calls explicit read acknowledgements.

That should usually avoid marking messages as read, but you should still test this with a sacrificial chat first because Telegram state can be subtle.

## Why not Telegram Bot API?

Because Telegram Bot API is for bots, not your actual personal account inbox.

If you want OpenClaw to read your real Telegram account like a normal client, you want **MTProto**, which is why this repo uses **Telethon**.

## License

MIT
