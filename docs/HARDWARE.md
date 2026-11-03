# Hardware guide

This is a reference build for connecting an ESP32 to an intercom's **existing door-release dry-contact input**. It is intended to complement the [ESPHome configuration](../esphome/intercom-buzzer.yaml), not replace it: use that file for the actual firmware and entity behavior.

> **Safety first:** An intercom is mains-adjacent equipment in some installations. Work only with the low-voltage door-release contacts, disconnect power before wiring, and verify the terminals in your intercom's documentation. If the release input is not a dry contact, or you are unsure, use a qualified installer.

## What this reference design does

The ESP32 drives a 5 V relay through a logic-level shifter. The relay's normally-open (NO) contacts are wired in parallel with the intercom's existing unlock switch/contact pair. Closing the relay therefore simulates pressing the unlock control; it must not inject voltage into the intercom.

Home Assistant provides the two-second door-release action and alert automation. The ESPHome device exposes the relay as a latching `Intercom Buzzer` switch and independently forces it off after 60 seconds as a failsafe.

## Bill of materials

| Item | Qty. | Notes |
| --- | ---: | --- |
| ESP32 development board | 1 | An ESP32-WROOM-32 / NodeMCU-32S-style board is suitable. |
| 5 V single-channel relay module | 1 | Select an isolated module appropriate for the intercom's dry-contact circuit. |
| 3.3 V ↔ 5 V logic-level shifter | 1 | A four-channel bidirectional module was used in this reference build. |
| 5 V USB power supply and cable | 1 | Powers the ESP32 and relay module. |
| Jumper wires and insulated connectors | as needed | Match the headers/connectors on your boards. |
| Enclosure and strain relief | recommended | Keep low-voltage wiring secure and protected. |

## Reference wiring

The current ESPHome configuration uses **GPIO26**. It treats the relay module as active-low (`inverted: true`); confirm that behavior with your own relay before connecting the intercom.

| From | Pin | To | Pin | Purpose |
| --- | --- | --- | --- | --- |
| ESP32 | `3V3` | level shifter | `LV` | Low-voltage reference |
| ESP32 | `VIN` / 5 V | level shifter | `HV` | High-voltage reference |
| level shifter | `HV` | relay module | `VCC` | 5 V supply branch |
| ESP32 | `GND` | level shifter | `GND` | Common low-voltage ground |
| ESP32 | `GND` | relay module | `GND` | Common relay ground |
| ESP32 | `GPIO26` | level shifter | `LV1` | Control signal |
| level shifter | `HV1` | relay module | `IN` | 5 V relay control signal |
| relay module | `COM` + `NO` | intercom | unlock-contact pair | Isolated momentary closure |

```text
ESP32 GPIO26 ── LV1  logic-level shifter  HV1 ── relay IN
ESP32 3V3    ── LV                           HV ── 5 V / relay VCC
ESP32 GND    ── shifter GND ────────────────────── relay GND
relay COM/NO ───────────────────────────────────── intercom unlock contacts
```

Do **not** connect the ESP32's 5 V or ground directly to the intercom unlock terminals. Those terminals should connect only to the relay's `COM` and `NO` contacts unless your intercom documentation explicitly says otherwise.

## Assembly and bring-up

1. With all equipment unpowered, identify the two intercom terminals that close when the physical door-release button is pressed. Confirm this with the manufacturer documentation or a multimeter—never by trial on unknown wires.
2. Wire the ESP32, shifter, and relay according to the table above. Keep all exposed joins insulated.
3. Copy `esphome/secrets.yaml.example` to `esphome/secrets.yaml`, add your Wi-Fi/API credentials, then flash [`intercom-buzzer.yaml`](../esphome/intercom-buzzer.yaml).
4. Before attaching the intercom, power the ESP32 and confirm that the relay is **off after boot**. Toggle the `Intercom Buzzer` switch briefly and verify the relay click and release.
5. Attach the relay's `COM`/`NO` pair to the confirmed unlock-contact pair. Run the Home Assistant Door Opener action once and verify that it releases for the configured two seconds.
6. Keep the ESPHome `max_on_seconds` failsafe and verify that the relay turns off automatically. Do not lengthen it casually; that safety net is there because relays and intercom buzzers have better things to do than become space heaters.

## Troubleshooting

| Symptom | Check |
| --- | --- |
| Relay LED changes but the relay does not click | Confirm the relay has a stable 5 V supply and that its input accepts the shifted signal. |
| Relay is on at boot | Recheck GPIO selection, relay polarity, and `inverted: true`. The intended idle state is off. |
| Door does not release but the relay clicks | Reconfirm the intercom's correct unlock-contact pair and that `COM`/`NO` are used—not `NC`. |
| Door remains released / buzzer continues | Immediately power down, inspect wiring and relay polarity, then confirm the ESPHome failsafe and HA timed action. |
| ESP32 disappears from Wi-Fi after a firmware update | Ensure `esphome/secrets.yaml` contains the existing Wi-Fi credentials before compiling/flashing. |

## Installation notes

Add photographs, model numbers, the intercom's approved terminal documentation, and any local electrical/safety requirements for your own installation here. Do not commit Wi-Fi credentials, API keys, or home-specific network details.
