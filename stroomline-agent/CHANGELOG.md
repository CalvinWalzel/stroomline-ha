# Changelog

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
