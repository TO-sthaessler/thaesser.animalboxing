# 🥊 ANIMAL BOXING

Ein völlig unwissenschaftlicher 8-Bit-Kampfsimulator: Wer gewinnt — einer von uns
oder 20 Chihuahuas?

Alles steckt in **einer einzigen Datei**: `index.html`. Kein Build, keine
Dependencies. Doppelklick genügt, oder über GitHub Pages hosten.

## Was drin ist

| | |
|---|---|
| **7 Kämpfer** | vollzählig |
| **10 Rollen** | Statprofil + echte Mechanik, frei zuweisbar |
| **50 Gegner** | mit halbwegs echten Werten (Gewicht, Beisskraft, Tempo, Aggression) |
| **8 Arenen** | animiert, mit Vordergrund-Ebene |
| **Auto-Battle** | Intro-Animation, HP-Balken, Schadenszahlen, Kampflog, Ergebnis-Screen |
| **Steuerung** | Pause, 1x/2x/4x, Foto — per Button oder Taste |
| **8-Bit-Sound** | komplett per WebAudio erzeugt, keine Dateien |
| **Zeitlupe** | der letzte K.O. läuft auf ein Fünftel, Kamera fährt rein |
| **22 Gegenstände** | jeder Kämpfer bringt einen mit, mit echten Auswirkungen |
| **Abschlussbericht** | teilbare Bilanz als Bild und Text, direkt übers Sharesheet |
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
Rollen dürfen doppelt vergeben werden.

### Vergeben

| Kämpfer | Rolle |
|---|---|
| Tizi | Der Blitz (Speedster) |
| Ole | Der Schreihals (Support) |
| Konrad | Der Ruhige (One-Punch) |
| Hirschi | Der Taktiker (Krit-Jäger) |
| Marsn | Der Tritt-König (Anti-Schwarm) |
| Sascha | Das Chaos (Glücksritter) |
| Roi | Die Wand (Blocker) |

Unbesetzt geblieben: Fels, Faust, Überlebender — stehen für künftige
Umbesetzungen bereit.

## Arenen

Eigener Schritt im Setup ("3. Arena wählen") mit echten Vorschaubildern —
jede Kachel ist die Arena selbst, in den Offscreen-Puffer gerendert und
verkleinert. Jede Arena hat eine Hintergrund-Ebene (`deco`)
und optional eine Vordergrund-Ebene (`fore`), die **vor** den Kämpfern
läuft. Beide bekommen eine Zeitachse, sind also animiert.

| Arena | Was sich bewegt |
|---|---|
| **Geisterbahn** | Blitz alle vierzehn Sekunden mit Nachzucken, flackernde Fenster, schwebende Geister, Fledermäuse, Bodennebel |
| **Strand** | anlaufende Wellen, kreisende Möwen mit Flügelschlag, Brandungssaum |
| **Fußballstadion** | Laola-Welle läuft durch sechs Ränge, Fans reissen die Arme hoch, Flutlichtkegel |
| **Innenstadt** | Passanten laufen im Vordergrund durchs Bild, Ampel schaltet, Fenster gehen an und aus |
| **Wolkenkratzer** | Hubschrauber zieht alle elf Sekunden vorbei, Rotor dreht, Wolken ziehen, Antennenlicht blinkt, Wolkenfetzen im Vordergrund |
| Wiese | Wolken ziehen |
| Dojo | statisch |
| Parkhaus | flackernde Neonröhren |

Neue Arena = ein Eintrag in `ARENAS`:

```js
meinearena: {
  name: 'Meine Arena', sky: ['#oben', '#unten'],
  ground: '#boden', groundDark: '#bodenkante',
  deco(c, W, H, hz, t) { /* hinter den Kämpfern */ },
  fore(c, W, H, hz, t) { /* davor, optional */ }
}
```

`t` läuft in Sekunden durch und stoppt nie — auch nicht zwischen den Kämpfen.

## Protokolle

Auf dem Ergebnis-Screen stehen zwei Protokolle:

**Kampfprotokoll** — der vollständige Kampflog zum Nachlesen, nichts
abgeschnitten. Im Kampf selbst wird die Anzeige bei 120 Zeilen gekappt,
damit sie flüssig bleibt; hier steht alles.

**Balancing-Log** — ein technisches Log zum Nachjustieren. Enthält die
aktuellen Stellschrauben (`TUNE`, `LUCK`, `SWEEP_RATIO`) samt Formeln,
eine Tabelle pro Kämpfer (Werte, Rest-HP, ausgeteilter und kassierter
Schaden, K.O.s, Angriffe, Trefferquote), dieselbe Tabelle für die Gegner
nach Art zusammengefasst, Gesamtzahlen inklusive Schaden pro Sekunde je
Seite, und am Ende alle Ereignisse. Zum Kopieren oder als `.txt`.

Das Log macht Zielkonflikte sichtbar, die man sonst nur ahnt — etwa dass
ein Kämpfer mit Gartenstuhl (+14 DEF, −4 SPD) in elf Sekunden nur zweimal
zum Schlag kommt.

## Nicht indexieren

Die Seite trägt `noindex, nofollow, noarchive, nosnippet, noimageindex`
als Meta-Tag, dazu `referrer: no-referrer`. Suchmaschinen, die sich daran
halten, nehmen sie nicht auf.

Die beiliegende `robots.txt` greift **nur**, wenn die Seite unter einer
eigenen Domain im Wurzelverzeichnis liegt. Bei GitHub Pages als
Projektseite (`…github.io/thaesser.animalboxing/`) wird sie unter einem
Unterpfad ausgeliefert und dort von Crawlern ignoriert — verlassen könnt
ihr euch dort allein auf das Meta-Tag. Wer die Seite wirklich privat
halten will, legt sie nicht auf eine öffentliche URL.

## Steuerung im Kampf

| | |
|---|---|
| **Pause** | Button oder Leertaste. Friert Kampf **und** Arena ein. |
| **1x / 2x / 4x** | Button oder Taste `1`, `2`, `4` |
| **Foto** | speichert das aktuelle Bild als PNG — zusammen mit Pause ein Standbild vom besten Moment |

## Sound

Alle Geräusche werden zur Laufzeit per WebAudio synthetisiert — Rechteck-
und Dreieckwellen plus gefiltertes Rauschen. Keine Audiodateien, die Seite
bleibt eine einzelne HTML.

Treffer, kritische Treffer, Specials, Blocks, Ausweichen, Fehlschläge,
K.O.s, Kampfstart, Sieg, Niederlage und ein Klick auf jedem Button. Die
Tonhöhe eines Treffers sinkt mit dem Schaden, und Treffer sind auf einen
alle 45 ms gedrosselt, damit ein Schwarm nicht in Krach ausartet.

Wichtig für die Architektur: **die Kampfschleife macht selbst keine
Geräusche.** Der Ton hängt an den Effekten, die sie erzeugt
(`sfxFromFx()`). Dadurch bleibt `fastSim()` lautlos und kann weiter
tausende Kämpfe in Sekunden durchrechnen.

Der Ton-Button steht in der Kampfleiste, die Einstellung überlebt in
`localStorage`. Der AudioContext wird erst beim ersten Klick geöffnet,
weil Browser ihn sonst blockieren.

## Siegesfeier

Nach der Zeitlupe feiern die Überlebenden: sie sammeln sich in Reihen in
der Bildmitte und hopsen versetzt, dazu Konfetti und eine Fanfare.

Menschen reissen dabei die Arme hoch — dafür hat der Mensch-Bauplan eine
zweite Pose (`armsUp`). Sie wird als eigener Sprite gebaut und im
Cache gehalten, Ärmel, Uhr und Tattoo wandern korrekt mit. Tiere hopsen
nur; ein springender Braunbär reicht auch so.

Der Abstand der Figuren richtet sich nach der grössten beteiligten Figur,
sonst stehen zwei Bären ineinander.

Gewinnen die Tiere, feiern eben sie — die Feier gehört der Seite, die
übrig ist.

## Zeitlupe

Wenn der letzte Gegner fällt, schaltet das Spiel in die Phase `slowmo`:
Zeit auf ein Fünftel, Kamera fährt anderthalb Sekunden lang auf den Ort
des letzten K.O. zu, dazu eine Vignette. Danach erst kommt das K.O.-Banner.

Technisch wird die Szene in dieser Phase in einen Offscreen-Puffer
gezeichnet und vergrössert aufs sichtbare Canvas geblittet. Der Fokuspunkt
wird so begrenzt, dass das vergrösserte Bild das Canvas immer noch füllt.
Bildglättung bleibt aus, die Pixel werden also grob statt matschig.

## Gegenstände

Beim Team-Setup bringt jeder Kämpfer genau einen Gegenstand mit. Sie sind
keine Deko: die Werte auf der Karte aktualisieren sich sofort, der
Gegenstand wird im Kampf in der Hand mitgeführt und steht im Bericht.

| Gegenstand | Wirkung |
|---|---|
| Bratpfanne | +8 ATK, −2 SPD |
| Regenschirm | +10 DEF, +10 % Block |
| Gartenstuhl | +14 DEF, +3 ATK, −4 SPD |
| Mülltonnendeckel | +12 DEF, +8 % Block |
| Bauhelm | +8 DEF, +30 HP |
| Nudelholz | +7 ATK |
| Bierbank | +13 ATK, +5 DEF, −6 SPD |
| Wanderstock | +4 ATK, mehr Reichweite |
| Selfie-Stick | +2 ATK, +3 SPD, viel Reichweite |
| Laubbläser | +4 SPD, ein Ziel mehr beim Rundumschlag |
| Fliegenklatsche | +1 ATK, zwei Ziele mehr beim Rundumschlag |
| Grillzange | +5 ATK, +3 SPD |
| Schneeschaufel | +7 ATK, +4 DEF, −2 SPD |
| Klobürste | +2 ATK, +10 % Ausweichen |
| Gummihuhn | +1 ATK, doppelte Glückschance |
| Trillerpfeife | das ganze Team macht +10 % Schaden |
| Stacheldraht-Weste | +4 DEF, 25 % Schaden zurück an den Angreifer |
| Tiefkühlpizza | +3 ATK, +25 HP |
| Handtasche | +9 ATK, −1 SPD |
| Wurfstern aus Pappe | +5 ATK, +12 % Krit |
| Sprudelkiste | +10 ATK, +3 DEF, −5 SPD |
| Gartenzwerg | +6 ATK, +6 DEF |

Die schweren Sachen sind ein echter Zielkonflikt: Bierbank und Sprudelkiste
hauen hart, kosten aber so viel Tempo, dass man gegen einen einzelnen
grossen Gegner schlechter fährt als mit blanken Fäusten.

Im Aufklappmenü stehen sie alphabetisch, „— nichts —" bleibt vorn.

Neue Gegenstände kommen in `ITEMS`. Das Feld `mod` versteht `atk`, `def`,
`spd`, `hp`, `reach`, `sweep`, `block`, `dodge`, `crit`, `luck`, `thorns`
und `aura`.

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
* Flags: `shades` (Sonnenbrille), `glasses` (kräftiger Rahmen), `shorts`,
  `shortSleeve` (T-Shirt statt Langarm), `socks`, `bag` (Umhängetasche),
  `watch`, `earring`, `tattoo` (Unterarm), `logo`, `tall`
* `shirtPat`: `diag` (Trikot mit Diagonalen), `stripes`, `check` (Karohemd)
  oder `panel` (andersfarbige Partie auf einer Seite). Liegt über Rumpf
  und Ärmeln.
* `hood: 1` für Kapuzenpulli (Kapuze im Nacken plus Kordeln)
* `shadesUp: 1` für die auf den Kopf geschobene Sonnenbrille
* `hat: 'tracht'` für den Trachtenhut mit Kordel und Feder
  (`hatCol`, `hatBand`)
* `jacket: 1` für einen offenen Janker über dem Hemd (`jacketCol`) —
  färbt auch die Ärmel um
* `sash: 1` für eine Bauchbinde (`sashCol`)
* `hairStyle` zusätzlich: `fringe` (Pony)
* Farben: `skin` `hair` `beardCol` `shirt` `shirt2` `shirt3` `pants` `shoe`
  `lens` `bagCol` `tattooCol` `jewel` `line`

`beard: 'stubble'` wird als 50-Prozent-Dither auf echten Pixeln gezeichnet,
nicht als Block — sieht dadurch wie Bartschatten aus statt wie ein Balken.
* `build`: `slim` | `normal` | `big`

Ole als Beispiel für die neuen Optionen:

```js
F('ole', 'Ole', 'support', {
  skin: '#f2cba8', hair: '#d4739e', hairStyle: 'buzz', beard: 'moustache',
  beardCol: '#a87545', glasses: 1, earring: 1, tattoo: 1, shortSleeve: 1, shorts: 1,
  shirt: '#1f6b4a', shirtPat: 'diag', shirt2: '#f0f0f2', shirt3: '#191a1f',
  pants: '#1c1c22', shoe: '#26262e'
})
```

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

## Abschlussbericht

Nach jedem Kampf gibt es eine Bilanz — als NES-Karte gerendert und als
Klartext. Drei Buttons auf dem Ergebnis-Screen:

* **Bericht teilen** — öffnet das native Sharesheet (`navigator.share`).
  Wo das Gerät Dateien unterstützt, geht das Bild mit raus, sonst nur der
  Text. Ohne Sharesheet fällt es auf die Zwischenablage zurück, und wenn
  auch die blockiert ist, erscheint der Text in einem Feld zum Markieren.
* **Bild speichern** — PNG herunterladen
* **Text kopieren** — Klartext in die Zwischenablage

Das Sharesheet braucht einen sicheren Kontext (HTTPS oder localhost).
Per Doppelklick aus dem Dateisystem geöffnet greift automatisch der
Fallback.

Im Bericht steht die **vollständige Aufstellung beider Seiten** — jeder
Kämpfer namentlich mit Gegenstand und Rest-HP beziehungsweise K.O., und
jede Gegnerart mit Anzahl und wie viele davon erledigt wurden. Nichts
wird abgeschnitten; die Bildhöhe wächst mit der Liste.

Dazu Ergebnis und Dauer, Meister Schaden, meiste K.O.s, meiste Prügel
eingesteckt, härtester Treffer mit Urheber, sowie Angriffe,
Gesamtschaden, Schaden pro Sekunde, ausgelöste Specials, Krits,
Glückstreffer, Fehlschläge, Ausweichmanöver und Blocks.

## Balancing selbst testen

Die Engine läuft auch ohne Grafik. In der Browser-Konsole:

```js
fastSim([FIGHTERS[0]], [{ def: ANIMALS.find(a => a.id === 'chihuahua'), n: 20 }])
```

## Noch offen

* Feintuning am Balancing, sobald ihr gespielt habt
* Weitere Arenen und Gegenstände sind je ein Eintrag in `ARENAS`
  beziehungsweise `ITEMS`

---

Keine Tiere wurden bei der Erstellung dieser Seite verletzt. Sie sind alle nur Pixel.
