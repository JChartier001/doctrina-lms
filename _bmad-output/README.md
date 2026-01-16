# BMAD Documentation for Doctrina LMS

**Project**: Doctrina LMS - Medical Aesthetics Learning Marketplace
**Phase**: Implementation (Phase 4)
**Sprint**: Sprint 2 - Backend Development
**Last Updated**: 2026-01-15

---

## Welcome to BMAD Documentation

This folder contains **BMAD-specific documentation** for the Doctrina LMS project. BMAD agents reference these files when planning, implementing, and reviewing work.

---

## Quick Start

### New BMAD Agent? Start Here 👋

1. **Read first**: `project-context.md` - The bible for this project
2. **Check sprint**: `active-sprint.md` - What we're working on now
3. **Understand transition**: `migration-from-droidz.md` - How we got here

### Starting a Story?

1. Read `project-context.md`
2. Read `active-sprint.md` → Find current story
3. Read epic summary in `epic-summaries/[epic-name].md`
4. Read story file in `docs/stories/story-XXX.md`
5. Read story context XML in `docs/stories/story-context-XXX.xml`

### Making an Architectural Decision?

1. Check `architecture-decisions/` for existing decisions
2. Create new ADR if needed
3. Update `project-context.md` if patterns change

---

## Documentation Structure

```
_bmad-output/
├── README.md                           # THIS FILE
├── project-context.md                  # ⭐ CRITICAL - Read this first
├── active-sprint.md                    # What needs to be done next
├── migration-from-droidz.md            # Transition guide
├── planning-docs/                      # PRD, Architecture, Epics, etc.
│   ├── README.md
│   ├── PRD.md
│   ├── ARCHITECTURE.md
│   ├── DB-SCHEMA.md
│   ├── EPICS.md
│   └── [18 other planning docs]
├── implementation-artifacts/           # Sprint tracking, stories, patterns
│   ├── sprint-status.yaml
│   ├── bmm-workflow-status.md
│   ├── stories/
│   └── patterns/
├── epic-summaries/                     # Epic technical context
│   ├── README.md
│   ├── epic-101-progress-tracking.md
│   └── epic-102-quiz-system.md
└── architecture-decisions/             # Key technical decisions (ADRs)
    ├── README.md
    ├── 001-convex-as-backend.md
    ├── 002-nextjs-app-router.md
    └── 005-usestate-less-pattern.md
```

---

## Core Documents

### 1. project-context.md ⭐ CRITICAL

**Purpose**: The **bible** that all BMAD agents reference

**Contains**:

- Tech stack overview
- Critical coding standards
- Architecture principles
- Database schema summary
- Testing requirements
- Anti-patterns (what NOT to do)
- Medical aesthetics domain constraints

**When to Read**: Before implementing ANY code

### 2. active-sprint.md

**Purpose**: Guide for what work remains

**Contains**:

- Incomplete work priorities
- Next stories to implement
- What needs to be built (not status tracking)
- Epic context references

**When to Read**: Before starting new work

**Note**: For official status tracking, see `implementation-artifacts/sprint-status.yaml`

### 3. migration-from-droidz.md

**Purpose**: Explain the transition from Droidz to BMAD

**Contains**:

- What Droidz was
- What BMAD is
- Key differences
- What stayed the same
- What changed
- Migration checklist
- Common questions

**When to Read**: Once, for context on project history

---

## Supplementary Folders

### epic-summaries/

**Purpose**: Technical context for each epic

**Files**:

- `epic-101-progress-tracking.md` (13 pts, Sprint 2)
- `epic-102-quiz-system.md` (25 pts, Sprint 2-3)
- More to be created as epics are planned

**Contains per Epic**:

- Problem statement
- Technical architecture (schema, functions)
- Stories breakdown with status
- Testing strategy
- Security considerations
- Performance considerations
- Dependencies

**When to Read**: Before starting any story in that epic

### architecture-decisions/

**Purpose**: Document key technical decisions (ADRs)

**Files**:

- `001-convex-as-backend.md` - Why Convex
- `002-nextjs-app-router.md` - Why App Router
- `005-usestate-less-pattern.md` - Why avoid useState for server data
- More ADRs to be added as decisions are made

**Contains per ADR**:

- Context (why decision was needed)
- Decision made
- Rationale (why this over alternatives)
- Consequences (positive and negative)
- Alternatives considered

**When to Read**: When making architectural choices, or challenging existing patterns

---

## Relationship to Other Documentation

### BMAD Docs vs Project Docs

- **BMAD Docs** (`_bmad-output/`): Agent-focused, implementation context
- **Project Docs** (`docs/`): Product requirements, epics, stories, sprint tracking

### BMAD Docs vs Legacy Droidz Docs

- **BMAD Docs** (`_bmad-output/`): New workflow system, current patterns
- **Droidz Docs** (`droidz/`): Legacy workflow, still valid reference for standards

### BMAD Docs vs Standards

- **BMAD Docs**: Project-specific patterns and context
- **Standards** (`.claude/standards/`): Tech-specific patterns (Next.js, React, Convex, TypeScript)

---

## BMAD Agent Workflows

### For Dev Agent (Amelia)

**Before Starting Story**:

1. Check `active-sprint.md` → Next story
2. Read `project-context.md` → Patterns
3. Read epic summary → Architecture
4. Read story file → Tasks
5. Read story context XML → Implementation details

**During Implementation**:

- Follow TDD red-green-refactor
- Reference `project-context.md` for patterns
- Check architecture decisions for guidance
- Ensure 100% test coverage

**After Completion**:

- Mark story as done
- Update sprint status
- Trigger code review

### For Architect Agent (Winston)

**During Planning**:

- Create epic summaries
- Document architecture decisions (ADRs)
- Update `project-context.md` with new patterns

**During Implementation**:

- Review epic summaries for accuracy
- Update ADRs when decisions change
- Guide dev agents on architectural questions

### For PM Agent (John)

**During Sprint Planning**:

- Update `active-sprint.md` with new sprint info
- Ensure epic summaries exist for planned epics
- Validate story prioritization

**During Sprint**:

- Check `active-sprint.md` for progress
- Update sprint metrics
- Adjust backlog as needed

### For Test Architect Agent (Murat)

**During Story Planning**:

- Review epic summaries for testing strategy
- Ensure test coverage requirements clear

**During Implementation**:

- Validate test coverage
- Review test quality
- Update testing sections in epic summaries

---

## Document Maintenance

### Who Updates What

| Document                      | Primary Owner                 | Update Frequency                         |
| ----------------------------- | ----------------------------- | ---------------------------------------- |
| `project-context.md`          | Architect (Winston)           | After major decisions                    |
| `active-sprint.md`            | SM (Bob), PM (John)           | Daily during sprint                      |
| `migration-from-droidz.md`    | Analyst (Mary)                | Once (historical)                        |
| `epic-summaries/*.md`         | SM (Bob), Architect (Winston) | Per epic creation, after epic completion |
| `architecture-decisions/*.md` | Architect (Winston)           | Per architectural decision               |

### Update Triggers

**project-context.md**:

- New tech stack choice
- New coding pattern adopted
- New anti-pattern discovered
- Major architecture change

**active-sprint.md**:

- Sprint start (new sprint details)
- Story completion (update status)
- Daily progress updates
- Sprint end (metrics summary)

**epic-summaries**:

- Epic planning (initial creation)
- Story completion (mark story status)
- Architecture change (update technical details)
- Epic retrospective (capture learnings)

**architecture-decisions**:

- New architectural decision made
- Existing decision superseded
- Decision deprecated

---

## Tips for Success with BMAD

### 1. Start with project-context.md

**Why**: It's the bible. Everything you need to know is there.

### 2. Use Party Mode for Questions

**How**: `/bmad:core:workflows:party-mode`

**Why**: Agents collaborate to answer questions. Mary finds gaps, Winston designs, Amelia codes.

### 3. Follow TDD Red-Green-Refactor

**Pattern**:

1. Write failing test
2. Write minimal code to pass
3. Refactor while keeping tests green

**Why**: 100% backend coverage is mandatory

### 4. Check Sprint Status Daily

**File**: `active-sprint.md`

**Why**: Priorities shift, blockers appear, stories complete

### 5. Reference Epic Summaries Before Stories

**Why**: Epic summaries provide architecture context that story files lack

### 6. Trust the Architecture Decisions

**Why**: ADRs document why decisions were made. Don't relitigate unless you have new information.

### 7. Update Docs as You Go

**Why**: Stale docs are worse than no docs. Keep them current.

---

## Common Questions

### Q: Where do I find coding standards?

**A**:

1. **Primary**: `.claude/standards/` (Next.js, React, TypeScript, Convex-specific)
2. **Project-specific**: `_bmad-output/project-context.md` (patterns, anti-patterns)
3. **Reference**: `droidz/standards/` (broader patterns, still valid)

### Q: Where are the stories?

**A**: `docs/stories/story-XXX.md` + `docs/stories/story-context-XXX.xml`

### Q: Where is the sprint status?

**A**:

- **YAML (official)**: `docs/sprint-status.yaml`
- **Markdown (human)**: `docs/bmm-workflow-status.md`
- **Active sprint (BMAD)**: `_bmad-output/active-sprint.md`

### Q: Where are the epics?

**A**:

- **High-level**: `docs/EPICS.md` (product-focused)
- **Technical**: `_bmad-output/epic-summaries/` (implementation-focused)

### Q: Where is the PRD?

**A**: `docs/PRD.md`

### Q: Where is the architecture doc?

**A**: `docs/ARCHITECTURE.md`

### Q: Where is the database schema?

**A**: `docs/DB-SCHEMA.md` (full details) + `_bmad-output/project-context.md` (summary)

### Q: How do I create a new ADR?

**A**:

1. Copy template from `architecture-decisions/README.md`
2. Number sequentially (check last ADR)
3. Write clearly
4. Commit to repo
5. Update `architecture-decisions/README.md`

---

## Feedback & Issues

### Found a Gap in Documentation?

1. Use Party Mode to discuss
2. Update the relevant doc
3. Commit your changes

### Documentation Outdated?

1. Check when it was last updated
2. Update the file
3. Update "Last Updated" date
4. Commit your changes

### Need a New Document Type?

1. Propose in Party Mode
2. Get consensus from agents
3. Create the document
4. Update this README

---

## Document Status Legend

| Status         | Meaning                   |
| -------------- | ------------------------- |
| ⭐ CRITICAL    | Must-read for all agents  |
| ✅ Complete    | Fully documented          |
| 🚧 In Progress | Partially documented      |
| 📋 Planned     | To be created             |
| 🔴 Deprecated  | Historical reference only |

### Current Document Status

| Document                                              | Status         |
| ----------------------------------------------------- | -------------- |
| `project-context.md`                                  | ⭐ CRITICAL ✅ |
| `active-sprint.md`                                    | ⭐ CRITICAL ✅ |
| `migration-from-droidz.md`                            | ✅ Complete    |
| `epic-summaries/epic-101-progress-tracking.md`        | ✅ Complete    |
| `epic-summaries/epic-102-quiz-system.md`              | ✅ Complete    |
| `architecture-decisions/001-convex-as-backend.md`     | ✅ Complete    |
| `architecture-decisions/002-nextjs-app-router.md`     | ✅ Complete    |
| `architecture-decisions/005-usestate-less-pattern.md` | ✅ Complete    |

---

**Welcome to BMAD! Let's build something great together.** 🚀

---

**Last Updated**: 2026-01-15
**Maintained By**: All BMAD Agents
**Next Review**: After Sprint 2 completion
