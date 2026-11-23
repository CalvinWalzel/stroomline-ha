# Stroomline for Home Assistant

Home Assistant app repository for [Stroomline](https://stroomline.pixelhub.nl),
a home energy planner for solar, home batteries and EV charging.

## Install

1. In Home Assistant, open **Settings → Apps → App store**.
2. Open the **⋮** menu → **Repositories** and add
   `https://github.com/CalvinWalzel/stroomline-ha`.
3. Install **Stroomline Agent**, start it and follow the pairing steps in its
   Documentation tab.

The agent image is published as `ghcr.io/calvinwalzel/stroomline-agent`.
This repository only holds the app's metadata; it is generated from the main
(private) Stroomline repository, so changes made here get overwritten.
