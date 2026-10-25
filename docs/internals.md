# Internals

How pengbot99 works out the schedule, and what the bot does with it. The design
reasoning behind profiles and switching is in the two design records
([event-profiles.md](event-profiles.md),
[schedule-controller.md](schedule-controller.md)); this page describes the code as
it is.

## The schedule is computed, not looked up

F-Zero 99 runs its events on fixed, repeating cycles. pengbot99 stores one cycle
per slot as a CSV, plus a moment in time at which a cycle began. From those two it
can say what's on at any instant, past or future, without a list of dates.

### A cycle file

```text
0,mace,knight,mqueen,king,...
10,teambattle,protracks,classic
30,classicprix,miniprix,miniprix
40,teambattle,protracks,classic
60,next
```

- The first column is the **minute within the cycle** at which the row's event
  starts. The `next` row closes the cycle, and its minute is the cycle's length
  (60 here). `schedule.load_schedule` refuses a file that doesn't end with `next`.
  Its docstring calls the first column a "duration"; it's a start minute, as
  `TimeTable` reads it.
- The other columns are a **rotation**. Each time the cycle reaches that row, the
  next name in the list is the event: here, the Grand Prix at minute 0 goes
  through Mirror Ace, Knight, Mirror Queen and so on, one per cycle.

### From a time to an event

- `schedule.origin` and `schedule.glitch_origin` are the moments, in UTC, at
  which a cycle started at its first row.
- A manager counts how many cycles have passed since then, and the minute within
  the current one. That's the `CycleInfo`.
- The `TimeTable` finds the row active at that minute. The `CycleInfo` says how
  many times each rotation has already advanced, so the right name is picked from
  each rotation list.

There are two managers, one per in-game slot:

- **Slot 1** (`Slot1ScheduleManager`): the 99-race slot, with its Glitch races.
- **Slot 2** (`Slot2ScheduleManager`): Grand Prix and special events, with one
  cycle for Monday to Friday (`slot2_schedule.csv`) and another for Saturday and
  Sunday (`slot2_schedule_weekend.csv`).

On top of these:

- **Secret League** (`secret_league.py`) replaces some Grand Prix in the slot 2
  rotation, every so many Grand Prix. `SECRET_LEAGUE_INTERVALS` lists the gaps,
  `SECRET_LEAGUE_OFFSET` lines them up, and `SECRET_LEAGUE_ENABLED` turns it on.
  The `WEEKEND_SECRET_LEAGUE_*` constants give weekends their own pattern.
- **Mini-Prix and Classic Mini-Prix** (`miniprix.py`): the slot 2 manager says
  when one runs, and `MiniPrixManager` works out which track selection it has,
  from the selection CSV and a line-up offset. Regular Mini-Prix also have a
  mirroring schedule. Private lobbies have their own selection schedule
  (`PrivateMPManager`), except that while a public Mini-Prix is running, its
  selection overrides the private one.
- **99 races** (`choicerace.py`): `FZ99Manager` works out the track pair each 99
  race offers, and lets a Glitch event override it.

### The constants

The cycles are right only if they line up with what the game is doing. The offsets
in `constants.dat`, such as `MINIPRIX_LINE_UP_OFFSET` and
`NINETYNINE_MINUTE_OFFSET`, shift each rotation to match the game. When the game
changes, it's usually an offset that needs adjusting, not a CSV. The `_ENABLED`
constants are feature flags, `1` or `0`.

## Profiles

An in-game event (a Leagues Weekend, a World Tour) changes the slot 2 cycle for a
while. Each one is a **profile**: a folder `config/event_<name>/` holding only the
CSVs and constants that differ from `config/event_default/`.

- A CSV in the profile folder replaces the default one whole. A CSV it doesn't
  have comes from `event_default`. A CSV whose name matches no default file is
  logged as a warning, since nothing would read it.
- `constants.dat` is merged by key: the profile's values override the default's.

### Switching

`schedule_controller.ScheduleController` holds:

- the slot 1 and 99-race managers, which no profile changes, built once;
- one `ScheduleProfile` per loaded profile: its slot 2, Mini-Prix and Secret
  League managers, built together so they agree with each other.

Exactly one profile is **active**. Code that asks the controller for a manager,
such as `controller.slot2mgr` or `controller.mp_mgr`, gets the active profile's
manager at that moment. That's how a switch takes effect without anything else
changing.

- `switch(name)` activates a profile, loading it first if needed.
- `load(name)` builds one ahead of time.
- `unload(name)` removes one, but never the active profile.

The trigger is `EVENT_SCHEDULE_FILE` (`config/server_events.csv`): rows of
`YYYY-MM-DD,profile`, each active from 00:00 UTC on its date.
`ProfileSwitchConfig` reads it, and sorts it by date. At 23:54 UTC every night, the
bot works out tomorrow's profile; if it differs, it switches and redraws its
messages straight away. Passing `--profile` turns all of this off.

## What the bot posts, and when

In `SCHEDULE_EDIT_CHANNEL`:

| Message | Refreshed |
|---|---|
| The main schedule: the event on now, the next nine, the next Glitch races, and the next of any kind not in that list | every `REFRESH_INTERVAL` minutes, and after a profile switch |
| A Mini-Prix thread and a Classic Mini-Prix thread, each with its upcoming track selections | at startup, as each Mini-Prix ends, and after a profile switch |

The message ids are kept in `.msg_struct` (see [setup.md](setup.md#the-first-run)).

The bot's **status line** shows the current Grand Prix, or the next one and how
many minutes away it is, refreshed with the schedule. `TICKER_OVERRIDE` replaces
it with fixed text.

`announce_schedule`, an hourly post in `ANNOUNCE_CHANNEL`, is still in the code but
is never started.

## Commands

All commands are py-cord slash commands in `bot.py`, registered globally except
`/ping`. They read the schedule through the controller, so they always answer from
the active profile. [commands.md](commands.md) lists them.

## Used as a library

fzd-api imports pengbot99 to serve `/v1/ingame/*`: the in-game rotation and what a
mode offers, minute by minute. It pins a tag (`v0.2.0`) from the
`feat/installable-library` branch, which packages the library so it installs
without the Discord bot's dependencies. That branch isn't merged into `main`. The
API reads its own copy of the config, so its rotation and the bot's can differ
until both are updated.
