---
name: agent-rules
description: "Globale Koordinationsregeln für alle Agenten. Wird automatisch geladen."
---

# Agenten-Koordinationsregeln

> Allgemeine Regeln (Sprache, Versionen, Qualität, Kommunikation) stehen in der globalen CLAUDE.md. Hier stehen NUR agenten-spezifische Koordinationsregeln.

> **Standard-Sektionen:** Diese Regeln gelten als Default-`<instructions>`, `<constraints>`, `<output_format>` und `<success_criteria>` für alle Agenten. Einzelne Agenten-Dateien brauchen diese Sektionen NICHT zu wiederholen — sie enthalten nur `<role>` und `<context>`.

## 1. Prompt-Struktur (Phase-0 Standard)

Jede Agenten-Datei nutzt diese XML-Sektionen im Body:
- `<role>`
- `<instructions>`
- `<constraints>`
- `<output_format>`
- `<context>`
- `<success_criteria>`

**Hinweis:** GSD-Agenten (gsd-*.md) werden als Subagenten-Prompts via `Task()` gespawnt und verwenden einen erweiterten Sektions-Standard. Sie müssen NICHT die Minimal-Sektionen der Standard-Agenten einhalten. Erlaubte erweiterte Tags:
- `<execution_flow>`, `<structured_returns>` — Ablauf und Rückgabeformat
- `<philosophy>`, `<methodology_reference>` — Forschungs-/Planungsmethodik
- `<core_principle>`, `<upstream_input>`, `<downstream_consumer>` — Datenfluss
- `<self_check>`, `<state_updates>`, `<final_commit>` — Qualitätssicherung
- `<completion_format>`, `<checkpoint_behavior>` — Abschluss und Checkpoints
- Prompt-Inhalt bleibt auf Englisch (bessere Claude-Verarbeitung), nur `description:` im Frontmatter auf Deutsch.

## 2. Plan vor Aktion

Vor nicht-trivialen Änderungen: kurz den Ansatz beschreiben (2-5 Zeilen), dann umsetzen.
Nicht bei Einzeilern oder offensichtlichen Fixes nötig.

## 4. Memory Hygiene (Auto-Pruning)

Wenn WebFetch neue Fakten liefert (Versionen, Links, Policies):
- veraltete Einträge in MEMORY.md entfernen oder markieren
- MEMORY.md <= 200 Zeilen halten

## 5. Kontext laden (optional, nicht Pflicht)

- Falls `PROJECT-CONTEXT.md` im Projekt existiert: lies sie, passe dich an.
- Falls nicht: arbeite mit dem was du hast. Frag NICHT danach, blockiere NICHT.
- Falls nötig: erstelle sie selbst oder empfehle `@project-profiler`.

## 6. Nichts ist hardcoded

- Alle Versionsnummern und Image-Tags in Agenten-Dateien sind **Beispiele**.
- Vor jeder konkreten Empfehlung: via WebFetch aktuelle stable Version prüfen.
- Veraltete Empfehlung entdeckt: sofort korrigieren + MEMORY.md updaten.

## 7. Standalone-Fähigkeit

- Jeder Agent funktioniert eigenständig — ohne Orchestrator, ohne andere Agenten.
- Jeder Agent kann aber auch im Team arbeiten wenn der Orchestrator delegiert.
- Agenten rufen sich NICHT gegenseitig auf — nur der Orchestrator delegiert.

## 8. Ausgabe-Standards

- Alle Ausgaben auf Deutsch (Code/Variablen auf Englisch).
- Keine Kommentare im Code (außer explizit angefordert).
- Keine Emojis (außer explizit angefordert).
- Findings immer mit: Was, Wo (Datei+Zeile), Warum, Fix, Schweregrad.
- Schweregrade: `KRITISCH` > `HOCH` > `MITTEL` > `NIEDRIG`.

## 9. Änderungen am Code

- Minimale, gezielte Änderungen — nie ganze Dateien umschreiben.
- Vor destruktiven Änderungen: Backup oder auskommentieren.
- Nach Änderungen: Build/Test empfehlen.
- Bestehende Patterns respektieren (Multi-AI-Schutz aus CLAUDE.md beachten).
- Keine Credentials hardcoden (es sei denn, explizit Projektstandard im privaten Repo).
- Bei Design-/Policy-Entscheidungen: stoppen und nachfragen.

## 10. Memory nutzen

- Verzeichnis: `~/.claude/agent-memory/<agent-name>/`
- `MEMORY.md` wird in System-Prompt geladen (max. 200 Zeilen).
- Recherche-Ergebnisse, Lessons Learned, Projektspezifisches speichern.
- Veraltetes löschen.
