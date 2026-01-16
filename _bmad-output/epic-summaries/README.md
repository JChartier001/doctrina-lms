# Epic Summaries

This folder contains technical context and implementation details for each epic in the Doctrina LMS project.

## Purpose

Epic summaries provide **deep technical context** that agents need when implementing stories. They bridge the gap between high-level epic definitions (`docs/EPICS.md`) and granular story files (`docs/stories/`).

## What's Included in Each Summary

- **Problem Statement**: Why this epic exists
- **Technical Architecture**: Database schema, backend functions, integration points
- **Stories Breakdown**: All stories in the epic with status
- **Testing Strategy**: Test scenarios and coverage requirements
- **Security Considerations**: Authorization, validation, data integrity
- **Performance Considerations**: Query optimization, scalability
- **Dependencies**: Upstream and downstream epic dependencies
- **Known Limitations**: MVP scope constraints
- **Future Enhancements**: Post-MVP roadmap

## Available Epic Summaries

### Sprint 2 (Active)

- **epic-101-progress-tracking.md**: Lesson Progress Tracking System (13 pts)
- **epic-102-quiz-system.md**: Quiz Submission & Grading System (25 pts)

### Backlog (To Be Created)

- **epic-103-reviews.md**: Course Reviews & Ratings (8 pts)
- **epic-104-instructor-verification.md**: Instructor Verification Workflow (21 pts)
- **epic-105-instructor-payouts.md**: Instructor Payouts (21 pts)
- **epic-106-discussions.md**: Discussion Forums & Community (21 pts)
- **epic-107-analytics.md**: Enhanced Instructor Analytics (13 pts)
- **epic-108-ce-credits.md**: CE Credit Management (13 pts)
- **epic-109-mock-replacement.md**: Replace Mock Data with Real Queries (23 pts)
- **epic-110-testing.md**: Testing & Quality Assurance (26 pts)

## How to Use These Summaries

### For Dev Agents (Implementing Stories)

1. **Before starting a story**:
   - Read the epic summary for context
   - Read the specific story file (`docs/stories/story-XXX.md`)
   - Read the story context XML (`docs/stories/story-context-XXX.xml`)

2. **During implementation**:
   - Reference epic summary for architecture decisions
   - Check integration points
   - Follow testing strategy

3. **After completion**:
   - Verify all acceptance criteria met
   - Confirm test coverage requirements

### For Planning Agents (Creating Stories)

1. **Read epic summary** to understand technical scope
2. **Identify integration points** with other epics
3. **Break down stories** based on architecture sections
4. **Set realistic effort estimates** using complexity analysis

### For Review Agents (Code Review)

1. **Check against architecture** defined in epic summary
2. **Verify security considerations** addressed
3. **Confirm test coverage** meets requirements
4. **Validate performance** considerations handled

## Relationship to Other Documentation

### Epic Summaries vs Epic Definitions

- **Epic Definitions** (`docs/EPICS.md`): High-level user stories, business value, acceptance criteria
- **Epic Summaries** (this folder): Technical implementation details, architecture, testing

### Epic Summaries vs Story Files

- **Epic Summaries**: Broad context for all stories in the epic
- **Story Files** (`docs/stories/`): Granular tasks/subtasks for a single story

### Epic Summaries vs Project Context

- **Project Context** (`_bmad-output/project-context.md`): Project-wide patterns, tech stack, anti-patterns
- **Epic Summaries**: Epic-specific architecture and implementation details

## Naming Convention

Files are named: `epic-[number]-[short-name].md`

Examples:

- `epic-101-progress-tracking.md`
- `epic-102-quiz-system.md`
- `epic-103-reviews.md`

## Maintenance

### When to Update

- After epic is first planned (initial creation)
- After story completion (mark stories as complete)
- When architecture changes (document decisions)
- After epic retrospective (capture learnings)

### Who Updates

- **SM Agent (Bob)**: Creates initial epic summaries during planning
- **Dev Agent (Amelia)**: Updates status during implementation
- **Architect (Winston)**: Updates architecture sections when design changes
- **Test Architect (Murat)**: Updates testing strategy sections

## Quick Reference

### For Current Sprint Work

Check `_bmad-output/active-sprint.md` → Find current epic → Read that epic summary here.

### For Future Planning

Check `docs/EPICS.md` → Identify next epic → Read/create epic summary before story creation.

---

**Last Updated**: 2026-01-15
**Maintained By**: BMAD agents (SM, Dev, Architect)
