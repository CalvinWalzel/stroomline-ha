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
- Answers `ping` commands. Without battery control (below) it doesn't change anything in Home Assistant.
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
  inside the agent, on top of the zendure_ha patch.
- When the plan ends, or there is none, it puts the Manager back on `smart`.
- It keeps `sensor.stroomline_agent_heartbeat` at the time of its last
  heartbeat, so a watchdog automation can put the Manager back on `smart` if
  the agent stops.
- If a unit reports more than 1000 W twice in a row, it stops control, sets the
  Manager to `smart_discharging` (never charges) and refuses commands until
  you restart the app.

Before going live, install the watchdog automation from
[stroomline-watchdog.yaml](https://github.com/CalvinWalzel/stroomline-ha/blob/main/stroomline-watchdog.yaml):
if the heartbeat is older than 10 minutes it puts the Manager back on `smart`
and sends a push notification.

Car charge current limits (6–16 A) are built in for later.
