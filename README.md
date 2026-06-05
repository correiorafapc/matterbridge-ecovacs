# matterbridge-ecovacs-yeedi

A [Matterbridge](https://github.com/Luligu/matterbridge) plugin that bridges **Yeedi** and **Ecovacs** robot vacuums into Matter, so they can be controlled from **Home Assistant**, Apple Home, Google Home, Amazon Alexa, and any other Matter controller.

This is a fork of [bubez81/matterbridge-ecovacs](https://github.com/bubez81/matterbridge-ecovacs) with added **Yeedi support** and a number of fixes for reliable operation in Home Assistant.

[![npm version](https://img.shields.io/npm/v/matterbridge-ecovacs-yeedi.svg)](https://www.npmjs.com/package/matterbridge-ecovacs-yeedi)
[![npm downloads](https://img.shields.io/npm/dt/matterbridge-ecovacs-yeedi.svg)](https://www.npmjs.com/package/matterbridge-ecovacs-yeedi)
[![License: Apache-2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)

---

## What's new in v1.1.1

- **Cross-platform token cache** — auth token path now uses `os.homedir()` instead of `$HOME`, fixing broken token caching on Windows.
- **Safe appVersion override** — the Ecovacs API version is now patched in-memory instead of rewriting the installed package file on disk.
- **Area ID collision fix** — non-numeric room IDs no longer all map to the same Matter area ID.
- **Deduplicated error handler** — removed a duplicate `'Error'` event listener that could cause double-processing.

---

## Features

- **Battery** level, charging state, and voltage
- **Operational state** — Docked, Running, Paused, Returning to dock, Charging
- **Operational errors** — wheels jammed, brush jammed, navigation sensor obscured, low battery, and more, mapped to Matter error states
- **Clean mode selector** — Vacuum, Mop, Vacuum and Mop, Vacuum then Mop
- **Consumable life sensors** — Filter, Main Brush, and Side Brush remaining life (%)
- **Empty Dust Bin** switch — trigger the auto-empty dock
- **Clean Completed** switch — turns on when a cleaning run finishes and the robot returns to the dock (useful for automations)
- **Room / segment cleaning** via service call
- **Start / Pause / Stop / Return to dock** commands

---

## Requirements

- A working [Matterbridge](https://github.com/Luligu/matterbridge) installation (v3.x)
- Node.js >= 20.19.0
- A Yeedi or Ecovacs account with your robot already set up in the official app

---

## Installation

### Via the Matterbridge UI (recommended)

1. Open the Matterbridge web interface
2. Go to the plugins section and choose to install a plugin
3. Enter the package name:
   ```
   matterbridge-ecovacs-yeedi
   ```
4. Install, then configure it (see below) and restart Matterbridge

### Via the command line

```bash
npm install -g matterbridge-ecovacs-yeedi
matterbridge -add matterbridge-ecovacs-yeedi
```

---

## Configuration

Configure the plugin in the Matterbridge UI, or in your Matterbridge config file.

| Option            | Description                                                          | Example          |
|-------------------|----------------------------------------------------------------------|------------------|
| `email`           | Your Yeedi/Ecovacs account email                                     | `you@email.com`  |
| `password`        | Your account password                                                | `********`       |
| `countryCode`     | Two-letter country code for your account                             | `US`             |
| `authDomain`      | `yeedi.com` for Yeedi robots, leave blank/`ecovacs.com` for Ecovacs  | `yeedi.com`      |
| `pollingInterval` | How often (seconds) to poll the robot for status                     | `15`             |
| `rooms`           | Optional list of room/segment names for segment cleaning             | see below        |

> **Yeedi users:** set `authDomain` to `yeedi.com`. This is required for the
> robot to authenticate and for map/room data to load correctly.

### Example

```json
{
  "email": "you@email.com",
  "password": "your-password",
  "countryCode": "US",
  "authDomain": "yeedi.com",
  "pollingInterval": 15,
  "rooms": ["Living Room", "Kitchen", "Bedroom"]
}
```

---

## Room / segment cleaning

Home Assistant's Matter vacuum card does not currently render a visual room
picker, but segment cleaning works through a service call:

```yaml
service: vacuum.send_command
target:
  entity_id: vacuum.yeedi_vac_robot
data:
  command: app_segment_clean
  params:
    segments: [0, 2]   # the room/segment IDs you want to clean
```

---

## Using the "Clean Completed" switch in automations

The **Clean Completed** switch turns **on** only after the robot has actually
cleaned and then returned to the dock with no error. It turns **off** when a new
run starts. This makes daily-run automations safe against double-triggering:

```yaml
alias: Daily Yeedi Clean
trigger:
  - platform: time
    at: "10:00:00"
condition:
  - condition: state
    entity_id: sensor.yeedi_vac_robot_operational_state
    state: "docked"
action:
  - service: switch.turn_off
    target:
      entity_id: switch.yeedi_vac_robot_clean_completed
  - service: vacuum.start
    target:
      entity_id: vacuum.yeedi_vac_robot
```

---

## Tested hardware

- **Yeedi Vac Pro** — fully tested

Other Yeedi and Ecovacs/Deebot models may work since they share the same
backend, but have not all been individually verified. Reports welcome.

---

## Troubleshooting

- **Robot won't connect:** double-check `email`, `password`, `countryCode`, and
  that `authDomain` is `yeedi.com` for Yeedi robots.
- **Rooms not loading:** room/map data loads a few seconds after connecting;
  give it a moment after a restart.
- **Plugin reverts after a restart:** make sure you installed from npm (not a
  one-off local file) so Matterbridge can reinstall it on container rebuilds.

---

## Credits

- Built on top of [bubez81/matterbridge-ecovacs](https://github.com/bubez81/matterbridge-ecovacs)
- Uses the [ecovacs-deebot.js](https://github.com/mrbungle64/ecovacs-deebot.js) library
- Runs on [Matterbridge](https://github.com/Luligu/matterbridge) by Luligu

---

## License

[Apache-2.0](LICENSE)
