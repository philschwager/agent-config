---
name: tsk:korr-lesetagebuecher
description: Bewertet alle Lesetagebücher in einem angegebenen Ordner. Jede Bewertung wird als Subagent in einem eigenen Kontext durchgeführt. Die Resultate werden als Markdown-Dateien unter korrekturen/ gespeichert.
argument-hint: [ordner-pfad]
allowed-tools: Read, Glob, Grep, Bash, Task, Write
---

Du bewertest alle Lesetagebücher im angegebenen Ordner: `$ARGUMENTS`

## Vorgehen

1. **Finde alle Schülerarbeiten** im Ordner `$ARGUMENTS`. Suche nach PDF-Dateien (.pdf), Bildern (.png, .jpg, .jpeg) und Textdateien (.md, .txt, .docx). Verwende das Glob-Tool.

2. **Erstelle den Ausgabeordner**: Leite aus dem Ordnerpfad den Ordnernamen ab (z.B. aus `schuelerarbeiten/lesetagebuecher2026` wird `lesetagebuecher2026`). Erstelle den Ordner `korrekturen/[ordnername]/` falls er nicht existiert.

3. **Prüfe für jede Arbeit, ob bereits eine Bewertung vorliegt**: Bestimme für jede gefundene Schülerarbeit den erwarteten Ausgabepfad (`korrekturen/[ordnername]/Bewertung_Lesetagebuch_[Schülername].md`). Prüfe mit dem Glob-Tool, ob diese Datei bereits existiert. Falls ja, überspringe diese Arbeit und melde in der Zusammenfassung, dass die Bewertung bereits vorhanden war. Nur Arbeiten **ohne** bestehende Bewertung werden an einen Subagent übergeben.

4. **Bewerte jede noch nicht bewertete Arbeit einzeln als Subagent**: Verwende für JEDE noch zu bewertende Schülerarbeit das **Task-Tool** mit `subagent_type: "general-purpose"`, damit jede Bewertung in einem eigenen Kontext stattfindet. Starte mehrere Subagents parallel, wenn möglich.

5. **Übergib dem Subagent alle nötigen Informationen**: Da der Subagent keinen Zugriff auf den bisherigen Kontext hat, musst du ihm im Prompt folgendes mitgeben:
   - Den vollständigen Dateipfad der zu bewertenden Arbeit
   - Den vollständigen Ausgabepfad für die Bewertung
   - Die Aufgabenstellung (siehe unten)
   - Die Bewertungskriterien (siehe unten)
   - Die Bewertungsvorlage (siehe unten)
   - Die Output-Regeln (siehe unten)

## Prompt für jeden Subagent

Verwende folgenden Prompt für jeden Subagent (ersetze die Platzhalter):

---

Du bewertest ein Lesetagebuch einer Schülerin oder eines Schülers der 2. Sekundarklasse (10. Schuljahr) im Kanton Zürich.

**Zu bewertende Datei:** [DATEIPFAD]
**Ausgabedatei:** [AUSGABEPFAD]

### Schritt 1: Arbeit lesen
Lies die Datei mit dem Read-Tool. Bei PDF-Dateien lies die Datei direkt (das Tool unterstützt PDFs). Bei Bildern (PNG, JPG) lies sie ebenfalls direkt (das Tool unterstützt Bilder). Versuche, den Namen der Schülerin oder des Schülers aus der Arbeit zu erkennen (Deckblatt, Kopfzeile etc.).

### Schritt 2: Bewertung durchführen
Bewerte die Arbeit anhand der folgenden Aufgabenstellung und Bewertungskriterien.

#### Aufgabenstellung Lesetagebuch

Die Schülerinnen und Schüler mussten ein Lesetagebuch zu einem selbst gewählten Buch führen mit folgenden Bestandteilen:

- **Deckblatt**: Titel des Buches, Autorin/Autor, Name, Klasse und Schuljahr, selbst gestaltetes Cover oder passendes Bild
- **Einleitung** (ca. 1/2 Seite): Warum dieses Buch? Vorwissen? Erwartungen?
- **Leseeinträge** (mindestens 8 Einträge, pro Abschnitt/Kapitel):
  - Datum und gelesener Abschnitt
  - Kurze Zusammenfassung (5–6 Sätze)
  - Persönlicher Kommentar (mindestens 5 Sätze): Gedanken, Fragen, beeindruckende Szenen, Wirkung der Figuren, Parallelen zum eigenen Leben
  - Sprache und Zitate: Eine sprachlich auffällige Stelle markieren und erklären
  - Reflexion: Wie verändert sich die Meinung? Was wurde gelernt?
- **Schlussreflexion** (ca. 1 Seite): Gesamteindruck mit Begründung, wichtige Themen, Empfehlung, neue Wörter/Redewendungen
- **Abgabetermin**: 7. Februar
- **Umfang**: Mindestens 8 Einträge

#### Bewertungskriterien

**1. Formale Anforderungen**
- Abgabe fristgerecht (7. Februar): ja / nein / verspätet
- Umfang (mind. 8 Einträge): erfüllt / nicht erfüllt (Anzahl angeben)
- Form (gemäss Vorgabe): passend / teilweise / nicht passend

**2. Inhaltliche Auseinandersetzung (40 Punkte)**
- Zeigt gutes Verständnis der Geschichte: ja / teilweise / nein (max. 10 Punkte)
- Einträge beziehen sich klar auf die gelesenen Abschnitte: ja / teilweise / nein (max. 10 Punkte)
- Reflexionen sind persönlich und gehen über reine Inhaltswiedergabe hinaus: ja / teilweise / nein (max. 10 Punkte)
- Schlussreflexion (ca. 1 Seite) vorhanden und inhaltlich vertieft: ja / teilweise / nein (max. 10 Punkte)

**3. Sprache und Stil (25 Punkte)**
- Wortschatz abwechslungsreich und verständlich: ja / teilweise / nein (max. 9 Punkte)
- Sätze klar und sinnvoll gebaut: ja / teilweise / nein (max. 8 Punkte)
- Rechtschreibung und Zeichensetzung überwiegend korrekt: ja / teilweise / nein (max. 8 Punkte)

**4. Gestaltung und Sorgfalt (20 Punkte)**
- Deckblatt mit allen Angaben (Titel, Autor/in, Name, Klasse, Schuljahr, Bild): vollständig / unvollständig (max. 5 Punkte)
- Einleitung (ca. 1/2 Seite) vorhanden und thematisch passend: ja / teilweise / nein (max. 5 Punkte)
- Lesbarkeit (Schrift, Layout, Ordnung): gut / genügend / ungenügend (max. 5 Punkte)
- Kreative und ansprechende Gestaltung: ja / teilweise / nein (max. 5 Punkte)

**5. Eigenständigkeit (15 Punkte)**
- Persönliche Kommentare in den Einträgen erkennbar: deutlich / ansatzweise / kaum (max. 5 Punkte)
- Eigene Fragen, Deutungen, Bezüge zum eigenen Leben / aktuellen Themen: häufig / vereinzelt / kaum (max. 5 Punkte)
- Insgesamt wirkt das Lesetagebuch selbstständig erarbeitet: ja / teilweise / nein (max. 5 Punkte)

**Notenberechnung:** Punktzahl / 100 × 5 + 1 = Note (Schweizer System 1–6, 6 ist die beste Note, halbe Noten sind üblich)

### Schritt 3: Bewertung im Vorlagenformat schreiben

Schreibe die Bewertung exakt in folgendem Format und speichere sie mit dem Write-Tool unter dem angegebenen Ausgabepfad:

```markdown
# Bewertungsrapport Lesetagebuch

| | |
|---|---|
| **Schüler/in** | [Name] |
| **Klasse** | [Klasse] |
| **Buch** | *[Buchtitel]* von [Autor/in] |
| **Abgabeformat** | [z.B. Digital (PDF, X Seiten) / Handschriftlich (X Seiten)] |

---

## 1. Formale Anforderungen

| Kriterium | Bewertung |
|---|---|
| Abgabe fristgerecht (7. Februar) | [ja / nein / verspätet] |
| Umfang (mind. 8 Einträge) | [erfüllt / nicht erfüllt (Anzahl angeben)] |
| Form (gemäss Vorgabe) | [passend / teilweise / nicht passend] |

**Bemerkungen:** [Kommentar zu formalen Aspekten]

---

## 2. Inhaltliche Auseinandersetzung (40 Punkte)

| Kriterium | Bewertung | Punkte |
|---|---|---|
| Zeigt gutes Verständnis der Geschichte | [ja / teilweise / nein] | ___ / 10 |
| Einträge beziehen sich klar auf die gelesenen Abschnitte | [ja / teilweise / nein] | ___ / 10 |
| Reflexionen sind persönlich und gehen über reine Inhaltswiedergabe hinaus | [ja / teilweise / nein] | ___ / 10 |
| Schlussreflexion (ca. 1 Seite) vorhanden und inhaltlich vertieft | [ja / teilweise / nein] | ___ / 10 |

**Stärken:**
- [Stärke 1]
- [Stärke 2]
- [Stärke 3]

**Verbesserungspotenzial:**
- [Verbesserung 1]
- [Verbesserung 2]

**Punktzahl: ___ / 40**

---

## 3. Sprache und Stil (25 Punkte)

| Kriterium | Bewertung | Punkte |
|---|---|---|
| Wortschatz abwechslungsreich und verständlich | [ja / teilweise / nein] | ___ / 9 |
| Sätze klar und sinnvoll gebaut | [ja / teilweise / nein] | ___ / 8 |
| Rechtschreibung und Zeichensetzung überwiegend korrekt | [ja / teilweise / nein] | ___ / 8 |

**Stärken:**
- [Stärke 1]
- [Stärke 2]

**Verbesserungspotenzial:**
- [Verbesserung 1]
- [Verbesserung 2]

**Punktzahl: ___ / 25**

---

## 4. Gestaltung und Sorgfalt (20 Punkte)

| Kriterium | Bewertung | Punkte |
|---|---|---|
| Deckblatt mit allen Angaben (Titel, Autor/in, Name, Klasse, Schuljahr, Bild) | [vollständig / unvollständig (was fehlt?)] | ___ / 5 |
| Einleitung (ca. 1/2 Seite) vorhanden und thematisch passend | [ja / teilweise / nein] | ___ / 5 |
| Lesbarkeit (Schrift, Layout, Ordnung) | [gut / genügend / ungenügend] | ___ / 5 |
| Kreative und ansprechende Gestaltung | [ja / teilweise / nein] | ___ / 5 |

**Stärken:**
- [Stärke 1]
- [Stärke 2]

**Verbesserungspotenzial:**
- [Verbesserung 1]

**Punktzahl: ___ / 20**

---

## 5. Eigenständigkeit (15 Punkte)

| Kriterium | Bewertung | Punkte |
|---|---|---|
| Persönliche Kommentare in den Einträgen erkennbar | [deutlich / ansatzweise / kaum] | ___ / 5 |
| Eigene Fragen, Deutungen, Bezüge zum eigenen Leben / aktuellen Themen | [häufig / vereinzelt / kaum] | ___ / 5 |
| Insgesamt wirkt das Lesetagebuch selbstständig erarbeitet | [ja / teilweise / nein] | ___ / 5 |

**Stärken:**
- [Stärke 1]
- [Stärke 2]

**Verbesserungspotenzial:**
- [Verbesserung 1]

**Punktzahl: ___ / 15**

---

## 6. Gesamtbewertung

| Kategorie | Punkte |
|---|---|
| Inhaltliche Auseinandersetzung | ___ / 40 |
| Sprache und Stil | ___ / 25 |
| Gestaltung und Sorgfalt | ___ / 20 |
| Eigenständigkeit | ___ / 15 |
| **Total** | **___ / 100** |

**Gesamtnote: ___**

*(Berechnung: Punktzahl / 100 × 5 + 1 = Note)*

---

## Allgemeine Rückmeldung

[Persönliche, wertschätzende Gesamtrückmeldung mit folgender Struktur:

1. **Stärken zuerst**: Was ist besonders gelungen? Was ist das Highlight der Arbeit?
2. **Inhaltliches Feedback**: Wie gut wurde das Buch verstanden und reflektiert? Was wäre der nächste Entwicklungsschritt?
3. **Sprachliches Feedback**: Was gelingt sprachlich gut? Wo liegt Entwicklungspotenzial?
4. **Abschluss**: Ermutigendes Fazit.]
```

### Output-Regeln
- Schreibe in Schweizer Hochdeutsch (ss statt ß, «» statt „")
- Professioneller, wertschätzender Ton
- Stärken immer zuerst nennen, dann Verbesserungspotenzial
- Notenvorschläge immer als Vorschlag formulieren, nie als Festlegung (z.B. «Notenvorschlag: 4.5»)
- Ersetze alle Platzhalter in eckigen Klammern mit den tatsächlichen Bewertungen
- Falls du den Namen der Schülerin / des Schülers nicht erkennen kannst, verwende den Dateinamen als Hinweis
- **Punkte pro Kriterium**: Trage in der Spalte «Punkte» jeder Bewertungstabelle (Abschnitte 2–5) die vergebene Punktzahl ein. Die Summe der Einzelpunkte eines Abschnitts muss der Abschnittspunktzahl entsprechen.

---

## Ablauf zusammengefasst

1. Finde alle Dateien im Ordner `$ARGUMENTS` mit Glob
2. Erstelle den Ausgabeordner `korrekturen/[ordnername]/` mit Bash (mkdir -p)
3. Für jede gefundene Datei:
   - Bestimme den Schülernamen aus dem Dateinamen (entferne Erweiterung und Präfixe wie «Lesetagebuch_»)
   - Setze den Ausgabepfad: `korrekturen/[ordnername]/Bewertung_Lesetagebuch_[Schülername].md`
   - **Prüfe mit Glob, ob der Ausgabepfad bereits existiert. Falls ja: überspringe diese Arbeit.**
   - Falls nein: Starte einen Subagent mit dem Task-Tool (subagent_type: "general-purpose") und übergib den vollständigen Prompt von oben mit allen eingesetzten Werten
4. Wenn alle Subagents fertig sind, gib eine Zusammenfassung aus mit drei Kategorien:
   - **Neu bewertet**: Welche Arbeiten wurden jetzt bewertet und wo gespeichert
   - **Übersprungen**: Welche Arbeiten hatten bereits eine Bewertung (mit Pfad zur bestehenden Datei)
   - **Total**: Anzahl bewertet / übersprungen / gesamt
