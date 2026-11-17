# ha-tado-heizung

Presence-based heating control for Home Assistant with adaptive pre-heating, built for tado° V3+ radiator thermostats connected locally via HomeKit.

🇩🇪 [Deutsche Anleitung](README.de.md)

Everything lives in one package file. All temperatures and times are helpers you change from a dashboard card, without touching YAML.

## What it does

- **Presence per person.** Each room group belongs to a person and is heated only while that person is home. Otherwise the away temperature applies.
- **Target times instead of switch-on times.** You set when a room should be warm. The package starts heating earlier by a learned lead time.
- **Learns lead and coast times.** Every day it checks whether the room was warm on time and corrects the lead time by half the error. In the evening it measures how much the room cools after switching down early and adjusts the coast time so the room loses no more than a set amount (default 0.5 °C).
- **Window contacts.** A room switches off after a window has been open for a set time and resumes afterwards. Only active below a set outdoor temperature.
- **Workdays and holidays** through the Workday integration, **vacation mode**, and **push notifications** when a room was not warm in time.
- **Targeted safeguard.** A problem sensor per room compares the thermostat with the intended target. If they differ for more than 2 minutes (for example because tado° fell back to its own schedule, or a thermostat came back after losing its radio link), only that room is corrected. While the difference persists, it retries every 15 minutes instead of sending constantly.

## Room layout in the example

| Group | Thermostats | Person | Warm on workdays |
|---|---|---|---|
| Wohnküche (living room + kitchen) | 2 | `person.bewohner_1` | early time |
| Bad (bathroom) | 1 | `person.bewohner_1` | early time |
| Schlafzimmer (bedroom) | 1 | `person.bewohner_1` | late time |
| Zimmer 1 | 1 | `person.bewohner_2` | late time |
| Zimmer 2 | 1 | `person.bewohner_3` | late time |
| Zimmer 3 | 1 | set in a text helper | late time |

Entity names, helper names and notification texts are in German.

## Requirements

- Home Assistant 2024.10 or newer (uses the `triggers:` / `actions:` syntax)
- Thermostats as `climate` entities. Developed with tado° V3+ over the **HomeKit Device** integration, which avoids the tado° cloud API limits.
- `person` entities with working home / not_home detection
- Window contacts as `binary_sensor`
- An outdoor temperature entity
- Optional: a separate room temperature sensor, the Workday integration, the mobile app for notifications

## Installation

1. Copy `packages/heizung.yaml` to `/config/packages/heizung.yaml`.
2. Enable packages in `configuration.yaml`:
   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```
3. Replace the placeholder entities with yours. The full list is at the top of `heizung.yaml`.
4. Optional: add the Workday integration twice, named "Arbeitstag heute" (offset 0) and "Arbeitstag morgen" (offset 1).
5. Check the configuration and restart Home Assistant.
6. Run the scripts **Heizung Standardwerte setzen** and **Heizung Lernwerte zurücksetzen** once.
7. Add the card from `dashboard/heizung_dashboard.yaml` as a manual card.

## tado° settings that matter

Most problems during development came from these:

- **Manual control** must be set to "until you resume the schedule" for every room. With "timer" or "until next automatic change", tado° silently falls back to its own schedule.
- Turn off **Early Start**, **open window detection** and **geofencing** in the tado° app. Home Assistant handles all three.
- Keep a simple tado° schedule as a fallback, for example one block at 18 °C all day.
- **Radio range.** If single thermostats often show as unavailable, check the position of the tado° Internet Bridge first. Drywall with metal studs or foil-backed insulation blocks the signal much more than the distance suggests. The package keeps radio traffic low on purpose and only sends commands where a value actually differs.

## How the learning works

- **Lead time (Vorlauf), 0–45 min.** Warm too early: lead time shrinks by half the difference. Warm too late: it grows by half the delay. Still not warm one hour after the target time: plus 10 minutes and a notification.
- **Coast time (Nachlauf), 0–60 min.** The room temperature is stored when heating is reduced early and compared at the setback time. From the measured cooling rate the package derives how early it may switch down.
- Nothing is learned while a window is open, the person is away, or the room was already warm.
- Rooms without their own sensor use the temperature measured at the radiator, which rises faster than the room. An extra buffer (default 5 min) compensates for that.
- Expect one to two weeks until the values settle.

## Limitations

- Tested in a single flat with a gas boiler and seven thermostats. Treat it as a starting point for your own setup.
- Switching down early works only for fixed times. Departures cannot be predicted.
- A manual change on the thermostat or in the tado° app is reverted by the safeguard after about 2 minutes. Change temperatures permanently with the helpers in Home Assistant.
- Times are evaluated once per minute.

## License

MIT, see [LICENSE](LICENSE).
