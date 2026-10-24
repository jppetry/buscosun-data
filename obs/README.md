# buscosun-data — Stationsmessungen `obs/`

Lizenz: DWD (CC BY 4.0 / GeoNutzV), GeoSphere Austria (CC BY 4.0), MeteoSchweiz (CC BY 4.0). Die Daten wurden umgerechnet (Einheiten) und neu angeordnet; nicht für amtliche Zwecke.


Jede offen zugängliche, real messende Station im DACH-Raum mit ihren **aktuellen Messwerten**, auch die
Stationen, die nur Niederschlag messen. Keine Modellwerte, kein Auffüllen von Lücken, keine Interpolation: was
die Quelle liefert, steht da, umgerechnet nur in einheitliche Einheiten. Spiegel: `scripts/obs-mirror.mjs`
(Kopie aus `buscosun-web/scripts/obs/obs-mirror.mjs`, ohne Abhängigkeiten), Workflow `.github/workflows/obs.yml`
(Dauerlauf 340 min, startet seinen Nachfolger selbst) + `obs-watchdog.yml` (alle 20 min: startet ihn, falls keiner
läuft). Aus: Repo-Variable `OBS_KILL=1`.

| Netz | Quelle | Stationen | Takt der Quelle (gemessen 07.10.2026) | Abruf |
|---|---|---|---|---|
| `dwd10` | DWD CDC 10 min `now` (Temperatur/Feuchte/Druck, Niederschlag, Wind, Böen, Sonne/Strahlung) | ≈ 1 430 (davon ≈ 900 nur Niederschlag) | halbstündlich je Produkt: Niederschlag :10/:40, Wind/Böen/Sonne :15/:45, Temperatur :20/:50; Werte bis ≈ 30 min alt | :11 :16 :21 :41 :46 :51, Wiederholung alle 3 min |
| `dwdDay` | DWD CDC Niederschlag täglich (`more_precip/recent`, haupt- und nebenamtliche Niederschlagsstationen) | ≈ 2 270 | einmal am Tag (≈ 09:15 UTC), Wert des Vortags | stündlich :17 (nur Verzeichnis, geänderte Dateien) |
| `tawes` | GeoSphere Austria TAWES (`tawes-v1-10min`) | 286 | alle 10 min, Stempel ≈ 1 min danach | :02 :12 … :52, Wiederholung alle 2 min; holt die letzten Stunden neu (heilt) |
| `klima` | GeoSphere `klima-v2-10min`, nur Stationen > 3 km von jeder TAWES-Station | 3 (1 liefert) | wie TAWES | mit TAWES |
| `smn` | MeteoSchweiz SwissMetNet: OGD `ogd-smn` + Sammeldatei `VQHA80` | 159 | OGD alle 20 min (:08/:28/:48), VQHA80 alle 10 min (:x0, Stempel 10 min alt); am selben Stempel wertgleich (37/37 geprüft) | :01 :09 :11 :21 :29 :31 :41 :49 :51 (bedingter Abruf) |
| `smnp` | MeteoSchweiz automatische Niederschlagsstationen: `ogd-smn-precip` + `VQHA98` | 141 | wie `smn` | wie `smn` |
| `nime` | MeteoSchweiz manuelle Niederschlagsstationen (`ogd-nime`) | 270 | einmal am Tag (≈ 11:20 UTC), Wert des Vortags | stündlich :25 (bedingter Abruf) |

```
obs/v1/stations.json         Katalog: id, name, lat, lon, elev, country (DE/AT/CH/LI), region, networks, vars
obs/v1/latest.json           je Station der neueste Stempel mit Werten (v), ältere Einzelwerte ≤ 60 min (older),
                             rr1h/rr24h mit Vollständigkeit (n von of), der neueste Tageswert (day)
obs/v1/series/<netz>.json    Reihe je Station und Größe auf gemeinsamer Zeitachse (t0, step, n; null = kein Wert):
                             10-min-Netze die letzten 26 h, Tagesnetze die letzten 10 Tage
obs/v1/status.json           je Quelle letzter Abruf, neuester Stempel, Stationen mit Werten, Fehler; Zählung je Land/Netz
obs/v1/state.json            Last-Modified je Datei (nur für den Spiegel)
```

**Kennungen.** `de:<DWD-Stationskennung>` (dieselbe Kennung in `dwd10` und `dwdDay` ist derselbe Ort, 0 km
geprüft), `at:<TAWES-Kennung>` bzw. `at:k<klima-Kennung>`, `ch:<MeteoSchweiz-Kürzel>` (Kürzel über `smn`/`smnp`/`nime`
hinweg derselbe Ort, 0 km geprüft). Im Katalog steht nur, wer aktuell geliefert wird; wer drei Tage in keiner
Quelle mehr steht, fällt heraus.

**Größen und Einheiten.** `t` °C · `td` °C · `rh` % · `ps` hPa Stationshöhe · `p` hPa reduziert (QFF/PRED, Bezug je
Netz verschieden) · `ff` m/s 10-min-Mittel · `dd` ° · `fx` m/s Böenspitze im Intervall · `rr` mm im Intervall ·
`sd` min Sonnenschein im Intervall · `gr` W/m² Globalstrahlung · `snow` cm Schneehöhe · `nsnow` cm Neuschnee.
Umgerechnet: DWD `SD_10` h → min, `GS_10` J/cm² → W/m², GeoSphere `SO` s → min, VQHA-Wind km/h → m/s.

**Stempel.** UTC, **Ende** des 10-min-Intervalls. Tageswerte: Datum D = DWD 05:50 UTC D … 05:50 UTC D+1 (an
11 Tagen gegen die 10-min-Summen geprüft), MeteoSchweiz 06 UTC D … 06 UTC D+1 (Parameterbeschreibung).

**Nicht dabei** (nicht offen oder nicht aktuell): Niederschlagsstationen der hydrographischen Dienste in
Österreich, Landes- und Privatnetze, MeteoSchweiz-Messtürme (`ogd-smn-tower`) und Totalisatoren (`ogd-tot`),
DWD-Stationen ohne freie Abgabe. Die Werte sind ungeprüft so, wie die Dienste sie veröffentlichen (vorläufig,
kann sich nachträglich ändern; z. B. Schneehöhe −1 cm vom Sensor).

