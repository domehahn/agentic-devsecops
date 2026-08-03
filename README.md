# Artikel schreiben für AgenticStack.eu

Jeder veröffentlichte Artikel ist eine einzelne `.mdx`-Datei in diesem Ordner. Die Content-Pipeline (`src/lib/content/articles.ts`) liest beim Build alle Dateien, validiert das Frontmatter und generiert automatisch URLs, Lesezeit, Themen-/Tag-Seiten, Serien und verwandte Artikel.

---

## 📋 Inhaltsverzeichnis

1. [Schnellstart](#schnellstart)
2. [Frontmatter-Referenz](#frontmatter-referenz)
3. [EU AI Act Transparenz (Pflicht ab 2026-08-02)](#eu-ai-act-transparenz)
4. [Workflow für ChatGPT/KI-generierte Artikel](#workflow-für-ki-generierte-artikel)
5. [Review-Checkliste](#review-checkliste)
6. [MDX-Komponenten](#mdx-komponenten)
7. [Best Practices](#best-practices)
8. [Häufige Fehler](#häufige-fehler)

---

## Schnellstart

### 1. Template kopieren

```bash
cp content/blog/_template.mdx content/blog/mein-artikel-titel.mdx
```

**Wichtig:** Der Dateiname (ohne `.mdx`) wird zur URL:
- `mein-artikel-titel.mdx` → `/blog/mein-artikel-titel`
- Verwenden Sie **nur** Kleinbuchstaben, Zahlen und Bindestriche
- ❌ Keine Leerzeichen, Umlaute oder Sonderzeichen

### 2. Frontmatter ausfüllen

Öffnen Sie die neue Datei und füllen Sie mindestens diese Felder aus:

```yaml
---
title: "Vollständiger Titel Ihres Artikels"
description: "Kurze Zusammenfassung (max. 160 Zeichen für SEO)"
date: "2026-08-02"
author: "dome"
topics:
  - agentic-engineering
ai:
  assisted: false  # Auf true setzen, wenn KI verwendet wurde
draft: true        # Auf false setzen zum Veröffentlichen
---
```

### 3. Artikel schreiben

Verwenden Sie Markdown und MDX-Komponenten (siehe [MDX-Komponenten](#mdx-komponenten)).

### 4. Veröffentlichen

```yaml
draft: false  # Von true auf false ändern
```

Dann bauen und deployen:

```bash
npm run build  # Validiert Frontmatter automatisch
npm run deploy
```

---

## Frontmatter-Referenz

### Pflichtfelder

```yaml
title: "Vollständiger Titel"
  # String, erscheint als <h1> und in Listen

description: "Ein-Satz-Zusammenfassung"
  # String, max. 160 Zeichen für SEO
  # Wird in Vorschauen, Meta-Tags und Social-Media-Cards verwendet

date: "2026-08-02"
  # String im Format YYYY-MM-DD
  # Artikel-Veröffentlichungsdatum

author: "dome"
  # String, muss in src/config/authors.ts existieren
  # Aktuell verfügbar: "dome"

topics:
  # Array mit mindestens einem Eintrag
  # Erlaubte Werte: siehe src/config/topics.ts
  - agentic-engineering
  - devsecops
  - ai-engineering
  - platform-engineering
  - compliance-automation
  - cloud-native
  - observability

draft: true
  # Boolean
  # true = Artikel wird NICHT gebaut/veröffentlicht
  # false = Artikel wird in Sitemap, RSS und Listen aufgenommen
```

### Optionale Felder

```yaml
updated: "2026-08-05"
  # String im Format YYYY-MM-DD
  # Zeigt "Updated am 05.08.2026" an, wenn unterschiedlich von date

tags:
  # Array mit freeform Strings
  # Generiert automatisch /tags/[tag]-Seiten
  - MCP
  - Kubernetes
  - SSRF

featured: false
  # Boolean
  # Maximal EIN Artikel sollte true sein
  # Wird auf der Startseite hervorgehoben

language: de
  # String: "de" oder "en"
  # Setzt <article lang="de"> für Screenreader
  # Standard: "de"

series:
  # Object für Artikel-Serien
  slug: agentic-skill-security-properties
  title: "Security Properties für Agentic Skills"
  order: 1
  # Generiert automatisch Serie-Navigation
```

---

## EU AI Act Transparenz

**⚠️ PFLICHT für alle Artikel ab 2026-08-02 (heute!)**

Gemäß EU AI Act Art. 50 muss jeder Artikel, der mit KI-Tools erstellt wurde, transparent gekennzeichnet werden.

### Minimale AI-Konfiguration

```yaml
ai:
  assisted: false
```

- `false` = Keine KI-Tools verwendet → Keine Disclosure
- `true` = KI-Tools wurden verwendet → Disclosure-Box wird angezeigt

### Vollständige AI-Konfiguration (empfohlen bei assisted: true)

```yaml
ai:
  assisted: true
  humanReviewed: true
  reviewedBy: "Dominik Hahn"
  reviewedAt: "2026-08-02"
  tools:
    - "ChatGPT"
    - "GitHub Copilot"
```

### AI-Felder im Detail

| Feld | Typ | Pflicht | Beschreibung |
|------|-----|---------|--------------|
| `assisted` | Boolean | ✅ Ja (ab 2026-08-02) | `true` wenn KI-Tools verwendet wurden |
| `humanReviewed` | Boolean | ❌ Nein | `true` wenn menschliche Review stattfand |
| `reviewedBy` | String | ✅ Ja (bei humanReviewed: true) | Name des Reviewers |
| `reviewedAt` | String | ✅ Ja (bei humanReviewed: true) | Review-Datum (YYYY-MM-DD) |
| `disclosure` | String | ❌ Nein | Custom Disclosure-Text (überschreibt Standard) |
| `tools` | Array | ❌ Nein | Liste verwendeter KI-Tools |

### Standard-Disclosure-Texte

**Bei `assisted: true` und `humanReviewed: true`:**
> "Dieser Beitrag wurde mit KI-Unterstützung erstellt und anschließend fachlich und redaktionell geprüft."

**Bei `assisted: true` ohne `humanReviewed`:**
> "Dieser Beitrag wurde mit KI-Unterstützung erstellt."

### Build-Zeit-Validierung

```bash
npm run build
```

**Fehler bei fehlendem `ai`-Feld:**
```
❌ Error: Artikel ab 2026-08-02 benötigen ein ai-Feld (assisted: true/false)
   Datei: content/blog/mein-artikel.mdx
```

**Fehler bei unvollständiger Review-Angabe:**
```
❌ Error: humanReviewed: true erfordert reviewedBy und reviewedAt
   Datei: content/blog/mein-artikel.mdx
```

---

## Workflow für KI-generierte Artikel

### Schritt 1: ChatGPT-Entwurf erstellen

```
Prompt-Beispiel:
"Schreibe einen technischen Blogartikel über [THEMA].
Zielgruppe: DevOps Engineers und Platform Engineers.
Länge: 1500-2000 Wörter.
Struktur: Einleitung, Hauptteil mit Codebeispielen, Fazit.
Füge Quellenangaben als Fußnoten hinzu."
```

### Schritt 2: Frontmatter vorbereiten

```yaml
---
title: "[Von ChatGPT generierter Titel]"
description: "[Kurze Zusammenfassung]"
date: "2026-08-02"
author: "dome"
topics:
  - agentic-engineering
ai:
  assisted: true
  humanReviewed: true
  reviewedBy: "Dominik Hahn"
  reviewedAt: "2026-08-02"
  tools:
    - "ChatGPT"
draft: true
---
```

### Schritt 3: ⚠️ KRITISCH - Quellennachweise prüfen

**Problem:** ChatGPT kann Quellen erfinden ("Halluzination")!

**Für jede Quelle im Artikel:**

1. ✅ **URL aufrufen** → Seite existiert?
2. ✅ **Inhalt lesen** → Aussage korrekt wiedergegeben?
3. ✅ **Datum prüfen** → Quelle aktuell?
4. ✅ **Bei Papers:** DOI/arXiv-ID verifizieren

**Bei erfundenen/falschen Quellen:**
- ❌ Quelle entfernen
- ✅ Eigene Recherche durchführen
- ✅ Korrekte Quellen hinzufügen

### Schritt 4: Code-Beispiele testen

```bash
# Alle Code-Beispiele im Artikel müssen funktionieren:
- TypeScript-Snippets: npm test
- Shell-Befehle: In Terminal testen
- YAML/JSON: Syntax validieren
```

### Schritt 5: Review durchführen

Siehe [Review-Checkliste](#review-checkliste) unten.

### Schritt 6: Build & Deploy

```bash
npm run build    # ✅ 282 Seiten gebaut
npm test         # ✅ 25/25 Tests bestanden
npm run deploy   # 🚀 Production Deployment
```

---

## Review-Checkliste

### ✅ Inhaltliche Qualität

- [ ] **Fachliche Korrektheit:** Alle technischen Aussagen verifiziert
- [ ] **Vollständigkeit:** Thema wird umfassend behandelt
- [ ] **Zielgruppe:** Angemessene Detailtiefe für DevOps/Platform Engineers
- [ ] **Struktur:** Logischer Aufbau mit klarer Gliederung
- [ ] **Lesbarkeit:** Verständliche Sprache, keine KI-typischen Floskeln

### ✅ Technische Validierung

- [ ] **Code-Beispiele:** Alle Snippets getestet und funktionsfähig
- [ ] **Befehle:** Shell-/Terminal-Befehle verifiziert
- [ ] **Versionsnummern:** Aktuelle Versionen verwendet
- [ ] **Links:** Alle URLs erreichbar und korrekt
- [ ] **Fußnoten:** Alle Referenzen existieren

### ✅ Quellen und Referenzen

- [ ] **Jede Quelle manuell geprüft** (siehe Schritt 3 oben)
- [ ] **Keine erfundenen Quellen**
- [ ] **DOIs/arXiv-IDs verifiziert**
- [ ] **Aktualität:** Quellen nicht veraltet

### ✅ Compliance

- [ ] **AI-Frontmatter vollständig:**
  - `assisted: true` gesetzt
  - `humanReviewed: true` gesetzt
  - `reviewedBy` ausgefüllt
  - `reviewedAt` ausgefüllt
  - `tools` Liste vorhanden
- [ ] **Datenschutz:** Keine personenbezogenen Daten ohne Einwilligung
- [ ] **Urheberrecht:** Keine Copy-Paste von fremden Texten
- [ ] **Lizenzen:** Code-Beispiele mit Lizenzhinweis

### ✅ SEO & Metadaten

- [ ] **Title:** Prägnant, max. 60 Zeichen
- [ ] **Description:** Aussagekräftig, max. 160 Zeichen
- [ ] **Topics:** Mindestens 1, max. 3 sinnvolle Topics
- [ ] **Tags:** 3-5 relevante Tags
- [ ] **Language:** `de` oder `en` korrekt gesetzt

### ✅ Build-Validierung

```bash
npm run build
# Erwartung: ✅ Build erfolgreich, keine Frontmatter-Fehler

npm test
# Erwartung: ✅ Alle Tests bestanden

npm run lint
# Erwartung: ✅ Keine Linting-Fehler
```

---

## MDX-Komponenten

### Callout-Boxen

```mdx
<Note>
Standard-Hinweis in blauer Box mit Info-Icon.
</Note>

<Warning>
Wichtige Warnung in roter Box mit Warndreieck.
</Warning>

<Architecture>
Architektur-Hinweis in grüner Box.
</Architecture>

<KeyTakeaway>
Wichtigste Erkenntnis in hervorgehobener Box.
</KeyTakeaway>
```

### Code-Blöcke mit Features

````mdx
```typescript title="src/lib/example.ts" {3-5}
const foo = 1;
const bar = 2;
// Zeilen 3-5 werden
// automatisch
// hervorgehoben
```
````

**Unterstützte Sprachen:** `typescript`, `javascript`, `bash`, `yaml`, `json`, `dockerfile`, `python`, `go`, `rust`, `sql`, u.v.m.

### Fußnoten

```mdx
Eine Behauptung mit Quellenangabe.[^1]

Weitere Informationen.[^2]

[^1]: Quelle 1: "Titel", Autor, 2026. https://example.com
[^2]: Quelle 2: "Titel", Autor et al., 2026. DOI: 10.1234/example
```

### Bilder (falls verwendet)

```mdx
![Alt-Text](./images/diagramm.png)
```

Bilder im Ordner `content/blog/images/` ablegen.

---

## Best Practices

### ✅ DOs

- ✅ **URL-freundliche Dateinamen:** `kubernetes-security-best-practices.mdx`
- ✅ **Kurze Beschreibungen:** Max. 160 Zeichen für SEO
- ✅ **Code testen:** Alle Beispiele müssen funktionieren
- ✅ **Quellen prüfen:** Besonders bei KI-generierten Texten
- ✅ **Review durchführen:** Immer `humanReviewed: true` bei assisted: true
- ✅ **Aktuell bleiben:** `updated`-Feld bei späteren Änderungen setzen

### ❌ DON'Ts

- ❌ **Keine Leerzeichen in Dateinamen:** `mein artikel.mdx` ❌
- ❌ **Keine Umlaute:** `über-kubernetes.mdx` ❌ → `ueber-kubernetes.mdx` ✅
- ❌ **Keine fehlenden Pflichtfelder:** Build schlägt fehl
- ❌ **Keine ungeprüften KI-Quellen:** Rechtliche Haftung!
- ❌ **Kein `draft: true` vergessen:** Sonst bleibt Artikel unveröffentlicht
- ❌ **Kein AI-Feld bei neuen Artikeln vergessen:** Build-Fehler ab 2026-08-02

---

## Häufige Fehler

### Build-Fehler: "ai-Feld erforderlich"

```
❌ Artikel ab 2026-08-02 benötigen ein ai-Feld (assisted: true/false)
```

**Lösung:**
```yaml
ai:
  assisted: false  # Oder true, je nach Verwendung
```

### Build-Fehler: "humanReviewed erfordert reviewedBy"

```
❌ humanReviewed: true erfordert reviewedBy und reviewedAt
```

**Lösung:**
```yaml
ai:
  assisted: true
  humanReviewed: true
  reviewedBy: "Dominik Hahn"  # ✅ Name hinzufügen
  reviewedAt: "2026-08-02"    # ✅ Datum hinzufügen
```

### Artikel erscheint nicht auf der Website

**Mögliche Ursachen:**
1. ❌ `draft: true` → Auf `false` setzen
2. ❌ Dateiname enthält Leerzeichen/Sonderzeichen
3. ❌ Frontmatter-Validierung fehlgeschlagen → `npm run build` prüfen

### Code-Beispiel wird nicht hervorgehoben

**Problem:** Falsche Sprach-ID

```mdx
❌ ```js-ts   // Falsch
✅ ```typescript  // Korrekt

❌ ```shell  // Falsch
✅ ```bash   // Korrekt
```

---

## Rechtliche Hinweise

### Urheberrecht bei KI-generierten Texten

**Status in Deutschland/EU (2026):**
- ❌ Reine KI-Outputs sind **nicht** urheberrechtlich geschützt
- ✅ Ihre redaktionelle Bearbeitung **kann** Urheberrecht begründen
- ⚠️ 1:1-Übernahme von KI-Output → kein Schutz

**Empfehlung:**
- Substanzielle menschliche Bearbeitung
- Eigene Kommentare/Einschätzungen hinzufügen
- Struktur/Gliederung selbst festlegen

### Haftung für Inhalte

**Sie haften für:**
- ✅ Fachliche Korrektheit
- ✅ Quellenangaben
- ✅ Urheberrechtsverletzungen
- ✅ Datenschutzverstöße

**Auch wenn ChatGPT den Text geschrieben hat!**

### Datenschutz

- ❌ Keine personenbezogenen Daten ohne Einwilligung
- ❌ Keine Screenshots mit Namen/E-Mails
- ❌ Keine API-Keys, Tokens, Passwörter in Code-Beispielen

---

## Hilfe & Support

### Dokumentation

- **Frontmatter-Schema:** `src/lib/content/schema.ts`
- **AI Policy:** `docs/compliance/editorial-ai-policy.md`
- **Datenschutzerklärung:** https://agenticstack.eu/datenschutz

### Build-Befehle

```bash
npm run build          # Statische Site generieren
npm test               # Unit-Tests
npm run lint           # ESLint
npm run dev            # Lokaler Dev-Server
npm run deploy         # Production Deployment
```

### Bei Problemen

1. **Build-Logs prüfen:** `npm run build` zeigt Validierungs-Fehler
2. **Schema checken:** `src/lib/content/schema.ts` → aktuelle Validierungsregeln
3. **Template vergleichen:** `_template.mdx` als Referenz verwenden

---

**Version:** 1.0  
**Stand:** 2. August 2026  
**Letzte Änderung:** Vollständige Überarbeitung mit AI Act Compliance

[^1]: The citation itself, rendered in a "Footnotes" section at the bottom.
```

## Adding a new topic

Topics are a curated, central list (not freeform like tags). Add a new one in
`src/config/topics.ts` before referencing its slug in an article.
