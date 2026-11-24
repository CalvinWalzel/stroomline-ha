# Changelog

## 0.6.0

- **Charge now**: Stroomline's Plan page can charge the battery from the grid
  for a while, optionally until a state of charge (`until_soc`, the units'
  mean). The agent ends it on its own when the time is up or the target is
  reached, hands the battery back to the plan or the Zendure Manager (smart),
  and tells Stroomline. Overrides now last up to 6 hours, and a new plan no
  longer cancels a running one.
- The heartbeat sensor carries `manual_until`. The watchdog automation also
  puts the Manager back on smart when it's still on manual more than 5 minutes
  after that, even while the agent is running (e.g. a write that didn't land).
  The agent updates the automation itself.

## 0.5.0

- The agent installs the watchdog automation itself (**Stroomline watchdog**,
  id `stroomline_watchdog`) while battery or EV control is on, keeps it up to
  date with the entities from the app options, and removes it when both are
  off. No more pasting YAML; if you pasted it by hand before, delete that copy.
- Notifications go to every `mobile_app` phone Home Assistant knows, or to the
  notify actions in the new **watchdog_notify** option (comma-separated).
- Set the new **watchdog** option off to manage the automation yourself.
- Stroomline shows on the Plan page whether the watchdog is installed.

## 0.4.0

- EV control (off by default): with the new **ev_control** option on, the agent
  carries out the wallbox part of Stroomline's EV plan on the Zaptec through
  Home Assistant: charging on/off (**ev_charging_entity**) and one or three
  phases (**ev_phase_entity**). Phase changes at most every 15 minutes (Zaptec
  fair use); nothing is written while Home Assistant already shows the value.
- Without a current EV plan the wallbox falls back to charging on, three
  phases, and the cars' charge current (**ev_car_amps_entities**) to 16 A.
- The heartbeat sensor also runs while only EV control is on, and carries the
  wallbox state; the watchdog automation restores the EV fallback too.

## 0.3.0

- Battery control (off by default): with the new **battery_control** option
  on, the agent carries out Stroomline's battery plan through the zendure_ha
  Manager (`select.zendure_manager_operation`, `number.zendure_manager_manual_power`).
  Only the modes smart, smart_discharging, store_solar and manual are used,
  manual power is capped at 800 W per unit (**battery_units**, default 3), and
  without a current plan the Manager goes back to smart.
- While battery control is on, the agent keeps `sensor.stroomline_agent_heartbeat`
  up to date for the watchdog automation.
- Safety lockout: a unit reading over 1000 W twice in a row (the 800 W
  zendure_ha patch is probably gone) stops control and sets the Manager to
  smart_discharging until the agent restarts.

## 0.2.0

- History backfill: on request from Stroomline (Household → Agents → History),
  the agent reads past measurements from the InfluxDB app with a read-only
  user and sends them as 15-minute values. New options: influxdb_url,
  influxdb_database, influxdb_username, influxdb_password.

## 0.1.1

- Entities can report a fixed value while unavailable (used for the PV
  inverter, which switches off at night and now counts as 0 W).

## 0.1.0

- First version: pairing, 1-minute telemetry, ping.
