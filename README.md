# Prognoza FV

A tiny, single-file web app that estimates solar (PV) production for the next few days — mainly so I know which afternoon is worth running the washing machine.

No build step, no backend, no API keys. One HTML file, hosted as a static page.

## What it does

For **today, tomorrow, and the day after**, shows:
- Estimated energy production in **kWh**, plus a rough **% of a fully clear day**.
- A small chart for each day, plotting *predicted* production against a *theoretical clear-sky maximum*, hour by hour, in kW.

## Data source

Cloud-aware solar irradiance comes from the [Open-Meteo](https://open-meteo.com/) API — free, no key required, called directly from the browser. Specifically, it uses **Global Tilted Irradiance (GTI)**: Open-Meteo does the full transposition of solar radiation onto a tilted, oriented surface for you (accounting for real sun position, season, and forecasted cloud cover), given your panels' tilt and azimuth.

## Settings

- **Coordinates** of the panels (paste `lat, lon` straight from Google Maps).
- **Tilt** — panel angle off horizontal, 0–90°.
- **Azimuth** — in the **Open-Meteo convention**: 0° = south, -90° = east, 90° = west, ±180° = north (not the compass bearing convention where 0° = north — to convert a compass bearing, subtract 180°).
- **Installed power (kWp)**.

There's no manual "sun-visible hours" setting — the relevant hourly window is derived automatically from the azimuth (see below).

## How the estimate works (and its limits)

This is **intentionally simplified**, hardcoded, and commented for future refinement rather than configurable — see the top of the `<script>` block in `index.html`.

**Which hours are shown.** The 7-hour window is centered on an hour derived from azimuth: `12 + floor(azimuth / 30)`, shifting earlier for east-facing panels and later for west-facing ones. Because Open-Meteo's GTI values are a *"preceding hour mean"* (the value labeled `14:00` is the average over `13:00–14:00`), the loop actually spans from `-3` to `+4` around that center to correctly cover 7 full one-hour intervals. This assumes tilted (not flat/horizontal) panels — flat panels would need a different, wider window; left for a future version.

**Converting W/m² to kWh.** `kWh ≈ kWp × (GTI / 1000) × PERFORMANCE_RATIO`. This follows directly from how kWp itself is defined (rated power at 1000 W/m², STC conditions). `PERFORMANCE_RATIO` (default 0.80) accounts for typical system losses (inverter, wiring, temperature, soiling) and should be tuned once you have a few days of real meter readings to compare against.

**The "% of clear day" figure.** Open-Meteo doesn't expose a ready-made "GTI at clear sky" value, so this is a deliberate shortcut rather than rigorous physics: it compares actual GTI against `terrestrial_radiation` (theoretical top-of-atmosphere irradiance, which Open-Meteo already computes from exact sun position) scaled by one hardcoded constant, `CLEAR_SKY_FACTOR` (default 0.75 — roughly how much of that theoretical radiation reaches the ground on a genuinely clear day). Because this reference is for a horizontal plane while GTI is for the tilted one, the result is clamped at 100% rather than being exact.

## Using it

1. Open the page and fill in settings (coordinates, tilt, azimuth, kWp).
2. Settings are stored in the browser's `localStorage` — nothing is sent anywhere except the forecast request to Open-Meteo.
3. Tap the gear icon any time to change settings; tap "Odśwież prognozę" to refetch.

## Adding it to your phone

Open the page in Chrome on Android and use **⋮ → Add to Home screen**. The app ships its own icon and a web manifest, so it gets a proper icon and name instead of a generic browser bookmark.

## Tech

Plain HTML/CSS/JavaScript, no framework, no dependencies, no charting library (the charts are hand-drawn inline SVG). Hosted on GitHub Pages.

## Status / known simplifications

Personal / hobby project. Known rough edges, in rough order of what to tackle next:
- `PERFORMANCE_RATIO` and `CLEAR_SKY_FACTOR` are single hardcoded guesses — worth calibrating against real production data.
- The hour-window formula assumes tilted panels; flat/horizontal panels aren't handled yet.
- The "% of clear day" comparison uses a horizontal reference for a tilted-plane value — a known, accepted approximation, not exact.

---

# Prognoza FV (PL)

Bardzo prosta, jednoplikowa aplikacja webowa szacująca produkcję instalacji fotowoltaicznej na najbliższe dni — głównie po to, żeby wiedzieć, które popołudnie warto przeznaczyć na pranie.

Bez budowania, bez backendu, bez kluczy API. Jeden plik HTML, hostowany jako zwykła strona statyczna.

## Co robi

Dla **dziś, jutro i pojutrze** pokazuje:
- Szacowaną produkcję energii w **kWh**, plus orientacyjny **procent w pełni bezchmurnego dnia**.
- Mały wykres dla każdego dnia, zestawiający *prognozowaną* produkcję z *teoretycznym maksimum przy bezchmurnym niebie*, godzina po godzinie, w kW.

## Źródło danych

Prognoza nasłonecznienia (uwzględniająca zachmurzenie) pochodzi z API [Open-Meteo](https://open-meteo.com/) — darmowego, bez klucza, wywoływanego bezpośrednio z przeglądarki. Konkretnie używane jest **Global Tilted Irradiance (GTI)**: Open-Meteo samo wykonuje pełną transpozycję promieniowania na nachyloną, zorientowaną powierzchnię (uwzględniając realną pozycję słońca, porę roku i prognozowane zachmurzenie), na podstawie podanego kąta nachylenia i azymutu paneli.

## Ustawienia

- **Współrzędne** paneli (wklej `lat, lon` prosto z Google Maps).
- **Kąt nachylenia** — 0–90° od poziomu.
- **Azymut** — w **konwencji Open-Meteo**: 0°=południe, -90°=wschód, 90°=zachód, ±180°=północ (nie konwencja kompasowa, gdzie 0°=północ — żeby przeliczyć azymut kompasowy, odejmij 180°).
- **Moc instalacji (kWp)**.

Nie ma osobnego pola na "godziny widoczności słońca" — okno godzinowe jest wyliczane automatycznie z azymutu (patrz niżej).

## Jak działa szacowanie (i jakie ma ograniczenia)

To celowo uproszczony, zahardkodowany i skomentowany model do dopracowania w przyszłości, a nie coś w pełni konfigurowalnego — szczegóły na górze bloku `<script>` w `index.html`.

**Które godziny są pokazywane.** 7-godzinne okno jest wyśrodkowane na godzinie wyliczonej z azymutu: `12 + floor(azymut / 30)` — przesuwa się wcześniej dla paneli zwróconych na wschód, później dla zwróconych na zachód. Ponieważ wartości GTI z Open-Meteo to *"preceding hour mean"* (wartość podpisana `14:00` to średnia z przedziału `13:00–14:00`), pętla faktycznie biegnie od `-3` do `+4` względem tej godziny centralnej, żeby poprawnie pokryć 7 pełnych przedziałów godzinowych. Zakłada to panele nachylone (nie leżące płasko) — płaskie panele wymagałyby innego, szerszego okna; zostawione na przyszłą wersję.

**Przeliczenie W/m² na kWh.** `kWh ≈ kWp × (GTI / 1000) × PERFORMANCE_RATIO`. Wynika to wprost z definicji kWp (moc znamionowa przy 1000 W/m², warunki STC). `PERFORMANCE_RATIO` (domyślnie 0,80) uwzględnia typowe straty systemowe (falownik, okablowanie, temperatura, zabrudzenie) i warto go doszlifować po zebraniu kilku dni prawdziwych odczytów z licznika.

**Procent "bezchmurnego dnia".** Open-Meteo nie udostępnia gotowej wartości "GTI przy bezchmurnym niebie", więc to świadomy skrót, a nie ścisła fizyka: rzeczywiste GTI jest porównywane z `terrestrial_radiation` (teoretyczne promieniowanie poza atmosferą, które Open-Meteo i tak liczy z dokładnej pozycji słońca), przeskalowanym przez jedną zahardkodowaną stałą `CLEAR_SKY_FACTOR` (domyślnie 0,75 — w przybliżeniu tyle z tego teoretycznego promieniowania dociera do ziemi w naprawdę bezchmurny dzień). Ponieważ ten punkt odniesienia dotyczy płaszczyzny poziomej, a GTI nachylonej, wynik jest przycinany do 100%, zamiast być ścisły.

## Jak używać

1. Otwórz stronę i uzupełnij ustawienia (współrzędne, kąt, azymut, kWp).
2. Ustawienia zapisują się w `localStorage` przeglądarki — nic nigdzie nie jest wysyłane poza samym zapytaniem o prognozę do Open-Meteo.
3. Ikona zębatki otwiera ustawienia w dowolnym momencie; "Odśwież prognozę" pobiera dane na nowo.

## Dodanie na telefon

Otwórz stronę w Chrome na Androidzie i wybierz **⋮ → Dodaj do ekranu głównego**. Aplikacja ma własną ikonę i manifest, więc dostanie właściwą ikonę i nazwę zamiast domyślnego skrótu przeglądarki.

## Technologia

Czysty HTML/CSS/JavaScript, bez frameworka, bez zależności, bez biblioteki do wykresów (wykresy to ręcznie rysowane inline SVG). Hostowane na GitHub Pages.

## Status / znane uproszczenia

Projekt osobisty/hobbystyczny. Znane niedoskonałości, w przybliżonej kolejności do ewentualnego dopracowania:
- `PERFORMANCE_RATIO` i `CLEAR_SKY_FACTOR` to pojedyncze, zahardkodowane szacunki — warto skalibrować względem prawdziwych danych produkcyjnych.
- Wzór na okno godzinowe zakłada panele nachylone; płaskie/poziome nie są jeszcze obsłużone.
- Porównanie "% bezchmurnego dnia" używa poziomego punktu odniesienia dla wartości z nachylonej płaszczyzny — znane, zaakceptowane przybliżenie, nie ścisłość.
