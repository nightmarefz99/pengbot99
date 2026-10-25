# pengbot99 documentation

pengbot99 is a Discord bot, and a Python library, that knows F-Zero 99's own
schedule: which Grand Prix is on and what comes next, the track selection of each
Mini-Prix and 99 race, and how the rotation changes during in-game events.

The [README](../README.md) covers installing, configuring and running the bot,
the schedule constants, and the event profiles. These pages cover the rest:

| Page | What's in it |
|---|---|
| [setup.md](setup.md) | What the README doesn't say about running it: required settings, the first run, the message file, tests |
| [structure.md](structure.md) | What lives where, in the code and in `config/` |
| [internals.md](internals.md) | How the schedule is computed, how profiles switch, what the bot posts and when |
| [commands.md](commands.md) | Every slash command: handler, arguments, who may run it, what it reads |

## Design records

Two earlier documents record how two features were designed. Both features are
built. Read them for the reasoning; for how things work today, see
[internals.md](internals.md).

- [event-profiles.md](event-profiles.md): the move from one branch per in-game
  event to `config/event_<name>/` folders that hold only what differs from
  `event_default`. Since it was written, `explain.dat` and `misa.csv`, listed
  there as out of scope, are no longer part of the bot.
- [schedule-controller.md](schedule-controller.md): switching schedule profiles
  without a restart. Two of its open items have since been settled:
  - `unload()` refuses to unload the active profile;
  - switching is triggered by `EVENT_SCHEDULE_FILE`, which the bot checks every
    night at 23:54 UTC.
