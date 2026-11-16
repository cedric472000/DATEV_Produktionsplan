# Produktionsplan DATEV magazin

Web-Seite, die aus **Planungsjahr und Versandterminen** den kompletten
Produktionsplan des *DATEV magazins* rückwärts terminiert. Sie rechnet in
einem Plan die Meilensteine der Redaktion (Kick-off bis Retro) und die
Produktionskette der Agentur (Platzieren bis Anlieferung): 28 Schritte je
Ausgabe. Dabei beachtet sie Wochenenden, die Feiertage in Bayern, Berlin und
Schleswig-Holstein und die Betriebsruhe zwischen den Jahren.

Die Regeln stammen aus den tatsächlichen Terminen 2026 und dem
Produktionsplan 12/2025 bis 01/2027 (siehe [Rechenregel](#rechenregel)). Die
Seite ist ein Ableger des Produktionsplans der *Berliner Wirtschaft* und
nutzt dieselbe Technik.

## Benutzung

```
python3 build.py     # erzeugt index.html aus src/app.html
```

| Fassung | Anmeldung | Ablage |
|---|---|---|
| **Website** (GitHub Pages + Firebase) | Mail und Passwort, nur Personen auf der Zugriffsliste | gemeinsam, live, Rechte auf dem Server |
| `index.html` als Datei | keine | Browserspeicher dieses Rechners |

Ohne `firebase-config.json` ist `index.html` die Datei-Fassung. Ein Doppelklick
genügt, es braucht keinen Server.

Im Repository liegen nur Code und Regeln. Pläne, Personen und alte
Produktionspläne gehören nicht hierher: Sie liegen in Firebase oder im
Browser.

## Website mit Anmeldung einrichten

Die Website liegt auf GitHub Pages. Anmeldung und gemeinsame Ablage kommen von
Firebase (Google), im kostenlosen Spark-Tarif, ohne Kreditkarte. Für DATEV
braucht es **ein eigenes Firebase-Projekt** (nicht das der Berliner Wirtschaft).
Dann sind Zugriffsliste und Daten getrennt. Die Einrichtung erfolgt einmalig.

**1. Firebase-Projekt**
1. <https://console.firebase.google.com> → *Projekt hinzufügen*, Name z. B.
   `datev-produktionsplan`, Google Analytics aus.
2. *Build → Authentication → Jetzt starten → Sign-in method*: **E-Mail-Adresse/Passwort**
   aktivieren (nur den oberen Schalter) → Speichern.
3. *Authentication → Einstellungen → Autorisierte Domains*: `cedric472000.github.io`
   hinzufügen.
4. *Authentication → Vorlagen*: Vorlagensprache **Deutsch**, optional Absendername
   „Produktionsplan DATEV magazin“.
5. *Build → Firestore Database → Datenbank erstellen*: Standort
   **europe-west3 (Frankfurt)**, im **Produktionsmodus** starten.
6. *Firestore → Regeln*: den Inhalt von [`firestore.rules`](firestore.rules)
   vollständig einfügen → **Veröffentlichen**.
7. *Projekteinstellungen → Allgemein → Meine Apps → Web (`</>`)*: App
   registrieren (ohne Hosting). Die Werte von `firebaseConfig` als
   `firebase-config.json` neben `build.py` ablegen:
   ```json
   {"apiKey": "…", "authDomain": "….firebaseapp.com", "projectId": "…",
    "storageBucket": "…", "messagingSenderId": "…", "appId": "…"}
   ```
   Diese Werte sind kein Geheimnis. Geschützt wird über die Regeln aus Schritt 6.
8. `python3 build.py`, dann `index.html` und `firebase-config.json` committen.

**2. GitHub Pages**
*Settings → Pages → Build and deployment → Deploy from a branch*, Branch `main`,
Ordner `/ (root)` → **Save**. Nach ein bis zwei Minuten läuft die Seite unter
<https://cedric472000.github.io/DATEV_Produktionsplan/>.

**3. Erster Login**
Die erste Person, die sich mit bestätigter Mailadresse anmeldet, wird
**Besitzer** und ist damit immer Admin. Den Schritt also sofort selbst
erledigen: *Konto anlegen*, Mail bestätigen, *Als Besitzer übernehmen*.
Danach trägt man im Reiter **Zugriff** die Kolleginnen und Kollegen mit Rolle
ein.

`.nojekyll` schaltet die Jekyll-Verarbeitung von GitHub Pages ab. Nach jeder
Änderung an `src/app.html`: `python3 build.py` und `index.html` mit committen.

## Aufbau

| Datei | Inhalt |
|---|---|
| `src/app.html` | Die komplette Anwendung: Styles, Markup, Rechenkern (einzige Quelle) |
| `build.py` | Verpackt `src/app.html` in ein vollständiges HTML-Dokument |
| `index.html` | Erzeugt – die auslieferbare Seite |
| `firebase-config.json` | Zugangsdaten des Firebase-Projekts; fehlt sie, wird die Datei-Fassung gebaut |
| `firestore.rules` | Zugriffsregeln der Website – hier wird über Rechte entschieden |
| `vendor/firebase.js` | Firebase-Bibliothek, gebündelt (neu bauen: `cd tools/firebase && npm install && npm run bundle`) |
| `firebase.json` | für die Firebase-Werkzeuge: Regeln hochladen, Emulator für Tests |

## Rechenregel

Ankertag jeder Ausgabe ist der **Versand Print**. Er ist zugleich der
Erscheinungstag.

| Regel | gelernt aus |
|---|---|
| Versand: letzter Freitag im Monat vor der Ausgabe. Für die Januar-Ausgabe gilt der Freitag vor Weihnachten. Ist der Freitag frei (Karfreitag, Heiligabend), eine Woche früher. | alle 13 Versandtermine 2026 und 01/2027 |
| Kick-off, Printplanung, Redaktionskonferenz, Konsolidierung, Grafikplanung: montags, 15½, 14½, 13½, 11½ und 10½ Wochen vor dem Versand. Es sind Wochentermine: bei einem Feiertag am Nachbartag, in der Betriebsruhe eine Woche früher. | tatsächliche Termine 2026 |
| Textabgabe: freitags 9 Wochen vor dem Versand. Bleiben wegen Weihnachten weniger als 39 Arbeitstage bis zum Versand (sonst 45), rückt sie eine Woche vor. | Produktionsplan: Februar- und März-Ausgabe 2026 |
| Platzieren: 3 Arbeitstage nach der Textabgabe. | Produktionsplan |
| Korrekturschleifen bis zur Proofausgabe: rückwärts von der Seitenfreigabe. Die redaktionelle Bearbeitung füllt die Zeit dazwischen (üblich 10 Arbeitstage). | Produktionsplan |
| Wird diese Zeit knapp: Bei wenigen fehlenden Tagen gibt nur die Bearbeitung nach, bis auf 8 Tage. Sonst geben alle Schritte von der Bearbeitung bis zur Proofausgabe anteilig nach. | Februar 2026 (Weihnachten), Juni/Juli 2026 (Ostern, Pfingsten) |
| Druckfreigabe (Beginn der finalen Seitenfreigabe): freitags drei Wochen vor dem Versand. Danach folgen in Arbeitstagen Proofbesprechung, Reinzeichnung, Daten an Druckerei, Freigabe-PDF, Druck (7 Tage), Puffer und Anlieferung (mittwochs vor dem Versand). | Produktionsplan, 12–14 von 14 Ausgaben |
| Landingpage und Online-Beiträge: freitags eine Woche vor dem Versand. Retro: freitags eine Woche danach. | tatsächliche Termine 2026 |
| Betriebsruhe von der Woche mit Heiligabend bis Silvester. | Produktionsplan: 22.12.2025–01.01.2026 wurde nicht gearbeitet |

Die Regel legt das **Normalraster** fest, also die Lage der Termine in einer
Woche ohne Feiertag. Feiertage wirken je nach **Bindung** des Schritts:

- **Kette:** hält zum Bezug die Arbeitstage. Ein Feiertag dazwischen zieht den
  Schritt einen Arbeitstag vor.
- **Wochentermin:** bleibt in seiner Woche.
- **Fester Termin:** bei einem Feiertag gilt der Arbeitstag davor.

Arbeitsphasen (Spalte *Dauer*) enden am Termin des Schritts. Der Plan zeigt
zusätzlich ihren Beginn („ab …“).

**Abgleich mit 2026.** Gegen die 14 Ausgaben 12/2025 bis 01/2027 trifft der
Plan:

- Versand, Textabgabe, Seitenfreigabe, Puffer und Anlieferung: 14 von 14;
- Daten an Druckerei, Freigabe-PDF und Druck: 13 von 14;
- Landingpage und Online-Beiträge: 13 von 13;
- Proofbesprechung und Reinzeichnung: 12 von 14;
- Planungstermine: 7 bis 12 von 13.

Die Korrekturschleifen zwischen Textabgabe und Seitenfreigabe treffen 4 bis 8
von 14. Sie liegen im Original selbst je Ausgabe unterschiedlich.

## Pläne, Aufgaben, Zugriff

Wie beim Produktionsplan der Berliner Wirtschaft:

- **mehrere Pläne** mit Status *In Arbeit* oder *Freigegeben*, *Duplizieren*
  für das nächste Jahr;
- **Aufgaben** je Abteilung und Person, an Schritt und Ausgabe gekoppelt;
  sind alle erledigt, wird der Plan freigegeben und gesperrt;
- **Rollen** im Reiter *Zugriff*.

| Rolle | darf |
|---|---|
| Admin | alles: Pläne, Termine, Regelwerk, Aufgaben, Team, Zugriffsliste |
| Beobachter | alles ansehen und exportieren, nichts ändern |
| Zugewiesene Aufgaben | wie Beobachter, dazu bei den eigenen Aufgaben: abhaken, Rückmeldung, ggf. den Termin des Schritts setzen |

Entschieden wird auf dem Server, in [`firestore.rules`](firestore.rules).

## Manuelle Termine

Ein Klick auf eine Zelle im Terminplan öffnet ein Fenster mit dem
automatischen Termin zum Vergleich. Es warnt bei Wochenende, Feiertag oder
falschem Wochentag. Was auf dem gesetzten Termin aufbaut, **wandert mit**,
im selben Abstand in Arbeitstagen. Wochentermine wandern um so viele Wochen
mit, wie der Vorgänger von Hand verschoben wurde. Wird zum Beispiel die
Textabgabe eine Woche früher gesetzt, rücken Grafikplanung, Konsolidierung,
Konferenz, Printplanung und Kick-off eine Woche vor, und die Bearbeitung
bekommt die gewonnene Zeit.

## Feiertage

Voreingestellt sind die gesetzlichen Feiertage in **Bayern, Berlin und
Schleswig-Holstein**. Hinzu kommt die **Betriebsruhe zwischen den Jahren**:
die Werktage von der Woche mit Heiligabend bis Silvester. Jeder Tag lässt
sich einzeln abschalten. Eigene freie Tage (Betriebsferien) kommen über das
Feld unter der Liste dazu.

## Prüfungen

Der Reiter **Prüfung** meldet:

- Termine auf freien Tagen;
- gebrochene Reihenfolgen;
- verkürzte Bearbeitungszeiten;
- Termine zwischen den Jahren;
- Abweichungen vom Normalraster wegen Feiertagen (ein Hinweis je Ausgabe).

## Export

- **Excel** mit zwei Blättern:
  - *Produktionsplan*: Schritte × Ausgaben, Arbeitsphasen als „von – bis“;
  - *Meilensteine*: je Ausgabe die Kalenderwochen der Planungstermine,
    Textabgabe, Druckfreigabe, Daten an Druckerei, LP und Online, Versand
    Print und Retro, aufgebaut wie die Tabelle „Planung 2027“.
- CSV als Matrix oder Terminliste.
- Kalenderdatei (.ics): Arbeitsphasen als mehrtägige Einträge.
- Konfiguration als JSON.
