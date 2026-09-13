# 🥊 ANIMAL BOXING

Ein völlig unwissenschaftlicher 8-Bit-Kampfsimulator: Wer gewinnt — einer von uns
oder 20 Chihuahuas?

Alles steckt in **einer einzigen Datei**: `index.html`. Kein Build, keine
Dependencies. Doppelklick genügt, oder über GitHub Pages hosten.

## Was drin ist

| | |
|---|---|
| **7 Kämpfer** | aktuell Platzhalter, werden gegen die echten Fotos getauscht |
| **50 Gegner** | mit halbwegs echten Werten (Gewicht, Beisskraft, Tempo, Aggression) |
| **4 Arenen** | Wiese, Dojo, Strand, Parkhaus |
| **Auto-Battle** | Intro-Animation, HP-Balken, Schadenszahlen, Kampflog, Ergebnis-Screen |
| **Comedy-Balancing** | Werte nach Lustigkeit getunt, nicht nach National Geographic |

## Wie die Sprites entstehen

Die Sprites sind nicht von Hand Pixel für Pixel gemalt, sondern werden aus
Primitiven (Ellipse, Rechteck, Dreieck) auf ein 24×24-Raster **gerastert**.
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

## Die echten Kämpfer eintragen

In `index.html` den Block `FIGHTERS` anpassen. Nur das letzte Objekt (`look`)
bestimmt das Aussehen:

```js
F('p1', 'Ole', 'Der Fels', 260, 18, 24, 10,
  { n: 'Breitbeiniger Stand', c: .18, m: 1.9, t: '$A steht einfach da. $B prallt ab.' },
  { skin: '#e8b48c', hair: '#3a2a1e', hairStyle: 'buzz', beard: 'full',
    shirt: '#3a6ed0', pants: '#22304e', build: 'big' })
```

* `hairStyle`: `short` `buzz` `long` `bun` `curly` `wild` `mohawk` `bald` `cap`
* `beard`: `full` `stubble` `goatee` `moustache` (oder weglassen)
* `glasses: 1`, `logo: 1`, `build`: `slim` | `normal` | `big`, `tall: 1`
* Die Zahlen sind HP / ATK / DEF / SPD.

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
