# Changelog

## 0.7.0

- Mains guard: the agent keeps battery charging from overloading your main
  fuse, on its own and within about 10 seconds, also when Stroomline is
  unreachable. When the grid draw passes 85% of all phases together (or, with
  **mains_phase_entities** set, any phase passes 95% of the fuse), a manual
  charge (plan slots or Charge now) pauses at 0 W. It resumes after at least
  5 minutes, once the draw with the charge back on stays under the limit.
  Set your fuse in **mains_fuse_amps** and **mains_phases** (default 3 × 25 A)
  and the grid power sensor in **mains_grid_entity** (default
  `sensor.grid_power`). It can't stop the Zendure firmware's own low-battery
  recharge. Discharging and solar-only modes aren't affected.

## 0.6.4

- No more safety lockout from the Zendure firmware's own low-battery
  recharge. A unit that sits idle at its minimum charge slowly drifts lower
  (for example 10% to 5%), and then charges itself from the grid at its full
  2400 W back up to the minimum, past the Zendure Manager and its 800 W
  per-unit limit. Before, the agent took that for a missing 800 W limit and
  stopped battery control until a restart. Now a unit at 12% charge or less
  may charge faster while the agent isn't charging manually; the agent logs a
  warning instead. Above 12%, during a Charge now or manual plan slots, or
  when a unit discharges above 1000 W, the lockout works as before.
- A lockout reaches Stroomline right away, so the app warns within minutes.
  Before, it only showed after the agent next reconnected.

## 0.6.3

- In manual mode (Charge now or manual plan slots) the agent also watches
  `number.zendure_manager_manual_power`. When it no longer matches what the
  agent wrote, for example because an automation or the Zendure app set it to
  0 W, the agent writes it again (at most once a minute) and logs a warning.
  Before, it only corrected a changed mode, so a charge could stop halfway
  while the Manager still showed `manual`.

## 0.6.2

- At full power (800 W × **battery_units**) the agent writes 100 W more to
  `number.zendure_manager_manual_power`, because the Zendure Manager leaves
  one unit idle when the setpoint is exactly units × 800 W. Each unit stays
  at 800 W through the Manager's per-unit limit, and the agent's 1000 W
  lockout still applies. Lower setpoints are written as they are.

## 0.6.1

- A running **Charge now** survives an app restart or update: the agent saves
  it in its data folder and picks it up again with the same end time and
  target. If it ended while the app was down, the battery goes back to smart.

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
