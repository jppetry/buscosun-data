# glacier/v1/snow — Schneeanalyse je österreichischem Gletscher

Täglich die Schneehöhe und der Schneewasserwert der Analyse SNOWGRID je Gletscher, dazu die Lage des Tageswerts unter den Wintern
1991–2020. Eigenes Produkt der Gletscher-Linie: **buscosun Fusion liest diese Dateien nicht**, und sie stehen nicht in `obs/v1`.

**Analyse auf einem 1-km-Gitter, keine Messung am Gletscher: der Wert steht für eine Zelle von 1 km² mit eigener Modellhöhe.**

**Abdeckung:** SNOWGRID deckt nur Österreich ab. Gletscher in der Schweiz und in Deutschland haben keinen Eintrag. (`coverage: 'AT'`)

| Datei | Inhalt |
|---|---|
| `<yyyymmdd>.json` | ein Tag: je Gletscher `[Schneehöhe cm, Schneewasserwert mm, Lage 0–100 oder null, Kennzeichen]` (Kennzeichen −1 = unter jedem Referenzjahr, 1 = über jedem); `missing` nennt Gletscher ohne Wert mit Grund; 400 Tage aufbewahrt, keine Ablage (die Quelle hält die ganze Reihe) |
| `static/cells/<build>.<n>.json` | welche Gitterzellen zu welchem Gletscher gehören: Zellen, deren Mittelpunkt im Umriss liegt (`how: outline`), sonst die nächste Zelle mit Wert zum Katalogpunkt (`how: nearest`, höchstens 1 km) |
| `static/clim/<build>.<n>/<region>.json` | je Gletscher und Kalendertag (366) die Perzentile 0, 10, 25, 50, 75, 90, 100 der Schneehöhe der Jahre 1991–2020 in cm, ungeglättet; 29.02.: der Tag selbst in Schaltjahren, sonst das Mittel aus 28.02. und 01.03. |

Das Raster selbst liegt hier nicht.

## Quelle und Lizenz

- **GeoSphere Austria** — SNOWGRID Klima v2.1 (Schneehöhe und Schneewasserwert, 1 km, täglich seit 1961). Lizenz: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Quelle: https://data.hub.geosphere.at/dataset/snowgrid_cl-v2-1d-1km, https://doi.org/10.60669/fsxx-6977. Geändert: Schneehöhe von m in cm umgerechnet; je Gletscher das Mittel der Gitterzellen, deren Mittelpunkt im Umriss liegt, sonst die nächste Zelle; Einordnung gegen die Jahre 1991–2020 von buscosun gerechnet.

## Einheiten und Grenzen

Schneehöhe in cm (die Quelle liefert m), Schneewasserwert in mm (kg/m²). Ein Wert außerhalb 0 … 1500 cm bzw. 0 … 8000 mm steht nicht im Dokument.
Je Gletscher EIN Wert: das Mittel seiner Zellen. Ein Gitterwert steht für ein Gebiet, nicht für einen Punkt; Unterschiede zwischen Nachbarzellen sind keine Auflösung.
Die meisten Gletscher sind kleiner als eine Zelle und lesen genau eine; benachbarte kleine Gletscher teilen sich oft dieselbe Zelle und tragen dann denselben Wert.

Code: `buscosun-web` `scripts/glacier/snow.mjs`, Vertrag `src/glacier/glacierSnowContract.ts`.
