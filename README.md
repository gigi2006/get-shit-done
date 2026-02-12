# GSD Fork -- Meine Anpassungen

Persoenlicher Fork von [GSD (Get Shit Done)](https://github.com/glittercowboy/get-shit-done) mit Optimierungen fuer deutschsprachige Workflows, eigenen Agenten und Qualitaetsverbesserungen.

- **Original-Repo:** <https://github.com/glittercowboy/get-shit-done>
- **Dieser Fork:** <https://github.com/gigi2006/get-shit-done>

---

## Was ist das?

GSD ist ein Meta-Prompting-System fuer Claude Code. Es loest das Problem "Context Rot" -- den Qualitaetsverlust, der entsteht, wenn Claude sein Kontextfenster fuellt. GSD strukturiert Projekte in Phasen, nutzt Sub-Agenten mit frischem Kontext und erzeugt atomare Git-Commits.

Dieser Fork enthaelt persoenliche Anpassungen, die GSD besser fuer deutschsprachige Projekte und meinen Workflow machen. Die Kern-Funktionalitaet (`/gsd:*`-Befehle) bleibt identisch zum Original.

---

## Was wurde angepasst?

### GSD-System (`get-shit-done/`)

1. **Portable Pfade**: Alle 48 Dateien von hartcodierten `C:/Users/Giorgo/.claude/get-shit-done` auf `~/.claude/get-shit-done` umgestellt -- funktioniert auf jedem Rechner.

2. **Deutsche Beschreibungen**: Alle 11 GSD-Agent-Beschreibungen ins Deutsche uebersetzt (Frontmatter `description:`-Feld).

3. **`config.json`**: `skip_checkpoints: false` -- Checkpoints werden jetzt respektiert (Upstream hatte sie deaktiviert).

4. **Auto-Pruning in `gsd-tools.js`**: `STATE.md` bleibt schlank -- maximal 20 Entscheidungen, maximal 10 Metriken, abgeschlossene Phasen werden komprimiert.

5. **Executor Self-Check erweitert**: Erkennt Stub-Dateien (< 5 Zeilen) und `TODO`/`FIXME`-Marker -- verhindert hohle Completions.

6. **Spot-Check verbessert**: `execute-phase` prueft jetzt ALLE erstellten Dateien, nicht nur die ersten 2.

7. **Research-Methodologie**: Gemeinsame `references/research-methodology.md` eliminiert Duplizierung zwischen `gsd-phase-researcher` und `gsd-project-researcher`.

8. **Neuer Workflow: `abort-phase.md`**: 3 Strategien (Rollback, Keep+Reset, Full Reset) zum sicheren Abbruch einer Phase.

9. **Integration-Checker**: Framework-agnostischer Hinweis hinzugefuegt -- nimmt nicht mehr React/Next.js als Standard an.

10. **Debugger Fix**: `WebSearch` zu `WebFetch` in der Frontmatter-Tools-Liste korrigiert.

11. **Adversarielle Verifikation**: 3-Agenten-System (Defender, Attacker, Auditor) ersetzt den Single-Verifier. Produziert DEFENSE.md, ATTACK.md, AUDIT.md + optionales DEBATE.md. Verdict: PASS / CONDITIONAL_PASS / FAIL.

12. **File Ownership**: Plan-Frontmatter deklariert OWN/SHARED/READ-ONLY pro Datei -- verhindert Merge-Konflikte bei paralleler Ausfuehrung.

13. **Retrospektive**: Nach Phase-Verifikation werden 2-3 Konventionen aus SUMMARY-Dateien extrahiert und dem User zur Aufnahme in CLAUDE.md vorgeschlagen.

### Custom Agents (`agents/`)

26 Agenten: 22 Domaenen-Agenten (Accessibility, Astro, Docker, SEO, Security usw.) + 3 Verifikations-Agenten (Defender, Attacker, Auditor) + 1 Router-Agent fuer automatische Agenten-Auswahl.

Boilerplate ist in `AGENT-RULES.md` zentralisiert -- Agenten enthalten nur `<role>` und `<context>`. Keine Emojis in Output-Templates (konfigurierbar).

### AGENT-RULES.md

Globale Koordinationsregeln fuer alle Agenten:

- GSD Extended Tags dokumentiert (`execution_flow`, `philosophy` usw.)
- Standard-Sektionen (`instructions`, `constraints`, `output_format`, `success_criteria`) als Defaults definiert
- Regel: "Plan vor Aktion" -- Ansatz beschreiben, bevor nicht-triviale Aenderungen gemacht werden
- Regel: "Keine Emojis", ausser auf explizite Anfrage

### CLAUDE.md (Globale Instruktionen)

v3-optimiert: ca. 160 Zeilen, kombiniert essentiellen Kontext aus v1 (347 Zeilen) mit schlanker Struktur von v2 (107 Zeilen).

Sektionen: Identitaet, Umgebung (Windows), Tech-Stack Defaults, Qualitaetsstandards, Versions-Policy, Workflows (Standard/Lite/GSD), Multi-AI-Schutz, Git-Konventionen, Fehlerkorrektur.

Kernregel: User ist "kein Programmierer" -- AI schreibt den gesamten Code, Empfehlungen sind Pflicht.

---

## Installation / Nutzung

Dies ist KEIN Ersatz fuer die Standard-GSD-Installation. Es ist ein persoenlicher Fork mit Zusatzdateien.

### 1. Fork klonen

```bash
git clone https://github.com/gigi2006/get-shit-done.git
```

### 2. Dateien nach `~/.claude/` kopieren

```bash
# GSD-System
cp -r get-shit-done/get-shit-done/ ~/.claude/get-shit-done/

# Custom Agents
cp -r get-shit-done/agents/ ~/.claude/agents/

# Globale Instruktionen
cp get-shit-done/CLAUDE.md ~/.claude/CLAUDE.md

# Agent-Regeln
cp get-shit-done/AGENT-RULES.md ~/.claude/agents/AGENT-RULES.md
```

### 3. Fertig

GSD-Befehle (`/gsd:new-project`, `/gsd:plan-phase`, `/gsd:execute-phase` usw.) funktionieren wie gewohnt.

---

## Upstream-Updates

So werden Aenderungen vom Original-Repo uebernommen:

```bash
git remote add upstream https://github.com/glittercowboy/get-shit-done.git
git fetch upstream
git merge upstream/main
```

- Custom-Dateien (`agents/`, `AGENT-RULES.md`, `CLAUDE.md`) liegen in anderen Verzeichnissen als Upstream -- Konflikte sind selten.
- Falls Konflikte in `get-shit-done/`-Dateien auftreten: manuell loesen, eigene Anpassungen beibehalten.
- Nach dem Merge: aktualisierte Dateien erneut nach `~/.claude/` kopieren.

---

## Agenten-Uebersicht

### Domaenen-Agenten

| Agent | Bereich | Trigger |
|-------|---------|---------|
| `accessibility-expert` | WCAG, BITV, BFSG | Barrierefreiheit, Screenreader |
| `ai-toolsmith` | Claude Code, Prompts, MCP | Agent erstellen, Prompt verbessern |
| `api-architect` | Flask, FastAPI, REST | API bauen, Endpoint |
| `astro-expert` | Astro Framework | Astro, Build, Komponenten |
| `code-debugger` | Bugs, Fehler | geht nicht, Fehler, kaputt |
| `content-auditor` | Textqualitaet, Recht | Rechtschreibung, Impressum |
| `database-expert` | MongoDB, PostgreSQL, SQLite | Query, Schema, DB |
| `docker-architect` | Docker, Traefik, Self-Hosting | Container, Docker Compose |
| `dotnet-expert` | C#, .NET, ASP.NET | .NET, C#, Visual Studio |
| `openclaw-expert` | OpenClaw AI | Claw, Skills, Channels |
| `agent-router` | Automatische Agenten-Auswahl | welche Agenten, Projektstart |
| `orchestrator` | Projektleitung (mit Auto-Detection) | optimiere alles, Status |
| `project-profiler` | Projektanalyse | Profil erstellen, Projektstart |
| `project-scanner` | Tech-Stack Scan | analysiere Projekt, Dependencies |
| `python-expert` | Python, pip, Automation | Python Script, pip |
| `security-scanner` | Credentials, Secrets | pruefe Credentials, Open-Source |
| `seo-optimizer` | SEO, Meta, Schema.org | SEO, Meta-Tags, Sitemap |
| `server-admin` | Linux VPS, SSH | Server einrichten, SSH |
| `site-auditor` | Qualitaetspruefung | Validierung, Score, Audit |
| `speed-optimizer` | Performance, Web Vitals | langsam, zu gross, Ladezeit |
| `template-architect` | Open-Source Templates | Template, Open Source |
| `ui-designer` | UI/UX, Tailwind | Design, Styling, Responsive |
| `windows-admin` | Windows, PowerShell, Office | PC einrichten, PowerShell |

### Verifikations-Agenten (adversarielle Phase-Pruefung)

| Agent | Rolle | Output |
|-------|-------|--------|
| `gsd-defender` | Baut Beweislage fuer Phase-Zielerreichung auf | DEFENSE.md |
| `gsd-attacker` | Red-Team: findet Stubs, Wiring-Luecken, Security-Issues | ATTACK.md |
| `gsd-auditor` | Neutraler Compliance-Audit mit gewichtetem Scoring | AUDIT.md + DEBATE.md |

Diese 3 Agenten werden automatisch von `execute-phase` nach Abschluss aller Plans gestartet. Kein manueller Aufruf noetig.

---

## Lizenz

MIT License (identisch mit Upstream). Siehe [LICENSE](LICENSE).
