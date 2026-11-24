# Stroomline Agent

Connects this Home Assistant to [Stroomline](https://stroomline.pixelhub.nl), the home energy planner.

## Setup

1. Set **server_url** to your Stroomline address and start the app.
2. Open the app's **Log**. It shows a pairing code like `ABCD-EFGH`.
3. In Stroomline, go to **Household → Agents** and enter the code (valid 15 minutes).
4. The agent stores its device token in its own data folder and connects. Within
   two minutes your values show up on the Live page.

## What it does (milestone 1)

- Reads the entities chosen under Household → Agents (entity mapping) and sends
  1-minute aggregates. Cars are reported as home/away only, never coordinates.
- Answers `ping` commands. Without battery or EV control (below) it doesn't change anything in Home Assistant.
- If the token is revoked in Stroomline, the agent disconnects, logs it and
  shows a new pairing code.

## History from InfluxDB (optional)

Stroomline can compute earnings for the time before the agent was installed
from the InfluxDB app's history. Create a read-only user in InfluxDB, e.g. in
the InfluxDB app's web UI (InfluxDB Admin → Users) or its CLI:

```
CREATE USER stroomline_ro WITH PASSWORD '…'
GRANT READ ON homeassistant TO stroomline_ro
```

Then set **influxdb_password** (and **influxdb_username**, **influxdb_database**
or **influxdb_url** if yours differ) in this app's configuration, restart it, and
press **Backfill history** under Household → Agents in Stroomline. The agent
only reads from InfluxDB; it doesn't change anything.

## Battery control (optional, off by default)

With **battery_control** off (the default) the agent only reads. Stroomline
runs its planner in shadow mode and compares it with what your battery does.

When you switch **battery_control** on and set the household to live in
Stroomline, the agent carries out the plan through the zendure_ha Manager:

- It only selects the Manager modes `smart`, `smart_discharging`,
  `store_solar` and `manual`, and sets `number.zendure_manager_manual_power`
  for manual. Manual power is capped at 800 W per unit × **battery_units**
  inside the agent, on top of the zendure_ha patch. At that full cap it writes
  100 W more, since the Manager otherwise leaves one unit idle; the per-unit
  800 W limit keeps each unit at 800 W.
- When the plan ends, or there is none, it puts the Manager back on `smart`.
- It keeps `sensor.stroomline_agent_heartbeat` at the time of its last
  heartbeat (also while only EV control is on), so a watchdog automation can
  put the Manager back on `smart` if the agent stops.
- If a unit reports more than 1000 W twice in a row, it stops control, sets the
  Manager to `smart_discharging` (never charges) and refuses commands until
  you restart the app.

With **battery_control** on, the **Charge now** button on Stroomline's Plan
page works too, also while the planner is still in shadow mode: the agent
charges the battery from the grid (manual, negative power, at most 800 W per
unit) for the chosen time, or until the units' mean state of charge reaches
the chosen level, whichever comes first (at most 6 hours). Then it hands the
battery back to the plan, or to the Manager (`smart`) when there is none. The
agent keeps the end time itself, so this also happens when Stroomline is
unreachable. A running charge survives an app restart or update (it's saved
in the app's data folder); if its end passed while the app was down, the
Manager goes back to `smart`.

While battery or EV control is on, the agent also installs the watchdog
automation **Stroomline watchdog** in Home Assistant (and keeps it up to date;
it removes it again when both are off). The automation runs in Home Assistant
on its own: if the heartbeat is older than 10 minutes it puts the Manager back
on `smart` (and it does the same when the Manager is still on `manual` more
than 5 minutes after a charge or the plan's manual slots should have ended), restores the wallbox fallback (three phases, charging on, cars at
16 A) and sends a push notification to every `mobile_app` phone, or to the
notify actions in **watchdog_notify** (comma-separated, such as
`notify.mobile_app_my_phone`). Stroomline's Plan page shows whether it's
installed. Set **watchdog** off if you'd rather manage the automation
yourself.

## EV control (optional, off by default)

With **ev_control** off (the default) the agent doesn't touch the wallbox or
the cars. Stroomline plans car charging in shadow mode.

When you switch **ev_control** on and set EV charging to live in Stroomline,
the agent carries out the wallbox part of the plan on the Zaptec through Home
Assistant (the car's charge current is set by Stroomline through Tessie):

- Charging on or off with **ev_charging_entity** (default
  `switch.gpn006745_charging`).
- One or three phases with **ev_phase_entity** (default
  `number.westerloane_29_3_to_1_phase_switch_current`: 32 = one phase,
  0 = three phases).
- Zaptec allows about one settings change per 15 minutes, so the agent changes
  the phases at most every 15 minutes and the charging switch at most every
  5 minutes, and writes nothing while Home Assistant already shows the wanted
  value.
- When the plan ends, or there is none, it falls back to charging on, three
  phases, and sets each car's charge current back to 16 A
  (**ev_car_amps_entities**, comma-separated; only where Home Assistant shows
  another value).
- While EV control is on it keeps the heartbeat sensor up to date too, so the
  watchdog automation can restore the same fallback if the agent stops.

