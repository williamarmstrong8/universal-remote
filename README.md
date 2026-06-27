# Universal Remote

ESPHome firmware for a [LilyGo T-Embed CC1101](https://github.com/Xinyuan-LilyGO/T-Embed-CC1101) that acts as a handheld Home Assistant remote. The UI is built with LVGL and driven by the onboard rotary encoder.

## Features

- **Home** — toggle lamps, all-on/all-off shortcuts, and navigation to TV and media panels
- **TV** — D-pad and OK controls sent to a Home Assistant `remote.*` entity
- **Media** — room picker and transport controls (play/pause, skip, volume) for Sonos `media_player.*` entities
- **Status bar** — clock, WiFi signal, and battery icon
- **Display sleep** — backlight turns off after 30 seconds of inactivity; encoder or button press wakes the screen

The CC1101 radio and sub-GHz remote receiver/transmitter are configured for future RF work but are not used by the current UI.

## Requirements

- LilyGo T-Embed CC1101
- [ESPHome](https://esphome.io/) (CLI or Home Assistant add-on)
- Home Assistant with the ESPHome integration

## Setup

1. Clone this repo.

2. Create `secrets.yaml` in the project root (gitignored):

```yaml
wifi_ssid: "your-wifi"
wifi_password: "your-password"
ap_password: "fallback-ap-password"
api_encryption_key: "generate-with-esphome"
```

Generate an API encryption key with:

```bash
esphome secrets generate-key
```

3. Update Home Assistant entity IDs in `t-embed-cc1101.yaml` to match your install:

| Area | What to change |
|------|----------------|
| Lamps | `switch.right_bed_lamp`, `switch.left_bed_lamp`, `switch.desk_lamp_2` |
| TV | `remote.kitchen_tv` in the TV tile `on_click` lambda (must be a `remote.*` entity, e.g. from the Android TV Remote integration) |
| Media rooms | `media_player.flr_2_kitchen`, `media_player.flr_1_living_room`, etc. |
| Now playing | Copy the Kitchen `text_sensor` trio and point each room at its `media_player` entity |

4. Compile and flash:

```bash
esphome run t-embed-cc1101.yaml
```

Or add the device through the ESPHome dashboard in Home Assistant and deploy from there.

## Controls

### Rotary encoder (all screens)

- **Turn** — navigate focused UI element; on the TV screen, sends directional keys (the matching D-pad button flashes to confirm)
- **Press** — activate focused element; on the TV screen, sends OK / `DPAD_CENTER` (the OK button flashes)
- **Double-press** (TV screen) — toggle dial orientation between horizontal (← →) and vertical (↑ ↓); the active axis is shown in the top-right of the TV screen
- **Hold ~500 ms** (TV screen) — return to home; a progress bar fills inside the Back button as you hold, and cancels if you let go early

On the TV screen the dial does **not** move the hover between buttons — it drives the D-pad directly. In horizontal mode a right/left turn sends `DPAD_RIGHT`/`DPAD_LEFT`; in vertical mode it sends `DPAD_DOWN`/`DPAD_UP`.

### TV screen layout

| Element | Purpose |
|---------|---------|
| ↑ ↓ ← → | D-pad indicators (flash on the matching dial turn) |
| OK | Center / select indicator (flashes on press) |
| Back (bottom) | Hold-to-home target; shows the long-press progress bar |
| Top-right ← → / ↑ ↓ | Current dial orientation |

## Customization

### Add a media room

1. Add a room button in `rooms_panel` with the correct `media_player` entity in its `on_click` lambda.
2. Copy the three `text_sensor` blocks used for Kitchen (`media_title`, `media_artist`, `media_state`) and update `entity_id` and the `selected_media_entity` condition.

### Add a lamp tile

Add a button in `home_panel` following the existing lamp tiles, plus an optional `binary_sensor` from Home Assistant to sync icon color with switch state.

### Display timeout

Change the `lvgl.on_idle.timeout` value (default `30s`).

## Project structure

```
t-embed-cc1101.yaml   ESPHome config (single source of truth)
secrets.yaml          WiFi and API credentials (not committed)
backups/              Previous config snapshots
```

## License

Private / personal use. Adjust entity IDs and UI to fit your home.
