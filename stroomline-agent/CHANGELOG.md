# Changelog

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
