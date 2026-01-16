# Planning Documents

This folder contains all critical planning documents for Doctrina LMS.

## Purpose

BMAD agents reference these documents when implementing features. All planning documents are now consolidated in this location.

---

## Core Planning Documents

### 1. Product Requirements Document (PRD)

**Location**: `PRD.md` (this folder)

**Contains**:
- Vision and mission
- Strategic goals (50+ courses, 50+ verified instructors by Month 3)
- Problem statement (market problem for students and instructors)
- Solution overview
- Target users (students, instructors)
- Core features
- Success metrics

**When to Read**:
- Understanding product vision
- Making product decisions
- Prioritizing features

---

### 2. Architecture Document

**Location**: `ARCHITECTURE.md` (this folder)

**Contains**:
- System architecture overview
- Tech stack rationale (Next.js, Convex, Clerk, Stripe)
- Design philosophy (serverless-first, real-time, type-safe)
- Layer breakdown (Client, Next.js, Backend Services)
- Integration patterns
- Security architecture

**When to Read**:
- Before implementing new features
- Making architectural decisions
- Understanding system design

---

### 3. Database Schema

**Location**: `DB-SCHEMA.md` (this folder)

**Contains**:
- Complete table definitions
- Relationships and indexes
- Query patterns
- Schema evolution guide
- Migration roadmap
- Performance best practices

**When to Read**:
- Before creating/modifying Convex tables
- Understanding data relationships
- Writing queries

---

### 4. Epic Definitions

**Location**: `EPICS.md` (this folder)

**Contains**:
- All epic definitions (EPIC-101 through EPIC-110)
- Problem statements
- User stories
- Acceptance criteria
- Story point estimates
- Priority levels (P0, P1, P2)

**When to Read**:
- Sprint planning
- Story creation
- Understanding feature scope

---

### 5. Testing Strategy

**Location**: `TESTING-STRATEGY.md` (this folder)

**Contains**:
- Testing philosophy (100% backend coverage)
- Test types (unit, integration, E2E)
- Testing tools (Vitest, convex-test)
- Coverage requirements
- CI/CD integration

**When to Read**:
- Before writing tests
- Setting up new test suites
- Debugging test failures

---

### 6. Security & Compliance

**Location**: `SECURITY-COMPLIANCE.md` (this folder)

**Contains**:
- HIPAA considerations
- Data privacy requirements
- Authentication/authorization patterns
- CE credit compliance
- Instructor verification requirements

**When to Read**:
- Implementing auth features
- Handling sensitive data
- Instructor vetting workflows

---

### 7. Convex Guide

**Location**: `CONVEX.md` (this folder)

**Contains**:
- Convex-specific patterns
- Query vs mutation guidelines
- Real-time subscription patterns
- Authentication in Convex
- Testing Convex functions

**When to Read**:
- Writing Convex backend functions
- Implementing real-time features
- Debugging Convex issues

---

## Quick Access

```bash
# View PRD
cat _bmad-output/planning-docs/PRD.md

# View Architecture
cat _bmad-output/planning-docs/ARCHITECTURE.md

# View Database Schema
cat _bmad-output/planning-docs/DB-SCHEMA.md

# View Epics
cat _bmad-output/planning-docs/EPICS.md

# View Testing Strategy
cat _bmad-output/planning-docs/TESTING-STRATEGY.md

# View Security Compliance
cat _bmad-output/planning-docs/SECURITY-COMPLIANCE.md

# View Convex Guide
cat _bmad-output/planning-docs/CONVEX.md
```

---

## Document Hierarchy

```
Strategic Level:
└── PRD.md (product vision, goals, metrics)

Architectural Level:
├── ARCHITECTURE.md (system design, tech stack)
└── SECURITY-COMPLIANCE.md (security requirements)

Implementation Level:
├── DB-SCHEMA.md (data model)
├── EPICS.md (feature definitions)
├── TESTING-STRATEGY.md (test requirements)
└── CONVEX.md (backend patterns)

Story Level:
├── docs/stories/story-XXX.md (implementation details)
└── docs/stories/story-context-XXX.xml (context)

Sprint Level:
├── docs/sprint-status.yaml (official tracking)
└── docs/bmm-workflow-status.md (human-readable status)
```

---

## Relationship to BMAD Docs

### BMAD Docs (`_bmad-output/`)
- **project-context.md**: Consolidated patterns (references planning docs)
- **epic-summaries/**: Technical implementation context
- **architecture-decisions/**: WHY decisions were made (ADRs)
- **active-sprint.md**: What needs to be done next

### Planning Docs (`_bmad-output/planning-docs/`)
- **Official source of truth** for product, architecture, schema
- **Living documents** updated as product evolves
- **All planning docs** consolidated in this folder

---

## When to Update Planning Docs

### PRD.md
- **Update When**: Product vision changes, new features added to scope
- **Owner**: PM (John)

### ARCHITECTURE.md
- **Update When**: Major tech stack changes, architectural patterns change
- **Owner**: Architect (Winston)

### DB-SCHEMA.md
- **Update When**: Tables added/modified, indexes changed
- **Owner**: Architect (Winston), Dev (Amelia)

### EPICS.md
- **Update When**: New epics created, epic scope changes
- **Owner**: SM (Bob), PM (John)

### TESTING-STRATEGY.md
- **Update When**: Test approach changes, new tools added
- **Owner**: Test Architect (Murat)

### SECURITY-COMPLIANCE.md
- **Update When**: New compliance requirements, security patterns change
- **Owner**: Architect (Winston), Analyst (Mary)

### CONVEX.md
- **Update When**: New Convex patterns discovered, best practices evolve
- **Owner**: Dev (Amelia), Architect (Winston)

---

## Tips for BMAD Agents

1. **Read planning docs BEFORE implementing** - Don't guess, reference the source
2. **Check freshness** - Look at "Last Updated" dates
3. **Combine with epic summaries** - Planning docs (WHAT) + Epic summaries (HOW)
4. **Reference in code** - Add comments linking to relevant planning docs
5. **Update when you learn** - Found a gap? Update the planning doc

---

**Last Updated**: 2026-01-15
**Maintained By**: All BMAD Agents
