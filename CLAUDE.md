# nashra

Automated Arabic news channel on Telegram. Successor to `AboUbida/xqueue` (X + Buffer),
which was paused in October 2026 because X's payout program excludes automated and
aggregated content. Telegram has no such rule, lets channels earn ad revenue share from
1,000 subscribers, and allows links, so sponsors and affiliates are the income plan.

## How it works
- `nashra.py` — one stdlib-only script, one run per invocation (see its docstring for env).
  Reads `sources.txt` (RSS feeds free; `@handle` lines use the X API and cost money), asks
  the model (Gemini via OpenAI-compatible endpoint, `gemini-3.8-flash`) for posts, draws a
  card with `cards.py`, sends with `sendPhoto` (multipart upload; no image hosting needed).
- Review mode (`REVIEW=1`): drafts go to `TELEGRAM_ADMIN` with inline buttons; each run
  first drains `getUpdates` and publishes approved drafts with `copyMessage`. A reply to a
  draft publishes the reply text instead. Pending drafts live in `state.json["pending"]`.
- Pacing: `MAX_PER_RUN`, `MAX_PER_DAY`, `GAP_MINUTES`, counted over history entries with
  status `draft`/`posted` that have no `draft_id` (i.e. created this way, not re-posted).
- `history.jsonl` statuses: draft, posted, skipped, expired. Dedupe by link and text.
- Workflow `.github/workflows/nashra.yml`: every 15 min + manual dispatch (dry run default).

## Rules
- Keep the script stdlib-only; Pillow optional and only for cards.
- Never put keys in files; GitHub secrets only. Test locally against a fake server
  (`TELEGRAM_API`/`LLM_URL` env override) rather than real keys.
- Commit messages end with `Co-Authored-By: Claude Fable 5.1 <noreply@anthropic.com>`.

## Local testing
```sh
.venv/bin/python nashra.py            # needs TELEGRAM_TOKEN, TELEGRAM_CHANNEL, LLM_KEY
DRY_RUN=1 .venv/bin/python nashra.py  # draws cards into cards/, posts nothing
```
Point `TELEGRAM_API=http://127.0.0.1:8765` and `LLM_URL=http://127.0.0.1:8765/llm` at a fake
server to exercise the posting and review paths without real accounts.
