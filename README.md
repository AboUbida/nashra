# nashra (نشرة)

An automated Arabic news channel on Telegram. A single stdlib-only Python script reads
RSS feeds (and, optionally, X accounts), rewrites the news in Arabic with an LLM, draws a
headline card, and posts to the channel through the Telegram Bot API. It runs on GitHub
Actions every 15 minutes.

Optional **review mode** sends each post to your private chat with the bot first, with
**نشر ✅** / **تجاهل ❌** buttons. Reply to a draft with corrected text to publish that
instead.

## Setup (about 15 minutes)

1. **Create the channel** in Telegram (public, with a username, is best: posts get
   shareable links and the channel can be found in search).
2. **Create the bot**: message [@BotFather](https://t.me/BotFather), send `/newbot`,
   and keep the token.
3. **Make the bot an administrator of the channel** with the *Post Messages* right
   (channel → Administrators → Add Admin → search the bot's username).
4. **Fork or push this repo**, then add in *Settings → Secrets and variables → Actions*:

   | Kind     | Name               | Value |
   |----------|--------------------|-------|
   | Secret   | `TELEGRAM_TOKEN`   | the BotFather token |
   | Secret   | `LLM_KEY`          | Gemini API key (or any OpenAI-compatible key; set `LLM_URL`/`LLM_MODEL` vars then) |
   | Variable | `TELEGRAM_CHANNEL` | `@yourchannel` (or the numeric `-100…` id for a private channel) |
   | Variable | `REVIEW`           | `1` to approve posts before they go out (recommended at first) |
   | Variable | `TELEGRAM_ADMIN`   | your chat id, needed with `REVIEW=1`: send the bot `/start`, then run the `setup` command below and copy the id it prints |
   | Secret   | `X_BEARER_TOKEN`   | only if `sources.txt` lists `@accounts` (X API, pay-per-use) |

5. **Run it once by hand**: *Actions → nashra → Run workflow* with *dry run* ticked.
   The log shows the posts it would send. Untick dry run for a real run.

The `setup` command checks everything and prints the chat ids the bot has seen:

```sh
TELEGRAM_TOKEN=... TELEGRAM_CHANNEL=@yourchannel python3 nashra.py setup
```

## Tuning (repository variables)

| Variable | Default | Meaning |
|---|---|---|
| `MAX_PER_RUN` | 2 | posts per run |
| `MAX_PER_DAY` | 15 | posts per 24 hours |
| `GAP_MINUTES` | 15 | minimum minutes between posts |
| `CREDIT` | 1 | `0` removes the "المصدر" link under each post |
| `FOOTER` | | HTML added under every post, e.g. `📣 <a href="https://t.me/yourchannel">نشرة</a>` or a sponsor line |
| `OWN_CARDS` | 1 | `0` uses the source's photo first and the card only when there is none |
| `CARD_BRAND` | @channel | text in the card's footer |
| `MAX_AGE_DAYS` | 2 | ignore items older than this |
| `REVIEW_HOURS` | 24 | drafts nobody approved expire after this |

Edit `brief.txt` to change the voice and rules, and `sources.txt` to change what is read.

## If GitHub's schedule does not fire

GitHub's cron is best-effort. If runs stop, create a timer at [cron-job.org](https://cron-job.org)
every 15 minutes that POSTs to

```
https://api.github.com/repos/<owner>/nashra/actions/workflows/nashra.yml/dispatches
```

with headers `Authorization: Bearer <fine-grained PAT with Actions: read/write>`,
`Accept: application/vnd.github+json`, and body `{"ref":"main","inputs":{"dry_run":"false"}}`.

## Files

- `nashra.py` — the whole pipeline. `cards.py` draws the cards (needs Pillow with libraqm).
- `brief.txt` — voice and rules given to the model. `sources.txt` — what to read.
- `history.jsonl` — every draft and post (dedupes across runs). `state.json` — last X post ids,
  Telegram update offset, drafts waiting for review. Both are committed by the workflow.
