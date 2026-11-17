# ha-tado-heizung

Anwesenheitsabhängige Heizungssteuerung für Home Assistant mit lernendem Vorheizen, entwickelt für tado° V3+ Heizkörperthermostate, die lokal über HomeKit eingebunden sind.

🇬🇧 [English version](README.md)

Alles steckt in einer Paket-Datei. Temperaturen und Zeiten sind Helfer, die du über eine Dashboard-Karte änderst, ohne YAML anzufassen.

## Funktionen

- **Anwesenheit pro Person.** Jede Raumgruppe gehört zu einer Person und wird nur geheizt, solange diese zu Hause ist. Sonst gilt die Abwesenheitstemperatur.
- **Zielzeiten statt Einschaltzeiten.** Du legst fest, wann ein Raum warm sein soll. Das Paket schaltet um einen gelernten Vorlauf früher ein.
- **Lernt Vorlauf und Nachlauf.** Jeden Tag wird geprüft, ob der Raum pünktlich warm war, und der Vorlauf um die halbe Abweichung korrigiert. Abends wird gemessen, wie stark der Raum nach dem frühen Absenken abkühlt, und der Nachlauf so angepasst, dass er höchstens einen eingestellten Wert verliert (Standard 0,5 °C).
- **Fensterkontakte.** Ein Raum schaltet ab, wenn ein Fenster eine eingestellte Zeit offen ist, und danach wieder ein. Nur aktiv unter einer eingestellten Außentemperatur.
- **Arbeitstage und Feiertage** über die Integration „Arbeitstag“, **Urlaubsmodus** und **Push-Nachrichten**, wenn ein Raum nicht rechtzeitig warm war.
- **Selbstkontrolle.** Eine Minute nach jeder Änderung werden die Sollwerte erneut gesendet. Ein Problem-Sensor pro Raum zeigt, wenn ein Thermostat von der Vorgabe abweicht.

## Raumaufteilung im Beispiel

| Gruppe | Thermostate | Person | Warm an Arbeitstagen |
|---|---|---|---|
| Wohnküche (Wohnzimmer + Küche) | 2 | `person.bewohner_1` | frühe Zeit |
| Bad | 1 | `person.bewohner_1` | frühe Zeit |
| Schlafzimmer | 1 | `person.bewohner_1` | späte Zeit |
| Zimmer 1 | 1 | `person.bewohner_2` | späte Zeit |
| Zimmer 2 | 1 | `person.bewohner_3` | späte Zeit |
| Zimmer 3 | 1 | über Textfeld einstellbar | späte Zeit |

## Voraussetzungen

- Home Assistant 2024.10 oder neuer (Schreibweise `triggers:` / `actions:`)
- Thermostate als `climate`-Entitäten. Entwickelt mit tado° V3+ über die Integration **HomeKit Device**, damit greifen die Limits der tado°-Cloud-API nicht.
- `person`-Entitäten mit funktionierender Anwesenheitserkennung
- Fensterkontakte als `binary_sensor`
- Eine Entität für die Außentemperatur
- Optional: eigener Raumtemperatursensor, Integration „Arbeitstag“, Mobile-App für Benachrichtigungen

## Installation

1. `packages/heizung.yaml` nach `/config/packages/heizung.yaml` kopieren.
2. Pakete in der `configuration.yaml` aktivieren:
   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```
3. Die Platzhalter-Entitäten durch deine ersetzen. Die vollständige Liste steht oben in der `heizung.yaml`.
4. Optional: die Integration „Arbeitstag“ zweimal anlegen, mit den Namen „Arbeitstag heute“ (Offset 0) und „Arbeitstag morgen“ (Offset 1).
5. Konfiguration prüfen und Home Assistant neu starten.
6. Die Skripte **Heizung Standardwerte setzen** und **Heizung Lernwerte zurücksetzen** einmal ausführen.
7. Die Karte aus `dashboard/heizung_dashboard.yaml` als manuelle Karte einfügen.

## Wichtige tado°-Einstellungen

Daran lagen während der Entwicklung die meisten Probleme:

- **Manuelle Steuerung** muss in jedem Raum auf „Bis du den Zeitplan fortsetzt“ stehen. Bei „Timer“ oder „Bis zur nächsten automatischen Änderung“ springt tado° unbemerkt auf den eigenen Zeitplan zurück.
- **Early Start**, **Fenster-offen-Erkennung** und **Geofencing** in der tado°-App ausschalten. Das übernimmt Home Assistant.
- Als Rückfall einen einfachen tado°-Zeitplan behalten, zum Beispiel einen Block mit 18 °C für den ganzen Tag.

## So funktioniert das Lernen

- **Vorlauf, 0–45 Min.** Zu früh warm: Der Vorlauf wird um die halbe Differenz kürzer. Zu spät warm: Er wird um die halbe Verspätung länger. Eine Stunde nach der Zielzeit noch nicht warm: plus 10 Minuten und eine Benachrichtigung.
- **Nachlauf, 0–60 Min.** Beim frühen Absenken wird die Raumtemperatur gemerkt und zur Absenkzeit verglichen. Aus der gemessenen Abkühlung ergibt sich, wie früh abgesenkt werden darf.
- Nicht gelernt wird, wenn ein Fenster offen war, die Person abwesend ist oder der Raum schon warm war.
- Räume ohne eigenen Sensor nutzen die Temperatur am Heizkörper, die schneller steigt als im Raum. Ein Zusatzpuffer (Standard 5 Min.) gleicht das aus.
- Bis sich die Werte einpendeln, dauert es ein bis zwei Wochen.

## Grenzen

- Getestet in einer einzelnen Wohnung mit Gasheizung und sieben Thermostaten. Sieh es als Ausgangspunkt für dein eigenes Setup.
- Früher absenken geht nur bei festen Zeiten. Wann jemand geht, lässt sich nicht vorhersehen.
- Eine Änderung von Hand am Thermostat oder in der tado°-App hält bis zum nächsten Ereignis (Anwesenheit, Fenster, Uhrzeit).
- Zeiten werden einmal pro Minute ausgewertet.

## Lizenz

MIT, siehe [LICENSE](LICENSE).
