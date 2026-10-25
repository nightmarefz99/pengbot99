# Structure

```text
py/pengbot99/
  bot.py                  the Discord bot: settings, startup, slash commands, refresh loops
  schedule_controller.py  the loaded schedule profiles, which one is active, and the
                          switch file (EVENT_SCHEDULE_FILE)
  schedule.py             the schedule engine: CSV loading, timetables, cycles, and the
                          slot 1 and slot 2 schedule managers
  miniprix.py             track selections for public and private Mini-Prix and
                          Classic Mini-Prix
  choicerace.py           track choices for 99 races, including Glitch overrides
  secret_league.py        which Grand Prix Secret League replaces
  events.py               the Event and MiniPrixEvent objects the managers return
  explain_cmd.py          /explain: the rotation, the current schedule, what's next
  event_profiles.py       display names and explanations of the in-game event profiles
  formatters.py           event names, Discord timestamps, track lists
  ui.py                   the choices the commands offer (event types, tracks)
  apiadapter.py           small Discord helpers: threads, message links, the status line
  utils.py                .env and constants loading, profile folders, logging, .msg_struct
config/
  event_default/          the baseline schedule: every CSV, and constants.dat
  event_<name>/           one in-game event profile: only the CSVs and constants
                          that differ from event_default
  server_events.csv       dates on which each profile becomes active
tests/                    unittest tests, with CSV fixtures in tests/fixtures/
docs/                     this documentation, and two design records
```

## The schedule files in `event_default/`

| File | Read by | What it describes |
|---|---|---|
| `slot1_schedule.csv` | the slot 1 manager | the 99-race slot's cycle, including Glitch races |
| `ninetynine_schedule.csv` | the 99-race manager | the track pairs offered in 99 races |
| `slot2_schedule.csv` | the slot 2 manager | the weekday cycle of Grand Prix and special events |
| `slot2_schedule_weekend.csv` | the slot 2 manager | the weekend cycle |
| `miniprix_schedule.csv`, `miniprix_mirroring_schedule.csv` | the public Mini-Prix manager | track selections, and which are mirrored |
| `classic_mp_schedule.csv` | the public Classic Mini-Prix manager | Classic track selections |
| `private_miniprix_schedule.csv`, `private_classic_mp_schedule.csv` | the private managers | what private lobbies get |
| `constants.dat` | everything | offsets that line the cycles up with the game, and feature flags |

`commands.md` lists every command with its handler and line.
