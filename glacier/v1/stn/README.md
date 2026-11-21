# glacier/v1/stn — Stationsmessungen in der Nähe der Gletscher

Alle 30 Minuten aus zwei Messnetzen, dazu die Werte der zugeordneten Stationen aus `obs/v1` (DWD, GeoSphere Austria, MeteoSchweiz).
Eigenes Produkt der Gletscher-Linie: **buscosun Fusion liest diese Dateien nicht** (der Motor liest Messungen nur aus `obs/v1/`).

| Datei | Inhalt |
|---|---|
| `stations.json` | Katalog der zwei Netze (Form von `obs/v1/stations.json`, dazu `operator`) |
| `h/<slot>.json` | Werte eines Laufs je Station (Form eines Eintrags von `obs/v1/latest.json`); der Slot heißt `yyyymmddhh` für die volle Stunde und `yyyymmddhh30` für die halbe; 72 h aufbewahrt; dauerhaft je Stunde eine Datei in `buscosun-archiv` `glacier/v1/<Tag>/stn-<slot>.json.gz` |
| `map/<build>.<n>.json` | Gletscher → nächste Station je Größe (Abstand ≤ 5 km, Stationshöhe in der Höhenspanne des Gletschers ±300 m) |

## Quellen und Lizenzen

- **Datenquelle: Land Tirol - data.tirol.gv.at** — Wetterstationsdaten Tirol (Lawinenwarndienst Tirol, Hydrographischer Dienst Tirol). Lizenz: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Quelle: https://www.data.gv.at/datasets/bb43170b-30fb-48aa-893f-51c60d27056f. Geändert: Wind und Böen von km/h in m/s umgerechnet; Werte außerhalb benannter Grenzen, mit Sprung oder Stillstand und Werte älter als drei Stunden weggelassen.
- **Autonome Provinz Bozen – Südtirol** — Wetterdaten der Messstationen Südtirols. Lizenz: [CC0](https://creativecommons.org/publicdomain/zero/1.0/). Quelle: https://data.civis.bz.it/de/dataset/misure-meteo-e-idrografiche. Geändert: Niederschlag der 5-min-Werte zu Summen addiert; Werte außerhalb benannter Grenzen, mit Sprung oder Stillstand und Werte älter als drei Stunden weggelassen.
- **DWD · GeoSphere Austria · MeteoSchweiz** — Messnetze DWD, GeoSphere Austria (TAWES) und MeteoSchweiz (SwissMetNet). Lizenz: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). Quelle: https://github.com/jppetry/buscosun-data/tree/main/obs/v1. Geändert: aus dem Messspiegel von buscosun übernommen, Werte unverändert.

Aus dem Feed `static.avalanche.report` wird nur gelesen, wenn die OGD-Datei des Landes Tirol nicht antwortet, und dann nur Stationen der Betreiber `HD Tirol` und `LWD Tirol…`. Kein anderer Betreiber wird gespeichert.

## Einheiten und Prüfungen

Einheiten wie in `obs/v1`: `t` °C Lufttemperatur, `td` °C Taupunkt, `rh` % relative Feuchte, `ff` m/s Wind (Mittel), `dd` ° Windrichtung, `fx` m/s Böe, `snow` cm Schneehöhe, `gr` W/m² Globalstrahlung; Summen `rr1h`, `rr6h`, `rr24h` in mm.

Die Rohdaten sind ungeprüft. Ein Wert steht nur im Dokument, wenn er jünger als 3 Stunden ist, in der benannten Spanne liegt (t -45 … 40 · td -60 … 30 · rh 1 … 105 · ff 0 … 75 · dd 0 … 360 · fx 0 … 90 · snow -5 … 1000 · gr -10 … 1500 · rr1h 0 … 150 · rr6h 0 … 400 · rr24h 0 … 700), gegen den letzten Wert der Station nicht springt und nicht seit 6 Stunden stillsteht. Was herausfällt, steht mit Grund unter `rejected`.

Stationen stehen auf Fels oder Grat, nicht auf dem Eis.

Code: `buscosun-web` `scripts/glacier/stations.mjs`, Vertrag `src/glacier/glacierStationContract.ts`.
