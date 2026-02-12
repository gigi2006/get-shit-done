# Roadmapping Methodology Reference

Detailed methodology for phase identification, goal-backward criteria, and coverage validation.

<philosophy>

## Solo Developer + Claude Workflow

You are roadmapping for ONE person (the user) and ONE implementer (Claude).
- No teams, stakeholders, sprints, resource allocation
- User is the visionary/product owner
- Claude is the builder
- Phases are buckets of work, not project management artifacts

## Anti-Enterprise

NEVER include phases for:
- Team coordination, stakeholder management
- Sprint ceremonies, retrospectives
- Documentation for documentation's sake
- Change management processes

If it sounds like corporate PM theater, delete it.

## Requirements Drive Structure

**Derive phases from requirements. Don't impose structure.**

Bad: "Every project needs Setup -> Core -> Features -> Polish"
Good: "These 12 requirements cluster into 4 natural delivery boundaries"

Let the work determine the phases, not a template.

## Goal-Backward at Phase Level

**Forward planning asks:** "What should we build in this phase?"
**Goal-backward asks:** "What must be TRUE for users when this phase completes?"

Forward produces task lists. Goal-backward produces success criteria that tasks must satisfy.

## Coverage is Non-Negotiable

Every v1 requirement must map to exactly one phase. No orphans. No duplicates.

If a requirement doesn't fit any phase -> create a phase or defer to v2.
If a requirement fits multiple phases -> assign to ONE (usually the first that could deliver it).

</philosophy>

<goal_backward_phases>

## Deriving Phase Success Criteria

For each phase, ask: "What must be TRUE for users when this phase completes?"

**Step 1: State the Phase Goal**
Take the phase goal from your phase identification. This is the outcome, not work.

- Good: "Users can securely access their accounts" (outcome)
- Bad: "Build authentication" (task)

**Step 2: Derive Observable Truths (2-5 per phase)**
List what users can observe/do when the phase completes.

For "Users can securely access their accounts":
- User can create account with email/password
- User can log in and stay logged in across browser sessions
- User can log out from any page
- User can reset forgotten password

**Test:** Each truth should be verifiable by a human using the application.

**Step 3: Cross-Check Against Requirements**
For each success criterion:
- Does at least one requirement support this?
- If not -> gap found

For each requirement mapped to this phase:
- Does it contribute to at least one success criterion?
- If not -> question if it belongs here

**Step 4: Resolve Gaps**
Success criterion with no supporting requirement:
- Add requirement to REQUIREMENTS.md, OR
- Mark criterion as out of scope for this phase

Requirement that supports no criterion:
- Question if it belongs in this phase
- Maybe it's v2 scope
- Maybe it belongs in different phase

## Example Gap Resolution

```
Phase 2: Authentication
Goal: Users can securely access their accounts

Success Criteria:
1. User can create account with email/password <- AUTH-01 checkmark
2. User can log in across sessions <- AUTH-02 checkmark
3. User can log out from any page <- AUTH-03 checkmark
4. User can reset forgotten password <- ??? GAP

Requirements: AUTH-01, AUTH-02, AUTH-03

Gap: Criterion 4 (password reset) has no requirement.

Options:
1. Add AUTH-04: "User can reset password via email link"
2. Remove criterion 4 (defer password reset to v2)
```

</goal_backward_phases>

<phase_identification>

## Deriving Phases from Requirements

**Step 1: Group by Category**
Requirements already have categories (AUTH, CONTENT, SOCIAL, etc.).
Start by examining these natural groupings.

**Step 2: Identify Dependencies**
Which categories depend on others?
- SOCIAL needs CONTENT (can't share what doesn't exist)
- CONTENT needs AUTH (can't own content without users)
- Everything needs SETUP (foundation)

**Step 3: Create Delivery Boundaries**
Each phase delivers a coherent, verifiable capability.

Good boundaries:
- Complete a requirement category
- Enable a user workflow end-to-end
- Unblock the next phase

Bad boundaries:
- Arbitrary technical layers (all models, then all APIs)
- Partial features (half of auth)
- Artificial splits to hit a number

**Step 4: Assign Requirements**
Map every v1 requirement to exactly one phase.
Track coverage as you go.

## Phase Numbering

**Integer phases (1, 2, 3):** Planned milestone work.

**Decimal phases (2.1, 2.2):** Urgent insertions after planning.
- Created via `/gsd:insert-phase`
- Execute between integers: 1 -> 1.1 -> 1.2 -> 2

**Starting number:**
- New milestone: Start at 1
- Continuing milestone: Check existing phases, start at last + 1

## Depth Calibration

Read depth from config.json. Depth controls compression tolerance.

| Depth | Typical Phases | What It Means |
|-------|----------------|---------------|
| Quick | 3-5 | Combine aggressively, critical path only |
| Standard | 5-8 | Balanced grouping |
| Comprehensive | 8-12 | Let natural boundaries stand |

**Key:** Derive phases from work, then apply depth as compression guidance. Don't pad small projects or compress complex ones.

## Good Phase Patterns

**Foundation -> Features -> Enhancement**
```
Phase 1: Setup (project scaffolding, CI/CD)
Phase 2: Auth (user accounts)
Phase 3: Core Content (main features)
Phase 4: Social (sharing, following)
Phase 5: Polish (performance, edge cases)
```

**Vertical Slices (Independent Features)**
```
Phase 1: Setup
Phase 2: User Profiles (complete feature)
Phase 3: Content Creation (complete feature)
Phase 4: Discovery (complete feature)
```

**Anti-Pattern: Horizontal Layers**
```
Phase 1: All database models <- Too coupled
Phase 2: All API endpoints <- Can't verify independently
Phase 3: All UI components <- Nothing works until end
```

</phase_identification>

<coverage_validation>

## 100% Requirement Coverage

After phase identification, verify every v1 requirement is mapped.

**Build coverage map:**

```
AUTH-01 -> Phase 2
AUTH-02 -> Phase 2
AUTH-03 -> Phase 2
PROF-01 -> Phase 3
PROF-02 -> Phase 3
CONT-01 -> Phase 4
CONT-02 -> Phase 4
...

Mapped: 12/12 checkmark
```

**If orphaned requirements found:**

```
Warning: Orphaned requirements (no phase):
- NOTF-01: User receives in-app notifications
- NOTF-02: User receives email for followers

Options:
1. Create Phase 6: Notifications
2. Add to existing Phase 5
3. Defer to v2 (update REQUIREMENTS.md)
```

**Do not proceed until coverage = 100%.**

## Traceability Update

After roadmap creation, REQUIREMENTS.md gets updated with phase mappings:

```markdown
## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| AUTH-01 | Phase 2 | Pending |
| AUTH-02 | Phase 2 | Pending |
| PROF-01 | Phase 3 | Pending |
...
```

</coverage_validation>
