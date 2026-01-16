# Migration from Droidz to BMAD

**Date**: 2026-01-15
**Project**: Doctrina LMS
**Migration Type**: AI Workflow System Change
**Status**: In Progress

---

## Executive Summary

This document explains the transition from **Droidz** (previous AI workflow system) to **BMAD** (Build Methods for Agile Development) for the Doctrina LMS project. It serves as a guide for understanding what changed, what stayed the same, and how to work with BMAD agents.

---

## What is Droidz?

**Droidz** was the previous AI-assisted development workflow system used for this project, featuring:

- **Spec-driven development**: `/shape-spec` → `/write-spec` → `/create-tasks` → `/orchestrate-tasks`
- **Standards-based**: Comprehensive standards in `droidz/standards/` folder
- **Product planning**: Mission, roadmap, and tech stack documentation in `droidz/product/`
- **Task orchestration**: Automated task breakdown and specialist assignment
- **Slash commands**: `/plan-product`, `/shape-spec`, `/write-spec`, `/implement-tasks`, etc.

### Droidz Workflow Phases

1. **Phase 0**: Setup Standards (`/standards-shaper`)
2. **Phase 1**: Product Planning (`/plan-product`)
3. **Phase 2**: Spec Shaping (`/shape-spec`)
4. **Phase 3**: Spec Writing (`/write-spec`)
5. **Phase 4**: Task Creation (`/create-tasks`)
6. **Phase 5**: Task Orchestration (`/orchestrate-tasks`)
7. **Phase 6**: Implementation (`/implement-tasks`)
8. **Phase 7**: Iteration & Refinement

---

## What is BMAD?

**BMAD** (Build Methods for Agile Development) is a multi-agent workflow system that replaces Droidz, featuring:

- **Multi-agent collaboration**: Party Mode with specialized agents (PM, Architect, Dev, Test Architect, etc.)
- **Phase-based workflows**: Solutioning (PRD, Architecture, UX) → Planning (Epics, Stories) → Implementation (Dev, Review)
- **Story-driven development**: Epic breakdown → Story creation → Story implementation
- **Skill-based commands**: `/bmad:bmm:workflows:prd`, `/bmad:bmm:workflows:create-epics-and-stories`, `/bmad:bmm:workflows:dev-story`
- **Agent personalities**: Each agent has a distinct communication style and expertise

### BMAD Workflow Phases

1. **Phase 1 - Foundation**: Workflow initialization, project type determination
2. **Phase 2 - Solutioning**: PRD, Architecture, UX Design creation
3. **Phase 3 - Planning**: Epic breakdown, Story creation
4. **Phase 4 - Implementation**: Story execution, code review, retrospectives

---

## Key Differences

| Aspect                     | Droidz                                           | BMAD                                                       |
| -------------------------- | ------------------------------------------------ | ---------------------------------------------------------- |
| **Approach**               | Spec-driven (single spec → tasks)                | Story-driven (epics → stories → tasks)                     |
| **Agents**                 | Generic specialists (backend, frontend, test)    | Named personas (Amelia, Winston, Bob, etc.)                |
| **Documentation Location** | `droidz/` folder                                 | `_bmad-output/` + `docs/` folders                          |
| **Commands**               | `/shape-spec`, `/write-spec`, `/implement-tasks` | `/bmad:bmm:workflows:prd`, `/bmad:bmm:workflows:dev-story` |
| **Standards**              | `droidz/standards/` (comprehensive)              | `.claude/standards/` (Next.js, React, Convex-specific)     |
| **Orchestration**          | Orchestration YAML with specialist assignment    | Sprint planning with story prioritization                  |
| **Context Files**          | `spec.md` + `tasks.md`                           | Story files + Story Context XML                            |
| **Testing Strategy**       | Standards-based, documented in `testing.md`      | 100% backend coverage enforced, TDD red-green-refactor     |

---

## What Stayed the Same

### 1. Project Fundamentals

- **Tech Stack**: Next.js 16, React 19, Convex, Clerk, Stripe (unchanged)
- **Architecture**: Serverless, real-time, type-safe (unchanged)
- **Git Repository**: Same repo, same branch (`master`)
- **Package Manager**: Yarn 4.10.3 (unchanged)

### 2. Coding Standards

- **TypeScript Strict Mode**: Still enforced
- **100% Test Coverage**: Still required for backend
- **Zero Warnings Policy**: ESLint still requires 0 warnings
- **FormProvider + Controller Pattern**: React Hook Form patterns preserved
- **Convex Real-Time Pattern**: useState-less pattern still in use

### 3. Existing Documentation

- **PRD**: `docs/PRD.md` (preserved)
- **Architecture**: `docs/ARCHITECTURE.md` (preserved)
- **Database Schema**: `docs/DB-SCHEMA.md` (preserved)
- **Epic Definitions**: `docs/EPICS.md` (preserved)
- **Sprint Status**: `docs/sprint-status.yaml` (preserved)

### 4. Project Structure

- **Frontend**: `app/`, `components/`, `lib/`, `hooks/` (unchanged)
- **Backend**: `convex/` (unchanged)
- **Tests**: `convex/__test__/` (unchanged)
- **Stories**: `docs/stories/` (unchanged)

---

## What Changed

### 1. Documentation Organization

#### Before (Droidz)

```
droidz/
├── product/
│   ├── mission.md
│   ├── roadmap.md
│   └── tech-stack.md
├── specs/
│   └── [YYYY-MM-DD-feature-name]/
│       ├── spec.md
│       ├── tasks.md
│       ├── orchestration.yml
│       └── planning/
└── standards/
    ├── global/
    ├── backend/
    └── frontend/
```

#### After (BMAD)

```
_bmad-output/
├── project-context.md          # NEW: Consolidated project context
├── active-sprint.md            # NEW: Current sprint info
├── migration-from-droidz.md    # NEW: This file
├── epic-summaries/             # NEW: Epic tech context
└── architecture-decisions/     # NEW: Key decisions

docs/
├── PRD.md                      # PRESERVED
├── ARCHITECTURE.md             # PRESERVED
├── EPICS.md                    # PRESERVED
├── stories/                    # PRESERVED (now primary)
│   ├── story-101.1.md
│   ├── story-context-101.1.xml
│   └── ...
└── sprint-status.yaml          # PRESERVED

.claude/
└── standards/                  # NEW: Claude-specific standards
    ├── nextjs.md
    ├── react-convex.md
    ├── react.md
    └── typescript.md
```

### 2. Workflow Commands

#### Before (Droidz)

```bash
/shape-spec               # Shape feature requirements
/write-spec               # Generate formal spec
/create-tasks             # Break spec into tasks
/orchestrate-tasks        # Assign specialists
/implement-tasks          # Execute implementation
```

#### After (BMAD)

```bash
/bmad:bmm:workflows:prd                        # Create/edit PRD
/bmad:bmm:workflows:create-architecture        # Architecture design
/bmad:bmm:workflows:create-epics-and-stories   # Epic breakdown
/bmad:bmm:workflows:create-story               # Create individual story
/bmad:bmm:workflows:dev-story                  # Implement story
/bmad:bmm:workflows:code-review                # Adversarial review
/bmad:bmm:workflows:sprint-status              # Check sprint state
/bmad:core:workflows:party-mode                # Multi-agent discussion
```

### 3. Agent Interactions

#### Before (Droidz)

- **Generic specialists**: "backend-specialist", "frontend-specialist", "test-specialist"
- **Task-focused**: Assigned to specific task groups
- **No personalities**: Agents were tools, not collaborators

#### After (BMAD)

- **Named agents with personalities**:
  - 📊 **Mary (Analyst)**: Business Analyst, excited treasure hunter
  - 🏗️ **Winston (Architect)**: System Architect, calm and pragmatic
  - 💻 **Amelia (Developer)**: Senior Engineer, ultra-succinct
  - 📋 **John (PM)**: Product Manager, asks WHY relentlessly
  - 🚀 **Barry (Quick Flow)**: Elite Full-Stack, ruthlessly efficient
  - 🏃 **Bob (Scrum Master)**: Story prep specialist, zero ambiguity
  - 🧪 **Murat (Test Architect)**: Master test architect, data + instinct
  - 📚 **Paige (Tech Writer)**: Documentation specialist, clarity master
  - 🎨 **Sally (UX Designer)**: UX expert, empathetic storyteller

- **Collaborative**: Agents discuss in party mode
- **Story-focused**: Agents work on stories, not generic task groups

### 4. Implementation Artifacts

#### Before (Droidz)

- **Primary**: `spec.md` (single specification document)
- **Tasks**: `tasks.md` (task breakdown with checkboxes)
- **Orchestration**: `orchestration.yml` (specialist assignments)
- **Context**: Embedded in spec.md

#### After (BMAD)

- **Primary**: Story files (`story-101.1.md`, `story-102.2.md`)
- **Context**: Story Context XML (`story-context-101.1.xml`)
- **Epic Summaries**: Tech context per epic (`epic-summaries/epic-101.md`)
- **Sprint Tracking**: `sprint-status.yaml` (YAML-based tracking)

---

## Migration Decisions

### What We Kept from Droidz

1. **Standards**: Preserved `droidz/standards/` folder as reference
   - Backend standards (API design, Convex patterns, authentication)
   - Frontend standards (components, styling, state management)
   - Global standards (error handling, testing, security)

2. **Product Documentation**: Preserved `droidz/product/` folder
   - Mission statement
   - Roadmap
   - Tech stack decisions

3. **Specs Folder**: Preserved `droidz/specs/` for historical reference
   - Contains early feature planning
   - Shows evolution of requirements

### What We Created for BMAD

1. **project-context.md**: Consolidated tech stack, patterns, anti-patterns
2. **active-sprint.md**: Living document for current sprint
3. **migration-from-droidz.md**: This file (transition guide)
4. **epic-summaries/**: Tech context for each epic
5. **architecture-decisions/**: Key technical choices documented

### What We Deprecated

1. **Orchestration YAMLs**: No longer using specialist assignment files
2. **Task.md files**: Replaced by Story files with tasks/subtasks
3. **Spec.md files**: Replaced by Epic definitions + Story breakdown
4. **Implementation prompts**: No longer generating prompts; agents work directly on stories

---

## How to Work with BMAD

### For Developers (If You're Implementing Code)

1. **Check Sprint Status**:

   ```bash
   # Read current sprint
   cat _bmad-output/active-sprint.md
   ```

2. **Read Story File**:

   ```bash
   # Example
   cat docs/stories/story-101.1.md
   ```

3. **Read Story Context XML**:

   ```bash
   # Example
   cat docs/stories/story-context-101.1.xml
   ```

4. **Check Project Context**:

   ```bash
   cat _bmad-output/project-context.md
   ```

5. **Implement Using TDD**:
   - Write failing test
   - Write minimal code to pass
   - Refactor while keeping tests green

6. **Use BMAD Workflows**:
   ```bash
   /bmad:bmm:workflows:dev-story     # Execute story
   /bmad:bmm:workflows:code-review   # Review code
   ```

### For Product/Planning Work

1. **Update PRD**:

   ```bash
   /bmad:bmm:workflows:prd
   ```

2. **Create/Update Epics**:

   ```bash
   /bmad:bmm:workflows:create-epics-and-stories
   ```

3. **Check Sprint Progress**:

   ```bash
   /bmad:bmm:workflows:sprint-status
   ```

4. **Multi-Agent Discussion**:
   ```bash
   /bmad:core:workflows:party-mode
   ```

### For Asking Questions

- Use **Party Mode** for collaborative discussions
- Agents will naturally engage based on their expertise
- Mary (Analyst) for requirements analysis
- Winston (Architect) for technical design
- John (PM) for product direction
- Murat (Test Architect) for testing strategy

---

## Common Questions

### Q: Should I delete the droidz/ folder?

**A**: No. Keep it as reference. The standards are still valuable, and historical specs document decisions.

### Q: Can I still use droidz commands?

**A**: No. Droidz commands (`/shape-spec`, `/write-spec`, etc.) are no longer available. Use BMAD workflows instead.

### Q: Where do I find coding standards now?

**A**:

1. **Primary**: `.claude/standards/` (Next.js, React, TypeScript, Convex-specific)
2. **Reference**: `droidz/standards/` (broader patterns, still valid)
3. **Context**: `_bmad-output/project-context.md` (project-specific patterns)

### Q: What happened to the orchestration.yml files?

**A**: BMAD doesn't use orchestration files. Instead:

- Stories define tasks/subtasks
- Sprint planning determines order
- Dev agent (Amelia) implements stories sequentially
- Code review workflow validates completion

### Q: How do I know what to work on next?

**A**: Check `_bmad-output/active-sprint.md` → "Sprint Backlog (Ordered by Priority)" section.

### Q: Where are the specs?

**A**:

- **Epics**: High-level feature definitions in `docs/EPICS.md`
- **Stories**: Implementation-ready specs in `docs/stories/story-XXX.md`
- **Context**: Story Context XML files provide implementation guidance

### Q: How do I track progress?

**A**:

- **YAML**: `docs/sprint-status.yaml` (official tracking)
- **Markdown**: `docs/bmm-workflow-status.md` (human-readable)
- **Active Sprint**: `_bmad-output/active-sprint.md` (current sprint details)

### Q: What if I need to reference an old droidz spec?

**A**: They're preserved in `droidz/specs/[date]-[feature-name]/`. Use them as historical reference but follow current story definitions for implementation.

---

## Migration Checklist

### Completed ✅

- [x] Preserve droidz/ folder for reference
- [x] Preserve docs/ folder (PRD, Architecture, Epics, Stories)
- [x] Create \_bmad-output/ folder structure
- [x] Create project-context.md (consolidated patterns)
- [x] Create active-sprint.md (current sprint details)
- [x] Create migration-from-droidz.md (this file)
- [x] Install BMAD agents (via agent-manifest.csv)

### In Progress 🚧

- [ ] Create epic-summaries/ (tech context for each epic)
- [ ] Create architecture-decisions/ (document key choices)
- [ ] Update README.md to reference BMAD workflows
- [ ] Train team on BMAD workflows
- [ ] Document BMAD best practices learned

### Future Tasks 📋

- [ ] Archive old droidz specs (move to archive folder)
- [ ] Create BMAD cheat sheet for common tasks
- [ ] Update onboarding docs to reference BMAD
- [ ] Record video walkthroughs of BMAD workflows

---

## Benefits of BMAD Over Droidz

### 1. Multi-Agent Collaboration

**Before**: Single agent working in isolation
**After**: Multiple agents with expertise collaborate in real-time

### 2. Story-Driven Implementation

**Before**: Large specs broken into generic tasks
**After**: Epics broken into implementation-ready stories with context

### 3. Real-Time Progress Tracking

**Before**: Manual task checkbox updates
**After**: YAML-based sprint tracking with status visualization

### 4. Adversarial Code Review

**Before**: Self-review or manual review
**After**: Adversarial review agent finds 3-10 issues per story

### 5. Context Preservation

**Before**: Context embedded in spec.md
**After**: Story Context XML provides rich implementation context

### 6. Agent Personalities

**Before**: Generic tool-like agents
**After**: Agents with distinct communication styles and expertise

---

## Challenges & Solutions

### Challenge 1: Learning New Workflow

**Issue**: Team familiar with droidz workflows needs to learn BMAD
**Solution**:

- This migration guide documents the transition
- Party Mode allows asking BMAD agents for help
- `_bmad-output/` docs provide quick reference

### Challenge 2: Duplicate Documentation

**Issue**: Standards exist in both `droidz/` and `.claude/`
**Solution**:

- `.claude/standards/` are authoritative for tech-specific patterns
- `droidz/standards/` remain as broader reference
- `project-context.md` consolidates critical patterns

### Challenge 3: Finding Right Workflow

**Issue**: Many BMAD workflows available, which to use?
**Solution**:

- Check `_bmad-output/active-sprint.md` for current work
- Use `/bmad:bmm:workflows:sprint-status` to get guidance
- Use Party Mode to ask agents for help

### Challenge 4: Story vs Spec Mindset

**Issue**: Droidz used monolithic specs; BMAD uses granular stories
**Solution**:

- Epics provide high-level context (like old specs)
- Stories provide implementation-ready work (like old tasks)
- Story Context XML provides rich context (like old spec details)

---

## Tips for Success with BMAD

### 1. Start with Party Mode

When unsure, launch Party Mode and ask questions. Agents will collaborate to help.

### 2. Read project-context.md First

Before implementing ANY story, read `_bmad-output/project-context.md`. It's the bible.

### 3. Follow TDD Red-Green-Refactor

BMAD agents expect TDD. Write failing tests, make them pass, refactor.

### 4. Use Story Context XML

Story Context XML files provide critical implementation context. Always read them.

### 5. Trust the Agent Personalities

Agents have distinct styles for a reason. Mary finds gaps, Winston designs pragmatically, Amelia codes precisely.

### 6. Check Sprint Status Regularly

`_bmad-output/active-sprint.md` is a living document. Check it for current priorities.

### 7. Reference Legacy Docs

`droidz/` folder still has valuable standards and decisions. Use as reference.

---

## Conclusion

The migration from Droidz to BMAD represents a shift from **spec-driven development** to **story-driven multi-agent collaboration**. While the underlying tech stack and coding standards remain the same, the workflow and documentation structure have evolved to support:

- More granular, implementation-ready stories
- Multi-agent collaboration with distinct personalities
- Richer context preservation (Story Context XML)
- Adversarial code review for quality
- Real-time sprint tracking

**Key Takeaway**: Droidz taught us how to build systematically. BMAD takes that foundation and adds collaborative intelligence, richer context, and more focused implementation artifacts.

---

**Document Status**: Living Document
**Last Updated**: 2026-01-15
**Next Review**: After Sprint 2 completion
**Maintained By**: BMAD core team
