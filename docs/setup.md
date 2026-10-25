# Setup

The [README](../README.md) covers installing, every configuration key, the
constants and the two ways of running the bot. This page adds what it leaves out.

## Settings the bot needs that the README doesn't list as required

| Setting | Why |
|---|---|
| `TEST_GUILD_ID` | `/ping` is registered only in this server, and the bot reads it at startup. Without it, startup fails with a `KeyError`. Set it to your own test server. |
| `REFRESH_INTERVAL` | The refresh loops are built from it when `bot.py` is imported, as `int(env.get("REFRESH_INTERVAL"), 10)`. Without it, startup fails with a `TypeError`. The README's "every 10 minutes if not specified" isn't what the code does. |

## The `.env` file

- One `NAME=value` per line. Lines starting with `#` are comments.
- **No blank lines:** `utils.load_env` splits every line on `=`, and a blank one
  stops the bot with a `ValueError`.
- Any key the bot doesn't use is read and ignored.

## The first run

On its first start, the bot posts its messages in `SCHEDULE_EDIT_CHANNEL`:

- a Mini-Prix thread and a Classic Mini-Prix thread, each with its track
  selection;
- the main schedule message.

It saves their ids in `.msg_struct`, in the working directory. On every later
start it finds them there and edits the same messages instead of posting new ones.
`.msg_struct` is ignored by git.

To start over in a channel (a new channel, or messages deleted by hand), delete
`.msg_struct` and restart the bot.

The schedule message starts refreshing at the next multiple of `REFRESH_INTERVAL`
minutes, then every `REFRESH_INTERVAL` minutes. Unless `TICKER_OVERRIDE` is set,
the bot's status shows the current or next Grand Prix at each refresh.

## Picking the schedule profile

| Started with | Profile used | Switches automatically |
|---|---|---|
| `--profile NAME` | `NAME`. An unknown name stops the bot. | no |
| no flag, with `EVENT_SCHEDULE_FILE` present | the row of that file active today; an unknown name falls back to `default`, with a warning | yes, checked at 23:54 UTC every night |
| no flag, no file | `default` | no |

Every profile named in `EVENT_SCHEDULE_FILE` is loaded at startup, so a broken
profile shows up in the log straight away rather than on the night it becomes
active.

## Run it

From the repository root, with `.env` there:

```bash
uv sync --dev
uv run python -m pengbot99.bot
uv run python -m pengbot99.bot --profile queen
```

## Tests

```bash
uv run python -m unittest discover -s tests
```

The tests use `unittest` and CSV fixtures in `tests/fixtures/`. They cover the
schedule computations, the Mini-Prix and 99-race track selections, `/explain`,
the schedule controller and the utilities. Nothing in them talks to Discord.
