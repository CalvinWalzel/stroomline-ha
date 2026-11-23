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
- Answers `ping` commands. It does **not** change anything in Home Assistant yet.
- If the token is revoked in Stroomline, the agent disconnects, logs it and
  shows a new pairing code.

Safety limits for later write commands are built into the agent itself:
at most 800 W per Zendure unit and 6–16 A charge current.
