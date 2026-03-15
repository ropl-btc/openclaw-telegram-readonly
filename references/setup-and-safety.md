# Telegram Readonly — Setup and safety

## What this skill is for

Use this skill to read Telegram chats from Robin's personal account via Telethon/MTProto in a controlled, read-only way.

This is not a Telegram bot skill.
It is for local access to a real user account.

## Safety model

The wrapper intentionally exposes only:
- `auth`
- `dialogs`
- `messages`
- `search`
- `unread-dialogs`
- `unread-dms`
- `help`

It does not expose:
- send
- edit
- delete
- mark-read calls
- background auto-reply logic

Important: the underlying Telethon session still has high privilege because it is a real Telegram login. The safety comes from the wrapper surface area, not from Telegram granting reduced permissions.

## Files and locations

- Skill root: `/root/.agents/skills/telegram-readonly`
- Wrapper script: `/root/.agents/skills/telegram-readonly/scripts/telegram_readonly.py`
- Local config: `~/.config/telegram-readonly/config.json`

## Prerequisites

1. Telegram API credentials from `https://my.telegram.org`
2. Telethon installed in a dedicated environment
3. One interactive login to create a session string

## Install Telethon

Recommended:

```bash
python3 -m venv /root/.venvs/telegram-readonly
/root/.venvs/telegram-readonly/bin/pip install --upgrade pip telethon
```

Then run commands with:

```bash
/root/.venvs/telegram-readonly/bin/python /root/.agents/skills/telegram-readonly/scripts/telegram_readonly.py ...
```

## Telegram API credentials

At `https://my.telegram.org`:
1. Log in with the Telegram account phone number.
2. Open API development tools.
3. Create an application.
4. Save `api_id` and `api_hash`.

## First auth flow

Use either env vars or CLI args.

Env example:

```bash
export TELEGRAM_API_ID='12345678'
export TELEGRAM_API_HASH='your_api_hash'
/root/.venvs/telegram-readonly/bin/python /root/.agents/skills/telegram-readonly/scripts/telegram_readonly.py auth
```

The script will prompt for:
- phone number
- login code
- 2FA password if enabled

It saves the resulting session string to `~/.config/telegram-readonly/config.json`.
Protect that file like a password.

## Read-only usage

Show built-in help:

```bash
/root/.venvs/telegram-readonly/bin/python /root/.agents/skills/telegram-readonly/scripts/telegram_readonly.py help
```

List chats:

`dialogs --query` uses token-based matching across `name`, `username`, and `title`.

```bash
/root/.venvs/telegram-readonly/bin/python /root/.agents/skills/telegram-readonly/scripts/telegram_readonly.py dialogs --limit 50
```

Read one chat:

```bash
/root/.venvs/telegram-readonly/bin/python /root/.agents/skills/telegram-readonly/scripts/telegram_readonly.py messages --chat '@username' --limit 50 --reverse
```

Search globally:

```bash
/root/.venvs/telegram-readonly/bin/python /root/.agents/skills/telegram-readonly/scripts/telegram_readonly.py search 'invoice' --limit 50
```

Search in one chat:

```bash
/root/.venvs/telegram-readonly/bin/python /root/.agents/skills/telegram-readonly/scripts/telegram_readonly.py search 'deadline' --chat '@username' --limit 50
```

List recent unread chats, excluding muted + archived by default:

```bash
/root/.venvs/telegram-readonly/bin/python /root/.agents/skills/telegram-readonly/scripts/telegram_readonly.py unread-dialogs --limit 10
```

List recent unread DMs only, excluding muted + archived by default:

```bash
/root/.venvs/telegram-readonly/bin/python /root/.agents/skills/telegram-readonly/scripts/telegram_readonly.py unread-dms --limit 10
```

Include muted and archived when needed:

```bash
/root/.venvs/telegram-readonly/bin/python /root/.agents/skills/telegram-readonly/scripts/telegram_readonly.py unread-dialogs --limit 10 --include-muted --include-archived
```

## Unread behavior

Goal: avoid changing unread state.

This wrapper never calls explicit read acknowledgements.
That should usually avoid marking messages as read, but this must be verified with a live test because Telegram state can be subtle.

Before broad use:
1. pick a sacrificial chat
2. confirm unread badge/state before read
3. fetch messages with the wrapper
4. verify whether unread state changed in official Telegram clients

If unread state changes unexpectedly, stop and adjust workflow before wider rollout.

## Operational guidance

- Prefer narrow reads over broad scraping.
- Start with specific chats or direct need.
- Do not add write actions to this wrapper unless Robin explicitly asks.
- If later automation is needed, create a separate write-capable tool instead of weakening this one.
