# Video zu Ton: fertig

Ein Video läuft, Striche stehen still, und was sie durchquert, wird Ton: Lichtton wie auf einem Filmstreifen. Das ist ein eigenes Repo, wie Spektral-Malen. Zum Einbauen ins Portfolio gibt es `merge-video-zu-ton.zip` mit eigenem LIESMICH.

**Einrichten**
- Inhalt über dein Repo `video-zu-ton` legen.
- `css/fonts/` aus dem Portfolio dazukopieren.
- Mit deinem lokalen Server ausliefern. Für die Kamera muss es `localhost` oder https sein.

---

## 0. Neu in dieser Runde

- **Drehung von Hand** (Quelle): Handyvideos liegen oft quer in der Datei, und eine Notiz in den Metadaten sagt „um 90° drehen“. Ob der Browser das beim Hochladen auf die Grafikkarte anwendet, ist verschieden. Darum gibt es jetzt die Auswahl. Gedreht wird im Shader beim Lesen (`gedreht(uv, drehung)`), das Feld tauscht Breite und Höhe.
- **Der Strich ist deckend weiß**, mit dunklem Rand, auch im exportierten Video. Ist er breiter als ein paar Pixel, zeigen zwei feine Linien die Lesebreite.
- **„Zeigen, was gelesen wird“** schaltet die roten Balken aus, bei der Wellenform die Kurve.
- **Vorlagen** (Tonhöhen): Fläche, Glocken, Bass-Drone, Glas, Rohdaten. Angenehm wird es mit wenigen Stimmen (12 bis 24), einer Tonleiter, engem Umfang und langer Glättung. 96 Zeilen stufenlos sind ein Cluster und klingen fast wie Rauschen.
- **Klangfarbe „warm“:** leise 2., 3. und 4. Teiltöne zum Sinus. Ein reiner Sinus klingt dünn, weil das Ohr fast nur bei Flöten und Pfeifen so etwas hört.
- **Ausgleich:** Viele gleichzeitige Stimmen werden durch die Wurzel ihrer Gesamtlautstärke geteilt. Töne mit zufälliger Phase addieren sich in der Leistung, nicht im Ausschlag: 16 gleich laute Töne sind 4-mal so laut, nicht 16-mal. Darum ist `tanh` jetzt nur noch Notbremse, und die Lautstärke steht anfangs auf 0,6.
- **Wellenform statt Lichtton:** umbenannt, siehe Teil 4, mit Kurve neben dem Strich.
- **Neue Art „Auslösen“:** siehe unten.

### Auslösen (`ausloeser.js`)

Wie in den Videos, in denen jedes Auto, das eine Linie überquert, einen Ton spielt. Statt dauernd zu klingen, spielt jedes **neue Objekt** auf dem Strich eine **Note**.

1. **Objekte:** zusammenhängende Zeilen über „Objekt ab“, mindestens „mindestens“ Zeilen lang. Mit **Hysterese**: Ein Objekt beginnt erst über der Schwelle, gehört dann aber bis hinunter zu 60 % davon dazu. Sonst zerfällt ein Auto mit dunkler Scheibe in drei Objekte.
2. **Neu** ist ein Objekt, das sich mit keinem Objekt des letzten Bildes überschneidet. Solange ein Auto über der Linie ist, liegt es in jedem Bild fast an derselben Stelle, also gibt es keinen neuen Ton.
3. **Note:** Die Zeile der Mitte gibt die Tonhöhe (mit Tonleiter gerastet). Größe und Helligkeit geben die Stärke. Wahlweise kommt der Dreiklang auf dieser Stufe der Leiter dazu.
4. **Instrumente** im Audio-Thread:
   - **Gezupft** (Karplus-Strong, 1983): Ein Puffer voller Rauschen, genau eine Schwingung lang (Abtastrate / f Samples), wird immer wieder abgespielt und dabei jedes Mal ein wenig geglättet. Hohe Obertöne verschwinden zuerst, wie bei einer echten Saite. Mit 20 Zeilen Code klingt das erstaunlich echt.
   - **Glocke:** drei Sinustöne in unharmonischen Abständen (1 : 2,76 : 5,4), die abklingen. Unharmonisch heißt: keine ganzzahligen Vielfachen, darum klingt es nach Metall statt nach Saite.
   - Der **Nachklang** ist die Zeit bis −60 dB.

Ehrlich:
- Mit **„Bewegung“** spielt ein gleichmäßig gefärbtes Objekt oft **zwei** Noten, eine für die Vorderkante und eine für die Hinterkante. Dazwischen ändert sich an der Linie nichts, also ist dort keine Bewegung.
- Mit **„Helligkeit“** (helle Objekte vor dunklem Grund, nachts Scheinwerfer) gibt es eine Note pro Objekt.
- Schatten, Gegenlicht und Bäume im Wind lösen auch aus. Zwei Autos, die sich auf der Linie berühren, sind ein Objekt.
- Im Test haben die drei Testautos (10 bis 50 Pixel hoch) bei „mindestens 3 Zeilen“ nur das größte ausgelöst. Das ist richtig so, aber der Regler ist wichtig.

---

## 1. Was drin ist (alles aus dem Fahrplan)

| Fahrplan | Wo |
|---|---|
| Videodatei oder Kamera (vorne und hinten) | Quelle |
| Strich senkrecht oder waagerecht, verschiebbar, als Streifen gemittelt | Striche, ziehen im Bild |
| Kantenerkennung (weich → Sobel → Schwelle) als Baustein in `/lib` | Vorbearbeitung, `lib/kanten.js` |
| Bewegung (Differenz zum vorigen Bild) | Vorbearbeitung „Bewegung“, „bewegte Kanten“ |
| tiefste und höchste Frequenz, Tonleiter, einzelne Zeilen von Hand | Tonhöhen |
| Zeilendichte frei: wo viel passiert, mehr Zeilen | Tonhöhen, „Dichte nach Bild“ |
| live eine Bank von Oszillatoren | `stimmen-worklet.js` |
| Export Bild für Bild | Export: WAV, Video rendern (dazu Bild speichern, Live aufnehmen) |
| Ausschnitt als Lupe oder Fenster | Ausschnitt |
| zwei Striche für Stereo, mehrere für Akkorde, jeder auch rückwärts | Striche: Vorlagen, Panorama, Halbtöne, rückwärts |
| Originalton dazumischen | Ton, „Originalton“ |
| Lichtton wörtlich: der Strich als Wellenform | Strich, Art „Wellenform (Wavetable)“, siehe Teil 4 |

**Dateien**

| Datei | Aufgabe |
|---|---|
| `js/quelle.js` | Datei oder Kamera in ein `<video>`, Springen für den Export |
| `js/gpu.js` | Video als Textur, Vorbearbeitung, Striche und Profile lesen, anzeigen |
| `js/vorbereitung-shader.js` | Helligkeit (weich), dann hell, Kanten, Bewegung, Schwelle |
| `lib/kanten.js` | **neu für /lib**: Helligkeit, Weichzeichnen, Sobel als GLSL-Bausteine |
| `js/lese-shader.js` | Strich → Helligkeit pro Zeile (Mittelwert über das Stück) |
| `js/anzeige-shader.js` | Video oder Bearbeitetes, mit Lupe und Fenster |
| `js/frequenzen.js` | Tonhöhe jeder Zeile (Tonleiter, eigene, Halbtöne) und ihre Lage (Dichte) |
| `js/striche.js` | die Striche: Vorlagen, Liste, Einstellungen aus einer Beschreibung |
| `js/stimmen-worklet.js` | Oszillatorbank pro Strich, Wellenform, Stereo, auch für den Export |
| `js/klang.js` | Seite ↔ Audio-Thread, Originalton, Ton für den Export rechnen |
| `js/kopf.js` | Striche, rote Balken (was gelesen wird), Skala, Rahmen des Ausschnitts |
| `js/ausloeser.js` | Objekte auf einem Strich finden, neue werden zu Noten |
| `js/export-video.js` | WAV und Video Bild für Bild (mit den Noten im Zeitplan) |
| `js/main.js` | alles verbunden |

Aus dem Portfolio kopiert: `lib/regler.js`, `maus-info.js`, `export.js` (die Fassung mit `await ton()` aus dem Spektral-Malen-Merge), `export-dialog.js`, `mediabunny*.js`, dazu `css/`.

---

## 2. Hintergrund

**Lichtton.** Auf 35-mm-Film liegt neben den Bildern eine Tonspur aus Licht. Im Projektor scheint eine Lampe durch einen schmalen Schlitz auf diese Spur, eine Fotozelle misst, wie viel durchkommt, und das ist der Ton.

Drei Leute haben das als Instrument benutzt:
- **Rudolf Pfenninger** zeichnete 1932 die Wellenformen von Hand auf Papier, abfotografiert direkt in die Tonspur („Tönende Handschrift“).
- **Norman McLaren** tat in „Synchromy“ (1971) das Umgekehrte: Er kopierte die Tonspur ins Bild, und man sieht, was man hört.
- **Daphne Oram** baute mit Oramics eine Maschine, in der gemalte Kurven auf Filmstreifen Tonhöhe und Lautstärke steuern.

**Was dieses Projekt anders macht.** Hier wird der Strich nicht als Wellenform gelesen (das kommt später als Option „Lichtton wörtlich“), sondern als **Spektrum**, wie in bild-zu-ton:
- jede Zeile des Strichs ist ein Ton,
- unten tief, oben hoch,
- je heller, desto lauter.

Der Unterschied zu bild-zu-ton: Dort wandert der Lesekopf über ein stehendes Bild. Hier steht der Strich, und das Bild wandert unter ihm durch. Eine Hand, die durchs Bild fährt, wird zu einem Glissando.

---

## 3. Lesen: vom Bild zur Helligkeit pro Zeile

### Durchgänge auf der Grafikkarte

Pro Videobild laufen vier Durchgänge:
1. **Helligkeit, weichgezeichnet** (`HELL_SHADER`), in höchstens 640 Pixeln Breite. Für Kanten und Bewegung reicht das, und es ist auch am Handy schnell.
2. **Bearbeiten** (`BEARBEIT_SHADER`): hell, Kanten, Bewegung oder bewegte Kanten, dann invertieren und Schwelle.
3. **Lesen** (`LESE_SHADER`): für jeden Strich eine Zeile einer kleinen Fläche, `zeilen` breit. Dazu kommen zwei Profile über das ganze Bild.
4. **Zurücklesen**: einmal `readPixels` für alles. Das sind ein paar hundert Bytes statt 4 MB pro Bild wie mit `getImageData`.

### Weichzeichnen, Sobel, Schwelle (`lib/kanten.js`)

- **Sobel** misst, wie stark sich die Helligkeit an einer Stelle ändert:
  - `gx` = rechte Spalte minus linke, `gy` = obere Zeile minus untere, die Mitte jeweils doppelt gewichtet.
  - Die Kante ist die Länge von (gx, gy).
  - Eine gleichmäßige Fläche wird 0, ein Umriss hell.
- **Vorher weichzeichnen** ist nötig, weil Sobel jeden Unterschied findet, und Rauschen ist auch ein Unterschied. Der 3×3-Gauß (Gewichte 1 2 1 / 2 4 2 / 1 2 1) ist die kleinste Glocke.
- **Die Schwelle danach** macht das, was übrig bleibt, zu echter Stille.

Das Video wird erst weichgezeichnet und dann in Helligkeit umgerechnet, nicht umgekehrt. Beides ist linear, also kommt dasselbe heraus, aber es spart einen Durchgang.

### Bewegung

Zwei Helligkeitsbilder wechseln sich ab (Ping-Pong): das von jetzt und das von vorher. Bewegung ist `|jetzt − vorher|`. Ein stehendes Bild ist still, nur was sich ändert, klingt. Bei Pause wird dasselbe Bild noch einmal gerechnet, dann ist die Differenz 0 und es wird still. Sonst klänge die letzte Bewegung ewig weiter. Derselbe Ping-Pong ist die Vorbereitung für die Chronofotografie.

### Warum ein Mittelwert und nicht ein Punkt

Ein Strich von 1 Pixel ist ehrlich, aber unruhig:
- **Rauschen.** Jedes Videobild rauscht ein wenig, vor allem bei der Kamera. Mittelt man über n Pixel, wird das Rauschen um den Faktor √n kleiner, bei 16 Pixeln also auf ein Viertel. Darum gibt es den Regler „Breite“.
- **Aliasing.** Bei 96 Zeilen und 720 Bildzeilen deckt jede Zeile etwa 7,5 Bildzeilen ab. Nimmst du nur eine davon, springt ein dünner heller Strich im Video zwischen „gehört“ und „nicht gehört“, je nachdem, ob er zufällig die Probe trifft. Ein Mittelwert über das ganze Stück ist ein **Boxfilter**, die einfachste Art, das zu verhindern. Dasselbe Prinzip hatte die Mittelung der Bildzeilen in Spektral-Malen (`bild-ebene.js`).

### Der Lese-Shader

Jede Zeile hat ein Stück des Strichs:
- längs von `g.x` bis `g.y` (aus `grenzen`, siehe Dichte),
- quer über die Breite.

Darin liegen `probenQuer × probenLaengs` Proben, jede in der **Mitte** ihres Teilstücks (`(i + 0,5) / n`), wie `gl_FragCoord`, das auch bei x,5 liegt. Die Helligkeit hat die Vorbearbeitung schon ausgerechnet (Rec. 709: Grün zählt am meisten). Am Ende wird der Mittelwert genommen; `float(…)`, weil GLSL `int` nicht von selbst umwandelt.

- **Rückwärts** heißt: Das Stück liegt am anderen Ende, `1 − …`, und dabei tauschen unten und oben.
- Alle Lagen sind Anteile des **Bereichs**, erst am Ende wird daraus uv im Video (`mix(bereich.xy, bereich.zw, lokal)`). Darum geht die Lupe ohne eigenen Code: Der Bereich ist einfach kleiner.
- Das **Fenster** zählt Proben außerhalb als Schwarz, ein halb verdecktes Stück wird darum halb so laut.

### Zeilendichte nach Bild (`frequenzen.js`)

Die zwei Profile sagen, wie viel in jedem Streifen des ganzen Bildes los ist (nach der Vorbearbeitung, also Kanten oder Bewegung, wenn die gewählt sind). Sie werden über etwa eine Sekunde geglättet, sonst zappeln die Zeilen.

Das Profil wird wie eine **Verteilung** behandelt: aufsummiert, und die Grenzen der Zeilen liegen dort, wo 1/n, 2/n und so weiter erreicht sind (Quantile). Wo viel los ist, werden die Zeilen schmal, also liegen dort mehr Töne. Die Tonhöhen selbst bleiben gleichmäßig verteilt, nur ihr Platz im Bild ändert sich. Mit „Dichte nach Bild“ mischst du zwischen gleichmäßig (0) und ganz nach Bild (1). Ganz ohne Gleichmäßiges bekäme ein leerer Streifen gar keine Zeile, und ein Ding, das dort hineinläuft, wäre stumm.

Die Skala und die roten Balken folgen den Grenzen, also siehst du, wie sich die Zeilen zusammenziehen.

---

## 4. Klingen: die Stimmen (`stimmen-worklet.js`)

### Helligkeit → Lautstärke

Wie immer: `10^((v − 1) · Dynamik / 20)`, bei v = 0 echte Stille.

### Oszillatorbank

Pro Zeile und Sample:
1. Die Lautstärke geht ein Stück Richtung Ziel (Glättung).
2. Die Phase dreht sich weiter.
3. Sinus mal Lautstärke kommt auf den Ausgang.

**Warum `phase += schritt` und nicht `sin(2π · f · t)`?** Rechnerisch ist es dasselbe, solange f gleich bleibt. Aber:
1. **t wächst immer weiter.** Nach einer Stunde ist `f · t` riesig, und ein Float verliert hinten Stellen: Der Ton wird unsauber. Eine Phase, die du mit `− 2π` immer klein hältst, bleibt genau.
2. **Ändert sich f** (in einem späteren Schritt, wenn Zeilen auf eine Tonleiter rasten), springt `sin(2π · f · t)` sofort an eine ganz andere Stelle. Das knackt. Eine Phase, die nur weitergedreht wird, macht einfach schneller weiter.

Das ist derselbe Gedanke wie die Phase im Worklet von Spektral-Malen, nur pro Sample statt pro Fenster.

Hier ist die Phase ein **Anteil einer Umdrehung** (0 bis 1) und nicht im Bogenmaß. Dann ist der Sinus ein Nachschlagen in einer Tabelle mit 4096 Werten, gerade verbunden. Bei 4 Strichen mit je 256 Zeilen sind das bis zu 49 Millionen Sinuswerte pro Sekunde, und dort zählt das. Stille Zeilen werden ganz übersprungen: Ein dunkles Video kostet fast nichts.

Weil sich die Phase nur weiterdreht, kann sich die Frequenz einer Zeile jederzeit ändern (Tonleiter, Halbtöne, eigene Zeilen), ohne dass es knackt. `baue()` behält darum Phasen und Lautstärken, wenn sich die Striche ändern.

### Glättung

Das Video liefert 25 oder 30 neue Werte pro Sekunde, der Ton braucht 48000. Springt die Lautstärke jedes Mal hart auf den neuen Wert, entsteht bei jedem Sprung ein kleiner Knack. 30 Knackser pro Sekunde klingen wie Knistern oder ein Brummen bei 30 Hz („zipper noise“).

`lautstärke += (ziel − lautstärke) · k` ist ein **Tiefpass erster Ordnung** (one-pole):
- Bei jedem Sample geht die Lautstärke den Anteil k des restlichen Wegs.
- Nach n Samples ist noch `(1 − k)^n` des Wegs übrig. Das ist eine Exponentialkurve, wie ein Kondensator, der sich auflädt.

Gesucht ist k, sodass nach der Zeit τ (der Regler „Glättung“ in ms) noch `1/e ≈ 37 %` übrig sind, also 63 % geschafft:

```
(1 − k)^(τ · Abtastrate) = e^(−1)
```

Gelöst: `k = 1 − e^(−1 / (τ · Abtastrate))`. Bei 30 ms und 48 kHz ist k ≈ 0,0007.

### Tonleiter und eigene Zeilen

- **Tonleiter:** Die Frequenz jeder Zeile wird auf den nächsten Ton der Leiter gezogen (über MIDI-Nummern: `69 + 12 · log2(f / 440)`). Mehrere Zeilen können dabei auf denselben Ton fallen, der wird dann lauter. Das ist ehrlich, die Zeilen liegen ja dichter als die Leiter.
- **Halbtöne pro Strich** verschieben alles um den Faktor `2^(n/12)`. Drei Striche an derselben Stelle mit 0, 4 und 7 Halbtönen sind ein Dur-Akkord aus einem Bild (Vorlage „Akkord“).
- **Einzelne Zeilen** legst du von Hand fest. Die Zeilennummer zeigt die Maus-Info.

### Stereo

Jeder Strich hat ein Panorama, geteilt mit **gleicher Leistung**: links `cos(w)`, rechts `sin(w)`, mit w von 0 bis 90°. In der Mitte sind das je 0,71, nicht 0,5. Bei 0,5 klänge die Mitte leiser, weil zwei Lautsprecher sich in der Leistung addieren und nicht im Ausschlag. Mit der Vorlage „Stereo“ liegt der linke Strich links, der rechte rechts: Etwas, das durchs Bild läuft, wandert im Kopfhörer mit.

### Wellenform (früher „Lichtton wörtlich“)

Auf einem Filmstreifen liegt neben den Bildern die Tonspur, eine helle Fläche mit wechselnder Breite. Der Film läuft an einem Schlitz vorbei. Durch den Schlitz fällt mal mehr, mal weniger Licht auf eine Fotozelle, und genau das bewegt die Lautsprechermembran. Die Form der Spur **entlang des Films** ist also die Schallwelle, und die Zeitachse ist die Laufrichtung des Films.

Hier wird der Strich **der Länge nach** gelesen: Die Helligkeit jeder Zeile (= wie viel Helles quer zum Strich liegt) ist die Auslenkung. Von unten nach oben ergibt das eine Kurve, die du jetzt neben dem Strich siehst. Diese Kurve gilt als **eine Schwingung** und wird mit dem Grundton immer wieder abgespielt (110 Hz = 110-mal pro Sekunde).
- Die **Tonhöhe** kommt vom Grundton.
- Das **Bild** bestimmt die **Klangfarbe**: Ein Rechteck auf dem Strich klingt wie eine Rechteckwelle (hohl, wie eine Klarinette), ein Verlauf wie ein Sägezahn (scharf).
- Ändert sich das Bild, ändert sich die Klangfarbe. Die Form wird dabei pro Block geglättet, sonst knackt jeder Bildwechsel.

**Ehrlich:** Das ist eine **Wellentabelle** (Wavetable, wie bei Oramics und in Vital), kein wörtlicher Lichtton. Beim Film läuft die Spur am Schlitz vorbei, bei uns steht der Strich still, und die Zeitachse ist seine Länge. Wirklich wörtlich ginge es mit Video nicht: Dafür bräuchte es tausende Bilder pro Sekunde, eines pro Sample. Darum der neue Name.

Pfenninger hat seine Wellen 1932 von Hand gezeichnet. Hier kannst du ein Blatt Papier mit Strichen vor die Kamera halten und hörst sofort, wie die Form klingt.

### Originalton

Das `<video>` geht über `createMediaElementSource` in denselben AudioContext, mit eigenem Regler. Das geht pro Element nur einmal, darum gibt es nur ein `<video>`, und Datei und Kamera wechseln sich darin ab. Die Kamera hat keinen Ton.

---

## 5. Export Bild für Bild (`export-video.js`)

Live klingt, was der Browser gerade schafft; ruckelt er, fehlen Bilder. Beim Export dagegen:
1. Jedes Bild wird einzeln angesprungen (`currentTime`, warten auf `seeked`) und gelesen. Daraus entsteht ein **Zeitplan**: Helligkeiten pro Bild und Strich.
2. Der ganze Ton entsteht auf einmal in einem OfflineAudioContext, mit **demselben Worklet**. Das bekommt den Zeitplan über `processorOptions` und nimmt sich bei jedem Sample die Werte des Bildes, in dem es gerade liegt (`currentFrame · fps / Abtastrate`). Der Originalton wird dekodiert und dazugemischt.
3. Für das Video wird jedes Bild noch einmal angesprungen, mit Strichen und Balken gezeichnet und mit seinem Stück Ton geschrieben.

So ist es egal, wie langsam der Rechner ist. Mit der Kamera geht das nicht (es gibt nichts zum Anspringen), dafür gibt es „Live aufnehmen“.

---

## 6. Ehrlich

- **Nur in Chromium getestet.** Der Reihe nach:
  - Balken oben im Bild wird oben gelesen.
  - Kanten, Bewegung und bewegte Kanten reagieren.
  - Bei Pause wird es in „Bewegung“ still.
  - Stereo: In der WAV ist links laut, wenn der Balken den linken Strich kreuzt, rechts leise.
  - Der Originalton (1 kHz) ist in der WAV.
  - Lupe, Fenster, Tonleiter, Dichte, Lichtton.
  - Video rendern mit Ton (in Headless kam VP9 heraus, in deinem Chrome H.264).
  - Kamera mit der Test-Kamera.
  - Handy-Layout.
- **Gehört habe ich es nicht.**
- **Firefox und Zen** kennen `requestVideoFrameCallback` vielleicht nicht, dann wird 60-mal pro Sekunde gelesen. Das geht, ist aber etwas ungenauer bei der Bewegung (dasselbe Bild zweimal heißt Bewegung 0).
- **Die Kamera am iPhone:** Vorder- und Rückkamera über `facingMode`, ungetestet.
- **Rechenlast:** 4 Striche × 256 Zeilen bei hellem Bild sind viel. Am Handy lieber 1 bis 2 Striche und 64 bis 96 Zeilen.
- **Originalton im Export:** Den Ton aus `.mov`/`.mp4` dekodiert nicht jeder Browser mit `decodeAudioData`. Dann kommt ein Hinweis, und das Video hat nur den erzeugten Ton.
