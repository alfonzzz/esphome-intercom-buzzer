# ESPHome Intercom Buzzer + Home Assistant Oref Alert

A small ESP32/relay project that provides:

- a **latching intercom buzzer switch** with a hardware-side auto-off failsafe;
- a **two-second Home Assistant door-opener button** built from that switch;
- an **Oref Alert** automation that sounds **18 short pulses** (one second on,
  four seconds off) over 90 seconds, only for the configured home area; and
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
   temporary two-pulse cadence, then restore its 18-pulse production cadence.

## Alert safety model

The Oref automation listens to `binary_sensor.oref_alert`, which represents
only the area selected when configuring Oref Alert (not its all-areas sensor).
A real alert triggers 18 one-second relay pulses, separated by four seconds,
over 90 seconds. Pre-alerts and all-clear timing do not extend the sound. This
avoids holding the intercom's original buzzer/lock circuit energized continuously;
ESPHome independently limits any individual continuous activation to 60 seconds
if Home Assistant becomes unavailable mid-run.

## Hardware

See [docs/HARDWARE.md](docs/HARDWARE.md) for a reference build, wiring,
bring-up steps, safety notes, and troubleshooting guidance.

## Before publishing

- Review entity IDs and relay polarity.
- Confirm no real Wi-Fi/API secrets are in the repository.
- Choose and add a license appropriate for your intended sharing.
