# SunEnergyXT 500 / 500 Pro Zero Feed-in Blueprint

Home-Assistant-Blueprint für die Nulleinspeisungsregelung eines SunEnergyXT 500 oder 500 Pro mit externem Netzzähler, SOC-Grenzen, Fail-safe-Logik, Diagnoseprotokollierung und optionaler zeitgesteuerter Netzladung mit PV-Vorrang.

## Funktionsumfang

- Automatische Zuordnung der relevanten SunEnergyXT-Entitäten über das ausgewählte Gerät
- Unterstützung mehrerer Zählertypen und benutzerdefinierter Zählerformeln
- Dynamische Regelintervalle und begrenzte Sollwertänderungen
- SOC-Unter- und Obergrenzen mit optional persistenten Halte-Helpern
- Zeitgesteuerte Netzladung nach Monat, Wochentag, Uhrzeit und Ziel-SOC
- Bedarfsgerechter Ladebeginn anhand Kapazität, Leistung, Wirkungsgrad und Sicherheitspuffer
- PV-Vorrang mit konfigurierbarer Sperr- und Freigabeschwelle
- Fail-safe bei ungültigen oder veralteten Messwerten
- Mehrstufiges Diagnoseprotokoll

## Installation

1. Kopiere die Blueprint-Datei nach:

```text
/config/blueprints/automation/sunenergyxt/sunenergyxt-500-zero-feed-in.yaml
```

2. Lade die Automationen beziehungsweise Blueprints in Home Assistant neu.
3. Erstelle unter **Einstellungen → Automatisierungen & Szenen → Blueprints** eine neue Automation aus dem Blueprint.
4. Wähle das SunEnergyXT-Gerät, den passenden Zählertyp und die gewünschten Einstellungen.

## PV-Hysterese-Helper

Für eine echte zustandsbehaftete PV-Hysterese wird ein Umschalt-Helper empfohlen.

1. Öffne **Einstellungen → Geräte & Dienste → Helfer**.
2. Wähle **Helfer erstellen → Umschalter**.
3. Verwende beispielsweise den Namen `SunEnergyXT PV-Netzladesperre`.
4. Wähle den Helper anschließend im Blueprint unter **PV-Vorrang und Hysterese → Optionaler Helper für PV-Netzladesperre** aus.

Beispiel als YAML, falls Helper per YAML verwaltet werden:

```yaml
input_boolean:
  sunenergyxt_pv_netzladesperre:
    name: SunEnergyXT PV-Netzladesperre
    icon: mdi:solar-power
```

### Verhalten der Hysterese

Bei einer Sperrschwelle von 1.000 W und einer Freigabeschwelle von 700 W gilt:

- PV-Leistung ab 1.000 W: Netzladung gesperrt, Helper eingeschaltet.
- PV-Leistung unter 700 W: Netzladung freigegeben, Helper ausgeschaltet.
- PV-Leistung zwischen 700 und 1.000 W: Der im Helper gespeicherte Zustand bleibt erhalten.
- Ohne Helper wird der Zwischenbereich vorsichtshalber als gesperrt behandelt.

## Konfigurationsreferenz

### SunEnergyXT-Einrichtung

Wählen Sie das SunEnergyXT-All-in-one-Gerät und die wichtigsten Benutzereinstellungen. Der Blueprint bindet passende Entitäten dieses Geräts automatisch und schreibt die SOC-Grenzen in die Geräteeinstellungen.

#### `aio_device`: SunEnergyXT-Gerät

Wählen Sie das SunEnergyXT 500 / 500 Pro Gerät. Der Blueprint sucht automatisch den Sollwert Leistung Netzanschluss, den Sollwert max. Wechselrichterleistung, die Systemleistung am Lastanschluss, die Systemleistung am Netzanschluss, die PV-Gesamteingangsleistung, das System-Speicherlevel und die SOC-Grenzentitäten dieses Geräts.

- **Eingabetyp:** Gerät (Home-Assistant-Geräteauswahl)
- **Standard:** Kein Standardwert

#### `full_battery_mode`: Verhalten bei voller Batterie

- **Eingabetyp:** Auswahl: Nach Vollladung Last folgen, Nach Vollladung PV folgen
- **Standard:** follow_load

#### `gs_max_w`: Maximale Netzeinspeiseleistung

Maximal erlaubte Netzeinspeiseleistung. Verwenden Sie 800 W für SunEnergyXT 500 und 2400 W für SunEnergyXT 500 Pro, sofern vor Ort kein niedrigerer Grenzwert erforderlich ist.

- **Eingabetyp:** Zahl (0 bis 2400, Schritt 1, W)
- **Standard:** 2400

#### `min_discharge_soc`: System Entladegrenze

Schreibt die System Entladegrenze des ausgewählten SunEnergyXT- Geräts. Die Automatisierung verwendet diesen Wert außerdem als Steuergrenze bei niedrigem SOC.

- **Eingabetyp:** Zahl (1 bis 30, Schritt 1, %)
- **Standard:** 10

#### `max_charge_soc`: System Ladegrenze

Schreibt die System Ladegrenze des ausgewählten SunEnergyXT-Geräts. Die Automatisierung verwendet diesen Wert außerdem als Steuergrenze bei voller Batterie.

- **Eingabetyp:** Zahl (70 bis 100, Schritt 1, %)
- **Standard:** 100

#### `ac_couple_max_charge_power_w`: Maximale AC-gekoppelte Ladeleistung

Begrenzt die AC-seitige Lade-/Aufnahmeleistung. SunEnergyXT 500 und 500 Pro können hier bis zu 2400 W verwenden, sofern vor Ort kein niedrigerer Grenzwert gilt.

- **Eingabetyp:** Zahl (0 bis 2400, Schritt 1, W)
- **Standard:** 2400

### Manuelle Netz-Ladung

Lädt den Speicher während frei definierbarer Zeitfenster aus dem Netz. Die normale Nulleinspeisungsregelung wird währenddessen temporär übersteuert. Optional kann die Netzladung bei ausreichender aktueller PV-Leistung gesperrt werden. Eine getrennte Sperr- und Freigabeschwelle bildet eine Hysterese und verhindert häufiges Umschalten bei wechselnder Bewölkung. Für eine echte zustandsbehaftete Hysterese kann optional ein input_boolean-Helper ausgewählt werden.

#### `grid_charge_enable`: Netzladung aktivieren

Aktiviert die zeitgesteuerte Ladung aus dem Stromnetz.

- **Eingabetyp:** Schalter
- **Standard:** Deaktiviert

#### `grid_charge_months`: Monate

In diesen Monaten darf die Netzladung erfolgen.

- **Eingabetyp:** Auswahl, Mehrfachauswahl: Januar, Februar, März, April, Mai, Juni, Juli, August, September, Oktober, November, Dezember
- **Standard:** 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12

#### `grid_charge_days`: Wochentage

An diesen Tagen darf die Netzladung erfolgen.

- **Eingabetyp:** Auswahl, Mehrfachauswahl: Montag, Dienstag, Mittwoch, Donnerstag, Freitag, Samstag, Sonntag
- **Standard:** mon, tue, wed, thu, fri

#### `grid_charge_start`: Startzeit

Beginn des günstigen Tarifzeitraums.

- **Eingabetyp:** Uhrzeit
- **Standard:** 10:00:00

#### `grid_charge_end`: Endzeit

Ende des günstigen Tarifzeitraums.

- **Eingabetyp:** Uhrzeit
- **Standard:** 16:00:00

#### `grid_charge_target_soc`: Ziel-SOC

Die Netzladung stoppt beim Erreichen dieses Ladezustands.

- **Eingabetyp:** Zahl (10 bis 100, Schritt 1, %)
- **Standard:** 90

#### `grid_charge_power_w`: Netzladeleistung

Gewünschte Ladeleistung aus dem Stromnetz.

- **Eingabetyp:** Zahl (100 bis 2400, Schritt 100, W)
- **Standard:** 2400

#### `grid_charge_battery_capacity_kwh`: Batteriekapazität

Nutzbare Batteriekapazität für die Berechnung der benötigten Netzladezeit.

- **Eingabetyp:** Zahl (1 bis 100, Schritt 0.1, kWh)
- **Standard:** 10

#### `grid_charge_efficiency`: Ladeeffizienz

Geschätzte Ladeeffizienz für die Berechnung. 90 % berücksichtigt Ladeverluste und sorgt für einen etwas früheren Ladestart.

- **Eingabetyp:** Zahl (50 bis 100, Schritt 1, %)
- **Standard:** 90

#### `grid_charge_safety_margin_min`: Sicherheitspuffer

Zusätzliche Zeitreserve vor dem Ende des günstigen Tarifzeitraums.

- **Eingabetyp:** Zahl (0 bis 60, Schritt 1, min)
- **Standard:** 10

### PV-Vorrang und Hysterese

Diese optionale Funktion verhindert eine unnötige Netzladung, wenn aktuell genügend PV-Leistung vorhanden ist. Die Sperrschwelle schaltet die Netzladung aus; die niedrigere Freigabeschwelle erlaubt sie wieder. Der Abstand zwischen beiden Werten bildet die Hysterese und verhindert häufiges Umschalten bei wechselnder Bewölkung. Für eine echte zustandsbehaftete Hysterese wird ein input_boolean-Helper empfohlen. Erstellen Sie ihn unter Einstellungen → Geräte & Dienste → Helfer → Helfer erstellen → Umschalter. Empfohlener Name: SunEnergyXT PV-Netzladesperre. Wählen Sie den erstellten Helper danach im Feld "Optionaler Helper für PV-Netzladesperre" aus. Ohne Helper wird der Bereich zwischen Freigabe- und Sperrschwelle vorsichtshalber als gesperrt behandelt.

#### `grid_charge_pv_priority_enable`: PV-Vorrang für Netzladung aktivieren

Aktiviert die Sperre der manuellen Netzladung bei ausreichender aktueller PV-Leistung. Ist diese Option deaktiviert, beeinflussen die beiden PV-Schwellenwerte die Netzladung nicht.

- **Eingabetyp:** Schalter
- **Standard:** Aktiviert

#### `grid_charge_pv_block_threshold_w`: PV-Leistung für Netzlade-Sperre

Erreicht oder überschreitet die aktuelle PV-Leistung diesen Wert, wird die manuelle Netzladung gesperrt und die PV erhält Vorrang. Die Sperre bleibt aktiv, bis die PV-Leistung unter die separate Freigabeschwelle fällt.

- **Eingabetyp:** Zahl (0 bis 2000, Schritt 50, W)
- **Standard:** 1000

#### `grid_charge_pv_release_threshold_w`: PV-Leistung für Freigabe der Netzladung

Fällt die aktuelle PV-Leistung unter diesen Wert, darf die manuelle Netzladung wieder starten. Dieser Wert sollte kleiner als die Sperrschwelle sein. Der Abstand zwischen beiden Werten bildet die Hysterese und verhindert wiederholtes Ein- und Ausschalten bei schwankender PV-Leistung.

- **Eingabetyp:** Zahl (0 bis 2000, Schritt 50, W)
- **Standard:** 700

#### `grid_charge_pv_block_helper`: Optionaler Helper für PV-Netzladesperre

Optionaler input_boolean-Helper, der den Sperrzustand innerhalb der Hysterese und über Automationsläufe hinweg speichert. Empfohlen für eine exakte Hysterese. Ohne Helper wird der Zwischenbereich zwischen Freigabe- und Sperrschwelle vorsichtshalber als gesperrt behandelt.

- **Eingabetyp:** Entität (input_boolean)
- **Standard:** Leer / optional

### Zähler-Einrichtung

Wählen Sie genau einen Zählertyp. Home Assistant Blueprints können Felder anhand dieser Auswahl nicht dynamisch ausblenden. Öffnen Sie daher nur das passende Zählerformular unten und lassen Sie die anderen Formulare leer.

#### `meter_power_source`: Zählertyp

Wählen Sie die Form der Zählerintegration. Füllen Sie nur die Felder aus, die zum ausgewählten Typ passen.

- **Eingabetyp:** Auswahl: Shelly Pro 3EM, EcoTracker, BitShake / Tasmota, Shelly 3EM, Benutzerdefinierter Zähler
- **Standard:** Kein Standardwert

### Shelly Pro 3EM Zähler

Dieses Formular nur ausfüllen, wenn als Zählertyp Shelly Pro 3EM ausgewählt ist.

#### `pro3em_meter_device`: Shelly Pro 3EM Gerät

Wählen Sie das Shelly Pro 3EM Zählergerät.

- **Eingabetyp:** Gerät (Home-Assistant-Geräteauswahl)
- **Standard:** Leer / optional

### EcoTracker-Zähler

Dieses Formular nur ausfüllen, wenn als Zählertyp EcoTracker ausgewählt ist.

#### `ecotracker_meter_device`: EcoTracker-Gerät

Wählen Sie das EcoTracker-Zählergerät.

- **Eingabetyp:** Gerät (Home-Assistant-Geräteauswahl)
- **Standard:** Leer / optional

### BitShake-/Tasmota-Zähler

Dieses Formular nur ausfüllen, wenn als Zählertyp BitShake / Tasmota ausgewählt ist.

#### `bitshake_meter_device`: BitShake-/Tasmota-Gerät

Wählen Sie das BitShake- oder Tasmota-Zählergerät.

- **Eingabetyp:** Gerät (Home-Assistant-Geräteauswahl)
- **Standard:** Leer / optional

### Shelly-3EM-Zähler

Dieses Formular nur ausfüllen, wenn als Zählertyp Shelly 3EM ausgewählt ist. Wählen Sie die drei Phasengeräte; der Blueprint liest die Leistung jeder Phase vom jeweiligen Gerät.

#### `meter_l1_device`: Shelly 3EM Phase A / L1 Gerät

Wählen Sie das HA-Gerät für Phase A / L1.

- **Eingabetyp:** Gerät (Home-Assistant-Geräteauswahl)
- **Standard:** Leer / optional

#### `meter_l2_device`: Shelly 3EM Phase B / L2 Gerät

Wählen Sie das HA-Gerät für Phase B / L2.

- **Eingabetyp:** Gerät (Home-Assistant-Geräteauswahl)
- **Standard:** Leer / optional

#### `meter_l3_device`: Shelly 3EM Phase C / L3 Gerät

Wählen Sie das HA-Gerät für Phase C / L3.

- **Eingabetyp:** Gerät (Home-Assistant-Geräteauswahl)
- **Standard:** Leer / optional

### Benutzerdefinierter Zähler

Dieses Formular nur ausfüllen, wenn als Zählertyp Benutzerdefinierter Zähler ausgewählt ist. Ein benutzerdefinierter Zähler benötigt kein Zählergerät; wählen Sie eine Formel und geben Sie die passenden Entitäten an.

#### `custom_meter_formula`: Leistungsformel des benutzerdefinierten Zählers

Der Blueprint wendet diese Formel auf die unten angegebenen Entitäten an.

- **Eingabetyp:** Auswahl: Vorzeichenbehaftete Gesamtleistung, L1 + L2 + L3, Einspeisung - Bezug, L1-L3 Einspeisung - Bezug, Entitätsattribut
- **Standard:** custom_signed_total_power

#### `meter_power_sensor`: Benutzerdefinierte Gesamt-/Aktualleistungsentität

Für vorzeichenbehaftete Gesamtleistung oder Entitätsattribut verwenden.

- **Eingabetyp:** Entität (sensor)
- **Standard:** Leer / optional

#### `meter_power_attribute`: Benutzerdefiniertes Leistungsattribut

Nur verwenden, wenn die Leistungsformel Entitätsattribut ist. Der übliche Attributname ist power.

- **Eingabetyp:** Text
- **Standard:** power

#### `meter_l1_power_sensor`: Benutzerdefinierte L1-/Phase-A-Leistungsentität

Für die benutzerdefinierte Formel L1 + L2 + L3 verwenden.

- **Eingabetyp:** Entität (sensor)
- **Standard:** Leer / optional

#### `meter_l2_power_sensor`: Benutzerdefinierte L2-/Phase-B-Leistungsentität

Für die benutzerdefinierte Formel L1 + L2 + L3 verwenden.

- **Eingabetyp:** Entität (sensor)
- **Standard:** Leer / optional

#### `meter_l3_power_sensor`: Benutzerdefinierte L3-/Phase-C-Leistungsentität

Für die benutzerdefinierte Formel L1 + L2 + L3 verwenden.

- **Eingabetyp:** Entität (sensor)
- **Standard:** Leer / optional

#### `meter_import_power_sensor`: Benutzerdefinierte Bezugsleistungsentität

Für die benutzerdefinierte Formel Einspeisung - Bezug verwenden.

- **Eingabetyp:** Entität (sensor)
- **Standard:** Leer / optional

#### `meter_export_power_sensor`: Benutzerdefinierte Einspeiseleistungsentität

Für die benutzerdefinierte Formel Einspeisung - Bezug verwenden.

- **Eingabetyp:** Entität (sensor)
- **Standard:** Leer / optional

#### `meter_l1_import_power_sensor`: Benutzerdefinierte L1-/Phase-A-Bezugsleistungsentität

Für die benutzerdefinierte Formel L1-L3 Einspeisung - Bezug verwenden.

- **Eingabetyp:** Entität (sensor)
- **Standard:** Leer / optional

#### `meter_l2_import_power_sensor`: Benutzerdefinierte L2-/Phase-B-Bezugsleistungsentität

Für die benutzerdefinierte Formel L1-L3 Einspeisung - Bezug verwenden.

- **Eingabetyp:** Entität (sensor)
- **Standard:** Leer / optional

#### `meter_l3_import_power_sensor`: Benutzerdefinierte L3-/Phase-C-Bezugsleistungsentität

Für die benutzerdefinierte Formel L1-L3 Einspeisung - Bezug verwenden.

- **Eingabetyp:** Entität (sensor)
- **Standard:** Leer / optional

#### `meter_l1_export_power_sensor`: Benutzerdefinierte L1-/Phase-A-Einspeiseleistungsentität

Für die benutzerdefinierte Formel L1-L3 Einspeisung - Bezug verwenden.

- **Eingabetyp:** Entität (sensor)
- **Standard:** Leer / optional

#### `meter_l2_export_power_sensor`: Benutzerdefinierte L2-/Phase-B-Einspeiseleistungsentität

Für die benutzerdefinierte Formel L1-L3 Einspeisung - Bezug verwenden.

- **Eingabetyp:** Entität (sensor)
- **Standard:** Leer / optional

#### `meter_l3_export_power_sensor`: Benutzerdefinierte L3-/Phase-C-Einspeiseleistungsentität

Für die benutzerdefinierte Formel L1-L3 Einspeisung - Bezug verwenden.

- **Eingabetyp:** Entität (sensor)
- **Standard:** Leer / optional

### Erweiterte Entitätsüberschreibungen

Für die automatische gerätebasierte Bindung leer lassen. Nur öffnen, wenn das ausgewählte Gerät ungewöhnliche Entitätsnamen bereitstellt und das automatische Ergebnis falsch ist.

#### `gs_number`: Entität für Sollwert Leistung Netzanschluss überschreiben

- **Eingabetyp:** Entität (number)
- **Standard:** Leer / optional

#### `is_number`: Entität für Sollwert max. Wechselrichterleistung überschreiben

- **Eingabetyp:** Entität (number)
- **Standard:** Leer / optional

#### `load_power_sensor`: Sensor für Systemleistung am Lastanschluss überschreiben

- **Eingabetyp:** Entität (sensor)
- **Standard:** Leer / optional

#### `grid_port_power_sensor`: Sensor für Systemleistung am Netzanschluss überschreiben

- **Eingabetyp:** Entität (sensor)
- **Standard:** Leer / optional

#### `pv_power_sensor`: Sensor für PV-Gesamteingangsleistung überschreiben

- **Eingabetyp:** Entität (sensor)
- **Standard:** Leer / optional

#### `soc_sensor`: Sensor für System-Speicherlevel überschreiben

- **Eingabetyp:** Entität (sensor)
- **Standard:** Leer / optional

#### `min_discharge_soc_number`: Number-Entität für System Entladegrenze überschreiben

- **Eingabetyp:** Entität (number)
- **Standard:** Leer / optional

#### `max_charge_soc_number`: Number-Entität für System Ladegrenze überschreiben

- **Eingabetyp:** Entität (number)
- **Standard:** Leer / optional

### Erweiterte Zähler-Vorzeichen- und Einheitseinstellungen

#### `meter_sign_mode`: Vorzeichenkonvention des Zählers

Automatisch verwendet die Vorgabe des ausgewählten Zählertyps. Wählen Sie eine feste Vorgabe, wenn Stromwandler, Skript oder Template das entgegengesetzte Vorzeichen liefern.

- **Eingabetyp:** Auswahl: Automatisch nach ausgewähltem Zählertyp, Einspeisung ist positiv, Netzbezug ist positiv
- **Standard:** auto_from_preset

#### `meter_power_multiplier`: Leistungsfaktor externer Zähler

1 verwenden, wenn die Entität bereits W liefert. 1000 verwenden, wenn die Entität kW liefert.

- **Eingabetyp:** Zahl (0.001 bis 1000, Schritt 0.001)
- **Standard:** 1

### Erweiterte Steuerungseinstellungen

#### `target_grid_power_w`: Ziel-Netzleistung

Nulleinspeisung verwendet 0 W. Positive Werte halten bewusst eine kleine Einspeisung; negative Werte halten bewusst einen kleinen Bezug.

- **Eingabetyp:** Zahl (-200 bis 200, Schritt 1, W)
- **Standard:** 0

#### `target_grid_power_entity`: Optionaler Helper für Ziel-Netzleistung

Optionaler input_number-Helper für einen dynamischen Zielwert. Ist kein Helper ausgewählt oder sein Zustand ungültig, wird die oben eingestellte Ziel-Netzleistung verwendet.

- **Eingabetyp:** Entität (input_number)
- **Standard:** Leer / optional

#### `gs_min_w`: Untere Grenze für Sollwert Leistung Netzanschluss

Minimal erlaubter Sollwert Leistung Netzanschluss. Negative Werte erlauben Netzbezug bzw. Laden, um Überschussleistung aufzunehmen.

- **Eingabetyp:** Zahl (-2400 bis 0, Schritt 1, W)
- **Standard:** -2400

#### `gs_resolution_w`: Schreibauflösung Sollwert Leistung Netzanschluss

- **Eingabetyp:** Zahl (1 bis 50, Schritt 1, W)
- **Standard:** 1

#### `gain_percent`: Korrekturverstärkung

- **Eingabetyp:** Zahl (20 bis 150, Schritt 5, %)
- **Standard:** 100

#### `small_error_w`: Schwelle für kleinen Fehler

- **Eingabetyp:** Zahl (1 bis 300, Schritt 1, W)
- **Standard:** 30

#### `large_error_w`: Schwelle für großen Fehler

- **Eingabetyp:** Zahl (10 bis 1000, Schritt 10, W)
- **Standard:** 150

#### `slow_interval_s`: Langsames Schreibintervall

- **Eingabetyp:** Zahl (1 bis 120, Schritt 0.5, s)
- **Standard:** 7

#### `medium_interval_s`: Mittleres Schreibintervall

- **Eingabetyp:** Zahl (0.5 bis 60, Schritt 0.5, s)
- **Standard:** 2.5

#### `fast_interval_s`: Schnelles Schreibintervall

- **Eingabetyp:** Zahl (1 bis 20, Schritt 1, s)
- **Standard:** 1

#### `gs_feedback_settle_s`: Einschwingzeit der Regelungsrückmeldung

Mindestzeit nach dem Schreiben des Sollwerts Leistung Netzanschluss oder des Sollwerts max. Wechselrichterleistung bis zum nächsten Schreibvorgang. Die Automatisierung wartet außerdem auf neue Netzanschluss- und Zählerwerte nach der letzten Änderung. Der Standardwert passt zur beobachteten SunEnergyXT-Rückmeldezeit und reduziert Schwingen bei Änderungen externer Geräte.

- **Eingabetyp:** Zahl (1 bis 30, Schritt 1, s)
- **Standard:** 5

#### `meter_stabilization_delay_s`: Zusätzliche Wartezeit bei verzögerter Zählerrückmeldung

Optionale zusätzliche Wartezeit nach einem GS- oder IS-Schreibvorgang, bevor erneut geregelt wird. Bei Zählern ohne merkliche Verzögerung 0 s beibehalten. Den Wert etwas größer als die gemessene inhaltliche Verzögerung einstellen, wenn der Zähler in Home Assistant zwar neue Zeitstempel meldet, der Leistungswert aber noch den Zustand vor dem letzten Schreibvorgang zeigt. Ein zu kleiner Wert kann Schwingen zulassen; ein zu großer Wert verlangsamt die Reaktion auf echte Laständerungen.

- **Eingabetyp:** Zahl (0 bis 60, Schritt 1, s)
- **Standard:** 0

#### `small_max_step_w`: Maximale Anpassung bei kleinem Fehler

- **Eingabetyp:** Zahl (1 bis 300, Schritt 1, W)
- **Standard:** 20

#### `medium_max_step_w`: Maximale Anpassung bei mittlerem Fehler

- **Eingabetyp:** Zahl (10 bis 1000, Schritt 10, W)
- **Standard:** 120

#### `fast_max_step_w`: Maximale Anpassung bei großem Fehler

- **Eingabetyp:** Zahl (50 bis 2400, Schritt 10, W)
- **Standard:** 450

#### `max_meter_age_s`: Maximales Alter des Zählerwerts

Maximales Alter des externen Zählerwerts, bevor die aktuelle Regelrunde pausiert und den aktuellen GS-Wert beibehält. Ein ausdrücklich ungültiger kritischer Eingang muss mindestens ebenso lange ungültig bleiben, bevor ein GS ungleich 0 einmalig auf 0 W zurückgesetzt wird.

- **Eingabetyp:** Zahl (2 bis 300, Schritt 1, s)
- **Standard:** 30

#### `is_resolution_w`: Schreibauflösung Sollwert max. Wechselrichterleistung

- **Eingabetyp:** Zahl (1 bis 100, Schritt 1, W)
- **Standard:** 10

#### `diagnostic_log_level`: Diagnose-Log-Stufe

Aus schreibt keine Diagnose-Logs. Ereignisse protokolliert SOC-Haltewechsel. Schreibvorgänge ergänzt Setpoint-Schreibvorgänge. Entscheidungen ergänzt periodische Regelentscheidungen. Debug schreibt Entscheidungs-Logs alle 5 Sekunden.

- **Eingabetyp:** Auswahl: Aus, Ereignisse, Schreibvorgänge, Entscheidungen, Debug
- **Standard:** Deaktiviert

#### `diagnostic_log_interval_s`: Intervall für Entscheidungs-Logs

Mindestintervall für periodische Entscheidungs-Logs. Debug- Entscheidungs-Logs verwenden ein festes Intervall von 5 s. Ereignis- und Schreib-Logs werden nur ausgegeben, wenn die jeweilige Aktion tatsächlich passiert.

- **Eingabetyp:** Zahl (10 bis 3600, Schritt 10, s)
- **Standard:** 60

#### `full_charge_hold_helper`: Optionaler Helper für Haltezustand volle Batterie

Optional. Leer lassen, außer die Automatisierung soll den Haltezustand volle Batterie über Neustarts hinweg speichern. Ohne diesen Helper leitet die Automatisierung den Zustand aus dem aktuellen SOC ab, mit einem 1%-Speicherband für volle Batterie im Modus Nach Vollladung PV folgen.

- **Eingabetyp:** Entität (input_boolean)
- **Standard:** Leer / optional

#### `low_discharge_hold_helper`: Optionaler Helper für Niedrig-SOC-Haltezustand

Optional. Leer lassen, außer die Automatisierung soll den Niedrig-SOC- Haltezustand über Neustarts hinweg speichern. Ohne diesen Helper leitet die Automatisierung den Zustand direkt aus dem aktuellen SOC ab.

- **Eingabetyp:** Entität (input_boolean)
- **Standard:** Leer / optional

## Empfohlene Grundeinstellungen

```text
Ziel-Netzleistung: 0 W
Sperrschwelle PV: 1000 W
Freigabeschwelle PV: 700 W
Netzladeleistung XT500 Pro: maximal 2400 W
Diagnose-Log-Stufe für Inbetriebnahme: Entscheidungen oder Debug
```

Passe Leistungsgrenzen, SOC-Grenzen, Batteriekapazität und Zeitfenster immer an das konkrete Gerät und die lokale Installation an.

## Diagnose

Über die Diagnose-Log-Stufe können Ereignisse, Schreibvorgänge, Entscheidungen oder Debug-Ausgaben aktiviert werden. Die Entscheidungsprotokolle enthalten zusätzlich den PV-Sperrzustand sowie Sperr- und Freigabeschwelle.

## Wichtiger Hinweis zur PV-Funktion

Diese Version berücksichtigt die **aktuelle PV-Leistung**. Sie enthält keine Vorhersage zukünftiger PV-Erzeugung und benötigt daher keine Forecast-Integration.

## Lizenz

Siehe Datei `LICENSE` im Repository.
