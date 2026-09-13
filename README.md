# 🥊 ANIMAL BOXING

Ein völlig unwissenschaftlicher 8-Bit-Kampfsimulator: Wer gewinnt — einer von uns
oder 20 Chihuahuas?

Alles steckt in **einer einzigen Datei**: `index.html`. Kein Build, keine
Dependencies. Doppelklick genügt, oder über GitHub Pages hosten.

## Was drin ist

| | |
|---|---|
| **7 Kämpfer** | Tizi steht, der Rest ist Platzhalter bis die Fotos da sind |
| **10 Rollen** | Statprofil + echte Mechanik, frei zuweisbar |
| **50 Gegner** | mit halbwegs echten Werten (Gewicht, Beisskraft, Tempo, Aggression) |
| **4 Arenen** | Wiese, Dojo, Strand, Parkhaus |
| **Auto-Battle** | Intro-Animation, HP-Balken, Schadenszahlen, Kampflog, Ergebnis-Screen |
| **Comedy-Balancing** | Werte nach Lustigkeit getunt, nicht nach National Geographic |

## Wie die Sprites entstehen

Die Sprites sind nicht von Hand Pixel für Pixel gemalt, sondern werden aus
Primitiven (Ellipse, Rechteck, Dreieck) **gerastert**. Die Baupläne rechnen
in einem 24×24-Design-Raster; die Konstante `R` legt fest, wie viele echte
Pixel eine Design-Einheit bekommt.

```js
const R = 3;   // 72x72 Sprites. R=2 -> 48x48, R=1 -> 24x24
```

`R` hochdrehen erhöht den Detailgrad, ohne dass ein einziger Bauplan
angefasst werden muss — Kurven und Kanten rastern einfach feiner.

Jedes Tier ist ein *Körperbauplan* plus Parameter:

```js
A('wolf', 'Wolf', 'Hunde', 45, 400, 60, 8, 1,
  { n: 'Rudel-Ruf', c: .15, m: 2.1, t: '$A heult — alle Wölfe kriegen kurz Muskeln.' },
  P('#8b8f98', { accent: '#20222a' }),
  () => quad({ brx: 5.8, bry: 2.9, legLen: 5.5, hrx: 2.9, snout: 3,
               ears: 'point', tail: 'bushy' }))
```

Vorhandene Baupläne: `quad` (Vierbeiner), `bird`, `snake`, `insect`, `spider`,
`scorpion`, `primate`, `kangaroo`, `croc`, `human`.

## Die 10 Rollen

Jede Rolle ist nicht nur ein Statprofil, sondern eine echte Mechanik im Kampf.

| Rolle | Kurz | Mechanik |
|---|---|---|
| Der Fels | Tank | nimmt 18 % weniger Schaden |
| Der Blitz | Speedster | doppelte Schlagzahl, 14 % Ausweichchance |
| Die Faust | Bruiser | höchster Grundschaden, Krits +40 % |
| Der Taktiker | Krit-Jäger | 26 % Kritchance statt 10 % |
| Die Wand | Blocker | 22 % Chance auf Komplettblock |
| Das Chaos | Glücksritter | Schadensstreuung 40–190 %, doppeltes Glück und Pech |
| Der Ruhige | One-Punch | halbe Schlagzahl, 2,6-facher Schaden |
| Der Tritt-König | Anti-Schwarm | Rundumtritt trifft einen mehr, 60 % mehr Reichweite |
| Der Schreihals | Support | +14 % Schaden für Verbündete in der Nähe |
| Der Überlebende | Comeback | bis +90 % Schaden bei fast leerer HP-Leiste |

Rollen stehen in `ROLES`, Zuweisung passiert in `FIGHTERS` über den dritten Parameter.

## Glück

Weil ein bisschen Glück dazugehören muss:

* **7 % Fehlschlag** — komplett danebengehauen
* **6 % Glückstreffer** — 2,5-facher Schaden, unabhängig vom kritischen Treffer
* **Das Chaos** hat beide Werte verdoppelt und zusätzlich eine wilde
  Schadensstreuung

Einstellbar über `LUCK`.

## Die echten Kämpfer eintragen

In `index.html` den Block `FIGHTERS` anpassen. Nur das letzte Objekt (`look`)
bestimmt das Aussehen:

```js
F('tizi', 'Tizi', 'chaos', {
  skin: '#f0c49a', hair: '#a8905e', hairStyle: 'buzz', beard: 'stubble',
  shades: 1, shortSleeve: 1, shorts: 1, socks: '#f4f4f6', watch: 1, bag: 1,
  shirt: '#f4f4f6', pants: '#d4bd92', shoe: '#e4e4e8'
})
```

Signatur: `F(id, Name, Rolle, Aussehen)`. Verfügbare Aussehen-Optionen:

* `hairStyle`: `short` `buzz` `long` `bun` `curly` `wild` `mohawk` `bald` `cap`
* `beard`: `full` `stubble` `goatee` `moustache` (oder weglassen)
* Flags: `shades` (Sonnenbrille), `glasses` (normale Brille), `shorts`,
  `shortSleeve` (T-Shirt statt Langarm), `socks`, `bag` (Umhängetasche),
  `watch`, `logo`, `tall`
* Farben: `skin` `hair` `shirt` `pants` `shoe` `lens` `bagCol` `line`
* `build`: `slim` | `normal` | `big`

## Kampfmathematik

* **Werte aus echten Daten**: HP aus dem Gewicht, ATK aus Gewicht + Beisskraft +
  Aggression, DEF aus Gewicht + Panzerung, SPD aus Tempo + Aggression.
* **Körpergrösse bestimmt, wie viele gleichzeitig angreifen dürfen.** Über
  Kreispackung: `n = π / asin(rA / (rT + rA))`. Deshalb können nur ~5 Chihuahuas
  gleichzeitig an einen Menschen ran — und genau deshalb ist 1 gegen 20 gewinnbar,
  1 gegen 50 aber nicht.
* **Rundumschlag**: Wer deutlich grösser ist als sein Ziel, trifft mehrere auf
  einmal. Ein Mensch tritt drei Chihuahuas weg, aber keine drei Wölfe.
  Umgekehrt räumt ein Elefant mit dem Rüssel mehrere Menschen ab.
* **Specials**: jedes Tier und jeder Kämpfer hat eine Comedy-Spezialattacke mit
  Auslösechance und Schadensmultiplikator.

### Aktuelle Siegquoten (aus der Simulation, je 40 Durchläufe)

| Paarung | Team gewinnt |
|---|---|
| 1 Mensch vs 20 Chihuahuas | 100 % |
| 1 Mensch vs 50 Chihuahuas | 5 % |
| 1 Mensch vs 10 Gänse | 45 % |
| 1 Mensch vs 1 Braunbär / Gorilla / Krokodil | 0 / 8 / 0 % |
| 7 Menschen vs 1 Braunbär | 100 % |
| 7 Menschen vs 1 Nashorn / Elefant | 0 % |
| 7 Menschen vs 20 Wölfe | 0 % |

Nachtunen lässt sich das über `TUNE`, `SWEEP_RATIO`, die Formeln in
`animalStats()` und den `fun`-Multiplikator pro Tier.

## Balancing selbst testen

Die Engine läuft auch ohne Grafik. In der Browser-Konsole:

```js
fastSim([FIGHTERS[0]], [{ def: ANIMALS.find(a => a.id === 'chihuahua'), n: 20 }])
```

## Noch offen

* **Gegenstände zum Verteidigen** (Pfanne, Regenschirm, Gartenstuhl …) — als
  Modifier auf ATK/DEF/Reichweite gedacht
* Echte Avatare statt Platzhalter
* Arena-Auswahl ist gebaut (Dropdown im Gegner-Screen), weitere Arenen sind je
  ein Eintrag in `ARENAS`

---

Keine Tiere wurden bei der Erstellung dieser Seite verletzt. Sie sind alle nur Pixel.
