<purpose>
3-round adversarial verification with Defender, Attacker, and Auditor agents.
Round 1: Independent assessment (parallel). Round 2: Structured debate. Round 3: Consensus (if needed).
Produces higher confidence than single-verifier approach by forcing evidence-based argumentation.
</purpose>

<core_principle>
Three independent perspectives — advocate, adversary, neutral judge — eliminate blind spots that a single verifier misses. Each agent works from the same evidence but with different objectives, producing a robust verdict through structured debate.
</core_principle>

<process>

<step name="initialize" priority="first">
Load phase context:

```bash
INIT=$(node ~/.claude/get-shit-done/bin/gsd-tools.js init verify-work "${PHASE_ARG}" --include state,roadmap)
```

Parse JSON for: `phase_dir`, `phase_number`, `phase_name`, `verifier_model`.

All 3 verification agents use the same model (from config or default opus).
</step>

<step name="write_context_brief">
Write `{phase_dir}/.context-brief.md` — a lightweight pointer file so agents know what to read:

```markdown
# Verification Context — Phase {X}: {Name}

## Phase Goal
{goal from ROADMAP.md}

## Phase Directory
{phase_dir}

## Files to Read
- PLAN.md and SUMMARY.md files in {phase_dir}/
- CLAUDE.md (project conventions)
- REQUIREMENTS.md or ROADMAP.md (requirements)
- Actual source code in src/ (verify claims against reality)
```

This keeps orchestrator context lean — agents read the actual files themselves.
</step>

<step name="round_1_independent">
Report:
```
## Adversarial Verification — Phase {X}: {Name}

Round 1: Independent Assessment (parallel)
  Defender  — builds evidence case
  Attacker  — red-teams for gaps
  Auditor   — neutral compliance audit

Spawning 3 agents...
```

Spawn all 3 in parallel via Task():

```
Task(
  subagent_type="gsd-defender",
  prompt="You are the Defender for Phase {phase}: {phase_name}.
Read context brief: {phase_dir}/.context-brief.md
Read all PLAN.md and SUMMARY.md files in {phase_dir}/.
Read CLAUDE.md and REQUIREMENTS.md/ROADMAP.md from project root.
Examine actual source code to verify claims.
Write DEFENSE.md to {phase_dir}/.",
  description="Defend Phase {phase}"
)

Task(
  subagent_type="gsd-attacker",
  prompt="You are the Attacker for Phase {phase}: {phase_name}.
Read context brief: {phase_dir}/.context-brief.md
Read all PLAN.md and SUMMARY.md files in {phase_dir}/.
Read CLAUDE.md and REQUIREMENTS.md/ROADMAP.md from project root.
Examine actual source code — run all 5 attacks.
Write ATTACK.md to {phase_dir}/.",
  description="Attack Phase {phase}"
)

Task(
  subagent_type="gsd-auditor",
  prompt="You are the Auditor for Phase {phase}: {phase_name}.
Read context brief: {phase_dir}/.context-brief.md
Read all PLAN.md and SUMMARY.md files in {phase_dir}/.
Read CLAUDE.md and REQUIREMENTS.md/ROADMAP.md from project root.
Examine actual source code and git log.
Write AUDIT.md to {phase_dir}/.",
  description="Audit Phase {phase}"
)
```

Wait for all 3 to complete. Verify outputs exist:
```bash
for f in DEFENSE ATTACK AUDIT; do
  [ -f "${PHASE_DIR}/${f}.md" ] && echo "OK: ${f}.md" || echo "MISSING: ${f}.md"
done
```

If any report missing: retry that agent once. If still missing: proceed with available reports.
</step>

<step name="consensus_detection">
Before debate, check for consensus findings — issues flagged by all 3 roles.

Read only finding IDs/titles from each report (not full content — stay lean).
Cross-reference to find issues all 3 mention.

Consensus findings skip debate — they're automatically confirmed.
</step>

<step name="round_2_debate">
**Skip if:** Attacker has no HIGH or CRITICAL findings (only MEDIUM/LOW).

If debate needed:

Report: `Round 2: Structured Debate — {N} findings to debate`

Spawn debate moderator:
```
Task(
  subagent_type="gsd-auditor",
  prompt="You are the debate moderator for Phase {phase}: {phase_name}.
Read DEFENSE.md, ATTACK.md, AUDIT.md from {phase_dir}/.
For each HIGH/CRITICAL non-consensus finding from ATTACK.md:
  1. Present Attacker's evidence
  2. Check Defender's counter-evidence from DEFENSE.md
  3. Rule: SUSTAINED (Attacker wins) / OVERRULED (Defender wins) / SPLIT (partial)
Write DEBATE.md to {phase_dir}/.",
  description="Debate Phase {phase}"
)
```
</step>

<step name="round_3_consensus">
Only if DEBATE.md has SPLIT rulings. Orchestrator resolves based on:
1. Weight of evidence (file:line references vs. assertions)
2. Severity precedent (safety > functionality > style)
3. Consensus overlap (2 of 3 agree wins)

Most phases won't need Round 3.
</step>

<step name="synthesize_verdict">
Report: `Synthesizing verdict...`

Spawn verdict synthesis (keeps orchestrator lean):
```
Task(
  subagent_type="gsd-verifier",
  prompt="Synthesize adversarial verification verdict for Phase {phase}: {phase_name}.
Read from {phase_dir}/: DEFENSE.md, ATTACK.md, AUDIT.md, DEBATE.md (if exists).
Read context brief: {phase_dir}/.context-brief.md

Verdict thresholds:
- PASS: Compliance >= 85%, no unresolved CRITICAL/HIGH findings
- CONDITIONAL_PASS: Compliance >= 70%, no unresolved CRITICAL, max 2 HIGH
- FAIL: Compliance < 70% OR any unresolved CRITICAL

Write {padded_phase}-VERIFICATION.md to {phase_dir}/ with:
- Verdict (PASS/CONDITIONAL_PASS/FAIL)
- Compliance score from AUDIT.md
- Confirmed findings (from consensus + sustained debate rulings)
- Dismissed findings (overruled)
- Recommendations

Return ONE line: VERDICT={PASS|CONDITIONAL_PASS|FAIL} score={N}% gaps={N}",
  description="Synthesize verdict Phase {phase}"
)
```

Parse verdict line from agent return.
</step>

<step name="commit">
```bash
FILES="${PHASE_DIR}/DEFENSE.md ${PHASE_DIR}/ATTACK.md ${PHASE_DIR}/AUDIT.md ${PHASE_DIR}/${PADDED_PHASE}-VERIFICATION.md"
[ -f "${PHASE_DIR}/DEBATE.md" ] && FILES="$FILES ${PHASE_DIR}/DEBATE.md"
[ -f "${PHASE_DIR}/.context-brief.md" ] && FILES="$FILES ${PHASE_DIR}/.context-brief.md"

node ~/.claude/get-shit-done/bin/gsd-tools.js commit "docs(phase-${PHASE}): adversarial verification — ${VERDICT}" --files $FILES
```
</step>

<step name="report">
**PASS:**
```
## Verification: PASS

Phase {X}: {Name} — {score}% compliance
All must-haves verified. No unresolved critical findings.
```

**CONDITIONAL_PASS:**
```
## Verification: CONDITIONAL PASS

Phase {X}: {Name} — {score}% compliance
{N} findings to address (non-blocking).

Options:
1. Accept and continue
2. `/gsd:plan-phase {X} --gaps` to fix findings
```

**FAIL:**
```
## Verification: FAIL

Phase {X}: {Name} — {score}% compliance
{N} unresolved findings ({critical} critical, {high} high).

Next: `/gsd:plan-phase {X} --gaps`
`/clear` first for fresh context
```
</step>

</process>

<context_efficiency>
Orchestrator stays under 15% context:
- Writes context brief (pointer file) instead of loading full reports
- Agents read their own files with fresh 200k context
- Verdict synthesis delegated to subagent — orchestrator receives only the verdict line
- No full report content flows through orchestrator
</context_efficiency>
