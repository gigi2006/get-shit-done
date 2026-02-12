# Research Methodology Reference

Shared methodology for all GSD researcher agents (gsd-phase-researcher, gsd-project-researcher).

## Philosophy

### Training Data = Hypothesis

Claude's training is 6-18 months stale. Treat pre-existing knowledge as hypothesis, not fact.

**The trap:** Claude "knows" things confidently, but knowledge may be outdated, incomplete, or wrong.

**The discipline:**
1. **Verify before asserting** — don't state library capabilities without checking Context7 or official docs
2. **Date your knowledge** — "As of my training" is a warning flag
3. **Prefer current sources** — Context7 and official docs trump training data
4. **Flag uncertainty** — LOW confidence when only training data supports a claim

### Honest Reporting

Research value comes from accuracy, not completeness theater.

**Report honestly:**
- "I couldn't find X" is valuable (now we know to investigate differently)
- "This is LOW confidence" is valuable (flags for validation)
- "Sources contradict" is valuable (surfaces real ambiguity)

**Avoid:** Padding findings, stating unverified claims as facts, hiding uncertainty behind confident language.

### Investigation, Not Confirmation

**Bad research:** Start with hypothesis, find evidence to support it
**Good research:** Gather evidence, form conclusions from evidence

When researching "best library for X": find what the ecosystem actually uses, document tradeoffs honestly, let evidence drive recommendation. Don't find articles supporting your initial guess.

## Tool Strategy

### Tool Priority Order

| Priority | Tool | Use For | Trust Level |
|----------|------|---------|-------------|
| 1st | Context7 | Library APIs, features, configuration, versions | HIGH |
| 2nd | WebFetch | Official docs/READMEs not in Context7, changelogs, release notes | HIGH-MEDIUM |
| 3rd | WebSearch | Ecosystem discovery, community patterns, pitfalls | Needs verification |

### Context7 Flow

Authoritative, current, version-aware documentation.

```
1. mcp__context7__resolve-library-id with libraryName: "[library]"
2. mcp__context7__query-docs with resolved ID + specific query
```

Resolve first (don't guess IDs). Use specific queries. Trust over training data.

### WebFetch — Authoritative Sources

For libraries not in Context7, changelogs, release notes, official announcements.

Use exact URLs (not search result pages). Check publication dates. Prefer /docs/ over marketing.

### WebSearch — Ecosystem Discovery

For finding what exists, community patterns, real-world usage.

**Query templates:**
```
Ecosystem: "[tech] best practices [current year]", "[tech] recommended libraries [current year]"
Patterns:  "how to build [type] with [tech]", "[tech] architecture patterns"
Problems:  "[tech] common mistakes", "[tech] gotchas"
```

Always include current year. Use multiple query variations. Mark WebSearch-only findings as LOW confidence.

### Verification Protocol

**WebSearch findings MUST be verified:**

```
For each finding:
1. Verify with Context7? YES → HIGH confidence
2. Verify with official docs? YES → MEDIUM confidence
3. Multiple sources agree? YES → Increase one level
   Otherwise → LOW confidence, flag for validation
```

**Never present LOW confidence findings as authoritative.**

## Source Hierarchy

| Level | Sources | Use |
|-------|---------|-----|
| HIGH | Context7, official documentation, official releases | State as fact |
| MEDIUM | WebSearch verified with official source, multiple credible sources agree | State with attribution |
| LOW | WebSearch only, single source, unverified | Flag as needing validation |

**Source priority:** Context7 → Official Docs → Official GitHub → WebSearch (verified) → WebSearch (unverified)

## Verification Protocol

### Known Pitfalls

#### Configuration Scope Blindness
**Trap:** Assuming global configuration means no project-scoping exists
**Prevention:** Verify ALL configuration scopes (global, project, local, workspace)

#### Deprecated Features
**Trap:** Finding old documentation and concluding feature doesn't exist
**Prevention:** Check current official docs, review changelog, verify version numbers and dates

#### Negative Claims Without Evidence
**Trap:** Making definitive "X is not possible" statements without official verification
**Prevention:** For any negative claim — is it verified by official docs? Have you checked recent updates? Are you confusing "didn't find it" with "doesn't exist"?

#### Single Source Reliance
**Trap:** Relying on a single source for critical claims
**Prevention:** Require multiple sources: official docs (primary), release notes (currency), additional source (verification)

### Pre-Submission Checklist

- [ ] All domains investigated (stack, features, architecture, patterns, pitfalls)
- [ ] Negative claims verified with official docs
- [ ] Multiple sources cross-referenced for critical claims
- [ ] URLs provided for authoritative sources
- [ ] Publication dates checked (prefer recent/current)
- [ ] Confidence levels assigned honestly
- [ ] "What might I have missed?" review completed
