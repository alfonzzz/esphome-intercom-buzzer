# ESPHome Intercom Buzzer + Home Assistant Oref Alert

A small ESP32/relay project that provides:

- a **latching intercom buzzer switch** with a hardware-side auto-off failsafe;
- a **two-second Home Assistant door-opener button** built from that switch;
- an **Oref Alert** automation that sounds the buzzer for **one minute** only for
  the configured home area; and
- Lovelace tile examples for all three controls/statuses.

## Setup

1. Copy `esphome/secrets.yaml.example` to `esphome/secrets.yaml`, populate it,
   and never commit it.
2. Adapt `GPIO26`, relay polarity, and the `max_on_seconds` failsafe in
   `esphome/intercom-buzzer.yaml` to your hardware. Flash through ESPHome.
3. Install and configure [Oref Alert](https://github.com/amitfin/oref_alert)
   in Home Assistant. Configure the integration for your **home area only**.
4. Add the script in `home-assistant/scripts.yaml` and the automation in
   `home-assistant/automations.yaml`, changing entity IDs if yours differ.
5. Add the matching Lovelace tiles from `dashboard/family-dashboard-snippets.yaml`.
6. Test the door opener with its configured two-second pulse. Test the alert automation with a
   temporary five-second delay, then restore its one-minute production delay.

## Alert safety model

The Oref automation listens to `binary_sensor.oref_alert`, which represents
only the area selected when configuring Oref Alert (not its all-areas sensor).
A real alert turns the relay on, then it turns off after one minute. Pre-alerts
and all-clear timing do not extend the sound. ESPHome independently shuts the
relay off after 60 seconds if Home Assistant becomes unavailable mid-run.

## Hardware

See [docs/HARDWARE.md](docs/HARDWARE.md). This intentionally remains a
placeholder for your site-specific wiring and intercom details.

## Before publishing

- Review entity IDs and relay polarity.
- Confirm no real Wi-Fi/API secrets are in the repository.
- Choose and add a license appropriate for your intended sharing.
