# Architecture Decisions

This folder documents key technical decisions made for the Doctrina LMS project, following the **Architecture Decision Record (ADR)** pattern.

## Purpose

Architecture decisions capture:
- **What** decision was made
- **Why** it was made (context, constraints, trade-offs)
- **When** it was made
- **Consequences** (positive and negative)

These records prevent re-litigating decisions and provide context for future developers.

## Format

Each decision follows this structure:

```markdown
# [Number]. [Title]

**Date**: YYYY-MM-DD
**Status**: Accepted | Superseded | Deprecated
**Deciders**: [Who made the decision]

## Context

[What is the issue we're seeing that is motivating this decision?]

## Decision

[What is the change that we're actually proposing or doing?]

## Rationale

[Why did we choose this option over alternatives?]

## Consequences

### Positive
- [Good outcome 1]
- [Good outcome 2]

### Negative
- [Trade-off 1]
- [Trade-off 2]

## Alternatives Considered

### Alternative 1
- **Pros**: ...
- **Cons**: ...
- **Why Not**: ...

### Alternative 2
- **Pros**: ...
- **Cons**: ...
- **Why Not**: ...

## References

- [Link to relevant docs]
- [Link to discussions]
```

## Available Decisions

### Core Architecture

1. **001-convex-as-backend.md**: Why we chose Convex over traditional backend
2. **002-nextjs-app-router.md**: Why we chose Next.js App Router over Pages Router
3. **003-clerk-authentication.md**: Why we chose Clerk for authentication
4. **004-stripe-payments.md**: Why we chose Stripe for payments

### Data & State Management

5. **005-usestate-less-pattern.md**: Why we avoid useState for server data
6. **006-real-time-by-default.md**: Why we use Convex real-time subscriptions

### Testing & Quality

7. **007-100-percent-backend-coverage.md**: Why we require 100% backend test coverage
8. **008-tdd-red-green-refactor.md**: Why we follow TDD methodology

### Code Organization

9. **009-server-components-default.md**: Why server components are the default
10. **010-form-provider-controller.md**: Why we use FormProvider + Controller pattern

## How to Use These Decisions

### For New Team Members

Read these decisions to understand **why** the codebase is structured the way it is.

### For Technical Discussions

When debating architecture changes:
1. Check if a decision already exists
2. If it does, reference it or propose superseding it
3. If it doesn't, create a new ADR

### For Implementation

When implementing a feature:
1. Check relevant ADRs
2. Follow the established patterns
3. If you must deviate, document why (create new ADR or update existing)

## Decision Status

- **Accepted**: Current, active decision
- **Superseded**: Replaced by a newer decision (reference the superseding ADR)
- **Deprecated**: No longer relevant but kept for historical context

## Creating New Decisions

### When to Create an ADR

Create an ADR when:
- Choosing between multiple technical approaches
- Making a decision with long-term impact
- Establishing a pattern that will be followed project-wide
- Reversing or superseding a previous decision

### How to Create an ADR

1. Copy the template above
2. Number it sequentially (check last ADR number)
3. Write clearly and concisely
4. Include rationale and alternatives
5. Commit to the repo
6. Update this README with a link

## Relationship to Other Documentation

### ADRs vs Project Context

- **ADRs**: Explain WHY a decision was made
- **Project Context** (`_bmad-output/project-context.md`): Documents WHAT patterns to follow (the outcome of ADRs)

### ADRs vs Epic Summaries

- **ADRs**: Cross-cutting architectural decisions
- **Epic Summaries**: Feature-specific technical details

### ADRs vs Standards

- **ADRs**: Strategic decisions (tech stack, architecture patterns)
- **Standards** (`.claude/standards/`, `droidz/standards/`): Tactical coding patterns

## Quick Reference by Topic

### Tech Stack Decisions

- 001-convex-as-backend.md
- 002-nextjs-app-router.md
- 003-clerk-authentication.md
- 004-stripe-payments.md

### Development Patterns

- 005-usestate-less-pattern.md
- 006-real-time-by-default.md
- 009-server-components-default.md
- 010-form-provider-controller.md

### Quality Standards

- 007-100-percent-backend-coverage.md
- 008-tdd-red-green-refactor.md

---

**Last Updated**: 2026-01-15
**Maintained By**: Architect (Winston) + Dev team
