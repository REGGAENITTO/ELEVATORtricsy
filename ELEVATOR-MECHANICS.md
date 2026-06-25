# BH Elevator — control mechanics (interactive demo)

> Destination-dispatch (KONE Destination) ovládač výťahov pre Biometrics House.
> Tento súbor popisuje **mechaniku ovládania**, ktorú demonštruje klikateľné HTML demo.
> Stav k **2026-06-25**.

---

## Súbory a spustenie

| Čo | Kde |
|---|---|
| **Klikateľné demo** | `~/Desktop/PROJEKTYclaude/BHscreenELEVATOR/elevator-demo.html` |
| **Pencil zdroj (1:1 vizuál)** | `elevator6upravymeet.pen`, screen **`y4ArJ`** (AI · Tricsy home/parking) |
| **Lokálny server** | `python3 -m http.server 8770` v priečinku projektu → **http://localhost:8770/elevator-demo.html** |
| Assety (wordmarky, dot-marky, maskot) | `floor-names/` (`*_s.png`, `dotb_*.png`, `tricsy3.png`, `bh_logo.png`) |

**Demo NIE je ručne kreslené** — je to **export `y4ArJ` z Pencilu** (`export_html`, html-css, `includeLayerIds`)
+ tenká interakčná vrstva (`<style id="demo-css">` + `<script>` pred `</body>`). Vizuál preto sedí 1:1 s Pencilom.
Prvky sa adresujú cez `data-pencil-id` / `data-pencil-name`.

---

## Budova (poschodia, zhora nadol)

| idx | č. | názov | pozn. |
|---|---|---|---|
| 0 | **3** | NATURE | najvyššie |
| 1 | **2** | FACE | |
| 2 | **1** | FINGER | |
| 3 | **0** | HUB | |
| 4 | **−1** | PARKING | najnižšie |

`idx` = poradie riadku zhora (0 hore). Menší idx = vyššie poschodie.

---

## MECHANIKA — dvojkrokový tok

### Default (idle)
- **Všetkých 5 poschodí biele** (normal, klikateľné), nič modré.
- **Oba výťahy PASÍVNE** na svojich domovských pozíciách:
  - **ľavý** výťah idle na **NATURE** (idx 0)
  - **pravý** výťah idle na **PARKING** (idx 4)
- Horné logo: skryté, kicker „STEP 1 · YOUR FLOOR", podtitul „Tap the floor you are on".

### Krok 1 — KDE SOM (prvý klik)
- Prvý klik na poschodie = **vyberiem, na ktorom poschodí sa nachádzam**.
- Toto poschodie sa **stlmí** (znížený jas — stav „current", opacity ~0.5, tmavé pozadie).
- **Horné logo sa zmení** na názov tohto poschodia (`<key>_s.png`); kicker „YOU ARE HERE",
  podtitul „Step 2 · tap your destination". Logo tu **ostáva**.
- Výťahy zatiaľ spia.

### Krok 2 — KAM IDEM (druhý klik)
- Klik na iné poschodie = **destinácia**; pill sa **zafarbí na modro** (gradient).
- **Aktivuje sa výťah, ktorý je POLOHOU najbližšie k môjmu (current) poschodiu** — príde po mňa.
- Aktívny výťah: modrý gradient kabína + glow + svietiaca linka + krúžky; príde na moje poschodie.
- **Šípka kabíny = smer NASLEDUJÚCEJ jazdy** (nie smer príchodu kabíny):
  - cieľ **vyššie** než ja (menší idx) → šípka **HORE**
  - cieľ **nižšie** → šípka **DOLE**
- Druhý výťah ostáva pasívny na svojej pozícii.
- Horný kicker „ON ROUTE", podtitul „Lift A/B — going UP/DOWN to <CIEĽ>". Karta dole sa updatne (badge + „Take <CIEĽ>").

### Reset
- Tlačidlo **↺ Reset** (vpravo dole) → späť do default (idle).

---

## Pravidlo dispečingu (KĽÚČOVÉ)

> **Aktivuje sa výťah, ktorého POZÍCIA je najbližšie k poschodiu, kde sa nachádzam (current).**
> Cieľom je vyzdvihnúť ma čo najrýchlejšie. NIE podľa blízkosti k cieľu.

```
dl = |currentIdx − leftHome(0)|     // ľavý idle na NATURE
dr = |currentIdx − rightHome(4)|    // pravý idle na PARKING
chosen = (dl <= dr) ? ĽAVÝ : PRAVÝ   // remíza → ľavý
```

Dôsledok pre default pozície (ľavý@NATURE, pravý@PARKING):
- current **NATURE / FACE / FINGER** (hore) → príde **ĽAVÝ**
- current **HUB / PARKING** (dole) → príde **PRAVÝ**

(História: skúšali sme aj „bližšie k cieľu" — Jakub to zamietol. Platí blízkosť k **current** poschodiu.)

---

## Vizuálne stavy (CSS triedy v deme)

**Poschodia (pill):**
- `.dnor` = biele/normal: fill #1B2230, border 2px #43536F, opacity 1.
- `.dcur` = current (kde som): fill #101319, border 1px #23262F, **opacity 0.5**, číslo #6B7280.
- `.dsel` = destinácia: modrý gradient #3B82F6→#1D4ED8, border #60A5FA, glow; badge biely, číslo #1D4ED8.

**Výťahy (lift container):**
- `.la` = active: rail gradient #4C8DF5→#15233B, krúžky #0A1120/stroke #3C5A7C, kabína gradient
  #4D8BF6→#1D4ED8 + border #84B4FF + glow, elevator ikona biela, šípka viditeľná.
- `.lp` = passive: rail #1B222E, krúžky #0A0E16/stroke #262D38, kabína #161B24/border #2C3340, ikona #3D4654, šípka skrytá.
- `.dir-up` na šípke = `rotate(180deg)` (základná SVG je arrow-down).

Pozícia kabíny: `top = idx*160 + 22` (px v rámci lift containera @top:94 v Body). Animuje sa cez CSS transition.

---

## Implementačné gotchas (z exportu Pencilu → CSS)

Pencilov layout engine niektoré veci rieši inak než CSS; v deme to riešia `!important` overridy:
- **`gap` na pilloch** → `gap:0` (Pencilov space_between min-gap sa exportuje ako literálny CSS gap a pretekal — „You are here" mimo pillu).
- **„Your car" / „You are here" tagy** (`carTag`, `youHereTag`) → `display:none` (pretekali; current floor je teraz ktorékoľvek, nie fixne NATURE).
- **Hero-overflow karty** (`tHead` šírka 853 nad maskotom): content (`gtbvI`) zafixovaný `flex:0 0 600px; min-width:0; overflow:visible` → maskot ostane vnútri karty, chip pretečie nad neho.
- **Topbar pilly na celú šírku**: tbLeft (`o7CTu`) → `display:contents` + TopBar `justify-content:space-between` → 4 pilly rovnomerne.
- **Šípka pre ľavý výťah**: pasívna kabína nemala `dir` element → v JS sa **klonuje** z pravého.
- **Dot-marky** pre f3 (NATURE) a fm1 (PARKING) sa **injektujú** v JS (mali tagy namiesto trailu).
- **Horné logo**: `Asset 4.png` = „NATURE + bodky" zlúčené (per-floor kombinované nemáme), preto sa swapuje len wordmark `<key>_s.png` (bez bodiek). TODO ak treba bodky: samostatný dots asset.
- **Render gotcha**: `export_html` referencuje relatívne cesty na assety → demo MUSÍ ostať v priečinku projektu.

---

## TODO / možné rozšírenia
- Po vyzdvihnutí nechať kabínu **pokračovať** animáciou až na cieľové poschodie (teraz príde po mňa + šípka).
- Číslo cieľa do kabíny.
- Horné logo s bodkami (potreba per-floor combined assetov alebo samostatného dots PNG).
- Voliteľne všetky topbar pilly do modra (ako OCCUPY referencia).
- Verejný tunel (cloudflared/ngrok) na náhľad z mobilu.
